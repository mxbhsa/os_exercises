#lec9 虚存置换算法spoc练习

## 个人思考题
1. 置换算法的功能？

2. 全局和局部置换算法的不同？

3. 最优算法、先进先出算法和LRU算法的思路？

4. 时钟置换算法的思路？

5. LFU算法的思路？

6. 什么是Belady现象？

7. 几种局部置换算法的相关性：什么地方是相似的？什么地方是不同的？为什么有这种相似或不同？

8. 什么是工作集？

9. 什么是常驻集？

10. 工作集算法的思路？

11. 缺页率算法的思路？

12. 什么是虚拟内存管理的抖动现象？

13. 操作系统负载控制的最佳状态是什么状态？

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象

可能出现以下三种情况:
1.  B(t)属于S(t),且B(t)属于S’(t),则经过这一步，需要将S(t)与S’(t)中的B(t)页都置于栈顶部，因为S1的栈大小大于S(t),所以对于B(t)元素，还是满足B(t)在S(t)中的位置，小于B(t)在S’(t)中的位置。对于原有在S(t)与S’(t)中的元素，在B(t)元素前的元素位置不变，在B(t)后的元素位置整体前移，大小关系保持不变。所以这种情况下依旧满足条件。 
2.  B(t)不属于S(t),且B(t)属于S’(t)。B(t)不属于S(t),就在栈顶压入元素，B(t)位置为n；在S‘(t)中找到B(t)，将B(t)元素移至栈顶，依旧满足S(t)中B(t)位置小于等于S‘(t)中位置。对于原有在S(t)中的元素，整体前移了一位，S’(t)中元素B(t)前的不变，B(t)后的整体前移1位，所以整体大小关系依旧满足。 
3.  B(t)不属于S(t),且B(t)不属于S‘(t),则都在栈尾部加入B(t)，S(t)中位置为n，S’(t)中位置为n+k。同时对于被弹出栈的元素，如果弹出元素相同，则依旧满足。如果弹出元素不同，因为S(t)中的对应元素位置小于等于S‘(t)的，所以S’(t)中弹出的元素必然已经不属于S(t)了。所以弹出后依旧满足S(t)属于S‘(t).即在这种情况下依旧满足假设。 由于假设的存在，S(t)属于S’(t),即不会出现B(t)属于S(t),B(t)不属于S‘(t)的情况。 综上所述，由数学归纳法得，对任意时刻t，S(t)属于S’(t),且任意S(t)中元素a，对应到S’(t)中元素a1，满足a的位置小于等于a1的栈位置。即对任意时刻，对S’(t)的缺页数量不会大于S(t)。即物理页数量增加，缺页率不会上升。即LRU算法不会出现belady现象。


(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)

```

#define SPACE 10
int work[SPACE] = {0};
int memo[SPACE] = {0};

void printMemo()
{
    printf("memo: ");
    for (int i = 0; i < SPACE; i++) {
        printf("%d ",memo[i]);
    }
    printf("\n");
}
void printWork()
{
    printf("work: ");
    for (int i = 0; i < SPACE; i++) {
        printf("%d ",work[i]);
    }
    printf("\n");
}

int newPage(unsigned nPage)
{
    //更新工作集
    for (int i = 0; i < SPACE-1; i++) {
        work[i] = work[i+1];
    }
    work[SPACE -1] = nPage;
    
    //若已在内存中不用替换
    for (int i = 0; i < SPACE; i++)
        if (memo[i] == nPage) {
            return i;//返回内存地址
        }
    
    //查找替换
    int out = 0;
    int match = 0;
    for (int i = 0; i < SPACE; i++) {//查找第一个不在工作集中的内存页
        match = 0;
        for (int j = 0; j < SPACE; j++) {
            if (memo[i] == work[j]) {
                match = 1;
            }
        }
        if (! match) {
            memo[i] = nPage;
            return out;//返回内存地址
        }
    }
    return -1;
}

int main(int argc, const char * argv[]) {
    for (int i = 0; i < 10; i++) {//初始化内存中没有任何页 即页号都为负数
        memo[i] = -i-1;
    }
    for (int i = 0; i < 10; i++) {//顺序访问0-9号页
        newPage(i);
    }
    printMemo();
    printf("visit 10 11 --------------------\n");
    newPage(10);//访问10号页
    newPage(11);
    printMemo();//查看内存中页号
    printf("visit 2 --------------------\n");
    newPage(2);
    printWork();//查看工作集
    printMemo();
    printf("visit 2 2 --------------------\n");
    newPage(2);
    newPage(2);
    printWork();
    printMemo();
    printf("visit 1 --------------------\n");
    newPage(1);
    printWork();
    printMemo();
    
}
```

## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)

