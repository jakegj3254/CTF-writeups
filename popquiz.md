# Summary
For this challenge we are given a executable that we must decompile and analyze in order to find out what the server wants us to input in order for us to aquire the flag. The only tools used for this solution is Ghidra as well as netcat (to access the server with the program). The challenge is divided into 3 questions each of which need a specific input to progress, each question has it's own section below so feel free to skip to a specific one. After solving all three of these questions we are given the flag.

# Question 1
For this question we are given a function that seems to be comparing your input to a constant value. (See below for code)
```c
bool question_1(void)
{
  long in_FS_OFFSET;
  int local_18;
  int local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_14 = 0x2325;
  printf("What number am I thinking of? ");
  __isoc99_scanf(&DAT_00100e27,&local_18);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return local_14 == local_18;
}
```
As you can see, this program is returning whether local_14 (constant of 0x2325) equals local_18 (user input), in Ghidra we can hover over 0x2325 to find the value of it, which turns out to be a value of 8997, which gives us the awnser we require to get past the first question

# Question 2
This one is a bit more complicated however it is fairly easy to understand with some knowledge of ASCII values and some c syntax
```c
bool question_2(void)

{
  long in_FS_OFFSET;
  undefined4 local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  printf("Gimme some characters: ");
  local_14 = 0;
  __isoc99_scanf("\n%c%c%c%c",&local_14,(long)&local_14 + 1,(long)&local_14 + 2,(long)&local_14 + 3)
  ;
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
    __stack_chk_fail();
  }
  return (int)local_14._3_1_ + (int)(char)local_14 + (int)local_14._1_1_ + (int)local_14._2_1_ ==
         0x1a0;
}
```
For this question, if you know some c syntax (which I didn't when doing this challenge, but I'm going to pretend that I did), you can see the scanf function is grabbing the input that has a format of 4 characters. This means our input has to be 4 characters. Additionally, the input is split up into 4 characters where their ASCII values and comparing it to a constant value (0x1a0). This value turns out to be (with hovering in Ghidra) 416, so if we find a string of 4 characters that add up to 416 with their ASCII values, then we can pass this question. Four lower case h's work, as well as the input beep. Or use any other string that matches the conditions.

# Question 3
This is where knowing syntax of c is incredibly useful for figuring out what this program is trying to do. (Code is below)
```c
void question_3(void)

{
  long in_FS_OFFSET;
  undefined4 local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  puts("???");
  local_14 = 0;
  __isoc99_scanf(&DAT_00100e27,&local_14);
  check(local_14);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
```
We can see that this function passes the main logic to the function check, passing the user input into it. Checking that function we can see the main logic of question 3 (below)
```c
undefined8 check(uint param_1)

{
  int iVar1;
  int iVar2;
  size_t sVar3;
  undefined8 uVar4;
  long lVar5;
  undefined8 *puVar6;
  long in_FS_OFFSET;
  int local_128;
  int local_124;
  undefined8 local_118;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  puVar6 = &local_118;
  for (lVar5 = 0x1f; lVar5 != 0; lVar5 = lVar5 + -1) {
    *puVar6 = 0;
    puVar6 = puVar6 + 1;
  }
  *(undefined4 *)puVar6 = 0;
  *(undefined2 *)((long)puVar6 + 4) = 0;
  *(undefined *)((long)puVar6 + 6) = 0;
  sprintf((char *)&local_118,"%d",(ulong)param_1);
  sVar3 = strnlen((char *)&local_118,0xff);
  iVar2 = (int)sVar3;
  if ((sVar3 & 1) == 0) {
    if (iVar2 < 4) {
      uVar4 = 0;
    }
    else {
      for (local_128 = 1; iVar1 = iVar2 / 2, local_128 < iVar2 / 2; local_128 = local_128 + 1) {
        if (*(char *)((long)&local_118 + (long)local_128) <=
            *(char *)((long)&local_118 + (long)(local_128 + -1))) {
          uVar4 = 0;
          goto LAB_00100baf;
        }
      }
      do {
        local_124 = iVar1 + 1;
        if (iVar2 <= local_124) {
          uVar4 = 1;
          goto LAB_00100baf;
        }
        lVar5 = (long)iVar1;
        iVar1 = local_124;
      } while (*(char *)((long)&local_118 + (long)local_124) < *(char *)((long)&local_118 + lVar5));
      uVar4 = 0;
    }
  }
  else {
    uVar4 = 0;
  }
LAB_00100baf:
  if (local_10 == *(long *)(in_FS_OFFSET + 0x28)) {
    return uVar4;
  }
                    
  __stack_chk_fail();
}
```
After some analysis and variable renaming we can make this much more readable and understandable (see below) - there will be explanations for these changes, just making it cleaner to understand.
```c
  sprintf((char *)&user_input,"%d",(ulong)param_1);
  input_length = strnlen((char *)&user_input,0xff);
  redundant_string_length = (int)input_length;
  if ((input_length & 1) == 0) {
    if (redundant_string_length < 4) {
      conditions_met = 0;
    }
    else {
                    /* for loop for first half of input */
      for (for_counter = 1; iVar1 = redundant_string_length / 2,
          for_counter < redundant_string_length / 2; for_counter = for_counter + 1) {
        if (*(char *)((long)&user_input + (long)for_counter) <=
            *(char *)((long)&user_input + (long)(for_counter + -1))) {
          conditions_met = 0;
          goto LAB_00100baf;
        }
      }
      do {
        counter = iVar1 + 1;
        if (redundant_string_length <= counter) {
          conditions_met = 1;
          goto LAB_00100baf;
        }
        lVar2 = (long)iVar1;
        iVar1 = counter;
      } while (*(char *)((long)&user_input + (long)counter) < *(char *)((long)&user_input + lVar2));
      conditions_met = 0;
    }
  }
  else {
    conditions_met = 0;
  }
LAB_00100baf:
                    /* Second half of input loop */
  if (local_10 == *(long *)(in_FS_OFFSET + 0x28)) {
    return conditions_met;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```
I named the local118 variable user_input, since sprintf takes the input parameter and puts it into that variable, and changed the uVar 4 variable to be conditions_met since it will only be 1 if all of the conditions are met, and will give us the flag only if the conditions are met.

## Condition 1
Since there are 4 total conditions for this last question there will be subsection for each condition, in this case the first condition (that is easy to understand) is that the input must be greater than 4 in length, since conditions_met variable will be set to 0, not giving use the flag.

## Condition 2
Technically this is the first condition, but since it takes longer to explain than the one above I'm just going to call this the second condition for the sake of organization. Before checking the length of the string, the program actually checks input_length & 1 == 0. This means in c that it is doing a bitwise operation between the length of the input and the bits of 1. This means that your value must not have a 1 in the final bit, since if it does, then the 1 will combine with the other 1 to form a byte that doesn't have a 0 in the first bit. Meaning that only even numbered length inputs will pass this condition, since any odd value has a 1 in the first bit.

## Condition 3
The next condition is within the first loop (the for loop in this case). This for loop searches the first half of the input since it divides the length in half, and then it checks if the first half of the string is in ascending order(the previous element is less than the current element).

## Condition 4
The final condition is the do while loop, which if you look closely is actually functionally the same to a for loop again. This loop checks the second half of the string input, and does another comparison to different inputs in chronological order, however this time it checks if the previous element is greater than the current element being compared. 

Meaning all together we need a string that is longer than 4 length, has a even numbered length, has the first half of it be in ascending order, and the back half of the input be in descending order.

A good example of this is the input 1234554321, and inputting that we finally get the program to output the flag for us to complete the challenge.
* * *
* * *