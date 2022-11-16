# Jetson Nano使用CSI相机

# 1 简介

使用带有CSI相机端口的NVIDIA Jetson Developer Kits的MIPI-CSI(2)相机(如树莓派版本2相机)的简单示例。这包括最近的Jetson Nano和Jetson Xavier NX。

## 1.1 背景

自2014年推出首款Jetson以来，最受欢迎的功能之一就是树莓派的摄像支持。Jetson Nano内置了支持，无需造假。Jetson家族一直支持MIPI-CSI相机。MIPI代表移动工业处理器接口，CSI代表相机串行接口。该协议用于摄像机与主机设备之间的高速传输。基本上它是一个直接连接处理器的软管，不像USB堆栈那样有很多开销。

然而，对于那些不是专业硬件/软件开发人员的人来说，通过这种接口访问廉价的成像设备是一个挑战。

这有几个原因。首先，相机连接和布线是通过一个连接器，大多数爱好者没有很好的访问。此外，在Linux内核中，相机的驱动程序有很多不稳定的地方，在成像发生之前需要对设备树进行操作。像我说的，专业的东西。大多数人选择阻力最小的方法，简单地使用USB摄像头。

## 1.2 树莓派摄像机模块V2

与此同时，最受欢迎的CSI-2相机之一是树莓派相机模块V2。该摄像机有一个带状连接器，它使用一个简单的连接器连接到开发板。树莓派相机的核心是一个索尼IMX-219成像仪，有不同的版本，有带红外滤镜的，也有不带红外滤镜的。在Pi NoIR相机(NoIR= No infrared)中去掉红外滤镜，使人们可以在与红外照明配合的情况下构建“夜视”相机。而且售价约25美元，物超所值!它们是终极的终极相机吗?不是，但你可以不用花很多钱就能使用这种相机。注意:V1树莓派相机模块不兼容默认的Jetson Nano安装。基本内核模块中不包括成像元素的驱动程序。

Jetson Nano开发工具包有一个RPi相机兼容连接器!IMX219的设备驱动程序已经安装好，摄像头已经配置好。只要插上电源，就可以了。

这里有一个专业的建议:在使用新相机之前，先去掉覆盖在相机镜头上的保护性塑料薄膜。你会得到更好的图像(不要问我是怎么知道的)。

## 1.3 必备知识

对于nano和Xavier NX，摄像头安装在开发板的MIPI-CSI摄像头连接器上。相机色带上的引脚应该面向Jetson模块，胶带条纹面向外。

一些Jetson开发套件有两个CSI摄像头插槽。你可以在`GStreamer`的`nvarguscamerasrc`元素中使用`sensor_mode`属性来指定相机。有效值为0或1(如果没有指定，默认值为0)，即。

```shell
nvarguscamerasrc sensor_id=0
```

测试相机:

```shell
# Simple Test
#  Ctrl^C to exit
# sensor_id selects the camera: 0 or 1 on Jetson Nano B01
$ gst-launch-1.0 nvarguscamerasrc sensor_id=0 ! nvoverlaysink

# More specific - width, height and framerate are from supported video modes
# Example also shows sensor_mode parameter to nvarguscamerasrc
# See table below for example video modes of example sensor
$ gst-launch-1.0 nvarguscamerasrc sensor_id=0 ! \
   'video/x-raw(memory:NVMM),width=1920, height=1080, framerate=30/1' ! \
   nvvidconv flip-method=0 ! 'video/x-raw,width=960, height=540' ! \
   nvvidconv ! nvegltransform ! nveglglessink -e
```

注意:相机可能报告不同于下面所示内容。您可以使用上面简单的`gst-launch`示例来确定所使用的传感器报告的相机模式。

```shell
GST_ARGUS: 1920 x 1080 FR = 29.999999 fps Duration = 33333334 ; Analog Gain range min 1.000000, max 10.625000; Exposure Range min 13000, max 683709000;
```

此外，显示可能对宽度和高度很敏感(在上面的例子中，width=960, height=540)。如果你遇到问题，检查一下你的显示宽度和高度的比例是否与所选的相机帧大小相同(在上面的例子中，960x540是1920x1080的1/4)。

# 2.案例

## 2.1 simple_camera.py

`simple_camera.py`是一个Python脚本，它从相机读取并使用OpenCV将帧显示到屏幕上:

创建`simple_camera.py`文件，输入以下内容：

