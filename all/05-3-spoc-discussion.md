# lab4 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 总体介绍

(1) ucore的线程控制块数据结构是什么？

### 关键数据结构

(2) 如何知道ucore的两个线程同在一个进程？

(3) context和trapframe分别在什么时候用到？
	context在切换进程时保存寄存器，在拷贝进程时
(4) 用户态或内核态下的中断处理有什么区别？在trapframe中有什么体现？

### 执行流程

(5) do_fork中的内核线程执行的第一条指令是什么？它是如何过渡到内核线程对应的函数的？
```
tf.tf_eip = (uint32_t) kernel_thread_entry;
/kern-ucore/arch/i386/init/entry.S
/kern/process/entry.S
```

(6)内核线程的堆栈初始化在哪？
```
tf和context中的esp
```

(7)fork()父子进程的返回值是不同的。这在源代码中的体现中哪？

(8)内核线程initproc的第一次执行流程是什么样的？能跟踪出来吗？

## 小组练习与思考题

(1)(spoc) 理解内核线程的生命周期。

> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

### 掌握知识点
1. 内核线程的启动、运行、就绪、等待、退出
2. 内核线程的管理与简单调度
3. 内核线程的切换过程

### 练习用的[lab4 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab4/lab4-spoc-discuss)


请完成如下练习，完成代码填写，并形成spoc练习报告

进程的创建对应了proc_init函数，在proc_init函数当中调用alloc_proc函数，首先在alloc当中进行一次分配，如果分配成功就对于pid，state等赋值为-1或者是不能建立，让os比较容易发现这个进程创建失败还是成功。如果alloc成功就在proc_init当中进行对于变量的初始化，state表示这个进程所处的状态，把state赋值为runnable，意思为可运行（可能是正在运行的）。将pid赋值为0，是整个os的第一个进程，在这第一个进程当中创造两个线程。cr3是页表起始页地址，在alloc_proc当中有boot_cr3进行赋值，context在涉及proc_run以及proc之间的switch的时候才会涉及，trapframe是中断的保存的堆栈，当一旦中断出现的时候就把需要保存的现场保存到这个trapframe当中去。
练习2：分析并描述新创建的内核线程是如何分配资源的

注意 理解对kstack, trapframe, context等的初始化

kstack在第一个进程建立的时候使用bootstack进行初始化，而当使用do_fork函数进行其他进程创建的时候在setup_kstack使用page相关的page2kva函数进行初始化，通过返回值看初始化是否成功；在alloc函数当中，初始的进程的trapframe为空，之后在copythread当中对于eax，esp，eflags进行了赋值eax为0，esp赋值为当前esp，一旦出现了中断就把现场保存进去；context在alloc的时候memset为0，也是在copy_thread当中对于context当中的eip和esp进行赋值。
当前进程中唯一，操作系统的整个生命周期不唯一，在get_pid中会循环使用pid，耗尽会等待

### 练习3：阅读代码，在现有基础上再增加一个内核线程，并通过增加cprintf函数到ucore代码中
[答案github链接](https://github.com/mxbhsa/ucore_lab/tree/master/related_info/lab4/lab4-spoc-discuss)
增加了新的线程，使用cprintf输出了新建线程的信息
在切换时能够显示切换的from和to进程
进入schedule函数时输出sssssssssssssss...
组员 战裕隆 丁延卓 杨俊 马晓彬

### 练习4 （非必须，有空就做）：增加可以睡眠的内核线程，睡眠的条件和唤醒的条件可自行设计，并给出测试用例，并在spoc练习报告中给出设计实现说明

### 扩展练习1: 进一步裁剪本练习中的代码，比如去掉页表的管理，只保留段机制，中断，内核线程切换，print功能。看看代码规模会小到什么程度。


