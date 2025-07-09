# 嵌入式Linux：ARM+Linux

## 学习路线

Linux驱动 = 面向对象的编程思想 + 良好的程序框架 + 硬件操作

—> 嵌入式Linux系统（Windows 软件系统）

​	流程：（1） 启动 （2）识别 （3） 运行 （4）

​	（1）Bootloader(BIOS)：常用u-boot。不需要修改，少学习

​	（2）Linux内核(windows)：包括内核本身+驱动程序。内核的裁剪、移植是固定的，少学习。

​	（3）根文件系统(C盘)：系统必备APP+我们的APP

​	（4）APP

—> 看课流程

​	（0）环境搭建：

​		第1~3篇

​	（1）应用开发：

​		第4篇：嵌入式Linux应用开发基础知识

​		实践：编写APP操作硬件

​	（2）驱动开发：

​		第5篇：嵌入式Linux驱动开发基础知识

​		实践：70天Linux驱动快速入门（各种模块驱动）

​	（3）实战项目：

​		第6篇：实战项目

​		更多实践：三类项目：调试bug/毕设项目/产品项目



## 环境搭建

安装VMware workstation pro, Ubuntu.



### 熟悉Linux环境

![image-20241024212617228](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20241024212617228.png)

### BootLoader

初始化硬件和引导操作系统

处理流程：Power on --> BootROM --> U-Boot SPL --> U-Boot --> Linux Kernet --> Root Filesystem

BootROM: 初始化硬件（minimum）；加载Secondary Bootloader到RAM

U-Boot: 常见的Secondary Bootloader；更复杂的初始化：硬件初始化（CPU, memory, clocks）；加载Linux Kernel到RAM；提供环境变量；支持更多的Boot源（SD card, NAND, emmc）



### Buildroot

通过交叉编译（Cross-Complie）建立嵌入式Linux系统

生成Bootloader, Kernel Image, and Root filesystem

生产交叉编译工具链；支持多种架构（ARM, x86）；包管理和依赖处理；通过menuconfig接口配置



### Init System（通用概念）

进程号PID为1的第一个Kernel程序；负责系统的初始化；管理系统服务（system services）和守护进程（daemons）

#### System V (SysV) Init

Unit Init System; 使用运行优先级0-6的顺序引导

#### Systemd

并行服务启动；



# 嵌入Linux应用开发基础

### GCC编译过程

预处理（Preprocessing）--> 编译（Compilation）--> 汇编（Assembly）--> 链接（Link）

main.c --> main.i --> main.S --> main.o --> main.elf

#### 预处理（Preprocessing, *.c -> *.i）

预处理器解决：头文件包含（inclusion, #include）; 宏扩展（#define）；条件编译（#ifdef）；注释删除；

#### 编译（Compilation, *.i -> *.S）

编译器将c语言转换为汇编语言：语法检查；代码优化（volatile: 不优化代码）；特定于框架（Architecture）的代码生成；检测警告和错误

```shell
\\ 列举头文件目标，库目录（寻找#include <>:头文件）
echo 'main(){}'| arm-buildroot-linux-gnueabihf-gcc -E -v -
```



#### 汇编（Assembly, *.S -> *.o）

汇编器将汇编语言转换为机器语言 / 目标代码（object code）：创建符号表（symbol table）；重定位（relocatable）

目标代码（object code）结构：.text; .rodata; .data; .bss; .comment

重定位：将映像文件拷贝到内存

#### 链接（Linking, *.o -> *.imx, *.img, ...）

生成最终可执行文件（Executable）：结合多个目标文件；解决符号引用；分配最终地址；链接库

#### 构建系统集成（Makefile: build system integration）

```shell
// 包含configure的交叉编译开源软件，万能命令
./configure --host=arm-buildroot-linux-gnueabihf --prefix=$PWD/tmp
make
make install
```





### File I/O Model

User Space: Applications使用了高级别函数（stdio.h）或者系统调用（system calls），例如：open(); read(); write(); close(); 系统调用是User Space和Kernel Space之间的接口；查询system calls文档: man 2 open

Kernel Space: 通过设备驱动处理实际的设备操作

```c
// 头文件
#include <sys/types.h>  // For basic data types
#include <sys/stat.h>   // For file permission macros
#include <fcntl.h>      // For file control options
#include <unistd.h>     // For POSIX API (read, write, close)
#include <stdio.h>      // For printf

// 验证命令行参数
if (argc != 3) {
    printf("Usage: %s <old-file> <new-file>\n", argv[0]);
    return -1;
}

// 打开原文件：只读
fd_old = open(argv[1], O_RDONLY);

// 创建目标文件
fd_new = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH);
/*
* O_WRONLY: Write only mode
* O_CREAT: Create if file doesn't exist
* O_TRUNC: Truncate if file exists
*/

// Copy循环：用1K缓存来实现高效复制
while ((len = read(fd_old, buf, 1024)) > 0) {
    if (write(fd_new, buf, len) != len) {
        printf("can not write %s\n", argv[2]);
        return -1;
    }
}

// 关闭所有文件
close(fd_old);
close(fd_new);
```

### 输入系统应用编程

#### 输入系统

输入系统：兼容所有输入设备（键盘、鼠标、触摸屏）的框架

输入系统框架：

① APP发起读操作，若无数据则休眠；

② 用户操作设备，硬件上产生中断；

③ 输入系统驱动层（input drivers）相应的驱动程序处理中断；

​	读取到数据，转换为标准的输入事件（struct input_event结构体）

④ 核心层（input core）决定把输入事件转发给事件层（input event handlers）中的某个handler来处理；

​	例如，evdev_handler：仅将数据input_event保存在内核buffer，APP读取时会得到原本的输入数据。

⑤ APP处理输入事件。

​	APP获取输入数据的两种方法：1）直接访问设备节点（比如/dev/input/event0, 1, 2, ...）；2）通过tslib、libinput等库来简介访问设备节点，简化对数据的处理。

#### 内核中的输入设备结构体struct input_dev

#### 输入事件结构体struct input_event

```c
struct input_event {
    struct timeval time;
    __u16 type;  // 表示某类事件，EV_KEY: 按键；EV_REL: 相对位移; EV_ABS: 绝对位移
    __u16 code;  // 该类事件下的某类操作，按键类: 数字键，字母键；触屏类: 绝对位置信息
    __u32 value;  // 事件值，按键类: 0:按下; 1:松开; 2:长按 
    // 事件之间的界线：驱动程序上报一系列数据后，再上报“同步事件”，表达数据上传完毕。此时，APP读取到完整的输入事件。“同步事件”也是struct input_event变量，其中type, code, value均为0。
}
```

#### 输入设备信息

输入设备节点名：/dev/input/eventX, X = 0, 1, 2, ... or /dev/eventX

```shell
// 查看设备节点
ls /dev/input/event* -l
// 查看与eventX对应的相关设备信息，包括设备名称，设备节点，位图（bitmaps, 包括设备支持的事件类型、具有的操作（按键/触屏）等）
// 在开发板执行
cat /proc/bus/input/devices
```

### APP与硬件互动的方式

#### 询问（Query, 非阻塞, Nonblock）

Evdev API提供了ioctl()函数查询设备名称（EVIOCGNAME），设备ID（EVIOCGID），设备状态和编码（EVIOCGBIT）

APP调用open()（system call）时，传入"O_NONBLOCK"

#### 休眠-唤醒（Sleep-Wake）

输入系统在休眠状态响应输入设备的输入事件：ioctl() （system call）+ EVIOCSWAKE

APP调用open()（system call）时，不要传入"O_NONBLOCK"

#### POLL/SELECT

“超时时间”：当驱动程序获得数据时（触发条件）或者到达超时时间，POLL()/SELECT()会返回。poll()需要指明待监测的文件标识符fd，所需要的事件revents和返回的事件events。

不需要反复询问，节省了CPU资源。

POLL事件类型：POLLIN, POLLOUT, ...

#### 异步通知（Asynchronous Note）

不主动询问，在特定输入事件发生时调用信号处理程序

驱动程序有数据时发送信号SIGIO给APP，APP调用相应的信号处理函数来处理SIGIO信号。

### 网络通信

#### 计算机网络协议：五层TCP/IP模型

五层TCP/IP模型是七层OSI模型的简化表示。

物理层（Physical layer）：将数据作为原始bits通过通信介质（电缆）进行物理传输（电信号/光信号，单工/半双工/全双工），处理数据传输。

链路层（Link layer）：在网络上的相邻节点之间提供准确的数据帧（data frame）传输。成帧：将数据包封装在带有帧头/帧尾的帧，用于寻址和错误检测；错误检测和纠正：类似于CRC；链路层协议：Ethernet, Wi-Fi和PPP

网络层（Network layer）：处理多个网络中数据包（data packets）的逻辑寻址和路由。逻辑寻址（Logical addressing）：为网络上的设备分配唯一的IP地址；路由（Routing）：确定数据包从源（source）传输到目的地（destination）的最佳路径；分段和重组（Fragmentation and reassembly）：将数据包分解为更小的单元，并在目的地重新组装；网络层协议：The Internet Protocol (IP)

传输层（Transport Layer）：确保运行在不同主机（host）的应用程序之间可靠高效的数据传输。分段和重组（Segmentation and reassembly）：将数据分成段（segment），并确保它们顺序传输；流量控制（Flow control）：管理数据传输速率以防止拥挤；错误控制（Error control）：检测丢失或损坏的数据段并从中恢复；连接建立和终止（Connection establishment and termination）：面向连接的协议（connection-oriented protocol）：TCP。传输层协议：TCP（Transmission Control Protocol，传输控制协议）和UDP（User Datagram Protocol，用户数据报协议）

​	----------TCP：面向连接的服务，可靠有序----------

​	连接建立：在数据传输之前，TCP使用三次握手(SYN，SYN-ACK，ACK)在发送方和接收方之间建立连接。这确保双方都准备好沟通。 

​	分段和排序：数据被分成段，每个段都有一个序列号。这允许接收方以正确的顺序重组数据，即使数据段到达时顺序不对。 

​	确认和重传：接收方确认收到的数据段。如果数据段丢失，发送方会重新传输它，以确保没有数据丢失。 

​	流量控制：TCP管理数据传输的速率，以防止发送方压倒接收方。

​	 拥塞控制：TCP通过降低发送速率来应对网络拥塞，有助于防止网络过载。

​	----------UDP：无连接服务，速度和效率----------

​	没有连接建立：UDP只是发送数据报，没有任何事先握手。这减少了开销和延迟。 

​	没有排序或确认：数据报是独立发送的，没有交付或顺序的保证。 

​	无流量控制或拥塞控制：UDP以恒定速率发送数据，不管网络条件如何。

​	用于视频、游戏等。

应用层（Application Layer）：为网络通信应用程序提供服务。网页浏览：HTTP和HTTPS，数据传输：FTP。

#### TCP编程

```c
/* --------server.c-------- */
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdio.h>
#include <signal.h>

/*
* 函数功能描述：从8888端口接收客户端数据
* 输入参数：无
* 输出参数：打印客户IP以及发来的消息
* 返回值：无
*/

/* socket
 * bind
 * listen
 * accept
 * send/recv
 */

#define SERVER_PORT 8888
#define BACKLOG     10

int main(int argc, char **argv)
{
	int iSocketServer;
	int iSocketClient;
	struct sockaddr_in tSocketServerAddr;
	struct sockaddr_in tSocketClientAddr;
	int iRet;
	int iAddrLen;

	int iRecvLen;
	unsigned char ucRecvBuf[1000];

	int iClientNum = -1;

	signal(SIGCHLD,SIG_IGN);
	
	/* 服务器端开始建立socket描述符：sockfd */
	iSocketServer = socket(AF_INET, SOCK_STREAM, 0);
	if (-1 == iSocketServer)
	{
		printf("socket error!\n");
		return -1;
	}

	/* 服务器端填充sockaddr_in结构 */
	tSocketServerAddr.sin_family      = AF_INET;
	/* 端口号转变为网络字节序 */
	tSocketServerAddr.sin_port        = htons(SERVER_PORT);  /* host to net, short */
	/* 接收本机所有网口的数据 */
 	tSocketServerAddr.sin_addr.s_addr = INADDR_ANY;
	memset(tSocketServerAddr.sin_zero, 0, 8);
	
	/* 捆绑sockfd描述符到特定的地址和端口 */
	iRet = bind(iSocketServer, (const struct sockaddr *)&tSocketServerAddr, sizeof(struct sockaddr));
	if (-1 == iRet)
	{
		printf("bind error!\n");
		return -1;
	}
	/* 侦听传入连接 */
	iRet = listen(iSocketServer, BACKLOG);
	if (-1 == iRet)
	{
		printf("listen error!\n");
		return -1;
	}

	while (1)
	{
		iAddrLen = sizeof(struct sockaddr);
		/* 服务器阻塞，直到客户端建立连接 */
		iSocketClient = accept(iSocketServer, (struct sockaddr *)&tSocketClientAddr, &iAddrLen);
		if (-1 != iSocketClient)
		{
			iClientNum++;
			/* inet_nota():将32位ipv4地址转换为相应的点分十进制数串 */
			printf("Get connect from client %d : %s\n",  iClientNum, inet_ntoa(tSocketClientAddr.sin_addr));
			if (!fork())
			{
				/* 子进程的源码 */
				while (1)
				{
					/* 接收客户端发来的数据并显示出来 */
					iRecvLen = recv(iSocketClient, ucRecvBuf, 999, 0);
					if (iRecvLen <= 0)
					{
						close(iSocketClient);
						return -1;
					}
					else
					{
						ucRecvBuf[iRecvLen] = '\0';
						printf("Get Msg From Client %d: %s\n", iClientNum, ucRecvBuf);
					}
				}				
			}
		}
	}
	
	close(iSocketServer);
	return 0;
}
```

