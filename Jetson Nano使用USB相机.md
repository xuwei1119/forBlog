# Jetson Nano使用USB相机

# 1.简介

在NVIDIA Jetson开发工具包上运行V4L2 USB摄像头的代码示例。

下面是一些关于如何通过V4L2和GStreamer将USB摄像头与Jetson Developer Kit连接的例子。

Linux上的“即插即用”USB摄像头，例如网络摄像头，使用内核模块uvcvideo与Video 4 Linux子系统(V4L2)进行接口。一些专业相机，如Stereolabs ZED相机和英特尔RealSense相机也使用uvcvideo。

在Jetson Camera架构中，您可以使用V4L2或GStreamer(运行在V4L2之上)来与这些设备交互。在这些示例中使用OpenCV演示如何读取摄像机。OpenCV可与多个摄像机前端工作，包括V4L2和GStreamer。OpenCV包含在默认的Jetson软件安装中，并在默认配置中支持V4L2和GStreamer。

OpenCV只是与这些接口交互的一种方法。例如，以下是来自NVIDIA的关于如何使用输入/输出控制(ioctl):[jetson-utils](https://github.com/dusty-nv/jetson-utils)与V4L2相机交互的示例。

许多人使用OpenCV是为了方便使用，打开摄像头，读取帧，然后显示帧，而不必担心配置视频捕获接口或GUI显示接口。

# 2.示例参考

这些示例的目的是提供一个最小的示例脚本，以便能够使用USB摄像机。您将需要配置脚本以满足您的需求。您需要分配正确的视频地址。`/dev/videoX`必须匹配您的摄像机地址，其中`X`是一个数字。例如,`/dev/video1`。

## 2.1 usb-camera-simple.py

`usb-camera-simple.py`使用OpenCV的V4L2后端与摄像机交互(`cv2.CAP_V4L2`)。您可以使用V4L2摄像机属性来配置摄像机。在设置属性之前先创建相机。有些属性只有在与OpenCV正确集成时才可用。例如，尽管V4L2对H.264视频有一个`fourcc`的描述，但默认的OpenCV不是用libx264编译的。这意味着视频不能被正确解码。

创建`usb-camera-simple.py`文件，输入以下内容：

```python
#!/usr/bin/env python3
#
#  USB Camera - Simple
#
#  Copyright (C) 2021-22 JetsonHacks (info@jetsonhacks.com)
#
#  MIT License
#

import sys

import cv2

window_title = "USB Camera"

def show_camera():
    # ASSIGN CAMERA ADDRESS HERE
    camera_id = "/dev/video0"
    # Full list of Video Capture APIs (video backends): https://docs.opencv.org/3.4/d4/d15/group__videoio__flags__base.html
    # For webcams, we use V4L2
    video_capture = cv2.VideoCapture(camera_id, cv2.CAP_V4L2)
    """ 
    # How to set video capture properties using V4L2:
    # Full list of Video Capture Properties for OpenCV: https://docs.opencv.org/3.4/d4/d15/group__videoio__flags__base.html
    #Select Pixel Format:
    # video_capture.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*'YUYV'))
    # Two common formats, MJPG and H264
    # video_capture.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*'MJPG'))
    # Default libopencv on the Jetson is not linked against libx264, so H.264 is not available
    # video_capture.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*'H264'))
    # Select frame size, FPS:
    video_capture.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
    video_capture.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
    video_capture.set(cv2.CAP_PROP_FPS, 30)
    """
    if video_capture.isOpened():
        try:
            window_handle = cv2.namedWindow(
                window_title, cv2.WINDOW_AUTOSIZE )
            # Window
            while True:
                ret_val, frame = video_capture.read()
                # Check to see if the user closed the window
                # Under GTK+ (Jetson Default), WND_PROP_VISIBLE does not work correctly. Under Qt it does
                # GTK - Substitute WND_PROP_AUTOSIZE to detect if window has been closed by user
                if cv2.getWindowProperty(window_title, cv2.WND_PROP_AUTOSIZE) >= 0:
                    cv2.imshow(window_title, frame)
                else:
                    break
                keyCode = cv2.waitKey(10) & 0xFF
                # Stop the program on the ESC key or 'q'
                if keyCode == 27 or keyCode == ord('q'):
                    break

        finally:
            video_capture.release()
            cv2.destroyAllWindows()
    else:
        print("Unable to open camera")


if __name__ == "__main__":

    show_camera()
```

然后在命令行输入以下命令：

```shell
$ python3 usb-camera-simple.py
```

## 2.2 usb-camera-gst.py

`usb-camera-gst.py`使用GStreamer后端为OpenCV提供与摄像机的接口(`cv2.CAP_GSTREAMER`)。在使用GStreamer后端时，不能使用V4l2相机属性。有一个`H.264 GStreamer`管道示例(已注释掉)。这已经通过[罗技C920](https://amzn.to/3qQzKfi)网络摄像头进行了测试。

创建`usb-camera-gst.py`文件，输入以下内容：

```python
#!/usr/bin/env python3
#
#  USB Camera - Simple
#
#  Copyright (C) 2021-22 JetsonHacks (info@jetsonhacks.com)
#
#  MIT License
#
import sys
import cv2

window_title = "USB Camera"
# ASSIGN CAMERA ADDRESS to DEVICE HERE!
pipeline = " ! ".join(["v4l2src device=/dev/video0",
                       "video/x-raw, width=640, height=480, framerate=30/1",
                       "videoconvert",
                       "video/x-raw, format=(string)BGR",
                       "appsink"
                       ])

# Sample pipeline for H.264 video, tested on Logitech C920
h264_pipeline = " ! ".join(["v4l2src device=/dev/video0",
                            "video/x-h264, width=1280, height=720, framerate=30/1, format=H264",
                            "avdec_h264",
                            "videoconvert",
                            "video/x-raw, format=(string)BGR",
                            "appsink sync=false"
                            ])

def show_camera():

    # Full list of Video Capture APIs (video backends): https://docs.opencv.org/3.4/d4/d15/group__videoio__flags__base.html
    # For webcams, we use V4L2
    video_capture = cv2.VideoCapture(pipeline, cv2.CAP_GSTREAMER)

    if video_capture.isOpened():
        try:
            window_handle = cv2.namedWindow(
                window_title, cv2.WINDOW_AUTOSIZE)
            # Window
            while True:
                ret_val, frame = video_capture.read()
                # Check to see if the user closed the window
                # Under GTK+ (Jetson Default), WND_PROP_VISIBLE does not work correctly. Under Qt it does
                # GTK - Substitute WND_PROP_AUTOSIZE to detect if window has been closed by user
                if cv2.getWindowProperty(window_title, cv2.WND_PROP_AUTOSIZE) >= 0:
                    cv2.imshow(window_title, frame)
                else:
                    break
                keyCode = cv2.waitKey(10) & 0xFF
                # Stop the program on the ESC key or 'q'
                if keyCode == 27 or keyCode == ord('q'):
                    break

        finally:
            video_capture.release()
            cv2.destroyAllWindows()
    else:
        print("Error: Unable to open camera")


if __name__ == "__main__":

    show_camera()
```

然后在命令行输入以下命令：

```shell
$ python3 usb-camera-gst.py
```

GStreamer是一个非常灵活的框架，这实际上意味着在配置管道以获得最大性能时可能会遇到一些挑战。不同的摄像机在相同的管道上可能有不同的结果。

注意，OpenCV需要一个`BGR`像素输出输入。它通常放在`appsink`元素之前，并带有一个相关联的`videoconvert`元素。

## 2.3 face-detect-usb.py

`face-detect-usb.py`是一个读取帧并在OpenCV中操作它们的例子。人脸和眼睛检测使用哈尔级联。示例Haar级联在默认的Jetson分布中。

创建`face-detect-usb.py`文件，输入以下内容：

```python
# MIT License
# Copyright (c) 2019 JetsonHacks
# See LICENSE for OpenCV license and additional information

# https://docs.opencv.org/3.3.1/d7/d8b/tutorial_py_face_detection.html
# On the Jetson Nano, OpenCV comes preinstalled
# Data files are in /usr/share/OpenCV

import cv2

window_title = "Face Detect"

def face_detect():
    face_cascade = cv2.CascadeClassifier(
        "/usr/share/opencv4/haarcascades/haarcascade_frontalface_default.xml"
    )
    eye_cascade = cv2.CascadeClassifier(
        "/usr/share/opencv4/haarcascades/haarcascade_eye.xml"
    )
    # ASSIGN CAMERA ADDRESS HERE
    camera_id="/dev/video0"
    # Full list of Video Capture APIs (video backends): https://docs.opencv.org/3.4/d4/d15/group__videoio__flags__base.html
    # For webcams, we use V4L2
    video_capture = cv2.VideoCapture(camera_id, cv2.CAP_V4L2)
    """ 
    # How to set video capture properties using V4L2:
    # Full list of Video Capture Properties for OpenCV: https://docs.opencv.org/3.4/d4/d15/group__videoio__flags__base.html
    #Select Pixel Format:
    # video_capture.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*'YUYV'))
    # Two common formats, MJPG and H264
    # video_capture.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*'MJPG'))
    # Default libopencv on the Jetson is not linked against libx264, so H.264 is not available
    # video_capture.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*'H264'))
    # Select frame size, FPS:
    video_capture.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
    video_capture.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
    video_capture.set(cv2.CAP_PROP_FPS, 30)
    """

    if video_capture.isOpened():
        try:
            cv2.namedWindow(window_title, cv2.WINDOW_AUTOSIZE)
            # Start counting the number of frames read and displayed
            # left_camera.start_counting_fps()
            while True:
                _, frame = video_capture.read()
                gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
                faces = face_cascade.detectMultiScale(gray, 1.3, 5)

                for (x, y, w, h) in faces:
                    cv2.rectangle(frame, (x, y), (x + w, y + h), (255, 0, 0), 2)
                    roi_gray = gray[y : y + h, x : x + w]
                    roi_color = frame[y : y + h, x : x + w]
                    eyes = eye_cascade.detectMultiScale(roi_gray)
                    for (ex, ey, ew, eh) in eyes:
                        cv2.rectangle(
                            roi_color, (ex, ey), (ex + ew, ey + eh), (0, 255, 0), 2
                        )

                # Check to see if the user closed the window
                # Under GTK+, WND_PROP_VISIBLE does not work correctly. Under Qt it does
                # GTK - Substitute WND_PROP_AUTOSIZE to detect if window has been closed by user
                if cv2.getWindowProperty(window_title, cv2.WND_PROP_AUTOSIZE) >= 0:
                    cv2.imshow(window_title, frame)
                else:
                    break
                
                keyCode = cv2.waitKey(30) & 0xFF
                # Stop the program on the ESC key or 'q'
                if keyCode == 27 or keyCode == ord('q'):
                    break
        finally:
            video_capture.release()
            cv2.destroyAllWindows()
    else:
        print("Unable to open camera")


if __name__ == "__main__":
    face_detect()
```

然后在命令行输入以下命令：

```shell
$ python3 face-detect-usb.py
```

# 3. OpenCV

示例使用OpenCV渲染帧。Jetson上的标准OpenCV版本目前使用GTK+作为图形输出。此外，这里编译的OpenCV，支持GStreamer和V4L2摄像机输入。如果遇到问题，并且没有安装标准OpenCV，请检查以确保已加载并选择了这些文件。

在Python3中，检查OpenCV构建时选择的选项:

```python
>>> import cv2
>>> print(cv2.getBuildInformation())
```

GUI部分列出了图形后端，视频I/O部分列出了视频前端。

## 3.1 工具

使用V4L2相机的一个有价值的工具是V`4L2-ctl`。如何安装:

```shell
$ sudo apt install v4l-utils
```

有用的命令:

+ 返回一个相机列表

  ```shell
  $ v4l2-ctl --list-devices
  ```

+ 获取相机像素格式

  获取支持的像素格式列表。`/dev/videoX`为摄像机地址，例如

  ```shell
  $ v4l2-ctl --list-formats-ext -d /dev/video0
  ```

+ 获取相机的所有信息

  列出驱动程序，像素格式，帧大小，控件

  ```shell
  $ v4l2-ctl --all -d /dev/video0
  ```

+ ### canberra-gtk-module

  如果你看到错误:`Failed to load module "canberra-gtk-module"`

  一般来说，这不会引起问题。然而，你可以用以下方法删除错误:

  ```shell
  $ sudo apt install libcanberra-gtk-module
  ```

  

# 参考目录

[https://github.com/JetsonHacksNano/USB-Camera](https://github.com/JetsonHacksNano/USB-Camera)