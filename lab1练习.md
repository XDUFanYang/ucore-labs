实验报告 网上参考：https://blog.csdn.net/szny_/article/details/102944695

# 练习1：

![image-20210502163518488](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210502163518488.png)

1.镜像文件如何一步一步生成的？

答：先是gcc将代码文件转换成.o文件，然后再用ld 指令将目标文件转换为执行程序，dd指令会把bootloader放到虚拟硬盘中去。

```makefile
+ cc kern/init/init.c
+ cc kern/libs/stdio.c
+ cc kern/libs/readline.c
+ cc kern/debug/panic.c
+ cc kern/debug/kdebug.c
+ cc kern/debug/kmonitor.c
+ cc kern/driver/clock.c
+ cc kern/driver/console.c
+ cc kern/driver/picirq.c
+ cc kern/driver/intr.c
+ cc kern/trap/trap.c
+ cc kern/trap/vectors.S
+ cc kern/trap/trapentry.S
+ cc kern/mm/pmm.c
+ cc libs/string.c
+ cc libs/printfmt.c
+ ld bin/kernel
+ cc boot/bootasm.S
+ cc boot/bootmain.c
+ cc tools/sign.c
+ ld bin/bootblock
'obj/bootblock.out' size: 492 bytes
build 512 bytes boot sector: 'bin/bootblock' success!
10000+0 records in
10000+0 records out
5120000 bytes (5.1 MB, 4.9 MiB) copied, 0.0146885 s, 349 MB/s
1+0 records in
1+0 records out
512 bytes copied, 0.000118597 s, 4.3 MB/s
155+1 records in
155+1 records out
79840 bytes (80 kB, 78 KiB) copied, 0.000286961 s, 278 MB/s
```

在编译后多了这些内容：

```makefile
bin/
├── bootblock
├── kernel
├── sign
└── ucore.img

obj/
├── boot
│   ├── bootasm.d
│   ├── bootasm.o
│   ├── bootmain.d
│   └── bootmain.o
├── bootblock.asm
├── bootblock.o
├── bootblock.out
├── kern
│   ├── debug
│   │   ├── kdebug.d
│   │   ├── kdebug.o
│   │   ├── kmonitor.d
│   │   ├── kmonitor.o
│   │   ├── panic.d
│   │   └── panic.o
│   ├── driver
│   │   ├── clock.d
│   │   ├── clock.o
│   │   ├── console.d
│   │   ├── console.o
│   │   ├── intr.d
│   │   ├── intr.o
│   │   ├── picirq.d
│   │   └── picirq.o
│   ├── init
│   │   ├── init.d
│   │   └── init.o
│   ├── libs
│   │   ├── readline.d
│   │   ├── readline.o
│   │   ├── stdio.d
│   │   └── stdio.o
│   ├── mm
│   │   ├── pmm.d
│   │   └── pmm.o
│   └── trap
│       ├── trap.d
│       ├── trapentry.d
│       ├── trapentry.o
│       ├── trap.o
│       ├── vectors.d
│       └── vectors.o
├── kernel.asm
├── kernel.sym
├── libs
│   ├── printfmt.d
│   ├── printfmt.o
│   ├── string.d
│   └── string.o
└── sign
    └── tools
        ├── sign.d
        └── sign.o
```

## 如何调试Makefile?

并不存在makefile 调试器之类的东西可用来查看特定规则是如何被求值的，或某个变量是如何被扩展的。相反，大部分的调试过程只是在输出的动作中查看makefile。

除了定义其中的脚本外，还可以通过可选项控制make输出：

+ “-n” “--just-print”不执行参数，这些参数只是打印命令，不管目标是否更新，把规则和连带规则下的命令打印出来，这些参数对于我们调试makefile很有用处。

+ “-t” “--touch” 这个参数的意思就是把目标文件的时间更新，但不更改目标文件。也就是说，make假装编译目标，但不是真正的编译目标，只是把目标变成已编译过的状态。

+ “-q” “--question” 这个参数的行为是找目标的意思，也就是说，如果目标存在，那么其什么也不会输出，当然也不会执行编译，如果目标不存在，其会打印出一条出错信息。

+ “-W <file>;” “--what-if=<file>;” “--assume-new=<file>;” “--new-file=<file>;” 这个参数需要指定一个文件。一般是是源文件（或依赖文件），Make会根据规则推导来运行依赖于这个文件的命令，一般来说，可以和“-n”参数一同使用，来查看这个依赖文件所发生的规则命令。
+ 当然还有别的很多选项等等...



