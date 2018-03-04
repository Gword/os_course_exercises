# lab0 SPOC思考题

\##**提前准备** （请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中是如何具体体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

------

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

  - **硬件支持：**

    进程需要：时钟中断；

    虚存需要：内存、外存和MMU；

    文件系统需要：磁盘读写。

  - **特权指令：**

    进程需要：用户态和内核态的转换；

    虚存需要：存储器空间的分配和回收、特殊寄存器的修改、页面置换、页面锁定；

    文件系统需要：读写硬盘。

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

  - **实模式：**可访问物理地址空间为1M。对操作系统和用户程序没有区别对待，指针都指向实际物理空间，没有提供对虚拟存储的支持。

    **保护模式：**可访问物理地址空间为4G。采用分页或分段存储管理机制，提供了存储共享和硬件保护，提供了虚拟存储的硬件支持。

  - **物理地址：**CPU外部地址总线上的寻址物理内存的地址信号。

  - **线性地址：**是逻辑地址到物理地址变换的中间层，分段机制下段偏移加上基址就是线性地址。若有分页机制则再变换会产生物理地址，没有则线性地址就是物理地址。

  - **逻辑地址：**分段机制下段偏移的部分，经过分段转换编程线性地址。


- 理解list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）


- 对于如下的代码段，请说明":"后面的数字是什么含义

```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };

```

这段代码是中断描述符表的位域结构，其总长度为64位，":"后的数字表示每个成员变量的位域长度。有些信息在存储时，并不需要占用一个完整的字节， 而只需占几个或一个二进制位，位域长度即表示该成员变量所占的二进制位数。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}

```

如果在其他代码段中有如下语句，

```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);

```

请问执行上述指令后， intr的值是多少？

gate.gd_off\_15\_0=0000 0000 0000 0011

gate.gd_ss=0000 0000 0000 0010

gate.gd_args=0000 0

gate.gd_rsv1=000

gate.gd_type=STS_TG32=1111

gate.gd_s=0

gate.gd_dpl=00

gate.gd_p=1

gate.gd_off\_31\_16=0000 0000 0000 0000

由于inter是小端系统故gate=0x00008F0000020003，intr只有32位，故intr=0x00020003。

### 课堂实践练习

#### 练习一

请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

- [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)

- ##### [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)

- ##### [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)

- ##### [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)

- ##### [[IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)]

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore中宏定义的用途，并举例描述其含义。

> 利用宏进行复杂数据结构中的数据访问； 利用宏进行数据类型转换；如 to_struct, 常用功能的代码片段优化；如 ROUNDDOWN, SetPageDirty