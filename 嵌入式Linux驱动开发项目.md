# 简历项目介绍

为什么做这个项目？一共几个人做？你是什么角色？你做了哪些部分？你认为最难的地方是什么？如何解决的？最后收获是什么？

自我介绍：

一分钟：

​	面试官您好，我是贾铭杰，是电子科技大学信息与通信工程学院的在读硕士。我对计算机底层工作原理和单片机编程开发感兴趣，我的意愿岗位是嵌入式 Linux 软件开发。

​	在项目经验上，我完成了基于 IMX6ULL 开发板的相机驱动开发项目，实现了多页面逻辑框架、触摸屏输入控制、摄像头模块控制和图片获取及解析。在操作系统底层实现方面，我完成了基于 RISC-V 架构的 xv6 操作系统开发项目，实现了 ... ；此外，我完整实现了 WLAN 组网中吞吐量预测模型搭建项目，实现了 WLAN 实测数据特征提取、 RSSI 序列软硬判决机制和机器学习模型实践应用。

​	我了解计算机底层工作原理，学习了单片机编程和嵌入式 Linux 软件开发技术，希望能在贵公司进一步学习高级的软件开发技术，在嵌入式领域有所贡献。谢谢面试官！

三分钟：

要求：

1. 开场白，名字，学校，专业，应聘岗位，应聘该岗位的原因；
2. 项目经验的简单介绍，简洁地说明自己做了哪些东西；
3. 学习能力的介绍
4. 一分钟（150 - 200 字）和三分钟（450 - 600 字）的自我介绍

## 如何介绍项目经验

说一下你最近的一个项目：

​	【项目基本信息】我完成了基于 IMX6ULL 开发板的相机驱动开发项目，实现了多页面逻辑控制、触控屏输入控制、摄像头驱动开发和图片获取及解析等，用时 1 个月，用到的技术包括 C 语言编程、Linux 系统编程、多线程并发/线程间通信、硬件模块控制、摄像头驱动开发等；

​	【技术细节/引导面试官发问/核实项目细节】多页面逻辑控制和摄像头驱动开发：

​	我学习并二次开发了基于 IMX6ULL 开发板的数码相框软件开发项目，具体来说，在多页面逻辑控制上，采用了多显存分配和状态控制，以及多页面绘制和切换逻辑，实现不同页面的切换；在触控屏输入控制上，实现了主线程“LCD屏页面显示 / 摄像头数据捕获”和子线程“触控屏输入控制”的多线程并发和线程同步功能；在摄像头模块控制上，启动数据流传输并利用队列来管理缓冲区中的数据；在图片获取及解析上；利用深度优先遍历指定目录下的子目录和文件。

​	在开源项目中，我二次开发了摄像头页面，实现了开启摄像头和拍照保存的功能。我学习了 Linux 系统编程（虚拟内存和物理内存管理，多进程/多线程编程，IO 多路复用，网络编程），在实践中应用了多线程处理、硬件模块控制等知识，学习了面向对象编程的嵌入式 Linux 软件开发技术。

​	技术细节：

多页面逻辑框架通过【维护内存空间链表】，通过 malloc() 分配虚拟内存空间，通过状态标记来管理已分配的内存空间，例如空闲内存/用于指定页面显示的内存/用于正在显示页面的内存。如果需要显示一个页面，根据优先级，先查找是否存在用于该页面显示的内存，如果存在，则将内存数据 memset 拷贝到 LCD 屏显存中，完成页面显示；否则查找空闲内存，将页面数据描绘到页面上，并标记为用于该页面显示的内存；如果还是没有找到，则用已经描绘了其他页面的内存来完成页面显示，并修改该内存的状态。

各页面运行和切换逻辑：

主页面：【LCD 屏控制、触控屏输入控制、多线程并发/线程间通信】取出合适的内存空间，描绘主页面待显示数据，并拷贝到 LCD 屏显存中完成主页面显示。主线程持续等待触控屏输入事件，阻塞访问共享资源 (互斥锁，wait() 释放互斥锁、进入休眠、等待信号唤醒)。触控屏输入线程持续等待用户输入，在获取用户输入数据后发送条件信号唤醒主线程，并释放互斥锁；主线程访问共享资源后释放互斥锁，解析输入事件并执行相应的程序/其他页面函数 (switch, case)。

连播页面：【深度优先遍历指定目录下的子目录和文件、解析图片数据、多线程并发/线程间通信】该页面循环获取并显示图片，其中获取图片时通过 scandir() 函数扫描目录中的条目，包括子目录和文件，通过`stat`系统调用查看文件类型，如果是子目录则递归遍历子目录条目，直到达到指定深度或者读取到指定数量文件时返回；如果是文件，则解析图片文件（JPEG 文件解压缩）获取图片数据（记录已经读取到的文件数量 records，再下次获取图片时，先计数到 records，即跳过 records 个文件，之后获取到的是未访问的新文件，直到访问过所有文件，再从头开始读取）。将图片数据拷贝到 LCD 屏显存上完成图片显示。同样地，连播页面每显示一张图片后会阻塞访问共享资源 (互斥锁，但不进入休眠)，如果获取到输入数据，则执行相应程序/其他页面函数。

相机页面：【相机驱动开发、缓冲区内存分配/磁盘文件内存映射、多线程并发/线程间通信】该页面显示摄像头捕获数据，具体来说，打开摄像头设备，通过 `ioctl` 设置参数 (SET_FMT，包括分辨率、输入捕获模式、像素格式)、申请缓冲区 (REQBUFS，`calloc()/malloc()`分配缓冲区结构体 (指针成员) 内存，`mmap()`分配真正的缓冲区内存，此时的虚拟内存空间与 fdVideo 磁盘文件物理内存空间相互关联)、将申请的缓冲区放入 incoming 队列 (QBUF)、开启摄像头数据流 (STREAMON)、循环读取图像数据帧 (`select()`阻塞等待摄像头准备图像数据帧、DQBUF 从 outgoing 队列中获取装有数据的缓冲区、并处理图像数据帧)、最后关闭数据流 (STREAMOFF)、释放缓冲区内存 (`munmap()/free()`)。此外，主线程持续等待触控屏输入事件，阻塞访问共享资源。在获取用户输入数据后，解析输入事件并执行相应的程序/其他页面函数。如果获取到拍照事件，则保存一帧图片到指定目录。

有趣的现象：摄像头捕获数据不是等时捕获，而且快速捕获一段时间后，等待一会儿，再快速捕获。

要求：

1. 项目基本信息（1 分钟）：讲出项目基本情况，比如项目名称，背景，给哪个客户做，完成了基本的事情，做了多久，项目规模多大，用到哪些技术，数据库用什么，然后酌情简单说一下模块。重点突出背景，技术，数据库和其他和技术有关的信息。
2. 我负责的部分：描述我在项目中的角色，要主动说出你做了哪些事情，这部分的描述一定需要和你的技术背景一致。
3. 技术细节：可以描述用到的技术细节，特别是你用到的技术细节，这部分尤其要注意，你说出口的，一定要知道，因为面试官后面就根据这个问的。你如果做了5个模块，宁可只说你能熟练说上口的2个。
4. 热门要素：这部分你风险自己承担，如果可以，不露声色说出一些热门的要素，比如Linux，大数据，大访问压力等。但一旦你说了，面试官就会直接问细节。
5. 隐藏加分项

第一印象：表述能力，引导面试官的提问，核实项目细节

面试官判定简历造假，直接中止面试

### 项目基本信息（1分钟）

要求：控制在1分钟里面，讲出项目基本情况，比如项目名称，背景，给哪个客户做，完成了基本的事情，做了多久，项目规模多大，用到哪些技术，数据库用什么，然后酌情简单说一下模块。重点突出背景，技术，数据库和其他和技术有关的信息。



### 我负责的部分

要求：描述我在项目中的角色，要主动说出你做了哪些事情，这部分的描述一定需要和你的技术背景一致。



### 技术细节

要求：可以描述用到的技术细节，特别是你用到的技术细节，这部分尤其要注意，你说出口的，一定要知道，因为面试官后面就根据这个问的。你如果做了5个模块，宁可只说你能熟练说上口的2个。



### 热门要素

要求：这部分你风险自己承担，如果可以，不露声色说出一些热门的要素，比如Linux，大数据，大访问压力等。但一旦你说了，面试官就会直接问细节。



### 隐藏加分项

不露痕迹地说出面试官爱听的话

在项目介绍的时候（当然包括后继的面试），面试官其实很想要听一些关键点，只要你说出来，而且回答相关问题比较好，这绝对是加分项。我在面试别人的时候，一旦这些关键点得到确认，我是绝对会在评语上加上一笔的。

要求：

1. 考虑到代码的扩展性，有参与框架设计的意识；
2. 有调优意识，能提供监控发现问题点，然后解决；
3. 动手能力强，肯干活，会的东西比较多，团队合作精神比较好；
4. 责任心比较强，能适应大压力的环境；
5. 有主见，能不断探索新的知识。



### 专业技能

尽量写和岗位匹配度高的。如果你应聘的是Linux驱动工程师，可以按照以下方式写。

- 熟悉C/C++语言，熟悉ARM架构和Linux内核移植与驱动开发流程

- 具有一定硬件基础知识，能看懂原理图和Dataheet

- 熟练掌握PCIe, DMA, SMMU, eMMC, DVFS等常用外围接口的驱动开发

  如果你应聘的是嵌入式应用工程师，可以按照以下方式写。

- 熟练Qt与C++编程，熟悉QSS，能够使用样式对界面和控件进行美化处理，可以独立编制定制控件，有良好的产品交互意识

- 熟悉多线程、事件驱动模式下的网络编程，熟悉TcpDump,WireShark等常用调试工具

- 熟悉Tcp/Udp的Socket编程、Http协议、Xml解析，串口等相关知识

  **如果你不知道怎么描述这些技能，可以去智联招聘看看对应岗位的一些要求**。



# 嵌入式Linux驱动开发项目

## 数码相框

### 00 学习目标

知识点：字符编码，矢量字体显示，Makefile，多线程，select，网络编程，链表，指针和libjpeg等

目标：

1. 操作LCD、触摸屏
2. 学习如何实现整个项目
3. 掌握面向对象的模块化编程思想
4. 搭建易扩展的程序框架

### 01 系统框架

1. 弄清需求

2. 设计框架

   -a. 输入进程

   ​	主控线程：得到上报事件，用socket打包发出

   ​	ts线程：使用tslib读取触摸板（ts），封装事件，上报

   ​	按键线程：

   -b. 显示进程

   ​	主控线程：根据得到的事件，决定显示哪一幅图片

   ​	socket线程：接受socket

   ​	上一幅线程：准备好上一幅图片

   ​	当前图片线程：准备好当前图片

   -c. 驱动

   ​	分配内存，DMA操作（发送指定内存到显存中）

3. 编写代码

   ```
   fonts: 		freetype字体显示
   libjpeg: 	jpg2rgb图片显示
   input: 		触摸屏和输入事件管理
   render: 	渲染字体和图片
   page: 		显示页面和执行对应操作
   	管理页面显示
       page_manager.c 		页面管理
       main_page.c 		主页面操作
           1. 显示主页面
           2. 创建Prepare线程
           3. 调用GetInputEvent获得输入事件，并完成相应命令
               --进入浏览模式
               --进入连播模式
               --进入设置Setting
       explore_page.c 		浏览模式
       browse_page.c 		查看图片
       auto_page.c 		自动连播模式
       setting_page.c 		设置
       interval_page.c  	间隔
   include:	头文件
   ```

   

4. 测试

### 02 字符编码

编码表：unicode

编码方式：UTF-8（变长）, UTF-16LE（小端），UTF-16BE（大端）

编译程序时指定字符集

```shell
man gcc , /charset
-finput-charset=charset  表示源文件的编码方式，默认UTF-8
-fexec-charset=charset  表示可执行程序中的编码方式，默认UTF-8
```

### 03 显示点阵字符

#### 上机实验

jz2440开发板与IMX6ULL的上机实验差别很大

一个需要配置、启动新内核（配置、修改内核支持把lcd.c编译进去，并且修改drivers/video/Makefile），而另一个直接运行即可（编译、拷贝到开发板上（home目录？？）、执行）。。。

#### 代码：显示点阵字符

点阵字符文件：lib\fonts\font_8x16.c；数组：static const u8 fontdata_8x16[FONTDATAMAX]，其中，16个u8元素为一组点阵。0x41字符对应的点阵地址：0x41 * 16~0x42 * 16 - 1

在LCD的位置(x, y)显示字符c

```c
// D:\Users\22499\Desktop\嵌入式学习\01_all_series_quickstart\04_嵌入式Linux应用开发基础知识\source\09_show_chinese\show_chinese.c
/* 关键函数 */
void lcd_put_ascii(int x, int y, u8 c);  // 显示ascii字符
u8 *dots = (u8 *)&fontdata_8x16[c*16];  // 字符c的点阵序列
lcd_put_pixel(x+7-b, y+i, 0xffffff);  // 填充8x16点阵
void lcd_put_chinese(int x, int y, u8 *str);  // 显示中文字符

/* 主函数 */
int main(int argc, char **argv);
fd = open('/dev/fb0', O_RDWR);  // 打开framebuffer设备文件
ioctl(fd_fb, FBIOGET_VSCREENINFO, &var);  // 获取可变信息
// mmap用法查询：shell man mmap, 并告知相关头文件
fbmem = (unsigned char *)mmap(NULL , screen_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd_fb, 0);  // 分配地址
lcd_put_ascii(var.xres/2, var.yres/2, 'A');  // 显示字符
printf("chinese code: %02x %02x\n", str[0], str[1]);  // 输出国标字符编码
munmap(fbmem, screen_size);  // 取消已分配的内存地址。
close(fd_fb);  // 关闭文件
```



### 04 矢量字体（freetype）多行字符串显示

把freetype头文件、库文件放到工具链目录，把库文件放到板子上的/lib 或/usr/lib 目录里：

​	freetype依赖于libpng，先编译、安装libpng-1.6.37，p-200

​	交叉编译、安装freetype-2.10.2

