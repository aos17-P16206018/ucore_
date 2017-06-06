 /* LAB5:EXERCISE2 YOUR CODE

         * replicate content of page to npage, build the map of phy addr of nage with the linear addr start
         *
         * Some Useful MACROs and DEFINEs, you can use them in below implementation.
         * MACROs or Functions:
         *    page2kva(struct Page *page): return the kernel vritual addr of memory which page managed (SEE pmm.h)
         *    page_insert: build the map of phy addr of an Page with the linear addr la
         *    memcpy: typical memory copy function
         *
         * (1) find src_kvaddr: the kernel virtual address of page
         * (2) find dst_kvaddr: the kernel virtual address of npage
         * (3) memory copy from src_kvaddr to dst_kvaddr, size is PGSIZE
         * (4) build the map of phy addr of  nage with the linear addr start
         */
    	void *kva_src = page2kva(page); // 得到page的虚拟地址
    	void *kva_dst = page2kva(npage); // 得到npage的虚拟地址
    	memcpy(kva_dst, kva_src, PGSIZE); // 直接拷贝
    
    	ret = page_insert(to, npage, start, perm); // 这个函数主要用于实现映射，具体来说，将start这个线性地址映射到npage这个物理页上
        assert(ret == 0);
        }
        start += PGSIZE;
