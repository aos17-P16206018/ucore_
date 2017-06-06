void
proc_run(struct proc_struct *proc) {

```c
if (proc != current) { // 要运行的进程不是当前的进程
    bool intr_flag;
    struct proc_struct *prev = current, *next = proc;
    local_intr_save(intr_flag); // 关闭中断
    {
        current = proc;
        load_esp0(next->kstack + KSTACKSIZE);
        lcr3(next->cr3); // 加载cr3寄存器中的值，实际上是完成了页表的切换
		// switch_to寄存器值的保存和切换！
        switch_to(&(prev->context), &(next->context));
    }
    local_intr_restore(intr_flag); // 开启中断
}
```
}

创建了两个，一个idle,一个用于打印的线程，完成后在回到idle查找待调度的线程

用于锁，关中断保证对寄存器的互斥访问。