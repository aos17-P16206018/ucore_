//LAB4:EXERCISE1 YOUR CODE

```c
/*
 * below fields in proc_struct need to be initialized
 *       enum proc_state state;                      // Process state # 进程状态
 *       int pid;                                    // Process ID # pid
 *       int runs;                                   // the running times of Proces
 *       uintptr_t kstack;                           // Process kernel stack
 *       volatile bool need_resched;                 // bool value: need to be rescheduled to release CPU?
 *       struct proc_struct *parent;                 // the parent process
 *       struct mm_struct *mm;                       // Process's memory management field
 *       struct context context;                     // Switch here to run process
 *       struct trapframe *tf;                       // Trap frame for current interrupt
 *       uintptr_t cr3;                              // CR3 register: the base addr of Page Directroy Table(PDT)
 *       uint32_t flags;                             // Process flag
 *       char name[PROC_NAME_LEN + 1];               // Process name
 */
	proc->state = PROC_UNINIT; // 表示还没有初始化,设置进程为“初始”态
	proc->pid = -1; // 设置进程pid的未初始化值
	proc->runs = 0; // 一次都没有执行
	proc->kstack = 0;
	proc->need_resched = 0;
	proc->parent = NULL;
	proc->mm = NULL;
	memset(&(proc->context), 0, sizeof(struct context));
	proc->tf = NULL;
	proc->cr3 = boot_cr3; // 使用内核页目录表的基址
	proc->flags = 0;
	memset(proc->name, 0, PROC_NAME_LEN);
```
context是当前运行状态上下文，即各个寄存器的值

tf是保存前一个被打断进程的状态，错误信息和寄存器信息等。

本实验中，他们可以在中断或复制过程中帮助记录当前寄存器信息和错误信息等，用于恢复到对应寄存器，或是将这些值复制给子进程。

