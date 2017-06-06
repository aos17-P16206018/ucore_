 /* LAB5:EXERCISE1 YOUR CODE

     * should set tf_cs,tf_ds,tf_es,tf_ss,tf_esp,tf_eip,tf_eflags
     * NOTICE: If we set trapframe correctly, then the user level process can return to USER MODE from kernel. So
     *          tf_cs should be USER_CS segment (see memlayout.h)
     *          tf_ds=tf_es=tf_ss should be USER_DS segment
     *          tf_esp should be the top addr of user stack (USTACKTOP)
     *          tf_eip should be the entry point of this binary program (elf->e_entry)
     *          tf_eflags should be set to enable computer to produce Interrupt
     */
    tf->tf_cs = USER_CS; // 用户代码段选择子
    tf->tf_ds = tf->tf_es = tf->tf_ss = USER_DS;
    tf->tf_esp = USTACKTOP;
    tf->tf_eip = elf->e_entry; // 程序的入口地址
    tf->tf_eflags = FL_IF; // 开启中断
    ret = 0;
先为进程创建代码段和数据段，然后创建内核线程，填写相关信息，把程序代码加载到进程代码段，然后开始执行，通过中断后修改栈的内容，或者其他的方法跳转到用户态，然后开始执行