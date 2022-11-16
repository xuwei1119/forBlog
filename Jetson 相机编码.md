# Jetson 相机编码

Jetson相机编码是即将发布的“实践”系列的相关代码。有三个存储库:

# 1. camera-caps

JetsonHacks Github存储库[camera-caps](https://github.com/jetsonhacks/camera-caps) 通过`v4l2-ctl`命令行工具提供了一个图形用户界面。您可能会发现，它可以方便地检查连接到Jetson上的V4L2相机的功能。这适用于CSI相机和USB相机。

这个应用程序是一个简单的软件草图，用来支持演示。它没有完全的特性，当然也不是产品质量的代码，但是您可能会发现它对您自己的研究和实验很有用。在NVIDIA Jetson系列产品中，连接的相机通常通过V4L2模块。USB摄像头通过与v4l2模块接口的uvcvideo模块连接。通过CSI/MIPI端口连接的摄像头(如树莓Pi摄像头、GMSL摄像头)与tegra-video模块连接，tegra-video模块又与v4l2模块连接。当连接到正确安装的驱动程序时，连接的摄像头显示为`/dev/videoX(其中X是ID号)`。正确连接和注册后，可以使用`v4l2-ctl`实用程序检查摄像机的属性。这包括可用的像素格式、帧大小、帧速率和属性。有调节相机属性的控件。GUI提供了一个统一视图:

![image-20221103152622323](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221103152622323.png)

并非所有相机都提供V4L2接口。有些相机具有不通过V4L2暴露的专有接口。

## 1.1 安装

```shell
$ sudo apt update
$ sudo apt install python3-pip
$ pip3 install dataclasses

# Install v4l2-ctl

$ sudo apt install v4l-utils
```

## 1.2 运行程序

在运行程序之前，首先确保要检查的摄像机已连接。该程序不检测动态附件。如果您插/拔摄像头，请重新启动程序。还要注意，USB摄像头在其`/dev/videoX`名称中没有一个保证地址。换句话说，当机器重新启动或添加其他摄像头时，地址可能会更改。运行:

```shell
$ python3 camera_caps.py
```

预览按钮尝试构建GStreamer管道并在预览窗口中运行它。预览窗口的大小不是视频图像的完整大小。

+ JetPack 4.6, L4T 32.6.1
+ Jetson Nano, Jetson Xavier NX的测试-其他Jetsons应该可以工作

链接：https://pan.baidu.com/s/1KZfxLNfa-qEEvkbCoObv2Q?pwd=xae6 
提取码：xae6

# 2. USB-Camera

[USB-Camera](https://github.com/jetsonhacks/USB-Camera)是一个Github存储库，其中有使用V4L2相机和Jetson开发工具包的示例Python脚本。这些示例使用OpenCV(包括在JetPack中)捕获摄像机并将其显示在屏幕上。一个例子展示了如何使用V4L2相机前端与相机连接。另一个例子使用GStreamer前端与摄像机连接。GStreamer在Jetson生态系统中非常重要，因为它为DeepStream智能视频分析(IVA)提供了基础。

第三个例子使用Haar级联来检测人脸和眼睛。这是一个如何从相机获取视频帧并处理它们的示例。

# 3. CSI-Camera

另一种将相机与Jetson连接的方法是通过MIPI相机串行接口(CSI)。MIPI是发布嵌入式系统标准的组织名称。[CSI-Camera](https://github.com/JetsonHacksNano/CSI-Camera)代码是对早期JetsonHacks文章[Jetson Nano + Raspberry Pi Camera](https://jetsonhacks.com/2019/04/02/jetson-nano-raspberry-pi-camera/)和 [Jetson Nano B01 – Dual Raspberry Pi Cameras](https://jetsonhacks.com/2020/04/08/jetson-nano-b01-dual-raspberry-pi-cameras/)的更新。

为了获得更好的帧率，GStreamer管道进行了精简。我们还向Python代码添加异常处理，以及一些其他的清理，以使代码更加健壮。

# BONUS

[jetsonUtilities](https://github.com/jetsonhacks/jetsonUtilities)拥有与NVIDIA Jetson开发工具包一起工作的实用程序。

在NVIDIA Jetson开发套件(TX1, TX2, AGX Xavier, Xavier NX, Nano, Nano 2GB)上获取有关NVIDIA Jetson操作系统环境的信息

关于NVIDIA Jetson Development Kit操作系统的信息分布在几个文件中。这是一个方便的参考工具。

Python脚本jetsoninfo.py将列出硬件、正在运行的L4T版本、Ubuntu版本和Linux内核版本。执行:

```shell
$ python3 jetsonInfo.py
```

硬件指示符源自文件:`/proc/cpuinfo`

L4T版本源自文件:`/etc/nv_tegra_release`

Ubuntu版本源自文件:`/etc/os-release`

Linux内核版本源自文件:`/proc/version`

# 参考目录

[https://jetsonhacks.com/2022/01/25/jetson-camera-coding/](https://jetsonhacks.com/2022/01/25/jetson-camera-coding/)