```c
/* --------client.c-------- */
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdio.h>

/*
* 函数功能描述：向指定IP的8888端口发送数据
* 输入参数：点分十进制服务器IP
* 输出参数：无
* 返回值：无
*/

/* socket
 * connect
 * send/recv
 */

#define SERVER_PORT 8888

int main(int argc, char **argv)
{
	int iSocketClient;
	struct sockaddr_in tSocketServerAddr;
	
	int iRet;
	unsigned char ucSendBuf[1000];
	int iSendLen;
	
	/* 输入参数判断 */
	if (argc != 2)
	{
		printf("Usage:\n");
		printf("%s <server_ip>\n", argv[0]);
		return -1;
	}

	/* 客户端建立sockfd描述符 */
	iSocketClient = socket(AF_INET, SOCK_STREAM, 0);

	/* 客户端填充服务端资料 */
	tSocketServerAddr.sin_family      = AF_INET;
	/* 主机字节序转换为网络字节序 */
	tSocketServerAddr.sin_port        = htons(SERVER_PORT);  /* host to net, short */
 	//tSocketServerAddr.sin_addr.s_addr = INADDR_ANY;
 	if (0 == inet_aton(argv[1], &tSocketServerAddr.sin_addr))
 	{
		printf("invalid server_ip\n");
		return -1;
	}
	memset(tSocketServerAddr.sin_zero, 0, 8);

	/* 客户端发起连接请求 */
	iRet = connect(iSocketClient, (const struct sockaddr *)&tSocketServerAddr, sizeof(struct sockaddr));	
	if (-1 == iRet)
	{
		printf("connect error!\n");
		return -1;
	}

	while (1)
	{
		if (fgets(ucSendBuf, 999, stdin))
		{
			/* 发送数据 */
			iSendLen = send(iSocketClient, ucSendBuf, strlen(ucSendBuf), 0);
			if (iSendLen <= 0)
			{
				close(iSocketClient);
				return -1;
			}
		}
	}
	
	return 0;
}
```

```makefile
all:server client
server:server.c
	gcc $^ -o $@
client:client.c
	gcc $^ -o $@
clean:
	rm server client -f
```



#### UDP编程



### 多线程编程

#### 线程概念

线程：操作系统可以调度的最小单位。

多线程编程：一个进程执行多个任务，所有线程都可以访问进程中的全局变量。

多进程编程：

进程标识：PID；线程表示：tid（pthread_t类型变量），线程号仅在其所属进程中有意义。

```c
#include <pthread.h>
/* 返回当前线程的tid */
pthread_t pthread_self(void);

/* 
 * 创建线程
 * pthread_t *thread: 新建线程的tid
 * const pthread_attr_t *attr: 线程属性，默认NULL
 * void *(*start_routine) (void *): 函数指针，返回值void *，形参为void *
 * void *arg: 线程处理函数（第三个函数指针参数）的输入参数，默认NULL
 */
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);

/* 线程的退出与回收 */
/* a. 进程结束，进程中的所有线程随之结束 */ 
/* b. 线程主动退出，在退出时可传递一个void *类型的数据给主线程 */
void pthread_exit(void *retval);
/* c. 线程被动退出，其他线程使用pthread_cancel()让另一个线程退出，成功执行返回0 */
int pthread_cancel(pthread_t thread);
/* d. 线程资源回收（阻塞方式），默认状态为阻塞，直到成功回收线程后(主进程)才返回 */
int pthread_join(pthread_t thread, void **retval);
/* e. 线程资源回收（非阻塞方式），成功回收返回0 */
int pthread_tryjoin_np(pthread_t thread, void **retval);
```

#### 线程控制

互斥锁（Mutual Exclusion）：通过对临界资源（进程中的全局变量）加锁来保护资源只被单个线程操作，待操作结束后，其余线程才可获得操作权。

```c
#include <pthread.h>
/* 初始化互斥量 */
/* a. 调用函数初始化互斥量
 * pthread_mutex_t *mutex: 互斥量指针
 * pthread_mutexattr_t *restrict attr: 控制互斥量的属性，默认NULL
 */
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *restrict attr);
/* b. 调用宏初始化互斥量 */
pthread_mutex_t mutex = PTHREAD_MUTEX_INITALIZER;

/* 互斥量加锁/解锁 */
/* a. 互斥量加锁（阻塞方式），如果互斥量已加锁，则等待直至互斥量解锁，成功返回0 */
int pthread_mutex_lock(pthread_mutex_t *mutex);
/* b. 互斥量加锁（非阻塞方式），如果互斥量已加锁，则立即返回EBUSY */
int pthread_mutex_trylock(pthread_mutex_t *mutex);
/* c. 互斥量解锁，成功返回0，处理结束后必须解锁，否则会导致其他线程陷入阻塞，形成死锁现象 */
int pthread_mutex_unlock(pthread_mutex_t *mutex);

/* 互斥量销毁，成功返回0 */
int pthread_mutex_destory(pthread_mutex_t *mutex);

// 互斥量加锁和解锁都放在循环内，当多核机器运行时，会发生“抢锁”现象。
```

信号量（semaphores）：解决线程执行顺序。线程A在等待某件事，线程B完成了这件事后就可以给线程A发信号。

```c
#include <pthread.h>
/* 初始化信号量，执行成功返回0
 * sem_t *sem: 信号量
 * int pshared: 0: 线程控制；否则为进程控制
 * unsigned int value: 信号量初始值：0表示阻塞，1表示运行
 */
int sem_init(sem_t *sem, int pshared, unsigned int value);

/* 信号量P/V操作 */
/* a. 检测指定信号量是否有资源可用：
	如果sem value大于0，则sem value-1，调用线程继续执行。如果sem value = 0，则调用线程将阻塞直至该值大于0 */
int sem_wait(sem_t *sem);
/* b. 信号量申请资源（非阻塞方式） */
int sem_trywait(sem_t *sem);
/* c. 释放指定信号量的资源，sem value+1 */
int sem_post(sem_t *sem);

/* 信号量销毁 */
int sem_destory(sem_t *sem);
```

条件变量：基于特定条件（**条件是什么呢？**）同步执行。它们运行线程在继续执行之前等待某个条件为真，来促进资源有效利用，防止竞争。

```c
#include <pthread.h>
/* 初始化条件变量 */
/* 调用函数初始化 */
int pthread_cond_init(pthread_cond_t *cond, pthread_condattr_t *cond_attr);
/* 调用宏初始化 */
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

/* 等待条件变量：阻塞线程，直到一个特定条件被发出信号（通过pthread_cond_signal() or pthread_cond_broadcast()） */
/* 
 * 自动释放相关的mutex，允许其他线程获取该mutex
 * 阻塞该线程，直到另一个线程使用pthread_cond_signal() or pthread_cond_broadcast()发出条件变量的信号
 * 返回前重新获取mutex，确保线程被唤醒后重新占有互斥量。
 */
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);

/* 通知条件变量 */
int pthread_cond_signal(pthread_cond_t *cond);

/* 条件变量销毁 */
int pthread_cond_destory(pthread_cond_t *cond);
```

### I2C应用编程

#### I2C概念

**IIC（Inter-Integrated Circuit，内部集成电路）**：双向数据传输（芯片与设备之间）；数据传输单位是1字节，即8 bits；需要9个时钟，前8个时钟用来传输8 bits数据，第9个时钟用来传输回应信号；传输时，先传输高位；开始信号（S），SCL（时钟线）保持高位，SDA（数据线）从高变低；结束信号（P），SCL保持高位，SDA从低变高；回应信号（ACK），SDA保持低位；SDA传输的数据在SCL高位期间保持稳定，仅能在SCL低位期间变化。

**写操作：芯片发送数据给设备**

​	a. 芯片发出start信号

​	b. 芯片发出设备地址，方向（读/写，1/0）

​	c. 设备回应

​	d. 芯片发送一个字节数据给设备，并等待回应（每传输一字节数据，需等待回应后再传输下一字节）

​	e. 数据传输完毕，芯片发出结束信号

**读操作：芯片接收来自设备的数据**

​	a. 芯片发出start信号

​	b. 芯片发出设备地址，方向（读/写，1/0）

​	c. 设备回应，并发送数据给芯片，等待回应（每传输一字节数据，需等待回应后再传输下一字节）

​	d. 数据传输完毕，芯片发出结束信号

上拉电阻：

​	当芯片/设备不发送数据时，则不驱动SDA的上拉电阻；芯片/设备驱动SDA的上拉电阻时，输出低电平，否则输出高电平。

​	当芯片/设备需要更多时间来处理数据时，则驱动SCL的上拉电阻，SCL保持低位，只有当SCL从低变高时，IIC总线才能被使用。

#### SMBus概念

SMBus（System Management Bus，系统管理总线）：IIC协议的一个子集；用于连接设备，包括电源相关设备、系统传感器和EEPROM通讯设备；VDD范围：1.8 V ~ 5 V；最小时钟频率：10 KHz，最大Clock Stretching（拉低SCL为设备处理数据提供更多时间）也有限制；设备地址回应（Address Acknowledge）必须发送，以了解设备状态（Busy，Failed，被移除）；明确定义了数据的传输格式；重复发出start信号（REPEATED START Condition）：在读、写之间，可以不发出P信号，直接发出S信号。

```c
/* 1. Symbols, 符号 */
S 		(1 bit): Start bit (开始位)
Sr 		(1 bit): 重复的开始位
P 		(1 bit): Stop bit (停止位)
R/W# 	(1 bit): Read/Write bit. Rd equals 1, Wr equals 0. (读写位)
A, N 	(1 bit): Accept and reverse accept bit. (回应位)
Address (7 bits): I2C 7 bit address. Note that this can be expanded as usual to get a 10 bit I2C address. (地址位, 7位地址)
Command Code (8 bits): Command byte, a data byte which often selects a register on the device. (命令字节, 一般用来选择芯片内部的寄存器)
Data Byte 	(8 bits): A plain data byte. Sometimes, I write DataLow, DataHigh for 16 bit data. (数据字节, 8位; 如果是16位数据的话, 用2个字节来表示: DataLow、DataHigh)
Count 		(8 bits): A data byte containing the length of a block operation. (在block 操作中, 表示数据长度)
[...]: Data sent by I2C device, as opposed to data sent by the host adapter. (中括号表示 I2C 设备发送的数据, 没有中括号表示 host adapter 发送的数据)

/* 2. SMBus Quick Command, 发送设备地址和方向，也用于某些开关设备的开/关 */
Functionality flag: I2C_FUNC_SMBUS_QUICK

/* 3. SMBus Receive Byte, 读取一个字节 */
Functionality flag: I2C_FUNC_SMBUS_READ_BYTE
i2c_smbus_read_byte();

/* 4. SMBus Send Byte, 发送一个字节 */
Functionality flag: I2C_FUNC_SMBUS_WRITE_BYTE
i2c_smbus_write_byte();

/* 5. SMBus Read Byte, 先发送Command code(芯片内部的寄存器地址)，再读取一个字节的数据 */
Functionality flag: I2C_FUNC_SMBUS_READ_BYTE_DATA
i2c_smbus_read_byte_data();

/* 6. SMBus Read Word, 先发送Command code(芯片内部的寄存器地址)，再读取2个字节的数据 */
Functionality flag: I2C_FUNC_SMBUS_READ_WORD_DATA
i2c_smbus_read_word_data();

/* 7. SMBus Write Byte */
Functionality flag: I2C_FUNC_SMBUS_WRITE_BYTE_DATA
i2c_smbus_write_byte_data();

/* 8. SMBus Write Word */
Functionality flag: I2C_FUNC_SMBUS_WRITE_WORD_DATA
i2c_smbus_write_word_data();

/* 9. SMBus Block Read, 先发出Command code，再读到一个字节（表示后续要读的字节数），然后读取全部数据 */
Functionality flag: I2C_FUNC_SMBUS_READ_BLOCK_DATA
i2c_smbus_read_block_data();

/* 10. SMBus Block Write */
Functionality flag: I2C_FUNC_SMBUS_WRITE_BLOCK_DATA
i2c_smbus_write_block_data();

/* 11. IIC Block Read */
Functionality flag: I2C_FUNC_SMBUS_READ_I2C_BLOCK
i2c_smbus_read_i2c_block_data();

/* 12. IIC Block Write */
Functionality flag: I2C_FUNC_SMBUS_WRITE_I2C_BLOCK
i2c_smbus_write_i2c_block_data();

/* 13. SMBus Block Write - Block Read Process Call，先写一块数据，再读一块数据 */
Functionality flag: I2C_FUNC_SMBUS_BLOCK_PROC_CALL

/* 14. Packet Error Checking (PEC)，在结束信号之前，发送方要发出一个字节的PEC码（CRC-8码），用于错误校验 */
```

#### IIC应用编程

重要结构体：

```c
// Linux内核文件：C:\02Myfiles\VMware\Linux-4.9.88\include\linux\i2c.h

/* 1. 结构体：i2c_adapter，表示一个IIC Bus(IIC Controller) 
 * 重要成分：
 * 		a. nr：表示第nr个IIC Bus(IIC Controller); 
 * 		b. i2c_algorithm: 存有IIC Bus的传输函数，用来收发IIC数据
 */
/*
 * i2c_adapter is the structure used to identify a physical i2c bus along
 * with the access algorithms necessary to access it.
 */
struct i2c_adapter {
	struct module *owner;
	unsigned int class;		  /* classes to allow probing for */
	const struct i2c_algorithm *algo; /* the algorithm to access the bus */
	void *algo_data;

	/* data fields that are valid for all devices	*/
	const struct i2c_lock_operations *lock_ops;
	struct rt_mutex bus_lock;
	struct rt_mutex mux_lock;

	int timeout;			/* in jiffies */
	int retries;
	struct device dev;		/* the adapter device */

	int nr;
	char name[48];
	struct completion dev_released;

	struct mutex userspace_clients_lock;
	struct list_head userspace_clients;

	struct i2c_bus_recovery_info *bus_recovery_info;
	const struct i2c_adapter_quirks *quirks;
};

/* 2. 结构体: i2c_algorithm */
struct i2c_algorithm {
	/* If an adapter algorithm can't do I2C-level access, set master_xfer
	   to NULL. If an adapter algorithm can do SMBus access, set
	   smbus_xfer. If set to NULL, the SMBus protocol is simulated
	   using common I2C messages */
	/* master_xfer should return the number of messages successfully
	   processed, or a negative value on error */
	int (*master_xfer)(struct i2c_adapter *adap, struct i2c_msg *msgs,
			   int num);
	int (*smbus_xfer) (struct i2c_adapter *adap, u16 addr,
			   unsigned short flags, char read_write,
			   u8 command, int size, union i2c_smbus_data *data);

	/* To determine what the adapter supports */
	u32 (*functionality) (struct i2c_adapter *);

#if IS_ENABLED(CONFIG_I2C_SLAVE)
	int (*reg_slave)(struct i2c_client *client);
	int (*unreg_slave)(struct i2c_client *client);
#endif
};

/* 3. 结构体: i2c_client，表示一个IIC Device
 * 重要成分：
 * 		a. addr: 设备地址
 * 		b. adapter: 连接的IIC Controller
 */
struct i2c_client {
	unsigned short flags;		/* div., see below		*/
	unsigned short addr;		/* chip address - NOTE: 7bit	*/
					/* addresses are stored in the	*/
					/* _LOWER_ 7 bits		*/
	char name[I2C_NAME_SIZE];
	struct i2c_adapter *adapter;	/* the adapter we sit on	*/
	struct device dev;		/* the device structure		*/
	int irq;			/* irq issued by device		*/
	struct list_head detected;
#if IS_ENABLED(CONFIG_I2C_SLAVE)
	i2c_slave_cb_t slave_cb;	/* callback for slave mode	*/
#endif
};

/* 4. 结构体: i2c_msg，表示要传输的数据 */
// 定义呢？？为什么不在i2c.h里？？

/* 5. 函数: i2c_transfer，APP通过i2c_adapter与i2c_client传输i2c_msg，因为i2c_msg中含有addr，所以函数不需要i2c_client */
```

