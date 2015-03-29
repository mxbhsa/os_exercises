#LAB2实验报告
####马晓彬


## 练习1

### 设计实现过程

直接运行发现原本的错误出现在分配的内存不是正确的块，即地址不对，但是还是分配了。原有的代码比较完全只是空闲空间的排列次序不符合要求，所以推断是排列次序的问题。所以通过查看分析快排列的代码，将default_free_pages函数中，将``` list_add(&free_list, &(base->page_link));```这个代码变为```list_add_before(&free_list, &(base->page_link));```就完成了练习一。

### 你的first fit算法是否有进一步的改进空间

算法。

---
## 练习2
### 设计实现过程

```
//此函数创建一个二级页表项
    pde_t *pdep = pgdir + PDX(la);   // (1) find page directory entry
    int present = *pdep & PTE_P;
    uintptr_t page_ptr;
    if( ! present)//探测有无二级页表
    {
    	if(! create)
    		return NULL;
    	struct Page * page  = alloc_page();//分配一个二级页表（多个表项）
    	set_page_ref(page,1); //设置有引用
    	page_ptr = page2pa(page);//将页管理区域的偏移转换为地址偏移
        *pdep = page_ptr | PTE_U | PTE_W | PTE_P; //设置三个flag位为1 2级页表用户访问权限默认为1
      //pdep为二级页表的入口
    }
    //先找到pdep中对应的物理地址的PDE，将其转换为虚地址，即得到了二级页表入口的虚地址
    //若有二级页表，则直接将pdep（二级页表入口）转换得到页表入口虚地址
    return  ((pte_t *) KADDR(PDE_ADDR(  *pdep)))+PTX(la);
```
### 请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中每个组成部分的含义和以及对ucore而言的潜在用处。

（3）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持8KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

###如果ucore执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？
---
## 练习3
###设计实现过程
###数据结构Page的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？
###如果希望虚拟地址与物理地址相等，则需要如何修改lab2
(1)请分析原理课的缺页异常的处理流程与lab3中的缺页异常的处理流程（分析粒度到函数级别）的异同之处。