```
1. 配置交叉工具链
vim /.bashrc
export ARCH=...
2. 依赖库按照：libpng（万能按照命令）
tar xJf libpng.tar.bz2
echo 'main(){}'| arm-buildroot-linux-gnueabihf-gcc -E -v – 查看头文件< ... >和库文件目录，新加的头文件、库文件都可以放到其中的任意目录中。板子上的系统目录：/lib，/usr/lib目录
头文件目录：/home/book/100ask_imx6ull-sdk/ToolChain/arm-buildroot-linux-gnueabihf_sdk-buildroot/arm-buildroot-linux-gnueabihf/sysroot/usr/include
库文件目录：/home/book/100ask_imx6ull-sdk/ToolChain/arm-buildroot-linux-gnueabihf_sdk-buildroot/arm-buildroot-linux-gnueabihf/sysroot/usr/lib/

./configure --host=arm-buildroot-linux-gnueabihf --prefix=$PWD/tmp
					（交叉编译前缀）						（保存目录）
（前提：libpng-1.6.37目录下有configure文件）
make
make install

3. 交叉编译freetype
tar xjf freetype-2.4.10.tar.bz2 
./configure --host=arm-linux
make
make DESTDIR=$PWD/tmp install （下载后可以看到其中的inc头文件和lib库文件）
注意：查看freetype-2.4.10中的vi docs/INSTALL.CROSS，文档中的配置Configuration指明了--host，安装位置默认:/usr/local（PC机），修改前缀--prefix=，寻找交叉工具编译链（echo $PATH），目录：/usr/local/arm/4.3.2，find -name stdio.h，进入对应目录后，pwd打印绝对路径，库文件目录：find -name lib。由于头文件、库文件的目录不同，所以不能指定一个前缀，而分别复制到目标目录。

4. 编译show_line.c
arm-buildroot-linux-gnueabihf-gcc -o show_line show_line.c -lfreetype
上传到开发板show_line
cp show_line simsun.ttc ~/nfs_rootfs/
运行show_line
[root@100ask:/mnt]# ./show_line ./simsun.ttc 0 0 40
以下是12年视频教程
arm-linux-gcc -finput-charset=GBK -o example1 example1.c  -lfreetype -lm （指定动态库freetype, m(数学库)，动态库在哪儿？，按理说在当前目录。。）
arm-linux-gcc -finput-charset=GBK -o show_font show_font.c  -lfreetype -lm
arm-linux-gcc -finput-charset=GBK -fexec-charset=GBK -o show_font show_font.c -lfreetype -lm （指定字符集GBK）
```



```c
/* 关键函数 */
/* 计算字形边框 */
int compute_string_bbox(FT_Faceface, wchar_t *wstr, FT_BBox  *abbox);
// 设置笔的位置
FT_Set_Transform(face, 0, &pen);
// 加载当前字符的字形
error = FT_Load_Char(face, wstr[i], FT_LOAD_RENDER);
// 得到字形的边界框
error = FT_Get_Glyph(face->glyph, &glyph);
FT_Glyph_Get_CBox(glyph, FT_GLYPH_BBOX_TRUNCATE, &glyph_bbox);

/* 主函数 */
int main(int argc, char **argv);
// 初始化FreeType库
error = FT_Init_FreeType( &library );
// 加载font文件
error = FT_New_Face( library, argv[1], 0, &face );
// 设置渲染的像素大小
FT_Set_Pixel_Sizes(face, font_size, 0);
```

LCD坐标系和笛卡尔坐标系（FreeType）

确定边框和坐标，再描画 / 先描画，再确定边框

流程：打开文件，初始化FreeType，确定边框，计算位置，绘制字符

在官网看官方文档，是从0到1把我们按照第一次接触来教的，会有配套的例程。网上教FreeRTOS的人，遇到解决不了的问题，第一时间是refer to 官方文档。

### 05 显示JPG图片

LCD控制器与LCD的关系；jpg文件解压后得到RGB原始文件；

关键：libjpeg库，如何了解呢？在官网查看API Documentation并学习其中的示例（其中有很多韦老师没讲到的实现细节），其中包括Data former(矩阵)，Decompression operation，Decompression detail

```c
/* 解压缩操作过程（Decompression operation） */

/*
Allocate and initialize a JPEG decompression object    // 分配和初始化一个decompression结构体
Specify the source of the compressed data (eg, a file) // 指定源文件
Call jpeg_read_header() to obtain image info		   // 用jpeg_read_header获得jpg信息
Set parameters for decompression					   // 设置解压参数,比如放大、缩小
jpeg_start_decompress(...); 						   // 启动解压：jpeg_start_decompress
while (scan lines remain to be read)
	jpeg_read_scanlines(...);						   // 循环调用jpeg_read_scanlines
jpeg_finish_decompress(...);						   // jpeg_finish_decompress
Release the JPEG decompression object				   // 释放decompression结构体
*/
```

LCD驱动程序：10.远程打印 --> 12.show_file_input_netprint --> display --> fb.c。没学过噻。。其中的函数直接复制来调用实现LCD屏幕显示。（相比较解压缩，好像LCD驱动程序是大头）

### 06 BMP图像解析

BitMap文件格式：介绍了bmp格式文件头数据内容（BITMAPFILEHEADER），可以通过UE（UltraEdit）打开十六进制bmp文件与文件信息头的格式相互对照，其中，bmp文件的RGB数据是从左下角开始保存（与LCD的左上角开始保存不同）

关键代码：bmp.c, pic_operation.h

```c
/* pic_operation.h */
/* 1. 声明：保存RGB数据的结构体：PixelDatas */
/* 2. 声明：保存bmp文件解析器（函数：从bmp中读取rgb数据）的结构体：PicFileParser */
```

```c
/* bmp.c */
/* 1. 定义bmp文件信息头：BITMAPFILEHEADER和BITMAPINFOHEADER，其中，后者所指内存位于前者末尾，在BitMap文件格式页面中可以找到相关定义 */
/* 2. 定义bmp文件解析器（函数：从bmp中读取rgb数据）的结构体：g_tBMPParser
 * --函数static int GetPixelDatasFrmBMP(unsigned char *aFileHead, PT_PixelDatas ptPixelDatas);
 * ----赋值两个bmp文件信息头，并提取biWidth, biHeight(分辨率)和BitCount(位宽)，依次计算rgb数据大小和行宽。
 * ----将rgb数据从bmp文件信息头中提取出来，并保存在RGB数据结构体：PixelDatas。
 */
```

```c
/* 如果make编译文件时报错：expected '=','asm' or '_attribute_' before 'g_tBMPParser，可能是因为没有包含声明变量的头文件 */
/* 报错：error: expected ')' before 'ptVideoMem'，结构体包含的函数缺少')'，大概率是函数包含未定义参数。将先定义的放到后定义的前方可避免错误 */
/* 函数调用参数时，该参数需要初始化 */
/* Source Insight搜索快捷键:ctrl+F只搜索本文件，ctrl+/搜索所有文件 */
/* 报错：implicit declaration of function，函数未声明 */
/* 报错：control reaches end of non-void function，函数没有返回值 */
/* 报错：/file.h: In function 'ID': error: storage class specified for parameter 'T_FileMap'，在结尾缺少';'就会有些奇怪的问题 */
/* 报错：initialization from incompatible pointer type，不兼容的类型，可能是函数前后的返回类型不一致 */
```

### 07 主页面显示

LCD控制器在内存中分配一份显存FlameBuffer，LCD控制器从FB中读取并显示在LCD上。

为了提高显示速度，先malloc分配一份与FB同样大小的内存，并将全部数据在malloc内存空间中生成，然后拷贝到FB（memcpy）

在写程序时，不要直接使用全局变量，而是通过某个函数来获得全局变量；传入函数的是结构体指针而不是结构体，因为结构体太大，而指针很简单。

iBpp：biBitCount每个像素用多少bit；makefile: obj-y += 添加目录；p前缀表示指针，t前缀表示结构体，a前缀表示数组；函数声明是为了不同文件间调用函数服务的。

```shell
# 开发板上安装驱动程序s3c_ts.ko，设置环境变量（目录）
insmod s3c_ts.ko

export TSLIB_TSDEVICE=/dev/input/event1
export TSLIB_CALIBFILE=/etc/pointercal
export TSLIB_CONFFILE=/etc/ts.conf
export TSLIB_PLUGINDIR=/lib/ts
export TSLIB_CONSOLEDEVICE=none
export TSLIB_FBDEVICE=/dev/fb0

在开发板上：mkdir -p /etc/digitpic/icons
把图标文件放到开发板的/etc/digitpic/icons
```

```
1. 挂载：mount -t nfs -o nolock,vers=3 192.168.5.11:/home/book/nfs_rootfs /mnt
2. 复制：make && cp digitpic ~/nfs_rootfs/
3. 图标路径：sudo cp icons ~/nfs_rootfs/root/etc/digitpic/ -rf
4. 查看设备节点：cat /proc/bus/input/devices
5. tslib安装路径 /home/book/100ask_imx6ull-sdk/Linux-4.9.88/tools/tslib
6. 校准触摸屏：ts_calibrate
7. 关闭黑屏：echo -e "\033[9;0]" > /dev/tty0
ls *free* -d 列出当前目录下文件名包含 "free" 的所有目录
tar xjf ...tax.bz2  解压
find -name "libjpeg" 查看相关文件
ls /dev/event* 确定设备节点
sudo chown book:book etc/ -R 修改文件夹权限属性

# 在交叉编译工具链里面寻找库文件
cd ~/100ask_imx6ull-sdk/ToolChain/arm-buildroot-linux-gnueabihf_sdk-buildroot && find -name "freetype"
```

```
# Ubuntu: 配置交叉编译工具链
2. vim ~/.bashrc
export ARCH=arm
export CROSS_COMPILE=arm-buildroot-linux-gnueabihf-
export PATH=$PATH:/home/book/100ask_imx6ull-sdk/ToolChain/arm-buildroot-linux-gnueabihf_sdk-buildroot/bin
```





## 视频监控

### 00 学习目标

衍生项目：数码相机，远程监控，远程串口，物联网（开关灯），小的扩展（3G上网卡升级为5G上网卡？？）

知识点：UVC驱动，CMOS驱动，WIFI网卡，3G上网卡，ALSA声卡，LCD应用程序

目标：

1. 在网页或者客户端实时观察摄像头数据
2. 掌握几大复杂驱动开发：UVC，CMOS，声卡

### 01 V4L2框架

V4L2: Video For Linux Version 2

字符驱动程序（√）、网卡驱动程序、块驱动程序

APP：open，read，write

驱动：drv_open，drv_read，drv_write

```c
/* 1. 构造file_operations */
.open = drv_open;
.read = drv_read;
/* 2. 注册file_operations */
register_chrdev(主设备号, file_operations, name);
/* cdev：分配cdev, 设置cdev（其中某一项为file_operations），cdev_add */
/* 3. 入口函数调用register_chrdev */
/* 4. 出口函数调用unregister_chrdev */
```

复杂的字符驱动程序采用分层结构，统一架构

```c
/* 1. App: open, read, write */
/* 2. （核心层）内核fbmem.c：a. 构造file_operations; b. register_chrdev; c. 入口、出口函数系统调用 */
/* 3. （驱动）硬件相关：a. 分配fb_info; b. 设置fb_info; c. 注册; d. 硬件相关操作 */
```

1. 虚拟机接入摄像头，dmsg查看内核打印信息，寻找Found UVC
2.  进入内核目录（/work/system/linux-3.4.2/drivers/），搜索grep "Found UVC" * -nR，位于Linux内核程序的uvc_driver.c中
3. **uvc_driver.c**向上注册相应结构体uvc_driver，其中包括**.probe = uvc_probe**，当发现可以支持的设备时就会调用probe函数，.id_table = uvc_ids列举了能够支持的设备。注意到uvc_probe函数中一定包含**file_operations注册**，例如v4l2_device_register，uvc_register_video，video_device_alloc，video_register_device，其中v4l2_device_register并不重要，最重要的是video_register_device，则分配（cdev_alloc()）、设置（cdev->ops = v4l2_fops）、注册（cdev_add）video_device。并通过**video_register_device反推出核心层v4l2_dev.c**

V4L2框架实现细节的理解：代码和文档（Linux内核提供的文档linux-3.4.2\Documentation\video4linux\v4l2-framework.txt，其中具体结构体细节繁琐，而没有对整个框架的解释。此外，合适的文件drivers\media\video\vivi.c（Virtual Video driver））

虚拟视频驱动vivi.c分析（v4l2驱动的实现）（总结）：

1. 分配/设置/注册 v4l2_device：关键函数v4l2_device_register，v4l2_device（为上层提供v4l2_fops.open调用接口）
2. 分配video_device（vfd）：关键函数video_device_alloc
3. 设置vfd->v4l2_dev指向第一步注册的v4l2_device，vfd.fops = &vivi_fops（被上层的v4l2_fops调用），APP可以通过ioctl来设置、获得亮度等信息；在驱动程序里，谁来接收/存储/设置到硬件/提供这些信息呢？在v4l2代码框架中抽象出了v4l2_ctrl来设置属性，由v4l2_ctrl_handler来管理。

```
v4l2_ctrl_handler:
	1. v4l2_ctrl_handler_init	// 初始化
	2. v4l2_ctrl_new_std		// 创建新属性v4l2_ctrl，并放入v4l2_ctrl_handler链表？？
	   v4l2_ctrl_new_custom
	3. 跟vdev相关联
	   dev->v4l2_dev.ctrl_handler = hdl;
	   video_dev->v4l2_dev = v4l2_dev;
	4. 被上层调用
```

分析：

```
vivi_init
    vivi_create_instance
        v4l2_device_register   // 不是主要, 只是用于初始化一些东西，比如自旋锁、引用计数，并没有注册
        video_device_alloc
        // 设置

1. vfd：
	.fops = &viv_fops
	.ioctl_ops 	= &vivi_ioctl_ops,
	.release	= video_device_release,
2. 
	vfd->v4l2_dev = &dev->v4l2_dev;

3. 设置"ctrl属性"(用于APP的ioctl，即是ioctl可以实现的属性)：
	v4l2_ctrl_handler_init(hdl, 11);
	dev->volume = v4l2_ctrl_new_std(hdl, &vivi_ctrl_ops,  // 添加新的标准的ctrl；
            		V4L2_CID_AUDIO_VOLUME, 0, 255, 1, 200);
	dev->brightness = v4l2_ctrl_new_std(hdl, &vivi_ctrl_ops,
            		V4L2_CID_BRIGHTNESS, 0, 255, 1, 127);  // 亮度，则可以通过ioctl来获得、设置亮度
	dev->contrast = v4l2_ctrl_new_std(hdl, &vivi_ctrl_ops,
            		V4L2_CID_CONTRAST, 0, 255, 1, 16); 
    dev->button = v4l2_ctrl_new_custom  // 添加新的客户自定义的ctrl

video_register_device(video_device, type:VFL_TYPE_GRABBER, nr)  // type根据类型设置不同的设备格式
            __video_register_device  // cdev分配、设置、注册流程
                vdev->cdev = cdev_alloc();
                vdev->cdev->ops = &v4l2_fops;
                cdev_add
				video_device[vdev->minor] = vdev;  // 以次设备号为下标，保存vdev设备

                if (vdev->ctrl_handler == NULL)  // 注册vdev->ctrl_handler
                    vdev->ctrl_handler = vdev->v4l2_dev->ctrl_handler;
```

