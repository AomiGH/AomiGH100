# Linux相关知识

## 面经

### Linux内核

Linux系统编程：**学习Linux内核的基本架构、内核模块的编写和加载，了解系统调用**等

​	学习内容：Linux基本命令、Shell编程、Linux下的C/C++开发、进程线程管理、网络编程、IO多路复用

系统调用

​	**内存管理和使用：**内存分配/内存管理

​	**多任务编程（进程/线程管理）：**

​		进程管理：进程概念（创建/终止/退出）；进程间通信（管道、无名管道、消息队列、信号、信号量、共享内存、套接字）

​		线程：线程概念；线程编程（创建、终止、等待、同步、互斥、信号）

​	**文件I/O编程：**

​		文件/文件描述符

​		文件结构/文件描述符

​		文件指针/文件描述符

​		标准I/O流、标准I/O操作、非阻塞I/O、异步I/O

​	**网络编程：**

​		TCP/IP协议栈

​		Socket通信（客户端/服务端）

​		TCP/UDP编程

​	学习资料：大丙的教学视频和笔记

​	实践：在Linux环境下编写和调试程序

### 嵌入式Linux

#### 学习整理

​	嵌入式Linux：**掌握如何将Linux移植到嵌入式平台**，如何配置交叉编译工具链，如何创建和调试嵌入式Linux系统，**在Linux开发板上实现一些嵌入式功能**。

​	河畔学长有详细的嵌入式Linux的知识点框架，值得借鉴；在熟悉项目的技术栈中，自己整理相应的知识点框架。目的：掌握专业知识，表达清楚开发板综合性例程的原理，

#### 烤鹅学长

​	linux内核部分可以参考一下韦东山的教学课程；嵌入式linux是我们常见的比如说树莓派，jetson，orin或者别的linux开发板，**在Linux开发板上实现一些嵌入式功能**

​	**有属于自己的项目很重要很重要，基础都学过了，之后不需要再细细的对着框架复习了，秋招也不算是考试嘛，对吧。重要的是能把自己项目整理好，包装好，把里面涉及到的技术栈都搞熟悉。面试基本是围绕你简历提到的技术栈提问的**

​	要稳住，别着急，方向要对，有疑惑我们多沟通。**互联网厂适合纯嵌入式的岗位很少很少。像华为这样去年是从4月份才开始机考**

​	在一个开源项目中的工作（**二次开发、功能迁移**）、付出、思考、效果和收获：

1、增加新功能：你可以去看看同类型的项目有啥功能，你手边做的这个如果没有，那你可以实现一下。
2、已有功能的深挖，比如已有功能有bug，或者能深入做，或者**该功能本身不够完善（有没考虑到的逻辑等等）**这些都可以做
3、又比如，假设你现在的项目是纯32做的，看看能不能上rtos实现，亦或者现在已经是rtos了，能不能再结合linux做点功能开发
最后，功能的实现不能自己”意淫“，可以去看看网上有没有类似的，查查资料。开题都要有研究背景，研究意义。项目也是一个道理。

​	MP3播放器，兼容MP3文件，可以控制音量、播放进度，展示歌曲名。涉及音频接口的驱动程序开发，使用音频解码库（如FFmpeg或者GStreamer）来处理MP3文件。进行内存管理和CPU调度的优化，确保系统稳定且快速响应。

#### 河畔学长