访问IIC设备：APP通过Linux自带的i2c-dev.c（IIC Device Driver）驱动adapter driver（IIC Controller Driver），再驱动IIC设备。

```c
/* 使用I2C-Tools */

/* 1. 列出当前的I2C adapter（I2C Bus, I2C Controller） */
i2cdetect -l

/* 2. 打印第"I2CBUS"个I2C adapter的Functionalities，其中I2CBUS = 0, 1, 2, ... */
i2cdetect -F I2CBUS

/* 3. 打印I2C设备 */
i2cdetect -y -a I2CBUS

/* 4. 读取某地址上的1 or 2个字节
 * I2CBUS 为 0、1、2 等整数
 * CHIP-ADDRESS 表示设备地址
 * DATA-ADDRESS: 芯片上寄存器地址
 * MODE：有 2 个取值, 
 * 	b-使用`SMBus Read Byte`先发出 DATA-ADDRESS, 再读一个字节, 中间无P信号；
 * 	c-先 write byte, 再 read byte，中间有 P 信号
 * 	w-表示先发出 DATA-ADDRESS，再读 2 个字节
 */
i2cget -f -y I2CBUS CHIP-ADDRESS DATA-ADDRESS MODE
// or 读一个字节:
i2cget -f -y I2CBUS CHIP-ADDRESS

/* 5. 在某地址上写字节
 * I2CBUS 为 0、1、2 等整数
 * CHIP-ADDRESS 表示设备地址
 * DATA-ADDRESS: 8 位芯片寄存器地址
 * VALUE: N个8 or 16位数值
 * MODE: 
 * 	b-VALUE为1个字节（8 bits），默认为b
 * 	w-VALUE为2字节（16 bits）
 * 	s-SMBus block data, VALUE为N个8 bits
 * 	i-I2CBus block data, VALUE为N个8 bits
 */
i2cset [-f] [-y] [-m MASK] [-r] [-a] I2CBUS CHIP-ADDRESS DATA-ADDRESS [VALUE] ... [MODE]

/* 6.I2C传输（不是基于SMBus）
 * I2CBUS 为 0、1、2 等整数
 * DESC 为传输形式: {r|w}LENGTH[@address]
 * 	1) read/write-flag 2) LENGTH (range 0-65535) 3) I2C address (use last one if omitted)
 * DATA 为Length个字节的写入数据，可通过后缀简写：
 * 	= (keep value constant until LENGTH)
 * 	+ (increase value by 1 until LENGTH)
 * 	- (decrease value by 1 until LENGTH)
 * 	p (use pseudo random generator until LENGTH with value as seed)
 */
i2ctransfer [-f] [-y] [-v] [-V] [-a] I2CBUS DESC [DATA] [DESC [DATA]]...
```

I2C应用：

① 怎么指定 I2C 控制器?

- i2c-dev.c 为每个 I2C 控制器 (I2C Bus、I2C Adapter) 都生成一个设备节点: /dev/i2c-0、/dev/i2c-1 等等;
- open 某个 /dev/i2c-X 节点, 就是去访问该 I2C 控制器下的设备;

② 怎么指定 I2C 设备?

通过 ioctl 指定 I2C 设备的地址

- ioctl(file, I2C_SLAVE, address)
  - 如果该设备已经有了对应的设备驱动程序, 则返回失败。
- ioctl(file, I2C_SLAVE_FORCE, address)
  - 如果该设备已经有了对应的设备驱动程序但是还是想通过 i2c-dev 驱动来访问它, 则使用这个 ioctl 来指定 I2C 设备地址。

③ 怎么传输数据?

两种方式

- 一般的 I2C 方式: ioctl(file, I2C_RDWR, &rdwr)
- SMBus 方式: ioctl(file, I2C_SMBUS, &args)

I2C-Tools源码：

I2C方式：i2ctransfer.c

SMBus方式：i2cget.c, i2cset.c

### Linux串口应用编程

#### 串口API概念

```c
/* 1. UART串口编程步骤：???
 * 	a. open;
 * 	b. 设置行规程，比如波特率、数据位、停止位、检验位、RAW 模式、一有数据就返回
 * 	c. read/write;
 */

/* 2. 行规程参数结构体: termios
 * 	tc: terminal control
 *	cf: control flag
 *	文件目录：？？？
 */
typedef unsigned char  cc_t;
typedef unsigned int  speed_t;
typedef unsigned int  tcflag_t;

#define NCCS 19
struct termios {
    tcflag_t c_iflag;             /* input mode flags */
    tcflag_t c_oflag;             /* output mode flags */
    tcflag_t c_cflag;             /* control mode flags */
    tcflag_t c_lflag;             /* local mode flags */
    cc_t c_line;                  /* line discipline */
    cc_t c_cc[NCCS];             /* control characters */
};

/* 3. 行规程函数 */
tcgetattr      get terminal attributes, 获得终端的属性
tcsetattr      set terminal attributes, 修改终端参数
tcflush        清空终端未完成的输入/输出请求及数据
cfsetispeed    sets the input baud rate, 设置输入波特率
cfsetospeed    sets the output baud rate, 设置输出波特率
cfsetspeed     同时设置输入、输出波特率

```

#### 串口收发实验

GPS模块实验。。。



# 嵌入式Linux驱动开发

## Linux驱动概念

### 文件在内核中的表示

```c
// 文件目录：D:\Users\22499\Desktop\嵌入式学习\02_100ask_imx6ull_pro_2022.08\09_myLinux源码\Linux-4.9.88\include\linux\fs.h
/* 1. 打开一个文件，会得到相应的文件句柄（一个整数）以及文件结构体struct file */
int open(const char *pathname, int flags, mode_t mode);

/* 2. 文件结构体：struct file */
struct file {
	union {
		struct llist_node	fu_llist;
		struct rcu_head 	fu_rcuhead;
	} f_u;
	struct path		f_path;
	struct inode		*f_inode;	/* cached value */
	const struct file_operations	*f_op;

	/*
	 * Protects f_ep_links, f_flags.
	 * Must not be taken from IRQ context.
	 */
	spinlock_t		f_lock;
	atomic_long_t		f_count;
	unsigned int 		f_flags;
	fmode_t			f_mode;
	struct mutex		f_pos_lock;
	loff_t			f_pos;
	struct fown_struct	f_owner;
	const struct cred	*f_cred;
	struct file_ra_state	f_ra;

	u64			f_version;
#ifdef CONFIG_SECURITY
	void			*f_security;
#endif
	/* needed for tty driver, and maybe others */
	void			*private_data;

#ifdef CONFIG_EPOLL
	/* Used by fs/eventpoll.c to link all the hooks to this file */
	struct list_head	f_ep_links;
	struct list_head	f_tfile_llink;
#endif /* #ifdef CONFIG_EPOLL */
	struct address_space	*f_mapping;
} __attribute__((aligned(4)));	/* lest something weird decides that 2 is OK */

/* 3. 文件结构体中包含：由驱动程序提供的结构体struct file_operation *f_op */
const struct file_operations	*f_op;

/* 4. 编写驱动程序步骤：
 *   驱动程序初始化：
 *  	a. 确定主设备号, 也可以让内核分配
 *  	b. 定义自己的 file_operations 结构体
 *  	c. 实现对应的 drv_open/drv_read/drv_write 等函数, 填入 file_operations 结构体
 *  	d. 把 file_operations 结构体告诉内核: register_chrdev
 *  e. 谁来注册驱动程序啊? 得有一个入口函数: 安装驱动程序时, 就会去调用这个入口函数
 *  f. 有入口函数就应该有出口函数: 卸载驱动程序时, 出口函数调用 unregister_chrdev
 *  e. 其他完善: 提供设备信息, 自动创建设备节点: class_create, device_create
 */

```

### 编写字符设备驱动程序

```c
/* 编写驱动程序 */
/* 1. 驱动程序的初始化函数 __init: 仅在初始化期间需要，之后可以被丢弃以节省内存 */
static int __init hello_init(void);
/* -a. 注册设备 register_chrdev: 主设备号，设备名称，指向定义驱动程序功能的结构体file_operations指针，输入参数如下：
 *   "0": 请求内核动态分配主设备号major
 *   "hello": 设备名称
 *   "&hello_drv": 指向定义驱动程序功能的结构体file_operations指针
 */
major = register_chrdev(0, "hello", &hello_drv);
/* -b. 创建设备类 class_create: 设备类用于组织/sys文件系统中的设备，包括指向当前模块的指针，类名称 */
/* 目录: /sys/class/hello_class/ */
hello_class = class_create(THIS_MODULE, "hello_class");
/* -b. 检查设备类是否创建成功 */
err = PTR_ERR(hello_class);
if (IS_ERR(hello_class)) {
	printk("%s %s line %d\n", __FILE__, __FUNCTION__, __LINE__);
	unregister_chrdev(major, "hello");
	return -1;
}
/* -c. 创建设备文件 device_create: 在/dev目标创建设备文件，输入参数如下：
 *   "hello_class": 设备类
 *   "NULL": 没有父设备
 *   "MKDEV(major, 0)": 设备号，包括主设备号和次设备号
 *   "NULL": 驱动程序数据
 *   "hello": 设备文件名称
 */
/* 目录: /devices/virtual/hello_class/hello/ */
device_create(hello_class, NULL, MKDEV(major, 0), NULL, "hello");

/* 2. 各种处理设备文件操作函数定义 */
/* -a. 读操作函数的输入参数如下：
 *   struct file* file: 指向与打开的文件相关联的结构文件的指针
 *   char __user* buf: 指向用户空间缓冲区的指针
 *   size_t size: 用户空间缓冲区的大小
 *   loff_t* offset: 文件中的当前读取偏移量
 */
static ssize_t hello_drv_read(struct file* file, char __user* buf, size_t size, loff_t* offset);
static ssize_t hello_drv_write(struct file* file, const char __user* buf, size_t size, loff_t* offset);
static int hello_drv_open(struct inode* node, struct file* file);
static int hello_drv_close(struct inode* node, struct file* file);
/* -b. 内核函数copy_to_user: 将数据从内核缓冲区(kernel_buf)复制到用户空间缓冲区(buf)，最多复制字节数MIN(1024, size) */
err = copy_to_user(buf, kernel_buf, MIN(1024, size));
err = copy_from_user(kernel_buf, buf, MIN(1024, size));

/* 3. 驱动程序的出口函数 __exit: 仅在模块移除期间使用 */
static void __exit hello_exit(void) {
    printk("%s %s line %d\n", __FILE__, __FUNCTION__, __LINE__);
	device_destroy(hello_class, MKDEV(major, 0));
	class_destroy(hello_class);
    unregister_chrdev(major, "hello");
}

/* 4. 模块初始化，清理与许可 */
/* -a. 模块初始化: 告诉内核当模块加载到内核中时调用哪个函数。 */
module_init(hello_init);
/* -b. 模块出口:  */
module_exit(hello_exit);
/* -c. 模块许可证: 许可证是GNU通用公共许可证(GPL)。 */
MODULE_LICENSE("GPL");
```

### 编写测试程序

```c
/* 编写测试程序步骤及源码 */
/* 1. 判断输入参数及提供输入提示 */
if (argc < 2) 
{
	printf("Usage: %s -w <string>\n", argv[0]);
	printf("       %s -r\n", argv[0]);
	return -1;
}
/* Shell界面输入 */
./hello_drv_test -w www.100ask.net  // 把字符串“www.100ask.net”发给驱动程序
./hello_drv_test -r  // 把驱动中保存的字符串读回来

/* 2. 打开文件 */
fd = open("/dev/hello", O_RDWR);
if (fd == -1)
{
	printf("can not open file /dev/hello\n");
	return -1;
}

/* 3. 读/写文件 */
if ((0 == strcmp(argv[1], "-w")) && (argc == 3))
{
	len = strlen(argv[2]) + 1;
	len = len < 1024 ? len : 1024;
	write(fd, argv[2], len);
}
else
{
	len = read(fd, buf, 1024);		
	buf[1023] = '\0';
	printf("APP read : %s\n", buf);
}

/* 4. 关闭文件 */
close(fd);
```

### Makefile文件编译

```makefile

# 1. 使用不同的开发板内核时, 一定要修改KERN_DIR
# 2. KERN_DIR中的内核要事先配置、编译, 为了能编译内核, 要先设置下列环境变量:
# 2.1 ARCH,          比如: export ARCH=arm64
# 2.2 CROSS_COMPILE, 比如: export CROSS_COMPILE=aarch64-linux-gnu-
# 2.3 PATH,          比如: export PATH=$PATH:/home/book/100ask_roc-rk3399-pc/ToolChain-6.3.1/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin 
# 注意: 不同的开发板不同的编译器上述3个环境变量不一定相同,
#       请参考各开发板的高级用户使用手册

KERN_DIR = /home/book/100ask_roc-rk3399-pc/linux-4.4

all:
	make -C $(KERN_DIR) M=`pwd` modules 
	$(CROSS_COMPILE)gcc -o hello_drv_test hello_drv_test.c 

clean:
	make -C $(KERN_DIR) M=`pwd` modules clean
	rm -rf modules.order
	rm -f hello_drv_test

obj-m	+= hello_drv.o

```