分析vivi.c的open,read,write,ioctl过程（向上提供调用接口，向下逐层调用）

```
1. open
app:     open("/dev/video0",....)  // 应用层调用open()
---------------------------------------------------
drv:     v4l2_fops.v4l2_open  // 驱动层调用注册的v4l2_fops结构体，位于v4l2-dev.c中
            vdev = video_devdata(filp);  // 根据次设备号从数组中得到video_device，而数组在__video_register_device中注册。
            ret = vdev->fops->open(filp);  // 找到设备后调用其中open()函数，而vdev的open函数在video_device_alloc中设置？？
                        vivi_ioctl_ops.open  // v4l2_file_operations结构体，位于vivi.c中
                            v4l2_fh_open  // 硬件相关层提供的open函数

2. read
app:    read ....
---------------------------------------------------
drv:    v4l2_fops.v4l2_read
            struct video_device *vdev = video_devdata(filp);
            ret = vdev->fops->read(filp, buf, sz, off);

3. ioctl  // 比较复杂
app:   ioctl
----------------------------------------------------
drv:   v4l2_fops.unlocked_ioctl
            v4l2_ioctl
                struct video_device *vdev = video_devdata(filp);
                ret = vdev->fops->unlocked_ioctl(filp, cmd, arg);
                            vivi_ioctl_ops.unlocked_ioctl = video_ioctl2
                                video_usercopy(file, cmd, arg, __video_do_ioctl);
                                    __video_do_ioctl
                                        struct video_device *vfd = video_devdata(file);
                                        根据APP传入的cmd来获得、设置"某些属性"

```

变量名称的理解：v4l2_ctrl_new_std()，添加新的标准的ctrl；v4l2_ctrl_new_custom，添加新的客户自定义的ctrl



### 02 虚拟驱动vivi测试

sudo route add default gw 192.168.1.1  # 添加默认路由

uname -a  # 确定ubuntu的内核版本

ls /dev/video*  # 查看设备节点

sudo modprobe vivi  # 安装ubuntu的vivi驱动（各种依赖文件都会安装好）

sudo rmmod vivi

sudo insmod ./vivi.ko  # 安装自己编写的vivi驱动

vivi驱动源码阅读

strace -o xawtv.log xawtv  # 其中包含所有系统调用(open, read, ioctl)

```
// 源码中没有涉及
3. ioctl(4, VIDIOC_G_FMT			// 获取数据格式
4. for()
        ioctl(4, VIDIOC_ENUM_FMT	// 列举数据格式
5. ioctl(4, VIDIOC_QUERYCAP			// 列举性能（capability）
6. ioctl(4, VIDIOC_G_INPUT          // 获得当前使用输入源
7. ioctl(4, VIDIOC_ENUMINPUT		// 列举输入源
8. ioctl(4, VIDIOC_QUERYCTRL		// 查询属性,比如亮度、对比度
9. ioctl(4, VIDIOC_QUERYCAP
10. ioctl(4, VIDIOC_ENUMINPUT

三、根据虚拟驱动vivi的使用过程(strace -o xawtv.log xawtv)彻底分析摄像头驱动（Drv0-v4l2.c，而且ubuntu vivi应用程序与源码有些差别）
// 1~7都是在v4l2_open里调用
1. open
2. ioctl(4, VIDIOC_QUERYCAP

// 3~7 都是在get_device_capabilities里调用
3. for()
        ioctl(4, VIDIOC_ENUMINPUT   // 列举输入源,VIDIOC_ENUMINPUT/VIDIOC_G_INPUT/VIDIOC_S_INPUT不是必需的
4. for()
        ioctl(4, VIDIOC_ENUMSTD     // 列举标准(制式), 不是必需的
5. for()        
        ioctl(4, VIDIOC_ENUM_FMT    // 列举图片数据可支持格式

6. ioctl(4, VIDIOC_G_PARM
7. for()
        ioctl(4, VIDIOC_QUERYCTRL   // 查询属性(比如说亮度值最小值、最大值、默认值)

// 8~10都是通过v4l2_read_attr来调用的（attribute）     
8. ioctl(4, VIDIOC_G_STD			// 获得当前使用的标准(制式), 不是必需的
9. ioctl(4, VIDIOC_G_INPUT			// 当前使用的输入通道
10. ioctl(4, VIDIOC_G_CTRL          // 获得当前属性, 比如亮度是多少

11. ioctl(4, VIDIOC_TRY_FMT         // 试试能否支持某种格式
12. ioctl(4, VIDIOC_S_FMT           // 设置摄像头使用某种格式

// 13~16在v4l2_start_streaming
13. ioctl(4, VIDIOC_REQBUFS         // 请求系统分配缓冲区
14. for()
        ioctl(4, VIDIOC_QUERYBUF    // 查询所分配的缓冲区
        mmap        
15. for ()
        ioctl(4, VIDIOC_QBUF             // 把所有缓冲区放入队列（Queue）        
16. ioctl(4, VIDIOC_STREAMON      // 启动摄像头

// 17里都是通过v4l2_write_attr来调用的
17. for ()
        ioctl(4, VIDIOC_S_CTRL           // 设置属性
    ioctl(4, VIDIOC_S_INPUT             // 设置输入源
    ioctl(4, VIDIOC_S_STD                 // 设置标准(制式), 不是必需的

// v4l2_nextframe > v4l2_queue_all，v4l2_waiton    
18. v4l2_queue_all
      v4l2_waiton    
          for ()
          {
	      // 查询有没有数据，有数据即从队列中取出并读取数据，然后再放入队列
	      // 不断循环这个过程来处理数据
              select(5, [4], NULL, NULL, {5, 0})      = 1 (in [4], left {4, 985979})
              ioctl(4, VIDIOC_DQBUF                // de-queue, 把缓冲区从队列中取出
              // 处理, 之以已经通过mmap获得了缓冲区的地址, 就可以直接访问数据        
              ioctl(4, VIDIOC_QBUF                 // 把缓冲区放入队列
          }

xawtv的几大函数：
1. v4l2_open
2. v4l2_read_attr/v4l2_write_attr  // 读/写属性
3. v4l2_start_streaming
4. v4l2_nextframe/v4l2_waiton

摄像头驱动程序非必需的ioctl:
    /* 用于选择输入源，在xawtv的界面里为video_source */
	.vidioc_enum_input	= vidioc_enum_input,
	.vidioc_g_input 		= vidioc_g_input,
	.vidioc_s_input		= vidio_s_input,

    /* 用于列举、设置、获得TV制式 */
	.vidioc_s_std		= vidioc_s_std,
    /* video_dev:
	.tvnorms			= V4L2_STD_525_60,    // 用于VIDIOC_ENUMSTD
	.current_norm		= V4L2_STD_NTSC_M,  // 用于VIDIOC_G_STD
     */
    /* 查询、获得、设置属性 */
	.vidioc_queryctrl	= vidioc_queryctrl,
	.vidioc_g_ctrl		= vidioc_g_ctrl,
	.vidioc_s_ctrl		= vidioc_s_ctrl,

摄像头驱动程序必需的11个ioctl:
    // 表示它是一个摄像头设备
	.vidioc_querycap      = vidioc_querycap,

    /* 用于列举、获得、测试、设置摄像头的数据的格式 */
	.vidioc_enum_fmt_vid_cap  = vidioc_enum_fmt_vid_cap,
	.vidioc_g_fmt_vid_cap     = vidioc_g_fmt_vid_cap,
	.vidioc_try_fmt_vid_cap   = vidioc_try_fmt_vid_cap,
	.vidioc_s_fmt_vid_cap     = vidioc_s_fmt_vid_cap,

    /* 缓冲区操作: 申请/查询/放入队列/取出队列 */
	.vidioc_reqbufs       = vidioc_reqbufs,
	.vidioc_querybuf      = vidioc_querybuf,
	.vidioc_qbuf          = vidioc_qbuf,
	.vidioc_dqbuf         = vidioc_dqbuf,

	// 启动/停止
	.vidioc_streamon      = vidioc_streamon,
	.vidioc_streamoff     = vidioc_streamoff,	
     
继续分析数据的获取过程：
1. 请求分配缓冲区: ioctl(4, VIDIOC_REQBUFS          // 请求系统分配缓冲区
                        videobuf_reqbufs(队列, v4l2_requestbuffers) // 队列在open函数用videobuf_queue_vmalloc_init初始化
                        // 注意：这个IOCTL只是分配缓冲区的头部信息，真正的缓存还没有分配呢

2. 查询映射缓冲区:
ioctl(4, VIDIOC_QUERYBUF        // 查询所分配的缓冲区
        videobuf_querybuf        	// 获得缓冲区的数据格式、大小、每一行长度、高度            
mmap(参数里有"大小")   // 在这里才分配缓存
        v4l2_mmap
            vivi_mmap
                videobuf_mmap_mapper
                    videobuf-vmalloc.c里的__videobuf_mmap_mapper
                            mem->vmalloc = vmalloc_user(pages);   // 在这里才给缓冲区分配空间

3. 把缓冲区放入队列:
ioctl(4, VIDIOC_QBUF             // 把缓冲区放入队列        
    videobuf_qbuf
        q->ops->buf_prepare(q, buf, field);  // 调用驱动程序提供的函数做些预处理
        list_add_tail(&buf->stream, &q->stream);  // 把缓冲区放入队列的尾部
        q->ops->buf_queue(q, buf);           // 调用驱动程序提供的"入队列函数"
        

4. 启动摄像头
ioctl(4, VIDIOC_STREAMON
    videobuf_streamon
        q->streaming = 1;  // 标志位
        

5. 用select查询是否有数据
          // 驱动程序里必定有: 产生数据、唤醒进程
          v4l2_poll
                vdev->fops->poll
                    vivi_poll   
                        videobuf_poll_stream
                            // 从队列的头部获得缓冲区
                			buf = list_entry(q->stream.next, struct videobuf_buffer, stream);
                            
                            // 如果没有数据则休眠                			
                			poll_wait(file, &buf->done, wait);

    谁来产生数据、谁来唤醒它？
    内核线程vivi_thread每30MS执行一次，它调用（内核线程产生数据/硬件产生数据）
    vivi_thread_tick
        vivi_fillbuff(fh, buf);  // 构造数据 
        wake_up(&buf->vb.done);  // 唤醒进程
          
6. 有数据后从队列里取出缓冲区
// 有那么多缓冲区，APP如何知道哪一个缓冲区有数据？调用VIDIOC_DQBUF
ioctl(4, VIDIOC_DQBUF 
    vidioc_dqbuf   
        // 在队列里获得有数据的缓冲区
        retval = stream_next_buffer(q, &buf, nonblocking);
        
        // 把它从队列中删掉
        list_del(&buf->stream);
        
        // 把这个缓冲区的状态返回给APP
        videobuf_status(q, b, buf, q->type);
        
7. 应用程序根据VIDIOC_DQBUF所得到缓冲区状态，知道是哪一个缓冲区有数据
   就去读对应的地址(该地址来自前面的mmap)

怎么写摄像头驱动程序:
1. 分配video_device:video_device_alloc
2. 设置
   .fops
   .ioctl_ops (里面需要设置11项)
   如果要用内核提供的缓冲区操作函数，还需要构造一个videobuf_queue_ops
3. 注册: video_register_device
```

### 03 摄像头驱动框架