​	熟悉计算机系统底层原理，我认为**最好的学习方式就是实际移植一款RTOS到一个新平台上**。哪里找新平台呢，在这里我推荐risc-v开源社区[**https://github.com/riscv/riscv-cores-list**](https://github.com/riscv/riscv-cores-list)。

​	**以下嵌入式比较重要的两个点，可以参考，STM32加实时操作系统，目前应用的非常广泛，ARM加Linux是一种提高，可以深入学习，处理性能会有大幅提升，比如Cortex-A系列**。

3.1 STM32+RTOS

这一部分的学习，主要是购买正点原子的开发板，结合正点原子和野火的文档，深入理解，野火的文档非常不错，对于一些知识讲得很好。

1. **C语言基本知识，比如：结构体、数组、指针等相关知识，以及用面向对象的思想写C代码，在linux内核源码中会有很多这样的例子，比如，函数指针的应用等**；
2. **STM32裸机相关知识，包括时钟、定时器、中断、GPIO、串口、IIC、SPI等，主要理解硬件原理以及通过代码编程练习对硬件的操作，main函数之前做了哪些事情等等**；
3. RTOS的相关知识，可以选择uCOS、FreeRTOS、RT-Thread等，我主要偏向于FreeRTOS的学习，因为项目会用到，其中**主要掌握FreeRTOS的启动流程，最好移植一个RTOS到STM32中并跑起来；任务调度的原理；任务间通信的几种方式，比如常用的信号量、消息队列，同时实时操作系统中对中断的相应，任务处理的过程，如何保证实时性，软件定时器的学习，包括优先级翻转的问题等**；

**3.2 ARM+Linux**

**我基本上是跟着韦东山老师的视频学习的，讲得很好，比较难，有时需要看2-3遍，最关键的就是自己要动手写代码。**

**1. ARM体系与架构，ARM裸机相关知识，与STM32裸机知识会有很多相通之处，中断处理流程，时钟的配置，外设的基本通信等；**

**2. Linux驱动开发，包括字符设备、块设备和网络设备，我们应届生可以着重字符设备驱动的学习，块设备需要了解Nand Flash/Nor Flash/EMMC的驱动框架，网络设备也作为了解即可。字符设备包括GPIO、IIC、SPI、LCD等，学习驱动，非常重要的就是理解驱动的框架，驱动主要就是框架+硬件操作，比如：理解字符设备驱动框架、总线设备驱动模型；**

**3. 同时需要学习Linux文件系统与设备驱动的关系，用户态与内核态的交互，比如：file、inode结构体的理解等；**

**4. 掌握Uboot启动流程、Linux内核的启动流程，最好能有移植U-boot、内核，构建根文件系统的经历，从而更好的理解嵌入式系统的组成；**

**5. 掌握常用Linux命令，网络编程，自己能写简单的TCP、UDP服务端和客户端代码；**

3.3 项目加持

前边的学习可以和项目同时结合，项目中学习也是非常好的，单片机项目有很多，常见的运用就是，将各种传感器结合起来，进行数据处理，可以加上目前比较火的物联网协议，比如MQTT/NB-IoT等；**ARM-Linux项目**，我一边是导师项目有需求，需要写驱动，另外需要写上位机分析数据、处理数据等，**一方面跟着韦东山老师的视频，视频中也会有很多项目，包括：电子量产工具、数码相框、摄像头视频监控、电源管理等项目，其中用到的编程思想和基本的操作方法不会过时，而且还会是学习的提升**（特此说明，仅仅是个人的学习路线，仅供参考，更没有打广告的意思哈）。朱友鹏老师的海思项目据说也不错，但是对于我来说太贵了，就没学，如果想学习音视频方向的，可以试试。但是我发现**面试过程中，面试官非常看重基础，以及对知识的灵活运用能力，项目的多与少影响不大，或者说是加分项吧**。



### 设备驱动

设备驱动：**学习如何编写Linux下的设备驱动程序，包括字符设备驱动**、块设备驱动等，**掌握Linux驱动开发框架**。

调试与优化：**学习如何使用GDB、JTAG等工具调试Linux系统，了解如何优化系统的性能和资源使用**。



## Linux内核

五个子系统：进程调度、内存管理、虚拟文件系统、网络接口和进程间通信

### 第三章 进程和线程

#### 01 进程控制

1. 进程和程序

​	程序：磁盘内的可执行文件，静态概念

​	进程：被执行后的程序，占用系统内存和CPU资源，动态概念

2. 并发和并行

​	并发：CPU为每个进程分配时间片以便进程运行，时间片结束后收回CPU使用权，而该进程被中断挂起等待下一个时间片。如果进程在时间片内阻塞或者结束，则立即收回CPU使用权。因此，CPU在不同进程中不断切换，宏观上同时运行多个进程。针对单个CPU资源

​	并行：多个CPU资源同时运行各自的进程。针对多个CPU资源

​	进程控制块（Process Control Block, PCB）：记录进程相关信息，包括进程id，进程状态，进程虚拟地址空间，文件描述符表，信号相关信息（在Linux中 调用函数, 键盘快捷键, 执行shell命令等操作都会产生信号。），阻塞信号集（记录当前进程中阻塞哪些已产生的信号，使其不能被处理）

3. 进程状态切换

​	进程状态：包括创建态、就绪态、运行态、阻塞态（挂起态）和终止态

<img src="D:\Users\22499\Desktop\嵌入式学习\typora笔记\笔记图片\01 Linux内核架构\01--进程状态切换.jpg" alt="01--进程状态切换" style="zoom: 25%;" />

4. 进程命令

​	查看进程：ps aux

​	杀死进程：kill -l # 查看发送给进程的标准信号

​			   kill -9 进程ID # 无条件杀死进程

5. 虚拟地址空间

​	定义：内核为每一个运行的进程创建一块用于存储程序数据的虚拟内存空间，它是一段连续的内存地址，起始0地址是虚拟的，而不是物理内存的0地址。32位操作系统虚拟内存空间大小为4 G字节。

​		   CPU中的内存管理单元MMU将进程的虚拟内存空间映射到物理内存上，以供CPU处理。

​	作用：1. 虚拟内存作为程序和物理内存的中间介质，将二者隔离开来，避免程序直接访问物理内存导致不安全行为；2. 动态变化的进程数据如果直接存入物理内存中，会导致物理内存地址也随之动态变化，难以加载数据；3. 物理内存的分配与管理受到硬件的严格限制，而抽象的虚拟内存具有很大的灵活性，运行操作系统以“页”为单位管理内存，提高内存管理效率；4. 扩展有限的物理内存，让程序拥有更多可使用的内存空间。

​	内核区：该虚拟空间保存有操作系统内核，并始终映射到同一片物理内存。不允许应用程序读写该区域内容。

​	用户区：存储应用程序运行中用到的各种数据，包括保留区（空指针指向该区域，任何对它的引用都是非法的）、.text段（程序执行代码）、.data段（存储初始化并且初始值不为0的全局变量和静态变量）、.bss段（未初始化以及初始值为0的全局变量和静态变量）、堆（存放进程运行中动态分配的内存 malloc，向高地址扩展）、内存映射区（用于加载磁盘文件 mmap，设备映射、进程间共享内存）、栈（存放函数内部声明的非静态局部变量、函数参数、函数返回地址等信息，向低地址扩展）、命令行参数（存放main()函数参数，即argc, argv）、环境变量（存放进程相关的工作路径）

6. 进程创建

​	fork()函数：创建一个新进程，返回子进程ID(pid_t类型)。

​	父子进程：父进程中创建(fork)一个子进程，注意：子进程的虚拟内存空间是基于父进程的拷贝而来的，父子进程的用户区的代码段相同，但是执行逻辑不同（父进程是从 main 函数开始执行，而子进程是从 fork 函数之后继续执行），同时，父子进程的虚拟内存空间彼此独立，不会互相干扰（也不会相互交互）；父子进程ID不同，进程状态也可能不同；父子进程的虚拟内存空间都会记录fork()函数的返回值（即存在两个返回值），父进程返回子进程ID，子进程返回0。多进程程序中尽管只有一份代码，但是有多份数据，并且运行逻辑不同，数据的值也不同。

​	终端显示问题：main 进程的父进程为当前终端，终端只能检测到 main 进程状态，在 main 进程执行期间终端切换到后台，main 进程执行完毕后终端切换回前台，此时终端可以正常接收键盘输入，但是子进程可能并未执行结束。

​	exec族函数：当 main 进程需要启动另一个进程时，先创建(fork)一个子进程，在子进程中调用 exec 族函数，此时，子进程的用户区数据被替换掉开始执行新的程序中的代码逻辑，而父进程不受任何影响可以继续工作。

​	结束进程：1. 在程序的任何位置调用 exit() 或者 _exit() 函数，函数的参数是程序退出的状态码；2. 在 main 函数中 return 也可以结束进程。

​	孤儿进程：当父进程在子进程结束之前率先退出，此时Linux的 init 进程(pid = 1)或者桌面终端会领养孤儿进程，并释放子进程的 pcb 资源（子进程可以自行释放用户区资源），避免系统资源的浪费。

​	僵尸进程：当子进程在父进程结束之前率先退出，如果父进程不负责释放子进程的 pcb 资源，此时，子进程成为僵尸进程。杀死僵尸进程的方式是杀死僵尸进程的父进程，使其被系统回收。

​	守护进程：在后台运行的进程，独立于控制终端而运行，用于提供系统服务，例如 SSH 服务、HTTP 服务。创建守护进程的步骤如下，

1. 调用`fork()`函数创建子进程，并让父进程退出，此时，子进程转为后台运行；
2. 调用`setsid()`函数创建新的会话，使子进程脱离控制终端而独立运行（原会话和进程组与控制终端管理）；
3. 再次调用`fork()`函数创建新的子进程，使其不再是会话组长，从而避免重新绑定控制终端；
4. 关闭继承自父进程的所有文件描述符，并将当前目录更改为根目录，避免锁定当前目录；
5. 调用`umask()`清除文件创建时的权限掩码，确保文件权限不受影响；
6. 设置`SIGCHLD`信号为`SIG_IGN`，避免产生僵尸进程。

​	回收进程：1. 阻塞方式 wait() 函数：如果子进程正在执行，wait() 函数一直阻塞等待。当子进程退出，该函数回收子进程资源。2. 精准回收 waitpid() 函数：可以选择阻塞或者非阻塞方式回收，可以精准指定回收某个或者某类或者全部子进程资源。



#### 02 管道

1. 管道概述：管道是进程间通信(IPC - InterProcess Communication)的一种方式，位于 4k 大小的内核缓冲区，通过环形队列来维护。管道的读写两端分别对应两个文件描述符，读数据相当于出队列，数据只能从写端到读端单向传输，读写操作默认阻塞。
2. 匿名管道：pipe() 用于父子进程间通信。子进程拷贝了父进程的文件描述符表，此时，父子进程都可以通过匿名管道的读写两端来访问。同时，为了保证管道的单向流动，仅开启某一进程的读/写端。
3. 有名管道：mkfifo() 在磁盘创建文件类型为 p 的实体文件，文件大小为 0（数据存储在内存缓冲区中）。打开管道文件可以得到读/写端的文件描述符。如果仅打开读端(O_WRONLY)或者写端(O_RDONLY)，进程会阻塞在 open() 函数直到另一个进程打开管道的对端。
4. 读写管道：

​	读管道：1. 写端未关闭，则 a. 管道中没有数据，读阻塞；b. 管道中有数据，读取数据。2. 写端已关闭，则 a. 管道中没有数据，读解除阻塞，read() 函数返回 0 ；b. 管道中有数据，读取完剩余数据后，read() 函数返回 0。

​	写管道：1. 读端未关闭，则 a. 管道中有存储空间，写入数据；b. 管道已满，写阻塞。2. 读端已关闭，则管道破裂（异常），进程直接退出。

​	管道设置：阻塞/非阻塞



#### 03 内存映射区

1. 内存映射区概念：mmap() 内存映射区使用用户区（包括内存映射区）的内存空间。由于每个进程的虚拟空间相互独立，需要通信的进程要将自己的内存映射区映射到同一个磁盘文件。这样，进程通过磁盘文件这一桥梁完成通信。当进程 A 中的内存映射区数据被修改，数据会同步到磁盘文件。同时，映射到磁盘文件的其余进程的内存映射区的数据也会同步修改。

​	父子进程间通信：父进程先创建 mmap() 内存映射区，再创建 fork() 子进程。此时，子进程会拷贝一份同样的内存映射区及对应的指针变量。父/子进程修改内存映射区数据，数据会相互同步。

​	进程 A, B 间通信：进程 A, B 需要创建 mmap() 映射相同磁盘文件的内存映射区。通过关联不同磁盘文件的内存映射区之间的数据拷贝，实现磁盘文件拷贝。



#### 04 共享内存

1. 创建/打开共享内存：1. shmget(key_t key, size_t size, IPC_CREAT|0664|IPC_EXCL)；2. ftok() 返回一个可用于创建、打开共享内存的 key  值。
2. 关联/解除关联：1. shmat() 关联共享内存，返回共享内存的起始地址；2. shmdt() 解除关联，当进程退出时，会自动解除关联。
3. 删除共享内存：当共享内存被标记为删除状态之后，直到所有进程全部和共享内存解除关联，共享内存才会被删除。
4. 进程间通信：a. 进程 A，B 和创建的共享内存进行关联，并进行读/写通信，通信结束后删除共享内存。
5. `shm`和`mmap`的区别

​	首先，共享内存是由 Linux 系统分配的，不属于任何进程，进程的退出不影响共享内存的使用；而进程的内存映射区会随着进程的退出而释放。

​	其次，`mmap()`实现虚拟地址空间与文件/设备空间的映射，访问时需要经过内存和文件之间的数据同步；而共享内存直接对内存进行操作，效率更高。

​	然后，共享内存需要与进程关联后使用，`mmap`也需要映射到同一磁盘文件来进行进程间通信。



#### 05 信号

1. 信号概述

​	信号：一种消息处理机制，是一个整数数组（一共128字节，包括1024个标志位，前31个标志位都对应一个 Linux 中的标准信号）。信号结构简单，不能携带很大的信息量，在系统中的优先级很高。

​	信号处理：五种默认处理动作，包括 Term 终止进程、Ign 信号产生后默认被忽略、Core 终止进程并且生成一个 core 文件、Stop 暂停进程和 Cont 继续运行进程。

​	信号状态：产生 -- 未决 -- 递达

2. 信号相关函数

​	发送信号：1. kill() 发送指定信号到指定进程；2. raise() 发送指定信号到当前进程；3. abort() 发送固定信号(SIGABRT)到当前进程。

​	定时器：1. alarm() 进行单次定时，定时结束后发送 SIGALRM 信号到当前进程；2. setitimer() 周期性定时，每触发一次定时器就会发送信号给当前进程。

3. 信号集

​	阻塞信号集：未阻塞/阻塞的标记位分别为 0/1，信号的阻塞表示让系统暂存信号，等待发送。

​	未决信号集：信号被阻塞，不能处理，标记位设置为 1；如果信号阻塞解除，马上处理信号，标记位清零。

​	信号集函数：用户调用系统函数来间接操作阻塞/未决信号集。1. sigprocmask() 读/写阻塞信号集；2. sigpending() 读未决信号集。

4. 信号捕获：signal() / sigaction() 用于捕获进程中产生的信号，并将用户自定义的回调函数(handler)注册给内核。
5. 子进程资源回收（基于信号）：当子进程退出、暂停或者恢复运行时，子进程会产生一个 SIGCHLD 信号发给父进程。父进程接收到 SIGCHLD 信号后调用自定义回调函数，循环回收子进程 waitpid()。

#### 06 守护进程

1. 进程组：是多个进程的集合，进程组 ID 等于组长 PID。
2. 会话：是由一个或者多个进程组构成的。
3. 创建守护进程：守护进程是 Linux 中的后台服务进程，名字一般以 d 结尾，通常独立于控制终端并且周期性地执行某项任务或者等待处理某些发生的事情。具体步骤如下：a. 创建子进程，让父进程退出；b. 通过子进程创建新的会话，调用函数 setsid()，脱离控制终端，变成守护进程；c. 改变当前进程的工作目录（不能是可卸载的文件系统）；d. 重新设置文件的掩码（更改文件权限）；e. 关闭/重定向文件描述符到特殊文件 /dev/null（即销毁数据）； f. 根据实际需求在守护进程中执行某些特定的操作。

#### 07 多线程

1. 线程概述：进程是资源分配的最小单位，线程是操作系统调度执行的最小单位

​	资源分配：进程具有独立的虚拟内存空间，多个线程共用同一个虚拟内存空间，其中包括共享 .text 段、.bss 段、.data 段、堆区和文件描述符表，同时又有各自独立的栈区和寄存器。

​	调度执行：多线程的上下文切换更快、更高效地利用系统资源。上下文切换是指任务从保存到再次加载这个过程。

2. 线程函数

​	创建线程：1. pthread_create() 创建一个子线程，并且指定处理函数 start_routine；2. pthread_self() 返回当前线程的线程 ID (pthread_t 类型)；3. 链接线程库文件 -lpthread

​	退出线程：pthread_exit()

​	回收线程：pthread_join() 阻塞回收单个线程（线程运行时阻塞，线程退出时解除阻塞），并且获得线程退出时返回的数据。

​	线程退出时返回数据：1. 使用全局变量，而不是栈区变量，因为线程退出会回收栈区数据；2. 使用主线程栈区（子线程创建时的输入参数），因为位于同一个地址空间的多个线程可以互相访问对方的栈区数据。

​	线程分离：pthread_detach() 分离子线程和主线程，当子线程退出时，其占用的内核资源被系统的其他进程接管并回收。如果主线程负责子线程的回收，子线程不退出，pthread_join() 函数始终阻塞，导致主线程无法执行自己的任务。

​	取消线程：pthread_cancel() 在一个线程中杀死另一个线程：1. 在线程 A 中调用 pthread_cancel() 函数，指定杀死线程 B；2. 在线程 B 中进行一次系统调用（a. 直接调用 Linux 系统函数；b. 调用标准 C 库函数）

3. C++ 线程类

​	https://subingwen.cn/cpp/thread/



#### 08 线程同步

1. 线程同步概述

​	线程同步：多个线程按照先后顺序依次访问内存中的共享资源，其余线程阻塞等待直到当前线程对共享资源访问完毕。

​	同步方式：包括互斥锁、读写锁、条件变量和信号量。临界资源：多个线程共同访问的变量，通常为全局数据区变量或者堆区变量。临界区：和临界资源相关的上下文代码。

​	同步步骤：1. 在临界区代码的上方添加加锁函数，对临界区加锁，其他调用临界资源的线程阻塞在锁上；2. 在临界区代码的下方添加解锁函数，对临界区解锁，允许其他线程抢占锁；3. 通过锁机制保证最多只有一个线程访问临界区。

2. 互斥锁

​	互斥锁用于保证同一时间只有一个线程可以访问临界区，当一个线程获取互斥锁时，其他线程只能等待锁被释放。申请锁失败的线程会主动放弃 CPU 时间片进入睡眠状态，直到锁被释放时被操作系统唤醒。

​	优点：实现了进程间通信，避免了忙等待，节省了 CPU 资源；申请锁失败的线程会进行上下文切换，具有一定的成本。

​	互斥锁概述：通过互斥锁来锁定共享资源，线程阻塞/非阻塞申请互斥锁以访问共享资源。

​	互斥锁函数：a. 定义互斥锁变量 pthread_mutex_t mutex；b. 锁定互斥锁 pthread_mutex_lock / 非阻塞锁定互斥锁 pthread_mutex_trylock；c. 解锁互斥锁 pthread_mutex_unlock；d. 销毁互斥锁 pthread_mutex_destroy

​	死锁：1. 加锁之后忘记解锁；2. 重复加锁，第二次加锁造成死锁。

3. 读写锁

​	读写锁允许多个线程同时读取共享资源，但在写操作时需要独占锁；写操作不能和读操作同时执行，并且写操作优先被处理。

​	优点：提高了读操作的并发性能。

​	读写锁概述：使用读写锁锁定了读操作，需要先解锁才能去锁定写操作；反之亦然。使用读锁锁定临界区，所有线程对临界区的访问是并行的；使用写锁则是串行的；写锁比读锁优先级高。

​	读写锁函数：a. 定义读写锁变量 pthread_rwlock_t rwlock；b. 锁定读锁 pthread_rwlock_rdlock() / 非阻塞锁定 pthread_rwlock_tryrdlock()；c. 锁定写锁 pthread_rwlock_wrlock() / 非阻塞锁定 pthread_rwlock_trywrlock()；d. 解锁读写锁 pthread_rwlock_unlock()；e. 销毁读写锁 pthread_rwlock_destroy

4. 条件变量

​	条件变量是一种线程同步机制，允许线程等待某一条件满足时再继续执行；当某个线程需要等待特定条件时，可以释放互斥锁进入等待/休眠状态，直到条件满足时被其他线程唤醒，适用于线程间协作场景。

​	优点：避免了线程忙等待，提高了资源利用率。

​	条件变量概述：配合互斥锁进行条件判断，用于处理生产者和消费者模型。

​	条件变量函数：a. 定义条件变量 pthread_cond_t cond；b. 条件线程阻塞 pthread_cond_wait() 在阻塞线程时，如果线程已经加锁互斥锁，则会解锁 / 在线程解除阻塞时，会再次锁上互斥锁；c. 唤醒阻塞在条件变量的线程 pthread_cond_signal()；d. 销毁条件变量 pthread_cond_destroy()

​	生产者/消费者模型：消费者条件阻塞申请互斥锁，如果有生产资源则正常申请互斥锁，否则阻塞等待生产者模型；生产者模型正常申请互斥锁，在生产结束时唤醒阻塞在条件变量的所有线程。

5. 信号量

​	信号量概述：配合互斥锁执行多线程多任务同步。信号量中的资源 <= 0 时，线程阻塞。

​	信号量函数：a. 定义信号量 sem_t sem；b. 阻塞获取信号量资源 sem_wait() / 非阻塞获取 sem_trywait()；c. 生产信号量资源 sem_post()；d. 查看信号量资源 sem_getvalue()；e. 销毁信号量 sem_destroy()

​	生产者/消费者模型：1. 生产者信号量资源 psem 初始化为 5，消费者信号量资源 csem 初始化为 0；2. 获取 1 个 psem 资源，生成后添加 1 个 csem 资源；3. 获取 1 个 csem 资源，消费后添加 1 个 psem 资源；4. 先阻塞获取信号量资源，再锁定互斥锁；顺序颠倒可能会导致死锁。

6. 自旋锁

​	自旋锁在线程未获取到锁时，持续尝试获取锁，而不会进入休眠状态，避免了线程的上下文切换，适用于锁持有时间较短的场景，例如内核临界区操作。

​	优点：在锁竞争较少、锁持有时间较短的场景下性能优异。



#### 09 手写线程池

1. 线程池概述：包括任务队列、工作者线程和管理者线程。

​	任务队列：存储需要处理的任务。生产者线程添加任务，并唤醒条件阻塞的工作者线程；工作者线程负责处理任务，没有任务时阻塞。

​	管理者线程：周期性检测任务队列中的任务数量以及处于运行状态的工作者线程个数。如果任务过多，则适当添加工作者线程；如果任务过少，则适当销毁工作者线程。

2. 手写线程池

​	线程池函数：1. 创建线程池并且初始化 threadPoolCreate()；2. 销毁线程池 threadPoolDestroy()；3. 给线程池添加任务 threadPoolAdd()；4. 获取线程池中工作的线程个数 threadPoolBusyNum()；5. 获取线程池中活着的线程个数 threadPoolAliveNum()；6. 工作者线程任务函数 worker()；7. 管理者线程任务函数 manager()；8. 单个线程退出函数 thread_exit()。

​	工作者线程任务函数 worker() 反复尝试取出 1 个任务并执行。如果线程池任务为空，则阻塞等待条件变量 pool -> notEmpty；成功取出 1 个任务后，发送条件变量 pool -> notFull。

​	管理者线程任务函数 manager() 循环取出线程池任务数量、当前线程数量和处于运行状态的线程数量，考虑是否添加线程（任务的个数>存活的线程个数 && 存活的线程数<最大线程数），是否销毁线程（忙的线程*2 < 存活的线程数 && 存活的线程>最小线程数）。

​	生产者线程添加任务：放入指定的函数指针 function 和输入参数 arg 到任务队列。



### 第四章 套接字通信

#### 01 套接字 Socket

1. 计算机通信概念

​	计算机网络地址 IP：IP协议版本包括 IPv4 和 IPv6。IPv4 使用 1 个 32 位整数来描述 IP 地址，分成 4 份，每份 1 字节，取值范围为 0 ~ 255；IPv6 使用 1 个 128 位整数来描述 IP 地址，分成 8 份，每份 2 字节，取值范围为 0000 ~ ffff。

​	进程端口：端口 ID 使用 1 个 16 位整数来表示，取值范围为 0 ~ 65535，用于定位主机上的某一个进程来接收到对应的网络数据。一个端口 ID 只能给一个进程使用。

​	OSI/ISO 网络分层模型：包括应用层、表示层、会话层、传输层、网络层、数据链路层和物理层。

​	TCP/IP 四层模型：包括应用层、传输层（建立进程端口之间的连接）、网络互联层（建立计算机 IP 地址之间的连接）和网络接口层。

​	网络协议：是计算机网络中的对等实体之间互相通信时所必须遵循的规则集合，包括通信环境、传输服务、词汇表、信息的编码方式、时序、规则和过程。传输层协议：TCP 协议和 UDP 协议。网络互联层协议：IP 协议。网络接口层协议：以太网帧协议。按照网络协议，数据进行封装和解封装。

2. Socket 编程：Linux 系统中的 TCP/IP 协议网络通信接口，包括客户端和服务端。

​	字节序：表示大于 1 个字符的数据类型在内存中的存放顺序。主机 IP 采用小端，数据的低位字节存储到内存的低位地址；端口 ID 采用大端，数据的低位字节存储到内存的高位地址。

​	字节序转换函数：1. inet_pton() 主机 IP 地址转换为网络字节序；2. inet_ntop() 将大端整数转换为主机 IP 地址；

​	Socket 函数：1. socket() 创建 1 个套接字，返回可用于套接字通信的文件描述符；2. bind() 将文件描述符和本地 IP 与端口进行绑定；3. listen() 给套接字设置监听，用于检测客户端的连接请求；4. accept() 等待并接收客户端的连接请求，建立新的连接，会得到一个文件描述符（套接字），用于和建立连接的客户端通信；当没有新的客户端连接请求时，accept() 函数阻塞。5. read() / recv() 阻塞等待数据到达，然后接收数据；6. write() / send() 发送数据；7. connect() 客服端指定服务端的 IP 和端口，发起连接请求。

3. TCP 通信流程

​	服务器端通信流程：1. socket() 创建用于监听的套接字；2. bind() 将得到的监听文件描述符和本地 IP 和端口进行绑定；3. listen() 设置监听，开始监听客户端的连接请求；4. accept() 阻塞等待并接受客户端的连接请求，建立新的连接，并返回 1 个用于通信的文件描述符；5. read() / write() 阻塞读写数据；6. close() 断开连接，关闭套接字，当两端都调用 close() 函数，则完成了四次挥手。

​	客户端通信流程：1. socket() 创建用于通信的套接字；2. connect() 连接到指定 IP 和端口的服务器端，完成三次握手；3. read() / write() 阻塞读写数据；4. close() 断开连接，关闭套接字。

​	套接字相关的文件描述符：对应 2 块内存，分别是读/写缓冲区。

​	用于监听的文件描述符：如果读缓冲区内有数据，则连接新的客户端。

​	用于通信的文件描述符：1. 发送数据时，将数据写入到 fd 的写缓冲区中，然后内核检测到 fd 的写缓冲区中有数据，将数据发送到网络中；2. 接收数据时，将数据从 fd 的读缓冲区中读出即可。

<img src="D:\Users\22499\Desktop\嵌入式学习\typora笔记\笔记图片\01 Linux内核架构\02--TCP 通信流程.png" alt="02--TCP 通信流程" style="zoom: 50%;" />



#### 02 TCP 协议

1. TCP 协议概述：1. 面向连接：是一次双向连接，通过三次握手完成， 断开连接需要通过四次挥手完成；2. 安全：TCP 通信过程中，对发送的每一个数据包都进行校验，如果发现数据丢失，会自动重传；3. 流式传输：发送端和接收端处理数据的速度、处理数据的量都可以不同。

​	TCP 头部信息：

​		源端口/目的端口：表示发送/接收端端口号，字段长 16 位，2 个字节

​		序号：表示本报文段所发送数据的第一个字节的序号，随机生成。字段长 32 位，4 个字节

​		确认序号：表示接收方期望接收的下一个字节的序号 = 首字节序号 + 发送数据量

​		8 个标志位：

​			ACK：该位置 1，表示确认应答，TCP 协议规定除了最初建立连接的 SYN 包之外该位必须置 1。

​			SYN：该位置 1，表示建立连接，并在其序号字段进行序号初始值设定。

​			FIN：该位置 1，表示断开连接。

​		窗口大小：表示从确认序号所指位置开始能够接受的数据大小，TCP 协议规定不允许发送超过窗口大小的数据。

<img src="D:\Users\22499\Desktop\嵌入式学习\typora笔记\笔记图片\01 Linux内核架构\03 -- TCP 协议数据包结构.png" alt="03 -- TCP 协议数据包结构" style="zoom:50%;" />	

2. 三次握手

​	前提：服务器端启动 socket，并设置监听。

​	第一次握手：1. 客户端(SYN_SENT)向指定 IP 和端口的服务器端发起连接请求，将报文的 SYN 位置 1，生成随机序号 x。2. 服务器(SYN_RCVD)接收客户端发送的请求数据，解析 TCP 协议，校验 SYN 标志位是否为 1，并得到序号 x。

​	第二次握手：1. 服务器(SYN_RCVD)向客户端发送 ACK 回复，确认序号设置为 x+1，表示同意客户端请求；客户端(ESTABLISHED)接收 ACK 包，并校验数据，包括校验 ACK 标志位和序号校验。2. 服务器(SYN_RCVD)向客户端发送 SYN 连接请求，生成随机序号 y。

​	第三次握手：1. 客户端(ESTABLISHED)向服务器发送 ACK 回复，确认序号设置为 y+1，表示同意服务器请求；服务器(ESTABLISHED)接收 ACK 包。

3. 四次挥手

​	第一次挥手：1. 客户端(FIN_WAIT_1)发送 FIN 断开连接请求，生成随机序号 x。2. 服务器(CLOSE_WAIT)接收数据包，解析 TCP 协议，得到序号 x。

​	第二次挥手：1. 服务器(CLOSE_WAIT)发送 ACK 回复，确认序号设置为 x+1；客户端(FIN_WAIT_2)接收 ACK 包，并校验数据。

​	第三次挥手：1. 服务器(LAST_ACK)发送 FIN 断开连接请求，生成随机序号 y。2. 客户端(TIME_WAIT)接收数据包...客户端必须处于 TIME_WAIT 状态，并等待 2MSL(Maximum Segment Lifetime) 时间，保证在它发送的 ACK 回复丢失的情况下，可以重新发送 ACK 回复。

​	第四次挥手：客户端(无状态)发送 ACK 回复，确认序号设置为 y+1；服务器(无状态)接收 ACK 包...

4. 流量控制：接收端主机向发送端主机通知自己可以接收的数据大小（窗口大小），发送端会发送不超过该大小的数据。当接收端的缓冲区空闲内存改变时，窗口大小也随之改变，从而控制数据的发送量，实现流量控制。

5. 半关闭：1. 服务器调用 close() 函数，不能发数据，只能读数据；2. 客户端保存着与服务器的连接，客户端只能发数据，不能读数据；此时，数据单向流动。
6. 端口复用：...
7. TCP 协议的沾包问题：当接收端一次接收到多个数据包时，由于数据包的长度不固定，无法拆分各个数据包。

​	解决方案：数据包由数据头和数据块组成，数据头固定为 4 字节大小，用于存储当前数据块的总字节数。

​	发端：1. 根据待发送的数据块长度 N+4 申请一块堆区内存；2. 将数据块长度写入申请的内存前 4 个字节中，需要转换为网络字节序；3. 将数据块写入内存后续的地址空间，发送数据包 sendMsg()；4. 释放申请的堆区空间。

​	收端：1. 首先接收 4 字节的数据头，将其从网络字节序转换为主机字节序，得到即将接收的数据总长度；2. 根据得到的数据块长度申请一块堆区内存；3. 接收数据包 recvMsg()，将数据块写入申请的内存中；4. 处理接收的数据；5. 释放申请的堆区空间。



#### 03 服务器并发

1. 单进程/单线程服务器场景：服务器被 accept() 函数阻塞时无法通信，被 read() 函数阻塞时无法建立连接，两者相互矛盾，因此无法处理多连接。
2. 多进程并发

​	父进程：1. 负责监听，循环调用 accept() 函数，处理客户端的连接请求；2. 创建子进程，用于与新连接的客户端进行通信；3. 回收子进程资源，防止出现僵尸进程。4. 为了节省系统资源，对于只有在父进程才能用到的资源，可以在子进程中释放掉。

​	子进程：负责通信，基于父进程建立新连接后得到的文件描述符，和对应客户端完成数据的发送和接收。

​	如果父进程调用 accept() 函数没有检测到新的客户端连接，父进程就阻塞在这儿了，这时候有子进程退出了，发送信号给父进程，父进程就捕捉到了这个信号 SIGCHLD， 由于信号的优先级很高，会打断代码正常的执行流程，因此父进程的阻塞被中断，转而去处理这个信号对应的函数 callback()，处理完毕，再次回到 accept() 位置，但是这是已经无法阻塞了，函数直接返回-1，此时函数调用失败，错误描述为 accept: Interrupted system call，对应的错误号为EINTR，由于代码是被信号中断导致的错误，所以可以在程序中对这个错误号进行判断，让父进程重新调用accept()，继续阻塞或者接受客户端的新连接。

```c++
int cfd = accept(lfd, (struct sockaddr*)&cliaddr, &clilen);
while(1)
{
        int cfd = accept(lfd, (struct sockaddr*)&cliaddr, &clilen);
        if(cfd == -1)
        {
            if(errno == EINTR)
            {
                // accept调用被信号中断了, 解除阻塞, 返回了-1
                // 重新调用一次accept
                continue;
            }
            perror("accept");
            exit(0);
 
        }
 }
```

3. 多线程并发

​	主线程：1. 负责监听；2. 创建用于通信的子线程；3. 主/子线程分离：因为线程回收会阻塞，从而影响 accept() 函数监听；4. 保存所有用于通信的文件描述符，因为父/子线程共用同一块地址空间中的文件描述符。

​	子线程：负责通信。



#### 04 套接字通信的类封装

1. 面向对象三要素

​	封装：封装之后的类可以隐藏掉某些属性使操作更简单并且类的功能要单一；

​	继承：如果要代码重用可以进行类之间的继承；

​	多态：如果要让函数的使用更加灵活可以使用多态；

2. 将套接字通信流程封装成类函数。



#### 05 IO 多路复用

1. IO 多路复用并发

​	概述：1. 使用 IO 多路复用函数委托内核阻塞检测服务器所有文件描述符（包括监听和通信两类），如果检测到已就绪的文件描述符，则解除阻塞，并将已就绪的文件描述符传出。2. 如果是监听的文件描述符，调用 accept() 和客户端建立连接；3. 如果是通信的文件描述符，调用 read() / write() 函数和已建立连接的客户端通信。

​	优势：系统不必创建多进程/多线程，节省系统资源。

2. IO 多路复用函数

​	select()：委托内核检测多个文件描述符对应的读写缓冲区状态。

​		输入参数：文件描述符集合 fd_set，用于存储需要检测的所有文件描述符；初始化 FD_ZERO()；放入文件描述符 FD_SET()。

​		局限：1. 待检测的文件描述符集合需要频繁地在内核区和用户区之间拷贝；2. select() 线性检测文件描述符，检测效率较低。

​	poll()：没有最大文件描述符数量限制，只能在 Linux 平台使用



#### 06 UDP 协议

1. UDP 协议概述：1. UDP 通信不需要建立连接；2. UDP 通信过程中，每次都需要指定数据接收端的 IP 和端口；3. UDP 协议不对接收的数据包排序；4. UDP 协议不需要回复确认信息，发端不会重发数据；5. 如果发生数据丢失，则全部丢弃当前数据包。
2. UDP 通信流程

​	服务器：1. 创建用于通信的套接字 socket()；2. bind() 将套接字和本地 IP 和端口绑定；3. 接收/发送数据 recvfrom() / sendto()；4. 关闭套接字。

​	客户端：1. 创建用于通信的套接字 socket()；2. 接收/发送数据 recvfrom() / sendto()；3. 关闭套接字。

3. UDP 广播特性

​	广播概述：通过广播向子网中多台计算机发送消息，并且子网中所有计算机都可以接收到发端发送的消息。

​	广播地址：局域网中的每一个网段都拥有一个特殊的广播地址：192.168.xxx.255



## 嵌入式Linux

### 技术栈细节

字符设备驱动：GPIO, IIC, SPI, LCD等，框架+硬件操作（韦东山老师的课，专业知识总结）

总线设备驱动：

Linux文件系统与设备驱动的关系

用户态和内核态的交互

掌握Uboot启动流程、Linux内核的启动流程

移植Uboot、内核，构建根文件系统，理解嵌入式系统组成

项目加持：数码相框、摄像头视频监控（加入按键硬件操作？？拍照功能移植？？）

学习资料：韦东山老师的嵌入式Linux驱动开发完全手册

​		   韦老师的三期项目实训（数码相框、摄像头驱动开发）

### 嵌入式Linux专业知识

#### 01 嵌入式Linux组成

嵌入式Linux组成

1. bootloader

​	a. BootLoader 是系统上电时运行的一段**初始化程序**，用于**引导并启动操作系统内核**。 这段 BootLoader 程序会先初始化 DDR 等外设，然后将 Linux 内核从 Flash（NAND，NOR FLASH，SD，MMC 等）拷贝到 DDR 中，最后启动 Linux 内核（映像文件，包括头部信息和可执行程序）。
​	b. 常用U-boot，用来引导并启动操作系统内核（还可以引导其他系统，包括USB、网络、SD卡），它分为两个阶段，即 boot + loader， boot 阶段启动系统，初始化硬件设备，建立内存空间映射图， loader 阶段将操作系统内核文件加载至内存，之后跳转到内核所在地址运行。

​	c. 根据需求设置Uboot配置文件，包括硬件支持、启动参数、启动流程等。

2. Linux内核

​	a. Linux内核主要由5部分组成，分别为：**进程管理子系统，内存管理子系统，文件子系统，网络子系统，设备子系统**。

​	b. 进程管理子系统（调度员）

​		a). 管理进程的创建、调度和终止；

