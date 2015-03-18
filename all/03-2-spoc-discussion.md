# lec6 SPOC思考题


NOTICE
- 有"w3l2"标记的题是助教要提交到学堂在线上的。
- 有"w3l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的
```
+ 采分点：说明64bit CPU架构的分页机制的大致特点和页表执行过程
- 答案没有涉及如下3点；（0分）
- 正确描述了64bit CPU支持的物理内存大小限制（1分）
- 正确描述了64bit CPU下的多级页表的级数和多级页表的结构或反置页表的结构（2分）
- 除上述两点外，进一步描述了在多级页表或反置页表下的虚拟地址-->物理地址的映射过程（3分）
```
-   64位CPU由于虚拟地址最大可以到64位，最多支持的物理地址大小为16TB，即44位寻址
    物理内存采用单级页表本身空间过大，所以采用大概4级页表来查找或者反转页表
    首先通过高几位作为第一级页表目录的偏移量，加上寄存器基址得到的地址，其中保存着二级页表目录的基址，加上后几位的偏移量，得到的内存地址读出得到下一级页表的基址，直到得到最后的物理地址或者硬盘地址。

>  

## 小组思考题
---

（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns。若缺页率是10%，为使有效访问时间达到0.5ms,求不在内存的页面的平均访问时间。请给出计算步骤。 

> 500000 = 150+0.1*x 解得 x = 499850ns

（2）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

```
Virtual Address 6c74 11011 00011 10100 
  --> pde index : 0x1b pde contents:(valid 1,pfn 0x20)
    --> pte index : 0x03 pte contents :(valid 1 pfn 0x61)
      --> Translates to Physical Address 0x6114 --> Value: 06
Virtual Address 6b22 11010 11001 00010
  --> pde index : 0x1a pde contents:(valid 1,pfn 0x52)
    --> pte index : 0x19 pte contents :(valid 1,pfn 0x47)
      --> Translates to Physical Address 0x4702 --> Value: 1a
Virtual Address 03df
  --> pde index : 0x00 pde contents:(valid 1,pfn 0x5a)
    --> pte index : 0x1e pte contents :(valid 1 pfn 0x05)
      --> Translates to Physical Address  --> Value: f
 后面的均是用程序运行的，只给出最后结果
Virtual Address 69dc
  -->Fault (page table entry not valid)
Virtual Address 317a
  --> 0x1e
Virtual Address 4546
  -->Fault (page table entry not valid)
Virtual Address 2c03
  -->0x16
Virtual Address 7fd7
  -->Fault (page table entry not valid)
Virtual Address 390e
  -->Fault (page directory entry not valid)
Virtual Address 748b
  -->Fault (page table entry not valid)
```


（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python, ruby, C, C++，LISP等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。
```
#include <iostream>
#include <fstream>
#include <sstream>

uint8_t memo[4096];//4KB内存
uint8_t disk[32768]; //32KB硬盘

uint8_t get_page(uint32_t x)
{
    int offset = x %32;
    x = x>>5;
    int  pte = x %32;
    x = x>>5;
    int pde = x %32;
    int pdbr = 544;
    int pde_ptr = pdbr + pde;
    printf("%x\n",memo[pde_ptr]);
    int page_dir_found = memo[pde_ptr] >> 7;
    
    if(page_dir_found == 0)
    {
        printf("Fault (page directory entry not valid)\n");
        system("pause");
        return 0;
    }
    else
    {
        int pter = memo[pde_ptr] % (1<<7);
        
        int pde_ptr = (pter << 5) + pte;
        
//        printf("%x\n",memo[pde_ptr]);
        int pframe = memo[pde_ptr];
        int page_table_found = pframe >> 7;
        if(page_table_found ==0)
        {
            printf("Fault (page table entry not valid)\n");
            return 0;
        }
        else
        {
            int pp = ((pframe %(1<<7)) << 5) + offset;
            printf("physical memo addr is 0x%x\n6",pp);
            printf("memo content is 0x%x\n",memo[pp]);
            return memo[pp];
        }
    }
}



using namespace std;

int main(int argc, const char * argv[]) {
    ifstream fin("1.txt");
    //读入内存
    char s[100];
    int c = 0, num;
    while (fin >> s) {
        if (strcmp(s, "page") == 0) {
            fin >> s;
            continue;
        }
        sscanf(s, "%x", &num);
        memo[c] = num;
        c++;
    }
    int addr;
    scanf("%x", &addr);
    get_page(addr);
}
```

（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。
```
    设计一个支持多级页表的系统，在通过页表读取数据时，先通过多级页表找到是否存在与内存中，若不在，则通过IO系统从外设读取至内存中。
```
(5)[X86的页面结构](http://os.cs.tsinghua.edu.cn/oscourse/OS2015/lecture06#head-1f58ea81c046bd27b196ea2c366d0a2063b304ab)
--- 

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。

--- 
