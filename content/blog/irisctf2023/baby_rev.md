---
title: "IrisCTF 2023: baby_rev"
date: 2023-01-10T21:37:37+08:00
draft: false
author: "Jack Sun"
tags:
  - CTFs
  - Writeups
  - IrisCTF
  - IrisCTF 2023
  - Cybersecurity
  - Reverse
image: /images/writeups/IrisCTF2023/banner.png
description: ""
toc:
---

## Description

- Name: baby_rev
- Category: Reverse
- 50 pts, 251 Solves
- Written by `nope`

I needed to edit some documents for my homework but I don't want to buy a license to this software! Help me out pls?
{{%attachments style="orange" /%}}

## Solution

We can decompile the binary using Ghidra. The decompiled code for the main function is as follows:

```c
undefined8 main(void)

{
  size_t sVar1;
  long in_FS_OFFSET;
  int local_7c;
  char local_78 [104];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setvbuf(stdin,(char *)0x0,2,0);
  setvbuf(stdout,(char *)0x0,2,0);
  puts("Welcome to SuperTexEdit!\n");
  puts("To begin using SuperTexEdit, please enter your registration code.");
  printf("Code: ");
  __isoc99_scanf(&DAT_00102071,local_78);
  sVar1 = strlen(local_78);
  if (sVar1 == 0x20) {
    local_78[0] = local_78[0] + -0x69;
    local_78[1] = local_78[1] + -0x71;
    local_78[2] = local_78[2] + -0x67;
    local_78[3] = local_78[3] + -0x70;
    local_78[4] = local_78[4] + -0x5f;
    local_78[5] = local_78[5] + -0x6f;
    local_78[6] = local_78[6] + -0x60;
    local_78[7] = local_78[7] + -0x74;
    local_78[8] = local_78[8] + -0x65;
    local_78[9] = local_78[9] + -0x60;
    local_78[10] = local_78[10] + -0x59;
    local_78[11] = local_78[11] + -0x67;
    local_78[12] = local_78[12] + -99;
    local_78[13] = local_78[13] + -0x66;
    local_78[14] = local_78[14] + -0x61;
    local_78[15] = local_78[15] + -0x57;
    local_78[16] = local_78[16] + -100;
    local_78[17] = local_78[17] + -0x4e;
    local_78[18] = local_78[18] + -0x65;
    local_78[19] = local_78[19] + -0x5c;
    local_78[20] = local_78[20] + -0x5e;
    local_78[21] = local_78[21] + -0x4f;
    local_78[22] = local_78[22] + -0x49;
    local_78[23] = local_78[23] + -0x4a;
    local_78[24] = local_78[24] + -0x5c;
    local_78[25] = local_78[25] + -0x46;
    local_78[26] = local_78[26] + -0x4e;
    local_78[27] = local_78[27] + -0x54;
    local_78[28] = local_78[28] + -0x51;
    local_78[29] = local_78[29] + -0x48;
    local_78[30] = local_78[30] + -0x1c;
    local_78[31] = local_78[31] + -0x5e;
    local_7c = 0;
    while( true ) {
      if (0x1f < local_7c) {
        puts("Key Valid!");
        puts("SuperTexEdit booting up...");
                    /* WARNING: Subroutine does not return */
        abort();
      }
      if (local_7c != local_78[local_7c]) break;
      local_7c = local_7c + 1;
    }
    puts("Invalid code!");
  }
  else {
    puts("Invalid code!");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 1;
}
```

Analysing the while loop, we find that if `local_7c` is greater than `0x1f` (which is 31 in decimal), we will be told that our code is valid.

Initially, `local_7c` is set to 0. At the end of each iteration, it increments by one.

Following this logic, after 32 iterations, we should arrive at the valid code. However, the following check breaks out of the loop if `local_7c` is not equal to `local_78` with index `local_7c`:

```c
if (local_7c != local_78[local_7c]) break;
```

Thus, as `local_7c` increments by one each iteration, we can solve this challenge by setting the ASCII value of each letter in `local_78` equal to its own corresponding index. To do this, we add back the corresponding offset number plus the index to the ASCII value.

We can use Ghidra to convert each address byte offset to a decimal representation.

```c
    local_78[0] = local_78[0] + -105;
    local_78[1] = local_78[1] + -113;
    local_78[2] = local_78[2] + -103;
    local_78[3] = local_78[3] + -112;
    local_78[4] = local_78[4] + -95;
    local_78[5] = local_78[5] + -111;
    local_78[6] = local_78[6] + -96;
    local_78[7] = local_78[7] + -116;
    local_78[8] = local_78[8] + -101;
    local_78[9] = local_78[9] + -96;
    local_78[10] = local_78[10] + -89;
    local_78[11] = local_78[11] + -103;
    local_78[12] = local_78[12] + -99;
    local_78[13] = local_78[13] + -102;
    local_78[14] = local_78[14] + -97;
    local_78[15] = local_78[15] + -87;
    local_78[16] = local_78[16] + -100;
    local_78[17] = local_78[17] + -78;
    local_78[18] = local_78[18] + -101;
    local_78[19] = local_78[19] + -92;
    local_78[20] = local_78[20] + -94;
    local_78[21] = local_78[21] + -79;
    local_78[22] = local_78[22] + -73;
    local_78[23] = local_78[23] + -74;
    local_78[24] = local_78[24] + -92;
    local_78[25] = local_78[25] + -70;
    local_78[26] = local_78[26] + -78;
    local_78[27] = local_78[27] + -84;
    local_78[28] = local_78[28] + -81;
    local_78[29] = local_78[29] + -72;
    local_78[30] = local_78[30] + -28;
    local_78[31] = local_78[31] + -94;
    local_7c = 0;
```

Then, we can use a text editor like VS Code to clean up and isolate the indices.

```python
[-105, -113, -103, -112, -95, -111, -96, -116, -101, -96, -89, -103, -99, -102, -97, -87, -100, -78, -101, -92, -94, -79, -73, -74, -92, -70, -78, -84, -81, -72, -28, -94]
```

Now, we can write a simple python script to invert each number and add its index, before converting that to a character.

Final solve script:

```python
data = [-105, -113, -103, -112, -95, -111, -96, -116, -101, -96, -89, -103, -99, -102, -97, -87, -100, -78, -101, -92, -94, -79, -73, -74, -92, -70, -78, -84, -81, -72, -28, -94]
out = ""

for i in range(len(data)):
    out += chr(-data[i] + i)
print(out)
```

Flag: `irisctf{microsoft_word_at_home:}`
