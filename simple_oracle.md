# Simple Oracle

## Summary 
In this challenge we are given access to a server that will tell us 
whether an encrypted message will decrypt, however it will not acutally
decrypt that message. With this knowledge we execute a padding oracle 
attack, where we bruteforce check every possible byte a block can be 
allowing us to recover the original message

## Script
This script access the flag and tests each byte individually in order
to check if the byte we expect to be valid which allows us to the find
the actual plaintext value.
```python
from base64 import b64encode, b64decode

from cryptography.hazmat.primitives.padding import PKCS7
from pwn import *

def oracle(io, iv, block):
    encoded = b64encode(iv + block)
    
    io.sendline(encoded)
    
    resp = io.recvline()
    return b"SUCCESS" in resp

def bruteforce_block(io, orig_iv, block):
    known_bytes = bytearray(16)
    
    # hack to avoid valid padding working properly lol
    skip = True

    # go over full block
    for pos in range(1, 17):
        # correct end padding because yeah
        end_padding = bytes(16 - (pos - 1)) + bytes([pos] * (pos - 1))

        # xor iv with known bytes + make known stuff valid
        mod_iv = xor(known_bytes, orig_iv, end_padding)
        
        # brute force the byte :(
        for cand in range(0 + skip, 256):
            # xor the core
            this_iv = bytearray(mod_iv)
            this_iv[-pos] ^= cand
            
            if oracle(io, this_iv, block):
                print(f"success! {cand = }")
                # xor padding byte with the candidate to recover the plaintext block
                known_bytes[-pos] = cand ^ pos
                break

        # </hack>
        skip = False

    return known_bytes


if __name__ == "__main__":
    # io = remote("chal.ctf-league.osusec.org", 1347)
    io = process(["python", "simple_oracle/server.py"])
    
    # get flag :)
    io.recvuntil(b"is: ")
    enc_flag = b64decode(io.recvline())
    
    flag_dec = bytes()
    for idx in range(0, len(enc_flag) - 16, 16):
        iv = enc_flag[idx:idx+16]
        block = enc_flag[idx+16:idx+32]
        
        decrypted = bruteforce_block(io, iv, block)
        flag_dec += decrypted
        
    unpadder = PKCS7(128).unpadder()
    flag = unpadder.update(flag_dec) + unpadder.finalize()
    print(flag.decode())  
```