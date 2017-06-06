if ((ptep = get_pte(mm->pgdir, addr, 1)) == NULL) { // 找到addr对应的页表项

```c
		cprintf("get_pte in do_pgfault failed\n");
		goto failed;
	}

	
	if (*ptep == 0) { // if the phy addr isn't exist, then alloc a page & map the phy addr with logical addr
		// 页表项为空，说明还没有为这个地址分配物理页
		// 分配了页之后，基本上缺页中断就消除了
		
		
		if (pgdir_alloc_page(mm->pgdir, addr, perm) == NULL) { // 那就分配一个页，并且实现了映射，也就是将addr映射到分配的页上面
			cprintf("pgdir_alloc_page in do_pgfault failed\n");
			goto failed;
		}
	}
	else { // if this pte is a swap entry, then load data from disk to a page with phy addr
		// and call page_insert to map the phy addr with logic addr
		// 页表项不为空，页已经存在了，那就是这个物理页被写入到了磁盘中，应该换入
		if (swap_init_ok) { // 也就是说开启了虚拟内存的机制
			struct Page *page = NULL;
			if ((ret = swap_in(mm, addr, &page)) != 0) { // 要换入一个页
				cprintf("swap_in in do pagefault failed\n");
				goto failed;
			}
			// 换页成功了！page记录了换入的数据的页
			// 最后只需要将这个addr映射到这个page就行了
			page_insert(mm->pgdir, page, addr, perm); // 将这个页映射到这个addr地址上面
			swap_map_swappable(mm, addr, page, 1); // 根据FIFO，这个地址应该是最晚被换出的,同时要将这个也标记为可以交换
			page->pra_vaddr = addr;// 记录下这个page对应的虚拟地址
		}
		else { // 不过swap还未初始化，报错。
			cprintf("no swap_init_ok but ptep is %x, failed\n", *ptep);
			goto failed;
		}

	}
	ret = 0;
failed:
	return ret;
```
pte中存在一些标志位记录页面信息，例如该页是否被访问过，是否被修改过，页面是在内存中还是在营盘镇等信息，从而帮助调度算法判断选择哪个页面进行调度。

引起一个中断，硬件将寄存器信息压栈保存，然后去处理页访问异常，最后回复现场从断点处继续往后执行。