​		b). 决定进程的CPU占用

​		c). 进程间通信

​	c. 内存管理子系统（仓库管理员）

​		a). 管理系统的物理内存和虚拟内存资源

​		b). 管理内存的分配和回收，并且实现内存保护机制，确保进程间共享内存和不互相干扰。

​	d. 文件子系统（图书馆管理员）

​		a). 管理系统的文件和目录，提供文件的存储、读取、写入等操作

​		b). 提供虚拟文件系统（VFS），为用户提供统一的文件操作接口

​	e. 网络子系统（邮局）

​		a). 实现网络协议栈，支持各种网络协议

​		b). 管理网络设备，提供网络数据的发送和接受功能。

​	f. 设备子系统（车间管理员）

​		a). 管理系统的硬件设备，包括字符设备、块设备和网络设备

​		b). 提供设备驱动程序接口，使内核能够与各种硬件设备进行交互

3. 根文件系统

交叉工具编译链：arm-linux-gnueabihf-gcc，包括，arm架构是一种精简指令集计算机（RISC）架构，编译生成的可执行程序运行在linux操作系统上，gnu c库是C标准库（STL），eabi表示嵌入式应用二进制接口，hf表示硬浮点运算，gcc表示GNU编译器集合套件。



#### 02 字符设备驱动

