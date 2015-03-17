# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
+ 采分点：说明四种算法的优点和缺点
- 答案没有涉及如下3点；（0分）
- 正确描述了二种分配算法的优势和劣势（1分）
- 正确描述了四种分配算法的优势和劣势（2分）
- 除上述两点外，进一步描述了一种更有效的分配算法（3分）
```
-   最优匹配：
        优点：
            大部分分配尺寸合适，空间利用率高
            能够容纳后续的大块地址分配任务
        缺点：
            外部碎片大小最小化，产生较多完全无用的小碎片
            释放空间时查找邻近分区耗费时间长
    最差匹配：
        优点：
            不会产生太多小碎片
        缺点：
            大分区被消耗较快，空余大分区存在少，不能分配
            仍然有外部碎片存在
            释放空间时查找邻近分区时间长
    最先匹配：
        优点：
            算法简单
            在高地址空间有大块空闲分区
        缺点：
            有外部碎片
            由于低地址分配较频繁，大块地址会被经常切分，分配大块时需要查找
    buddy system：
        优点：
            不会有外部碎片
            释放时
        缺点：
            空间有一部分被浪费
            实现时较为复杂，维护数据较多

>  

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)
//
//  main.cpp
//  memo_manager
//
//  Created by mxbttt2 on 15/3/16.
//  Copyright (c) 2015年 mxb. All rights reserved.
//

#include <iostream>


struct MemBlock
{
int size;
int head;
bool isUse;
MemBlock *next;
MemBlock(int _size)
{
size = _size;
next = NULL;
head = 1;
isUse = false;
}
MemBlock(){}
};

MemBlock *memHead ;

int Malloc(unsigned int num_bytes)
{
MemBlock *now = memHead;
int memo_return;
while (now && now->next) {
if (now->size >= num_bytes && !now->isUse) {
break;
}
now = now->next;
}//找到第一个足够大的块

if (!now->next && (now->size < num_bytes || now->isUse)) {
return -1;
}//未找到足够大的块

if (now->size == num_bytes) {
now->isUse = true;
}//使一个块消失
else
{
MemBlock *ne = (MemBlock *)malloc(sizeof(MemBlock));
ne->next =  now->next;
ne->size = now->size - num_bytes;
ne->head = now->head + num_bytes;
ne->isUse = false;

memo_return = now->head;
now->isUse = true;
now->size = num_bytes;
now->next = ne;
}//会生成碎片

return memo_return;
}

void Free(int memo_head)
{
if (memo_head < 0) {
return;
}
MemBlock *now = memHead, *last = NULL;
while (now && now->head != memo_head) {
last = now;
now = now->next;
}

if (!now) {
return;
}
now->isUse = false;
if (last && !last->isUse) {
last->size += now->size;
last->next = now->next;
free(now);
now = last;
}
if (now->next && !now->next->isUse) {
now->size += now->next->size;
now->next = now->next->next;
}

}

void print()
{
MemBlock *now = memHead;
int n = 1;
printf("----------------\n");
while (now) {
printf("BLOCK.%d\tHead-%d\tSize-%d\tisUse-%d\n",n++,now->head,now->size,now->isUse);
now = now->next;
}
printf("----------------\n");
}

int main(int argc, const char * argv[]) {
memHead = (MemBlock *)malloc(sizeof(MemBlock));
memHead->size = 1000;
memHead->isUse = false;
memHead->next = 0;


int newBolck1 = Malloc(100);
int newBolck2 = Malloc(100);
int newBolck3 = Malloc(100);
Free(newBolck2);
Free(newBolck1);
print();
int newBolck4 = Malloc(10);
int newBolck5 = Malloc(30);
print();
int newBolck6 = Malloc(200);
print();
int newBolck7 = Malloc(30);
Free(newBolck3);
print();
return 0;
}

--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存的结构和工作机理？

- [x]  

>  

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