```
写一个USB摄像头驱动程序
1.构造一个usb_driver
2.设置
   probe:
        2.1. 分配video_device:video_device_alloc
        2.2. 设置
           .fops
           .ioctl_ops (里面需要设置11项)
           如果要用内核提供的缓冲区操作函数，还需要构造一个videobuf_queue_ops
        2.3. 注册: video_register_device      
  id_table: 表示支持哪些USB设备      
3.注册： usb_register

UVC: USB Video Class
UVC驱动：drivers\media\video\uvc\

uvc_driver.c分析:
1. usb_register(&uvc_driver.driver);
2. uvc_probe
        uvc_register_video
            vdev = video_device_alloc();
            vdev->fops = &uvc_fops;
            video_register_device

USB摄像头的内部框架
在www.usb.org下载 uvc specification（UVC规格书）,
UVC 1.5 Class specification.pdf : 有详细描述
USB_Video_Example 1.5.pdf    : 有示例

通过VideoControl Interface来控制，
通过VideoStreaming Interface来读视频数据，
VC里含有多个Unit/Terminal等功能模块，可以通过访问这些模块进行控制，比如调亮度。Unit: Select Unit, Process Unit; Terminal: Input Terminal, Camera Terminal, Output Terminal（用于"内""外"连接）
            
分析UVC驱动调用过程：
const struct v4l2_file_operations uvc_fops = {
	.owner		= THIS_MODULE,
	.open		= uvc_v4l2_open,
	.release	= uvc_v4l2_release,
	.ioctl		= uvc_v4l2_ioctl,
	.read		= uvc_v4l2_read,
	.mmap		= uvc_v4l2_mmap,
	.poll		= uvc_v4l2_poll,
};

1. open:
        uvc_v4l2_open
2. VIDIOC_QUERYCAP   // video->streaming->type 应该是在设备被枚举时分析描述符时设置的
		if (video->streaming->type == V4L2_BUF_TYPE_VIDEO_CAPTURE)
			cap->capabilities = V4L2_CAP_VIDEO_CAPTURE
					  | V4L2_CAP_STREAMING;
		else
			cap->capabilities = V4L2_CAP_VIDEO_OUTPUT
					  | V4L2_CAP_STREAMING;
3. VIDIOC_ENUM_FMT // format数组应是在设备被枚举时设置的
        format = &video->streaming->format[fmt->index];
4. VIDIOC_G_FMT
        uvc_v4l2_get_format  // USB摄像头支持多种格式format, 每种格式下有多种frame(比如分辨率)
            	struct uvc_format *format = video->streaming->cur_format;
            	struct uvc_frame *frame = video->streaming->cur_frame;
5. VIDIOC_TRY_FMT
        uvc_v4l2_try_format
            /* format: Check if the hardware supports the requested format. */

        	/* frame: Find the closest image size. The distance between image sizes is
        	 * the size in pixels of the non-overlapping regions between the
        	 * requested size and the frame-specified size.
        	 */
6. VIDIOC_S_FMT  // 只是把参数保存起来(以便调用)，还没有发给USB摄像头
        uvc_v4l2_set_format
            uvc_v4l2_try_format
        	video->streaming->cur_format = format;
        	video->streaming->cur_frame = frame;
7. VIDIOC_REQBUFS  // 申请缓冲区
        uvc_alloc_buffers
           	for (; nbuffers > 0; --nbuffers) {  // 尝试分配n个缓冲区，分配失败则n--
        		mem = vmalloc_32(nbuffers * bufsize);
        		if (mem != NULL)
        			break;
        	}
8. VIDIOC_QUERYBUF  // 查询缓冲区
        uvc_query_buffer
            __uvc_query_buffer
                memcpy(v4l2_buf, &buf->buf, sizeof *v4l2_buf);  // 复制参数
9. mmap  // 根据缓冲区大小（QUERYBUF返回值）分配内存空间
        uvc_v4l2_mmap
            
10. VIDIOC_QBUF  // 放置缓冲区到队列中
        uvc_queue_buffer
        	list_add_tail(&buf->stream, &queue->mainqueue);
        	list_add_tail(&buf->queue, &queue->irqqueue);

11. VIDIOC_STREAMON  // 启动摄像头（启动数据传输）
        uvc_video_enable(video, 1)  // 把所设置的参数发给硬件,然后启动摄像头
            /* Commit the streaming parameters. 提交参数给硬件 */
            uvc_commit_video
                uvc_set_video_ctrl  /* 设置格式format, frame */
                    	ret = __uvc_query_ctrl(video->dev /* 哪一个USB设备 */, SET_CUR, 0,
                    		video->streaming->intfnum  /* 哪一个接口: VS interface */,
                    		probe ? VS_PROBE_CONTROL : VS_COMMIT_CONTROL, data, size,
                    		uvc_timeout_param);
                    
            /* 启动：Initialize isochronous/bulk URBs and allocate transfer buffers. */
            uvc_init_video(video, GFP_KERNEL);
                    uvc_init_video_isoc / uvc_init_video_bulk
                        urb->complete = uvc_video_complete; (收到数据后此函数被调用,它又调用video->decode(urb, video, buf); ==> uvc_video_decode_isoc(实时端点)/uvc_video_encode_bulk(批量端点) => uvc_queue_next_buffer => wake_up(&buf->wait);唤醒应用程序)
                        
                    usb_submit_urb                    	
12. poll
        uvc_v4l2_poll            
            uvc_queue_poll
                poll_wait(file, &buf->wait, wait);  // 休眠等待有数据

13. VIDIOC_DQBUF
        uvc_dequeue_buffer
        	list_del(&buf->stream);

14. VIDIOC_STREAMOFF            
        uvc_video_enable(video, 0);
    		usb_kill_urb(urb);
    		usb_free_urb(urb);
        
分析设置亮度过程：
ioctl: VIDIOC_S_CTRL
            uvc_ctrl_set
            uvc_ctrl_commit  /* 设置后提交 */
                __uvc_ctrl_commit(video, 0);
                    uvc_ctrl_commit_entity(video->dev, entity, rollback);
                			ret = uvc_query_ctrl(dev  /* 哪一个USB设备 */, SET_CUR, ctrl->entity->id  /* 哪一个unit/terminal */, dev->intfnum /* 哪一个接口: VC interface */, ctrl->info->selector, uvc_ctrl_data(ctrl, UVC_CTRL_DATA_CURRENT), ctrl->info->size);
                        
     
总结：
1. UVC设备有2个interface: VideoControl Interface, VideoStreaming Interface
2. VideoControl Interface用于控制，比如设置亮度。它内部有多个Unit/Terminal(在程序里Unit/Terminal都称为entity(实体))
   可以通过类似的函数来访问：
                			ret = uvc_query_ctrl(dev  /* 哪一个USB设备 */, SET_CUR, ctrl->entity->id  /* 哪一个unit/terminal */,
                				dev->intfnum /* 哪一个接口: VC interface */, ctrl->info->selector,
                				uvc_ctrl_data(ctrl, UVC_CTRL_DATA_CURRENT),
                				ctrl->info->size);
3. VideoStreaming Interface用于获得视频数据，也可以用来选择format/frame(VS可能有多种format, 一个format支持多种frame， frame用来表示分辨率等信息)
   可以通过类似的函数来访问：
                    	ret = __uvc_query_ctrl(video->dev /* 哪一个USB设备 */, SET_CUR, 0,
                    		video->streaming->intfnum  /* 哪一个接口: VS */,
                    		probe ? VS_PROBE_CONTROL : VS_COMMIT_CONTROL, data, size,
                    		uvc_timeout_param);
4. 我们在设置FORMAT时只是简单的使用video->streaming->format[fmt->index]等数据，
   这些数据哪来的？
   应是设备被枚举时设置的，也就是分析它的描述符时设置的。

5. UVC驱动的重点在于：
   描述符的分析
   属性的控制: 通过VideoControl Interface来设置
   格式的选择：通过VideoStreaming Interface来设置
   数据的获得：通过VideoStreaming Interface的URB包来获得

```

### 04 分析描述符

电脑通过USB设备的描述符得知它是什么设备。接入USB设备后，电脑会自动安装驱动程序，删除sudo rmmod uvcvideo, sudo insmod ./myuvc.ko, dmsg # 查看系统启动时以及运行时内核产生的大量信息, lsusb -v -d 厂家id # 查看设备描述符

```
设备描述符（usb_device_descriptor）
配置描述符（usb_config_descriptor）
    接口描述符（usb_interface_descriptor），接口是逻辑上的设备
        端点描述符（usb_endpoint_descriptor）
            访问设备时，访问某个接口——接口表示逻辑设备
            传输数据时，读写某个端点——端点是数据
        端点描述符（usb_endpoint_descriptor）
    接口描述符（usb_interface_descriptor）
		
lsusb.c:  /* usbutils-006 */
main
    dumpdev
        dump_device  /* 设备描述符 */
        dump_config  /* 配置描述符 */
        	for (i = 0 ; i < config->bNumInterfaces ; i++)  /* 接口描述符 */
        		dump_interface(dev, &config->interface[i]);
                	for (i = 0; i < interface->num_altsetting; i++)  /* 设置描述符 */
                		dump_altsetting(dev, &interface->altsetting[i]);
                        for (m = 0; m < interface->bNumEndpoints; m++)
                            endpoint = &intf->altsetting[j].endpoint[m].desc;
                            dump_endpoint();
                		
UVC自定义描述符：每个描述符的起始成员是该描述符的长度
buffer = intf->cur_altsetting->extra;
buflen = intf->cur_altsetting->extralen;  /* 总长度 */
printk十六进制的描述符内容，从中分析自定义描述符（extra desc）的类型，输入来源（bsourceID），并以此分析VideoControl框架。
手动分析UVC自定义描述符，用代码实现解析并打印描述符（7th&8th-代码，参考Usbutils-006中的Lsusb.c中dump_videocontrol_interface和dump_videostreaming_interface）。
VideoControl Interface的自定义描述符:
extra buffer of interface 0:
extra desc 0: 0d 24 01 00 01 4d 00 80 c3 c9 01 01 01 
                    VC_HEADER
extra desc 1: 12 24 02                 01 01 02 00 00 00 00 00 00 00 00 03 0e 00 00 
                    VC_INPUT_TERMINAL  ID
extra desc 2: 09 24 03                 02 01 01          00             04         00 
                    VC_OUTPUT_TERMINAL ID wTerminalType  bAssocTerminal bSourceID
extra desc 3: 0b 24 05                 03 01         00 00           02           7f 14 00 
                    VC_PROCESSING_UNIT ID bSourceID  wMaxMultiplier  bControlSize bmControls
extra desc 4: 1a 24 06                 04 ad cc b1 c2 f6 ab b8 48 8e 37 32 d4 f3 a3 fe ec 08            01        03         01 3f 00
                    VC_EXTENSION_UNIT  ID GUID                                            bNumControls  bNrInPins baSourceID

IT(01)  ===>  PU(03)  ===>  EU(04)  ===>  OT(02)
其中，Processing Unit描述符表明了控制功能，Extension Unit描述符表明了厂家扩展的功能

VideoStreaming Interface的自定义描述符:
extra buffer of interface 1:
extra desc 0: 0e 24 01              01 			df 00 81 00 02 02 01 01 01 00 
                    VS_INPUT_HEADER bNumFormats 
extra desc 1: 1b 24 04                     01           05                   59 55 59 32 00 00 10 00 80 00 00 aa 00 38 9b 71  10              01 00 00 00 00 
                    VS_FORMAT_UNCOMPRESSED bFormatIndex bNumFrameDescriptors GUID                                              bBitsPerPixel
extra desc 2: 1e 24 05                     01          00              80 02   e0 01   00 00 ca 08 00 00 ca 08 00 60 09 00                15 16 05 00            01                 15 16 05 00 
                    VS_FRAME_UNCOMPRESSED  bFrameIndex bmCapabilities  wWidth  wHeight                         dwMaxVideoFrameBufferSize  dwDefaultFrameInterval bFrameIntervalType dwMinFrameInterval
                                                                       640x480
extra desc 3: 1e 24 05                     02          00              60 01   20 01   00 80 e6 02 00 80 e6 02 00 18 03 00    15 16 05 00 01 15 16 05 00 
                    VS_FRAME_UNCOMPRESSED                              352x288                                dwMaxVideoFrameBufferSize
extra desc 4: 1e 24 05                     03          00              40 01   f0 00   00 80 32 02 00 80 32 02 00 58 02 00    15 16 05 00 01 15 16 05 00 
                                                                       320x240                                dwMaxVideoFrameBufferSize
extra desc 5: 1e 24 05                     04          00              b0 00   90 00   00 a0 b9 00 00 a0 b9 00 00 c6 00 00    15 16 05 00 01 15 16 05 00 
                                                                       176x144                                dwMaxVideoFrameBufferSize
extra desc 6: 1e 24 05                     05          00              a0 00   78 00   00 a0 8c 00 00 a0 8c 00 00 96 00 00    15 16 05 00 01 15 16 05 00 
                                                                       160x120                                dwMaxVideoFrameBufferSize
extra desc 7: 1a 24 03 00 05 80 02 e0 01 60 01 20 01 40 01 f0 00 b0 00 90 00 a0 00 78 00 00 
                    VS_STILL_IMAGE_FRAME
extra desc 8: 06 24 0d 01 01 04 

VS_INPUT_HEADER 0x01
VS_STILL_IMAGE_FRAME 0x03
VS_FORMAT_UNCOMPRESSED 0x04
VS_FRAME_UNCOMPRESSED 0x05
VS_COLORFORMAT 0x0D
```

### 05 实现数据传输_框架

sudo ifconfig eth3 192.168.1.124  # 配置IP

```
从零写UVC驱动之实现数据传输
A. 设置ubuntu让它从串口0输出printk信息
a. 设置vmware添加serial port, 使用文件作为串口
b. 启动ubuntu，修改/etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"

sudo update-grub
sudo reboot

c. ubuntu禁止root用户登录
先修改root密码: sudo passwd root
然后执行"su root"就可以用root登录了

d. echo "8 4 1 7" > /proc/sys/kernel/printk  # 设置打印级别

再次重启后，只要执行这2个命令就可以：
su root
echo "8 4 1 7" > /proc/sys/kernel/printk

B. 写代码：
1.构造一个usb_driver
2.设置
   probe:
        2.1. 分配video_device:video_device_alloc
        2.2. 设置
           .fops
           .ioctl_ops (里面需要设置11项)
           如果要用内核提供的缓冲区操作函数，还需要构造一个videobuf_queue_ops
        2.3. 注册: video_register_device      
  id_table: 表示支持哪些USB设备      
3.注册： usb_register

C. 上机实验
rmmod myuvc
接入USB到Ubuntu
ls /dev/vid*
rmmod uvcvideo
insmod ./myuvc.ko
 
USB摄像头型号:
a. 视频里用的是: 环宇飞扬 6190 ,它输出的是原始YUV数据，不支持输出MJPEG压缩数据
   大概35元
b. 你也可以使用其它符合UVC规范的摄像头: 就是接到WINDOWS电脑上后不用装驱动的摄像头
   如果你要从零写驱动，就需要你会变通。
c. 我们也会生产一款摄像头, 有2个接口：USB、CMOS(ITU-R BT. 601/656)
   支持输出YUV、MJPEG格式数据, 正在生产调试中, 2013年8月20号左右会在100ask.taobao.com发布
   大概100元
   生产出来后, 我会针对它补录一个视频，现场修改代码

注意：即使不支持MJPEG格式的摄像头，也可以做完项目视频的所有实验，
      只是进行远程视频传输时，需要用软件进行图像压缩，导致视频播放有些卡
```

### 06 实现数据传输_简单函数

参考：/drivers/media/video/uvc目录下的一系列文件

通过error宏定义EIO查找错误

```
/* 在每一行都打印信息，来查找错误位置 */
printk("%s %s %d\n", __FILE__, __FUNCTION__, __LINE__);
```

### 07 实现数据传输_上机实验

```
0. 安装xawtv
	a. 修改镜像源配置文件
		sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup
		sudo vim /etc/apt/sources.list
		添加#  阿里源
	b. sudo apt-get install xawtv
1. 查看printk信息
	a. dmesg # 内核崩溃无法查看printk信息
	b. 设置ubuntu让它从串口0输出printk信息
	c. 设置vmware添加serial port, 使用文件作为串口
    d. 启动ubuntu，修改/etc/default/grub
    GRUB_CMDLINE_LINUX_DEFAULT=""
    GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"
    e. sudo update-grub
       sudo reboot
2. vim ~/.bashrc
export ARCH=arm
export CROSS_COMPILE=arm-buildroot-linux-gnueabihf-
export PATH=$PATH:/home/book/100ask_imx6ull-sdk/ToolChain/arm-buildroot-linux-gnueabihf_sdk-buildroot/bin
3. .ioctl => .unlocked_ioctl
   usb_alloc_urb / usb_alloc_coherent
   usb_free_coherent / usb_buffer_free
4. su # 申请root
5. echo "8 4 1 7" > /proc/sys/kernel/printk  # 设置打印级别
6. make
7. rmmod uvcvideo
8. insmod myuvc.ko
9. 在Ubuntu前台运行xawtv

KERN_DIR = /usr/src/linux-headers-2.6.31-14-generic
```

[ 1005.275298] ---[ end trace fae0d48ce374407b ]---

```
修改
wMaxPacketSize
myuvc_bEndpointAddress
```

### 08 设置摄像头属性

```
sudo modprobe uvcvideo  # 安装Ubuntu自带的摄像头驱动
```

