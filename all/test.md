#LAB2实验报告
####马晓彬


## 练习1

### 设计实现过程

* default_init函数沿用原有的代码未作修改
* default_init_memmap函数未作修改，原有的代码将所有页的管理块都初始化并分为了一整块添加到了free_list链表的后边
* Default_alloc_pages函数仅是将``` list_add(&free_list, &(p->page_link));```这个代码段


### 缺页异常嵌套

（1）缺页异常可用于虚拟内存管理中。如果在中断服务例程中进行缺页异常的处理时，再次出现缺页异常，这时计算机系统（软件或硬件）会如何处理？请给出你的合理设计和解释。
```
系统无法执行相应的处理程序，说明操作系统内核程序出现问题，系统会重启。因为理论上操作系统应该能够正确处理第一次遇到的问题，处理的过程应该是完全正确的。
```

### 缺页中断次数计算
（2）如果80386机器的一条机器指令(指字长4个字节)，其功能是把一个32位字的数据装入寄存器，指令本身包含了要装入的字所在的32位地址。这个过程最多会引起几次缺页中断？
> 提示：内存中的指令和数据的地址需要考虑地址对齐和不对齐两种情况。需要考虑页目录表项invalid、页表项invalid、TLB缺失等是否会产生中断？
```
指令缺页中断两次，装入字缺页中断两次。
```
### 虚拟页式存储的地址转换

（3）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持8KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示内存映射不存在（有两种情况：a.对应的物理页帧swap out在硬盘上；b.既没有在内存中，页没有在硬盘上）。
PFN6..0:页帧号或外存中的后备页号
PT6..0:页表的物理基址>>5
```

已经建立好了1个页目录表和8个页表，且页目录表的index为0~7的页目录项分别对应了这8个页表。

在[物理内存模拟数据文件](./04-1-spoc-memdiskdata.md)中，给出了4KB物理内存空间和4KBdisk空间的值，PDBR的值。

请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents，the value of addr in phy page OR disk sector。
```
Virtual Address 6653: 页目录表未找到 page directory not found

Virtual Address 1c13:
pde index:0x00000007    pde contents:0x000000bd valid: 1    pfn: 0x0000003d
physical memo addr is 0xed3
memo content is 0x12

Virtual Address 6890: 页目录表未找到 page directory not found

Virtual Address 0af6:
pde index:0x00000002    pde contents:0x000000a1 valid: 1    pfn: 0x00000021
physical memo addr is 0xff6
memo content is 0x1a

Virtual Address 1e6f:
pde index:0x00000007    pde contents:0x000000bd valid: 1    pfn: 0x0000003d
physical memo addr is 0x2cf
memo content is 0x0
```

**提示:**
```
页大小（page size）为32 Bytes(2^5)
页表项1B

8KB的虚拟地址空间(2^15)
一级页表：2^5
PDBR content: 0xd80（1101_100 0_0000, page 0x6c）

page 6c: e1(1110 0001) b5(1011 0101) a1(1010 0001) c1(1100 0001)
b3(1011 0011) e4(1110 0100) a6(1010 0110) bd(1011 1101)
二级页表：2^5
页内偏移：2^5

4KB的物理内存空间（physical memory）(2^12)
物理帧号：2^7

Virtual Address 0330(0 00000 11001 1_0000):
--> pde index:0x0(00000)  pde contents:(0xe1, 11100001, valid 1, pfn 0x61(page 0x61))
page 61: 7c 7f 7f 4e 4a 7f 3b 5a 2a be 7f 6d 7f 66 7f a7
69 96 7f c8 3a 7f a5 83 07 e3 7f 37 62 30 7f 3f 
--> pte index:0x19(11001)  pte contents:(0xe3, 1 110_0011, valid 1, pfn 0x63)
page 63: 16 00 0d 15 00 1c 1d 16 02 02 0b 00 0a 00 1e 19
02 1b 06 06 14 1d 03 00 0b 00 12 1a 05 03 0a 1d
--> To Physical Address 0xc70(110001110000, 0xc70) --> Value: 02

Virtual Address 1e6f(0 001_11 10_011 0_1111):
--> pde index:0x7(00111)  pde contents:(0xbd, 10111101, valid 1, pfn 0x3d)
page 6c: e1 b5 a1 c1 b3 e4 a6 bd 7f 7f 7f 7f 7f 7f 7f 7f
7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f
page 3d: f6 7f 5d 4d 7f 04 29 7f 1e 7f ef 51 0c 1c 7f 7f
7f 76 d1 16 7f 17 ab 55 9a 65 ba 7f 7f 0b 7f 7f 
--> pte index:0x13  pte contents:(0x16, valid 0, pfn 0x16)
disk 16: 00 0a 15 1a 03 00 09 13 1c 0a 18 03 13 07 17 1c 
0d 15 0a 1a 0c 12 1e 11 0e 02 1d 10 15 14 07 13
--> To Disk Sector Address 0x2cf(0001011001111) --> Value: 1c
```

附源程序：
```

#include <iostream>
#include <fstream>
#include <sstream>

using namespace std;


uint8_t memo[4096];//4KB内存
uint8_t disk[4096]; //32KB硬盘

void load(uint8_t* target, char* name)
{
    ifstream fin(name);
    //读入内存
    char s[100];
    int c = 0, num;
    while (fin >> s) {
        if (strcmp(s, "page") == 0) {
            fin >> s;
            continue;
        }
        sscanf(s, "%x", &num);
        target[c] = num;
        c++;
    }

}

uint8_t get_page(uint32_t x)
{
    int offset = x %32;
    x = x>>5;
    int  pte = x %32;
    x = x>>5;
    int pde = x %32;
    int pdbr = 0xd80;
    int pde_ptr = pdbr + pde;
    
    //printf("%x\n",memo[pde_ptr]);
    int page_dir_found = memo[pde_ptr] >> 7;
    
    printf("pde index:0x%08x\tpde contents:0x%08x\tvalid: %d\tpfn: 0x%08x\n",pde,memo[pde_ptr],page_dir_found,memo[pde_ptr] % (1<<7));
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
        //printf("%x\n",memo[pde_ptr]);
        int pframe = memo[pde_ptr];
        int page_table_found = pframe >> 7;
        if(page_table_found ==0)
        {
            ///printf("Fault (page table entry not valid)\n");
            int pp = ((pframe %(1<<7)) << 5) + offset;
            printf("physical memo addr is 0x%x\n",pp);
            printf("memo content is 0x%x\n",disk[pp]);
            return disk[pp];
            return 0;
        }
        else
        {
            int pp = ((pframe %(1<<7)) << 5) + offset;
            printf("physical memo addr is 0x%x\n",pp);
            printf("memo content is 0x%x\n",memo[pp]);
            return memo[pp];
        }
    }
}


int main(int argc, const char * argv[]) {
    load(memo, "1.txt");
    load(disk, "2.txt");
    int addr;
    scanf("%x", &addr);
    get_page(addr);
}
```

## 扩展思考题
---
(1)请分析原理课的缺页异常的处理流程与lab3中的缺页异常的处理流程（分析粒度到函数级别）的异同之处。