### 补充知识。。

```c
/* 1. 驱动程序中入口函数module_init(hello_init)和出口函数module_exit(hello_exit)的宏展开 */
/* -a. 在内核文件include\linux\module.h中的宏展开形式 */
#ifndef MODULE
#define module_init(x)	__initcall(x);
#define module_exit(x)	__exitcall(x);

#else /* MODULE */

/* Each module must use one module_init(). */
#define module_init(initfn)					\
	static inline initcall_t __inittest(void)		\
	{ return initfn; }					\
	int init_module(void) __attribute__((alias(#initfn)));

/* This is only required if you want to be unloadable. */
#define module_exit(exitfn)					\
	static inline exitcall_t __exittest(void)		\
	{ return exitfn; }					\
	void cleanup_module(void) __attribute__((alias(#exitfn)));

#endif

/* -b. 驱动程序被编译进内核中，宏展开 */
/* 内核启动时，会去段".initcall6.init"里取出这些函数指针来执行，所以入口函数hello_init就被执行了。
一个驱动被编进内核后，它是不会被卸载的，所以段“.exitcall.exit”不会被用到，内核启动后会释放这块段空间。
 */
static initcall_t __initcall_hello_init6 __used \
__attribute__((__section__(".initcall6.init"))) = hello_init;
static exitcall_t __exitcall_hello_exit __used \
    __section(.exitcall.exit) = hello_exit;

/* -c. 驱动程序被编译为ko文件中，宏展开 */
/* 分别定义了 2 个函数：第 1 个函数名为 init_module，它是 hello_init 函数的别名；第 2 个函数名为 cleanup_module，它是 hello_exit 函数的别名。
以后我们使用 insmod 命令加载驱动时，内核都是调用 init_module 函数，实际上就是调用 hello_init 函数；使用 rmmod 命令卸载驱动时，内核都是调用cleanup_module 函数，实际上就是调用 hello_exit 函数。
 */
static inline initcall_t __inittest(void) \
{ return hello_init; } \
int init_module(void) __attribute__((alias("hello_init")));
static inline exitcall_t __exittest(void) \
{ return hello_exit; } \
void cleanup_module(void) __attribute__((alias("hello_exit")));

/* 2. chrdev数组：链表结构，next指向下一个元素，每个元素指定了主设备号major、次设备号baseminor、个数minorct、cdev包含file_operations结构体 */
/* (major, baseminor)到(major, baseminor+minorct-1)的设备们都用同一个file_operations。 */
static struct char_device_struct {
    struct char_device_struct *next;
	unsigned int major;
	unsigned int baseminor;
    int minorct;
    char name[64];
    struct cdev *cdev; /* will die */
} *chrdevs[CHRDEV_MAJOR_HASH_SIZE];
/* -a. 通过函数major_to_index确定数组下标 */
static inline int major_to_index(unsigned major) {
    return major % CHRDEV_MAJOR_HASH_SIZE;
}
/* -b. register_chrdev函数源码（精简后） */
static inline int register_chrdev(unsigned int major, const char *name,
 const struct file_operations *fops) {
	return __register_chrdev(major, 0, 256, name, fops);
}
int _register_chrdev(unsigned int major, unsigned int baseminor,
                    unsigned int count, const char *name,
                    const struct file_operations *fops) {
    struct char_device_struct *cd;
    struct cdev *cdev;
    int err = -ENOMEM;

    cd = _register_chrdev_region(major, baseminor, count, name);

    cdev = cdev_alloc();

    cdev->owner = fops->owner;
    cdev->ops = fops;

    kobject_set_name(&cdev->kobj, "%s", name);
	/* 注册cdev结构体到内核 */
    err = cdev_add(cdev, MKDEV(cd->major, baseminor), count);
}
/* -c. cdev函数源码：注册cdev结构体到内核 */
/* 这个函数涉及 kobj 的操作，这是一个通用的链表操作函数。它的作用是：把 cdev 结构体放入 cdev_map 链表中，对应的索引值是“dev”到“dev+count1”。以后可以从 cdev_map 链表中快速地使用索引值取出对应的 cdev。 */
/* err = cdev_add(cdev, MKDEV(1, 2), 10); 其中的 MKDEV(1,2)构造出一个整数“1<<8 | 2”，即 0x102；上述代码将cdev 放入 cdev_map 链表中，对应的索引值是 0x102 到 0x10c(即 0x102+10)。以后根据这 10 个数值(0x102、0x103、0x104、……、0x10c)中任意一个，都可以快速地从 cdev_map 链表中取出 cdev 结构体。 */
/* APP 打开某个字符设备节点时，进入内核。在内核里根据字符设备节点的主、次设备号，计算出一个数值(major<<8 | minor，即 inode->i_rdev)，然后使用这个数值从 cdev_map 中快速得到 cdev，再从 cdev 中得到 file_operations结构体。 */
int cdev_add(struct cdev *p, dev_t dev, unsigned count)
{
    int error;

    p->dev = dev;
    p->count = count;

    error = kobj_map(cdev_map, dev, count, NULL,
                     exact_match, exact_lock, p);

    if (error)
        return error;

    kobject_get(p->kobj.parent);
    
    return 0;
}
```

## GPIO概念

引脚复用功能以及引脚属性配置

GPIO使能：CCM_CCGRx

输出方向：GPIOx_GDIR；输出电平：GPIOx_DR

## LED驱动程序

### 驱动程序框架

```c
/* 包括驱动程序led_drv.c，具体硬件操作board_led.c，头文件（函数声明）led_opr.h */
/* 1. 依赖于硬件的细节在驱动程序中被抽象出来：把LED操作抽象为led_operations结构体 */
/* 注意：board_demo.c只是示例程序，没有真正操作硬件 */
struct led_operations {
	int (*init) (int which);
	int (*ctl) (int which, char status);
};
/* 函数声明 */
struct led_operations* get_board_led_opr(void);
/* 函数定义 */
struct led_operations* get_board_led_opr(void)
{
	return &board_demo_led_opr;
}
/* LED操作细节 */
static int board_demo_led_init(int which)
{
	printk("%s %s line %d, led %d\n", __FILE__, __FUNCTION__, __LINE__, which);
	return 0;
}

static int board_demo_led_ctl(int which, char status)
{
	printk("%s %s line %d, led %d, %s\n", __FILE__, __FUNCTION__, __LINE__, which, status ? "on" : "off");
	return 0;
}

static struct led_operations board_demo_led_opr = {
	.init = board_demo_led_init,
	.ctl = board_demo_led_ctl,
};

/* 2. 驱动程序框架：__init, __exit, file_operations(调用具体硬件操作p_led_opr = get_board_led_opr(); p_led_opr->init(minor); p_led_opr->ctl(minor, status)) */
/* 注册驱动：file_operations */
```

### 具体硬件操作

```c
/* 1. ioremap函数的使用 */
/* -a. 芯片手册中的寄存器地址为物理地址，需要通过内核函数ioremap映射为虚拟地址再使用 */
/* -b. 函数原型 */
#include <asm/io.h>
void __iomem *ioremap(resource_size_t res_cookie, size_t size);
/* -c. 函数使用 */
/* 把物理地址 phys_addr 开始的一段空间(大小为 size)，映射为虚拟地址；返回值是该段虚拟地址的首地址。 */
/* 按页（4096字节）取整：
 *   假设 phys_addr = 0x10002，size=4，ioremap 的内部实现是：
 *     phys_addr 按页取整，得到地址 0x10000
 *     size 按页取整，得到 4096
 */
virt_addr = ioremap(phys_addr, size);
/* -d. 取消虚拟映射 */
void iounmap(volatile void __iomem *cookie);

/* 2. 具体硬件操作：board_led.c */
static volatile unsigned int *CCM_CCGR1                              ;
static volatile unsigned int *IOMUXC_SNVS_SW_MUX_CTL_PAD_SNVS_TAMPER3;
static volatile unsigned int *GPIO5_GDIR                             ;
static volatile unsigned int *GPIO5_DR                               ;

static int board_demo_led_init (int which) /* 初始化LED, which-哪个LED */       
{
    unsigned int val;

    //printk("%s %s line %d, led %d\n", __FILE__, __FUNCTION__, __LINE__, which);
    if (which == 0)
    {
        if (!CCM_CCGR1)
        {
            CCM_CCGR1                               = ioremap(0x20C406C, 4);
            IOMUXC_SNVS_SW_MUX_CTL_PAD_SNVS_TAMPER3 = ioremap(0x2290014, 4);
            GPIO5_GDIR                              = ioremap(0x020AC000 + 0x4, 4);
            GPIO5_DR                                = ioremap(0x020AC000 + 0, 4);
        }
        
        /* GPIO5_IO03 */
        /* a. 使能GPIO5
         * set CCM to enable GPIO5
         * CCM_CCGR1[CG15] 0x20C406C
         * bit[31:30] = 0b11
         */
        *CCM_CCGR1 |= (3<<30);
        
        /* b. 设置GPIO5_IO03用于GPIO
         * set IOMUXC_SNVS_SW_MUX_CTL_PAD_SNVS_TAMPER3
         *      to configure GPIO5_IO03 as GPIO
         * IOMUXC_SNVS_SW_MUX_CTL_PAD_SNVS_TAMPER3  0x2290014
         * bit[3:0] = 0b0101 alt5
         */
        val = *IOMUXC_SNVS_SW_MUX_CTL_PAD_SNVS_TAMPER3;
        val &= ~(0xf);
        val |= (5);
        *IOMUXC_SNVS_SW_MUX_CTL_PAD_SNVS_TAMPER3 = val;
        
        
        /* b. 设置GPIO5_IO03作为output引脚
         * set GPIO5_GDIR to configure GPIO5_IO03 as output
         * GPIO5_GDIR  0x020AC000 + 0x4
         * bit[3] = 0b1
         */
        *GPIO5_GDIR |= (1<<3);
    }
    
    return 0;
}

static int board_demo_led_ctl (int which, char status) /* 控制LED, which-哪个LED, status:1-亮,0-灭 */
{
    //printk("%s %s line %d, led %d, %s\n", __FILE__, __FUNCTION__, __LINE__, which, status ? "on" : "off");
    if (which == 0)
    {
        if (status) /* on: output 0*/
        {
            /* d. 设置GPIO5_DR输出低电平
             * set GPIO5_DR to configure GPIO5_IO03 output 0
             * GPIO5_DR 0x020AC000 + 0
             * bit[3] = 0b0
             */
            *GPIO5_DR &= ~(1<<3);
        }
        else  /* off: output 1*/
        {
            /* e. 设置GPIO5_IO3输出高电平
             * set GPIO5_DR to configure GPIO5_IO03 output 1
             * GPIO5_DR 0x020AC000 + 0
             * bit[3] = 0b1
             */ 
            *GPIO5_DR |= (1<<3);
        }
    
    }
    return 0;
}

static struct led_operations board_demo_led_opr = {
    .num  = 1,
    .init = board_demo_led_init,
    .ctl  = board_demo_led_ctl,
};

struct led_operations *get_board_led_opr(void)
{
    return &board_demo_led_opr;
}

/* 3. 上机实验：禁止内核中的原LED驱动"heatbeat" */
[root@100ask:~]#echo none > /sys/class/leds/cpu/trigger
```

## 驱动设计思想

```c
/* 驱动设计思想 */
/* 1. 面向对象 */
/* -a. 驱动程序框架中抽象出file_operations结构体 */
/* -b. 具体硬件操作中抽象出led_operations结构体 */

/* 2. 分层 */
/* -a. 上层实现硬件无关操作，即注册驱动 */
/* -b. 下层实现硬件相关操作，即实现寄存器操作 */

/* 3. 分离 */
/* -a. 考虑到引脚操作的规律性，则针对某芯片实现通用的硬件操作代码，适用于所有GPIO引脚。 */
/* -b. 使用时指明对应引脚即可 */
```

示例代码

```c
// 源码目录：D:\Users\22499\Desktop\嵌入式学习\01_all_series_quickstart\05_嵌入式Linux驱动开发基础知识\source\02_led_drv\03_led_drv_template_seperate
/* 程序仍分为上下结构：上层 leddrv.c 向内核注册 file_operations 结构体；下层 chip_demo_gpio.c 提供 led_operations 结构体来操作硬件。 */
/* 下层的代码分为 2 个：chip_demo_gpio.c 实现通用的 GPIO 操作，board_A_led.c 指定使用哪个 GPIO，即“资源”。 */
```

## 总线设备驱动模型