1. 字符设备

​	a. **只能一个字节一个字节的读写**的设备，不能随机读取设备内存中的某一数据，**读取数据需要按照先后顺序进行**。字符设备是面向流的设备，常见的字符设备如鼠标、键盘、串口、控制台、LED等。

​	b. 字符设备在/dev目录下有相应的设备文件，**Linux用户程序通过设备文件（设备节点）来使用驱动程序操作字符设备**。

2. 字符设备、Linux内核和用户空间，三者之间的关系

<img src="C:\Users\22499\AppData\Roaming\Typora\typora-user-images\image-20250302171504416.png" alt="image-20250302171504416" style="zoom: 33%;" />

​	a. **Linux内核使用cdev结构体来描述字符设备**，**通过其成员dev_t来定义设备号**，确保字符设备的唯一性；**通过其成员file_operations来定义字符设备驱动提供给VFS的接口函数**，如open(), write()和ioctl()。

​	b. 在Linux字符设备驱动中，模块加载函数（MODULE_INIT()）**通过register_chrdev_region( ) 或alloc_chrdev_region( )来静态或者动态获取设备号**，**通过cdev_init( )建立cdev与file_operations之间的连接**，**通过cdev_add( )向系统添加一个cdev以完成注册**。模块卸载函数（MODULE_EXIT()）**通过cdev_del( )来注销cdev，通过unregister_chrdev_region( )来释放设备号**。

