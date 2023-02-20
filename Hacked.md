# Summary
For this challenge we are inspecting a dump of a windows machine when it was compromised, we are trying to find out what happened and then find the flag within the dump. 

# Tools 
First things first, the tool useful for this challenge would be volatility3 which is incredibly useful traversing and searching this memory dump. 

# Entry
Within the description of the challenge it says something about the user seeing a note on the screen. Which means a good plae to start would be to search the memory of the notepad process that was running. Running the command `python path/to/vol.py -f ./memory_dump.raw memmap --pid 1348 --dump` gives us the output 
```
The flag is the ntlm hash of the user wrapped.
Ex: flag{miH7CvSGyutSFtQB6w3AshXaDjbuqktXUQ}
The flag is the ntlm hash of the user wrapped.
Ex: flag{miH7CvSGyutSFtQB6w3AshXaDjbuqktXUQ}
```

From this we run the command `python3 volatility3-2.4.0/vol.py -f memory_dump.raw windows.hashdump` which grabs the hashes of the users which gives us the output 
```
Volatility 3 Framework 2.4.0
Progress:  100.00        PDB scanning finished                        
User    rid    lmhash    nthash

Administrator    500    aad3b435b51404eeaad3b435b51404ee    296788975c1ce6fafb8221f54f5aa68c
Guest    501    aad3b435b51404eeaad3b435b51404ee    31d6cfe0d16ae931b73c59d7e0c089c0
DefaultAccount    503    aad3b435b51404eeaad3b435b51404ee    31d6cfe0d16ae931b73c59d7e0c089c0
WDAGUtilityAccount    504    aad3b435b51404eeaad3b435b51404ee    67594cc62423c1d68acd9b5620eec6d0
Jimothy    1000    2d60630381393c46ac2e9b858d5427bc    80a1850fba580325595eb75c2ec50207
```
Taking Jimothy's hash wrapped in osu{} gives us the correct flag that we can submit