我们用make -n指令后，得到如下信息：

```makefile
echo + cc kern/init/init.c
gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o
#gcc：调用编译器 -Ikern/init/：寻找第一个头文件目录是kern/init/ 
#-fno-builtin：确保自己编写的函数和库函数重名冲突的情况下正确编译
#-Wall：输出所有警告信息
#-ggdb：生成专门用于gdb的调试信息
#-m32:int long 指针大小为32位，并且程序能在i386系统上运行
#-gstabs:此选项以stabs格式声称调试信息，但是不包括gdb调试信息。
#-nostdinc：使编译器不在系统默认的头文件目录里面找头文件
#-fno-stack-protector：停用栈保护机制
#-c kern/init/init.c：只编译不连接，产生.o文件
#-o obj/kern/init/init.o：产生init.o文件
echo + cc kern/libs/stdio.c
gcc -Ikern/libs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/libs/stdio.c -o obj/kern/libs/stdio.o
echo + cc kern/libs/readline.c
gcc -Ikern/libs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/libs/readline.c -o obj/kern/libs/readline.o
echo + cc kern/debug/panic.c
gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/panic.c -o obj/kern/debug/panic.o
echo + cc kern/debug/kdebug.c
gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/kdebug.c -o obj/kern/debug/kdebug.o
echo + cc kern/debug/kmonitor.c
gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/kmonitor.c -o obj/kern/debug/kmonitor.o
echo + cc kern/driver/clock.c
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/clock.c -o obj/kern/driver/clock.o
echo + cc kern/driver/console.c
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/console.c -o obj/kern/driver/console.o
echo + cc kern/driver/picirq.c
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/picirq.c -o obj/kern/driver/picirq.o
echo + cc kern/driver/intr.c
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/intr.c -o obj/kern/driver/intr.o
echo + cc kern/trap/trap.c
gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/trap.c -o obj/kern/trap/trap.o
echo + cc kern/trap/vectors.S
gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/vectors.S -o obj/kern/trap/vectors.o
echo + cc kern/trap/trapentry.S
gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/trapentry.S -o obj/kern/trap/trapentry.o
echo + cc kern/mm/pmm.c
gcc -Ikern/mm/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/mm/pmm.c -o obj/kern/mm/pmm.o
echo + cc libs/string.c
gcc -Ilibs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/  -c libs/string.c -o obj/libs/string.o
echo + cc libs/printfmt.c
gcc -Ilibs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/  -c libs/printfmt.c -o obj/libs/printfmt.o
mkdir -p bin/
echo + ld bin/kernel
ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel  obj/kern/init/init.o obj/kern/libs/stdio.o obj/kern/libs/readline.o obj/kern/debug/panic.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/picirq.o obj/kern/driver/intr.o obj/kern/trap/trap.o obj/kern/trap/vectors.o obj/kern/trap/trapentry.o obj/kern/mm/pmm.o  obj/libs/string.o obj/libs/printfmt.o
#ld:链接指令  -m elf_i386：模拟i386架构
#-nostdlib：链接时不用标准库  -T:指定自己的链接脚本, 它将代替默认的连接脚本
#-o bin/kernel:输出文件
objdump -S bin/kernel > obj/kernel.asm
#objdump -S：将代码段反汇编的同时，将反汇编代码和源代码交替显示
objdump -t bin/kernel | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$/d' > obj/kernel.sym
#objdump -t：输出目标文件的符号表
echo + cc boot/bootasm.S
gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o
echo + cc boot/bootmain.c
gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootmain.c -o obj/boot/bootmain.o
echo + cc tools/sign.c
gcc -Itools/ -g -Wall -O2 -c tools/sign.c -o obj/sign/tools/sign.o
gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign
echo + ld bin/bootblock
ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
objdump -S obj/bootblock.o > obj/bootblock.asm
objcopy -S -O binary obj/bootblock.o obj/bootblock.out
bin/sign obj/bootblock.out bin/bootblock
#dd：用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换
#if：输入文件名  of：输出文件名 count=10000:拷贝1000个块 conv=notrunc：指定参数 出错不停止
#seek=1:从输出文件开头跳过1个块后再开始复制
dd if=/dev/zero of=bin/ucore.img count=10000
dd if=bin/bootblock of=bin/ucore.img conv=notrunc
dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc
```

