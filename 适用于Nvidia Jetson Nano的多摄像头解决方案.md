适用于Nvidia Jetson Nano的多摄像头解决方案

# 1.在Jetson Nano上使用多摄像头的注意事项

Nvidia Jetson系列以其在人工智能方面的非凡性能和小巧的外形而闻名。由于Jetson TX1/TX2可能过于昂贵而无法在爱好者中流行，因此低端性价比的Jetson Nano于2019年4月推出，仅售99美元，它立即成为另一个制造商喜爱的平台Raspberry的巨大竞争对手。

Jetson Nano的性能远远优于Rpi，尤其是在需要更多计算能力的[相机应用](https://www.arducam.com/product-category/nvidia-jetson-nano-camera-arducam/)中，例如：

+ 无人机
+ 高级机器人视觉
+ 自动驾驶汽车
+ 神经网络
+ 运动跟踪
+ 机器学习
+ 计算机视觉
+ 人工智能

![image-20221104091843538](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221104091843538.png)

适用于 Jetson Nano 的 Arducam 高清摄像头            

它甚至设计有Raspberry Pi风格的40GPIO引脚和MIPI CSI-2摄像头连接器，以兼容Raspberry Pi生态系统中的现有配件，例如[Raspberry摄像头模块](https://www.arducam.com/arducam-for-raspberry-pi-4-hd-cameras-hand-on-arducam-ensures-your-compatibilities-while-exploring-new-opportunities/)和Raspberry HAT。由于它的设计类似于Raspberry Pi，因此很难将多个摄像头连接到Jetson Nano板上。不过，这对Arducam来说不是问题，因为已经在 [Raspberry Pi 平台上解决了这个问题](https://www.arducam.com/docs/cameras-for-raspberry-pi/multi-camera-adapter-board/)，这些解决方案也可以应用到 Jetson Nano 上。

# 2. Arducam的3个多摄像头解决方案

![image-20221104095926599](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221104095926599.png)

Arducam 致力于使用我们的高清摄像头模块和摄像头适配器为单板计算机提供嵌入式和物联网摄像头解决方案。我们在 Raspberry Pi 上的多摄像头解决方案方面处于领先地位，可以轻松地将所有这些功能带入 Jetson Nano 平台。目前，我们提供三种方法将多于 1 个摄像头连接到单个 Jetson Nano，分别是用于顺序激活 多达4 个摄像头模块的[多摄像头适配器板](https://www.arducam.com/product/multi-camera-v2-1-adapter-raspberry-pi/)、用于同步双摄像头视觉的[立体摄像头 HAT](https://www.arducam.com/dual-camera-hat-synchronize-stereo-pi-raspberry/) ，以及用于 USB 多摄像头的[USB 3.0 摄像头屏蔽](https://www.arducam.com/product/usb3-0-camera-shield/)相机解决方案。

## 2.1 Arducam 多路复用器：Jetson Nano 的 4 合 1 适配器板

![image-20221104100735133](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221104100735133.png)

![image-20221104101011620](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221104101011620.png)    ![image-20221104101018895](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221104101018895.png)

多摄像头适配器板，也被称为摄像头多路复用器，可以将多达4个MIPI CSI摄像头连接到树莓派或Jetson Nano上的单个MIPI摄像头端口。唯一的问题是它是基于时分复用的，所以一次只能激活一个摄像头，适配器会在软件的帮助下在这些摄像头之间快速切换，使它像四个摄像头同时工作。它适用于不需要同步摄像流，只需要静态图像分析的系统。

## 2.2 USB3.0 摄像头屏蔽

![image-20221104101228700](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221104101228700.png)

Jetson Nano的另一个高速接口是USB 3.0端口，它可以提供相当于4GByte/s的5Gbps带宽。Arducam有一个USB 3.0摄像头屏蔽解决方案，也可以在Jetson Nano上使用。您可以将多个USB3摄像头连接到Jetson Nano上，只要USB总带宽在4GByte/s限制内。相机之间的同步需要全局快门相机的外部触发输入功能。

[点击这里了解更多关于Arducam USB摄像头屏蔽](https://www.arducam.com/docs/usb-cameras/introduction-to-arducam-usb-camera-shields/)。

## 2.3 不久的将来:立体相机HAT

![image-20221104101507046](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221104101507046.png)

我们刚刚发布了用于 Raspberry pi 的革命性立体摄像头 HAT，它允许您将同时工作的两个摄像头连接到 Raspberry Pi 上的单个 MIPI CSI-2 端口。最重要的是，这两个摄像头是完全同步的。由于 Jetson Nano 使用与 RPi 相同的相机连接器，因此我们可以将其带到这个平台上。Arducam 现在正在努力将这款立体摄像机 HAT 移植到 Jetson Nano 上。

## 2.4 Jetson Nano + Arducam – 使用多摄像头做更多事情

Jetson 系列的推出旨在凭借其 GPU 能力和 AI 性能“实现一切自动化”，他们还发布了一款可用于项目的 DIY 自主机器人套件，名为 JetBot，以展示 Jetson Nano 的能力。但是，我们认为您可以使用社区软件和 Arducam 的多摄像头解决方案做更多事情。

我们以自动驾驶机器人汽车为例。现在，您可以在 Jetson Nano 上使用 CUDA 编程（Nvidia 的计算平台和 API）和机器人操作系统（ROS，其中集成了 AI 方法和算法）。使用我们的立体摄像头 HAT，您可以将前视摄像头和后视摄像头添加到基于 Jetson Nano 的机器人中或利用立体视觉，当您从事需要同时定位和映射 (SLAM) 的行业项目时，它可能会很有用以获得更好的导航甚至增强现实。 

# 参考目录

[https://www.arducam.com/multi-camera-solutions-for-nvidia-jetson-nano/](https://www.arducam.com/multi-camera-solutions-for-nvidia-jetson-nano/)