```python
# MIT License
# Copyright (c) 2019-2022 JetsonHacks

# Using a CSI camera (such as the Raspberry Pi Version 2) connected to a
# NVIDIA Jetson Nano Developer Kit using OpenCV
# Drivers for the camera and OpenCV are included in the base image

import cv2

""" 
gstreamer_pipeline returns a GStreamer pipeline for capturing from the CSI camera
Flip the image by setting the flip_method (most common values: 0 and 2)
display_width and display_height determine the size of each camera pane in the window on the screen
Default 1920x1080 displayd in a 1/4 size window
"""

def gstreamer_pipeline(
    sensor_id=0,
    capture_width=1920,
    capture_height=1080,
    display_width=960,
    display_height=540,
    framerate=30,
    flip_method=0,
):
    return (
        "nvarguscamerasrc sensor-id=%d !"
        "video/x-raw(memory:NVMM), width=(int)%d, height=(int)%d, framerate=(fraction)%d/1 ! "
        "nvvidconv flip-method=%d ! "
        "video/x-raw, width=(int)%d, height=(int)%d, format=(string)BGRx ! "
        "videoconvert ! "
        "video/x-raw, format=(string)BGR ! appsink"
        % (
            sensor_id,
            capture_width,
            capture_height,
            framerate,
            flip_method,
            display_width,
            display_height,
        )
    )


def show_camera():
    window_title = "CSI Camera"

    # To flip the image, modify the flip_method parameter (0 and 2 are the most common)
    print(gstreamer_pipeline(flip_method=0))
    video_capture = cv2.VideoCapture(gstreamer_pipeline(flip_method=0), cv2.CAP_GSTREAMER)
    if video_capture.isOpened():
        try:
            window_handle = cv2.namedWindow(window_title, cv2.WINDOW_AUTOSIZE)
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

然后执行以下命令：

```shell
$ python simple_camera.py
```

## 2.2 face_detect.py

`face_detect.py`是一个python脚本，它从摄像头读取数据，并使用Haar Cascades来检测人脸和眼睛:

创建`face_detect.py`文件，输入以下内容：

```python
# MIT License
# Copyright (c) 2019-2022 JetsonHacks
# See LICENSE for OpenCV license and additional information

# https://docs.opencv.org/3.3.1/d7/d8b/tutorial_py_face_detection.html
# On the Jetson Nano, OpenCV comes preinstalled
# Data files are in /usr/sharc/OpenCV

import cv2

# gstreamer_pipeline returns a GStreamer pipeline for capturing from the CSI camera
# Defaults to 1920x1080 @ 30fps
# Flip the image by setting the flip_method (most common values: 0 and 2)
# display_width and display_height determine the size of the window on the screen
# Notice that we drop frames if we fall outside the processing time in the appsink element

def gstreamer_pipeline(
    capture_width=1920,
    capture_height=1080,
    display_width=960,
    display_height=540,
    framerate=30,
    flip_method=0,
):
    return (
        "nvarguscamerasrc ! "
        "video/x-raw(memory:NVMM), "
        "width=(int)%d, height=(int)%d, framerate=(fraction)%d/1 ! "
        "nvvidconv flip-method=%d ! "
        "video/x-raw, width=(int)%d, height=(int)%d, format=(string)BGRx ! "
        "videoconvert ! "
        "video/x-raw, format=(string)BGR ! appsink drop=True"
        % (
            capture_width,
            capture_height,
            framerate,
            flip_method,
            display_width,
            display_height,
        )
    )


def face_detect():
    window_title = "Face Detect"
    face_cascade = cv2.CascadeClassifier(
        "/usr/share/opencv4/haarcascades/haarcascade_frontalface_default.xml"
    )
    eye_cascade = cv2.CascadeClassifier(
        "/usr/share/opencv4/haarcascades/haarcascade_eye.xml"
    )
    video_capture = cv2.VideoCapture(gstreamer_pipeline(), cv2.CAP_GSTREAMER)
    if video_capture.isOpened():
        try:
            cv2.namedWindow(window_title, cv2.WINDOW_AUTOSIZE)
            while True:
                ret, frame = video_capture.read()
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
    face_detect()
