if (*ptep & PTE_P) {

```c
	struct Page *page = pte2page(*ptep);
	if (page_ref_dec(page) == 0) { // 减少对这个页面的引用
		free_page(page); // 释放这个页面
	}
	*ptep = 0;
	tlb_invalidate(pgdir, la);
}
```
问题

与页目录项没有直接对应关系，因为记录的是页表的基址

页表项的前20位记录页面的物理地址，与page的每一页一一对应，而不同的进程的页表项可能会同时指向同一个物理页

需要对页表中每个pte进行修改。通过线性地址查找到其对应的pte,将该pte的高20位修改为线性地址la的高20位，则虚拟地址与物理地址相同。