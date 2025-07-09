[toc]

# 单片机核心/RTOS必备

### 学习思路

—> 每章重点+章与章的串联

—> 艰难自学

—> 架构--基础



### 开发板使用手册

—> 启动开发板

—> 串口连接：通过电脑与开发板交互（传输数据）

—> 烧写整个系统或者更新部分系统

​	—> USB启动模式用来在EMMC或者SD/TF卡上烧写系统

​	—> EMMC和SD/TF卡这两种启动模式前提是已有烧写好的系统

​	—> 烧写：整个系统（映像文件）、设备树（.dtb, device tree biob）、裸机程序（.img, .imx, image:镜像文件）、U-boot（.imx）、Linux内核（zImage）



## 裸机开发

### 启动流程

—> 启动方式：采用引脚BOOT_MODE1、BOOT_MODE0，然后存入BOOT_MODE寄存器。之后这两个引脚可用于其他功能。

| BOOT_MODE[1:0] |                 Boot Type                  |
| :------------: | :----------------------------------------: |
|       00       |          Boot from Fuses--Fuses值          |
|       01       |    Serial Downloader--USB、串口下载程序    |
|       10       | Internal Boot--内部启动，即SD/TF卡或者EMMC |
|       11       |               Reserved--保留               |

—> 10模式：eFUSE为熔丝，只能烧写一次。出厂值为BT_FUSE_SEL=0，此时通过GPIO（General-purpose input/output）的值

—> BOOT_CFG1[7:4]用来选择启动设备，BOOT_CFG1[3:0]用来指定设备参数

​	—> 举例采用SD/TF卡启动

​		a. 设置eFUSE的BOOT_CFG1[7:5]为0b010，对应的GPIO引脚为LCD1_DATA07~05

​		b. 设置BOOT_CFG1[4:0]来获取设备的参数，对应的GPIO引脚为LCD1_DATA04~00

​		c. 设置BOOT_CFG2[4:3]来确定使用哪一个SD/TD卡接口，对应的GPIO引脚为LCD1_DATA12~11

| Boot Device | BOOT_CFG1[7] | BOOT_CFG1[6] | BOOT_CFG1[5] | BOOT_CFG1[4]     | BOOT_CFG1[3]  | BOOT_CFG1[2]  | BOOT_CFG1[1]          | BOOT_CFG1[0]                 |
| ----------- | ------------ | ------------ | ------------ | ---------------- | ------------- | ------------- | --------------------- | ---------------------------- |
| SD/eSD      | 0            | 1            | 0            | Fast boot enable | SD/SDXC speed | SD/SDXC speed | SD power cycle enable | SD loopback clock source sel |
| MMC/eMMC    | 0            | 1            | 1            | Fast boot enable | SD/SDXC speed | SD/SDXC speed | SD power cycle enable | SD loopback clock source sel |

—> IMX6ULL拨码开关表格

| Boot Device | SW1(LCD_DATA5) | SW2(LCD_DATA11) | SW3(BOOT_MODE0) | SW4(BOOT_MODE1) |
| ----------- | -------------- | --------------- | --------------- | --------------- |
| EMMC        | OFF            | OFF             | ON              | OFF             |
| SD/TF       | ON             | ON              | ON              | OFF             |
| USB         | x              | x               | OFF             | ON              |

—> 启动流程

​	a. 复位：Reser

​	b. 检查CPU ID

​	c. 检查Reset Type: 冷启动(Boot) or 唤醒(wakeup)

​	 —> 冷启动：

​		d. 检查Boot Mode(Internal or Serial)(SD/TF卡、EMMC or USB烧写)

​		e. 检查Persistent Bit: Primary Image or Secondary Image，如果检查通过，则执行Image

​			如果失败 && Primary，设置为Persistent Bit = Secondary，再执行SW(Software) Reset

​			如果识别 && Secondary，则从存储Primary Image的存储器Serial EEPROM / 闪存中重新读取、下载、认证。如果失败，则从USB模式读取和下载。

​	—> 唤醒

​		d. 检查电源状态，如果失败，则清除电源低功耗状态（Clear Power Low Power Status）

​		e. 从SRC寄存器(System Reset Controller)使用唤醒程序和参数(Wakeup handle and Argument)

#### 映像文件

—> 映像文件 = (1K数据：内含分区表等信息) + IVT(Image Vector Table) + Boot Data + DCD(Device Configuration Data) + 用户数据（程序本身、bin文件）

​	—> 映像文件放在固定地址（启动设备地址）用于启动

​	—> IVT: 存储一系列地址

​		a. header

​		b. 用户数据（程序）的第1条指令的地址

​		c. DCD地址

​		d. Boot Data地址

​		e. IVT自身的地址

​	—> Boot Data：

​		a. start：映像文件的地址（启动设备数据的起始数据地址）

​		b. length：映像文件的大小

​		c. plugin

​	—> DCD: 保存寄存器值

​		a. Header: tag、length、version

​		b. CMD: Write data command

​			Tag, Length, Parameter

​			Parameter: ==b[2:0]表示写操作的字节数（字节1 byte、半字节2 byte、字4 byte）==

​					     b[7:3]控制命令的标志位

| b[4] "Mask" | b[3] "Set" |        Action        |   Interpretation   |
| :---------: | :--------: | :------------------: | :----------------: |
|      0      |     0      |  *address = val_msk  |  写值 write value  |
|      0      |     1      |  *address = val_msk  |  写值 write value  |
|      1      |     0      | *address &= ~val_msk | 清位 clear bitmask |
|      1      |     1      | *address \|= val_msk |  设位 set bitmask  |

—> 映像文件的制作流程

​	a. 确定入口地址Entry: 用户程序第1条指令的地址

​	b. 映像文件首地址Start: Start = Entry - Load_size