```

然后执行以下命令：

```shell
$ python face_detect.py
```

Haar Cascades是一种基于机器学习的方法，它从大量正和负样本图像中训练级联函数。然后使用该函数检测其他图像中的对象。参见:https://docs.opencv.org/3.3.1/d7/d8b/tutorial_py_face_detection.html

## 2.3 dual_camera.py

注意:你需要安装numpy才能让Dual Camera Python示例正常工作:

```shell
$ pip3 install numpy
```

此示例用于较新的Jetson板(Jetson Nano, Jetson Xavier NX)，具有两个CSI-MIPI摄像头端口。这是一个简单的Python程序，它读取两个CSI摄像机并在一个窗口中显示它们。窗口大小是1920x540。为了性能考虑，脚本使用一个单独的线程来读取每个相机图像流。

创建`dual_camera.py`文件，输入以下内容：

```python
# MIT License
# Copyright (c) 2019-2022 JetsonHacks

# A simple code snippet
# Using two  CSI cameras (such as the Raspberry Pi Version 2) connected to a
# NVIDIA Jetson Nano Developer Kit with two CSI ports (Jetson Nano, Jetson Xavier NX) via OpenCV
# Drivers for the camera and OpenCV are included in the base image in JetPack 4.3+

# This script will open a window and place the camera stream from each camera in a window
# arranged horizontally.
# The camera streams are each read in their own thread, as when done sequentially there
# is a noticeable lag

import cv2
import threading
import numpy as np


class CSI_Camera:

    def __init__(self):
        # Initialize instance variables
        # OpenCV video capture element
        self.video_capture = None
        # The last captured image from the camera
        self.frame = None
        self.grabbed = False
        # The thread where the video capture runs
        self.read_thread = None
        self.read_lock = threading.Lock()
        self.running = False

    def open(self, gstreamer_pipeline_string):
        try:
            self.video_capture = cv2.VideoCapture(
                gstreamer_pipeline_string, cv2.CAP_GSTREAMER
            )
            # Grab the first frame to start the video capturing
            self.grabbed, self.frame = self.video_capture.read()

        except RuntimeError:
            self.video_capture = None
            print("Unable to open camera")
            print("Pipeline: " + gstreamer_pipeline_string)


    def start(self):
        if self.running:
            print('Video capturing is already running')
            return None
        # create a thread to read the camera image
        if self.video_capture != None:
            self.running = True
            self.read_thread = threading.Thread(target=self.updateCamera)
            self.read_thread.start()
        return self

    def stop(self):
        self.running = False
        # Kill the thread
        self.read_thread.join()
        self.read_thread = None

    def updateCamera(self):
        # This is the thread to read images from the camera
        while self.running:
            try:
                grabbed, frame = self.video_capture.read()
                with self.read_lock:
                    self.grabbed = grabbed
                    self.frame = frame
            except RuntimeError:
                print("Could not read image from camera")
        # FIX ME - stop and cleanup thread
        # Something bad happened

    def read(self):
        with self.read_lock:
            frame = self.frame.copy()
            grabbed = self.grabbed
        return grabbed, frame

    def release(self):
        if self.video_capture != None:
            self.video_capture.release()
            self.video_capture = None
        # Now kill the thread
        if self.read_thread != None:
            self.read_thread.join()


""" 
gstreamer_pipeline returns a GStreamer pipeline for capturing from the CSI camera
Flip the image by setting the flip_method (most common values: 0 and 2)
display_width and display_height determine the size of each camera pane in the window on the screen
Default 1920x1080
"""


def gstreamer_pipeline(
    sensor_id=0,
    capture_width=1920,
    capture_height=1080,
    display_width=1920,
    display_height=1080,
    framerate=30,
    flip_method=0,
):
    return (
        "nvarguscamerasrc sensor-id=%d ! "
        "video/x-raw(memory:NVMM), width=(int)%d, height=(int)%d, framerate=(fraction)%d/1 ! "
        "nvvidconv flip-method=%d ! "
        "video/x-raw, width=(int)%d, height=(int)%d, format=(string)BGRx ! "
        "videoconvert ! "
        "video/x-raw, format=(string)BGR ! appsink"
        % (
            sensor_id,
            capture_width,
            capture_height,
            framerate,
            flip_method,
            display_width,
            display_height,
        )
    )


