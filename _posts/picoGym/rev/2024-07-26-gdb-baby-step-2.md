---
title: GDB baby step 2
date: 2024-07-26 22:00:00 +0600
categories: [picoGym, rev]
tags: [picoGym, rev]     # TAG names should always be lowercase
author: refaat
description: Can you find the final value of the eax register?
---

## Description
Can you figure out what is in the eax register at the end of the main function? Put your answer in the picoCTF flag format: picoCTF{n} where n is the contents of the eax register in the decimal number base. If the answer was 0x11 your flag would be picoCTF{17}.
Debug this.

## Files given
We are given a binary executable file called debugger0_b

## Procedure

### Initial steps
First we need to make it executable using this command:

```terminal
chmod +x ./debugger0_b
```


Then run it to see if there’s any output : 
```terminal
./debugger0_b
```


Nothing here. Time to debug it using gdb.
```terminal
gdb ./debugger0_b
```


We see the following message :
![Desktop View](https://drive.google.com/thumbnail?id=1cCtf_hM4XTt_AOs3UyaTBVD6b_8tOcu1&sz=w1000){: w="700" h="400" }
*Image 1 : no debugging symbols found*


Now is a good time to disassemble the main function, but before that let’s set the flavor to intel syntax :
```
set disassembly-flavor intel 
disas main
```
![Desktop View](https://drive.google.com/thumbnail?id=1OfLRHmpSRUWy3ZbV-fiVOSA8Dx33_nQT&sz=w1000){: w="700" h="400" }
*Image 2 : assembler code dump for main*
Okay, there are two ways to go about this. We can understand what the program is doing and figure out the value of the eax register ourselves. Or we can just set a breakpoint before the program ends and display the value of the eax register.

You can skip to the end of this post if you’re interested in the shortcut.

### The Hard Way
In the beginning of the function, nothing much interesting happens. The key thing to note is the line of code at 0x40112a. : 
```
jmp    0x401136 <main+48>
```

This makes the execution jump to 0x401136. 
1. Note how the value stored at the address [rbp-0x8] is being copied to the eax register. If you look at the line 0x401123, the value stored at [rbp-0x8] is 0x0. So the value of eax is now 0x0.
2. Then, at the address [rbp-0x4], we are adding the value of eax. 
3. In the next line, we are incrementing the value stored at [rbp-0x8] by 1. 
4. Then we move the updated value of [rbp-0x8] to the eax. 
5. Finally, we compare the value with [rbp-0xc] and if eax < [rbp-0xc] then we repeat the whole process again.

Steps 1-5:
```
   0x000000000040112c <+38>:	mov    eax,DWORD PTR [rbp-0x8]
   0x000000000040112f <+41>:	add    DWORD PTR [rbp-0x4],eax
   0x0000000000401132 <+44>:	add    DWORD PTR [rbp-0x8],0x1
   0x0000000000401136 <+48>:	mov    eax,DWORD PTR [rbp-0x8]
   0x0000000000401139 <+51>:	cmp    eax,DWORD PTR [rbp-0xc]
   0x000000000040113c <+54>:	jl     0x40112c <main+38>
```
Then, we move the value of [rbp-0x4] to eax, and that is our final value.

So, let’s take a closer look at how the value at [rbp-0x4] is changing. In the beginning, the value is 0x1e0da , i.e. 123098. The value of eax is copied from [rbp-0x8] each time, and it increments by 1. 
```
123098 + 0 = 123098
123098 + 0 + 1 = 123099
123098 + 0 + 1 + 2 = 123101
.
.
.
123098 + 0 + 1 + 2 + .... + 606 = 307019
```

When eax is 607 , it doesn’t come back to the line 0x40112c, so the last value added is 606.

In other words, the arithmetic sum of the first 606 numbers is added to the initial value of [rbp-0x4].

The arithmetic sum is calculated as follows:

(first term + last term) * (no. of terms) /2


### The Shortcut

1. Set a breakpoint at the end of the code at address 0x401141 (after the final value of [rbp-0x4] is copied to the eax register.)
2. Run the code
3. Display the value of the eax register 

![Desktop View](https://drive.google.com/thumbnail?id=1uQzNuVft9dNEapvW6VT3KkClowg9f6v8&sz=w1000){: w="700" h="400" }
*Image 3 : The Shortcut*