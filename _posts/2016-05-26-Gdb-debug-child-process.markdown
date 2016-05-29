---
layout: post
title:  "gdb on child process"
date:   2016-05-26 15:50:00
categories: gdb 
---

* content
{:toc}

## 1. Debug on the child process

### 1. debug the code in [shared library](http://visualgdb.com/gdbreference/commands/sharedlibrary)

{% highlight bash linenos %}
b main
r
info sharedlibrary
{% endhighlight %}

this will make program stoping at the main function, it means the shared library has been loaded, then you can get the source code info in the shared library when you set breakpoint on that.

### 2. debug on the [child process](https://sourceware.org/gdb/onlinedocs/gdb/Forks.html)

	void VddkManager::initialize()
	{
		WRITE_LOG1("VddkManager::initialize() begin.\n");
		
		const char *libDir = (isSAN_ ? cfg_.libDir : NULL);
		const char *configFile = cfg_.configPath;
		
		WRITE_LOG1("call VixDiskLib_InitEx().\n");
		if (cfg_.isRemote) {
			vixError_ = VixDiskLib_InitEx(VIXDISKLIB_VERSION_MAJOR,
										  VIXDISKLIB_VERSION_MINOR,
										  logFunc, logFunc, logFunc,
										  libDir, configFile);
			CHECK_AND_THROW(vixError_);

		} else {
			vixError_ = VixDiskLib_InitEx(VIXDISKLIB_VERSION_MAJOR,
										  VIXDISKLIB_VERSION_MINOR,
										  NULL, NULL, NULL,
										  NULL, NULL);
			CHECK_AND_THROW(vixError_);
		}

		WRITE_LOG1("VddkManager::initialize() end.\n");
	}

	
	set follow-fork-mode child
	b vddk_manager.cpp:106 
	c
	info sharedlibrary

    
**Note:** you need run the command `set follow-fork-mode child` first, then set the break point. only if you run the command in this order, the breakpoint can set on the child process.

### 3. debug on the parent process again.
because the the api(`VixDiskLib_InitEx`) in vmware will use `fork` to  create process, but we just want to debug our code, so we need to set the mode to parent before call `VixDiskLib_InitEx`. 


	void VmdkManager::open()
	{
		WRITE_LOG1("VmdkManager::open() begin\n");
		const VixDiskLibConnection conn = vddkMgr_.getConnection();
		const ConfigData& cfg = vddkMgr_.getConfig();
		assert(conn != NULL);
		
		uint32 openFlags = 0;
		if (vddkMgr_.isReadOnly()) {
			openFlags |= VIXDISKLIB_FLAG_OPEN_READ_ONLY;
		}

		WRITE_LOG1("call VixDiskLib_Open().\n");
		{
			//bool isExclusive = true;
			//ScopedResourceLock lk
			//(vddkMgr_.getConfig().lockResourceName, isExclusive);
			WRITE_LOG0("VmdkManager::open() begin\n");
			vixError_ = VixDiskLib_Open(conn, cfg.vmdkPath, openFlags, &handle_);
			WRITE_LOG0("VmdkManager::open() end\n");
		}
		CHECK_AND_THROW(vixError_);
		WRITE_LOG1("VmdkManager::open() end\n");
	}


	set follow-fork-mode parent
	b vddk_manager.cpp:361 
	c


### 4. [show the process](https://sourceware.org/gdb/onlinedocs/gdb/SVR4-Process-Information.html) info in gdb

## 2. Debug the shared library issue
### 1. issue: 
vmdkbap segment fault, because it need libcrypto.so.0.9.8, but it actually link to libcrypto.so.10

