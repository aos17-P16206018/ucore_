lab1
====
练习一  
----
gcc命令  
-I 指定文件夹  
-fno-builtin 不识别不是以‘_builtin_’为开头的内建功能  
-wall 开启所有warning  
-ggdb 产生gdb的调试信息  
-m32 以32位进行编译
-gstabs 以stabs格式生成调试信息  
-nostdinc 不在标准系统文件夹里找headers文件，只找 -I选项的文件夹  
-fstack-protector 加上额外代码来检测缓冲区是否溢出  
-Os 优化代码体积  
-c 只激活预处理,编译,和汇编,也就是他只把程序做成obj文件  
-o 指定输出文件名

结果用gcc命令编译了init.o，stdio.o，readline.o，clock.o等一系列目标文件。
然后用ld命令链接
-nostdlib 只在命令中明确指定的库文件夹中搜索  
-m 模拟
-o 指定输出文件  

完成后，再编译出bootasm bootmain sign这几个.o文件
用ld链接  
-N 设置程序段和数据段可读写
-e动态链接  
-Ttext 设置该段的首地址 到0x7c00
结果生成bootblock.out文件

dd指令 转换和复制文件  
if= 指定输入路径  
of=指定输出路径  
count= 设置拷贝n块输入块  
conv =notrunc 不截断输出文件  

扇区特征 输入个数必须是3
输入的第二位必须是stat这种数据结构，大小不能超过510字节  
buf[510] = 0x55;  
buf[511] = 0xAA;

练习2
----
1.用make 指令编译后，在lab1_result 目录下执行lab1-mon跟踪bios  
2.在tools/lab1init 文件中已经有  
b *0x7c00  
命令，在0x7c00处设置断点
3.使用  
x /i $pc  
next  
nexti  
step  
stepi  
等指令查看或执行下一步  
4.用vim 修改 lab1init 中的指令
b *0x7c00  
到任意地址，即可从该地址设置断点并开始调试

练习3
----
16行开始  
cli 关中断
cld 关字符串自增加功能  
将ds es ss全部置0
seta20.1:  
和seta20.2:
段 开启a20  
切换到保护模式  
长跳转切换到保护模式

练习4
----

进入bootmain函数后，先调用readseg函数读取函数头
readseg函数通过调用readsec函数完成对扇区的读取
都到到函数头中参数判断，符合条件后继续执行
然后循环将elf文件读入内存中
最后跳转指令将控制权转给ucore



练习5
----

    uint32_t ebp = read_ebp();
    uint32_t eip = read_eip();
    int i, j;
    for (i = 0; ebp != 0 && i < STACKFRAME_DEPTH; i ++) {
        cprintf("ebp:0x%08x eip:0x%08x args:", ebp, eip);
        uint32_t *args = (uint32_t *)ebp + 2;
        for (j = 0; j < 4; j ++) {
            cprintf("0x%08x ", args[j]);
        }
        cprintf("\n");
        print_debuginfo(eip - 1);
        eip = ((uint32_t *)ebp)[1];
        ebp = ((uint32_t *)ebp)[0];
    }

调用read_eip 和 read_ebp获得值
在循环中
输出ebp和eip
指针args指向ebp后两位，输出args四位内的内容
调用print_debuginfo输出调用函数信息
更新eip和ebp的值

练习6
-
1.在mmu.h文件中有gatedesc的结构，共64bit即8字节
 其中 段选择子ss 的2字节
 gd_off 15-0位 31-16位 4自己
 48位共同表示isr入口地址
2.
```c
extern uintptr_t __vectors[];
int i;
for (i = 0; i < sizeof(idt) / sizeof(struct gatedesc); i ++) {
    // DPL代表的是特权级
		// idt[i] 是地址,而__vectors[i]是对应处理函数的地址
    SETGATE(idt[i], 0, GD_KTEXT, __vectors[i], DPL_KERNEL);
}
// 设置转移的入口
SETGATE(idt[T_SWITCH_TOK], 0, GD_KTEXT, __vectors[T_SWITCH_TOK], DPL_USER);
// 加载idt
lidt(&idt_pd);
```

3.
	ticks ++;
	if (ticks % TICK_NUM == 0) {
	        print_ticks();
	    }
	break;



















