---
title: "MCU 单片机固件逆向分析入门 - Hyun's home"
source: "https://www.hyun.tech/archives/mcu%E5%8D%95%E7%89%87%E6%9C%BA%E5%9B%BA%E4%BB%B6%E9%80%86%E5%90%91%E5%88%86%E6%9E%90%E5%85%A5%E9%97%A8"
author:
published:
created: 2025-09-06
description: "本篇文章以STM32F411CEU6开发板为例，同时又不局限于该型号，一步步从固件 Dump 开始逐步分析并获得全部工程源代码"
tags:
  - "clippings"
---
## 前言

本篇文章以 `STM32F411CEU6` 开发板为例，同时又不局限于该型号，一步步从固件 Dump 开始逐步分析并获得全部工程源代码

## 0x1 提取固件

> 如果你已经有了目标设备的固件升级包, 则可以跳过此步

### 连接设备

找到目标设备的 J-TAG 或者 ST-LINK 接口, 连接电脑, STM32 通常使用四线 SWD 接口:

**`VCC` `GND` `DIO` `SCK`**

建议不要接 VCC, 接上剩余三线并插入设备电源即可

### 准备工作

新建一个名为 `stlink.cfg` 的文件, 并编辑:

```
# choose st-link/j-link/dap-link etc.
#adapter driver cmsis-dap
#transport select swd
source [find interface/stlink.cfg]
transport select hla_swd
source [find target/stm32f4x.cfg]
# download speed = 10MHz
adapter speed 10000
```

这里的 `interface/stlink.cfg` `target/stm32f4x.cfg` 可以打开 `OpenOCD安装目录\share\openocd\scripts` 按你的实际情况, 选择对应配置

成功连接后输出如下

```shell
> openocd -s D:\OpenOCD\share\openocd\scripts -f stlink.cfg

Open On-Chip Debugger 0.11.0 (2021-07-29) [https://github.com/sysprogs/openocd]
Licensed under GNU GPL v2
libusb1 09e75e98b4d9ea7909e8837b7a3f00dda4589dc3
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
adapter speed: 10000 kHz

Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : clock speed 10000 kHz
Info : STLINK V2J37S7 (API v2) VID:PID 0483:3748
Info : Target voltage: 3.275331
Info : stm32f4x.cpu: hardware has 6 breakpoints, 4 watchpoints
Info : starting gdb server for stm32f4x.cpu on 3333
Info : Listening on port 3333 for gdb connections
```