关于dd命令的更多参数：https://www.cnblogs.com/classics/p/11512709.html

关于该问题的其他回答: https://blog.csdn.net/tangyuanzong/article/details/78595854

2.符合规范的硬盘主引导扇区的特征是什么？(提示：tools/sign.c)

```c++
#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <sys/stat.h>
int main(int argc, char *argv[]) {
    struct stat st;
    if (argc != 3) {
        fprintf(stderr, "Usage: <input filename> <output filename>\n");
        return -1;
    }
    if (stat(argv[1], &st) != 0) {
        fprintf(stderr, "Error opening file '%s': %s\n", argv[1], strerror(errno));
        return -1;
    }
    printf("'%s' size: %lld bytes\n", argv[1], (long long)st.st_size);
    //1.确保st.st_size小于等于510
    if (st.st_size > 510) {
        fprintf(stderr, "%lld >> 510!!\n", (long long)st.st_size);
        return -1;
    }
    char buf[512];
    memset(buf, 0, sizeof(buf));
    FILE *ifp = fopen(argv[1], "rb");
    int size = fread(buf, 1, st.st_size, ifp);
    //将ifp指针所指文件中的内容 以1位基本单元 往buf中输出st.st_size个
    if (size != st.st_size) {
        fprintf(stderr, "read '%s' error, size is %d.\n", argv[1], size);
        return -1;
    }
    fclose(ifp);
    //最后一个位为0xAA 倒数第二位为0x55
    buf[510] = 0x55;
    buf[511] = 0xAA;
    FILE *ofp = fopen(argv[2], "wb+");
    size = fwrite(buf, 1, 512, ofp);
    if (size != 512) {
        fprintf(stderr, "write '%s' error, size is %d.\n", argv[2], size);
        return -1;
    }
    fclose(ofp);
    printf("build 512 bytes boot sector: '%s' success!\n", argv[2]);
    return 0;
}
```

1.主引导扇区大小为512b

2.最后两位内容分别是0x55 0xAA 其余位置内容设为0

# 练习2：

![image-20210502211911629](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210502211911629.png)

调试过程：单步跟踪BIOS的执行；并且在初始化位置0x7c00设置断点，得到如下结果，断点正常。

![image-20210502222504749](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210502222504749.png)

从0x7c00开始跟踪代码执行，将单步跟踪反汇编得到的代码与bootasm.S和bootblock.asm比较：

![image-20210508111103314](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210508111103314.png)

与bootasm.S比较：

![image-20210508111416045](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210508111416045.png)

发现一样。

与bootblock.asm比较：

![image-20210508113606478](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210508113606478.png)

发现断点后反编译的代码和两个代码文件中的一样。

自己找断点，找的是0x7c14，发现没有问题。

# 练习3

![image-20210502223313592](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210502223313592.png)

是从16->56行 但是没有这个文件啊 妈的！看一下别人的解析吧！

###  1.关中断和清除数据段寄存器

```assembly
.globl start
start:
.code16                                             
    cli              //关中断                          
    cld              //清除方向标志                           
    xorw %ax, %ax    //ax清0                           
    movw %ax, %ds    //ds清0                                
    movw %ax, %es    //es清0                               
    movw %ax, %ss    //ss清0                 
```

### [练习3.1] 为何开启A20，以及如何开启A20？

初始时A20为0，访问超过1MB的地址时，就会从0循环计数，将A20地址线置为1之后，才可以访问4G内存。A20地址位由8042控制，8042有2个有两个I/O端口：0x60和0x64。

打开流程：

1.等待8042 Input buffer为空；

2.发送Write 8042 Output Port （P2）命令到8042 Input buffer；(为什么要打开两次？) 

3.等待8042 Input buffer为空；

4.将8042 Output Port（P2）得到字节的第2位置1，然后写入8042 Input buffer；

**（这里不太明白？为啥需要等待两次？）**

```assembly
seta20.1:            //等待8042键盘控制器不忙
    inb $0x64, %al   //从0x64端口中读入一个字节到al中           
    testb $0x2, %al  //测试al的第2位
    jnz seta20.1     //al的第2位为0，则跳出循环

    movb $0xd1, %al  //将0xd1写入al中                         
    outb %al, $0x64  //将0xd1写入到0x64端口中                          

seta20.2:            //等待8042键盘控制器不忙
    inb $0x64, %al   //从0x64端口中读入一个字节到al中           
    testb $0x2, %al  //测试al的第2位
    jnz seta20.2     //al的第2位为0，则跳出循环

    movb $0xdf, %al  //将0xdf入al中                         
    outb %al, $0x60  //将0xdf入到0x60端口中，打开A20                  
```

