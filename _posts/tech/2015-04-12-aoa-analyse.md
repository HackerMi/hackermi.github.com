---
layout: post
title: Android Open Accessory 协议分析与开发
categories: tech
tags:  Android AOA USB_Accessory
---
### 1. 背景介绍
&emsp;2011年Google推出Android开放配件协议AOA（Android Open Accessory Protocol）及配件开发工具包ADK（Accessory Development Kit）。当时开源硬件和硬件创业都比较热，其中以[Arduino平台](http://baike.baidu.com/view/1268436.htm)为代表。Google想借着这个硬件平台开拓智能家具的市场，借此推出了AOA协议和对应的开发平台，以便打通Android手机和周边硬件的连接通道，从而扩大Android的生态链。2012年Google推出了2.0版本的协议，增加了音频相关的支持，随后没有进一步的更新。从目前的状况看，这个协议并没有被广泛的认可和使用。最近的消息是2014年Google发布的Android Auto基于该协议构建了Android Auto Porotocol，Google未来可能会将其作为Android系统与周边硬件进行USB连接的基础协议。  
&emsp;更多背景资料请参考[Google 2011 I/O大会视频](http://pan.baidu.com/s/1hqIigaK)， 其中36分后开始介绍AOA。下面主要针对AOA协议及其开发进行分析和介绍。

<!--more-->


### 2. 协议分析

#### 2.1 Android 系统与配件的连接方式

![连接方式]({{ site.url }}/assets/aoa_analyse_dev/connect_method.jpg)


Android系统官方定义了两种连接配件的方式：蓝牙和USB，未来可能提供WiFi的连接方式。  

#### 蓝牙连接

&emsp;Android系统定义的连接配件协议是SSP/A2DP，这些都是很成熟的协议，其优点很明显——开发资源丰富，协议稳定，并且蓝牙本身是无线连接，使用方便，尽管覆盖距离没有手机信号覆盖范围大，但已能够满足日常所需。缺点主要是传输带宽有限，特别是给配件传输高保真音频基本是不可能的。另外一个问题是耗电，不过随着低功耗蓝牙协议的推出这个问题已经得到解决。

#### USB连接

&emsp;相较蓝牙连接，USB连接本身具有传输稳定、带宽高、持续供电的优点，不过其缺点也比较突出，通讯距离比较短，有线连接对于遥控设备不是很方便。至于连接协议，Google并没有借用现有的USB协议，而是另起炉灶定义了自己的协议—— AOA（Android Open Accessory Protocol），其中的缘由已经不得而知。说句题外话，Android Auto正是看到USB连接的优点比较适合手机与车机连接的，在第一版的Android Auto中选择了这种连接方式。

#### WiFi连接

&emsp;这种连接方式可以解决蓝牙带宽不足的问题，又可以提供较好的覆盖范围，不过Google目前还没有公开支持。只是从Android Auto的发布会视频中得知未来可能会使用WiFi-Direct协议，车内互联是现在可以看得到的应用场景。

#### 2.2 USB通讯模式

&emsp;这里主要讨论的USB通讯模式包括：USB Slave模式，USB Host模式(例如OTG)，USB 配件模式(例如AOA)

#### USB Slave模式

![USB Slave]({{ site.url }}/assets/aoa_analyse_dev/usb_slave.jpg)

&emsp;这种模式通常应用于PC与Android手机连接，PC端作为服务的提供方，主要完成设备枚举，搜索相关驱动并完成安装，建立数据通路等工作。该模式下PC对Android手机供电。

#### USB Host模式

![USB OTG]({{ site.url }}/assets/aoa_analyse_dev/usb_otg.jpg)

&emsp;这种模式通常应用于Android手机以OTG连接的方式连接外设，比如：键盘，鼠标等。Android手机端是简化版的USB host端 ———— USB Embedded Host。它通过内置的外围设备列表TPL(Target Peripheral List)完成设备枚举和驱动加载，不具有加载未知设备驱动的能力。该模式下Android手机对外设供电。

#### USB 配件模式

![USB AOA]({{ site.url }}/assets/aoa_analyse_dev/usb_aoa.jpg)

&emsp;这种模式是Android定义的一种新的通讯模式，它将配件作为协议交互的主要角色，配件内置USB Embedded Host端，可以为Android手机供电，并且识别Android手机，建立数据通道。使得配件成为一个简化版的PC Host端，方便市面上Android手机拓展其功能。官方宣称Android 2.3.4增加lib库即可支持，Android 3.1以上系统直接支持。

#### 2.3 AOA协议工作过程

&emsp;协议整体交互图： 

![协议交互]({{ site.url }}/assets/aoa_analyse_dev/protocol_overview.jpg)

1. 配件端等待手机连接
2. 手机接入配件
3. 配件通过USB驱动的控制通道（0通道）获取手机的vendor ID与product ID
4. 配件分析获取的vendor ID是否为0x2D01, 0x2D02, 0x2D03或0x2D04，以及product ID是否为0x18D1。如果均符合，则说件现在手机已经是配件模式，可以按照配件模式的要求直接重新配置USB端点和接口。否则启动尝试进入配件模式流程。
5. 确定手机已经进入配件模式，重新枚举USB设备，手机重新进行USB协商。
6. 按照配件模式重新配置USB端点和接口，建立配件模式的数据通道。

&emsp;尝试启动配件模式交互流程：  

![协议探测]({{ site.url }}/assets/aoa_analyse_dev/protocol_detect.jpg)

1. 配件发送序号51的USB请求报文，手机收到后查询自己的AOA协议版本，发送响应报文给配件。
2. 配件校验协议版本号，目前为1或2，其他的均为不支持。
3. 配件发送序号52的USB请求报文，通过Index字段携带配件自身信息，包括制造商，型号，版本，设备描述，序列号，URI等。手机根据这些信息启动响应的APP。
4. 配件发送序号53的USB请求报文，切换USB模式，主要是根据切换vendor ID和product ID。
5. 重新枚举USB设备，准备建立AOA数据通道。

### 3. 软件开发

&emsp;Android 提供了[ADK(Accessory Development Kit)](http://developer.android.com/tools/adk/adk.html
)帮助开发者进行配件开发。ADK是一套完整的开发环境，基于Arduino 开源硬件平台，主要包括： 

* USB主控板（Arduino Mega2560）及对应的firmware
* USB Host Shield 及相应的library
* Android Accessory protocol library
* 一些Sensor和接口library
* Android 手机控制APK

&emsp;ADK开发平台结构如下图： 

![平台结构]({{ site.url }}/assets/aoa_analyse_dev/aoa_dev.jpg)

&emsp;从上图可以看出，Arduino平台帮助开发者实现了AOA协议和Host驱动，从而搭建出一套Android手机与配件直接通讯的数据通道。整个结构有些像路由器的转发，Arduino平台将手机的控制命令转发给硬件控制器，使得手机可以控制硬件。

&emsp;Android ADK开发主要包括三个方面：Arduino硬件开发环境搭建、Android手机软件开发、配件硬件开发。 

#### Arduino硬件开发环境搭建

&emsp;具体参见官网[详细流程](http://developer.android.com/tools/adk/adk.html)。这里简单说一下做的事情，Arduino开发板是一个嵌入式系统，首先需要在PC机上搭建对应的开发环境，主要是lib库，编译环境，而后为嵌入式系统烧入对应的固件，保证可以和PC机正常通讯，而且能够正确工作即可。

#### Android手机软件开发

&emsp;相关接口和文档[参见官网](http://developer.android.com/guide/topics/connectivity/usb/accessory.html#manifest)。Androd系统的framework和kernel已经处理的AOA协议大部分工作，APP开发者只需响应对应的通知事件即可。Android将Accessory的大部分实现均封装在UsbAccessory类中，主要流程是：   

1. 接收连接建立通知
2. 枚举可用设备
3. 获取访问权限
4. 建立通讯通道
5. 发送控制命令，接收控制响应

#### 配件硬件开发

&emsp;主要是指配件的硬件设计及相应的驱动开发，每个硬件都不相同，这里不做进一步讨论。