​	Boot ROM程序启动时，在读取映像文件之前，先读取"Initial Load Region"（位于启动设备0地址），包含IVT, Boot data, DCD，读取DCD，初始化设备。

​	不同启动设备对应的"Initial Load Region Size"

​	c. IVT首地址Self: Self = Start + IVT_offset

​	d. Boot Data首地址Boot_data: Boot_data = Self + IVT_size(32 bytes)

​	e. DCD首地址DCD: DCD = Boot_data + Boot_data_size(12 bytes)

​	f. 写入DCD数据（==之前应依次写入IVT, Boot_data数据==）: 

​	设置时钟：DDR需要时钟

​	设置引脚：DDR需要很多引脚

​	设置DDR控制器：Multi-mode DDR controller (MMDC)

​	g. 写入用户数据

​	h. 烧入启动设备

—> 制作映像文件的代码：

```
./tools/mkimage -n ./tools/imximage.cfg.cfgtmp -T imximage -e 0x80100000 -d led.bin led.imx
```

文件：imximage.cfg.cfgtmp

命令mkimage: 把imximage.cfg.cfgtmp中的内容转换为DCD数据

==验证映像文件的内容==

—> 烧写映像文件

==开发板TF卡烧写的映像文件开头的1024字节的0值数据==

​	a. 通过USB下载u-boot到内存

​	b. 通过USB下载程序到内存

​	c. 通过USB输入命令运行u-boot

​	d. 通过u-boot烧写映像文件

​	—> 烧写工具：100ask_imx6ull_flashing_tool

​	—> 通过USB运行裸机程序（无烧写）

​		a. USB启动

​		b. 使用USB线连接电脑--开发板的OTG口

​		c. 运行烧写工具（==开发板复用，确保"设备已连接"==）

​		d. 裸机运行imx文件

​	—> 通过USB烧写SD/TF卡

​		a. USB启动

​		b. c. 

​		d. 烧写img文件到SD/TF卡裸机

​		e. SD/TF卡启动，观察效果

​	—> 通过USB烧写EMMC

### LED程序

CCM: Clock Controller Module中有CCGR: CCM Clock Gating Register

—> LED原理图

​	1--高电平：3.3 V, 1.2 V(三极管)

​	0--低电平

#### —> GPIO引脚操作方法

​	GPIO: General-purpose Input/Output

##### ——> 结构

​	a. 多组GPIO，每组有多个GPIO

​	b. Enable(使能): Power(电源), Clock(时钟)

​	c. Mode(模式): Functionality(功能)

​	d. Direction(方向): Input, Output

​	e. Value: Output, 设置电平(Pad status); Input, 读取寄存器得到引脚的当前电平

##### ——> 寄存器操作

​	a. Enable Register: Power, Clock

​	b. Mode Controller Register

​	c. Direction Controller Register

​	d. Value Register

```
// 修改bit n为1
val = data_reg;
val = val | (1<<n);  // 1<<n得到只有第n位为1的二进制，例如1<<3, 00001000
data_reg = val;
// or
set_reg = 1<<n;

// 删除bit n，即修改bit n为0
val = data_reg;
val = val & ~(1<<n);
data_reg = val;
// or
clr_reg = 1<<n;
```

##### ——> 防抖动、中断、唤醒

##### ——> IMX6ULL的GPIO模块结构

​	GPIO1有32个引脚：GPIO1_IO0~GPIO1_IO31；

​	GPIO2有22个引脚：GPIO2_IO0~GPIO2_IO21；

​	GPIO3有29个引脚：GPIO3_IO0~GPIO3_IO28；

​	GPIO4有29个引脚：GPIO4_IO0~GPIO4_IO28；

​	GPIO5有12个引脚：GPIO5_IO0~GPIO5_IO11；

​	GPIO单元的控制涉及a. 时钟控制单元CCM（使能控制）, IO复用控制单元IOMUXC（包括功能选择和赋值）, GPIO本身（方向寄存器GDIR、输出电平寄存器GPIO_DR、输入电平寄存器GPIO_PSR）

###### 	a. 时钟控制单元CCM（使能控制）

​	CCM_CCGR0寄存器(CCGR: CCM Clock Gating Register)：

​	基地址(Address)：20C_4000h, 偏移(base)：+68h，绝对地址(Offset)：20C_4068h

​	b31-b30(CG15): GPIO2_Clocks(GPIO2_CLK_ENABLE)

​	CCM_CCGR1寄存器：

​	b31-b30(CG15): GPIO5_Clocks(GPIO5_clk_enable)

​	b27-b26(CG13): GPIO1_Clocks(GPIO1_clk_enable)

​	CCM_CCGR2寄存器：

​	b27-b26(CG13): GPIO3_Clocks

​	CCM_CCGR3寄存器：

​	b13-b12(CG6): GPIO4_Clocks

###### 	b. IO复用控制单元IOMUXC(功能选择和赋值)

​		—> 选择功能：

​		IOMUXC_SW_MUX_CTL_PAD_<PADNAME> ：Mux pad xxx，选择某个pad的功能

​		IOMUXC_SW_MUX_CTL_GRP_<GROUP NAME>：Mux grp xxx，选择某组引脚的功能

​		IOMUXC_SW_MUX_CTL_PAD_GPIO1_IO00寄存器：

​		b4(SION, Software Input on Field): 1--Enabled, 强制为输入模式；0--Disabled, 多功能复用

​		b3-b0: 9个功能

| MUX_MODE | Functionality  |
| -------- | -------------- |
| 0101     | ALT5, 用于GPIO |

​		—> 设置电平：

​		IOMUXC_SW_PAD_CTL_PAD_<PAD_NAME>：pad pad xxx，设置某个pad的参数

​		IOMUXC_SW_PAD_CTL_GRP_<GROUP NAME>：pad grp xxx，设置某组引脚的参数

​		IOMUXC_SW_PAD_CTL_PAD_GPIO1_IO00寄存器：

