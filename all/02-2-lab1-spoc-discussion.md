# lab1 SPOC思考题

## 个人思考题

NOTICE
- 有"w2l2"标记的题是助教要提交到学堂在线上的。
- 有"w2l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。
---

请描述ucore OS配置和驱动外设时钟的准备工作包括哪些步骤？ (w2l2)
对IDT的初始化，时钟中断；
初始化8259中断控制器
初始化8253时钟外设,计数器置零，控制外设

>  

lab1中完成了对哪些外设的访问？ (w2l2)
	时钟、串口、并口、键盘和CGA
>  

lab1中的cprintf函数最终通过哪些外设完成了对字符串的输出？ (w2l2)
cprintf通过多层函数调用，最终看到如下函数
/* cons_putc - print a single character @c to console devices */
void
cons_putc(int c) {
    lpt_putc(c);
    cga_putc(c);
    serial_putc(c);
}
说明输出至并口、串口和cga屏幕，调用了这三种外设  

---

## 小组思考题

---

lab1中printfmt函数用到了可变参，请参考写一个小的linux应用程序，完成实现定义和调用一个可变参数的函数。(spoc)

 double sum_series(int num, ...) 
{   
	double sum= 0.0, t;   
	va_list argptr;   
	va_start(argptr, num);   
	for(; num; num--)   
	{   
		t= va_arg(argptr, double);   
		sum= sum+ t; 
	}   
	va_end(argptr);   
	return sum;   
}


如果让你来一个阶段一个阶段地从零开始完整实现lab1（不是现在的填空考方式），你的实现步骤是什么？（比如先实现一个可显示字符串的bootloader（描述一下要实现的关键步骤和需要注意的事项），再实现一个可加载ELF格式文件的bootloader（再描述一下进一步要实现的关键步骤和需要注意的事项）...） (spoc)
- 首先实现一个可以显示字符串的bootloader，在实现一个可加载ELF格式文件的bootloader，实现ucore操作系统的内核，再跳转到elf的入口。

> 


如何能获取一个系统调用的调用次数信息？如何可以获取所有系统调用的调用次数信息？请简要说明可能的思路。(spoc)
- [x]  在系统内核系统调用表中添加一个整形数，每次启动之后清零，每次系统调用必须调用的函数段中使这个数加一，就可以统计了。
	若要获取某个系统调用的信息，可以只在该调用的函数的代码中添加信息。

> 

如何修改lab1, 实现一个可显示字符串"THU LAB1"且依然能够正确加载ucore OS的bootloader？如果不能完成实现，请说明理由。
- 在init.c文件中有下列的代码，可以修改：
kern_init(void) {
    extern char edata[], end[];
    memset(edata, 0, end - edata);

    cons_init();                // init the console

    const char *message = "(THU.CST) os is loading ...";
    cprintf("%s\n\n", message);

> 

对于ucore_lab中的labcodes/lab1，我们知道如果在qemu中执行，可能会出现各种稀奇古怪的问题，比如reboot，死机，黑屏等等。请通过qemu的分析功能来动态分析并回答lab1是如何执行并最终为什么会出现这种情况？
- 直接使用make qemu执行，会出现无限重启的情况，但是单步调试时发现卡死在while(1)处，可能是在等待中断，不能继续。

> 

对于ucore_lab中的labcodes/lab1,如果出现了reboot，死机，黑屏等现象，请思考设计有效的调试方法来分析常在现背后的原因。
- 
在tools/gdbinit中增加命令，define hook-stop，x/2i $pc，end显示当前执行的两条汇编命令，并在其中添加break *(0x7c00)使模拟器在主入口地址7c00处停止。使用stepi命令查看单条汇编代码，对照即可调式。

> 

---

## 开放思考题

---

如何修改lab1, 实现在出现除零错误异常时显示一个字符串的异常服务例程的lab1？
- [x]  

> 


在lab1/bin目录下，通过`objcopy -O binary kernel kernel.bin`可以把elf格式的ucore kernel转变成体积更小巧的binary格式的ucore kernel。为此，需要如何修改lab1的bootloader, 能够实现正确加载binary格式的ucore OS？ (hard)
- [x]  

>

GRUB是一个通用的bootloader，被用于加载多种操作系统。如果放弃lab1的bootloader，采用GRUB来加载ucore OS，请问需要如何修改lab1, 能够实现此需求？ (hard)
- [x]  

>


如果没有中断，操作系统设计会有哪些问题或困难？在这种情况下，能否完成对外设驱动和对进程的切换等操作系统核心功能？
- [x]  

>  

---

## 各知识点的小练习

### 4.1 启动顺序
---
读入ucore内核的代码？

- 

> 

跳转到ucore内核的代码？

- [x]  

> 

全局描述符表的初始化代码？

- [x]  

> 

GDT内容的设置格式？初始映射的基址和长度？特权级的设置位置？

- [x]  

> 

可执行文件格式elf的各个段的数据结构？

- [x]  

> 

如果ucore内核的elf是否要求连续存放？为什么？

- [x]  

> 
---

### 4.2 C函数调用的实现
---

函数调用的stackframe结构？函数调用的参数传递方法有哪几种？
- [x]  

> 

系统调用的stackframe结构？系统调用的参数传递方法有哪几种？

- [x]  

> 
---

### 4.3 GCC内联汇编
---

使用内联汇编的原因？

- [x]  

> 特权指令、性能优化

对ucore中的一段内联汇编进行完整的解释？

- [x]  

> 
---

### 4.4 x86中断处理过程
---

4.4 x86中断处理过程

中断描述符表IDT的结构？

- [x]  

> 

中断描述表到中断服务例程的地址计算过程？

- [x]  

> 

中断处理中硬件压栈内容？用户态中断和内核态中断的硬件压栈有什么不同？

- [x]  

> 

中断处理中硬件保存了哪些寄存器？

- [x]  

> 
---

### 4.5 练习一 ucore编译过程
---

gcc编译、ld链接和dd生成两个映像对应的makefile脚本行？

- [x]  

> 
---

### 4.6 练习二 qemu和gdb的使用
---

qemu的命令行参数含义解释？

- [x]  

> 

gdb命令格式？反汇编、运行、断点设置

- [x]  

> 
---

### 练习三 加载程序
---

A20的使能代码分析？

- [x]  

> 

生成主引导扇区的过程分析？

- [x]  

> 

保护模式的切换代码？

- [x]  

> 

如何识别elf格式？对应代码分析？

- [x]  

> 

跳转到elf的代码？

- [x]  

> 
函数调用栈获取？

- [x]  

> 
---

### 4.8 练习四和五 ucore内核映像加载和函数调用栈分析
---

如何识别elf格式？对应代码分析？

- [x]  

> 

跳转到elf的代码？

- [x]  

> 

函数调用栈获取？
- [x]  

> 
---


### 4.9 练习六 完善中断初始化和处理
---

各种设备的中断初始化？
- [x]  

> 
中断描述符表IDT的排列顺序？
- [x]  

> 中断号
CPU加电初始化后中断是使能的吗？为什么？

- [x]  

> 
中断服务例程的入口地址在什么地方设置的？

- [x]  

> 
alltrap的中断号是在哪写入到trapframe结构中的？

- [x]  

> 
trapframe结构？
- [x]  

> 
---