​	c. 用户空间访问该设备的程序通过Linux系统调用，如open( )、read( )、write( )，来“调用”file_operations以定义字符设备驱动提供给VFS的接口函数。

3. 字符设备驱动模板

<img src="C:\Users\22499\AppData\Roaming\Typora\typora-user-images\image-20250302172115966.png" alt="image-20250302172115966" style="zoom: 33%;" />

​	a. 上图缺少class的相关内容，class主要是用来自动创建设备节点的，还有就是一个比较常用的ioctl()函数没有列在上边。

​	b. 每个步骤的具体细节

```
1. 驱动初始化
1.1 分配cdev
1.2 初始化cdev
1.3 注册cdev
1.4 硬件初始化

2. 实现设备操作
2.1 open()
2.2 read()
2.3 write()
2.4 close()
2.5 ioctl()

3. 驱动注销
3.1 删除cdev
3.2 释放设备号

4. 字符设备驱动程序基础
4.1 cdev结构体
4.2 操作cdev结构体
4.3 分配和释放cdev结构体
4.4 cdev结构体中的fileoperations结构体
4.5 file结构体
4.6 inode结构体
4.7 模块加载与卸载函数

5. 自动创建设备节点
```

#### 03 应用开发：数码相框

上机实验：

##### 01 Linux 驱动开发