​		设置输入参数：①②③④

​		①b16(HYS. Enabled Field)(输入方式): Hysteresis_Enabled(迟滞)1--Schmitt trigger, 0--CMOS input mode

​		④b15-b14(PUS, Pull Up / Down Config. Field)(设置电平): 00--PUS_0_100K_Ohm_Pull_Down; 

01--PUS_1_47K_Ohm_Pull_Up; 10--PUS_2_100K_Ohm_Pull_Up; 11--PUS_3_22K_Ohm_Pull_Up

​		③b13(PUE, Pull / Keep Select Field)(选择Pull or Keep): 0--PUE_0_Keeper

​		②b12(PKE, Pull / Keep Enabled Field)(使能Pull / Keep):1--PKE_1_Pull_Keeper_Enabled

​		设置输出参数：①②③④

​		①b11(ODE, Open Drain Enabled Field)(推拉or开漏，引脚输出特性): 0--Push-Pull Mode(推拉)，引脚可主动拉高或拉低，标准输出；1--Open-Drain Mode(开漏)，可以拉低，但需外部上拉电阻提高电压

​		③b7-6(SPEED, Speed Field)(输出速率)

​		②b5-b3(DSE, Drive Strength Field)(驱动能力，输出驱动强度):

​		④b0(SRE, Slew Rate Field)(压摆率/转换速率)

###### 		c. GPIO内部单元

​	3个寄存器：

​	① GPIOx_GDIR寄存器：Direction, 1--Output, 0--Input

​	② GPIOx_DR寄存器：Data bits Register, 设置输出引脚的电平，1--高电平, 0--低电平

​	③ GPIOx_PSR寄存器：Pad Status Register, 读取（指示）引脚的电平，1--高电平, 0--低电平

##### ——> GPIO编程方法

​	a. 写GPIO

​	b. 读GPIO

#### —> LED程序

==如何编译？？make为啥报错？==

##### ——> 看原理图确定引脚及操作方法

##### ——> 寄存器操作及代码编程



#### —> 编程知识

 Cortes-A7架构，ARM芯片，精简指令集计算机(Reduced Instruction Set Computing)

| 模式            | 描述                                                   |
| --------------- | ------------------------------------------------------ |
| User            | 用户模式，非特权模式，大部分程序运行的时候就处于此模式 |
| Sys(System)     | 系统模式，用于运行特权级的操作系统任务                 |
| FIQ             | 快速中断模式，进入 FIQ 中断异常                        |
| IRQ             | 一般中断模式                                           |
| ABT(Abort)      | 数据访问终止模式，用于虚拟存储以及存储保护             |
| SVC(Supervisor) | 超级管理员模式，供操作系统使用                         |
| UND(Undef)      | 未定义指令终止模式                                     |
| MON(Monitor)    | 用于安全扩展模式                                       |
| HYP(Hypervisor) | 用于虚拟化扩展                                         |

##### ——> ARM架构

###### ———> 运行模式

运行模式通过软件、中断、异常等方式切换。

###### ———> 寄存器组

每种运行模式拥有一组与之对应的寄存器组

​	a. 未备份寄存器，即R0~R7

​		切换运行模式，R0~R7寄存器中的数据会被破坏，其中数据不会用于特殊用途。

​	b. 备份寄存器，即R8~R14

​		R8~R12，快速中断请求模式(Fast Interrupt Qequest, FIQ)，用于快速执行中断，而不需要保存、恢复R8~R12

​		R13(Stack Point, SP)，栈指针，除了User和Sys模式共用，而其他7个模式各有一个物理寄存器。

​		R14(Link Register, LR)，链接寄存器，除了User、Sys和Hyp模式，而其他6个模式各有一个物理寄存器。存储当前子函数（子程序）的返回地址，在子函数中，将R14(LR)中的值赋给R15(PC)即可完成子函数返回，如mov pc, lr

​	c. 程序计数器，即R15

​		ARM处理器是三级流水线：取指 —> 译码 —> 执行，循环执行（三步同时运行，R15(PC)存放第三条指令的地址）

​		R15(PC) = 当前执行指令地址 + 8个字节

​	d. 程序状态寄存器

​		当前程序状态寄存器CPSR与备份程序状态寄存器SPSR

​		CPSR所有模式共用

​		SPSR，除了Uesr和Sys模式外，其余7个模式各有一个专用的物理状态寄存器，当状态退出时，可以用SPSR恢复CPSR的数据。

​		寄存器结构

| N(b31) | Z    | C    | V    | Q    | IT[1:0] (b26-b25) | J    | Reserved | GE[3:0] (b19-b16) | IT[7:2] (b15-b10) | E    | A    | I    | F    | T    | M[4:0] |
| ------ | ---- | ---- | ---- | ---- | ----------------- | ---- | -------- | ----------------- | ----------------- | ---- | ---- | ---- | ---- | ---- | ------ |
|        |      |      |      |      |                   |      |          |                   |                   |      |      |      |      |      |        |

###### ———> 汇编与机器码，伪指令

汇编指令，对应32位数值的机器码

ARM指令机器码的格式

| 31-28  | 27-26  | 25-23 | 22-21  | 20-16    | 15-12      | 11-8     | 7-4    | 3-0            |
| ------ | ------ | ----- | ------ | -------- | ---------- | -------- | ------ | -------------- |
| 条件码 | 操作码 | S位   | 操作数 | 源寄存器 | 目标寄存器 | 移位类型 | 移位值 | 立即数或偏移量 |

汇编指令的格式

```
label:  // 标签，表示指令/数据地址位置
	instruction @ comment  // 指令，汇编指令或伪指令；注释
```

常用汇编指令

```
mov r1, #10 @ 将立即数10赋值给寄存器r1。机器码：e3a0100a
mov pc, lr @ 机器码：e1a0f00e
```