```
八、从零写UVC驱动之实现设置属性(比如亮度)
1. 先看APP以确定需要实现哪些接口
xawtv.c:
    grabber_scan
        ng_vid_open
            v4l2_driver.open // v4l2_open
                get_device_capabilities(h);
                    // 调用VIDIOC_QUERYCTRL ioctl确定是否支持某个属性
                    /* controls */
                    for (i = 0; i < MAX_CTRL; i++) {
                	h->ctl[i].id = V4L2_CID_BASE+i;
                	if (-1 == xioctl(h->fd, VIDIOC_QUERYCTRL, &h->ctl[i], EINVAL) ||
                	    (h->ctl[i].flags & V4L2_CTRL_FLAG_DISABLED))
                	    h->ctl[i].id = -1;
                    }
怎么去获得/设置属性？
看drv0-v4l2.c
可见这2个函数:
v4l2_read_attr  : VIDIOC_G_CTRL
v4l2_write_attr : VIDIOC_S_CTRL

所以: 视频驱动里要实现3个ioctl:
VIDIOC_QUERYCTRL
VIDIOC_G_CTRL
VIDIOC_S_CTRL


2. 硬件上怎么设置属性?
2.1 UVC规范里定义了哪些属性 : uvc_ctrl.c里数组: static struct uvc_control_info uvc_ctrls[]

	{
		.entity		= UVC_GUID_UVC_PROCESSING, // 属于哪了个entity(比如PU)
		.selector	= PU_BRIGHTNESS_CONTROL,   // 用于亮度
		.index		= 0,                       // 对应Processing Unit Descriptor的bmControls[0]
		.size		= 2,                       // 数据长度为2字节
		.flags		= UVC_CONTROL_SET_CUR | UVC_CONTROL_GET_RANGE
				| UVC_CONTROL_RESTORE,
	},


 
2.2 我们的设备支持哪些属性
    这需要去看描述符, 比如 Processing Unit Descriptor的bmControls的值为7f 14
    可知BIT0为1，表示支持BRIGHTNESS
    
    在代码里：
uvc_drvier.c
uvc_ctrl_init_device    
    // 对于每一个entity(IT,PU,SU,OT等)
	list_for_each_entry(entity, &dev->entities, list) {
	    // 取出bmControls
	    bmControls = ....
	    
	    // 计算bmControls里位值为1的个数，就是支持的属性个数
	    ncontrols += hweight8(bmControls[i]);    
	    
	    // 为每一个属性分配一个struct uvc_control
	    entity->controls = kzalloc..
	    
	    // 设置这些struct uvc_control
	    ctrl = entity->controls;
	    for (...)
	    {
    		ctrl->entity = entity;
    		ctrl->index = i;
		}

        // 把uvc_control和uvc_control_info挂构
        uvc_ctrl_add_ctrl(dev, info);
            ctrl->info = 某个uvc_control_info数组项(同属于一个entity, index相同)

2.3 怎么去操作这些属性
    参考 uvc_query_v4l2_ctrl
    uvc_find_control
    	__uvc_find_control
        找到一个uvc_control_mapping结构体: uvc_ctrl.c里有static struct uvc_control_mapping uvc_ctrl_mappings[] 
        	{
        		.id		= V4L2_CID_BRIGHTNESS,  // APP根据ID来找到对应的属性
        		.name		= "Brightness",
        		.entity		= UVC_GUID_UVC_PROCESSING,  // 属于哪了个entity(比如PU)
        		.selector	= PU_BRIGHTNESS_CONTROL,    // 用于亮度
        		.size		= 16,                       // 数据占多少位
        		.offset		= 0,                        // 从哪位开始
        		.v4l2_type	= V4L2_CTRL_TYPE_INTEGER,   // 属性类别
        		.data_type	= UVC_CTRL_DATA_TYPE_SIGNED,// 数据类型
        	},

         uvc_control_mapping结构体 用来更加细致地描述属性，与uvc_control_info结构体一一对应

    uvc_query_ctrl
        usb_control_msg  /* 发起USB控制传输 */


举例说明: 要设置亮度，怎么操作？
a. 根据PU的描述符（extra desc => Head: VC_PROCESSING_UNIT）的bmControls, 从它的bit0等于1知道它支持调节亮度
b. 在uvc_ctrls数组(uvc_control_info结构体)中根据entity和index找到这一项（match）：
	{
		.entity		= UVC_GUID_UVC_PROCESSING,
		.selector	= PU_BRIGHTNESS_CONTROL,
		.index		= 0,
		.size		= 2,
		.flags		= UVC_CONTROL_SET_CUR | UVC_CONTROL_GET_RANGE
				| UVC_CONTROL_RESTORE,
	},

知道了uvc_ctrls[index].flags：这个设备支持SET_CUR, GET_CUR, GET_MIN等
要设置时，可以向PU的selector发数据, 发的数据是2字节

c. 在uvc_ctrl_mappings数组中根据ID找到对应的数组项
   从而知道了更加细致的信息，
   然后使用usb_control_msg读写数据
   

3. 怎么写代码？
实现3个ioctl: vidioc_queryctrl/vidioc_g_ctrl/vidioc_s_ctrl
vidioc_queryctrl : 发起USB控制传输获得亮度的最小值、最大值、默认值、步进值
vidioc_s_ctrl    : 把APP传入的亮度值通过USB传输发给硬件
vidioc_g_ctrl    : 发起USB传输获得当前亮度值

要点：数据发给谁？发给usb_device的
                          VideoControl Interface
                                    里面的Processing Unit 
                                            里面的PU_BRIGHTNESS_CONTROL
```

### 09

```
lsusb -v -d 0x038f:
```

### 10 在LCD上显示摄像头数据_测试

```
cd /usr/local/arm/4.3.2/
grep "V4L2_PIX_FMT_YUYV" * -nR  # 搜索包含宏的头文件

让程序在开发板上直接运行，当它发生错误时，令它产生core dump文件
然后是用gdb根据core dump文件找到发生错误的地方
在ARM板上：
1. ulimit -c unlimited
2. 执行应用程序：程序出错时会在当前目录下生成名为core的文件

在PC上：
3. /bin/arm-linux-gdb ./test_debug ./core
backtrace

ls /dev/vid*
```

## 数码相框扩展项目

### 01 页面切换框图

<img src="C:\Users\22499\AppData\Roaming\Typora\typora-user-images\image-20250303160342727.png" alt="image-20250303160342727" style="zoom:33%;" />

### 02 页面处理逻辑

1. 获取空闲的显存

​	为了使页面之间的切换更加平滑，使用多块显存空间，用于不同页面的显示。在程序开始阶段，会分配若干块显存，每次显示页面之前会根据页面的名字（char转化成int后相加）取出对应内存，如果此块内存的状态已是REDAY，则可以直接写入到lcd显存中，可以减少重新生成图片数据的时间，使页面之间的切换更加平滑。

2. 生成页面显示图标的坐标

3. 获取图片数据后进行缩放操作（zoom）放入指定位置（merge）

4. 刷新显存数据到LCD的FrameBuffer（Fb）

5. 等待输入事件

​	触摸屏控制：输入事件的获取使用多线程方式，子线程进入输入事件的阻塞读取，读取到之后唤醒主线程。在每次获取到触摸屏原始数据后，将按下位置的XY坐标与每个图标区域的起始结束XY坐标进行比较，如果每次按下松开（去除按下后滑动的情况）都是处于同一个图标区域内，则判断按下了此图标。

**文件夹内容信息获取**：使用scandir函数进行获取，

```
int scandir(const char *dir,struct dirent **namelist ,int (*filter)(const void *b),
      int ( * compare )( const struct dirent **, const struct dirent ** ) );
```

​	函数scandir扫描dir目录下(不包括子目录)满足filter过滤模式的文件，返回的结果是使用compare函数经过排序的，并保存在 namelist中。注意namelist是通过malloc动态分配内存的,所以在使用时要注意释放内存。alphasort和versionsort 是使用到的两种排序的函数。 　　
​	当函数成功执行时返回找到匹配模式文件的个数，如果失败将返回-1。namelist内容是dir（dir需要如下这种格式//mnt/才能获取成功）目录下的所有文件夹与文件，包括"."，".."。可使用stat()获取文件信息后，使用S_ISREG()或S_ISDIR()函数判断是文件夹还是文件。

**浏览页面的文件夹显示**：首先使用scandir()获取文件夹内容（第5点），读取到的内容根据文件类型，填充不同的图标，文件夹名字按ASCII码方式进行解析，得到Unicode码后使用freetype库得到位图后显示。每次进行翻页时，获取文件夹在namelist中（第5点）的计数，然后进行显示。

**连播页面图片显示**：使用多线程方式，子线程负责准备图片（预先加载图片数据到指定显存中）、休眠指定间隔时间、显示图片（将准备好的显存数据刷到lcd显存中，如果图片数据未准备好则重新准备），主线程负责接收触摸屏输入事件，在接收到触摸屏事件后，设置互斥量后等待子线程退出。

**连播页面文件获取**：使用深度优先的方式遍历设置的文件夹，最高支持10级递归调用，每次获取10个文件名，然后对获取的文件（绝对路径）进行分析，是否为可支持的显示文件，如果不行就找到一个能显示的为止。

**主界面main_page显存管理**，实现：分配多块显存空间并获取当前页面需要的显存空间，在显存中生成所需图像数据（该视频还未介绍），最后把显存中的图像数据拷贝到FB中实现LCD显示

### 03 项目技术栈整理（CSDN）

项目背景、项目目标、项目任务、项目成果、项目收获

项目目标：数字相框共实现了主页面、文件浏览页面、图片模式浏览页面、图片联播页面、设置页面、联播模式时间间隔设置页面。图片部分支持BMP（自解析）、JPG（JPEG库）格式的图片。所有页面支持触摸屏（tslib库）控制。支持LCD页面显示。支持标准输出（stdout）与网络打印（socket）调试信息。

页面逻辑框架：1. 获取空闲的显存（**显存分配与状态管理，减少生成图片数据的时间，实现页面平滑切换**）；2. 将获取到的图片数据（**解析图片文件，libjpeg库、BitMap Parser**）进行放缩操作（**缩放：近邻取样差值算法**）后放入指定位置中（**写入显存**）；3. 刷新显存数据到LCD的FrameBuffer上（**LCD屏控制**）；4. 等待输入事件

触控屏控制：输入事件的获取使用**多线程方式，子线程进入输入事件的阻塞读取，读取到之后唤醒主线程**。在每次获取到触摸屏原始数据（**触摸屏读取数据控制，相关库文件？**）后，将按下位置的XY坐标与每个图标区域的起始结束XY坐标进行比较，如果每次按下松开（去除按下后滑动的情况）都是处于同一个图标区域内，则判断按下了此图标。

页面的平滑切换（**显存分配与状态管理**）：为了使页面之间的切换更加平滑，使用多块显存空间，用于不同页面的显示。在程序开始阶段，会分配若干块显存，每次显示页面之前会根据页面的名字（char转化成int后相加）取出对应内存，如果此块内存的状态已是REDAY，则可以直接写入到lcd显存中，可以减少重新生成图片数据的时间，使页面之间的切换更加平滑。

**连播页面图片显示：**使用**多线程方式，子线程负责准备图片（预先加载图片数据到指定显存中）、休眠指定间隔时间、显示图片（将准备好的显存数据刷到lcd显存中，如果图片数据未准备好则重新准备），主线程负责接收触摸屏输入事件，在接收到触摸屏事件后，设置互斥量后等待子线程退出**。

**连播页面文件获取**：使用深度优先的方式遍历设置的文件夹（**文件夹内容信息的获取，深度优先遍历文件夹**），最高支持10级递归调用，每次获取10个文件名，然后对获取的文件（绝对路径）进行分析，是否为可支持的显示文件，如果不行就找到一个能显示的为止。

**点击图标按键效果：**当按下图标时，将LCD显存中图标区域内的每个字节数据进行取反处理，代表已经按下了该按键。

浏览页面的文件夹显示：首先使用scandir()获取文件夹内容（第5点），读取到的内容根据文件类型，填充不同的图标，文件夹名字按ASCII码方式进行解析，得到Unicode码后使用freetype库得到位图后显示。每次进行翻页时，获取文件夹在namelist中（第5点）的计数，然后进行显示。

BMP文件解析：

https://www.cnblogs.com/Matrix_Yao/archive/2009/12/02/1615295.html

JPEG文件解析：

https://blog.csdn.net/xipiaoyouzi/article/details/53257720

**设置间隔页面长按累加：**使用tslib库得到的触摸屏原始数据内，会有按下或松开的时间值。在首次接收到按下操作时，记录下此时的触摸屏原始数据（**触摸屏读取数据控制**），将之后按下的时间值与初次记录的值做比较，符合一定时长判断为长按。（如果触摸屏驱动处理长按的方式为使用定时器重复上报输入事件）

打印信息调试：支持**标准输出（stdout）与网络打印（socket）调试信息**。

网络打印选择使用UDP方式进行数据传输（打印数据的数据准确性要求不高），程序中使用了数据发送子进程和数据接收子进程。在每次打印数据时，会先将数据放入循环缓冲区，之后唤醒发送进程，发送进程会查看是否已经有客户端连接，如果已有客户端连接则发送打印数据。接收数据进程一直处于接收客户端信息状态，如果接收到客户端的开始打印信息会设置一个标志位告诉发送线程可以进行发送操作。在开启接收打印数据之前，也可先行设置打印等级，以及开关打印通道。

**遇到的问题：**

使用Scandir()获取文件夹内容，dir需要如下这种格式//mnt/才能获取成功。不然就返回获取到的为0。很奇怪

### 04 项目整理：数码相框

#### A 项目背景

实现了数码相框的主要功能，包括多页面逻辑框架、摄像头控制、图片的获取及解析、LCD屏控制、触摸屏数据读取及处理和调试信息打印。

驱动开发。。不知道该怎么写，在专业技能中写？类似于STM32开发板裸机开发（LED, 按键控制，UART，IIC串口通信）

#### B 主要任务

##### 01 多页面逻辑框架

1. 多页面逻辑框架：该项目分配多块显存用于处理当前页面和等待显示页面，利用**不同显存间的状态管理和LCD屏控制逻辑**，实现多个页面的展示、运行和切换。

具体细节：

多页面逻辑框架通过【维护内存空间链表】，通过 malloc() 分配虚拟内存空间，通过状态标记来管理已分配的内存空间，例如空闲内存/用于指定页面显示的内存/用于正在显示页面的内存。如果需要显示一个页面，根据优先级，先查找是否存在用于该页面显示的内存，如果存在，则将内存数据 memset 拷贝到 LCD 屏显存中，完成页面显示；否则查找空闲内存，将页面数据描绘到页面上，并标记为用于该页面显示的内存；如果还是没有找到，则用已经描绘了其他页面的内存来完成页面显示，并修改该内存的状态。

各页面运行和切换逻辑：

主页面：取出合适的内存空间，描绘主页面待显示数据，并拷贝到 LCD 屏显存中完成主页面显示。主线程持续等待触控屏输入事件，阻塞访问共享资源 (互斥锁，wait() 释放互斥锁、进入休眠、等待信号唤醒)。触控屏输入线程持续等待用户输入，在获取用户输入数据后发送条件信号唤醒主线程，并释放互斥锁；主线程访问共享资源后释放互斥锁，解析输入事件并执行相应的程序/其他页面函数 (switch, case)。