```c
/* 1. 注册platform_device结构体 */
/* -a. 初始化platform_device结构体 */
/* 包含两个引脚 */
static struct platform_device board_A_led_dev = {
        .name = "100ask_led",
        .num_resources = ARRAY_SIZE(resources),  // 资源数（resources阵元数）
        .resource = resources,
        .dev = {
                .release = led_dev_release,
         },
};

static struct resource resources[] = {  // 与设备相关的硬件资源
        {
                .start = GROUP_PIN(3,1),  // 起始地址
                .flags = IORESOURCE_IRQ,  // 标记为IRQ中断
                .name = "100ask_led_pin",
        },
        {
                .start = GROUP_PIN(5,8),
                .flags = IORESOURCE_IRQ,
                .name = "100ask_led_pin",
        },
};

static void led_dev_release(struct device *dev) {
}

/* -b. 在入口函数__init led_dev_init里注册platform_device结构体 */
static int __init led_dev_init(void)
{
    int err;
    /* 向内核注册平台设备board_A_led_dev */
    err = platform_device_register(&board_A_led_dev);   
    
    return 0;
}

/* 2. 注册platform_driver结构体（资源） */
/* 在嵌入式Linux中，平台驱动程序（platform drivers）是管理与硬件平台紧密集成的设备的常用方法，例如与处理器在同一芯片上的设备。这些驱动程序与内核交互来处理设备发现、资源分配和通信。 
 * 在内核注册platform_driver，识别符合驱动程序标准的设备。当找到匹配的设备时，调用驱动程序的探测函数(本例中为chip_demo_gpio_probe)。允许驱动程序管理设备及其资源。
 * 内核将识别新的驱动程序，并将其与匹配其名称字段的任何设备相关联(例如，“100ask_led”)。然后将调用驱动程序的探测函数，允许它初始化led并设置任何必要的数据结构。
 */
/* -a. 初始化platform_driver结构体 */
/* --.probe: 当匹配设备(device)找到时调用
 * --.remove: 当删除设备时调用
 * --.name: 驱动名称，用于匹配设备
 */
static struct platform_driver chip_demo_gpio_driver = {
    .probe      = chip_demo_gpio_probe,
    .remove     = chip_demo_gpio_remove,
    .driver     = {
        .name   = "100ask_led",
    },
};

static int chip_demo_gpio_probe(struct platform_device *pdev)
{
    struct resource *res;
    int i = 0;

    while (1)
    {
        /* 检索与设备相关的硬件资源（例如引脚pin信息） */
        res = platform_get_resource(pdev, IORESOURCE_IRQ, i++);
        if (!res)
            break;
        /* 保存硬件资源到g_ledpins数组(int数组)，并创建设备节点(device nodes)用于LEDs */
        g_ledpins[g_ledcnt] = res->start;
        led_class_create_device(g_ledcnt);
        g_ledcnt++;
    }
    return 0;
    
}

static int chip_demo_gpio_remove(struct platform_device *pdev)
{
    struct resource *res;
    int i = 0;

    while (1)
    {
        res = platform_get_resource(pdev, IORESOURCE_IRQ, i);
        if (!res)
            break;
        /* 删除probe期间创建的设备节点 */
        led_class_destroy_device(i);
        i++;
        g_ledcnt--;
    }
    return 0;
}

/* -b. 在入口函数__init chip_demo_gpio_drv_init里注册platform_driver结构体 */
err = platform_driver_register(&chip_demo_gpio_driver);

/* 3. 注册led_operations结构体（硬件操作） */
/* -a. 初始化led_operations结构体 */
static struct led_operations board_demo_led_opr = {
    .init = board_demo_led_init,
    .ctl  = board_demo_led_ctl,
};

static int board_demo_led_init (int which) {  
}

static int board_demo_led_ctl (int which, char status) {
}

/* -b. 在入口函数__init chip_demo_gpio_drv_init里注册led_operations结构体 */
register_led_operations(&board_demo_led_opr);
```

## 设备树

### 设备树文件格式

```c
/* 设备树源（Device tree source）文件格式 */
/* 1. DTS文件布局 */
/dts-v1/;  // 表示版本
[memory reservations]  // 格式为: /memreserve/ <address> <length>;
/ {
 [property definitions]
 [child nodes]
};

/* 2. 基本单元node格式 */
[label:] node-name[@unit-address] {  // label可以省略
 [properties definitions]
 [child nodes]
};

/* 3. properties格式 */
[label:] property_name = value;  // 赋值
[label:] property_name;  // 不赋值
/* 赋值类型：
 * -arrays of cells(1 个或多个 32 位数据, 64 位数据使用 2 个 32 位数据表示),
 * -- interrupts = <17 0xc>;
 * -- clock-frequency = <0x00000001 0x00000000>;  // 64位
 * -string(字符串),
 * -- compatible = "simple-bus";  // 字符串
 * -bytestring(1 个或多个字节)
 * -- local-mac-address = [00 00 12 34 56 78]; // 每个 byte 使用 2 个 16 进制数来表示
 * -各种类型的组合
 * -- example = <0xf00f0000 19>, "a strange property format";
 */

/* 4. 芯片设备树模板.dtsi文件 */
/* 目录: arch/arm/boot/dts; .dtsi: i表示"include"，被其他文件引用 */

/* 5. 常用属性 */
/* -a. 起始地址和大小：#address-cells、#size-cells
 * --. cell 指一个 32 位的数值，
 * --. address-cells：address 要用多少个 32 位数来表示,
 * --. size-cells：size 要用多少个 32 位数来表示。
 */
/ {
#address-cells = <1>;
#size-cells = <1>;
memory {
reg = <0x80000000 0x20000000>;
	};
};

/* -b. 兼容性(支持硬件的多个驱动)：compatible
 * --. compatible 的值，建议取这样的形式："manufacturer,model"，即“厂家名，模块名”。
 * --. 根节点下也有 compatible 属性，用来选择哪一个“machine desc”(机器描述)：一个内核可以支持 machine A，也支持 machine B，内核启动后会根据根节点的compatible 属性找到对应的 machine desc 结构体，执行其中的初始化函数。
 */
led {
compatible = “A”, “B”, “C”;
};

/* -c. 硬件名称：model */
{
compatible = "samsung,smdk2440", "samsung,mini2440";
model = "jz2440_v3";
};

/* -d. 设备节点属性：status
 * --. "okey": 设备正常运行
 * --. "disabled": 设备不可操作，但之后可以恢复工作
 * --. "fail": 发生了错误，需要修复
 * --. "fail-sss": 发生了错误，需要修复；sss表示错误信息。
 */
&uart1 {
 status = "disabled";
};

/* -e. 寄存器or内存的地址空间：reg */
/dts-v1/;
/ {
#address-cells = <1>;
#size-cells = <1>;
memory {
reg = <0x80000000 0x20000000>;
};
};

/* 6. 常用节点node */
/* -a. 根节点：dts文件必须有一个根节点；根节点必须包含"model", "compatible", "#address-cells", "size-cells" */
/dts-v1/;  // 版本
/ {
	model = "SMDK24440";  // 板子名称
	compatible = "samsung,smdk2440";// 板子兼容的"machine desc"(机器描述，平台)
    							// uImage : smdk2410 smdk2440 mini2440 ==> machine_desc
	#address-cells = <1>;  // 使用多少个u32整数来表示地址
	#size-cells = <1>;  // 使用多少个u32整数来表示大小
};

/* -b. CPU节点：在dtsi文件中定义好了，#include<.dtsi>即可 */
cpus {
	#address-cells = <1>;
	#size-cells = <0>;
	cpu0: cpu@0 {
		 .......
 	}
};

/* -c. memory节点：厂商设置 */
memory {
	reg = <0x80000000 0x20000000>;
};

/* -d. chosen节点：通过设备树文件向内核传递参数 */
chosen {
	bootargs = "noinitrd root=/dev/mtdblock4 rw init=/linuxrc console=ttySAC0,115200";
};

```

### 编译、更换设备树

```c
/* 内核对设备树的处理过程 */
/* 1. dts文件编译为dtb文件 */
/* -a. 设置 ARCH、CROSS_COMPILE、PATH 这三个环境变量后，在内核源码目录中执行 */
make dtbs;

/* 2. u-boot启动文件把dtb文件传给内核 */

/* 3. 内核解析 dtb 文件，把每一个节点都转换为 device_node 结构体 */
/* -a. 根节点被保存在全局变量 of_root 中，从 of_root 开始可以访问到任意节点。 */

/* 4. 对于某些 device_node 结构体，会被转换为 platform_device 结构体 */
/* -a. 转换为platform_device的设备树节点：
 * --. 根节点下含有 compatile 属性的子节点
 * --. 含有特定 compatile 属性的节点的子节点，特定compatile: "simple-bus", "simple-mfd", "isa", "arm, amba-bus"
 * --. 总线 I2C、SPI 节点下的子节点：不转换为 platform_device，交给对应的总线驱动程序来处理，但/i2c节点（i2c控制器）和/spi节点（SPI控制器）会转换为platform_device.
 */

/* -b. platform_device从device_node中获得资源（resources）
 * --. platform_device 中含有 resource 数组, 它来自 device_node 的 reg, interrupts 属性;
 * --. platform_device.dev.of_node 指向 device_node, 可以通过它获得其他属性
 */

/* 查看设备树 */
/* --. /sys/firmware/devicetree 目录下是以目录结构程现的 dtb 文件, 根节点对应 base 目录, 每一个节点对应一个目录, 每一个属性对应一个文件。
 * --. /sys/firmware/fdt 文件，它就是 dtb 格式的设备树文件，可以把它复制出来放到 ubuntu 上，执行下面的命令反编译出来(-I dtb：输入格式是 dtb，-O dts：输出格式是 dts)
 cd 板子所用的内核源码目录
./scripts/dtc/dtc -I dtb -O dts /从板子上/复制出来的/fdt -o tmp.dts
 */
# ls /sys/firmware/
devicetree fdt;
```

### 设备与驱动配对

```c
/* platform_device和platform_driver配对 */
/* 1. 配对过程 */
/* 从设备树转换得来的 platform_device 会被注册进内核里，以后每注册一个 platform_driver 时，它们就会两两确定能否配对，如果能配对成功就调用 platform_driver 的 probe 函数。 */

/* -a. 设备与驱动的匹配规则源码 */
/* 目录: ... */
static int platform_match(struct device *dev, struct device_driver *drv)
{
    struct platform_device *pdev = to_platform_device(dev);
    struct platform_driver *pdrv = to_platform_driver(drv);

    /* --. 最先比较：比较platform_device.driver_override和platform_driver.driver.name，相同则强制选择该platform_driver */
    if (pdev->driver_override)
        return !strcmp(pdev->driver_override, drv->name);

    /* --. 然后比较：比较platform_device.dev.of_node和platform_driver.of_match_table */
    /* ----. platform_device结构体
     * struct device_node {
    		const char *name; // 来自节点的name属性
    		const char *type; // 来自节点的device_type属性
    		phandle phandle;
    		const char *full_name;
    		struct fwnode_handle fwnode;
    		struct property *properties; // 含有compatible属性
		};
     */
    /* ----. platform_driver.driver.of_match_table数组类型（of_device_id结构体）
     * struct of_device_id {
    		char name[32];
    		char type[32];
    		char compatible[128];
    		const void *data;
    	};
     */
    /* ----. dev和drv配对过程：
     * 1） 如果 of_match_table 中含有 compatible 值，就跟 dev 的 compatile属性比较，若一致则成功，否则返回失败；
     * 2） 如果 of_match_table 中含有 type 值，就跟 dev 的 device_type 属性比较，若一致则成功，否则返回失败；（不常用）
     * 3） 如果 of_match_table 中含有 name 值，就跟 dev 的 name 属性比较，若一致则成功，否则返回失败。（不常用）
     */
    if (of_driver_match_device(dev, drv))
        return 1;

    /* --. 接着比较：比较platform_device.name和platform_driver.id_table[i].name */
    /* ----. platform_driver.id_table 是“platform_device_id”指针，表示该 drv 支持若干 device，它里面列出了各个 device 的{.name, .driver_data}，其中的“name”表示该 drv 支持的设备的名字，driver_data 是些提供给该 device 的私有数据。
 */
    if (acpi_driver_match_device(dev, drv))
        return 1;

    /* --. 最后比较：比较platform_device.name和platform_driver.driver.name */
    if (pdrv->id_table)
        return platform_match_id(pdrv->id_table, pdev) != NULL;

    /* fall-back to driver name match */
    return (strcmp(pdev->name, drv->name) == 0);
}
```

### 内核操作设备树的常用函数

#### 设备树相关头文件

```c
/* 内核中设备树相关的头文件介绍 */
/* 设备树处理流程：dtb文件 -> device_node -> platform_device */
/* of: open firmware（开放固件） */
/* 1. 处理dtb文件 */
of_fdt.h; // dtb 文件的相关操作函数, 我们一般用不到，dtb 文件在内核中已经被转换为 device_node 树

/* 2. 处理device_node树 */
of.h // 提供设备树的一般处理函数,
// 比如 of_property_read_u32(读取某个属性的 u32 值),
// of_get_child_count(获取某个 device_node 的子节点数)
of_address.h // 地址相关的函数,
// 比如 of_get_address(获得 reg 属性中的 addr, size 值)
// of_match_device (从 matches 数组中取出与当前设备最匹配的一项)
of_dma.h // 设备树中 DMA 相关属性的函数
of_gpio.h // GPIO 相关的函数
of_graph.h // GPU 相关驱动中用到的函数, 从设备树中获得 GPU 信息
of_iommu.h // 很少用到
of_irq.h // 中断相关的函数
of_mdio.h // MDIO (Ethernet PHY) API
of_net.h // OF helpers for network devices.
of_pci.h // PCI 相关函数
of_pdt.h // 很少用到
of_reserved_mem.h; // reserved_mem 的相关函数
    
/* 3. 处理platform_device */
of_platform.h // 把 device_node 转换为 platform_device 时用到的函数,
 // 比如 of_device_alloc(根据 device_node 分配设置 platform_device),
 // of_find_device_by_node (根据 device_node 查找到 platform_device),
 // of_platform_bus_probe (处理 device_node 及它的子节点)
of_device.h; // 设备相关的函数, 比如 of_match_device
```

#### platform_device和device_node相关函数

```c
/* 1. platform_device相关函数 */
/* 我们只用其中几个 */
/* -a. of_find_device_by_node */

/* -b. platform_get_resource */

/* 2. 访问device_node结构体（dtb文件生成的），因为有些device_node不会转换为platform_device */
/* 内核源码 incldue/linux/of.h 中声明了 device_node 和属性 property 的操作函数 */
/* 找到节点 */
/* -a. of_find_node_by_path */

/* -b. of_find_node_by_name */

/* -c. of_find_node_by_type */

/* -d. of_find_compatible_node */

/* -e. of_find_node_by_phandle */

/* -f. of_get_parent */

/* -g. of_get_next_parent */

/* -h. of_get_next_child */

/* -i. of_get_next_available_child */

/* -j. of_get_child_by_name */

/* 找到属性 */
/* -a. —of_find_property */

/* 获得属性的值 */
/* -a. of_get_property */

/* -b. of_property_count_elems_of_size */

/* -c. of_property_read_u32 读取u32整数 */

/* -d. of_property_read_u32_index */

/* -e. of_property_read_variable_u32_array 读取u32数组 */

/* -f. of_property_read_string 读取字符串 */
```

### 设备树文件与驱动程序的关系