mov机器码：e3a0100a

| 31-28           | 27-26 | 25   | 24-21 | 20   | 19-16          | 15-12        | 11-0            |
| --------------- | ----- | ---- | ----- | ---- | -------------- | ------------ | --------------- |
| 条件码cond(0xe) | 00    | 1    | 1101  | 0(S) | 0000(Reserved) | Rd(寄存器R0) | imm12(立即数10) |

条件码cond(0xe)：无条件执行；

BL(Branch with Link)机器码：eb000000，跳转并把返回地址记录在LR寄存器

```
bl test_tag  // 跳转并把返回地址记录在LR寄存器。这行地址addrA，pc = addrA+8，lr = addrA+4
			 // 执行指令地址addrA时，译码指令地址addrA+4，pc已经取指令地址addrA+8
mov r1 #10  // r1 = 10。地址addrA+4

test_tag:  // 标签，表示指令/数据地址位置
	mov r3, #0  // r3 = 0。地址addrA+8
	mov pc, lr  // pc = lr。地址addrA+12
```

| 31-28           | 27-24 | 23-0                        |
| --------------- | ----- | --------------------------- |
| 条件码cond(0xe) | 1011  | imm24(PC值与标签的偏移值/4) |

B(Branch)机器码，跳转指令，不保存返回地址

```
b test_tag
mov r1 #10

test_tag:
	mov r3, #0
```

加减(Add/Sub)机器码，Add, e2812004; Sub, e2412004

```
mov r1, #10
add r2, r1, #4  // r2 = r1+4
sub r2, r1, #4  // r2 = r1-4
```

Add，e2812004

| 31-28     | 27-26 | 25   | 24-21 | 20   | 19-16        | 15-12          | 11-0  |
| --------- | ----- | ---- | ----- | ---- | ------------ | -------------- | ----- |
| cond(0xe) | 00    | 1    | 0100  | S    | Rn(源寄存器) | Rd(目标寄存器) | imm12 |

Sub，e2412004

| 31-28     | 27-26 | 25   | 24-21 | 20   | 19-16        | 15-12          | 11-0  |
| --------- | ----- | ---- | ----- | ---- | ------------ | -------------- | ----- |
| cond(0xe) | 00    | 1    | 0010  | S    | Rn(源寄存器) | Rd(目标寄存器) | imm12 |

LDR/STR指令，Load register from memory (读内存的值到寄存器) / Store register to memory (存寄存器的值到内存)

```assembly
mov r0, #400H @ 0x400
mov r1, #aH @ 0xa
str r1, [r0] @ [r0] = r1
ldr r2, [r0] @ r2 = [r0]
```

STR机器码，e5801000

| 31-28     | 27-25 | 24   | 23   | 22   | 21   | 20   | 19-16          | 15-12        | 11-0  |
| --------- | ----- | ---- | ---- | ---- | ---- | ---- | -------------- | ------------ | ----- |
| cond(0xe) | 010   | 1(P) | 1(U) | 0    | 0(W) | 0    | Rn(目标寄存器) | Rt(源寄存器) | imm12 |

LDR机器码，e5902000

| 31-28     | 27-25 | 24   | 23   | 22   | 21   | 20   | 19-16          | 15-12        | 11-0  |
| --------- | ----- | ---- | ---- | ---- | ---- | ---- | -------------- | ------------ | ----- |
| cond(0xe) | 010   | 1(P) | 1(U) | 0    | 0(W) | 1    | Rn(目标寄存器) | Rt(源寄存器) | imm12 |

LDR伪指令，会被拆分成几个真正的ARM指令

```assembly
ldr sp, =0x80200000 @ 立即数(imm32)超出了指令集的直接编码范围，则汇编器在当前代码段附近(PC的偏移量范围内)放置一个文字常量(literal pool，文字池)，然后用LDR指令从该内存位置加载值。
@ 等效于ldr sp, [pc, #-4] @ [pc, #-4] = 0x80200000
@ sp, 堆栈指针寄存器, 指向当前的堆栈位置
```

LDM/STM指令，Load multiple registers(从指定内存起始地址递增/递减把值加载进多个寄存器) / Store Multiple(把多个寄存器的值存储到内存)

```assembly
ldr{cond} Rn{!}, reglist @ cond, 条件; Rn, 基址寄存器，含有内存的起始地址; !, 表示最后地址写回Rn;
str{cond} Rn{!}, reglist

ldr r1, =0x10000000
ldmib r1!, {r0, r4-r6} @ ib, 先增加地址再传输, 指定内存范围0x100000(04-10), 最后地址写回r1 = 0x10000010。 顺序: R0 --> R6, 低地址的值存入低号寄存器
stmda r1!, {r0, r4-r6} @ da, 先传输再减小地址, 指定内存范围0x100000(10-04), 最后地址写回r1 = 0x10000000。 顺序: R6 --> R0, 低号寄存器的值存入低地址

stmfd sp!, {r0-r2} @ 入栈 0x10000000(r2) --> 0x0FFFFFF8(r0)
ldmfd sp!, {r0-r2} @ 出栈 0x0FFFFFF8(r0) --> 0x10000000(r2)
```

###### ———> 进制

|          | 十六进制 | 十进制   | 八进制    | 二进制     |
| -------- | -------- | -------- | --------- | ---------- |
| 汇编语言 | 0xA / AH | 10 / 10D | 012 / 12O | 0b10 / 10B |
| C语言    | 0xA      | 10       | 012       |            |

###### ———> 大小端

小端模式(Little-endian)，数据的低字节保存在内存的低地址中，机器码0x12345678

| 内存地址 | 大端模式 | 小端模式 |
| -------- | -------- | -------- |
| addr+3   | 0x78     | 0x12     |
| addr+2   | 0x56     | 0x34     |
| addr+1   | 0x34     | 0x56     |
| addr     | 0x12     | 0x78     |