连播页面：该页面循环获取并显示图片，其中获取图片时通过 scandir() 函数扫描目录中的条目，包括子目录和文件，通过`stat`系统调用查看文件类型，如果是子目录则递归遍历子目录条目，直到达到指定深度或者读取到指定数量文件时返回；如果是文件，则解析图片文件（JPEG 文件解压缩）获取图片数据（记录已经读取到的文件数量 records，再下次获取图片时，先计数到 records，即跳过 records 个文件，之后获取到的是未访问的新文件，直到访问过所有文件，再从头开始读取）。将图片数据拷贝到 LCD 屏显存上完成图片显示。同样地，连播页面每显示一张图片后会阻塞访问共享资源 (互斥锁，但不进入休眠)，如果获取到输入数据，则执行相应程序/其他页面函数。

相机页面：该页面显示摄像头捕获数据，具体来说，打开摄像头设备，通过 `ioctl` 设置参数 (SET_FMT，包括分辨率、输入捕获模式、像素格式)、申请缓冲区 (REQBUFS，`calloc()/malloc()`分配缓冲区结构体 (指针成员) 内存，`mmap()`分配真正的缓冲区内存，此时的虚拟内存空间与 fdVideo 磁盘文件物理内存空间相互关联)、将申请的缓冲区放入 incoming 队列 (QBUF)、开启摄像头数据流 (STREAMON)、循环读取图像数据帧 (`select()`阻塞等待摄像头准备图像数据帧、DQBUF 从 outgoing 队列中获取装有数据的缓冲区、并处理图像数据帧)、最后关闭数据流 (STREAMOFF)、释放缓冲区内存 (`munmap()/free()`)。此外，主线程持续等待触控屏输入事件，阻塞访问共享资源。在获取用户输入数据后，解析输入事件并执行相应的程序/其他页面函数。如果获取到拍照事件，则保存一帧图片到指定目录。

有趣的现象：摄像头捕获数据不是等时捕获，而且快速捕获一段时间后，等待一会儿，再快速捕获。

```
多页面逻辑框架：
1. 分配显存空间与状态管理
disp_manager.c
	AllocVideoMem
		struct VideoMem 链表头 g_ptVideoMemHead->ptNext 首先放入FB
		malloc(ptNew->tPixelDatas.iTotalBytes) 分配内存空间，但是FB是g_pucFBMem = (unsigned char *)mmap(NULL , g_dwScreenSize, PROT_READ | PROT_WRITE, MAP_SHARED, g_fd, 0); 因为非FB显存需要将内容copy到FrameBuf中。
			mmap(): 将进程的虚拟内存映射到物理内存区域，实现对外设的内存访问
			malloc(): 仅分配虚拟内存，管理进程的堆内存
		ptNew->iID = 0;	用于页面ID的显存
		ptNew->bDevFrameBuffer = 0; 该显存是不是FrameBuf
		ptNew->eVideoMemState  = VMS_FREE/VMS_USED_FOR_PREPARE/VMS_USED_FOR_CUR; 显存状态管理，空闲/用于（预备）页面ID的显存/用于当前线程的显存
		ptNew->ePicState       = PS_BLANK/PS_GENERATED; 内存图片数据状态

2. 各页面的运行函数
A. 主页面
main_page.c
    MainPageRun
        /* 1. LCD显示主页面 */
        ShowMainPage: 取ID("main")显存，描画图标数据，刷到FB中。
        /* 2. 主线程持续等待输入事件 */
        MainPageGetInputEvent: 触摸板输入数据
            GenericGetInputEvent: 
                GetInputEvent: 
                    static pthread_mutex_t g_tMutex  = PTHREAD_MUTEX_INITIALIZER;
                    static pthread_cond_t  g_tConVar = PTHREAD_COND_INITIALIZER;
                    /* 休眠 */
                	/* 1. 访问共享资源g_tInputEvent之前，线程先尝试获取互斥锁，否则进入阻塞 */
                    pthread_mutex_lock(&g_tMutex);
                	/* 2. 使当前线程进入休眠状态，并释放已经获取的互斥锁，等待其他线程通过pthread_cond_signal 发送信号以唤醒，唤醒后会重新获取互斥锁 */
                    pthread_cond_wait(&g_tConVar, &g_tMutex);	
                	/* 3. 线程被唤醒并重新获取互斥锁后，从共享资源g_tInputEvent中读取输入事件，并存储到ptInputEvent中 */
                    *ptInputEvent = g_tInputEvent;
                	/* 4. 完成对共享资源的访问后，线程释放互斥锁，以便其他线程可以访问共享资源 */
                    pthread_mutex_unlock(&g_tMutex);
                    
B. 连播页面
auto_page.c
	AutoPageRun()
		/* 1. 启动新线程来连续显示图片 */
		pthread_create(&g_tAutoPlayThreadID, NULL, AutoPlayThreadFunction, NULL);
			子线程ID：g_tAutoPlayThreadID
			子线程的线程函数：AutoPlayThreadFunction
				static void *AutoPlayThreadFunction(void *pVoid)
					int bFirst = 1;
                    int bExit;
                    PT_VideoMem ptVideoMem;
                    /* 反复显示下一张图片 */
                    while (1) {
                    /* 1. 读取共享资源 g_bAutoPlayThreadShouldExit 判断是否退出子线程 */
                        pthread_mutex_lock(&g_tAutoPlayThreadMutex);
                        bExit = g_bAutoPlayThreadShouldExit;
                        pthread_mutex_unlock(&g_tAutoPlayThreadMutex);
                        if (bExit) {
                        	return NULL;
                        }
                        /* 2. 准备要显示的图片 */
                        ptVideoMem = PrepareNextPicture(1);
                        /* 3. 如果是第一张图片，则直接显示；否则等待指定时间后再显示 */
                        if (!bFirst) {
                            sleep(g_tPageCfg.iIntervalSecond);
                        }
                        bFirst = 0;
                        /* 刷到FB上去 */
                        FlushVideoMemToDev(ptVideoMem);
                        /* 解放显存 */
                        PutVideoMem(ptVideoMem); 
                    }
```

文件夹目录：取出一张图片，扩展：保存一张图片到指定目录

```
连播页面：取出下一张图片并显示
PrepareNextPicture()
	/* 1. 获取显存，并清屏 */
	ptVideoMem = GetVideoMem(-1, bCur);
	ClearVideoMem(ptVideoMem, COLOR_BACKGROUND);
	/* 2. 连续取出下一张图片并显示 */
	while (1) {
		/* a. 取出下一个图片文件名称 */
		GetNextAutoPlayFile(strFileName);
			/* a1. 取出（最多）10张图片文件名称 g_apstrFileNames，关键变量：g_iStartNumberToRecord，其大小表明了之前已经读取了多少个文件，然后 g_iCurFileNumber >= g_iStartNumberToRecord 接着往后读取文件，直到全部读取完毕，再重新开始读取 g_iStartNumberToRecord = 0。用于依次读取已有10张图片：g_iNextProcessFileIndex */
			GetFilesIndir(g_tPageCfg.strSeletedDir, &g_iStartNumberToRecord, &g_iCurFileNumber, &g_iFileCountHaveGet, g_iFileCountTotal, g_apstrFileNames);
				/* aa. 记录该目录下的所有顶层子目录、顶层文件，并按名称排序 */
				GetDirContents(strDirName, &aptDirContents, &iDirContentsNumber);
					/* aa1. 返回目录中的条目数量 */
					iNumber = scandir(strDirName, &aptNameList, 0, alphasort);
					/* 分配 PT_DirContent 结构体的指针数组内存 */
					PT_DirContent *aptDirContents = malloc(sizeof(PT_DirContent) * (iNumber - 2));
					for (i = 0; i < iNumber-2; i++) {
						aptDirContents[i] = malloc(sizeof(T_DirContent));
					}
					/* aa2. 先从条目名称列表 aptNameList 中取出目录，存入aptDirContents */
					if (isDir(strDirName, aptNameList[i]->d_name)) { -> if ((stat(strTmp, &tStat) == 0) && S_ISDIR(tStat.st_mode)) 获取路径的状态信息 tStat，判断其是否表示一个目录
						/* 存入 aptDirContents，并（删除）aptNameList[i] = NULL */
						strncpy(aptDirContents[j]->strName, aptNameList[i]->d_name, 256);
                        aptDirContents[j]->strName[255] = '\0';
                        aptDirContents[j]->eFileType    = FILETYPE_DIR;
                        free(aptNameList[i]);
                        aptNameList[i] = NULL;
                        j++;
					}
					/* aa3. 再从条目名称列表 aptNameList 中取出常规文件，存入aptDirContents */
					if (isRegFile(strDirName, aptNameList[i]->d_name)) { -> if ((stat(strTmp, &tStat) == 0) && S_ISREG(tStat.st_mode)) 类似目录判断isDir
                        strncpy(aptDirContents[j]->strName, aptNameList[i]->d_name, 256);
                        aptDirContents[j]->strName[255] = '\0';
                        aptDirContents[j]->eFileType    = FILETYPE_FILE;
                        free(aptNameList[i]);
                        aptNameList[i] = NULL;
                        j++;
                    }
                    /* aa4. 释放未使用的条目名称列表，释放未使用的aptDirContents内存 */
                    Free...
                /* ab. 记录 aptDirContents 指针数组中的文件 */
                if (aptDirContents[i]->eFileType == FILETYPE_FILE) {
                	/* 将文件路径记录在 apstrFileNames 二维char数组中 */
                	/* 当 *piFileCountHaveGet >= iFileCountTotal，返回 */
                }
                /* ac. 递归处理 aptDirContents 指针数组中的目录 */
                if (aptDirContents[i]->eFileType == FILETYPE_DIR) {
                	/* 进入下一级目录，记录该级目录的顶层子目录和顶层文件，取出常规文件存入 apstrFileNames 中，递归处理其子目录...直到达到指定文件数量。如果递归深度大于指定深度，则退出。 */
                }
                /* ad. 释放 aptDirContents 指针数组的内存 */
                FreeDirContents(aptDirContents, iDirContentsNumber);
			/* a2. 已有（最多）10张图片文件名称，逐个取出 */
			if (g_iNextProcessFileIndex < g_iFileCountHaveGet) {
                strncpy(strFileName, g_apstrFileNames[g_iNextProcessFileIndex], 256);
                g_iNextProcessFileIndex++;
                return 0;
            }
		/* b. 解析图片文件，获取图片数据 */
		GetPixelDatasFrmFile(strFileName, &tOriginIconPixelDatas);
			/* b1. 打开图片文件，将文件数据mmap到内存中？？为什么不是malloc呢 */
			MapFile(&tFileMap);
			/* b2. 寻找合适的解析器 ptParser */
			ptParser = GetParser(&tFileMap);
				isBMPFormat();
				isJPGFormat();
			/* b3. 解析图片文件，获取图片数据 */
			ptParser->GetPixelDatas(&tFileMap, ptPixelDatas);
				GetPixelDatasFrmBMP();
				GetPixelDatasFrmJPG();
	}
	/* 将图片显示到LCD中 */
	memcpy(FrameBuf, VideoMem, Memlength)...
```



##### 02 摄像头控制

1. 摄像头控制：**阅读Linux内核文档**的V4L2章节及其示例，**独立完成摄像头模块的设置与运行**，包括数据流的启动与关闭、数据缓冲区的分配与管理逻辑、捕获图像数据的解析和LCD屏展示功能。

V4L2框架，启动摄像头、拍照保存图片到指定目录、读取文件夹图片

```
1. 七个示例，逐次实现摄像头模块的各个功能
```

##### 03 图片的获取及解析

1. 图片的获取及解析：阅读libjpeg库文档，**实现摄像头图像数据的解析操作**；利用**深度优先遍历**指定目录下的子目录和文件，实现读取和保存指定图片文件。

解析图片文件：libjpeg库和BitMap Parser

```
1. libjpeg库图像解析
	解压缩操作：细节/大纲
	BGR888 -> RGB888类型转换/格式适配
2. BitMap Parser图像解析
	a. bmp文件信息头
        ptBITMAPFILEHEADER = (BITMAPFILEHEADER *)aFileHead;
        ptBITMAPINFOHEADER =(BITMAPINFOHEADER *)(aFileHead+sizeof(BITMAPFILEHEADER));
    b. bmp文件图像数据
        pucSrc = aFileHead + ptBITMAPFILEHEADER->bfOffBits;
        pucSrc = pucSrc + (iHeight - 1) * iLineWidthAlign;
3. 指定目录的文件读取和保存
	图片的保存格式是什么，有没有用到jpeg压缩操作？？？
```

##### 04 触控屏输入控制

1. 触摸屏输入控制：**主线程“LCD屏页面显示”和子线程“触控屏输入控制”并发运行**，子线程获得输入事件后，**阻塞申请获得互斥锁，处理进程间共享资源，在满足条件后主动唤醒主线程并且释放互斥锁**。之后，主线程申请读取共享资源并进行对应的逻辑处理，实现了**多线程并发运行和相互通信功能**。

子线程获取输入事件后阻塞申请共享资源并告知主线程，以便主线程进行对应的逻辑操作

```
A. 学习资料：
a. tslib.org
b. github tslib -> libts API

B. 触摸屏处理流程
1. InputInit() 定义并注册输入设备（触控屏）
	TouchScreenInit()
		（定义）g_tTouchScreenOpr = {
			.name 			= "touchscreen",
			.DeviceInit 	= TouchScreenDevInit, -> ts_open(), ts_config()
			.GetInputEvent 	= TouchScreenGetInputEvent, -> ts_read(), struct ts_sample, PT_InputEvent->iType, iX, iY, iPressure
		};
		（注册）放入PT_InputOpr链表头：g_ptInputOprHead -> g_tTouchScreenOpr
2. AllInputDevicesInit() 调用输入设备初始化(DeviceInit)，并创建子线程
	（初始化设备）g_ptInputOprHead->DeviceInit()
	（创建子线程）pthread_create(&(ptTmp->tTreadID), NULL, InputEventThreadFunction, ptTmp->GetInputEvent);
		子线程ID：ptTmp->tThreadID
		子线程的线程函数（子线程main函数）：InputEventThreadFunction
		传递给线程函数的参数：ptTmp->GetInputEvent = TouchScreenGetInputEvent
			static void *InputEventThreadFunction(void *pVoid)
				T_InputEvent tInputEvent;
				/* 定义并初始化函数指针GetInputEvent */
                int (*GetInputEvent)(PT_InputEvent ptInputEvent);
                GetInputEvent = (int (*)(PT_InputEvent))pVoid;
                /* 反复尝试读取触控屏数据，读取到则唤醒主线程 */
                while (1) {
                	if (!GetInputEvent(&tInputEvent)) { /* ts_read */
                		/* a. 访问临界资源前，先获得互斥量 */
                        pthread_mutex_lock(&g_tMutex);
                        /* b. 将tInputEvent赋值给共享资源（全局变量） */
                        g_tInputEvent = tInputEvent;
                        /* c. 唤醒主线程 */
                        pthread_cond_signal(&g_tConVar);
                        /* d. 释放互斥量 */
                        pthread_mutex_unlock(&g_tMutex);
                	}
                }
                return NULL;
```