### [练习3.2] 如何初始化GDT表？

#### 1 载入GDT表

```c++
 lgdt gdtdesc       //载入GDT表1
```

#### 2 进入保护模式：

通过将cr0寄存器PE位置1便开启了保护模式

cro的第0位为1表示处于保护模式

```assembly
movl %cr0, %eax       //加载cro到eax
orl $CR0_PE_ON, %eax  //将eax的第0位置为1
movl %eax, %cr0       //将cr0的第0位置为1
```

#### 3 通过长跳转更新cs的基地址:

 上面已经打开了保护模式，所以这里需要用到逻辑地址。$PROT_MODE_CSEG的值为0x80

```assembly
ljmp $PROT_MODE_CSEG, $protcseg
.code32                          
protcseg:
```

#### 4 设置段寄存器，并建立堆栈

```assembly
 movw $PROT_MODE_DSEG, %ax //                      
 movw %ax, %ds                                  
 movw %ax, %es                                   
 movw %ax, %fs                                   
 movw %ax, %gs                                   
 movw %ax, %ss                                   
 movl $0x0, %ebp  //设置帧指针
 movl $start, %esp  //设置栈指针
```

### 5 转到保护模式完成，进入boot主方法

```assembly
call bootmain //调用bootmain函数
```

### [练习3.3] 如何使能和进入保护模式

将cr0寄存器置1

## [练习4] 分析bootloader加载ELF格式的OS的过程。

### [练习4.1] bootloader如何读取硬盘扇区的？

读取扇区硬盘的代码：

bootloader让CPU进入保护模式后，下一步的工作就是从硬盘上加载并运行OS。考虑到实现的简单性，bootloader的访问硬盘都是LBA模式的PIO（Program IO）方式，即所有的IO操作是通过CPU访问硬盘的IO地址寄存器完成。

在上一个联系中我们的BootLoader已经成功的进入了保护模式，接下来我们要做的就是从硬盘读取并运行我们的OS。对于硬盘来说，我们知道是分成许多扇区的其中每个扇区的大小为512字节。读取扇区的流程我们通过查询指导书可以看到：
1、等待磁盘准备好；
2、发出读取扇区的命令；
3、等待磁盘准备好；
4、把磁盘扇区数据读到指定内存。
接下来我们需要了解下如何具体的从硬盘读取数据，因为我们所要读取的操作系统文件是存在0号硬盘上的，所以，我们来看一下关于0号硬盘的I/O端口：