Linux 源码目录：/home/book/100ask_imx6ull-sdk/Linux-4.9.88
配置交叉工具链：
export ARCH=arm 
export CROSS_COMPILE=arm-linux-gnueabihf-
export PATH=$PATH:/home/book/100ask_imx6ull-sdk/ToolChain/gcc-linaro-6.2.1-2016.11-x86_64_arm-linux-gnueabihf/bin
echo \$ARCH
arm-linux-gnueabihf-gcc -v

编译完成后，在arch/arm/boot目录下生成zImage内核文件, 在arch/arm/boot/dts目录下生成设备树的二进制文件100ask_imx6ull-14x14.dtb。

Crtl+Shift+P	C/C++: Edit configurations(JSON) 	打开 C/C++编辑配置文件

运行测试驱动程序：
挂载根文件系统：mount -t nfs -o nolock,vers=3 192.168.5.11:/home/book/nfs_rootfs /mnt
将 chrdevbase.ko 和 chrdevbaseApp.o 拷贝到 ~/nfs_rootfs/ 目录中
加载驱动：[root@100ask:/lib/modules/4.9.88]# insmod chrdevbase.ko / depmod	 modprobe chrdevbase
查看驱动：cat /proc/devices
创建设备节点文件：mknod /dev/chrdevbase c 200 0	ls /dev/
运行测试程序：./chrdevbaseApp /dev/chrdevbase 1
关闭黑屏：echo -e "\033[9;0]" > /dev/tty0
查看设备节点：cat /proc/bus/input/devices

```
运行结果：
[root@100ask:/lib/modules/4.9.88]# insmod chrdevbase.ko
[root@100ask:/lib/modules/4.9.88]# ./chrdevbaseApp /dev/chrdevbase 1
read data:kernel data!
[root@100ask:/lib/modules/4.9.88]# ./chrdevbaseApp /dev/chrdevbase 2
[root@100ask:/lib/modules/4.9.88]# rmmod chrdevbase.ko

--chrdevbase printk 没有输出
```

##### 02 设备树

设备树的重点内容：

1. DTS、DTB 和 DTC 之间的区别，如何将 .dts 文件编译为 .dtb 文件

2. 设备树语法：如何修改设备树
3. 设备树的特殊子节点
4. 设备树的 OF 操作函数，在驱动文件中如何读取设备树节点的属性信息

##### 03 设备树 LED 驱动

无设备树：在驱动文件 newchrled.c 中定义有关寄存器物理地址，然后使用 io_remap 函数进行内存映射，得到对应的虚拟地址，最后操作寄存器对应的虚拟地址完成对 GPIO 的初始化。

有设备树：使用设备树来向 Linux 内核传递相关的寄存器物理地址，Linux 驱动文件使用上一章讲解的 OF 函数从设备树中获取所需的属性值，然后使用获取到的属性值来初始化相关的 IO

1. 在 imx6ull-alientek-emmc.dts 文件中创建相应的设备节点。

在根节点 ''/'' 下创建名为 ''alphaled'' 的子节点

```
alphaled {
     #address-cells = <1>;
     #size-cells = <1>;
     compatible = "atkalpha-led";
     status = "okay";
     reg = < 0X020C406C 0X04 	/* CCM_CCGR1_BASE */
             0X020E0068 0X04 	/* SW_MUX_GPIO1_IO03_BASE */
             0X020E02F4 0X04 	/* SW_PAD_GPIO1_IO03_BASE */
             0X0209C000 0X04 	/* GPIO1_DR_BASE */
             0X0209C004 0X04 >; /* GPIO1_GDIR_BASE */
 };
```

编译设备树 `make dtbs`

使用新的 xxx.dtb 启动 Linux 内核

Linux 启动成功以后，在 ''/proc/device-tree/'' 目录下查看 ''alphaled'' 节点