###### ———> 位操作

```c
int a = 0x6;
int b = a >> 1;  // 右移, 0b0110 --> 0b0011
int c = a << 1;  // 左移, 0b0110 --> 0b1100
int d = ~a;  // 反转, 0b0110 --> 0b1001
int e = b&c;  // 位与, 0b0000
int f = b|c;  // 位或, 0b1111
int e |= (1<<3);  // 置位, 0b0000 --> 0b1000
int f &= ~(1<<3);  // 清位, 0b1111 --> 0b0111
```

##### ——> 汇编程序调用C程序

###### ———> 寄存器和栈的使用规则

ARM-Thumb procedure call standard (ATPCS, ARM-Thumb过程调用标准)，ARM指令集--Thumb指令集，规定了调用函数如何传递参数，被调用函数如何获取参数，以何种方式传递函数返回值。

寄存器R0~R15在ATPCS规则的使用：

​	l 在函数中，通过寄存器R0~R3来传递参数，被调用的函数在返回前无需恢复寄存器R0~R3的内容。当参数大于4个时，多余参数传入栈中，最后一个参数先入栈（最后出栈）

​	l 在函数中，通过寄存器R4~R11来保存局部变量

​	l 寄存器R12用作函数间scratch寄存器

​	l 寄存器R13用作栈指针，记作SP，在函数中寄存器R13不能用做其他用途，寄存器SP在进入函数时的值和退出函数时的值必须相等

​	l 寄存器R14用作链接寄存器，记作LR，它用于保存函数的返回地址，如果在函数中保存了返回地址，则R14可用作其它的用途

​	l 寄存器R15是程序计数器，记作PC，它不能用作其他用途

传递函数返回值

​	l 结果为一个32位的整数时，通过寄存器R0返回

​	l 结果为一个64位整数时，通过R0和R1返回，依此类推.

​	l 结果为一个浮点数时，通过浮点运算部件的寄存器f0，d0或s0返回

​	l 结果为一个复合的浮点数时，通过寄存器f0-fN或者d0~dN返回

​	l 对于位数更多的结果，通过调用内存来传递

栈的作用

​	a. 保存现场/上下文

​	保存一些寄存器的数值（因为在调用函数后，仍会使用这些寄存器而破坏现场），通过push指令将对应的寄存器的数值存入栈中。

​	恢复现场（待被调用的子函数执行完毕后），通过pop指令将栈中的值赋值给对应的寄存器

​	b. 传递参数

​	超过4个的函数参数通过栈的方式传递

###### ———> C语言读写寄存器

```c
// 找到寄存器地址，通过指针指向寄存器单元，通过读写指针指向的值，来实现读写寄存器

// volatile关键字--编译器不优化此指针
volatile unsigned int *CCM_CCGR1 = (volatile unsigned int *)(0x20C406C);
val = *CCM_CCGR1;  // 读寄存器
*CCM_CCGR1 |= (3 << 30);  // 写寄存器，在b31-b30位置置1
```

###### ———> start.S文件

```assembly
.text @ (汇编系统预定义段名)，表示代码段
.global  _start @ 表示_start是一个全局符号
_start: @ 汇编程序的默认入口, 可以在链接脚本中使用ENTRY来指明其他的入口点

//设置栈
	ldr  sp,=0x80200000

	bl clean_bss
	
	bl main

halt:
	b  halt 

clean_bss:
	/* 清除BSS段 */
	ldr r1, =__bss_start
	ldr r2, =__bss_end
	mov r3, #0
clean:
	str r3, [r1] @ [r1] = r3 r1指向的地址赋值为0
	add r1, r1, #4 @ r1 = r1+#4  起始r1 = __bss_start
	cmp r1, r2
	bne clean @ ne：不一样就继续执行标签clean
	
	mov pc, lr

```

###### ———> led.ids文件

bootRom程序：读取led.img文件，并按照流程运行

RAM(Random Access Memory, 随机存取存储器)：快速读写、临时存储、芯片内部。

Led.ids文件格式

C语言代码: delay(100000);

| 内存地址 | 机器码    | 指令                                                         |
| -------- | --------- | ------------------------------------------------------------ |
| 8010017a | f244 2040 | movw r0, #16960  @ 立即数加载于寄存器的低16位                |
| 8010017e | f2c0 000f | movt r0, #15  @ 立即数加载于寄存器的高16位, 0x000F4240 = 100000 |
| 80100182 | f7ff ffe3 | bl 8010014c <delay>                                          |

机器码存储形式：小端模式(Little-endian)

| 地址              | 机器码      |
| ----------------- | ----------- |
| 8010017a          | 40          |
| 8010017b          | 20          |
| 8010017c          | 44          |
| 8010017d (高地址) | f2 (高字节) |

#### —> Makefile与GCC

##### ——> 交叉编译过程

① 预处理：预处理命令"#include, #define, #if, #ifdef"，分别插入头文件、展开宏定义、条件编译命令

```makefile
$ gcc -E main.c -o main.i
```

② 编译：将C源码"翻译"为汇编代码

```makefile
$ gcc -S main.c -o main.s
```

③ 汇编：将汇编代码"翻译"为机器代码

```makefile
$ gcc -c main.c [-Wall] -o main.o
```

④ 链接

```makefile
# 静态库的制作和运行
$ gcc add.c -o add.o -c
$ ar -rc libadd.a add.o  # 静态库名字一般为"libxxx.a"
$ gcc main.c -o output -ladd -L.  # -L. 表示库在当前目录(.)
# 动态库
$ gcc -shared -fPIC lib.c -o libtest.so  
$ sudo cp  libtest.so /usr/lib/  # 动态库名字一般为"libxxx.so"
$ gcc main.c -L. -ltest -o output
```



GDB调试

① 产生调试信息

````makefile
$ gcc main.c -g -o main.c
````

