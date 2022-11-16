# 基于OpenCV的Jetson Nano实现实时人脸检测

使用树莓派相机和OpenCV实现人脸检测!

Jetson Nano是一款支持GPU的边缘计算平台，用于人工智能和深度学习应用。它由一个64位的四核ARM-CortexA57 CPU提供支持，板载4GB RAM。它拥有128核的英伟达Maxwell GPU，专门用于几个AI和深度学习应用，使其适合原型开发和生产。

Jetson Nano开发工具包包含了Nano模块、配件、针孔和端口，可以开箱即用。它运行的是名为Linux4Tegra的定制版Ubuntu 18.04，用于支持英伟达Jetson模块的Tegra系列芯片。

Jetson Nano开发工具包有一个移动工业处理器接口(MIPI)驱动的相机串行接口(CSI)端口，支持常见的相机模块，如 [Raspberry Pi camera v2](https://www.raspberrypi.org/products/camera-module-v2/)或 [Arducam camera](https://www.arducam.com/product/5mp-ov5647-motorized-focus-camera-sensor-raspberry-pi/)。这些相机体积小，但适用于机器学习和计算机视觉应用，如物体检测，人脸识别，图像分割，使用片上系统(SoC)估计视觉里程。商用USB摄像头也适用于这项任务，但容易出现延迟和传输开销。

对于本教程中的人脸检测任务，我们使用RPi摄像模块v2。它由索尼IMX219 8-MP传感器提供动力。它支持静态图像以及不同分辨率的高清视频，包括1080p@30FPS、720p@60FPS和VGA90。

# 1.准备资料

+ Jetson Nano开发工具包

+ [Raspberry Pi Camera v2](https://www.raspberrypi.org/products/camera-module-v2/)

+ 开发人员工具包的电源

+ 支持HDMI的键盘，鼠标和显示器



# 2. 将相机连接到Jetson Nano上

+ 安装Jetson Nano Developer Kit。

+ 添加键盘、鼠标和显示器。

+ 拔出CSI接口，将摄像头色带线插入该接口。确保端口上的连接引线与彩带上的连接引线对齐。彩带上的连接应该面向散热器。

+ 启动开发人员工具包。

+ 验证摄像头是否正确配置和识别。

![image-20221102173914354](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102173914354.png)

​                                                                                                                          摄像头彩带线连接CSI口。

Nvidia提供的JetPack SDK已经支持带有预装驱动程序的RPi相机，并且很容易作为即插即用的外设使用，消除了驱动程序安装和兼容性问题的痛苦。

不能使用内置的用于在Ubuntu操作系统上录制视频和拍照的GNOME应用Cheese检查摄像头连接。这是因为RPi相机本身并不兼容UV4L，默认的Linux应用程序无法识别它。

![image-20221102174120316](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102174120316.png)

​                                                                                             树莓派相机模块V2与Jetson Nano开发工具包

检查摄像头连接的简单命令行指令是:

```shell
akshay@jetson_nano:~$ ls -l /dev/video0
Crw-rw----+ 1 root video 81, 0 Jan 2 14:30 /dev/video0
```

如果命令的输出为空白，则表示开发人员工具包没有附加摄像头。英伟达Jetson系列使用GStreamer管道来处理媒体应用程序。GStreamer是一个多媒体框架，用于后端处理任务，如格式修改、显示驱动程序协调和数据处理。对于树莓派(Raspberry Pi)相机，我们将使用同样的方法。

更古怪的方法是检查摄像头连接，使用GStreamer应用程序gst-launch启动一个显示窗口，并确认可以从摄像头看到实时流。

```shell


akshay@jetson_nano:~$ gst-launch-1.0 nvarguscamerasrc ! 'video/x-raw(memory:NVMM),width=3820, height=2464, framerate=21/1, format=NV12' ! nvvidconv flip-method=0 ! 'video/x-raw,width=480, height=320' ! nvvidconv ! nvegltransform ! nveglglessink -e
```

上面的命令创建了一个GStreamer管道，其中的元素以!定义管道的属性。它启动一个宽480高320的显示窗口，FPS为10帧，还有其他几个Nvidia配置参数(超出了本文的范围)。关于管道的不同元素的详细信息可以在gst-launch的文档网站上找到。

# 3.安装OpenCV

OpenCV是一个开源的计算机视觉库，用C++编写，但也有Python和Lua的包装器。Jetson Nano的图像文件上的JetPack SDK预装了OpenCV。OpenCV已经使用Haar Cascades和Viola Jones算法训练了人脸检测、眼睛检测等模型。OpenCV网站上有一个算法实现的[教程](https://docs.opencv.org/3.3.1/d7/d8b/tutorial_py_face_detection.html)。

## 3.1 简单的Python人脸检测应用程序

```python
import cv2
import numpy as np

HAAR_CASCADE_XML_FILE_FACE = "/usr/share/OpenCV/haarcascades/haarcascade_frontalface_default.xml"

GSTREAMER_PIPELINE = 'nvarguscamerasrc ! video/x-raw(memory:NVMM), width=3280, height=2464, format=(string)NV12, framerate=21/1 ! nvvidconv flip-method=0 ! video/x-raw, width=960, height=616, format=(string)BGRx ! videoconvert ! video/x-raw, format=(string)BGR ! appsink'

def faceDetect():
    # Obtain face detection Haar cascade XML files from OpenCV
    face_cascade = cv2.CascadeClassifier(HAAR_CASCADE_XML_FILE_FACE)

    # Video Capturing class from OpenCV
    video_capture = cv2.VideoCapture(GSTREAMER_PIPELINE, cv2.CAP_GSTREAMER)
    if video_capture.isOpened():
        cv2.namedWindow("Face Detection Window", cv2.WINDOW_AUTOSIZE)

        while True:
            return_key, image = video_capture.read()
            if not return_key:
                break

            grayscale_image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
            detected_faces = face_cascade.detectMultiScale(grayscale_image, 1.3, 5)

            # Create rectangle around the face in the image canvas
            for (x_pos, y_pos, width, height) in detected_faces:
                cv2.rectangle(image, (x_pos, y_pos), (x_pos + width, y_pos + height), (0, 0, 0), 2)

            cv2.imshow("Face Detection Window", image)

            key = cv2.waitKey(30) & 0xff
            # Stop the program on the ESC key
            if key == 27:
                break

        video_capture.release()
        cv2.destroyAllWindows()
    else:
        print("Cannot open Camera")

if __name__ == "__main__":
    faceDetect()
```

## 3.2 代码描述

代码基本上使用预先训练的存储在OpenCV库中的Haar Cascade实现。正如前面提到的，代码使用GStreamer管道来创建相机和操作系统之间的接口。该管道用于创建VideoCapture()对象。

来自摄像机直播流的每个图像帧都被处理和测试人脸检测。该代码实现了将彩色图像转换为灰度图像的附加步骤，因为颜色并不决定面部特征。这避免了计算开销并提高了性能。一旦确定图像中包含人脸，就会在边界周围画一个矩形。

# 参考目录

[https://maker.pro/nvidia-jetson/tutorial/real-time-face-detection-on-jetson-nano-using-opencv](https://maker.pro/nvidia-jetson/tutorial/real-time-face-detection-on-jetson-nano-using-opencv)