```
/* 韦东山IMX6ULL：创建设备节点 */
/* 1. 修改、编译设备树 100ask_imx6ull-14x14.dts */
内核源码目录：/home/book/100ask_imx6ull-sdk/Linux-4.9.88
设备树位置：~/100ask_imx6ull-sdk/Linux-4.9.88/arch/arm/boot/dts
如何修改代码添加 LED 设备节点：创建 VSCode WorkSpace, 位置：~/linux/linux_drivers/02_LED_dts
/* jiamingjie 2025/07/01 */
    jmjled {
            #address-cells = <1>;
            #size-cells = <1>;
            compatible = "jmj-led";
            status = "okay";
            reg = < 0X020C406C 0X04 	/* CCM_CCGR1_BASE */
                    0X02290014 0X04 	/* SW_MUX_GPIO5_IO03_BASE */
                    0X02290058 0X04 	/* SW_PAD_GPIO5_IO03_BASE */
                    0X020AC000 0X04 	/* GPIO5_DR_BASE */
                    0X020AC004 0X04 >;  /* GPIO5_GDIR_BASE */
        };
编译dts：
export ARCH=arm 
export CROSS_COMPILE=arm-linux-gnueabihf-
export PATH=$PATH:/home/book/100ask_imx6ull-sdk/ToolChain/gcc-linaro-6.2.1-2016.11-x86_64_arm-linux-gnueabihf/bin
~/100ask_imx6ull-sdk/Linux-4.9.88$ make dtbs

/* 2. 将新的设备树 .dtb  */
关闭黑屏：echo -e "\033[9;0]" > /dev/tty0
挂载mount：mount -t nfs -o nolock,vers=3 192.168.5.11:/home/book/nfs_rootfs /mnt
查看设备节点：/proc/device-tree/
拷贝：cp ~/100ask_imx6ull-sdk/Linux-4.9.88/arch/arm/boot/dts/100ask_imx6ull-14x14.dtb ~/nfs_rootfs/
拷贝：/boot/
查看原来的和新的设备节点 ls /proc/device-tree
```



2. 编写 LED 驱动程序，获取设备树中的相关属性值。

```
/* 编写 LED 驱动程序 */
打开并修改 C/C++ 编辑配置文件：Crtl+Shift+P	C/C++: Edit configurations(JSON)
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "~/100ask_imx6ull-sdk/Linux-4.9.88/include",
                "~/100ask_imx6ull-sdk/Linux-4.9.88/arch/arm/include",
                "~/100ask_imx6ull-sdk/Linux-4.9.88/arch/arm/include/generated/"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "clang-x64"
        }
    ],
    "version": 4
}
驱动程序代码：
	/* 1、获取设备节点：jmjled */
	dtsled.nd = of_find_node_by_path("/jmjled");
	// 后续其他 OF 函数通过 device_node nd 设备树设备节点来获取属性值
	/* 获取compatible属性内容 */
	proper = of_find_property(dtsled.nd, "compatible", NULL);
	/* 获取status属性内容 */
	ret = of_property_read_string(dtsled.nd, "status", &str);
	/* 获取reg属性内容 */
	ret = of_property_read_u32_array(dtsled.nd, "reg", regdata, 10);
	
	/* 2、reg 属性读取和内存映射 */
	/* 使用“古老”的 ioremap 函数完成内存映射，将读取到的 regdata 数组中的寄存器物理地址转换为虚拟地址 */
	ioremap 函数需要配合 of_property_read_u32_array 函数
	/* 使用 of_iomap 函数一次性完成读取 reg 属性和内存映射 */
	
	/* 3、LED 操作 */
	使能 GPIO5 时钟				val |= (3 << 30);	// 与正点原子不同
	设置 GPIO5_IO03 的复用功能   writel(5, SW_MUX_GPIO5_IO03);
	xxx
编译驱动程序
export ARCH=arm 
export CROSS_COMPILE=arm-linux-gnueabihf-
export PATH=$PATH:/home/book/100ask_imx6ull-sdk/ToolChain/gcc-linaro-6.2.1-2016.11-x86_64_arm-linux-gnueabihf/bin
make
make -j32 # 生成驱动模块文件
测试程序代码：
	/* 1、打开驱动设备 */
	fd = open(filename, O_RDWR);
	/* 2、读取输入：将字符串 '1'/'0' 转换为整型 */
	databuf[0] = atoi(argv[2]);
	/* 3、写入 */
	retvalue = write(fd, databuf, sizeof(databuf));
	/* 4、关闭设备 */
	close(fd);
arm-linux-gnueabihf-gcc ledApp.c -o ledApp	# 编译 ledApp.c
```





3. 使用获取到的有关属性值来初始化 LED 所使用的 GPIO。

##### 04 开源之夏(配置 git email)

```
22499@Jimmy MINGW64 ~
$ cd /d/Users/22499/Desktop/MyGitRepository/

22499@Jimmy MINGW64 /d/Users/22499/Desktop/MyGitRepository
$ git config user.email "jiamingjie0322@163.com"
fatal: not in a git directory

22499@Jimmy MINGW64 /d/Users/22499/Desktop/MyGitRepository
$ git init
Initialized empty Git repository in D:/Users/22499/Desktop/MyGitRepository/.git/

22499@Jimmy MINGW64 /d/Users/22499/Desktop/MyGitRepository (master)
$ git config user.email "jiamingjie0322@163.com"

22499@Jimmy MINGW64 /d/Users/22499/Desktop/MyGitRepository (master)
$ git config user.email
jiamingjie0322@163.com
```

##### 05 gcov

