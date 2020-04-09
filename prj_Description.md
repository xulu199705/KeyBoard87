# 从零开始的客制化87键机械键盘

> *STM32,嵌入式,USB-HID*

> 还有半年就要毕业了，在做毕业设计之余，我准备客制化一个87键机械键盘。键盘用STM32做主控，USB type-C做接口，以基本操作为最低目标。当然了，由于主专业的毕业设计和第二专业的毕业论文，留给这个项目的时间不会太多，项目的开发周期会(geng)有(xin)半(wan)年(quan)之(sui)久(yuan)。

-----

## 项目初衷

最初决定做键盘是在1月初的时候。当然，一开始想做的并不是键盘，而是一个USB-to-Bluetooth的转换器（如下图），这样就可以在iPad上用机械键盘了。

![USB-to-Bluetooth Convertor](pic/other/bt_connector.jpg?raw=true)

转换器的主控同样是准备采用STM32，毕竟STM32作为我比较熟悉的MCU，不仅功能强大，而且有过几次基于STM32系统的制作经验。

这时候，本着“不如搞个大事情”的原则，我就想着不如直接做个机械键盘吧。~~就这样，我踏上了从零开始的客制化87键机械键盘不归路~~。

项目软件调通之后会将硬件原理图、封装库和STM32的程序发布到我的[博客](https://www.blueschwarz.top)，连接会在附录中，敬请期待···

-----

## 硬件设计

这一章介绍一下客制化键盘的硬件设计，包括以下4项：主控芯片、传输接口、轴体选择、键盘布局。

### 主控芯片

在确定STM32作为主控芯片之前，我在网上搜了一下，也拆了手上的ikbc C87，ATMEL的ATMEGA32芯片使用的比较多。我最终还是选择了STM32，主要原因是对STM32比较熟悉。

当然，作为32位的MCU，STM32不管是在IO数量、USB接口支持，还是程序运算速度，都能满足机械键盘的需求。

- USB接口

机械键盘使用时是作为USB Slave使用的，所以只需要芯片支持USB就行，不需要OTG。STM32F1的所有芯片都支持全速USB 2.0，所以随便选一款就行。

- IO数量

简单计算一下机械键盘需要的IO口，见下表（实际IO分配以实际原理图为准）。

|  序号  |  用途  |  数量  |  备注  |
| --- | --- | --- | ---|
|  1  |  键盘列  |  17  |  87键键盘布局，最多的一行（第2行）共17个按键  |
|  2  |  键盘行  |  6  |  87键键盘布局，共6行  |
|  3  |  USB接口  |  2  |  USBDN和USBDP  |
|  4  |  调试接口  |  2  |  使用STLink连接，仅需JTMS-SWDIO、JTCK-SWCLK和GND三个接口  |

简单计算一下，键盘需求的IO口为27个。

- 运算速度

STM32有四个时钟源：

|  时钟源  |  备注  |
| --- | --- |
|  高速外部时钟(HSE)  |  可取4-16MHz，一般采用8MHz  |
|  高速内部时钟(HSI)  |  8MHz，内部RC振荡器产生，不稳定  |
|  低速外部时钟(LSE)  |  外部晶振，一般采用32.768KHz  |
|  低速内部时钟(LSI)  |  内部RC振荡器产生，约40KHz  |

机械键盘用8MHz外部晶振，内部的系统时钟最高为72MHz，满足机械键盘的输入精度要求。

### 传输接口

机械键盘的传输接口已经确定为USB。一般来说，USB有以下几种接口类型：

| USB分类 | USB A-type | USB B-type | miniUSB | microUSB | C-type |
| --- | --- | --- | --- | --- | --- |
| 图片 | ![](pic/usb_type/type_a.jpg?raw=true) | ![](pic/usb_type/type_b.jpg?raw=true) | ![](pic/usb_type/miniusb.jpg?raw=true) | ![](pic/usb_type/microusb.jpg?raw=true) | ![](pic/usb_type/type_c.jpg?raw=true) |

现在市面上键线分离的机械键盘一般用的是miniUSB或microUSB接口。最近半年来，USB C-type特别流行。这个接口不仅提供大功率传输，而且支持正反随便插，大大降低了插第二次的郁闷。机械键盘就决定用USB C-type。

### 轴体选择

在选择机械键盘的轴体时，因为正在使用的ikbc C87，所以直接就决定用樱桃的MX轴。常用的MX轴有黑青茶红四种，茶轴肯定是我的首选，不管用做什么，都可以满足要求。

MX轴的封装尺寸：[PDF](Reference/Cherry_MX/Cherry_MX_sw.pdf)

### 键盘布局

键盘布局选择和手头使用的ikbc c87一样的87键布局，在[这里](http://www.keyboard-layout-editor.com/)可以将需要的布局信息生成一堆文本，将文本复制到[这里](http://builder.swillkb.com/)，生成定位板文件，方面在PCB中定位每个键轴的位置。

布线图
![](pic/pcb/pcb_2d_bottomlayer.png?raw=true)

|3D顶部视图||
| --- | --- |
| ![](pic/pcb/2.PNG?raw=true) | ![](pic/pcb/pcb_3d_top.png?raw=true) |

|3D底部视图||
| --- | --- |
| ![](pic/pcb/1.PNG?raw=true) | ![](pic/pcb/pcb_3d_bottom.png?raw=true) |

LOGO视图
![](pic/pcb/pcb_3d_logo.png?raw=true)

-----

## PCB打样

PCB打样就交给嘉立创了，下单系统虽然UI做的不怎么样，但系统操作和服务看上去特别高大上。而且最近嘉立创对自己的SMT进行了升级，添加了很多可贴元器件，这样一来，板子一拿回来就只要焊四个0805的贴片和直插的按键轴体。

下单大概四天，板子就寄到了。一层一层的红色气泡纸把五块板子包得严严实实。

![](pic/pcb/pcb_package.jpg?raw=true)

-----

## 软件设计

### USB-HID协议

-----

## 定位板&&外壳

-----

## 项目总结

-----

## 附录