```c
/* 编写设备树节点要依据驱动程序的要求 */
/* 1. 设备树节点要与platform_driver匹配 */
/* -a. 设备树要有 compatible 属性，它的值是一个字符串 */
/* -b. platform_driver 中要有 of_match_table，其中一项的.compatible 成员设置为一个字符串 */
/* -c. 上述 2 个字符串要一致。 */

/* 2. 设备树节点指定platform_driver需要的资源 */
/* -a. 如果在设备树节点里使用reg属性，那么内核生成对应的platform_device时会用 reg 属性来设置 IORESOURCE_MEM 类型的资源。??? */
/* -b. 驱动程序中根据 pin 属性来确定引脚，那么我们就在设备树节点中添加 pin 属性。 */
#define GROUP_PIN(g,p) ((g<<16) | (p))
100ask_led0 {  // 设备树节点
 compatible = “100ask,led”;
 pin = <GROUP_PIN(5, 3)>;
};

/* -c. 驱动程序中，可以从 platform_device 中得到 device_node，再用 of_property_read_u32 得到属性的值 */
struct device_node* np = pdev->dev. of_node;
int led_pin;
int err = of_property_read_u32(np, “pin”, &led_pin);
```



## LED驱动程序的设备树

```c
/* 修改设备树文件以匹配设备树节点和驱动程序，并且指定驱动所需资源。 */
/* 设备树文件目录：内核源码目录中的arch/arm/boot/dts/100ask_imx6ull-14x14.dts */
```

## GPIO按键驱动程序

### 按键驱动的四种方法

```c
/* 按键驱动的四种方法 */
/* 建议在open函数里配置引脚pin，因为下载驱动程序不一定调用硬件，而调用硬件必须open */
/* 1. 查询方法 */
/* 驱动程序中构造、注册一个 file_operations 结构体，里面提供有对应的open,read 函数。
 * APP 调用 open 时，导致驱动中对应的 open 函数被调用，在里面配置 GPIO 为输入引脚。
 * APP 调用 read 时，导致驱动中对应的 read 函数被调用，它读取寄存器，把引脚状态直接返回给 APP。
 */

/* 2. 休眠-唤醒方法 */
/* 驱动程序中构造、注册一个 file_operations 结构体，里面提供有对应的open,read 函数。
 * APP 调用 open 时，导致驱动中对应的 open 函数被调用，在里面配置 GPIO 为输入引脚；并且注册 GPIO 的中断处理函数。
 * APP 调用 read 时，导致驱动中对应的 read 函数被调用，如果有按键数据则直接返回给 APP；否则 APP 在内核态休眠。
 */
/* 具体流程：
 * 当用户按下按键时，GPIO 中断被触发，导致驱动程序之前注册的中断服务程序被执行。它会记录按键数据，并唤醒休眠中的 APP。
 * APP 被唤醒后继续在内核态运行，即继续执行驱动代码，把按键数据返回给APP(的用户空间)。
 */

/* 3. poll/select 方式 */
/* 驱动程序中构造、注册一个 file_operations 结构体，里面提供有对应的open,read,poll 函数。
 * APP 调用 open 时，导致驱动中对应的 open 函数被调用，在里面配置 GPIO 为输入引脚；并且注册 GPIO 的中断处理函数。
 * APP 调用 poll 或 select 函数，意图是“查询”是否有数据，这 2 个函数都可以指定一个超时时间，即在这段时间内没有数据的话就返回错误。这会导致驱动中对应的 poll 函数被调用，如果有按键数据则直接返回给 APP；否则 APP 在内核态休眠一段时间。
 */
/* 具体流程：
 * 当用户按下按键时，GPIO 中断被触发，导致驱动程序之前注册的中断服务程序被执行。它会记录按键数据，并唤醒休眠中的 APP。APP 被唤醒有 2 种原因：用户操作了按键，超时。
 * 被唤醒的 APP 在内核态继续运行，即继续执行驱动代码，把“状态”返回给 APP(的用户空间)。
 * APP 得到 poll/select 函数的返回结果后，如果确认是有数据的，则再调用 read 函数，这会导致驱动中的 read 函数被调用，这时驱动程序中含有数据，会直接返回数据。
 */

/* 4. 异步通知方法 */
/* 驱动程序中构造、注册一个 file_operations 结构体，里面提供有对应的open,read,fasync 函数。
 * APP 调用 open 时，导致驱动中对应的 open 函数被调用，在里面配置 GPIO 为输入引脚；并且注册 GPIO 的中断处理函数。
 * APP 给信号 SIGIO 注册自己的处理函数：my_signal_fun。
 * APP 调用 fcntl 函数，把驱动程序的 flag 改为 FASYNC，这会导致驱动程序的 fasync 函数被调用，它只是简单记录进程 PID。
 * 当用户按下按键时，GPIO 中断被触发，导致驱动程序之前注册的中断服务程序被执行。它会记录按键数据，然后给进程 PID 发送 SIGIO 信号。
 * APP 收到信号后会被打断，先执行信号处理函数：在信号处理函数中可以去调用 read 函数读取按键值。
 */
```

### 查询方法程序框架

```c
/* 查询方法程序框架 */
/* 1. button_drv.c 分配/设置/注册 file_operations 结构体 */
/* 呈上：向上提供 button_open,button_read 供 APP 调用。
 * 启下：同时这 2 个函数又会调用底层硬件提供的 p_button_opr 中的 init、read 函数操作硬件。
 */

/* 2. board_xxx.c 分配/设置/注册 button_operations 结构体 */
/* button_operations 结构体定义单板 xxx 的按键操作函数。
 * 对于不同的单板，只需要替换 board_xxx.c 提供自己的 button_operations 结构体即可。
 */
```

## Pinctrl子系统和GPIO子系统

### Princtrl子系统概念

```c
/* 1. pin controller */
/* -a. 用来复用引脚、配置引脚用于GPIO、I2C等功能或者工作、休眠状态。而GPIO controller用于设置引脚为输入、输出 */
pincontroller {
    state_0_node_a {
        function = "uart0";  // 复用为uart0功能
        groups = "u0rxtx", "u0rtscts";  // 用到的引脚
    }
    state_1_node_a {
        groups = "u0rxtx", "u0rtscts";  // 用到的引脚
        output-high;  // 配置输出高电平
    }
}

/* 2. client device */
/* -a. pinctrl子系统的客户设备，在设备树里定义为一个节点，并声明需要的引脚 */
device {
    pinctrl-name = "default", "sleep";  // 设备有两个状态：默认状态，配置为UART功能；休眠状态，引脚配置为高电平（省电）
    pinctrl-0 = <&state_0_node_a>;  // 第0个状态的名字是default，对应引脚在pinctrl-0里定义。
    pinctrl-1 = <&state_1_node_a>;
}

/* 3. 当设备切换状态时，对应的pinctrl会自动调用 */
/* 通过函数调用（不常用） */
devm_pinctrl_get_select_default(struct device *dev); // 使用"default"状态的引脚
pinctrl_get_select(struct device *dev, const char *name); // 根据 name 选择某种状态的引
脚
pinctrl_put(struct pinctrl *p); // 不再使用, 退出时调用
```

### GPIO子系统概念

```c
/* 1. 设备树中的GPIO controller节点指明引脚 */
gpio1: gpio@0209c000 {
    compatible = "fsl,imx6ul-gpio", "fsl,imx35-gpio";

    reg = <0x0209c000 0x4000>;
    interrupts = <GIC_SPI 66 IRQ_TYPE_LEVEL_HIGH>,
                 <GIC_SPI 67 IRQ_TYPE_LEVEL_HIGH>;

    gpio-controller;  // 表明这个节点是一个GPIO controller.
    #gpio-cells = <2>;  // 表明该节点下每一个引脚用2个u32数据表示，第一个u32表示引脚，第二个表示电平
    interrupt-controller;
    #interrupt-cells = <2>;
    gpio-ranges = <&iomuxc 0 23 10>, <&iomuxc 10 17 6>,
                  <&iomuxc 16 33 16>;
};

gpio2: gpio@020a0000 {
    compatible = "fsl,imx6ul-gpio", "fsl,imx35-gpio";

    reg = <0x020a0000 0x4000>;
    interrupts = <GIC_SPI 68 IRQ_TYPE_LEVEL_HIGH>,
                 <GIC_SPI 69 IRQ_TYPE_LEVEL_HIGH>;

    gpio-controller;
    #gpio-cells = <2>;
    interrupt-controller;
    #interrupt-cells = <2>;
    gpio-ranges = <&iomuxc 0 49 16>, <&iomuxc 16 111 6>;
};

/* 2. 在设备节点中调用GPIO controller：使用属性"[<name>-]gpios" */
/* 示例： */
led0: cpu {
    label = "cpu";
    gpios = <&gpio5 3 GPIO_ACTIVE_LOW>;  // 引用GPIO引脚，名称gpio5，第3个引脚，电平为LOW
    default-state = "on";
    linux, default-trigger = "heartbeat";
};

gt9xx@5d {
    compatible = "goodix,gt9xx";
    reg = <0x5d>;
    status = "okay";
    interrupt-parent = <&gpiol>;
    interrupts = <5 IRQ_TYPE_EDGE_FALLING>;
    pinctrl-names = "default";
    pinctrl-0 = <&pinctrl_tsc_reset &pinctrl_touchscreen_int>;
    
    reset-gpios = <&gpio5 2 GPIO_ACTIVE_LOW>;  // 引用GPIO引脚
    irq-gpios = <&gpio1 5 IRQ_TYPE_EDGE_FALLING>;  // 引用GPIO引脚
};

/* 3. 在驱动程序中调用GPIO子系统：通过GPIO子系统的接口函数 */
/* -a. 基于描述符的接口与旧的接口 */
// 基于描述符：函数前缀"gpiod_"，用gpio_desc结构体表示一个引脚
#include <linux/gpio/consumer.h> // descriptor-based
// 旧的：函数前缀"gpio_"，用一个整数来表示一个引脚
#include <linux/gpio.h> // legacy

/* -b. 常用接口函数 */
/* --. 获得GPIO */
gpiod_get;  // gpio_request
gpiod_get_index;  // 示例：gpiod_get_index(dev, "led", 0, GPIOD_OUT_HIGH);
gpiod_get_array;  // gpio_request_array
devm_gpiod_get;
devm_gpiod_get_index;
devm_gpiod_get_array;
/* --. 设置方向 */
gpiod_direction_input;  // gpio_direction_input
gpiod_direction_output;  // gpio_direction_output
/* --. 读值、写值 */
gpiod_get_value;  // gpio_get_value
gpiod_set_value;  // gpio_set_value，设置的为"逻辑值"，不一定等于"物理值"
/* --. 释放GPIO */
gpio_free gpio_free;
gpiod_put gpio_free_array;
gpiod_put_array;
devm_gpiod_put;  // devm_: 设备资源管理，设备销毁时自动释放资源
devm_gpiod_put_array;
/* --. 确定引脚号：在 GPIO 子系统中，每注册一个 GPIO Controller 时会确定它的“base number”，那么这个控制器里的第 n 号引脚的号码就是：base number + n。 */

/* 4. 在sysfs中访问GPIO的过程： */
/* -a. 先确定某个 GPIO Controller 的基准引脚号(base number)，再计算出某个引脚的号码。 */
/* --. 先在开发板的/sys/class/gpio 目录下，找到各个 gpiochipXXX 目录 */
/* --. 然后进入某个 gpiochip 目录，查看文件 label 的内容 */
/* --. 根据 label 的内容对比设备树，label 内容来自设备树，比如它的寄存器基地址。用来跟设备树(dtsi 文件)比较，就可以知道这对应哪一个 GPIO Controller。 */

/* -b. 基于 sysfs 操作引脚（如果 GPIO4_14 的号码是 96+14=110，可以如下操作读取按键值：） */
/* 好麻烦。。 */
[root@100ask:~]# echo 110 > /sys/class/gpio/export;
[root@100ask:~]# echo in > /sys/class/gpio/gpio110/direction;
[root@100ask:~]# cat /sys/class/gpio/gpio110/value;
[root@100ask:~]# echo 110 > /sys/class/gpio/unexport;
```

### 基于GPIO子系统的LED驱动程序

```c
/* 基于GPIO子系统的LED驱动程序的流程： */
/* 1. 定义、注册一个platform_driver */
static const struct of_device_id ask100_leds[] = {
    { .compatible = "100ask,leddrv" },  // 与设备树节点的compatible对应
    { },
};
/* 定义 platform_driver */
static struct platform_driver chip_demo_gpio_driver = {
    .probe = chip_demo_gpio_probe,
    .remove = chip_demo_gpio_remove,
    .driver = {
        .name = "100ask_led",
        .of_match_table = ask100_leds,
    },
};
/* 在入口函数注册 platform_driver */
static int __init led_init(void)
{
    int err;

    printk("%s %s line %d\n", __FILE__, __FUNCTION__, __LINE__);

    err = platform_driver_register(&chip_demo_gpio_driver);
    return err;
}

/* 2. 在platform_driver的probe函数中： */
/* -a. 根据 platform_device 的设备树信息确定 GPIO：gpiod_get */
led_gpio = gpiod_get(&pdev->dev, "led", 0);
if (IS_ERR(led_gpio)) {
	dev_err(&pdev->dev, "Failed to get GPIO for led\n");
	return PTR_ERR(led_gpio);
}

/* -b. 定义、注册一个 file_operations 结构体 */
major = register_chrdev(0, "100ask_led", &led_drv); /* /dev/led */

led_class = class_create(THIS_MODULE, "100ask_led_class");
if (IS_ERR(led_class)) {
    printk("%s %s line %d\n", __FILE__, __FUNCTION__, __LINE__);
    unregister_chrdev(major, "led");
    gpiod_put(led_gpio);
    return PTR_ERR(led_class);
}

device_create(led_class, NULL, MKDEV(major, 0), NULL, "100ask_led%d", 0); /* /dev/100ask_led0 */

/* -c. 在 file_operarions 中使用 GPIO 子系统的函数操作 GPIO：gpiod_direction_output、gpiod_set_value */
gpiod_direction_output(led_gpio, 0);  // 在open函数中设置GPIO引脚方向
gpiod_set_value(led_gpio, status);  // 在write函数中设置GPIO引脚值
gpiod_put(led_gpio);  // 释放GPIO

/* 3. 补充：在设备树中添加pinctrl信息 */
/* --. 有些芯片提供了设备树生成工具，在 GUI 界面中选择引脚功能和配置信息，就可以自动生成 Pinctrl 子结点。把它复制到你的设备树文件中，再在 client device 结点中引用就可以。
 * --. 有 些 芯 片 只 提 供 文 档 ， 那 就 去 阅 读 文 档 ， 一 般 在 内 核 源 码 目 录Documentation\devicetree\bindings\pinctrl 下面，保存有该厂家的文档。
 * --. 如果连文档都没有，那只能参考内核源码中的设备树文件，在内核源码目录arch/arm/boot/dts 目录下。
 * --. 最后一步，网络搜索。 */

/* 4. 补充：在设备树中添加 */
```

