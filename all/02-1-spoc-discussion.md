#lec 3 SPOC Discussion

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
### 1. 比较UEFI和BIOS的区别。
 - 区别：UEFI有图形化界面，多样的操作方式，允许植入硬件驱动等等，更加易用、更加多功能、更加方便；
 - 通过保护预启动或预引导进程，抵御bootkit攻击，从而提高安全性；
 - 缩短了启动时间和从休眠状态恢复的时间；
 - 支持容量超过2.2 TB的驱动器；
 - 支持64位的现代固件设备驱动程序，系统在启动过程中可以使用它们来对超过172亿GB的内存进行寻址；
 - UEFI已具备文件系统的支持，它能够直接读取FAT分区中的文件；
 - 可开发出直接在UEFI下运行的应用程序，这类程序文件通常以efi结尾；
 - 这些都是BIOS做不到的。因为BIOS下启动操作系统之前，必须从硬盘上指定扇区读取系统启动代码（包含在主引导记录中），然后从活动分区中引导启动操作系统。
>   

### 2. 描述PXE的大致启动流程。
 - 大致流程：
 -  客户端个人电脑开机后， 在 TCP/IP Bootrom 获得控制权之前先做自我测试。
 -  Bootprom 送出 BOOTP/DHCP 要求以取得 IP。
 -  如果服务器收到个人电脑所送出的要求， 就会送回 BOOTP/DHCP 回应，内容包括客户端的 IP 地址， 预设网关， 及开机映像文件。否则，服务器会忽略这个要求。
 -  Bootprom 由 TFTP 通讯协议从服务器下载开机映像文件。
 -  个人电脑通过这个开机映像文件开机， 这个开机文件可以只是单纯的开机程式也可
以是操作系统。
 -  开机映像文件将包含 kernel loader 及压缩过的 kernel，此 kernel 将支持NTFS root
系统。
 -  远程客户端根据下载的文件启动机器。

## 3.2 系统启动流程
### 1. 了解NTLDR的启动流程。
NTLDR文件的是一个隐藏的，只读的系统文件，位置在系统盘的根目录，用来装载操作系统。
一般情况系统的引导过程是这样的代码
1、电源自检程序开始运行
2、主引导记录被装入内存，并且程序开始执行
3、活动分区的引导扇区被装入内存
4、NTLDR从引导扇区被装入并初始化
5、将处理器的实模式改为32位平滑内存模式
6、NTLDR开始运行适当的小文件系统驱动程序。
小文件系统驱动程序是建立在NTLDR内部的，它能读FAT或NTFS。
7、NTLDR读boot.ini文件
8、NTLDR装载所选操作系统
如果windows NT/windows 2000/windows XP/windows server 2003这些操作系统被选择，NTLDR运行Ntdetect。
对于其他的操作系统，NTLDR装载并运行Bootsect.dos然后向它传递控制。
windows NT过程结束。
9.Ntdetect搜索计算机硬件并将列表传送给NTLDR，以便将这些信息写进\\HKE Y_LOCAL_MACHINE\HARDWARE中。
10.然后NTLDR装载Ntoskrnl.exe，Hal.dll和系统信息集合。
11.Ntldr搜索系统信息集合，并装载设备驱动配置以便设备在启动时开始工作
12.Ntldr把控制权交给Ntoskrnl.exe，这时,启动程序结束,装载阶段开始
### 2. 了解GRUB的启动流程。
装载GRUB和操作系统的过程，包括以下几个操作步骤：

装载记录

基本引导装载程序所做的唯一的事情就是装载第二引导装载程序。

装载Grub

这第二引导装载程序实际上是引出更高级的功能，以允许用户装载一个特定的操作系统。

装载系统

如linux内核。GRUB把机器的控制权移交给操作系统。
不同的是，微软操作系统都是使用一种称为链式装载的引导方法来启动的，主引导记录仅仅是简单地指向操作系统所在分区的第一个扇区。
### 3. 比较NTLDR和GRUB的功能有差异。
 
 
 
### 4. 了解u-boot的功能。
- U-Boot，全称 Universal Boot Loader，是遵循GPL条款的开放源码项目。
- U-Boot不仅仅支持嵌入式Linux系统的引导，它还支持NetBSD, VxWorks, QNX, RTEMS, ARTOS, LynxOS, android嵌入式操作系统。其目前要支持的目标操作系统是OpenBSD, NetBSD, FreeBSD,4.4BSD, Linux, SVR4, Esix, Solaris, Irix, SCO, Dell, NCR, VxWorks, LynxOS, pSOS, QNX, RTEMS, ARTOS, android。这是U-Boot中Universal的一层含义，另外一层含义则是U-Boot除了支持PowerPC系列的处理器外，还能支持MIPS、 x86、ARM、NIOS、XScale等诸多常用系列的处理器。这两个特点正是U-Boot项目的开发目标，即支持尽可能多的嵌入式处理器和嵌入式操作系统。就目前来看，U-Boot对PowerPC系列处理器支持最为丰富，对Linux的支持最完善。

