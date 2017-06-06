pde_t *pdep = &pgdir[PDX(la)]; // 首先是找到对应page directory entry

```c
if (!(*pdep & PTE_P)) { // PTE_P是存在，也即测试这个page directory entry对应的page table entry是否存在
	struct Page *page; // 如果不存在的话，那就需要重新分配一个页面了

	if (!create || (page = alloc_page()) == NULL) {
		return NULL;
	}
	set_page_ref(page, 1); // 创建了一个页，然后设置这个页已经被使用了
	uintptr_t pa = page2pa(page); // 得到对应的物理地址
	*pdep = pa | PTE_U | PTE_W | PTE_P; // 这里标记已经存在了这样一个页面啦，并且将新申请的页面的物理地址写入*pdep
}
//这个函数要求返回一个虚拟地址，以供放置page table entry

// 中间的10位是page table index,最后的12位是页内的偏移
// PTX是返回这个线性对应的page table index

// PDE_ADDR(*pdep）是从*pdep这个pde中取出页表的基地址，然后PTX(la)指出这个la在页表中的偏移
// 然后返回这个地址
// KADDR(pa)返回pa这个物理地址对应的虚拟地址
return &((pte_t *)KADDR(PDE_ADDR(*pdep)))[PTX(la)];
```
}

问题
pde指向一个页表基址，pte指向一个物理页面基址，都在高20位，然后剩下一些标志位，例如p表示页面是否存在，u表示用户/内核态，这些位置对进行页面替换算法时起到选择换出页面的用处，p位标志是否存在。

要请求一个中断，然后将当前环境和错误等保存，启动缺页异常的中断服务例程，把硬盘上的页面读入，然后将寄存器恢复中断前的值，继续执行。