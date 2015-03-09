# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- 是 在暑假小学期中学过

>  http://www.imada.sdu.dk/Courses/DM18/Litteratur/IntelnATT.htm

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
- 页表功能对硬盘的控制方法
  启动时boot磁盘过程
  进程切换的需求

>   


哪些困难（请分优先级）会阻碍你自主完成lab实验？
- [对已有的代码结构不够了解
  对操作系统的功能实现方式不了解
  对C语言调用硬件的工作原理不了解

>   

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- 使用反汇编工具将物理地址中的代码转变成汇编代码再使用汇编代码确定

>   

了解函数调用栈对lab实验有何帮助？
- 
  知道函数在内存中的存在方式，CPU执行每条汇编代码的过程

>   

你希望从lab中学到什么知识？
- 了解操作系统内核的运行方式，各种系统调用的过程
  系统空闲时需要进行的维护工作
  了解基于硬件的C语言编程

>   

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- VirtualBox虚拟机无法打开给定的镜像文件，在Ubuntu系统图标会无限循环
  使用其他虚拟机软件运行的Ubuntu系统，通过apt-get update更新源服务器路径之后，使用apt-get install命令安装了所有需要的开发软件
  之后可以编辑、调试源代码

> 

熟悉基本的git命令行操作命令，从github上
的 http://www.github.com/chyyuu/ucore_lab 下载
ucore lab实验
- 打开该网址后 得到https://github.com/chyyuu/ucore_lab.git
  使用git pull https://github.com/chyyuu/ucore_lab.git命令将源码下载
  

> 

尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- 已完成。

> 

对于如下的代码段，请说明”：“后面的数字是什么含义
```
/* Gate descriptors for interrupts and traps */
struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment 		10B
    unsigned gd_ss : 16;            // segment selector					1B
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates		0
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)		0
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})			STS_IG32
    unsigned gd_s : 1;                // must be 0 (system)				0
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level		11B
    unsigned gd_p : 1;                // Present					1B
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment		0
};
```

- 是将一个字节中的二进位划分为几个不同的区域，其中每个区域的位数，以比特表示

> 

对于如下的代码段，
```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \ 	2
    (gate).gd_ss = (sel);                                \ 	1
    (gate).gd_args = 0;                                    \ 	0
    (gate).gd_rsv1 = 0;                                    \ 	0
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \	STS_IG32
    (gate).gd_s = 0;                                    \	0
    (gate).gd_dpl = (dpl);                                \	11B
    (gate).gd_p = 1;                                    \	1B
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \	0
}
```

如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 0,1,2,3);
```
请问执行上述指令后， intr的值是多少？

- intr共64位，8字节，其值为0x0002 0001 0017 0000

> 

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- 
static inline void list_init(list_entry_t *elm) __attribute__((always_inline));						//将一个节点next和prev初始化
static inline void list_add(list_entry_t *listelm, list_entry_t *elm) __attribute__((always_inline));			//在listelm之后加入节点elm
static inline void list_add_before(list_entry_t *listelm, list_entry_t *elm) __attribute__((always_inline));		//在listelm之前加入节点elm
static inline void list_add_after(list_entry_t *listelm, list_entry_t *elm) __attribute__((always_inline));		//在listelm之后加入节点elm
static inline void list_del(list_entry_t *listelm) __attribute__((always_inline));					//删除节点listelm
static inline void list_del_init(list_entry_t *listelm) __attribute__((always_inline));					//删除并初始化节点listelm
static inline bool list_empty(list_entry_t *list) __attribute__((always_inline));					//测试节点list是否是初始化的
static inline list_entry_t *list_next(list_entry_t *listelm) __attribute__((always_inline));				//返回listelm之后的节点
static inline list_entry_t *list_prev(list_entry_t *listelm) __attribute__((always_inline));				//返回listelm之前的节点

static inline void __list_add(list_entry_t *elm, list_entry_t *prev, list_entry_t *next) __attribute__((always_inline));//在prev和next两个节点之间加入节点elm
static inline void __list_del(list_entry_t *prev, list_entry_t *next) __attribute__((always_inline));  			//直接连接两个节点

list_entry_t head;
list_init(head);
if(! list_empty(head))
	exit(0);
list_entry_t *new_elm = (list_entry_t*)malloc(sizeof(list_entry_t));
list_add(head, new_elm);
list_del(new_elm);
> 

---

## 开放思考题

---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- [x]  

>  

---
