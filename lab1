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