![0号硬盘端口](https://img-blog.csdn.net/20171122220412091?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGFuZ3l1YW56b25n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```c++
static void
waitdisk(void) { //如果0x1F7的最高2位是01，跳出循环
    while ((inb(0x1F7) & 0xC0) != 0x40)
        /* do nothing */;
}
/* readsect - read a single sector at @secno into @dst */
static void
readsect(void *dst, uint32_t secno) {
    // wait for disk to be ready
    waitdisk();

    outb(0x1F2, 1);        //读取一个扇区
    outb(0x1F3, secno & 0xFF);  //要读取的扇区编号
    outb(0x1F4, (secno >> 8)&0xFF);//用来存放读写柱面的低8位字节 
    outb(0x1F5, (secno >> 16)&0xFF);//用来存放读写柱面的高2位字节
          // 用来存放要读/写的磁盘号及磁头号
    outb(0x1F6, ((secno >> 24) & 0xF) | 0xE0);
    outb(0x1F7, 0x20);       // cmd 0x20 - read sectors

    // wait for disk to be ready
    waitdisk();//（为啥都需要两个等待呢？）

    // read a sector
    insl(0x1F0, dst, SECTSIZE / 4); //获取数据
}
```

一般主板有2个IDE通道，每个通道可以接2个IDE硬盘。访问第一个硬盘的扇区可设置IO地址寄存器`0x1f0-0x1f7`实现的，具体参数见下表。一般第一个IDE通道通过访问IO地址`0x1f0-0x1f7`来实现，第二个IDE通道通过访问`0x170-0x17f`实现。每个通道的主从盘的选择通过第6个IO偏移地址寄存器来设置。从`outb()`可以看出这里是用LBA模式的PIO（Program IO）方式来访问硬盘的。从磁盘IO地址和对应功能表可以看出，该函数一次只读取一个扇区。`readseg`简单包装了`readsect`，可以从设备读取任意长度的内容。 

```c++
static void
    readseg(uintptr_t va, uint32_t count, uint32_t offset) {//这个va是啥 count是啥 offset是啥
        uintptr_t end_va = va + count;//看起来count是可以装的内容 end_va是后面的边界

        va -= offset % SECTSIZE;
    
        uint32_t secno = (offset / SECTSIZE) + 1; 
        // 加1因为0扇区被引导占用
        // ELF文件从1扇区开始

        for (; va < end_va; va += SECTSIZE, secno ++) {
            readsect((void *)va, secno);
        }
    }
```

### [练习4.2] bootloader是如何加载ELF格式的OS？

ELF定义：

```c++
/* file header */
struct elfhdr {
    uint32_t e_magic;     // must equal ELF_MAGIC 判断ELF文件是否符合相应正确格式
    uint8_t e_elf[12];
    uint16_t e_type;      // 1=relocatable, 2=executable, 3=shared object, 4=core image
    uint16_t e_machine;   // 3=x86, 4=68K, etc.
    uint32_t e_version;   // file version, always 1
    uint32_t e_entry;     // entry point if executable 程序入口所对应的虚拟地址
    uint32_t e_phoff;     // file position of program header or 0 是program header表的位置偏移
    uint32_t e_shoff;     // file position of section header or 0
    uint32_t e_flags;     // architecture-specific flags, usually 0
    uint16_t e_ehsize;    // size of this elf header
    uint16_t e_phentsize; // size of an entry in program header
    uint16_t e_phnum;     // number of entries in program header or 0 program header表中的入口数目
    uint16_t e_shentsize; // size of an entry in section header
    uint16_t e_shnum;     // number of entries in section header or 0
    uint16_t e_shstrndx;  // section number that contains section name strings
};
```

在这里我们只需要关注其中的几个参数，e_magic，是用来判断读出来的ELF格式的文件是否为正确的格式；e_phoff，是program header表的位置偏移；e_phnum，是program header表中的入口数目；e_entry，是程序入口所对应的虚拟地址。

在bootmain函数中:

```C++
#define ELFHDR          ((struct elfhdr *)0x10000) 
#define SECTSIZE        512

void
    bootmain(void) {
        // 首先读取ELF的头部 八个扇区的数据读到0x10000
        readseg((uintptr_t)ELFHDR, SECTSIZE * 8, 0);

        // 通过储存在头部的幻数判断是否是合法的ELF文件
        if (ELFHDR->e_magic != ELF_MAGIC) {
            goto bad;
        }

        struct proghdr *ph, *eph;

        // ELF头部有描述ELF文件应加载到内存什么位置的描述表，
        // 先将描述表的头地址存在ph
        ph = (struct proghdr *)((uintptr_t)ELFHDR + ELFHDR->e_phoff);
        eph = ph + ELFHDR->e_phnum;

        // 按照描述表将ELF文件中数据载入内存
        for (; ph < eph; ph ++) {
            readseg(ph->p_va & 0xFFFFFF, ph->p_memsz, ph->p_offset);
        }
        // ELF文件0x1000位置后面的0xd1ec比特被载入内存0x00100000
        // ELF文件0xf000位置后面的0x1d20比特被载入内存0x0010e000

        // 根据ELF头部储存的入口信息，找到内核的入口
        ((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))();

    bad:
        outw(0x8A00, 0x8A00);
        outw(0x8A00, 0x8E00);
        while (1);
    }

```

总结一下就是：

1. 从硬盘读了8个扇区数据到内存`0x10000`处，并把这里强制转换成`elfhdr`使用；
2. 校验`e_magic`字段；
3. 根据偏移量分别把程序段的数据读取到内存中。

# 练习5

通过之前的read_ebp() read_eip()进行print_stackframe()的编写。print_debuginfo()\

这里有关于栈帧的讲解：

https://www.zhihu.com/zvideo/1270314551532396544

解释最后一行各个数值的含义：

ebp:0x00007bf8 eip:0x00007d71 arg:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8 
    <unknow>: -- 0x00007d70 --

ebp是当前ebp寄存器里面的内容 eip是当前eip中的内容 arg是传进来的实参

# 练习6

请完成编码工作和回答如下问题：

1. 中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？其中哪几位代表中断处理代码的入口？(占？)
2. 请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init。在idt_init函数中，依次对所有中断入口进行初始化。使用mmu.h中的SETGATE宏，填充idt数组内容。每个中断的入口由tools/vectors.c生成，使用trap.c中声明的vectors数组即可。
3. 请编程完善trap.c中的中断处理函数trap，在对时钟中断进行处理的部分填写trap函数中处理时钟中断的部分，使操作系统每遇到100次时钟中断后，调用print_ticks子程序，向屏幕上打印一行文字”100 ticks”。

**1.中断与异常的学习** ：

https://chyyuu.gitbooks.io/ucore_os_docs/content/lab1/lab1_3_3_2_interrupt_exception.html

所有的中断都会引起特权级的转换嘛？

特权级的转变，会引起用户态的栈想特权级的栈的转变吗？

这个联系6还是存疑的...

表结构：

```c++
struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
};//这是位域 就是第一个是16位 门控电路经常这么搞
```

可以看到，中断向量表一个表项占用8字节，其中2-3字节是段选择子，0-1字节和6-7字节拼成偏移量，

通过段选择子去GDT中找到对应的基地址，然后基地址加上偏移量就是中断处理程序的地址，中断处理代码的入口由offset和ss指定。显然占用64位，也就是8字节，中断处理代码的入口由offset和ss指定。

这里的unsigned 怎么限定后面的位数？

SETGATE函数的实现:

```c++
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```

```c++
void
idt_init(void) {
    extern uintptr_t __vectors[];  //保存在vectors.S中的256个中断处理例程的入口地址数组
    int i;
   //使用SETGATE宏，对中断描述符表中的每一个表项进行设置
    for (i = 0; i < sizeof(idt) / sizeof(struct gatedesc); i ++) { //IDT表项的个数
    //在中断门描述符表中通过建立中断门描述符，其中存储了中断处理例程的代码段GD_KTEXT和偏移量__vectors[i]，特权级为DPL_KERNEL。这样通过查询idt[i]就可定位到中断服务例程的起始地址。
     SETGATE(idt[i], 0, GD_KTEXT, __vectors[i], DPL_KERNEL);
    }
    SETGATE(idt[T_SWITCH_TOK], 0, GD_KTEXT,     
    __vectors[T_SWITCH_TOK], DPL_USER);
     //建立好中断门描述符表后，通过指令lidt把中断门描述符表的起始地址装入IDTR寄存器中，从而完成中段描述符表的初始化工作。
    lidt(&idt_pd);
}
```

**有一个大问题：就是特权级和用户级都是istrap为0吗？**

为啥没有时钟中断...看一下 ...emmm那这里的时钟中断从哪里来的呢？

有没有什么好方法可以将文件书的*.c内容打印出来？

```