② 运行调试

```makefile
$ gdb -q main		<---进入调试程序
(gdb) run			<---开始执行程序
(gdb) list 1		<---列出第一行附近的源码，每次10行
(gdb) <Enter>		<---按Enter键，列出下10行源码
(gdb) break 7		<---设置断点，第7行
(gdb) info break	<---查看断点命令
(gdb) delete breakpoint 1	<---删除断点
(gdb) print a		<---显示变量值
(gdb) step 7		<---单步执行命令
(gdb) continue		<---继续执行命令
```

##### ——> Makefile语法

```makefile
DIR = ./100/ask/	# 定义变量，大写
FOO = $(DIR)		# 变量取值，指向DIR变量地址的指针
FOO_CONST := $(DIR) # 即时变量
FOO ?= 100			# 如果变量已有值，则不执行

# 系统自带变量，默认值可修改，例如CC = gcc, PWD, CLFAG
# 1）CPPFLAGS：预处理器需要的选项，如：-l
# 2）CFLAGS：编译的时候使用的参数，-Wall -g -c
# 3）LDFLAGS：链接库使用的选项，-L -l

# 自动变量
# 1）$@：规则中的目标
# 2）$<：规则中的第一个依赖文件(OBJECT)
# 3）$^：规则中的所有依赖文件

# 模式规则
# %.o %.c：匹配各自的文件，如main.o -- main.c, add.o -- add.c

# 伪目标：目标和实际文件名重合
.PHONY:clean
clean:
        rm $(OBJ) output

```



```makefile
# 变量(指针)
SOURCE = $(wildcard ./src/*.c)  # 格式为./src/*.c的文件
OBJECT = $(patsubst %.c, %.o, $(SOURCE))  # 替换.c--.o

INCLUDES = -I ./inc  # 在目录./inc中查找头文件

TARGET  = 100ask 
CC      = gcc
CFLAGS  = -Wall -g  # 编译器选项，提示警告，生成调试信息

$(TARGET): $(OBJECT)
	@mkdir -p output/  # @抑制命令回显，创建目录output/
	$(CC) $^ $(CFLAGS) -o output/$(TARGET)  # 生成可执行文件output/100ask.o

%.o: %.c  # 任何目标文件.o，任何源文件.c
	$(CC) $(INCLUDES) $(CFLAGS) -c $< -o $@

.PHONY: clean  # 伪命令，用于make clean
clean:
	@rm -rf $(OBJECT) output/
	
# Makefile函数...
```

编译结果

```bash
最后编译的结果，如下:
$ make
gcc -I ./inc  -c src/main.c -o src/main.o
gcc -I ./inc  -c src/add.c -o src/add.o
gcc -I ./inc  -c src/sub.c -o src/sub.o
gcc src/main.o src/add.o src/sub.o  -o output/100ask
$tree
.
├── inc
│   ├── add.h
│   └── sub.h
├── Makefile
├── output
│   └── 100ask
└── src
    ├── add.c
    ├── add.o
    ├── main.c
    ├── main.o
    ├── sub.c
    └── sub.o
```



### 时钟模块

#### —> IMX6ULL时钟体系

晶体振荡电路：产生==低频源时钟信号(24MHz, 32KHz)==；

​	24MHz源时钟信号：模块XTALOSC24M

​	32KHzRTC(Real-time clock)时钟信号：模块RTC_XTAL，用于记录时间

内置RC(电阻-电容)振荡器：产生低精度源时钟信号(24MHz, 32KHz)



锁相环(PLL, Phase-Locked Loops)电路：稳定和倍频时钟信号，包括鉴相器PD、低通滤波器LPF和压控振荡器VCO

工作模式

① 正常工作：

② Bypass模式(Bypass位)：PLL输入的参考时钟直接传递到输出

③ 输出禁止模式(Enable位)：无论Bypass时钟或者PLL生成的时钟均被禁止

④ 断电模式(Powerdown位)：PLL中大部分电路断电

| 锁相环电路      | 功能                                                   |
| --------------- | ------------------------------------------------------ |
| ARM_PLL, PLL1   | 倍频，1.3 GHz，ARM核心驱动工作                         |
| SYS_PLL, PLL2   | 倍频，x 22，528 MHz，4个分相器(PFD)，系统总线驱动，    |
| USB1_PLL, PLL3  | 倍频，x 20，480 MHz，4个分相器(PFD)，用于UART          |
| AUDIO_PLL, PLL4 | 标准音频时钟信号，650 MHz~1300 MHz，/1、/2或/4分频     |
| VIDEO_PLL, PLL5 | 标准视频时钟信号，650 MHz~1300 MHz，/1、/2、/4或/8分频 |
| ENET_PLL, PLL6  | 倍频，x 20+(5/6)，500 MHz，以太网接口/通用功能         |
| USB2_PLL, PLL7  | 倍频，x 20，480 MHz，                                  |



根时钟信号电路：包括时钟切换电路（switcher）和根时钟生成电路（root generator）

改变CPU工作频率：① 修改CCSR[pll1_sw_clk_sel]将pll1_sw_clk切换为step_clk；② 修改PLL1参数，等待输出时钟信号稳定；③ 将pll1_sw_clk切换为pll1_main_clk



#### —> IMX6ULL时钟相关寄存器

IMX6ULL时钟相关寄存器：分布于CCM和ANALOG_DIG两个模块，均连接在AIPS-1(Advanced Integrated Peripheral System, 高度集成外设系统)总线上。

ANALOG_DIG模块：负责晶体振荡电路和锁相环电路的设置

​	① CCM_ANALOG_PLL_xxx寄存器：设置对应PLL的参数和工作模式

​	② CCM_ANALOG_MISCx(x=0~2)寄存器：杂项的设置或状态显示，包括晶体振荡电路的控制参数