查看 [芯片手册](https://www.st.com/resource/en/datasheet/stm32f411ce.pdf), 并找到 **Memory Mapping** 部分, 可看到下面这样的图

#### Memory Mapping

![image.png](https://img.hyun.tech/blog/image_1633522113360.png)

其中 **Flash memory** 中就存着我们需要的固件, 从图中可以看出 **Flash memory** 的地址映射范围为 `0x08000000 - 0x0807FFFF` 相减得到 Flash 起始地址: `0x08000000` 、大小 `0x7FFFF` 。

### Dump 固件

如图开启 Telnet

![image.png](https://img.hyun.tech/blog/image_1633520822344.png)

连接到 OpenOCD, 并输入一下命令 Dump 固件

```shell
> telnet 127.0.0.1 4444

> Open On-Chip Debugger
> help dump_image
dump_image filename address size
> dump_image out.img 0x08000000 0x7FFFF
dumped 524287 bytes in 3.494784s (146.504 KiB/s)
```

完事! 当前目录下 `out.img` 就是目标设备的固件!

## 0x2 基础分析

### 导入 IDA

![image.png](https://img.hyun.tech/blog/image_1633523200229.png)

选择 `ARM Little-Endian` ARM 小端, 并进入处理器选项菜单

![image.png](https://img.hyun.tech/blog/image_1633523303634.png)

如图单击 **编辑 ARM 架构选项**, 在打开的新窗口中选择 **ARMv7-M**

![image.png](https://img.hyun.tech/blog/image_1633523663420.png)

此处建议勾上 `Create RAM section` 给内存区域建一个段, 方便接下来分析过程中对全局变量重命名与引用查找(以后在 Segments 窗口再创建也行)

至于 `RAM` 的地址跟大小可以参考 [Memory Mapping](https://www.hyun.tech/archives/#memory-mapping) 中的 `SRAM(128 KB aliased by bit-banding)`, 同理, `ROM` 可参考 `Flash memory`, `Loading address` 与 `Loading size` 与 `ROM` 保持一致即可。

![image.png](https://img.hyun.tech/blog/image_1633524486954.png)

鼠标选中箭头指向地址, 重复按 `D` 将其转为Double Word大小。重复操作, 直至想下图这样  
![image.png](https://img.hyun.tech/blog/image_1633524646062.png)

双击第二个地址(这里是 `0x80001B1`), 由于 ARM 的特性, 需要选中跳转后地址的 **上一行** (`0x80001B0` 处), 按 `C` 键反编译

![image.png](https://img.hyun.tech/blog/image_1633524806048.png)

没有什么意外的话 `IDA` 已经识别出了所有被使用过的函数

![image.png](https://img.hyun.tech/blog/image_1633524973987.png)

![image.png](https://img.hyun.tech/blog/image_1633525004071.png)

![image.png](https://img.hyun.tech/blog/image_1633525022979.png)

![image.png](https://img.hyun.tech/blog/image_1633525035780.png)

如图连续跟进，即可找到主函数

![image.png](https://img.hyun.tech/blog/image_1633525169498.png)

此时按 TAB, 就可以看到 IDA 给主函数生成的伪代码

![stm32idafakecode.png](https://img.hyun.tech/blog/stm32-ida-fake-code_1633525348958.png)

### 环境搭建

> STM32设备的开发常使用到 `CMSIS` 与 `HAL`, 这些库帮我们实现了大部分 **对硬件底层的操作**, 降低了开发门槛、提高了开发效率但同时也为我们逆向提供了方便。我们逆向使用这些库的固件, 就可以充分利用这些 **开源** 的库。

此时, 代码由于描述符的缺失, 还是很难阅读与深入分析, 因此, 我们可以搭建 **与目标设备一致** 的环境，并编译 **带有描述符** 的固件，与目标设备固件对比，快速识别库函数, 节省大量逆向花费的时间。

这里的目标固件使用 `HAL` 库开发。我们在 `STM32CubeMX` 新建一个Demo项目,

![image.png](https://img.hyun.tech/blog/image_1633526042378.png)

尽可能多的调用外设以便接下来比对时识别更多的函数

![image.png](https://img.hyun.tech/blog/image_1633526243025.png)

生成代码并用 Keil 打开, 编译并在项目目录下找到生成的axf文件

![image.png](https://img.hyun.tech/blog/image_1633526480594.png)

IDA 载入, 可以看到很清晰的逻辑, 然后 **关闭IDA**, 关闭时请保持默认选择 **Pack数据库文件**

### 快速比对函数

这里使用 `BinDiff` 工具对 **自行编译的固件** 与 **目标设备固件** 进行函数级的比对

#### 安装 BinDiff

在 [官网](https://www.zynamics.com/software.html) 下载 **msi 安装包** 一路下一步即可

#### BinDiff 使用

安装好后新建个 `Workspace`, 右键 `New Diff`, 选择使用 **IDA打开目标设备固件生成的 `idb` 文件** 与 **自己编译出来的 `axf` 生成的 `idb`**

![image.png](https://img.hyun.tech/blog/image_1633527358547.png)

> 这里有个小坑, IDA 7.6 的 idb 会报错, 换成 IDA 7.5 以下版本就一切正常

可以看到比中了很多函数, confidence 在 0.9 以上就基本可以确定是相同函数, 使用 IDA 打开目标固件, Ctrl+6 打开 BinDiff 插件的菜单:

![image.png](https://img.hyun.tech/blog/image_1633527555279.png)

**Diff Database** 选择自行编译固件生成的 idb 文件

![image.png](https://img.hyun.tech/blog/image_1633527688305.png)

打开 view->bindiff->matched functions 右键import symbols/comments 可以一键导入函数的描述符

### 导入设备头文件

![image.png](https://img.hyun.tech/blog/image_1633498421345.png)

选择 工程路径\\Drivers\\xxxxx\_HAL\_Driver\\Inc 中的 xxxxx\_hal.h

如果没有意外的话, 你会收获一个或多个错误, 大部分的错误都是缺少文件, 从其他地方找到并放入同一目录一般可以解决问题, 关于错误修正的详细过程, 这里不再赘述

## 0x3 深入了解

### 进一步分析库函数

`tips` 注意观察函数名的规律, 有的编译器会按照 **字典序** 给函数名排序

### 外设基址计算

`待补充...`

  

**参考资料**

> 使用 ida 逆向分析 stm32 的 bin 固件文件  
> [https://www.angelic47.com/archives/97/](https://www.angelic47.com/archives/97/)  
> IDA编译STM32 Hex\\Bin文件成C代码  
> [https://blog.csdn.net/daidi1989/article/details/86304843](https://blog.csdn.net/daidi1989/article/details/86304843)