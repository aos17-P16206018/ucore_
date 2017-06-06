对⽐可以发现，lab7相对于lab6，改动的⽂件不少。
⾸先是vmm.[ch]，mm_struct中的mm_lock被替换成mm_sem，即将原来的锁替换成了使⽤信号量
来加锁；
接着是proc.[ch]，在系统创建的第⼆个内核进程中，创建完⽤户进程后，调⽤了check_sync来检验
哲学家问题。然后增加了do_sleep函数，⽤来使当前进程睡眠⼀定时间，函数中将当前进程设为
SLEEPING状态，同时增加timer来达到定时的作⽤，最后调⽤schedule来放弃CPU；
然后sched.c中max time slice被减⼩到了20；
接着syscall.c中增加了sys_sleep这个系统调⽤，通过调⽤之前的do_sleep来实现进程sleep；
测试程序中增加了sleep.c，sleepkill.c⽤来测试新增的sleep函数以及之前的kill函数是否正常⼯作;
最后，改动最⼤的就是sync部分：
增加了wait.[ch]，实现了等待队列（wait queue）以及wait结构的各种基本操作，增加wakeup_wait
⽤于唤醒等待队列中⼀个特定的wait，wakeup_first⽤于唤醒等待队列中的第⼀个wait，
wakeup_queue⽤于唤醒整个等待队列中所有的wait，wait_current_set⽤于使当前进程进⼊等待队
列；
增加了sem.[ch]，实现了信号量，up和down分别对应于信号量的signal和wait，分别调⽤内部__up
和__down；
增加了monitor.[ch]，利⽤前⾯实现的信号量，实现了管程和条件变量（需要后⾯⾃⼰实现），增加
cond_signal和cond_wait函数；
check_sync.c实现了哲学家问题，分别以信号量和条件变量的⽅式实现，每个哲学家都是在不断的
循环进⾏思考、拿起筷⼦（叉⼦）、放下筷⼦（叉⼦），从⽽根据最后实际的进餐情况可以知道是
信号量（或条件变量）是否确实起作⽤了。