def run_cameras():
    window_title = "Dual CSI Cameras"
    left_camera = CSI_Camera()
    left_camera.open(
        gstreamer_pipeline(
            sensor_id=0,
            capture_width=1920,
            capture_height=1080,
            flip_method=0,
            display_width=960,
            display_height=540,
        )
    )
    left_camera.start()

    right_camera = CSI_Camera()
    right_camera.open(
        gstreamer_pipeline(
            sensor_id=1,
            capture_width=1920,
            capture_height=1080,
            flip_method=0,
            display_width=960,
            display_height=540,
        )
    )
    right_camera.start()

    if left_camera.video_capture.isOpened() and right_camera.video_capture.isOpened():

        cv2.namedWindow(window_title, cv2.WINDOW_AUTOSIZE)

        try:
            while True:
                _, left_image = left_camera.read()
                _, right_image = right_camera.read()
                # Use numpy to place images next to each other
                camera_images = np.hstack((left_image, right_image)) 
                # Check to see if the user closed the window
                # Under GTK+ (Jetson Default), WND_PROP_VISIBLE does not work correctly. Under Qt it does
                # GTK - Substitute WND_PROP_AUTOSIZE to detect if window has been closed by user
                if cv2.getWindowProperty(window_title, cv2.WND_PROP_AUTOSIZE) >= 0:
                    cv2.imshow(window_title, camera_images)
                else:
                    break

                # This also acts as
                keyCode = cv2.waitKey(30) & 0xFF
                # Stop the program on the ESC key
                if keyCode == 27:
                    break
        finally:

            left_camera.stop()
            left_camera.release()
            right_camera.stop()
            right_camera.release()
        cv2.destroyAllWindows()
    else:
        print("Error: Unable to open both cameras")
        left_camera.stop()
        left_camera.release()
        right_camera.stop()
        right_camera.release()



if __name__ == "__main__":
    run_cameras()
```

然后执行以下命令：

```shell
$ python3 dual_camera.py
```

## 2.4 simple_camera.cpp

最后一个例子是一个简单的C++程序，它从摄像头读取数据并使用OpenCV显示到屏幕上:

创建`simple_camera.cpp`文件，输入以下内容：

```c++
// simple_camera.cpp
// MIT License
// Copyright (c) 2019-2022 JetsonHacks
// See LICENSE for OpenCV license and additional information
// Using a CSI camera (such as the Raspberry Pi Version 2) connected to a 
// NVIDIA Jetson Nano Developer Kit using OpenCV
// Drivers for the camera and OpenCV are included in the base image

#include <opencv2/opencv.hpp>

std::string gstreamer_pipeline (int capture_width, int capture_height, int display_width, int display_height, int framerate, int flip_method) {
    return "nvarguscamerasrc ! video/x-raw(memory:NVMM), width=(int)" + std::to_string(capture_width) + ", height=(int)" +
           std::to_string(capture_height) + ", framerate=(fraction)" + std::to_string(framerate) +
           "/1 ! nvvidconv flip-method=" + std::to_string(flip_method) + " ! video/x-raw, width=(int)" + std::to_string(display_width) + ", height=(int)" +
           std::to_string(display_height) + ", format=(string)BGRx ! videoconvert ! video/x-raw, format=(string)BGR ! appsink";
}

int main()
{
    int capture_width = 1280 ;
    int capture_height = 720 ;
    int display_width = 1280 ;
    int display_height = 720 ;
    int framerate = 30 ;
    int flip_method = 0 ;

    std::string pipeline = gstreamer_pipeline(capture_width,
	capture_height,
	display_width,
	display_height,
	framerate,
	flip_method);
    std::cout << "Using pipeline: \n\t" << pipeline << "\n";
 
    cv::VideoCapture cap(pipeline, cv::CAP_GSTREAMER);
    if(!cap.isOpened()) {
	std::cout<<"Failed to open camera."<<std::endl;
	return (-1);
    }

    cv::namedWindow("CSI Camera", cv::WINDOW_AUTOSIZE);
    cv::Mat img;

    std::cout << "Hit ESC to exit" << "\n" ;
    while(true)
    {
    	if (!cap.read(img)) {
		std::cout<<"Capture read error"<<std::endl;
		break;
	}
	
	cv::imshow("CSI Camera",img);
	int keycode = cv::waitKey(10) & 0xff ; 
        if (keycode == 27) break ;
    }

    cap.release();
    cv::destroyAllWindows() ;
    return 0;
}

```

然后执行以下命令：

```shell
$ g++ -std=c++11 -Wall -I/usr/lib/opencv -I/usr/include/opencv4 simple_camera.cpp -L/usr/lib -lopencv_core -lopencv_highgui -lopencv_videoio -o simple_camera

$ ./simple_camera
```

这个程序是一个简单的大纲，不能很好地处理所需的错误检查。要获得更好的C++代码，请使用https://github.com/dusty-nv/jetson-utils

下面这个cpp文件更加简单，也可以使用：

创建`gstreamer_view.cpp`文件，输入以下内容：

```c++
/*
  Example code for displaying gstreamer video from the CSI port of the Nvidia Jetson in OpenCV.
  Created by Peter Moran on 7/29/17.
  https://gist.github.com/peter-moran/742998d893cd013edf6d0c86cc86ff7f
*/

