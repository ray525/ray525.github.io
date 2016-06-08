---
layout: post
title:  "ELF file on disk"
date:   2016-06-08 12:22:00
categories: Linux
tags: ELF linux
---

* content
{:toc}

![magnifying](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/magnifying.png)

ELF, what do you look like???



## Source code and output

```c++
#include <stdio.h>

char *global_string = "abc";
char *global_string2 = "Hello World !";
const int const_num = 128;
int global_num = 1024;
int global_uninit_num;

int main()
{
	int stack_num = 2048;
	int stack_static_num = 123;
	char *stack_string = "Hello World !";
	printf("addr of global_string is 0x%x,\naddr of global_string2 is 0x%x,\naddr of global_num is 0x%x,\naddr of global_uninit_num is 0x%x,\naddr of stack_num is 0x%x,\naddr of stack_string is 0x%x \n", &global_string, &global_string2, &global_num, &global_uninit_num, &stack_num, &stack_string);
	printf("stack_static_num = %d, global_uninit_num=%d\n", stack_static_num, global_uninit_num);
	printf("the address in the global_string is 0x%x\nthe address in the global_string2 is 0x%x\nthe address in in the stack_string is 0x%x\n", global_string, global_string2, stack_string);
	return 0;
}
```

The build command is : `gcc -Wall -o0 test.c -o test`

Output is :

```c++
addr of global_string is 0x8049860,
addr of global_string2 is 0x8049864,
addr of global_num is 0x8049868,
addr of global_uninit_num is 0x8049874,
addr of stack_num is 0xbfcabc28,
addr of stack_string is 0xbfcabc24 
stack_static_num = 123, global_uninit_num=0
the address in the global_string is 0x8048544
the address in the global_string2 is 0x8048548
the address in in the stack_string is 0x8048548
```

## test(executable file) on disk

### Content of test

To see the conetent, run this command: `hexdump -C test`
The output of this command is [here](https://github.com/ray525/ray525.github.io/blob/master/asset/src/test_hexdump-C).

We can see that the content are digital numbers, it is the command which can be run on the computer.

![hexdump](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/hexdump.png)

Look the properties of test, we can see the size is <span style="color:green">5396 = 0X1514</span>. 

**NOTE:** Size on disk should be a little more than Size because of allocation units in Windows.

![test-size](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/test-size.png)

### The structure of test

#### Overview

![elf-overview](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/elf-overview.png)

**Note:** in our test file, there are two sections under section header table.

#### Analysis

To see structure, run the command:`readefl -a test`
The output is [here](https://github.com/ray525/ray525.github.io/blob/master/asset/src/test_readelf-a).

This is the ELF header info, it will show the outline the ELF file (test):

![elf-header](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/elf-header.png)

We can see that:

- The entry point is <span style='color:green'>0x8048340</span>, this is the address of <span style='color:green'>_start but not main</span>. We will discuss this later.
- The size of ELF header is 52
- The size of program header table is 32*8 and the offset is 52.
<span style='color:green'>32*8 + 52 = 308 = 0x134</span>
- The offset of section header table is <span style='color:green'>2500 = 0x9c4</span>, and the size of this table is <span style='color:green'>40*30 = 1200 = 0x4B0, 0x4B0 + 0x9c4 = 0xE74</span>. It means the scope of section header table is from: <span style='color:green'>0x9c4 to 0xE74</span>

This is the detail info of all the sections:

![elf-section-header](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/elf-section-header.png)

We can see that：

- The offset of <span style='color:green'>Section [1] is 0x134</span>, the content ahead is ELF header, and program header table.
- Section header table is <span style='color:green'>between [27] to [28]</span>. we have known that the section header table is from 0x9c4 to 0xE74, and <span style='color:green'>0x8c5 + 0xfc = 0x9c1</span>. For <span style='color:green'>aligning</span>, the offset of the Section header table is 0x9c4
- The size of the ELF file is <span style='color:green'>0x12d4+ 0x240 = 0x1514 = 5396</span>


#### The skeleton of the test

![test-skeleton-compare](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/test-skeleton-compare.png)