### 2. pre-knowledge
1. shared library [link and load search path](https://typecodes.com/cseries/gcclderrlibrarypath.html)

    GCC编译、链接生成可执行文件时，动态库的搜索路径顺序如下（注意不会递归性地在其子目录下搜索）：
    > 1、gcc编译、链接命令中的-L选项；
    2、gcc的环境变量的LIBRARY_PATH（多个路径用冒号分割）；
    3、gcc默认动态库目录：/lib:/usr/lib:usr/lib64:/usr/local/lib。
    
    执行二进制文件时的动态库搜索路径链接生成二进制可执行文件后，在运行程序加载动态库文件时，搜索的路径顺序如下：
    > 1、编译目标代码时指定的动态库搜索路径：用选项-Wl,rpath和include指定的动态库的搜索路径，比如gcc -Wl,-rpath,include -L. -ldltest hello.c，在执行文件时会搜索路径`./include`；
    2、环境变量LD_LIBRARY_PATH（多个路径用冒号分割）；
    3、在 /etc/ld.so.conf.d/ 目录下的配置文件指定的动态库绝对路径（通过ldconfig生效，一般是非root用户时使用）；
    4、gcc默认动态库目录：/lib:/usr/lib:usr/lib64:/usr/local/lib等。

2. show the soname of the shared library, the soname will show in the `ldd`


	# readelf -d libcrypto.so.1.0.1e
	Dynamic section at offset 0x1d23b0 contains 28 entries:
	  Tag        Type                         Name/Value
	 0x0000000000000001 (NEEDED)             Shared library: [libdl.so.2]
	 0x0000000000000001 (NEEDED)             Shared library: [libz.so.1]
	 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
	 0x000000000000000e (SONAME)             Library soname: [libcrypto.so.10]
	 0x0000000000000010 (SYMBOLIC)           0x0
	 0x000000000000000c (INIT)               0x3ba3a69568
	 0x000000000000000d (FINI)               0x3ba3b5dbe8
	 0x000000006ffffef5 (GNU_HASH)           0x3ba3a001f0
	 0x0000000000000005 (STRTAB)             0x3ba3a21da8
	 0x0000000000000006 (SYMTAB)             0x3ba3a09220
	 0x000000000000000a (STRSZ)              78495 (bytes)
	 0x000000000000000b (SYMENT)             24 (bytes)
	 0x0000000000000003 (PLTGOT)             0x3ba3dd2fe8
	 0x0000000000000002 (PLTRELSZ)           2664 (bytes)
	 0x0000000000000014 (PLTREL)             RELA
	 0x0000000000000017 (JMPREL)             0x3ba3a68b00
	 0x0000000000000007 (RELA)               0x3ba3a37228
	 0x0000000000000008 (RELASZ)             202968 (bytes)
	 0x0000000000000009 (RELAENT)            24 (bytes)
	 0x000000006ffffffc (VERDEF)             0x3ba3a37140
	 0x000000006ffffffd (VERDEFNUM)          4
	 0x000000006ffffffe (VERNEED)            0x3ba3a371a8
	 0x000000006fffffff (VERNEEDNUM)         2
	 0x000000006ffffff0 (VERSYM)             0x3ba3a35048
	 0x000000006ffffff9 (RELACOUNT)          8446
	 0x000000006ffffdf8 (CHECKSUM)           0x5b6c6280
	 0x000000006ffffdf5 (GNU_PRELINKED)      2015-10-16T19:35:45
	 0x0000000000000000 (NULL)               0x0
	 
	 # ldd vmdkbkp
	linux-vdso.so.1 =>  (0x00007fffebbff000)
	libvmdkbkp.so => /root/vmbkp-master/vmdkbkp/src/./libvmdkbkp.so (0x00007fe9f1298000)
	libvixDiskLib.so.5 => /usr/lib/vmware-vix-disklib/lib64/libvixDiskLib.so.5 (0x00007fe9f0eaa000)
	libboost_iostreams-mt.so.1.48.0 => /usr/lib64/libboost_iostreams-mt.so.1.48.0 (0x00007fe9f0c7e000)
	librt.so.1 => /lib64/librt.so.1 (0x0000003344c00000)
	libcrypto.so.10 => /usr/lib64/libcrypto.so.10 (0x0000003ba3a00000)
	libboost_thread-mt.so.1.48.0 => /usr/lib64/libboost_thread-mt.so.1.48.0 (0x00007fe9f0a63000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00000036eee00000)
	libstdc++.so.6 => /usr/lib64/libstdc++.so.6 (0x0000003bbde00000)
	libm.so.6 => /lib64/libm.so.6 (0x0000003bb2e00000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x0000003bbda00000)
	libc.so.6 => /lib64/libc.so.6 (0x0000003bb1e00000)
	libdl.so.2 => /lib64/libdl.so.2 (0x0000003bb2600000)
	libcrypt.so.1 => /lib64/libcrypt.so.1 (0x0000003bbd600000)
	libz.so.1 => /lib64/libz.so.1 (0x0000003bb3200000)
	/lib64/ld-linux-x86-64.so.2 (0x0000003bb1a00000)
	libbz2.so.1 => /lib64/libbz2.so.1 (0x0000003bc4a00000)
	libfreebl3.so => /lib64/libfreebl3.so (0x0000003bbd200000)


### 3. open the flag about [core dump](http://www.akadia.com/services/ora_enable_core.html)

	// show the core dump status
	ulimit -c
	// set it to unlimited
	ulimit -c unlimited

    
### 4. how to find the issue
1. check the call stack using gdb, find the segment falut in the libcrypto called by function belong to vmware

    ```
    (gdb) bt
    #0  0x0000003ba3ad64a4 in engine_unlocked_finish () from /usr/lib64/libcrypto.so.10
    #1  0x0000003ba3ad6591 in ENGINE_finish () from /usr/lib64/libcrypto.so.10
    #2  0x0000003ba3ae9f47 in EVP_DigestInit_ex () from /usr/lib64/libcrypto.so.10
    #3  0x00007fee120eb7b4 in ssl23_connect () from /usr/lib/vmware-vix-disklib/lib64/libssl.so.0.9.8
    #4  0x00007fee10d44be8 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #5  0x00007fee10d556e7 in Curl_ssl_connect_nonblocking () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #6  0x00007fee10d31c0e in ?? () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #7  0x00007fee10d3af66 in Curl_protocol_connect () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #8  0x00007fee10d50498 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #9  0x00007fee10d5094f in ?? () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #10 0x00007fee10d50a1f in curl_multi_socket_action () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #11 0x00007fee10f6c106 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libgvmomi.so.0
    #12 0x00007fee11e846bb in ?? () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLibVim.so
    #13 0x00007fee11e8470f in ?? () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLibVim.so
    #14 0x00007fee11e85122 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLibVim.so
    #15 0x00007fee11e80272 in VixDiskLibVim_GetNfcTicket () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLibVim.so
    #16 0x00007fee18bb651a in ?? () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLib.so.5
    #17 0x00007fee18bb6a89 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLib.so.5
    #18 0x00007fee1903620d in VmdkManager::open (this=0xa0f700) at vddk_manager.cpp:397
    #19 0x00007fee1903a723 in VddkWorker::open (this=0x9a2c50) at vddk_wrapper.cpp:198
    #20 0x00007fee1903a332 in VddkWorker::dispatch (this=0x9a2c50, cmd="open") at vddk_wrapper.cpp:166
    #21 0x00007fee19039bf6 in VddkWorker::run (this=0x9a2c50) at vddk_wrapper.cpp:132
    #22 0x00007fee19043402 in ForkManager::start (this=0x9a2c50) at fork_manager.hpp:137
    #23 0x00007fee1903e976 in VddkController::start (this=0x7fff94742b30) at vddk_wrapper.cpp:459
    #24 0x00007fee19087410 in Command::doDumpFork (this=0x7fff94742e90) at command.cpp:519
    #25 0x00007fee19084e00 in Command::run (this=0x7fff94742e90) at command.cpp:157
    #26 0x0000000000407721 in main (argc=12, argv=0x7fff94743088) at vmdkbkp.cpp:52
    ```
2. check the vddk sample code(it run correctly), and set break point on the function which will be called in both execute file

    ```
    (gdb) b EVP_DigestInit_ex
    Breakpoint 3 at 0x7ffff179e720
    ```

3. find the vddk sample code call the `EVP_DigestInit_ex` in libcrypto.so.0.9.8
    ```
    (gdb) bt
    #0  0x00007ffff179e720 in EVP_DigestInit_ex () from /usr/lib/vmware-vix-disklib/lib64/libcrypto.so.0.9.8
    #1  0x00007ffff179a417 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libcrypto.so.0.9.8
    #2  0x00007ffff179af0d in RAND_load_file () from /usr/lib/vmware-vix-disklib/lib64/libcrypto.so.0.9.8
    #3  0x00007ffff01f7a51 in Curl_ossl_seed () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #4  0x00007ffff01f7de6 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #5  0x00007ffff02086e7 in Curl_ssl_connect_nonblocking () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #6  0x00007ffff01e4c0e in ?? () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #7  0x00007ffff01edf66 in Curl_protocol_connect () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #8  0x00007ffff0203498 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #9  0x00007ffff020394f in ?? () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #10 0x00007ffff0203a1f in curl_multi_socket_action () from /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
    #11 0x00007ffff0431106 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libgvmomi.so.0
    #12 0x00007ffff13496bb in ?? () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLibVim.so
    #13 0x00007ffff134970f in ?? () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLibVim.so
    #14 0x00007ffff134a122 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLibVim.so
    #15 0x00007ffff1345272 in VixDiskLibVim_GetNfcTicket () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLibVim.so
    #16 0x00007ffff7c3451a in ?? () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLib.so.5
    #17 0x00007ffff7c34a89 in ?? () from /usr/lib/vmware-vix-disklib/lib64/libvixDiskLib.so.5
    #18 0x0000000000405935 in VixDisk::VixDisk (this=0x7fffffffdba0, connection=0x66ce80, path=0x7fffffffe132 "[datastore1 (2)] rainbow-centos2/rainbow-centos2_2.vmdk", flags=4) at vixDiskLibSample.cpp:610
    #19 0x000000000040346e in DoInfo () at vixDiskLibSample.cpp:1050
    #20 0x00000000004025b4 in main (argc=11, argv=0x7fffffffdd98) at vixDiskLibSample.cpp:777
    ```
4. note:
    - show the shared library info in gdb using `info sharedlirary`
    - the libcrypto.so is needed by vddk, it maybe loaded dynamic. so if you want to check the shared library info when segment falut happen, you should set a breakpoint after function `VixDiskLib_InitEx`, then run `info sharedlirary`
    ```
    (gdb) info sharedlibrary
From                To                  Syms Read   Shared Object Library
0x0000003bb1a00b00  0x0000003bb1a199cb  Yes (*)     /lib64/ld-linux-x86-64.so.2
0x00007ffff7c32d40  0x00007ffff7d4a8a8  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libvixDiskLib.so.5
0x0000003bbde563f0  0x0000003bbdec3396  Yes (*)     /usr/lib64/libstdc++.so.6
0x0000003bb2e03e70  0x0000003bb2e43fb8  Yes (*)     /lib64/libm.so.6
0x0000003bbda02910  0x0000003bbda12f78  Yes (*)     /lib64/libgcc_s.so.1
0x0000003bb1e1eaa0  0x0000003bb1f400cc  Yes (*)     /lib64/libc.so.6
0x00000036eee05760  0x00000036eee110c8  Yes (*)     /lib64/libpthread.so.0
0x0000003bb2600de0  0x0000003bb2601998  Yes (*)     /lib64/libdl.so.2
0x0000003bbd600c00  0x0000003bbd6059a8  Yes (*)     /lib64/libcrypt.so.1
0x0000003bb3202120  0x0000003bb320d3a8  Yes (*)     /lib64/libz.so.1
0x0000003bbd203ec0  0x0000003bbd2550e8  Yes (*)     /lib64/libfreebl3.so
0x00007ffff1b591f0  0x00007ffff1b61648  Yes (*)     /lib64/libnss_files.so.2
0x00007ffff173fcc0  0x00007ffff17ed0f8  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libcrypto.so.0.9.8
0x00007ffff159cbd0  0x00007ffff15c9c78  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libssl.so.0.9.8
0x00007ffff1340bb0  0x00007ffff136b7b8  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libvixDiskLibVim.so
0x00007ffff115b560  0x00007ffff11da748  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libglib-2.0.so.0
0x00007ffff10442d0  0x00007ffff10451d8  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libgmodule-2.0.so.0
0x00007ffff0f08e40  0x00007ffff0f31d48  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libgobject-2.0.so.0
0x00007ffff0dfd880  0x00007ffff0dfebf8  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libgthread-2.0.so.0
0x00007ffff0bd34c0  0x00007ffff0cb2e28  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libxml2.so.2
0x00007ffff04233c0  0x00007ffff0804d68  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libgvmomi.so.0
0x0000003344c02140  0x0000003344c054e8  Yes (*)     /lib64/librt.so.1
0x00007ffff01dfc70  0x00007ffff0212228  Yes (*)     /usr/lib/vmware-vix-disklib/lib64/libcurl.so.4
(*): Shared library is missing debugging information.

    ```
    