​		与电源管理模块（PMU）共用，在PMU中成为PMU_MISCx(x=0~2)，通常不需要修改

CCM模块：控制根时钟信号的产生、分发和分频控制

地址范围：

| Start Address | End Address | NIC Port   | Size  |
| ------------- | ----------- | ---------- | ----- |
| 020C_8000     | 020C_8FFF   | ANALOG_DIG | 16 KB |
| 020C_4000     | 020C_4FFF   | CCM        | 16 KB |

<img src="C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20241104213120754.png" alt="image-20241104213120754" style="zoom:80%;" />

<img src="C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20241104213229216.png" alt="image-20241104213229216" style="zoom:80%;" />

CCM_CCGRx(x=0~6)：控制各时钟信号在不同模式下是否被屏蔽 

##### ——> 锁相环电路寄存器

ARM_PLL寄存器结构

​	Address: 20C_8000h, base + 0h, offset + 4d×i, where i = 0d to 3d

| 31   | 30 to 20 | 19      | 18 to 17 | 16     | 15 to 14 | 13     | 12        | 11 to 7 | 6 to 0     |
| ---- | -------- | ------- | -------- | ------ | -------- | ------ | --------- | ------- | ---------- |
| LOCK | Reserve  | PLL_SEL | Reserve  | Bypass | Reserve  | ENABLE | POWERDOWN | Reserve | DIV_SELECT |

​	正常工作：Bypass = 0, Powerdown = 0, ENABLE = 1

​	锁相环电路稳定：LOCK = 1

​	输出频率：Fref * DIV_SELECT / 2

其他PLL与ARM_PLL类似，而AUDIO_PLL和VIDEO_PLL包括额外的分频参数：NUM和DENOM

​	① CCM_ANALOG_PLL_AUDIO_NUM

​	② CCM_ANALOG_PLL_AUDIO_DENOM

​	输出频率：Fref * (DIV_SELECT + NUM / DENOM)

​	此外，还在时钟切换电路（switcher）中设置额外的分频参数为/1、/2、/4、/8或/16，这些值分布在寄存器CCM_ANALOG_PLL_AUDIO、CCM_ANALOG_PLL_VIDEO和CCM_ANALOG_MISC2中。

SYS_PLL和USB1_PLL含有4个分相器(PFD)

CCM_ANALOG_PFD_480n 和 CCM_ANALOG_PFD_528n寄存器结构

| 31           | 30          | 29 to 24  | 23   | 22   | 21 to 16 | 15   | 14   | 13 to 8 | 7    | 6    | 5 to 0 |
| ------------ | ----------- | --------- | ---- | ---- | -------- | ---- | ---- | ------- | ---- | ---- | ------ |
| PFD3_CLKGATE | PFD3_STABLE | PFD3_FRAC |      |      |          |      |      |         |      |      |        |

​	输出频率：Fvco * 18 / PFD_FRAC，其中Fvco为相应PLL的输出频率，而PFD_FRAC的取值范围：12 to 35



##### ——> 根时钟控制寄存器

时钟切换电路（Switcher）寄存器CCM_CCSR：选择pll1_sw_clk和pll3_sw_clk的信号来源

| 31 to 9 | 8        | 7 to 4  | 3                 | 2               | 1    | 0               |
| ------- | -------- | ------- | ----------------- | --------------- | ---- | --------------- |
| Reserve | Step_Sel | Reserve | SECONDARY_CLK_SEL | PLL1_SW_CLK_SEL | R    | PLL3_SW_CLK_SEL |

ARM根时钟信号由pll1_sw_clk分频得来，相应的分频寄存器CCM_CACRR(CCM Arm Clock Root Register)，控制后分频系数，寄存器结构如下

| 31 to 3 | 2 to 0                                        |
| ------- | --------------------------------------------- |
| Reserve | ARM_PODF(Arm Post Divider Factor，后分频系数) |

​	ARM时钟信号频率：pll1_sw_clk / (Arm_PoDF + 1)

总线根时钟信号由寄存器CCM_CBCDR(CCM Bus Clock Divider Register)来选择和分频，寄存器结构如下

| 31 to 30 | 29 to 27         | 26              | 25             | 24 to 19 | 18 to 16 | 15 to 13 | 12 to 10 | 9 to 8   | 7           | 6       | 5 to 3           | 2 to 1            |
| -------- | ---------------- | --------------- | -------------- | -------- | -------- | -------- | -------- | -------- | ----------- | ------- | ---------------- | ----------------- |
|          | PERIPH_CLK2_PODF | Periph2_CLK_SEL | Periph_CLK_SEL |          | AXI_PoDF |          | AHB_PoDF | IPG_PoDF | AXI_ALT_SEL | AXI_SEL | Fabric_MMDC_PoDF | Periph2_CLK2_PoDF |



##### ——> 模块时钟屏蔽寄存器

① Run模式：**CCM_CLPCR（CCM Low Power Controller Register）**[LPM]的值为0

② Wait模式：CCM_CLPCR[LPM] = 1

③ Stop模式：CCM_CLPCR[LPM] = 2

对于每个时钟信号，imx6ull提供了在不同工作模式下是否屏蔽这些时钟信号的控制方法，这些控制位集中放在**寄存器CCGRx**（x = 0-6）中，每个CCGRx寄存器结构如下图所示：

<img src="C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20241104220847772.png" alt="image-20241104220847772" style="zoom:80%;" />

每两位为一个单位，控制一个时钟信号。比如，在寄存器CCGR0中，CG15控制时钟信号gpio2_clocks，CG14控制时钟信号uart2_clock等等。这两位值的含义如下所示：

| CGx （x=0-15） | 时钟信号活动状态描述                              |
| -------------- | ------------------------------------------------- |
| 00             | 时钟信号在三个模式中均被屏蔽。                    |
| 01             | 时钟信号仅在RUN模式开启，在其它模式中被屏蔽。     |
| 10             | 保留                                              |
| 11             | 时钟信号在RUN和WAIT模式开启，在STOP模式中被屏蔽。 |



