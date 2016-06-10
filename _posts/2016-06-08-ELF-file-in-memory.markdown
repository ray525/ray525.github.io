---
layout: post
title:  "ELF file in memory"
date:   2016-06-10 15:50:00
categories: Linux
tags: ELF linux
---

* content
{:toc}

![xielunyan](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/xielunyan.png)

ELF, what do you look like in memory???



## Virtual address space for common sense

- Outline

	![va-outline](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/va-outline.png)

- Detail

	![va-detail](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/va-detail.png)

## VA info for test

- We know the VA of the sections according the section header table. And we focus on the virtual address column.
![va-section](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/va-section.png)

- We can know the address of symbol according the .symtab section.
![va-symtab](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/va-symtab.png)

- The constant string in VA: 

	Check the output of “hexdump -C test”, we can see that the offset of string (such as: abc, hello…) is 540+4, and we can find the string is in .rodata section.
	![va-rodata](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/va-rodata.png)

## VA structure of test

- Segment mapping 

		Program Headers:
		  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
		  PHDR           0x000034 0x08048034 0x08048034 0x00100 0x00100 R E 0x4
		  INTERP         0x000134 0x08048134 0x08048134 0x00013 0x00013 R   0x1
			  [Requesting program interpreter: /lib/ld-linux.so.2]
		  LOAD           0x000000 0x08048000 0x08048000 0x00764 0x00764 R E 0x1000
		  LOAD           0x000764 0x08049764 0x08049764 0x00108 0x00114 RW  0x1000
		  DYNAMIC        0x000778 0x08049778 0x08049778 0x000c8 0x000c8 RW  0x4
		  NOTE           0x000148 0x08048148 0x08048148 0x00044 0x00044 R   0x4
		  GNU_EH_FRAME   0x0006c4 0x080486c4 0x080486c4 0x00024 0x00024 R   0x4
		  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x4

		 Section to Segment mapping:
		  Segment Sections...
		   00     
		   01     .interp 
		   02     .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rel.dyn .rel.plt .init .plt .text .fini .rodata .eh_frame_hdr .eh_frame 
		   03     .ctors .dtors .jcr .dynamic .got .got.plt .data .bss 
		   04     .dynamic 
		   05     .note.ABI-tag .note.gnu.build-id 
		   06     .eh_frame_hdr 
		   07     


- Segment structure

	![va-structure](https://raw.githubusercontent.com/ray525/ray525.github.io/master/asset/img/va-structure.png)