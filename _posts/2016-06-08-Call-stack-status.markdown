---
layout: post
title:  "Call stack status"
date:   2016-06-11 21:46:00
categories: Linux
tags: ELF linux
---

* content
{:toc}


What is the call stack look like when called a func ?



## Source code

```c
#include <stdio.h>

int global_ini = 0x1111;
int global_unini;
int func_op(void) {return 0;}

int func(int i)
{

	unsigned long long val = 64;
	val = 0xffffeeeeddddcccc;
	global_unini = i;
	return global_unini;
}

#define MAX_WORD 16

int main(void)
{
	unsigned int i = 0;
	char words[MAX_WORD] = "Hello world";
	char word;

	int (*func_point)(void) = &func_op;
	
	i = 0xabcd;
	
	if(i != 0x1234)
	{
		i = 0;
	}
	
	while(i == 0)
	{
		i++;
	}

	func(0x5555);

	i = func_point();
	
	for(i = 0; i < MAX_WORD-1; i++)
	{
		word = words[i];
	}

	return 0;
}
```

## Dump the ELF file info

Get the structure of assemble using command: `readefl -a`, the out put is [here](https://github.com/ray525/ray525.github.io/blob/master/asset/src/assemble_readefl-a)

Get assembly code of the source code (it is in the .text section), run the command:	

`objdump -d assemble`, the output is [here](https://github.com/ray525/ray525.github.io/blob/master/asset/src/assemble_objdump-d)

## Debug assemble

### Set breakpoint

Run the command to start gdb, and stop at main:

```
gdb assemble
start
```

![gdb-start](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/gdb-start.png)

Run the command `b` to set the break point before call the func and in the func, then `continue` to make the program stop at calling func.

![gdb-continue](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/gdb-continue.png)

### Stack of main

![stack-main](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/stack-main.png)

We can find that:
The parameter 0x5555 will be push to stack first, then it is the return address. (“call” will push the address 0x804843c to stack), then eip point to the beginning of func
Then we can draw the outline of the VA space:

![stack-main-va](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/stack-main-va.png)

### Stack of func

![stack-func](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/stack-func.png)

We can find that, the program has been in func, then we can get the stack info:

![stack-func-detail](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/stack-func-detail.png)

It means it will push the ebp to the stack first, then make ebp point to esp, and sub esp, the stack outline like this:

![stack-func-va](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/stack-func-va.png)