### UART串口模块

==UART，通用异步收发串口，寄存器设置，收发数据帧==





==递进关系是什么？ 晶体振荡电路——锁相环电路——根时钟信号电路==

==寄存器功能（引脚）以及在实例中的各层电路所利用寄存器的功能和递进关系==



### 重定位



### 异常与中断

==代码里是如何进入UND异常和SVC异常， 编译和仿真结果==

==GICD_ISENABLER寄存器配置文件在哪儿呢？ ARM中断控制器架构规范里面没有写寄存器地址==



### 汇编语言：指令

1. `mrc` 是 ARM 汇编中的 **Move to Register from Coprocessor**，用于从协处理器（例如 CP15 系统控制协处理器）读取值到通用寄存器。

   ```assembly
   mrc p<coprocessor>, <opcode1>, <Rt>, <CRn>, <CRm>, <opcode2>
   ```

   **`p<coprocessor>`**：指定协处理器编号。例如，`p15` 是 ARM 的系统控制协处理器。

   **`<opcode1>`**：用于选择操作。

   **`<Rt>`**：目标通用寄存器，用于存储从协处理器读取的值。

   **`<CRn>`** 和 **`<CRm>`**：协处理器寄存器编号。

   **`<opcode2>`**：选择次级操作。

   例子：

   ```assembly
   mrc p15, 0, r0, c1, c0, 0  /* 从 CP15 的系统控制寄存器 (c1) 读取值到 r0 */
   ```

2. `bic` 是 ARM 汇编中的 **Bit Clear** 指令，用于清除指定位（将某些位设置为 0）

   ```assembly
   bic <Rd>, <Rn>, <operand>
   ```

   **`<Rd>`**：目标寄存器，存储操作结果。

   **`<Rn>`**：源寄存器，提供操作数。

   **`<operand>`**：需要清除的位的掩码。

   例子：

   ```assembly
   bic r0, r0, #(0x1 << 13)  /* 清除 r0 中第 13 位 */
   ```

3. `mcr` 是 ARM 汇编中的 **Move to Coprocessor from Register**，用于将通用寄存器的值写入到协处理器寄存器。

   ```assembly
   mcr p<coprocessor>, <opcode1>, <Rt>, <CRn>, <CRm>, <opcode2>
   ```

   **`p<coprocessor>`**：指定协处理器编号，例如 `p15`。

   **`<opcode1>`**：选择操作。

   **`<Rt>`**：源通用寄存器。

   **`<CRn>`** 和 **`<CRm>`**：协处理器寄存器编号。

   **`<opcode2>`**：选择次级操作。

   例子：

   ```assembly
   mcr p15, 0, r0, c1, c0, 0  /* 将 r0 的值写入到 CP15 的系统控制寄存器 (c1) */
   ```

4. `cps` 是 ARM 汇编中的 **Change Processor State** 指令，用于切换处理器的模式（例如，切换到管理模式、未定义模式等）。

   ```assembly
   cps <mode>
   ```

   **`<mode>`**：指定处理器要切换到的模式。

   - ARM 的处理器模式包括：
     - **`#0x10`**：用户模式（User Mode）。
     - **`#0x13`**：管理模式（Supervisor Mode）。
     - **`#0x1B`**：未定义模式（Undefined Mode）。
     - **`#0x1F`**：系统模式（System Mode）。

```assembly
cps #0x1B  /* 切换到未定义模式 */
```

5. `ldr` 是 ARM 汇编中的 **Load Register** 指令，用于从内存地址加载值到寄存器。

   ```assembly
   ldr <Rt>, [<address>]
   ldr <Rt>, =<value>
   ```

   **`<Rt>`**：目标寄存器，用于存储从内存加载的值。

   **`[<address>]`**：内存地址，从该地址加载数据。

   **`=<value>`**：立即数，将其加载到寄存器。

   例子：

   ```assembly
   ldr r0, [r1]  /* 从 r1 指向的内存地址加载值到 r0 */
   ldr sp, =0x80300000  /* 将值 0x80300000 加载到 sp 寄存器 */
   ```

6. `stmdb` 是 "Store Multiple Decrement Before" ，将多个寄存器的内容按降序地址存储到内存中（通常是栈中）。

   ```assembly
   stmdb <基址寄存器>!, {<寄存器列表>}
   ```

   `<基址寄存器>`：用于指向内存地址的基址寄存器（通常是栈指针 `sp`）。

   `!`：表示在存储完成后更新 `<基址寄存器>` 的值。

   `{<寄存器列表>}`：需要存储的寄存器集合。

7. `ldmia` 是 "Load Multiple Increment After" ，从内存中按升序地址加载多个寄存器的值。

   ```assembly
   ldmia <基址寄存器>!, {<寄存器列表>}[^]
   ```

   `<基址寄存器>`：用于指向内存地址的基址寄存器（通常是栈指针 `sp`）。

   `!`：表示加载完成后更新 `<基址寄存器>` 的值。

   `{<寄存器列表>}`：需要加载值的寄存器集合。

   `^`：可选，用于指示在加载完成后，将保存程序状态寄存器（SPSR）的值恢复到当前程序状态寄存器（CPSR）。

8. `mrs`将一个程序状态寄存器的值复制到通用寄存器中。

   ```assembly
   mrs <目标寄存器>, <源寄存器>
   ```

   `<目标寄存器>`：通用寄存器，用于保存状态寄存器的值。

   `<源寄存器>`：状态寄存器，可以是以下两种：

   - **CPSR**：当前程序状态寄存器，包含当前模式的状态信息（如条件标志位、中断屏蔽位、处理器模式等）。
   - **SPSR**：保存程序状态寄存器，保存被中断的模式的 CPSR 值。