Linux 内核中的 gcov 开发工具 (gcov profiling kernel support, GCC's coverage testing tool)
Linux 内核的覆盖率数据通过 "gcov" debugfs directory 目录以 gcov-compatible format 格式导出
获取特定文件的覆盖率数据，要切换到 the kernel build 目录并且使用带有 -o 的 gcov，如下所示

```
# cd /tmp/linux-out
# gcov -o /sys/kernel/debug/gcov/tmp/linux-out/kernel spinlock.c
```

这将会在当前目录下创建带有执行计数注释的源代码文件。此外，图形化 gcov 前端 (lcov 开发工具) 可用于自动化执行整个内核的数据收集过程，并且提供 HTML 格式的覆盖率概览。

Preparation

配置内核 Configure the kernel (使用 profiling flags 编译的内核会显著增大并且运行更慢)

```shell
CONFIG_DEBUG_FS = y
CONFIG_GCOV_KERNEL = y
CONFIG_GCOV_PROFILE_ALL = y	# to get coverage data for the entire kernel
```

只有挂载了 debugfs 之后才能访问分析数据

```
mount -t debugfs none /sys/kernel/debug
```

如果分析特定的文件或者目录，需要在相应的 kernel Makefile 中添加类似的代码行

```
GCOV_PROFILE_main.o := y	# For a single file (e.g. main.o)
GCOV_PROFILE := y			# For all files in one directory

GCOV_PROFILE_main.o := n	# 排除指定文件
GCOV_PROFILE := n			# 排除指定目录
```

Files

The gcov kernel support 会在 debugfs 中创建以下文件

```
/sys/kernel/debug/gcov			# 所有 gcov 相关文件的父目录
/sys/kernel/debug/gcov/reset	# 全局重置文件：写入时将所有覆盖数据重置为 0
/sys/kernel/debug/gcov/path/to/compile/dir/file.gcda	# gcov 工具能够理解的实际 gcov 数据文件，写入时重置文件覆盖数据为 0
/sys/kernel/debug/gcov/path/to/compile/dir/file.gcno	# 指向 gcov 工具所需要的静态数据文件的符号链接，由使用 option -ftest-coverage 编译的 gcc 来创建
```

##### 06 GPIO驱动开发：pinctrl和GPIO子系统

###### pinctrl子系统

大多数开发板的`pin`都支持复用功能，而且我们需要配置`pin`的电气特性，包括上拉/下拉、速度、驱动能力等。传统配置`pin`的方式是直接操作相应的寄存器，但是这种配置方式比较繁琐，并且容易出错。

1、设备树设备节点

pinctrl子系统：获取设备树中的`pin`信息，并且根据获取到的`pin`信息来初始化`pin`。
pinctrl子系统的源码目录：`drivers/pinctrl`
`imx6ull.dtsi`和`100ask_imx6ull-14x14.dts`位于`Linux-4.9.88\arch\arm\boot\dts`

```
/* iomuxc节点：配置SOC的引脚的复用功能和电气属性，例如GPIO */
imx6ull.dtsi
			iomuxc: iomuxc@020e0000 {
				compatible = "fsl,imx6ul-iomuxc";
				reg = <0x020e0000 0x4000>;
			};
			
100ask_imx6ull-14x14.dts
&iomuxc {
    pinctrl-names = "default";
    pinctrl-0 = <&pinctrl_hog_1>;
    imx6ul-evk {
        pinctrl_hog_1: hoggrp-1 {
            fsl,pins = <
                MX6UL_PAD_UART1_RTS_B__GPIO1_IO19   0x17059 /* SD1 CD */
                MX6UL_PAD_GPIO1_IO00__ANATOP_OTG1_ID    0x17059 /* USB OTG1 ID */
                // MX6UL_PAD_CSI_DATA07__GPIO4_IO28           0x000010B0
                MX6ULL_PAD_SNVS_TAMPER5__GPIO5_IO05        0x000110A0
            >;
        };
......

重要概念：
	compatible = "fsl,imx6ul-iomuxc";				// Linux内核根据compatible属性值来查找对应的驱动文件
	MX6UL_PAD_UART1_RTS_B__GPIO1_IO19   0x17059		// UART1_RTS_B引脚的配置信息
		宏定义：#define MX6UL_PAD_UART1_RTS_B__UART1_DCE_RTS		0x0090 0x031c 0x0620 0 3
		5个值的含义：<mux_reg conf_reg input_reg mux_mode input_val>
			mux_reg：mux_reg寄存器偏移地址(MUX_CTL_PAD, 复用功能)。寄存器地址 = 基地址(iomuxc节点给定) + 偏移量 (mux_reg)
			conf_reg：conf_reg寄存器偏移地址(PAD_CTL_PAD，电气属性)。
			input_reg：input_reg寄存器偏移地址。一些外设需要配置。
			mux_mode：mux_reg寄存器值
			input_val：input_reg寄存器值
		0x17059：conf_reg寄存器值，由用户自行设置
		
```

2、PIN驱动程序讲解

````
/* PIN驱动程序讲解 */
pinctrl-imx6ul.c
    static struct of_device_id imx6ul_pinctrl_of_match[] = {
        { .compatible = "fsl,imx6ul-iomuxc", .data = &imx6ul_pinctrl_info, },
        { .compatible = "fsl,imx6ull-iomuxc-snvs", .data = &imx6ull_snvs_pinctrl_info, },
        { /* sentinel */ }
    };
    of_device_id结构体数组保存着驱动文件的兼容性值，用于与设备树节点的compatible比较，判断是否可以使用此驱动。
    iomuxc节点与此驱动匹配，该驱动文件会完成PIN配置工作。
    
    static int imx6ul_pinctrl_probe(struct platform_device *pdev)
    {
        const struct of_device_id *match;
        struct imx_pinctrl_soc_info *pinctrl_info;

        match = of_match_device(imx6ul_pinctrl_of_match, &pdev->dev);

        if (!match)
            return -ENODEV;

        pinctrl_info = (struct imx_pinctrl_soc_info *) match->data;

        return imx_pinctrl_probe(pdev, pinctrl_info);
    }
    imx6ul_pinctrl_probe()
    	imx_pinctrl_probe()	分配、初始化、注册PIN控制器(pinctrl_desc结构体)
    		imx_pinctrl_probe_dt()	...	解析设备树中关于PIN的配置信息[mux_reg, conf_reg, input_reg, ...]
    		pinctrl_register()		...	向Linux内核注册pinctrl

    static struct platform_driver imx6ul_pinctrl_driver = {
        .driver = {
            .name = "imx6ul-pinctrl",
            .of_match_table = of_match_ptr(imx6ul_pinctrl_of_match),
        },
        .probe = imx6ul_pinctrl_probe,
    };
    platform_driver结构体的成员函数.probe在设备和驱动匹配成功后会被执行。
````

###### gpio子系统

GPIO子系统用于初始化 GPIO 并且提供相应的 API 函数，比如设置 GPIO 为输入输出，读取 GPIO 的值等。

1、设备树设备节点

```
100ask_imx6ull-14x14.dts
    &iomuxc {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_hog_1>;
        imx6ul-evk {
            pinctrl_hog_1: hoggrp-1 {
                fsl,pins = <
                    MX6UL_PAD_UART1_RTS_B__GPIO1_IO19   0x17059 /* SD1 CD */
    pinctrl子系统配置好SD卡CD引脚PIN配置参数
    
100ask_im6ull-14x14.dts
    &usdhc1 {
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_usdhc1>;
        cd-gpios = <&gpio1 19 GPIO_ACTIVE_LOW>;
        keep-power-in-suspend;
        enable-sdio-wakeup;
        bus-width = <4>;
        status = "okay";
    };
    - 设备树SD卡节点位于I.MX6ULL的usdhc1接口上。
    - 尽管usdhc1节点没有指定CD引脚的PIN信息，即没有pinctrl-1 = <&pinctrl_hog_1>，但是，因为在iomuxc节点下引用了 pinctrl_hog_1 这个节点，所以 Linux 内核中的 iomuxc 驱动就会自动初始化 pinctrl_hog_1 节点下的所有 PIN。
    - cd-gpios属性描述了SD卡的CD引脚使用的IO：&gpio1, 19表示第19号IO, GPIO_ACTIVE_LOW表示低电平有效。
    
imx6ul.dtsi
			gpio1: gpio@0209c000 {
				compatible = "fsl,imx6ul-gpio", "fsl,imx35-gpio";
				reg = <0x0209c000 0x4000>;
				interrupts = <GIC_SPI 66 IRQ_TYPE_LEVEL_HIGH>,
					     <GIC_SPI 67 IRQ_TYPE_LEVEL_HIGH>;
				gpio-controller;
				#gpio-cells = <2>;
				interrupt-controller;
				#interrupt-cells = <2>;
				gpio-ranges = <&iomuxc  0 23 10>, <&iomuxc 10 17 6>,
					      <&iomuxc 16 33 16>;
			};
    - compatible属性用于查找GPIO驱动程序
    - reg属性设置GPIO1控制器基地址
    - #gpio-cells属性表示有2个cell，分别用于表示GPIO编号和GPIO极性（低电平有效/高电平有效）

```

2、GPIO驱动程序

```
gpio-mxc.c
    static const struct of_device_id mxc_gpio_dt_ids[] = {
        { .compatible = "fsl,imx1-gpio", .data = &mxc_gpio_devtype[IMX1_GPIO], },
        { .compatible = "fsl,imx21-gpio", .data = &mxc_gpio_devtype[IMX21_GPIO], },
        { .compatible = "fsl,imx31-gpio", .data = &mxc_gpio_devtype[IMX31_GPIO], },
        { .compatible = "fsl,imx35-gpio", .data = &mxc_gpio_devtype[IMX35_GPIO], },
        { /* sentinel */ }
    };
    - of_device_id结构体数组保存着驱动文件的兼容性值，用于匹配设备树节点。在匹配成功后probe

    static struct platform_driver mxc_gpio_driver = {
        .driver		= {
            .name	= "gpio-mxc",
            .pm = &mxc_gpio_dev_pm_ops,
            .of_match_table = mxc_gpio_dt_ids,
        },
        .probe		= mxc_gpio_probe,
        .id_table	= mxc_gpio_devtype,
    };
    - platform_driver结构体的成员函数
    
    mxc_gpio_probe()
    - 维护mxc_gpio_port结构体(对GPIO的抽象)，mxc_gpio_port->bgc.gc(gpio_chip结构体)是对GPIO控制器操作函数的抽象，例如输入、输出
    - 通过imx35_gpio_hwdata结构体(GPIO寄存器组结构体)和reg属性指定的寄存器基地址可以访问GPIO1的所有寄存器，这些寄存器地址会赋值给参数bgc的成员变量：reg_dat, reg_set, reg_clr, reg_dir。
    - gpiochip_add()向Linux内核注册gpio_chip，之后我们就可以在驱动中使用gpiolib提供的各种API函数：
    - gpio_request(): 用于申请一个 GPIO 管脚
    - gpio_free(): 释放GPIO引脚
    - gpio_direction_input(): 设置某个 GPIO 为输入
    - gpio_direction_output(): 设置某个 GPIO 为输出，并且设置默认输出值
    - gpio_get_value(): 获取某个 GPIO 的值(0 或 1)
    - gpio_set_value(): 设置某个 GPIO 的值
```

