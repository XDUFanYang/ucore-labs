# 启动

cpu加电 第一条指令在哪 内存分为RAM ROM 第一条指令从ROM中执行。

![image-20210430162452410](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210430162452410.png)

基本输入输出：键盘显示器...

系统设置：硬盘/网络/光盘启动？BIOS设置管理

然后进行开机自检和自启动

首先BIOS将加载程序从磁盘的引导扇区(512字节)放到相应内存地址中，然后将控制权交给读进来程序。这个程序能将操作系统的代码和数据从硬盘加载到内存中，并且跳转到操作系统的起始位置。然后再将控制权转移到os内核代码，os就可以开始运行。

![image-20210430170239727](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210430170239727.png)

BIOS要实现的基本功能：

+ 用中断调用的方式，提供了基本的i/o功能 比如：字符显示 磁盘扇区读写 检测内存大小等等
+ 只能在x86下的实模式进行访问
+ 受限于保护模式访问

## 计算机启动流程

系统启动流程：加电以后去读bios->bios读你的加载程序->加载程序去读内核映像  过程可以进一步细化

主引导记录：从哪个文件系统中去读加载程序；然后是其中的主引导扇区中的活动扇区来去加载程序，写的时候要按照一定的格式去写

![image-20210430173240916](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210430173240916.png)

bios首先硬件自检，确保内存不出错

![image-20210430174515086](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210430174515086.png)

![image-20210430174748697](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210430174748697.png)

![image-20210430175016034](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210430175016034.png)

![image-20210430175330948](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210430175330948.png)

![image-20210430180435485](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210430180435485.png)

# 中断、异常和系统调用

### 为什么需要中断、异常和系统调用？

在计算机运行中 内核是被信任的第三方；需要内核提供其他程序的接口；方便应用程序

### 中断和异常希望解决的问题

键盘键入和除零

当外设连接计算机时 会出现什么现象？

当处理意想不到的行为时，会出现什么？控制权给操作系统

### 系统调用希望解决的问题

用户应用程序如何得到系统服务？应用程序正确使用服务并且保证安全

系统调用/功能调用的不同之处？  

### 内核的进入和退出

![image-20210501152032651](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210501152032651.png)

中断时硬件的处理：设置中断标志；依据中断向量确定设备和例程

软件处理：现场保存(编译器)、中断服务处理(服务例程)、清除中断标志(服务例程)、现场恢复(编译器)

硬件中断可以被打断... 优先级

答错的问题：

1.下列选项中，不可能在用户态发生的事件是（）。

- ```
  系统调用
  ```

- ```
  外部中断
  ```

- ```
  进程切换
  ```

- ```
  缺页
  ```

2.中断处理和子程序调用都需要压栈以保护现场，中断处理一定会保存而子程序调用不需 要保存其内容的是（）。

- ```
  程序计数器
  ```

- ```
  程序状态字寄存器
  ```

- ```
  通用数据寄存器
  ```

- ```
  通用地址寄存器
  ```

## 系统调用

例子：printf()调用底层的write()

系统调用的外界使用情况：

![image-20210501154842903](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210501154842903.png)

系统调用的内部实现：

![image-20210501155312170](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210501155312170.png)

系统调用和应用调用的不同之处：切换堆栈！

![image-20210501155819514](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210501155819514.png)

中断 异常和系统调用的开销：

![image-20210501160221401](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210501160221401.png)

## 系统调用实例

文件复制过程中的系统调用序列：输入相关文件名 检查相关的文件合法性 然后循环读写 显示相应信息

利用read() 函数

![image-20210501161505798](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210501161505798.png)

ucore中系统调用read()的实现：

![image-20210501161733208](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210501161733208.png)

代码演示：

read->sys_read->sys_call

![image-20210501163830639](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210501163830639.png)

sys_call用户态的代码，用户态已经跟踪不到，需要去看内核态：

![image-20210501163952372](C:\Users\12092\AppData\Roaming\Typora\typora-user-images\image-20210501163952372.png)

...