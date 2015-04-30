# 同步互斥(lec 17) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 背景
 - 请给出程序正确性的定义或解释。
 - 在一个新运行环境中程序行为与原来的预期不一致，是错误吗？
 - 程序并发执行有什么好处和障碍？
 - 什么是原子操作？

### 现实生活中的同步问题

 - 家庭采购中的同步问题与操作系统中进程同步有什么区别？
 - 如何通过枚举和分类方法检查同步算法的正确性？
 - 尝试描述方案四的正确性。
 - 互斥、死锁和饥饿的定义是什么？

### 临界区和禁用硬件中断同步方法

 - 什么是临界区？
 - 临界区的访问规则是什么？
 - 禁用中断是如何实现对临界区的访问控制的？有什么优缺点？

### 基于软件的同步方法

 - 尝试通过枚举和分类方法检查Peterson算法的正确性。
 - 尝试准确描述Eisenberg同步算法，并通过枚举和分类方法检查其正确性。

### 高级抽象的同步方法

 - 如何证明TS指令和交换指令的等价性？
 - 为什么硬件原子操作指令能简化同步算法的实现？
 
## 小组思考题

1. （spoc）阅读[简化x86计算机模拟器的使用说明](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/lab7-spoc-exercise.md)，理解基于简化x86计算机的汇编代码。

2. （spoc)了解race condition. 进入[race-condition代码目录](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/race-condition)。

 - 执行 `./x86.py -p loop.s -t 1 -i 100 -R dx`， 请问`dx`的值是什么？
 - 执行 `./x86.py -p loop.s -t 2 -i 100 -a dx=3,dx=3 -R dx` ， 请问`dx`的值是什么？
 - 执行 `./x86.py -p loop.s -t 2 -i 3 -r -a dx=3,dx=3 -R dx`， 请问`dx`的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 1 -M 2000`, 请问变量x的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -a bx=3 -M 2000`, 请问变量x的值是什么？为何每个线程要循环3次？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0`， 请问变量x的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 1`， 请问变量x的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 2`， 请问变量x的值是什么？ 
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 1`， 请问变量x的值是什么？ 

3. （spoc） 了解software-based lock, hardware-based lock, [software-hardware-lock代码目录](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/software-hardware-locks)

  - 理解flag.s,peterson.s,test-and-set.s,ticket.s,test-and-test-and-set.s 请通过x86.py分析这些代码是否实现了锁机制？请给出你的实验过程和结论说明。能否设计新的硬件原子操作指令Compare-And-Swap,Fetch-And-Add？
  
  
*	flag.s没有实现锁机制，在时间片为1时两个线程交替进行。
	在执行以下代码时：
	
	```
	.acquire
	mov  flag, %ax      # get flag
	test $0, %ax        # if we get 0 back: l
  	```
  	两个进程都会得到flag为0，认为critical代码空闲，会先后进入critical执行，不能。

*	peterson.s能够实现锁机制。

	```
	.acquire
	mov $1, 0(%fx,%bx,4)    # flag[self] = 1
	mov %cx, turn           # turn       = 1 - self
	```
	以上代码会运行时，无论何时都会有一个进程作为最后一个运行```mov %cx, turn```，将turn赋值为某个唯一的值，而后进行以下判断：
	
	```	
	.spin1
	mov 0(%fx,%cx,4), %ax   # flag[1-self]申请使用
	test $1, %ax
	jne .fini               # if flag[1-self] != 1 进程空闲, skip past loop to .fini
	mov turn, %ax           # else 另一进程在使用
	test %cx, %ax           # compare 'turn' and '1 - self'
	je .spin1               # if turn==1-self, go back and start spin again
	```
	相当于测试①另一进程不需要使用②另一进程不能使用，二者成立一个就继续进行。而对于判断条件无论何时只可能有一个turn和储存的cx（另一进程号）不相等，即只有一个能通过第二次测试。故可以实现锁。

*	test-and-set.s可以实现锁机制。
	以下代码：
	
	```
	.acquire
	mov  $1, %ax        
	xchg %ax, mutex     # atomic swap of 1 and mutex
	test $0, %ax        # if we get 0 back: lock is free!
	jne  .acquire       # if not, try again
	```
	xchg命令能够保证原子操作，即无论何时只有一个能够将为0的mutex读入ax，即只有一个能够通过```jne  .acquire```的测试跳转，即只有一个进入critical区域。
	
*	ticket.s可以实现锁机制，使用fetchadd原子操作可以分配无论何时都唯一的ticket和turn，当turn和ticket相等时才能进入，而只有一个turn和一个ticket匹配，即critical区域只有一个进程能够进入。
  
*	test-and-test-and-set.s可以实现锁机制。类似于test-and-set.s，xchg命令能够保证原子操作，即无论何时只有一个能够将为0的mutex读入ax，且多一层判断减小等待切换的时间，效率能够提高。
	
```
Compare-And-Swap
int CompareAndSwap(int *ptr, int expected, int new) {
  int actual = *ptr;
  if (actual == expected)
    *ptr = new;
  return actual;
}
```

```
Fetch-And-Add
int FetchAndAdd(int *ptr) {
  int old = *ptr;
  *ptr = old + 1;
  return old;
}
```