├── boot
│   ├── asm.h
│   ├── bootasm.S
│   └── bootmain.c
├── kern
│   ├── debug
│   │   ├── assert.h
│   │   ├── kdebug.c
│   │   ├── kdebug.h
│   │   ├── kmonitor.c
│   │   ├── kmonitor.h
│   │   ├── panic.c
│   │   └── stab.h
│   ├── driver
│   │   ├── clock.c
│   │   ├── clock.h
│   │   ├── console.c
│   │   ├── console.h
│   │   ├── intr.c
│   │   ├── intr.h
│   │   ├── kbdreg.h
│   │   ├── picirq.c
│   │   └── picirq.h
│   ├── init
│   │   └── init.c
│   ├── libs
│   │   ├── readline.c
│   │   └── stdio.c
│   ├── mm
│   │   ├── memlayout.h
│   │   ├── mmu.h
│   │   ├── pmm.c
│   │   └── pmm.h
│   └── trap
│       ├── trap.c
│       ├── trapentry.S
│       ├── trap.h
│       └── vectors.S
├── libs
│   ├── defs.h
│   ├── elf.h
│   ├── error.h
│   ├── printfmt.c
│   ├── stdarg.h
│   ├── stdio.h
│   ├── string.c
│   ├── string.h
│   └── x86.h
├── Makefile
└── tools
    ├── function.mk
    ├── gdbinit
    ├── grade.sh
    ├── kernel.ld
    ├── sign.c
    └── vector.c

```

# 拓展练习