## Linux系统对中断的处理

### 异常、中断概念及处理流程

```c
/* 1. 异常、中断概念 */
/* -a. 异常：指令未定义；指令、数据访问有问题；SWI(软中断)；快中断；中断*/
/* -b. 中断：按键；定时器；ADC 转换完成；UART 发送完数据、收到数据 */

/* 2. 中断处理流程 */
/* -a. 初始化GIC通用中断控制器 */
/* --. 设置中断源，让它可以产生中断：按键中断使能，注册中断ID对应的中断处理函数 */
/* --. 设置中断控制器(可以屏蔽某个中断，优先级) */
/* --. 中断使能 */

/* -b. 执行其他程序：正常程序；产生中断：比如按下按键--->中断控制器（GIC_distributor -- GIC_CPU interface）--->CPU */

/* -c. CPU 每执行完一条指令都会检查有无中断/异常产生（读取到伪中断1023，表示该内核不再有任何待处理中断） */

/* -d. CPU 发现有中断/异常产生，开始处理，跳到中断ID对应的处理程序 */
/* --. 保存现场(各种寄存器)
 * --. 处理异常(中断):分辨中断源，再调用不同的处理函数
 * --. 恢复现场
 */

/* 2. 异常向量表 */
_start: b reset
ldr pc, _undefined_instruction
ldr pc, _software_interrupt
ldr pc, _prefetch_abort
ldr pc, _data_abort
ldr pc, _not_used
ldr pc, _irq //发生中断时，CPU 跳到这个地址执行该指令 **假设地址为 0x18**
ldr pc, _fiq;

```

### 进程、线程概念

在 Linux 中：资源分配的单位是进程，调度的单位是线程。

### 硬件中断、软件中断

中断处理原则：1. Linux系统中断不能嵌套；2. 越快越好

中断拆分：上半部（紧急事情），下半部（非紧急事情，tasklet（小任务），workqueue（工作队列，使用kworker线程处理中断））

request_threaded_irq(); 为每一个中断都创建一个内核线程；多个中断的内 核线程可以分配到多个 CPU 上执行，这提高了效率。

能弄清楚下面这个图，对 Linux 中断系统的掌握也基本到位了

![image-20250129214829226](C:\Users\22499\AppData\Roaming\Typora\typora-user-images\image-20250129214829226.png)

### 中断系统结构体

```c
/* 1. irq_desc结构体 */
/* -a. 定义：include/linux/irqdesc.h */

/* -b. 函数：irq_flow_handler_t handler_irq */
/* --. 读取GIC控制器获得中断号A，irq_desc[A].handle_irq 细分出GPIO中断 B
 * --. 对于 GPIO 模块向 GIC 发出的中断 B，irq_desc[B].handle_irq会调用action链表里的函数，这些函数由外部设备提供。
 * --. action链表中的函数自行判断该中断是否自己产生，若是则处理。
 */

/* 2. irqaction结构体 */
/* -a. 定义：include/linux/interrupt.h */

/* -b. 当调用 request_irq、request_threaded_irq 注册中断处理函数时，内核就会构造一个 irqaction 结构体。在里面保存 name、dev_id、handler、thread_fn、thread。 */
/* --. handler 是中断处理的上半部函数，用来处理紧急的事情。
 * --. thread_fn 对应一个内核线程 thread，当 handler 执行完毕，Linux 内核会唤醒对应的内核线程。在内核线程里，会调用 thread_fn 函数。
 *   可以提供 handler 而不提供 thread_fn，就退化为一般的 request_irq 函数。
 *   可以不提供 handler 只提供 thread_fn，完全由内核线程来处理中断。
 *   也可以既提供 handler 也提供 thread_fn，这就是中断上半部、下半部。
 * --. dev_id：中断处理函数执行时，可以使用 dev_id；卸载中断时要传入 dev_id，这样才能在 action 链表中根据 dev_id 找到对应项
 */

/* 3. irq_data结构体 */
/* -a. 定义：include/linux/irq.h */

/* -b. irq_data结构体包含 irq_chip 指针 irq_domain 指针，都是指向别的结构体 */

/* 4. irq_domain结构体 */
/* -a. 定义：include/linux/irqdomain.h */

/* -b. irq_domain结构体包含irq_domain_ops 结构体，里面有各种操作函数 */
/* --. xlate()用来解析设备树的中断属性，提取出 hwirq、type 等信息。
 * --. map()把 hwirq 转换为 irq。
 */

/* -c. irq_domain会把本地的 hwirq（硬件中断号） 映射为全局的 irq（软件中断号）。比如 GPIO 控制器里有第 1 号中断，UART 模块里也有第 1 号中断，这两个“第 1 号中断”是不一样的，它们属于不同的"域"irq_domain。 */

/* 5. irq_chip结构体 */
/* 我们在 request_irq 后，并不需要手工去使能中断，原因就是系统调用对应的 irq_chip 里的函数帮我们使能了中断。我们提供的中断处理函数中，也不需要执行主芯片相关的清中断操作，也是系统帮我们调用 irq_chip 中的相关函数。但是对于外部设备相关的清中断操作，还是需要我们自己做的。 */
```

### 中断示例

```c
/* 1. 设备树中断节点语法 */
/* -a. 层级关系（父：GIC硬件中断控制器、子：GPIOx软件中断控制器）、中断源、中断号：假设 GPIO1 有 32 个中断源，用到 GIC 的两个中断（16个中断源汇聚成1个中断号），会涉及 GIC 里的 2 个 hwirq（中断号）。 */
/* -b. 中断控制器节点属性 */
/* --. interrupt-controller: 表明它是“中断控制器”
 * --. #interrupt-cells = <n>: 表明引用这个中断控制器的话需要多少个 cell，如果是2个cell，则另一个cell表示中断触发类型
 */
/* -c. 外设使用中断（指明中断控制器和中断引脚） */
i2c@7000c000 {
	gpioext: gpio-adnp@41 {
        compatible = "ad,gpio-adnp";
        interrupt-parent = <&gpio>;  // 指明中断控制器
        interrupts = <160 1>;  // 指明中断引脚
        // interrupts-extended = <&intc1 5 1>, <&intc2 1 0>;  既指定“interrupt-parent”，也指定“interrupts”
        gpio-controller;
        #gpio-cells = <1>;
        interrupt-controller;
        #interrupt-cells = <2>;
    }
};
/* -d. 设备树中断节点示例：在 arch/arm/boot/dts 目录下可以看到 2 个文件：imx6ull.dtsi、100ask_imx6ull-14x14.dts，把里面有关中断的部分内容抽取出来。
 */

/* 2. 代码中的中断信息 */
/* -a. 指定中断属性的中断节点转换为platform_device后可以获得IORESOURCE_IRQ资源（中断号），函数如下：
 * 输入参数：
 *   dev: platform_device;
 *   type: 资源类型，如IORESOURCE_MEM、IORESOURCE_REG、IORESOURCE_IRQ 等
 *   num: 资源中的哪一个
 */
struct resource *platform_get_resource(struct platform_device *dev, unsigned int type, unsigned int num);

/* -b. I2C 设备节点会被转换为一个 i2c_client 结构体，中断号会保存在 i2c_client 的 irq 成员里，代码如下(drivers/i2c/i2c-core.c)
 * SPI 设备会被转换为一个 spi_device 结构体，中断号会保存在 spi_device 的 irq 成员里，代码如下(drivers/spi/spi.c)
 */

/* -c. 如果设备节点既不能转换为 platform_device，它也不是 I2C 设备，不是 SPI 设备，那么在驱动程序中可以自行调用 of_irq_get 函数去解析设备树，得到中断号。 */

/* -d. GPIO设备节点示例 */
/* 文件：drivers/input/keyboard/gpio_keys.c */
gpio-keys {
    compatible = "gpio-keys";
    pinctrl-names = "default";
    user {
        label = "User Button";
        gpios = <&gpio5 1 GPIO_ACTIVE_HIGH>;
        gpio-key,wakeup;
        linux,code = <KEY_1>;
    };
};
// 那么可以使用下面的函数获得引脚和 flag：
button->gpio = of_get_gpio_flags(pp, 0, &flags);
bdata->gpiod = gpio_to_desc(button->gpio);
// 再去使用 gpiod_to_irq 获得中断号：
irq = gpiod_to_irq(bdata->gpiod);

/* 3. 编程思路：使用中断的按键驱动程序 */
/* 内核源码：drivers/input/keyboard/gpio_keys.c */
/* -a. 设备树节点信息 */
/* --. 查看原理图确定按键使用的引脚，再在设备树中添加节点，在节点里指定中断信息。 */
gpio_keys_100ask {
    compatible = "100ask,gpio_key";
    gpios = <&gpio5 1 GPIO_ACTIVE_HIGH
            &gpio4 14 GPIO_ACTIVE_HIGH>;
    pinctrl-names = "default";
    pinctrl-0 = <&key1_pinctrl
                &key2_pinctrl>;
};
/* -b. 驱动程序 */
/* 源码：01_all_series_quickstart\05_嵌入式 Linux 驱动开发基础知识\source\06_gpio_irq\01_simple */
/* --. 从设备树中获得GPIO */
count = of_gpio_count(node);
for (i = 0; i < count; i++)
    gpio_keys_100ask[i].gpio = of_get_gpio_flags(node, i, &flag);
/* --. 从GPIO中获得中断号 */
gpio_keys_100ask[i].irq = gpio_to_irq(gpio_keys_100ask[i].gpio);
/* --. 中断函数 */
static irqreturn_t gpio_key_isr(int irq, void *dev_id) {
    struct gpio_key *gpio_key = dev_id;、
    int val;
    val = gpiod_get_value(gpio_key->gpiod);
    printk("key %d %d\n", gpio_key->gpio, val);
    return IRQ_HANDLED;
}
/* --. 注册中断 */
err = request_irq(gpio_keys_100ask[i].irq, gpio_key_isr, \
IRQF_TRIGGER_RISING | IRQF_TRIGGER_FALLING, "100ask_gpio_key", &gpio_keys_100ask[i]);
```

## 驱动程序基石

### 休眠-唤醒机制

```c
/* 1. 休眠-唤醒机制执行过程： */
/* -a. APP调用read函数试图读取数据（用户态） */
/* -b. APP调用驱动程序中read对应的函数（内核态），如果没有数据，则休眠；否则，发生按键中断（记录数据、唤醒APP），复制数据到用户态（copy_to_user） */

/* 2. 内核函数 */
/* -a. 休眠函数 */
/* 目录：include\linux\wait.h，函数原型如下：
 * 输入参数：
 * --. wq：waitqueue，等待队列。休眠时将程序状态改为非Running，而且把进程放入到wq中，以便后续唤醒。
 * --. condition: 可以是变量，或者任何表达式。
 */
wait_event_interruptible(wq, condition);  // 休眠，直到 condition 为真；休眠期间是可被打断的，可以被信号(Sig??)打断
wait_event(wq, condition);  // 休眠，直到 condition 为真；退出的唯一条件是 condition 为真
wait_event_interruptible_timeout(wq, condition, timeout);  // 休眠，直到 condition 为真或超时，可以被信号打断
wait_event_timeout(wq, condition, timeout);

/* -b. 唤醒函数 */
/* 目录：include\linux\wait.h，函数原型如下： */
wake_up_interruptible(x);  // 唤醒 x 队列中状态为“TASK_INTERRUPTIBLE”的线程，只唤醒其中的一个线程
wake_up_interruptible_nr(x, nr);  // 唤醒 x 队列中状态为“TASK_INTERRUPTIBLE”的线程，只唤醒其中的 nr 个线程
wake_up_interruptible_all(x);  // 唤醒 x 队列中状态为“TASK_INTERRUPTIBLE”的线程，唤醒其中的所有线程
wake_up(x);  // 唤 醒 x 队 列 中 状 态 为 “ TASK_INTERRUPTIBLE ” 或 “TASK_UNINTERRUPTIBLE”的线程，只唤醒其中的一个线程
wake_up_nr(x, nr);  // 唤 醒 x 队 列 中 状 态 为 “ TASK_INTERRUPTIBLE ” 或 “TASK_UNINTERRUPTIBLE”的线程，只唤醒其中 nr 个线程
wake_up_all(x);  // 唤 醒 x 队 列 中 状 态 为 “ TASK_INTERRUPTIBLE ” 或 “TASK_UNINTERRUPTIBLE”的线程，唤醒其中的所有线程

/* 3. 驱动程序框架 */
/* -a. 初始化 wq 队列（要休眠的线程） */
/* -b. 在驱动的 read 函数中，调用 wait_event_interruptible：它本身会判断 event 是否为 FALSE，如果为 FASLE 表示无数据，则休眠。当从 wait_event_interruptible 返回后，把数据复制回用户空间(copy_to_user)。 */
/* -c. 在中断服务程序里：设置 event 为 TRUE，并调用 wake_up_interruptible 唤醒线程。 */

/* 4. 关键代码 */
/* -a. 设备树GPIO节点 */
/* -b. 驱动程序read函数 */
static DECLARE_WAIT_QUEUE_HEAD(gpio_key_wait);  // 定义"wait queue"
static ssize_t gpio_key_drv_read (struct file *file, char __user *buf, size_t siz
e, loff_t *offset) {
    int err;
    wait_event_interruptible(gpio_key_wait, g_key);  // 判断g_key是否为TRUE，是否休眠
    err = copy_to_user(buf, &g_key, 4);  // 要么有了数据(g_key 为 TRUE)，要么有信号等待处理（暂不涉及）。
    g_key = 0;
    return 4;
}
/* -c. 中断程序 */
static irqreturn_t gpio_key_isr(int irq, void *dev_id) {
    struct gpio_key *gpio_key = dev_id;
    int val;
    val = gpiod_get_value(gpio_key->gpiod);
    printk("key %d %d\n", gpio_key->gpio, val);
    g_key = (gpio_key->gpio << 8) | val;
    wake_up_interruptible(&gpio_key_wait);  // 唤醒 gpio_key_wait 中的第 1 个线程
    return IRQ_HANDLED;
}
/* -d. 测试程序 */
/* -e. 使用环形缓冲区以避免按键数据丢失 */
```

