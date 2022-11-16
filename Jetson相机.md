# Jetson相机

# 1.简介

NVIDIA Jetson Developer Kits支持直接连接相机，主要使用两种方法。第一种方法是使用MIPI摄像机串行接口(CSI)。MIPI联盟是为移动生态系统开发技术规范的行业组织的名称。在像Nano这样的Jetsons上，这可能是一个传感器模块，就像我们熟悉的基于索尼IMX219图像传感器的树莓派V2相机一样。

第二种是通过USB接口连接的摄像头，例如网络摄像头。在本文中，我们将讨论USB摄像头。

# 2.Jetson相机架构

Jetson的核心是使用Linux内核模块Video Four Linux(版本2)(V4L2)。V4L2是Linux的一部分，并不是Jetson所独有的。V4L2可以完成各种各样的任务，这里我们主要关注摄像机的视频捕获。

以下是Jetson相机架构(来自NVIDIA Jetson文档):

![image-20221103173447997](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221103173447997.png)

## 2.1 CSI输入

如前所述，从直接连接到Jetson的摄像机中获取视频主要有两种方式。首先，通过CSI端口，使用libargus，这是一个Jetson专用库，将相机传感器连接到Tegra SoC图像信号处理器(ISP)。ISP硬件执行各种各样的图像处理任务，例如消除图像的色彩、管理白平衡、对比度等。这不是小事，需要特殊的工具和专业知识，以产生高质量的视频。更多信息参考https://developer.ridgerun.com/wiki/index.php?title=NVIDIA_Jetson_ISP_Control#NVIDIA_Jetson_ISP_Control_Description

## 2.2 USB输入

USB摄像头的视频则采用不同的路径。USB视频类驱动(uvcvideo)是一个Linux内核模块。uvcvideo 模块通过映射和驱动程序特定的 ioctl 接口支持对 V4l2 的扩展单元控制（XU 控制）。输入/输出控件(ioctls)是V4L2 API的接口。更多信息:https://www.kernel.org/doc/html/v4.9/media/v4l-drivers/uvcvideo.html

这意味着USB摄像头设备中的驱动程序可以指定不同的依赖于设备的控件，而不需要uvcvideo或V4l2专门理解它们。USB摄像头驱动程序将一个特性的代码以及一系列可能的值传递给uvcvideo。当用户为相机指定一个特征值时，uvcvideo检查参数的边界，并将它与特征代码一起传递给相机。然后相机进行相应的调整。

因为USB摄像头不直接与ISP连接，所以有一个更直接的路径。这里有一个图:

![image-20221103174157693](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221103174157693.png)

术语用户空间指的是用户在其应用程序中可以访问相机的位置。

## 2.3 USB盒外

这是将USB摄像头视频传输到Jetson的一个途径。并非所有相机制造商都提供这种类型的实现。大多数“即插即用”的网络摄像头都是这样，许多成熟的“专业”摄像头也是如此。让我们在这里定义专业是指具有特殊或多个传感器的相机。许多热感相机都提供这种功能，以及像Stereolabs ZED相机和Intel RealSense相机这样的深度相机。

然而，一些特殊的相机可能通过V4L2有限地访问其功能集，或者可能有一个独立的SDK。毕竟，拥有一个独立的SDK更简单，它只从给定的USB串行端口读取，而不必处理通过内核接口的问题。然后人们通过SDK与相机连接。我们在这里不讨论那些类型的相机。

## 2.4 相机功能

摄像头功能描述了摄像头如何工作的许多方面。这包括许多特性，包括像素格式、可用帧大小和帧速率(帧每秒)。您还可以检查控制相机操作的不同参数，如亮度、饱和度、对比度等。

有一个名为`v4l2-ctl`的命令行工具，您可以使用它来检查摄像机功能。要安装`v4l2-ctl`，请安装`v4l-utils debian`包:

```shell
$ sudo apt install v4l-utils
```

下面是一些有用的命令:

```shell
# List attached devices
$ v4l2-ctl --list-devices
# List all info about a given device
$ v4l2-ctl --all -d /dev/videoX
# Where X is the device number. For example:
$ v4l2-ctl --all -d /dev/video0
# List the cameras pixel formats, images sizes, frame rates
$ v4l2-ctl --list-formats-ext -d /dev/videoX
```

还有更多的命令，请使用帮助命令获取更多信息。

# 3.与应用程序交互

在Jetson相机体系结构中，V4L2相机流可以通过两种不同的方式用于应用程序。第一种方法是直接使用ioctl控件或通过具有V4L2后端的库使用V4L2接口。第二种方法是使用GStreamer，它是一个媒体处理框架。

## 3.1 ioctl

Jetson上最流行的通过`ioctl`与`V4L2`摄像机交互的库是Github库[dust-nv/Jetson-utils](https://github.com/dusty-nv/jetson-utils)。来自NVIDIA的Dustin Franklin在存储库中有用于相机、编解码器、GStreamer、CUDA和OpenCV的C/C++包装Linux工具。查看`camera`文件夹，获取通过`ioclt`与`V4L2`相机接口的示例。

## 3.2 GStreamer

GStreamer是Jetson相机架构的主要组成部分。GStreamer架构是可扩展的。NVIDIA实现了DeepStream元素，当将其添加到GStreamer管道中时，可以对视频流进行深度学习分析。NVIDIA称之为智能视频分析。

GStreamer有一些工具可以让它作为一个独立的应用程序运行。还有一些库允许GStreamer成为应用程序的一部分。GStreamer可以通过多种方式集成到应用程序中，例如使用OpenCV。例如，Github上流行的Python库[NVIDIA-AI-IOT/jetcam](https://github.com/NVIDIA-AI-IOT/jetcam)使用OpenCV来帮助管理相机和显示。

## 3.3 OpenCV

OpenCV是构建计算机视觉应用程序的流行框架。许多人使用OpenCV处理摄像头输入并在窗口中显示视频。这只需要几行代码就可以完成。OpenCV也很灵活。在默认的Jetson发行版中，OpenCV可以使用V4L2 (ioctl)接口或GStreamer接口。此外，OpenCV可以实现图形的GTK+或QT显示。

示例在JetsonHacks Github存储库:[USB-Camera](https://github.com/jetsonhacks/USB-Camera)。[usb-camera-simple.py](https://github.com/jetsonhacks/USB-Camera/blob/main/usb-camera-simple.py)利用V4L2相机输入。[usb-camera-gst.py](https://github.com/jetsonhacks/USB-Camera/blob/main/usb-camera-gst.py) 利用GStreamer作为它的输入接口。

这里需要注意的是，如果您使用的OpenCV版本与默认的Jetson发行版不同，那么您需要确保链接了适当的库，如V4L2和GStreamer。环顾Github，你应该能找到其他Jetson相机助手。看看周围的人在做什么总是很有趣的!



# 4.总结

Jetson的USB带宽可能不符合您的期望。例如在Jetson Nano和Xavier NX上，有4个USB 3端口。然而，USB端口连接到Jetson内部的一个集线器。这意味着您被集线器的速度限制，这是少于4个超高速USB端口。此外，为了节约电力，Jetson实现了所谓的“USB自动挂起”。这将在不使用时关闭USB端口。大多数USB摄像头都能正确处理这个问题，不会让USB端口自动挂起。然而，偶尔你可能会遇到不这样做的相机。您通常会遇到这种情况，因为在一段时间内，一切似乎都正常工作，然后就停止了。如果你不知道发生了什么，你会有点困惑。幸运的是，你可以关闭USB自动挂起。这在不同的操作系统版本和Jetson类型上有所不同。您应该能够搜索Jetson论坛，了解如何为您的特定配置控制这一点。

# 参考目录

[https://jetsonhacks.com/2022/02/02/in-practice-usb-cameras-on-jetson/](https://jetsonhacks.com/2022/02/02/in-practice-usb-cameras-on-jetson/)