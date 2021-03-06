### as86/id86(Bruce Evans)
	
	立即数前面要加“#”号符（mov eax, #0x1）否则加载其地址处的值;使用一个符号变量前也要加“#”（mov eax, #msg1）否则加载其内容到eax
	
### 编译

	as86 -0 -a -o boot.o boot.s
	ld86 -0 -s -0 boot boot.o
	
	dd ds=32 if-boot of=/dev/fd0 skip=1
	
	
	as [-03agjuw] [-b [bin]] [-lm [list]] [-n name] [-o objfile] [-s sym] srcfile
	默认设置：
		-3		使用80386的32位输出
		list	在标准输出上显示
		name	源文件的基本名称
	
	-0	使用16比特代码段
	-3	使用32比特代码段
	-a	开启GNU as ld的部分兼容性选项
	-b	产生二进制文件，后面可以跟文件名
	-g	在目标文件中仅存入全局符号
	-j	使所有跳转语句均为长跳转
	-l	产生列表文件，后面可以跟随列表文件名
	-m	在列表中扩展宏定义
	-n	后面跟随模块名称
	-o	产生目标文件，后跟目标文件名
	-s	产生符号文件，后跟符号文件名
	-u	将未定义符号作为输入的未指定段的符号
	-w	不显示警告信息
	
	ld [-03Mims[-]] [-T textaddr] [-llib_extension] [-o outfile] infile ...
	默认设置：
		-03		32位输出
		outfile	a.out格式输出
		
	-0	产生具有16比特魔数的头结构，并且对-lx选项使用i86子目录
	-3	产生具有32比特魔数的头结构，并且对-lx选项使用i386子目录
	-M	在标准输出设备上显示已链接的符号
	-T	后面跟随正文基地址
	-i	分离的指令与数据段（I&D）输出
	-lx	将库/local/lib/subdir/libx.a加入链接的文件列表中
	-m	在标准输出设备上显示已链接的模块名
	-o	指定输出文件名，后跟输出文件名
	-r	产生适合于进一步重定位的输出
	-s	在目标文件中删除所有符号

### 嵌入汇编
	
	__asm__("汇编语句"
			：输出寄存器
			：输入寄存器
			：会被修改的寄存器);
			
|代码|说明|代码|说明|
| :---: | --- | :---: | --- |
|a|使用寄存器eax|m|使用内存地址|
|b|使用寄存器ebx|o|使用内存地址并可以加偏移值|
|c|使用寄存器ecx|I|使用常数0-31|
|d|使用寄存器edx|J|使用常数0-63|
|S|使用esi|K|使用常数0-255|
|D|使用edi|L|使用常数0-65535|
|q|使用动态分配字节可寻址寄存器(eax,ebx,ecx,edx)|M|使用常数0-3|
|r|使用任意动态分配的寄存器|N|使用1字节常数（0-255）|
|g|使用通用有效地址即可（eax, ebx, ecx, edx, 内存变量）|O|使用常数0-31|
|A|使用eax与edx联合(64位)|＝|输出操作数。输出值将替换前值|
|+|表示操作数可读可写|&|早期会变的操作数。表示在使用完操作数之前，内容会被修改
 
例：
	
	#define get_seg_byte(seg, addr) \
	({ \
	register char __res; \					
	__asm__("push %%fs; \					//保存寄存器原值
			mov %%ax, %%fs; \				
			movb %%fs:%2, %%al; \			//取一个byte到al寄存器
			pop %%fs" \						//还原寄存器
			：“＝a” (__res) \				//输出寄存器
			："0" (seg), "m" (*(addr))); \	//输入寄存器
			__res;})		
	嵌入式汇编入输出寄存器到输入器作了统一编号，从%0, %1, ... %9.

定义寄存器变量
	
	register int res __asm__("ax");
	
volatile关键字,禁止GCC优化作修改
	
	__asm__ __volatile__(. . . );

圆括号组合语句,一搬以宏写义

	（｛ int y = foo(); int z;
		if (y > 0) z = y;
		else z = -y;
		3 + z;｝）
		
	res = x + （｛. . . ｝）+ b;

### [Makefile](assets/Makefile2)

	目标（target）... : 先决条件（prerequisites）...
		命令（command）

目标可以是程序生成的文件名，也可以是所采用的活动名，比如（clean）。先决条件是一个或多个文件名。命令是make要执行的操作。一条规则可以有多个命令，每个命令自成一行（table键开头）。

##### 自变量(make -p > [Makefile](assets/Makefile))

	$^	表示规则的所有先决条件，包括它们所处目录的名称。
	$<	表示规则中的第一个先决条件
	$@	表示目标对象
	$?	命令执行的返回结果(rm, ls)
	-	上一个命令执行出错，继续执行下一条命令(rm *.o)
	@	不显示命令本身(@echo hello world)
	.PHONY : clean
	make -C path 进入指定目录

	obj = hello.o world.o
	app : $(obj)
		gcc $^ -o $@
	%.o : %.c
		gcc -c $^ -o $@

##### [Makefile函数调用](assets/Makefile.jpeg)
	
##### 隐式规则(不常用)

双后缀是用一对后缀定义的，源后缀和目标后缀

	.c.s :
		cc $(CFLAGS) -nostdinc -Iinclude -S -o $*.s $<
		
	‘$<’的值是'*.c' make规则含义是将*.c编译成'*.s'代码