### POLL/SELECT机制

```c
/* 1. 不同于休眠-唤醒机制的read函数，POLL/SELECT机制改用poll函数，可以指定休眠的时长 */
/* -a. drv_poll函数实现功能：
 * --. 把当前线程挂入队列 wq：poll_wait，使用内核的函数 poll_wait 把线程挂入队列，如果线程已经在队列里了，它就不会再次挂入。
 * --. APP 调用 poll 函数时，有可能是查询“有没有数据可以读”：POLLIN，也有可能是查询“你有没有空间给我写数据”：POLLOUT。所以 drv_poll 要返回自己的当前状态：(POLLIN | POLLRDNORM) 或 (POLLOUT | POLLWRNORM)
 */
/* -b. drv_poll函数代码： */
static unsigned int gpio_key_drv_poll(struct file *fp, poll_table * wait) {
    printk("%s %s line %d\n", __FILE__, __FUNCTION__, __LINE__);
    poll_wait(fp, &gpio_key_wait, wait);
    return is_key_buf_empty() ? 0 : POLLIN | POLLRDNORM;
}

/* 2. 测试代码 */
struct pollfd fds[1];
int timeout_ms = 5000;
int ret;
fds[0].fd = fd;  // 监测哪一个文件
fds[0].events = POLLIN;  // 监测哪种事件
ret = poll(fds, 1, timeout_ms);
if ((ret == 1) && (fds[0].revents & POLLIN)) {
    read(fd, &val, 4);
    printf("get button : 0x%x\n", val);
}

/* 3. 内核代码文件详解：*/
/* -a. sys_poll：超时参数处理，并调用do_sys_poll  */
/* -b. do_sys_poll：初始化poll_wqueues变量table，并调用do_poll */
/* -c. do_poll：核心代码，第一次调用驱动程序的poll函数，其中poll_wait会调用__pollwait函数把线程放入某个队列中，pt->_qproc设置为NULL，则第二次调用poll函数不会再次把线程放入某个队列。如果驱动程序的 poll 返回有效值，则 count 非 0，跳出循环；否则休眠一段时间；当休眠时间到，或是被中断唤醒时，会再次循环、再次调用驱动程序的 poll。 */
```

### 异步通知机制

```c
/* 1. 异步通知机制过程： */
/* -a. signal(SIGIO, func): APP 给 SIGIO 这个信号注册信号处理函数 func，以后 APP 收到 SIGIO信号时，这个函数会被自动调用； */
/* -b. sys_fcntl: 把 APP 的 PID(进程 ID)告诉驱动程序，这个调用不涉及驱动程序，在内核的文件系统层次记录 PID；读取驱动程序文件 Flag；设置 Flag 里面的 FASYNC 位为 1：当 FASYNC 位发生变化时，会导致驱动程序的 fasync 被调用（使能异步通知功能） */
/* -c. 调 用 faync_helper ， 它 会 根 据 FAYSNC 的值决定是否设置 button_async->fa_file=驱动文件 filp（驱动文件 filp 结构体里面含有之前设置的 PID。） */
/* -d. APP 可以做其他事； */
/* -e. 按下按键，发生中断，驱动程序的中断服务程序被调用，里面调用 kill_fasync 发信号（如果button_async->fa_file非空，则从中取出PID，向它发信号）； */
/* -f. APP 收到信号后，它的信号处理函数被自动调用，可以在里面调用 read 函数读取按键。 */

/* 2. 驱动程序函数： */
/* -a. drv_fasync函数：fasync_helper 函 数 会 分 配 、 构 造 一 个 fasync_struct 结构体
button_async */
static struct fasync_struct *button_async;
static int drv_fasync (int fd, struct file *filp, int on) {
    return fasync_helper (fd, filp, on, &button_async);
}
/* -b. kill_fasync (&button_async, SIGIO, POLL_IN): 
 * 输入参数：
 * --. button_async->fa_file 非空时，可以从中得到 PID，表示发给哪一个 APP
 * --. SIGIO: 发送的信号
 * --. POLL_IN：为什么发送信号，这个表示有数据可读
 */

/* 3. 测试代码： */
/* -a. 编写信号处理函数 */
static void sig_func(int sig) {
    int val;
    read(fd, &val, 4);
    printf("get button : 0x%x\n", val);
}
/* -b. 注册信号处理函数 */
signal(SIGIO, sig_func);
/* -c. 打开驱动 */
fd = open(argv[1], O_RDWR);
/* -d. 把进程 ID 告诉驱动 */
fcntl(fd, F_SETOWN, getpid());
/* -e. 使能驱动的 FASYNC 功能 */
flags = fcntl(fd, F_GETFL);
fcntl(fd, F_SETFL, flags | FASYNC);

/* 4. 内核代码：F_SETOWN, F_GETFL, F_SETFL, FASYNC, send_sigio */
```

### 阻塞与非阻塞

阻塞，就是等待某件事情发生。比如调用 read 读取按键时，如果没有按键数据则 read 函数不会返回，它会让线程休眠等待。APP 调用 open 函数时，传入 O_NONBLOCK，就表示要使用非阻塞方式；默认是阻塞方式。在 open 之后，也可以通过 fcntl 修改为阻塞或非阻塞。

```c
/* 1. 设置（非）阻塞方式 */
/* -a. open时设置 */
int fd = open(“/dev/xxx”, O_RDWR | O_NONBLOCK); /* 非阻塞方式 */
int fd = open(“/dev/xxx”, O_RDWR ); /* 阻塞方式 */
/* -b. open之后设置 */
int flags = fcntl(fd, F_GETFL);
fcntl(fd, F_SETFL, flags | O_NONBLOCK); /* 非阻塞方式 */
fcntl(fd, F_SETFL, flags & ~O_NONBLOCK); /* 阻塞方式 */

/* 2. 驱动编程 */
/* 从驱动代码也可以看出来，当 APP 打开某个驱动时，在内核中会有一个 struct file 结构体对应这个驱动，这个结构体中有 f_flags，就是打开文件时的标记位；可以设置 f_flasgs 的 O_NONBLOCK 位，表示非阻塞；也可以清除这个位表示阻塞。 */
static ssize_t drv_read(struct file *fp, char __user *buf, size_t count, loff_t *ppo
s) {
    if (queue_empty(&as->queue) && fp->f_flags & O_NONBLOCK)
        return -EAGAIN;
    wait_event_interruptible(apm_waitqueue, !queue_empty(&as->queue));
    ......;
}

```

### 定时器

```c
/* 1. 定时器内核函数 */
/* 目录：include\linux\timer.h */
setup_timer(timer, fn, data);  // 设置定时器，主要是初始化 timer_list 结构体，设置其中的函数、参数
void add_timer(struct timer_list *timer);  // 向内核添加定时器。timer->expires 表示超时时间。当超时时间到达时，内核就会调用函数timer->function(timer->data)。
int mod_timer(struct timer_list *timer, unsigned long expires);  // 修改定时器的超时时间，等同于：del_timer(timer); timer->expires = expires; add_timer(timer)
int del_timer(struct timer_list *timer);  // 删除定时器

/* 2. 定时器时间单位 */
CONFIG_HZ=100;  // 表示内核每秒中会发生 100 次系统滴答中断(tick)，每发生一次 tick 中断，全局变量 jiffies 就会累加 1。CONFIG_HZ=100 表示每个滴答是 10ms。
/* -a. 修改超时时间 */
/* 在 add_timer 之前，直接修改： */
timer.expires = jiffies + xxx;  // xxx 表示多少个滴答后超时，也就是 xxx*10ms
timer.expires = jiffies + 2*HZ; // HZ 等于 CONFIG_HZ，2*HZ 就相当于 2 秒
/* 在 add_timer 之后，使用 mod_timer 修改： */
mod_timer(&timer, jiffies + xxx); // xxx 表示多少个滴答后超时，也就是 xxx*10ms
mod_timer(&timer, jiffies + 2*HZ); // HZ 等于 CONFIG_HZ，2*HZ 就相当于 2 秒

/* 3. 使用定时器处理按键抖动 */
/* 在 GPIO 中断中并不立刻记录按键值，而是修改定时器超时时间，10ms 后再处理。如果 10ms 内又发生了 GPIO 中断，那就认为是抖动，这时再次修改超时时间为 10ms。只有 10ms 之内再无 GPIO 中断发生，那么定时器的函数才会被调用。在定时器函数中记录按键值。 */

/* 4. 定时器就是通过软件中断来实现的，它属于 TIMER_SOFTIRQ 软中断 */
/* 如何高效地找到超时的timer */
```

### 中断下半部分tasklet

```c
/* 1. 内核函数 */
/* -a. tasklet_struct结构体：
 * --. u64 state有2位，bit0表示 TASKLET_STATE_SCHED，等于 1 时表示已经执行了 tasklet_schedule 把该 tasklet 放入队列了。bit1 表示 TASKLET_STATE_RUN，等于 1 时，表示正在运行 tasklet 中的 func 函数；函数执行完后内核会把该位清 0。
 * --. 其中的 count 表示该 tasklet 是否使能：等于 0 表示使能了，非 0 表示被禁止了。对于 count 非 0 的 tasklet，里面的 func 函数不会被执行。
 */
struct tasklet_struct {
    struct tasklet_struct *next;
    unsigned long state;
    atomic_t count;
    void (*func)(unsigned long);
    unsigned long data;
};
/* --. 使用宏来定义tasklet_struct结构体 */
#define DECLARE_TASKLET(name, func, data) \
struct tasklet_struct name = { NULL, 0, ATOMIC_INIT(0), func, data }  // 使能
#define DECLARE_TASKLET_DISABLED(name, func, data) \
struct tasklet_struct name = { NULL, 0, ATOMIC_INIT(1), func, data }  // 禁止
/* --. 使用函数来初始化tasklet_struct结构体 */
extern void tasklet_init(struct tasklet_struct *t, void (*func)(unsigned long), unsigned long data);

/* -b. 使能/禁止tasklet函数： */
static inline void tasklet_enable(struct tasklet_struct *t);  // count+1
static inline void tasklet_disable(struct tasklet_struct *t);  // count-1

/* -c. 调度tasklet */
/* 把 tasklet 放入链表，并且设置它的 TASKLET_STATE_SCHED 状态为 1。 */
static inline void tasklet_schedule(struct tasklet_struct *t);

/* -d. 删除tasklet */
/* 如果一个 tasklet 未被调度， tasklet_kill 会把它的 TASKLET_STATE_SCHED 状态清 0；如果一个 tasklet 已被调度，tasklet_kill 会等待它执行完华，再把它的 TASKLET_STATE_SCHED 状态清 0。 */
extern void tasklet_kill(struct tasklet_struct *t);

/* -e. tasklet使用方法 */
/* 先定义 tasklet，需要使用时调用 tasklet_schedule，驱动卸载前调用 tasklet_kill，其中tasklet_schedule 只是把 tasklet 放入内核队列，它的 func 函数会在软件中断的执行过程中被调用。
 */

/* 2. tasklet内核代码： */
/* -a. tasklet属于TASKLET_SOFTIRQ软件中断，入口函数为tasklet_action，这在内核 kernel\softirq.c 中设置 */
void __init softirq_init(void)
{
    int cpu;

    for_each_possible_cpu(cpu) {
        per_cpu(tasklet_vec, cpu).tail = &per_cpu(tasklet_vec, cpu).head;
        per_cpu(tasklet_hi_vec, cpu).tail = &per_cpu(tasklet_hi_vec, cpu).head;
    }

    open_softirq(TASKLET_SOFTIRQ, tasklet_action);
    open_softirq(HI_SOFTIRQ, tasklet_hi_action);
}
/* -b. 当驱动程序调用 tasklet_schedule 时，会设置 tasklet 的 state 为 TASKLET_STATE_SCHED，并把它放入某个链表。 */
/* --. tasklet_schedule 调度 tasklet 时，其中的函数并不会立刻执行，而只是把 tasklet 放入队列；
 * --. 调用一次 tasklet_schedule，只会导致 tasklnet 的函数被执行一次
 * --. 如果 tasklet 的函数尚未执行，多次调用 tasklet_schedule 也是无效的，只会放入队列一次
 */
```

### 工作队列

使用“工作队列”，只需要把“工作”放入“工作队列中”，对应的内核线程就会取出“工作”，执行里面的函数。

```c
/* 1. 内核函数 */
/* -a. 定义work_struct结构体，目录：include\linux\workqueue.h */
#define DECLARE_WORK(n, f) \
struct work_struct n = __WORK_INITIALIZER(n, f)
#define DECLARE_DELAYED_WORK(n, f) \
struct delayed_work n = __DELAYED_WORK_INITIALIZER(n, f, 0)
#define INIT_WORK(_work, _func)  // 初始化work_struct结构体

/* -b. 调用schedule_work */
/* 调用 schedule_work 时，就会把 work_struct 结构体放入队列中，并唤醒对应的内核线程。内核线程就会从队列里把 work_struct 结构体取出来，执行里面的函数。 */

/* -c. 其他函数 */
create_workqueue;  // 创建工作队列
create_singlethread_workqueue;
......;
```

### 中断的线程化处理

workqueue和threaded_irq的区别（Gemini）

```c
/* 1. 内核函数 */
/* -a. 注册irq_desc结构体数组，request_threaded_irq，目录：kernel\irq\manage.c */
int request_threaded_irq(unsigned int irq, irq_handler_t handler,
irq_handler_t thread_fn, unsigned long irqflags,
const char *devname, void *dev_id) {
    // 分配、设置一个 irqaction 结构体
    action = kzalloc(sizeof(struct irqaction), GFP_KERNEL);
    if (!action)
        return -ENOMEM;
    action->handler = handler;
    action->thread_fn = thread_fn;
    action->flags = irqflags;
    action->name = devname;
    action->dev_id = dev_id;
    retval = __setup_irq(irq, desc, action); // 进一步处理
}
/* -b. 卸载中断：free_irq */
/* -c. 中断的执行：*/
/* --. __handle_irq_event_percpu函数，目录：kernel\irq\handle.c
 * --.  irq_thread函数，目录：kernel\irq\handle.c
 */
```

### 内存映射mmap（memory map）

？？？
