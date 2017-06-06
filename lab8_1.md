//LAB8:EXERCISE1 YOUR CODE HINT: call sfs_bmap_load_nolock, sfs_rbuf, sfs_rblock,etc. read different kind of blocks in file

	/*
	 * (1) If offset isn't aligned with the first block, Rd/Wr some content from offset to the end of the first block
	 *       NOTICE: useful function: sfs_bmap_load_nolock, sfs_buf_op
	 *               Rd/Wr size = (nblks != 0) ? (SFS_BLKSIZE - blkoff) : (endpos - offset)
	 * (2) Rd/Wr aligned blocks 
	 *       NOTICE: useful function: sfs_bmap_load_nolock, sfs_block_op
	 * (3) If end position isn't aligned with the last block, Rd/Wr some content from begin to the (endpos % SFS_BLKSIZE) of the last block
	 *       NOTICE: useful function: sfs_bmap_load_nolock, sfs_buf_op	
	 */
	
	// offset是文件在块中开始的位置, offset % SFS_BLKSIZE表示开始的位置和边界的偏移量
	// 如果nblk != 0，则
	// +				      offset --->   +						 +  <--- SFS_BLKSIZE
	// +									+--------> size <------- +
	// +------------------------------------+------------------------+
	if ((blkoff = offset % SFS_BLKSIZE) != 0) { // 还带有一点儿偏移,也就是说第一个block并没有和块的边界对齐
	    size = (nblks != 0) ? (SFS_BLKSIZE - blkoff) : (endpos - offset);
		// nblks 代表要读/写入的块的数目,如果nblks == 0,表示都在一个块里 读/写
		// size表示这次要写入数据的大小
	    if ((ret = sfs_bmap_load_nolock(sfs, sin, blkno, &ino)) != 0) { // 这里的blkno代表的是索引, ino记录下了返回的值
	        goto out;
	    }
	    if ((ret = sfs_buf_op(sfs, buf, size, ino, blkoff)) != 0) { // sfs_buf_op一般用于写入/读取size长度的数据, blkoff代表偏移量
	        goto out;
	    }
	    alen += size;
	    if (nblks == 0) {
	        goto out;
	    }
		buf += size;
		blkno++;
		nblks--;
	}
	
	size = SFS_BLKSIZE;
	while (nblks != 0) { // 接下来是一个一个读取，写入
		// blkno代表下一个文件块的索引
	    if ((ret = sfs_bmap_load_nolock(sfs, sin, blkno, &ino)) != 0) { // ino得到blkno对应的文件块在磁盘上面的索引
	        goto out;
	    }
	    if ((ret = sfs_block_op(sfs, buf, ino, 1)) != 0) {// 继续读取/写入(现在是一块一块读取或者写入)
	        goto out;
	    }
		alen += size;
		buf += size;
		blkno++; // 下一个文件块的索引(在sin中的索引)
		nblks--; // 需要读取，或者写入的块的数目
	}
	
	if ((size = endpos % SFS_BLKSIZE) != 0) { // 仍然可能没有读完/写完，需要一个结尾的过程
	    if ((ret = sfs_bmap_load_nolock(sfs, sin, blkno, &ino)) != 0) {
	        goto out;
	    }
	    if ((ret = sfs_buf_op(sfs, buf, size, ino, 0)) != 0) {
	        goto out;
	    }
	    alen += size;
	}
out:
    *alenp = alen; // 记录下读入/写入的数据的长度
    if (offset + alen > sin->din->size) {
        sin->din->size = offset + alen;
        sin->dirty = 1;
    }
    return ret;
}

注释说的很清楚，⼤概就是要分三段来做，第⼀段是开头未对⻬部分，接着是中间对⻬部分，最后
是末尾的未对⻬部分。然后可以看到sfs_buf_op和sfs_block_op分别在前⾯已经根据是读⼊还是输出
设定为对应的函数了。sfs_rbuf和sfs_rblock的参数意思也很明确，需要注意的是，由于整体是采取
索引的结果，⼀个⽂件的所有的block都不⼀定会相邻，故⽽在读⼊中间对⻬部分的时候，
sfs_rblock参数中的nblks不能直接设为整个总数，⽽应设为1，⼀个个的依次读⼊（或输出）。