#### C 结果收获

面向对象编程思想，项目框架搭建及其具体实现，嵌入式Linux软件开发经验，Linux内核文档的阅读和学习，多线程处理，嵌入式硬件模块控制

### 05 项目整理：吞吐量预测

#### A 项目背景

基于 WLAN 实测数据，分析 WLAN 网络拓扑、节点间 RSSI、信道接入机制等因素对 WLAN 数据发送、速率的影响，利用机器学习模型实现精准快速的吞吐量预测，为 WLAN 组网优化提供技术提升空间。

#### B 主要任务

01 WLAN 实测数据特征提取

​	该项目按照同频 AP 个数分别进行特征分析，在不同的网络拓扑下，绘制各节点发送机会与所有可能参数的关系曲线，重点关注其中的强相关特征。

02 RSSI 序列软硬判决机制

​	该项目考虑到阈值硬判决会导致阈值附近的 RSSI 序列产生大幅波动，提出通过评估 RSSI 序列与阈值的接近程度，利用阈值软判决机制更平滑地反映数据特征

03 机器学习模型应用

​	基于提取的数据强相关特征，该项目采用随机森林回归模型实现 WLAN 组网吞吐量预测，此外，采用随机森林分类模型实现 WLAN 组网 SINR 水平估计。

#### C 结果收获

实现了较为准确的 WLAN 组网吞吐量预测，荣获国家二等奖，深化了代码编程和项目实现能力。

### 06 简历内容大纲

简历的主要内容大概包括：**嵌入式Linux应用及驱动开发开源项目**的学习（数码相框和摄像头驱动开发），**STM32开发板裸机开发**（LED, 按键控制，UART，IIC串口通信），**第二十一届华为杯数学建模国家二等奖**（通信系统吞吐量预测--基于机器学习模型），**雷达方向SCI 2区论文两篇**，国际会议一篇，挂名专利三篇，一等、**二等优秀学生奖学金**，**双九学历**。



## 操作文档

### 01 学习资料

Linux内核(5.4.0)--V4L2文档

### 02 打开设备

由于硬件的复杂性，V4L2 驱动程序往往非常复杂：大多数设备都有多个 IC，在 /dev 中导出多个设备节点，并且还创建非 V4L2 设备，例如 DVB、ALSA、FB、I2C 和输入（IR）设备。

v4l2-pci-skeleton.c 源代码：V4L2 PCI Skeleton Driver是基于PCI总线的V4L2驱动程序框架，该框架设置了所有驱动程序所需的基本构建块，并且该框架可以更容易地将通用代码重构为所有驱动程序共享的实用函数。

所有驱动程序都具有以下结构：

1. 每个设备实例都有一个包含设备状态的结构。
2. 初始化和命令子设备（如果有）的方法。
3. 创建 V4L2 设备节点（/dev/videoX、/dev/vbiX 和 /dev/radioX）并跟踪设备节点特定数据。
4. 包含每个文件句柄数据的文件句柄特定结构；
5. 视频缓冲区处理。

V4L2框架结构：与驱动程序结构非常相似：它有一个用于设备实例数据的 v4l2_device 结构、一个用于引用子设备实例的 v4l2_subdev 结构、一个用于存储 V4L2 设备节点数据的 video_device 结构以及一个用于跟踪文件句柄实例的 v4l2_fh 结构。

1. 打开/关闭设备

​	设备可以支持多种功能，例如视频输入/输出，ALSA音频采样/播放。每个设备节点仅支持一项功能。

​	open()和close()函数

2. 查询功能

​	所有 V4L2 驱动程序都必须支持ioctl VIDIOC_QUERYCAP。应用程序应始终在打开设备后调用此 ioctl来验证设备。

​	uvc_v4l2.c uvc_ioctl_querycap()

### 03 图像格式

内存中图像的格式和布局：结构体struct v4l2_pix_format(单平面)，struct v4l2_pix_format_mplane(多平面)

图像格式通过VIDIOC_S_FMT ioctl来协商

### 04 V4L2 example 1

参考资料：github: Video for Linux Version 2 (V4L2) examples

#### 代码

```c++
/**
 * 注意：
 *     1. int fd = open(argv[0], O_RDWR);
 		  perror("Can't open Device");
 *     2. struct v4l2_capability caps = {0};
 		  ret = ioctl(fd, VIDIOC_QUERYCAP, &caps);
 *     3. 解析 v4l2_capability 结构体
 */
#include <stdio.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/videodev2.h>
#include <unistd.h>

int main(int argc, char** argv) {
	struct v4l2_capability caps = {0};
    int fd;
    int ret;
    
	if (argc != 2) {
        printf("%s </dev/video0, 1,...>", argv[0]);
        return 0;
    }
    /* 1. 打开设备
     * void perror(const char *s);
     */
    fd = open(argv[1], O_RDWR);
    if (fd < 0) {
        perror("Can't open Device");
        return errno;
    }
    /* 2. ioctl VIDIOC_QUERYCAP
     * #define VIDIOC_QUERYCAP		 _IOR('V',  0, struct v4l2_capability)
     * _IOR: 从驱动程序读取数据
     * 成员.device_caps: 
     This field is only set if the capabilities field contains the V4L2_CAP_DEVICE_CAPS capability.
     */
    ret = ioctl(fd, VIDIOC_QUERYCAP, &caps);
    if (ret == -1) {
        perror("Can't querying device capabilities");
        return errno;
    }
    printf("Driver Caps:\n"
         "  Driver: \"%s\"\n"
         "  Card: \"%s\"\n"
         "  Bus: \"%s\"\n"
         "  Version: %u.%u.%u\n"
         "  Capabilities: %08x\n"
         "  Device_caps: %08x\n",
         caps.driver,
         caps.card,
         caps.bus_info,
         (caps.version >> 16) & 0xFF,
         (caps.version >> 8) & 0xFF,
         (caps.version) & 0XFF,
         caps.capabilities,
         caps.device_caps);

	close(fd);
	return 0;
}

```

#### 结果解析

```
输出结果：
Driver Caps:
  Driver: "uvcvideo"
  Card: "HD Camera: HD Camera"
  Bus: "usb-0000:02:03.0-1"
  Version: 5.4.233
  Capabilities: 84a00001
  Device_caps : 04200001

```

结果解析：

```
v4l2_capability结构体解析：
1. 查询linux文档的 7.48. ioctl VIDIOC_QUERYCAP
Linux Media Infrastructure userspace API -> Part I - Video for Linux API
	7.48.4. Description
2. .capabilities = 84a00001
	V4L2_CAP_DEVICE_CAPS	0x80000000	The driver fills the device_caps field.
	V4L2_CAP_STREAMING		0x04000000	The device supports the streaming I/O method.
	V4L2_CAP_META_CAPTURE	0x00800000	The device supports the Metadata Interface capture interface.
	V4L2_CAP_EXT_PIX_FORMAT	0x00200000	The device supports the struct v4l2_pix_format extended fields.
	V4L2_CAP_VIDEO_CAPTURE	0x00000001	The device supports the single-planar API through the Video Capture interface.
```

### 05 V4L2 example 2

参考资料：github: Video for Linux Version 2 (V4L2) examples

#### 代码

```c++
/** 
 * 注意：
 *     1. struct v4l2_cropcap cropcaps = {0};
 		  ret = ioctl(fd, VIDIOC_CROPCAP, &cropcaps);
 *     2. 解析 v4l2_cropcap 结构体
 */
#include <stdio.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/videodev2.h>
#include <unistd.h>

int main(int argc, char** argv) {
	struct v4l2_cropcap cropcaps = {0};
    int fd;
    int ret;
    
	if (argc != 2) {
        printf("%s </dev/video0, 1,...>", argv[0]);
        return 0;
    }
    /* 1. 打开设备
     * void perror(const char *s);
     */
    fd = open(argv[1], O_RDWR);
    if (fd < 0) {
        perror("Can't open Device");
        return errno;
    }
    /* 2. ioctl VIDIOC_CROPCAP
     * #define VIDIOC_CROPCAP		_IOWR('V', 58, struct v4l2_cropcap)
     * _IOR: 双向数据传输
     * 成员 .type: 
     enum v4l2_buf_type -> V4L2_BUF_TYPE_VIDEO_CAPTURE
     * 成员 .bound:
     v4l2_rect 结构体，表示最大裁剪区域，包括区域的宽，高和左上角坐标
     * 成员 .defrect:
     默认裁剪区域
     * 成员 .aspect:
     v4l2_fract 结构体，表示单个像素的宽高比（正方形：1/1）
     */
	cropcaps.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
	ret = ioctl(fd, VIDIOC_CROPCAP, &cropcaps);
	if (ret == -1) {
		perror("Can't query Device Crop Capabilities");
		return errno;
	}

	printf("Camera Cropping:\n"
	     "  Bounds: width x height = %d x %d, coordinate: (%d, %d)\n"
	     "  Default: width x height = %d x %d, coordinate: (%d, %d)\n"
	     "  Aspect: %d / %d\n",
	     cropcaps.bounds.width, cropcaps.bounds.height, cropcaps.bounds.left, cropcaps.bounds.top,
  	     cropcaps.defrect.width, cropcaps.defrect.height, cropcaps.defrect.left, cropcaps.defrect.top,
	     cropcaps.pixelaspect.numerator, cropcaps.pixelaspect.denominator);  

	close(fd);
	return 0;
}

```

#### 结果解析

```
输出结果：
Camera Cropping:
  Bounds: width x height = 1280 x 720, coordinate: (0, 0)
  Default: width x height = 1280 x 720, coordinate: (0, 0)
  Aspect: 1 / 1

```

### 06 V4L2 example 3

#### 代码

```c++
#include <stdio.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/videodev2.h>
#include <unistd.h>

int main(int argc, char** argv) {
	struct v4l2_fmtdesc fmtdesc = {0};
	struct v4l2_format  fmtset  = {0};
	struct v4l2_format  fmtget  = {0};
    int fd;
    int ret;
    
	if (argc != 2) {
        printf("%s </dev/video0, 1,...>", argv[0]);
        return 0;
    }
    /* 1. 打开设备
     * void perror(const char *s);
     */
    fd = open(argv[1], O_RDWR);
    if (fd < 0) {
        perror("Can't open Device");
        return errno;
    }
    /* 2. ioctl VIDIOC_ENUM_FMT
     * #define VIDIOC_ENUM_FMT		_IOWR('V',  2, struct v4l2_fmtdesc)
     * _IOWR: 双向数据传输
     * 成员 .type: 
           enum v4l2_buf_type
     * 成员 .flags:
          Image Format Description Flags
     * 成员 .description:
          Description of the format
     * 成员 .pixelformat:
     	  Format fourcc, 4个字符
     * 返回值：成功/失败，0/-1
     */
	fmtdesc.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
	while (0 == ioctl(fd, VIDIOC_ENUM_FMT, &fmtdesc)) {

		printf("ENUM FORMAT: \n"
			   "index    : %d \n"
			   "type     : %08x \n"
			   "flags    : %08x \n"
			   "desc     : %s \n"
			   "pixelfmt : %c%c%c%c \n", 
			   fmtdesc.index, fmtdesc.type, fmtdesc.flags, fmtdesc.description,
			   (fmtdesc.pixelformat & 0xFF), (fmtdesc.pixelformat >> 8) & 0xFF, 
			   (fmtdesc.pixelformat >> 16) & 0xFF, (fmtdesc.pixelformat >> 24) & 0xFF);
		printf("flags : %s, %s \n", 
				(fmtdesc.flags & 0x0001) ? "COMPRESSED" : "", 
				(fmtdesc.flags & 0x0002) ? "EMULATED" : "");
		fmtdesc.index++;
	}

	/* 3. ioctl VIDIOC_S_FMT
	 * #define VIDIOC_S_FMT		_IOWR('V',  5, struct v4l2_format)
	 * 成员 .type
	 * 成员 .fmt
	 * 		成员 .pix
	 *		v4l2_pix_format 结构体
	 *      	.width, .height, .pixelformat, .field, .bytesperline
	 * 			.sizeimage, .colorspace, .priv, .flags
	 * 			查询 Image Formats
	 */
	/* 选择视频抓取 */
	fmtset.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
	fmtset.fmt.pix.width  = 640;
	fmtset.fmt.pix.height = 480;
	fmtset.fmt.pix.pixelformat = V4L2_PIX_FMT_YUYV;
	ret = ioctl(fd, VIDIOC_S_FMT, &fmtset);
	if (ret < 0) {
		perror("Can't set V4L2 Format");
		return errno;
	}
	printf("Set V4L2 Format: \n"
			"width    = %d \n"
			"height   = %d \n"
			"pixelfmt = %c%c%c%c \n",
			fmtset.fmt.pix.width, fmtset.fmt.pix.height, 
			(fmtset.fmt.pix.pixelformat & 0xFF), (fmtset.fmt.pix.pixelformat >> 8) & 0xFF, 
			(fmtset.fmt.pix.pixelformat >> 16) & 0xFF, (fmtset.fmt.pix.pixelformat >> 24) & 0xFF);

	/* 4. ioctl VIDIOC_G_FMT
	 * #define VIDIOC_G_FMT		_IOWR('V',  4, struct v4l2_format)
	 * 
	 */
	// memset(&fmtget, 0, sizeof(fmtget));
	fmtget.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
	ret = ioctl(fd, VIDIOC_G_FMT, &fmtget);
	if (ret < 0) {
		printf("Can't get V4L2 Format");
		return errno;
	}
	printf("Get V4L2 Format: \n"
			"width    = %d \n"
			"height   = %d \n"
			"pixelfmt = %c%c%c%c \n",
			fmtget.fmt.pix.width, fmtget.fmt.pix.height, 
			(fmtget.fmt.pix.pixelformat & 0xFF), (fmtget.fmt.pix.pixelformat >> 8) & 0xFF, 
			(fmtget.fmt.pix.pixelformat >> 16) & 0xFF, (fmtget.fmt.pix.pixelformat >> 24) & 0xFF);

	close(fd);
	
	return 0;
}

```



#### 结果解析