#include <opencv2/opencv.hpp>

std::string get_tegra_pipeline(int width, int height, int fps) {
    return "nvarguscamerasrc ! video/x-raw(memory:NVMM), width=(int)" + std::to_string(width) + ", height=(int)" +
           std::to_string(height) + ", format=(string)NV12, framerate=(fraction)" + std::to_string(fps) +
           "/1 ! nvvidconv flip-method=0 ! video/x-raw, format=(string)BGRx ! videoconvert ! video/x-raw, format=(string)BGR ! appsink";
}

int main() {
    // Options
    int WIDTH = 1280;
    int HEIGHT = 720;
    int FPS = 60;

    // Define the gstream pipeline
    std::string pipeline = get_tegra_pipeline(WIDTH, HEIGHT, FPS);
    std::cout << "Using pipeline: \n\t" << pipeline << "\n";

    // Create OpenCV capture object, ensure it works.
    cv::VideoCapture cap(pipeline, cv::CAP_GSTREAMER);
    if (!cap.isOpened()) {
        std::cout << "Connection failed";
        return -1;
    }

    // View video
    cv::Mat frame;
    while (1) {
        cap >> frame;  // Get a new frame from camera

        // Display frame
        imshow("Display window", frame);
        cv::waitKey(1); //needed to show frame
    }
}
```

这里有个示例程序。这个程序需要在安装OpenCV时启用GStreamer支持。这个例子是用L4T 32.2.1和OpenCV 4.1.1进行测试的。例子假设Nano上安装了一个RPi V2 CSI摄像机。您可能需要更改USB相机的`/dev/video*`路径。

编译`gstreamer_view.cpp`并运行:

```c++
$ gcc -std=c++11 `pkg-config --cflags opencv` `pkg-config --libs opencv` gstreamer_view.cpp -o gstreamer_view -lstdc++ -lopencv_core -lopencv_highgui -lopencv_videoio
$ ./gstreamer_view
```



# 3.必备基础操作

## 3.1 相机图像格式

您可以使用`v4l2-ctl`来确定摄像机功能。对于树莓派V2相机，`$ sudo apt-get install v4l-utils `典型的输出是(假设相机是`/dev/video0`):

```shell
$ v4l2-ctl --list-formats-ext
ioctl: VIDIOC_ENUM_FMT
	Index       : 0
	Type        : Video Capture
	Pixel Format: 'RG10'
	Name        : 10-bit Bayer RGRG/GBGB
		Size: Discrete 3280x2464
			Interval: Discrete 0.048s (21.000 fps)
		Size: Discrete 3280x1848
			Interval: Discrete 0.036s (28.000 fps)
		Size: Discrete 1920x1080
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.017s (60.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.017s (60.000 fps)
```

## 3.2 GStreamer参数

对于`GStreamer`管道，`nvvidconv flip-method`参数可以旋转/翻转图像。当相机的安装方向与默认方向不同时，这是很有用的。

```shell

flip-method         : video flip methods
                        flags: readable, writable, controllable
                        Enum "GstNvVideoFlipMethod" Default: 0, "none"
                           (0): none             - Identity (no rotation)
                           (1): counterclockwise - Rotate counter-clockwise 90 degrees
                           (2): rotate-180       - Rotate 180 degrees
                           (3): clockwise        - Rotate clockwise 90 degrees
                           (4): horizontal-flip  - Flip horizontally
                           (5): upper-right-diagonal - Flip across upper right/lower left diagonal
                           (6): vertical-flip    - Flip vertically
                           (7): upper-left-diagonal - Flip across upper left/low
```

## 3.3 OpenCV和Python

从L4T 32.2.1 / JetPack 4.2.2开始，OpenCV内置了GStreamer支持。OpenCV的版本是3.3.1。请注意，如果你使用的是早期版本的OpenCV(很可能是从Ubuntu存储库安装的)，你会得到“无法打开相机”的错误。

如果您可以从命令行在GStreamer中打开相机，但在Python中打开相机时遇到问题，请检查OpenCV版本。

```python
>>>cv2.__version__
```

# 参考目录

[https://github.com/JetsonHacksNano/CSI-Camera](https://github.com/JetsonHacksNano/CSI-Camera)