## 3.3 中断、异常和系统调用比较
 1. 举例说明Linux中有哪些中断，哪些异常？
 1. Linux的系统调用有哪些？大致的功能分类有哪些？  (w2l1)

 Linux系统调用很多地方继承了Unix的系统调用，但Linux相比传统Unix的系统调用做了很多扬弃，它省去了许多Unix系统冗余的系统调用，仅仅保留了最基本和最有用的系统调用，所以Linux全部系统调用只有250个左右.
 大致分类有进程控制/文件系统控制/系统控制/内存管理/网络管理/socket控制/用户管理/进程间通信 等等。
 其中文件系统控制包括读写和系统操作（例如改变文件的可存取性/工作目录/权限），进程间通信又分为信号/消息/管道/信号量/共享内存。
```
  + 采分点：说明了Linux的大致数量（上百个），说明了Linux系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
 1. 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？(w2l1)
 参考文件：https://github.com/chyyuu/ucore_lab/blob/master/labcodes_answer/lab8_result/kern/syscall/syscall.c
 读上述文件（ucore lab8的answer下的syscall目录）可知，ucore共有22个系统调用，分为系统管理/进程管理/文件系统管理等。
 sys_exit/sys_wait等明显是系统控制部分，sys_kill则是进程控制部分，后面的read/write则是明显的文件操作了。
 
 ```
  + 采分点：说明了ucore的大致数量（二十几个），说明了ucore系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
## 3.4 linux系统调用分析
 1. 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(w2l1)
 
 objdump加-d参数可以用于反汇编可执行文件：如下是部分反汇编结果：
```
Disassembly of section .init:

00000000004003a8 <_init>:
  4003a8:	48 83 ec 08          	sub    $0x8,%rsp
  4003ac:	48 8b 05 45 0c 20 00 	mov    0x200c45(%rip),%rax        # 600ff8 <_DYNAMIC+0x1d0>
  4003b3:	48 85 c0             	test   %rax,%rax
  4003b6:	74 05                	je     4003bd <_init+0x15>
  4003b8:	e8 33 00 00 00       	callq  4003f0 <__gmon_start__@plt>
  4003bd:	48 83 c4 08          	add    $0x8,%rsp
  4003c1:	c3                   	retq   

Disassembly of section .plt:

00000000004003d0 <__libc_start_main@plt-0x10>:
  4003d0:	ff 35 32 0c 20 00    	pushq  0x200c32(%rip)        # 601008 <_GLOBAL_OFFSET_TABLE_+0x8>
  4003d6:	ff 25 34 0c 20 00    	jmpq   *0x200c34(%rip)        # 601010 <_GLOBAL_OFFSET_TABLE_+0x10>
  4003dc:	0f 1f 40 00          	nopl   0x0(%rax)
 ```
 nm命令用来列出目标文件的符号清单：（一部分结果）
```
0000000000000002 a AF_INET
000000000060105c B __bss_start
000000000060105c b completed.6972
0000000000601028 D __data_start
0000000000601028 W data_start
0000000000400430 t deregister_tm_clones
00000000004004a0 t __do_global_dtors_aux
0000000000600e18 t __do_global_dtors_aux_fini_array_entry
0000000000601030 D __dso_handle
0000000000600e28 d _DYNAMIC
000000000060105c D _edata
0000000000601060 B _end
0000000000400564 T _fini
00000000004004c0 t frame_dummy
0000000000600e10 t __frame_dummy_init_array_entry
0000000000400670 r __FRAME_END__
0000000000601000 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000601038 d hello
00000000004003a8 T _init
0000000000600e18 t __init_array_end
0000000000600e10 t __init_array_start
0000000000400570 R _IO_stdin_used
0000000000000006 a IPPROTO_TCP
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000600e20 d __JCR_END__
……
```
 
 file命令用于确定文件类型：（一部分输出）
 ```
lab1-ex0.exe: ELF 64-bit LSB  executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=337b4a665f50f5ae7931487124df3038c7318f17, not stripped
```
 系统调用是应用程序同系统之间的接口，由操作系统提供。
 操作系统的主要功能是为管理硬件资源和为应用程序开发人员提供良好的环境来使应用程序具有更好的兼容性，为了达到这个目的，内核提供一系列具备预定功能的多内核函数，通过一组称为系统调用（system call)的接口呈现给用户。 系统调用把应用程序的请求传给内核，调用相应的的内核函数完成所需的处理，将处理结果返回给应用程序。
 
 对应在反汇编的代码中，可以看见4003b8:	e8 33 00 00 00       	callq  4003f0 <__gmon_start__@plt>一行，就是系统调用的一个实例。

 ```
  + 采分点：说明了objdump，nm，file的大致用途，说明了系统调用的具体含义
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 
 ```
 
 1. 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(w2l1)
 

 ```
  + 采分点：说明了strace的大致用途，说明了系统调用的具体执行过程（包括应用，CPU硬件，操作系统的执行过程）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
## 3.5 ucore系统调用分析
 1. ucore的系统调用中参数传递代码分析。
 1. ucore的系统调用中返回结果的传递代码分析。
 1. 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
 1. 以ucore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。
 
## 3.6 请分析函数调用和系统调用的区别
 1. 请从代码编写和执行过程来说明。
   1. 说明`int`、`iret`、`call`和`ret`的指令准确功能
 