```
输出结果：
ENUM FORMAT:
index    : 0
type     : 00000001
flags    : 00000001
desc     : Motion-JPEG
pixelfmt : MJPG
flags : COMPRESSED,
ENUM FORMAT:
index    : 1
type     : 00000001
flags    : 00000000
desc     : YUYV 4:2:2
pixelfmt : YUYV
flags : ,
Set V4L2 Format:
width    = 640
height   = 480
pixelfmt = YUYV
Get V4L2 Format:
width    = 640
height   = 480
pixelfmt = YUYV

MJPEG支持的摄像头分辨率
Width = 1280, Height = 720
Width = 1920, Height = 1080
Width = 640, Height = 480

```



```
结果解析：
1. 枚举pixelformat
2. 设置v4l2_format
3. 读取v4l2_format
```



### 07 V4L2 example 4

参考资料：github V4L2 example 4

​		   Linux-5.4内核文档中的 3.2.1. Example: Mapping buffers in the single-planar API

分配和设置streaming I/O 缓冲区（Buffers）

通过在应用程序和设备驱动程序之间传递指向缓冲区的指针，通过缓冲区交换数据。此方法（传递指针而不是数据本身）使用内存映射来避免在驱动程序和应用程序之间昂贵的数据复制。

缓冲区可以入队，等待从设备接收数据；也可以出队，等待应用程序读取数据。Stackoverflow 上有一张很好的图片，[以状态图的形式说明了缓冲区状态的生命周期](https://stackoverflow.com/questions/10634537/v4l2-difference-between-enque-deque-and-queueing-of-the-buffer)。缓冲区出队后，必须再次入队，驱动程序才能向其中写入数据。一旦将数据写入缓冲区，就不能再向其中写入新数据。

分配设置缓冲区数量和类型：ioctl VIDIOC_REQBUFS

将缓冲区映射到地址空间：mmap()，使用 ioctl VIDIOC_QUERYBUF 确定设备内存中缓冲区的位置，其中得到的 v4l2_buffer 结构体的成员 m.offset 和 length 作为第6和第2个参数传给 mmap()

释放分配的内存（前提：所有缓冲区仍未映射）

#### 代码

```c++
#include <stdio.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/videodev2.h>
#include <unistd.h>
#include <stdlib.h>
#include <assert.h>
#include <string.h>
#include <sys/mman.h>


int main(int argc, char** argv) {
	struct v4l2_requestbuffers bufReq = {0};
	struct v4l2_buffer bufHas = {0};
	struct {
		void*  start;
		size_t length;
	} *buffersHas;		/* recommended */
    int fd;
    int ret;
	int i;
    
	if (argc != 2) {
        printf("%s </dev/video0, 1,...>", argv[0]);
        return 0;
    }
    /* 1. 打开设备
     * void perror(const char *s);
     */
    fd = open(argv[1], O_RDWR);
    if (fd < 0) {
        perror("Can't open Device");
        return errno;
    }
    /* 2. ioctl VIDIOC_REQBUFS
     * 申请分配缓冲区，设置缓冲区数量和类型
	 * #define VIDIOC_REQBUFS		_IOWR('V',  8, struct v4l2_requestbuffers)
     * _IOWR: 双向数据传输
     * 成员 .count: 
          分配的缓冲区数量
     * 成员 .type:
          enum v4l2_buf_type
     * 成员 .memory:
          enum v4l2_memory
     * 成员 .reserved:
     * 返回值：成功/失败，0/-1
     */
	bufReq.count  = 20;
	bufReq.type   = V4L2_BUF_TYPE_VIDEO_CAPTURE;
	bufReq.memory = V4L2_MEMORY_MMAP;
	ret = ioctl(fd, VIDIOC_REQBUFS, &bufReq);
	if (ret < 0) {
		if (errno == EINVAL) {
			printf("Can't support Video Capture or Memory MMap\n");
		}
		else {
			perror("Can't request Buffers");
		}
		return errno;
	}
	printf("Request Buffer Count: %d\n", bufReq.count);

	/* 3. ioctl VIDIOC_QUERYBUF
	 * 查询缓冲区
	 * #define VIDIOC_QUERYBUF		_IOWR('V',  9, struct v4l2_buffer)
	 * 成员 .index:
	 * 成员 .type:
	 * 成员 .bytesused:
	 * 成员 .flags
	 * 成员 .field
	 * 成员 .sequence
	 * ---- memory location
	 * 成员 .memory
	 * 成员 .m
	 			.offset
	 			.fd
	 * 成员 .length
	 */
	
	/* 4. mmap()
	 * 映射缓冲区到虚拟地址空间
	 */
	buffersHas = calloc(bufReq.count, sizeof(*buffersHas));
	assert(buffersHas != NULL);

	for (i = 0; i < bufReq.count; i++) {
		memset(&bufHas, 0, sizeof(bufHas));
		bufHas.index  = i;
		bufHas.type   = V4L2_BUF_TYPE_VIDEO_CAPTURE;
		bufHas.memory = V4L2_MEMORY_MMAP;

		ret = ioctl(fd, VIDIOC_QUERYBUF, &bufHas);
		if (ret < 0) {
			perror("Can't query Buffers");
			return errno;
		}

		buffersHas[i].length = bufHas.length;
		buffersHas[i].start  = mmap(NULL, bufHas.length, 
						PROT_READ | PROT_WRITE, /* recommended */
						MAP_SHARED,				/* recommended */
						fd, bufHas.m.offset); 
		if (buffersHas[i].start == MAP_FAILED) {
			perror("Can't MMap");
			return errno;
		}
	}
	printf("MMap Success! \n");

	/* 5. munmap()
	 * 解除mmap()内存映射
	 */
	for (i = 0; i < bufReq.count; i++) {
		munmap(buffersHas[i].start, buffersHas[i].length);
	}
	free(buffersHas);
	printf("MUNMap Success! \n");

	close(fd);
	
	return 0;
}

```



#### 结果解析

```
输出结果：
Request Buffer Count: 20
MMap Success!
MUNMap Success!
```



```
结果解析：
1. ioctl VIDIOC_REQBUF
2. ioctl VIDIOC_QUERYBUFS
   mmap()
3. munmap()
```



### 08 V4L2 example 5

最初，所有映射的缓冲区都处于出列状态，驱动程序无法访问。对于捕获应用程序，通常首先将所有映射的缓冲区入队，然后开始捕获并进入读取循环。在这里，应用程序等待，直到填满的缓冲区可以出队，并在不再需要数据时将缓冲区重新入队。输出应用程序填充并出队缓冲区，当足够的缓冲区堆积起来时，用 VIDIOC_STREAMON 开始输出。在写循环中，当应用程序用完可用缓冲区时，它必须等待，直到一个空缓冲区可以出队并被重用。

要使缓冲区应用程序入队和出队，请使用 VIDIOC_QBUF 和 VIDIOC_DQBUF ioctl。使用 ioctl VIDIOC_QUERYBUF ioctl 可以随时确定缓冲区的状态是映射、排队、满还是空。有两种方法可以暂停应用程序的执行，直到一个或多个缓冲区可以出队。默认情况下，当输出队列中没有缓冲区时，VIDIOC_DQBUF 会阻塞。当 O_NONBLOCK 标志被赋予 open() 函数时，如果没有可用的缓冲区，VIDIOC_DQBUF 会立即返回一个 EAGAIN 错误代码。select() 或 poll() 函数总是可用的。

要启动和停止捕获或输出应用程序，请调用 VIDIOC_STREAMON 和 VIDIOC_STREAMOFF ioctl。

#### 代码



#### 结果解析

```
输出结果1：
[root@100ask:/mnt]# ./video2lcd /dev/video0
/dev/video0 is not a video capture device
VideoDeviceInit for /dev/video0 error!

```

```
输出结果2：
Driver Caps      :
  Driver                : "usb-0000:02:03.0-1"
  Version               : 5.4.233
  Capabilities  : 84a00001
  Device_caps   : 04200001
  V4L2_CAP_STREAMING     : "Yes"
  V4L2_CAP_VIDEO_CAPTURE : "Yes"
ENUM FORMAT:
  index      : 0
  type       : 00000001
  flags      : 00000001
  desc       : "Motion-JPEG"
  pixelfmt   : "MJPG"
  COMPRESSED : "Yes"
ENUM FORMAT:
  index      : 1
  type       : 00000001
  flags      : 00000000
  desc       : "YUYV 4:2:2"
  pixelfmt   : "YUYV"
  COMPRESSED : "No"
Using the Format YUYV 4:2:2
Set V4L2 Format:
width    = 640
height   = 480
pixelfmt = "YUYV"
Request Buffer Count: 20
....................................................................................................
```



```
结果解析：
1. 报错：摄像头捕获不到数据 Select Timeout: Invalid argument
   解决：虚拟机USB兼容问题，将设置中的USB兼容性改为USB 3.1，摄像头即可正常捕获数据。
```

### 09 V4L2 example 6

```
gdb debug 命令行操作
1. gdb ./video_try
2. br 186
3. run /dev/video0
4. x /4xb buffersHas[bufHas.index].start
5. quit
```

#### 结果解析

```
输出结果：
(gdb) x /16xb buffersHas[bufHas.index].start
0x7ffff7f45000: 0x93    0x46    0x91    0x9b    0x92    0x4b    0x93    0x9b
0x7ffff7f45008: 0x98    0x50    0x98    0x9b    0x8f    0x53    0x95    0x9e

(gdb) x /8xb buffersHas[bufHas.index].start
0x7ffff7eaf000: 0x8c    0x3f    0x95    0xa0    0x95    0x3f    0x90    0x9e

```



```
结果解析：
YUYV格式，Each four bytes (0x__) is two Y’s, a Cb and a Cr. Each Y goes to one of the pixels, and the Cb and Cr belong to both pixels. As you can see, the Cr and Cb components have half the horizontal resolution of the Y component.
```

### 10 V4L2 example 7

获取一张截图 .jpg 文件

使用MJPG图像数据像素格式

#### 代码

```c++
static int V4L2_Processframe(const void* pBuffer, int frameIdx) {
	char filename[20];
	snprintf(filename, sizeof(filename), "./frame_%d.jpg", frameIdx);

	FILE* file = fopen(filename, "w+");
	if (!file) {
		perror("Can't Create .jpg File");
		return errno;
	}
	fwrite(pBuffer, bufHas.length, 1, file);
	fclose(file);

	// saveMJPEG(&t_buffer, mVideoBuffer[t_buffer.index].data, frameId);
	// fputc('.', stdout);		/* 向指定的文件流（标准输出：控制台）写入一个字符 */
	// fflush(stdout);			/* 刷新指定文件流的输出缓冲区，将缓冲区中的数据立即输出到控制台 */
	return 0;
}
```

### 11 libjpeg库

参考资料：github, libjpeg；

​		    libjpeg-turbo

```
1. /usr/share/doc# ls libjpeg*
	libjpeg8: ...
	libjpeg-turbo8: ...
    
2. libjpeg.txt
	Outline of typical usage
		解压缩操作大纲：the rough outline of a JPEG decompression operation is
		
        Allocate and initialize a JPEG decompression object
        Specify the source of the compressed data (eg, a file)
        Call jpeg_read_header() to obtain image info
        Set parameters for decompression
        jpeg_start_decompress(...);
        while (scan lines remain to be read)
                jpeg_read_scanlines(...);  /* Use jpeg12_read_scanlines() for
                                              9-bit through 12-bit data
                                              precision and
                                              jpeg16_read_scanlines() for
                                              13-bit through 16-bit data
                                              precision. */
        jpeg_finish_decompress(...);
        Release the JPEG decompression object

	Basic library usage:
		解压缩细节：Decompression details
a. Allocate and initialize a JPEG decompression object.

b. Specify the source of the compressed data (eg, a file).

c. Call jpeg_read_header() to obtain image info.

d. Set parameters for decompression.

e. jpeg_start_decompress(...);

f. while (scan lines remain to be read)
        jpeg_read_scanlines(...);
        
g. jpeg_finish_decompress(...);

h. Release the JPEG decompression object.

i. Aborting.
```



### 12 LCD显示

```
1. 打开设备 / 设备初始化
	头文件：Linux内核 -- linux\uapi\linux fb.h
	static struct fb_var_screeninfo varFB;
	static struct fb_fix_screeninfo fixFB;
	fdFB = open("/dev/fb0", O_RDWR);
	ret = ioctl(fdFB, FBIOGET_VSCREENINFO, &varFB);
	成员：.xres, .yres, .bits_per_pixel, .dwScreenSize, .iLineWidth, .pucFBMem(.aucPixelDatas), dwLineWidth, .dwPixelWidth
	ret = ioctl(fdFB, FBIOGET_FSCREENINFO, &fixFB);
	.iPixelFormat = (g_ptDefaultDispOpr->iBpp == 16) ? V4L2_PIX_FMT_RGB565 : \
					(g_ptDefaultDispOpr->iBpp == 32) ?  V4L2_PIX_FMT_RGB32 : \
                     0;
	
2. 清屏
	根据.Bpp(.bits_per_pixel)值 = 8, 16, 32
	memset(.pucFBMem, dwBackColor, .dwScreenSize);
	
3. V4L2摄像头启动流程，读取摄像头数据

4. MJPEG -> V4L2_PIX_FMT_RGB32

5. 图像分辨率缩放

6. PicMerge: 将VideoBuf中的数据合并到FrameBuf

7. 下一帧数据
```



#### 结果解析

```
输出结果1：
FB Screen Info:
   .Xres: 1024, .Yres: 600, .ScreenRes: 1474560000, .Bpp : 32
   .colorspace:
Segmentation fault

输出结果2：
FB Screen Info:
   .Xres: 1024, .Yres: 600, .dwScreenSize: 2457600, .Bpp : 32
Set V4L2 Format:
   width          = 640, height   = 480
   pixelfmt = "MJPG"
Request Buffer Count: 20
Application transferred too few scanlines

debug:
varFB.width -> varFB.xres
min_width      = (varFB.xres < cinfo.output_width) ? varFB.xres : cinfo.output_width;
min_height     = (varFB.yres < cinfo.output_height) ? varFB.yres : cinfo.output_height;
```



### 13 触控屏输入控制

显示摄像头页面，触控屏输入控制

启动摄像头数据捕获，实现拍照、相册功能

触控屏按下和松开两次返回数据（简单：只处理松开输入事件）

#### 结果解析

```
# 注意链接动态库 -lts, -lpthread
FB Screen Info:
   .Xres: 1024, .Yres: 600, .dwScreenSize: 2457600, .Bpp : 32
Picture: width = 250, height = 167
Picture: width = 250, height = 167
GetInputEvent: snap!
GetInputEvent: snap!
GetInputEvent: photo!
GetInputEvent: photo!

```



