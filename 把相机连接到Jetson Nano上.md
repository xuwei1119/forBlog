# 把相机连接到Jetson Nano上

# 1.前提条件

+ 已经安装了NVIDIA Jetson Nano (4GB, B01)。

+ 已经安装了OpenCV（可选）

# 2.需要的器件

本节是此项目所需组件的完整列表

+ Raspberry Pi Camera Module V2(是的，这款相机适用于Jetson Nano)
+ Jetson Nano B01亚克力外壳，配有专用散热风扇（可选）

![image-20221102163600064](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102163600064.png)

# 3.把相机连接到Jetson Nano上

确保Jetson Nano完全关闭，没有电源连接。

![image-20221102163703295](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102163703295.png)

提起CSI连接器的塑料卡扣(Camera0)。将带状电缆完全滑入连接器，使其不倾斜。蓝色标记应该面向板的外部，远离散热器。带状电缆的触点需要朝向散热器。

# 4.测试相机

打开你的Jetson Nano。

打开一个新的终端窗口，输入:

```shell
ls /dev/video0
```

如果你看到这样的输出，这意味着你的相机是连接的。

![image-20221102164127571](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102164127571.png)

# 5.拍照

现在打开一个新的终端窗口，并将其移动到桌面边缘。

键入以下命令。你可以改变方向的值(0-3)，以获得正确的相机方向。

```shell
nvgstcapture-1.0 --orientation=2
```

按CTRL + C退出。

# 6.测试Python安装

让我们看看能否运行一个使用相机的Python脚本。

打开一个新的终端窗口，克隆由JetsonHacksNano创建的存储库。

```shell
git clone https://github.com/JetsonHacksNano/CSI-Camera.git
```

切换到新目录。

```
cd CSI-Camera
```

输入以下命令，这是一个命令:

```
gst-launch-1.0 nvarguscamerasrc sensor_id=0 ! 'video/x-raw(memory:NVMM),width=3280, height=2464, framerate=21/1, format=NV12' ! nvvidconv flip-method=2 ! 'video/x-raw, width=816, height=616' ! nvvidconv ! nvegltransform ! nveglglessink -e
```

按CTRL + C关闭它。

安装NumPy

```shell
sudo apt-get update

sudo apt install python3-numpy
```

安装libcanberra。

```shell
sudo apt install libcanberra-gtk-module
```

运行人脸检测python脚本。

```
python3 face_detect.py
```

如果输出结果不像您希望的那样，打开face_detect.py脚本。

```shell
gedit face_detect.py
```

注意:如果你没有安装gedit，输入:

```shell
sudo apt-get install gedit
```

编辑flip-method参数:

```python
def gstreamer_pipeline(
    …
    framerate=21,
    flip_method=2,
```

下面是`flip_method`参数的选项:

默认值:0,“none”

+ (0):none-无旋转

+ (1):counterclockwise -逆时针旋转90度

+ (2):rotate-180 -旋转180度

+ (3):clockwise-顺时针旋转90度

+ (4): horizontal-flip -水平翻转

+ (5): upper-right-diagonal -翻转右上/左下对角线

+ (6):vertical-flip-垂直翻转

+ (7):upper-left-diagonal-翻转左上/下

下面是face_detect.py的完整代码

```python
# MIT License
# Copyright (c) 2019 JetsonHacks
# See LICENSE for OpenCV license and additional information
 
# https://docs.opencv.org/3.3.1/d7/d8b/tutorial_py_face_detection.html
# On the Jetson Nano, OpenCV comes preinstalled
# Data files are in /usr/sharc/OpenCV
import numpy as np
import cv2
 
# gstreamer_pipeline returns a GStreamer pipeline for capturing from the CSI camera
# Defaults to 1280x720 @ 30fps
# Flip the image by setting the flip_method (most common values: 0 and 2)
# display_width and display_height determine the size of the window on the screen
 
 
def gstreamer_pipeline(
    capture_width=3280,
    capture_height=2464,
    display_width=820,
    display_height=616,
    framerate=21,
    flip_method=2,
):
    return (
        "nvarguscamerasrc ! "
        "video/x-raw(memory:NVMM), "
        "width=(int)%d, height=(int)%d, "
        "format=(string)NV12, framerate=(fraction)%d/1 ! "
        "nvvidconv flip-method=%d ! "
        "video/x-raw, width=(int)%d, height=(int)%d, format=(string)BGRx ! "
        "videoconvert ! "
        "video/x-raw, format=(string)BGR ! appsink"
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
    face_cascade = cv2.CascadeClassifier(
        "/usr/share/opencv4/haarcascades/haarcascade_frontalface_default.xml"
    )
    eye_cascade = cv2.CascadeClassifier(
        "/usr/share/opencv4/haarcascades/haarcascade_eye.xml"
    )
    cap = cv2.VideoCapture(gstreamer_pipeline(), cv2.CAP_GSTREAMER)
    if cap.isOpened():
        cv2.namedWindow("Face Detect", cv2.WINDOW_AUTOSIZE)
        while cv2.getWindowProperty("Face Detect", 0) >= 0:
            ret, img = cap.read()
            gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
            faces = face_cascade.detectMultiScale(gray, 1.3, 5)
 
            for (x, y, w, h) in faces:
                cv2.rectangle(img, (x, y), (x + w, y + h), (255, 0, 0), 2)
                roi_gray = gray[y : y + h, x : x + w]
                roi_color = img[y : y + h, x : x + w]
                eyes = eye_cascade.detectMultiScale(roi_gray)
                for (ex, ey, ew, eh) in eyes:
                    cv2.rectangle(
                        roi_color, (ex, ey), (ex + ew, ey + eh), (0, 255, 0), 2
                    )
 
            cv2.imshow("Face Detect", img)
            keyCode = cv2.waitKey(30) & 0xFF
            # Stop the program on the ESC key
            if keyCode == 27:
                break
 
        cap.release()
        cv2.destroyAllWindows()
    else:
        print("Unable to open camera")
 
 
if __name__ == "__main__":
    face_detect()
```

按CTRL+C停止脚本。

# 7.测试C++安装

打开simple_camera.cpp，如果需要的话编辑main函数中的flip_method。

编辑simple_camera.cpp

```c++
int main()
{
…
    int framerate = 60 ;
    int flip_method = 2;
```

保存文件并关闭它。

编译simple_camera.cpp。您可以复制并粘贴此命令。

```shell
g++ -std=c++11 -Wall -I/usr/include/opencv4 simple_camera.cpp -L/usr/lib/aarch64-linux-gnu -lopencv_core -lopencv_highgui -lopencv_videoio -o simple_camera
```

运行simple_camera.cpp

```shell
./simple_camera
```

按ESC退出。

要关闭计算机，输入:

```shell
sudo shutdown -h now
```

以下是simple_camera.cpp的完整代码:

```
// simple_camera.cpp
// MIT License
// Copyright (c) 2019 JetsonHacks
// See LICENSE for OpenCV license and additional information
// Using a CSI camera (such as the Raspberry Pi Version 2) connected to a 
// NVIDIA Jetson Nano Developer Kit using OpenCV
// Drivers for the camera and OpenCV are included in the base image
 
// #include <iostream>
#include <opencv2/opencv.hpp>
// #include <opencv2/videoio.hpp>
// #include <opencv2/highgui.hpp>
 
std::string gstreamer_pipeline (int capture_width, int capture_height, int display_width, int display_height, int framerate, int flip_method) {
    return "nvarguscamerasrc ! video/x-raw(memory:NVMM), width=(int)" + std::to_string(capture_width) + ", height=(int)" +
           std::to_string(capture_height) + ", format=(string)NV12, framerate=(fraction)" + std::to_string(framerate) +
           "/1 ! nvvidconv flip-method=" + std::to_string(flip_method) + " ! video/x-raw, width=(int)" + std::to_string(display_width) + ", height=(int)" +
           std::to_string(display_height) + ", format=(string)BGRx ! videoconvert ! video/x-raw, format=(string)BGR ! appsink";
}
 
int main()
{
    int capture_width = 1280 ;
    int capture_height = 720 ;
    int display_width = 1280 ;
    int display_height = 720 ;
    int framerate = 60 ;
    int flip_method = 2 ;
 
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
    int keycode = cv::waitKey(30) & 0xff ; 
        if (keycode == 27) break ;
    }
 
    cap.release();
    cv::destroyAllWindows() ;
    return 0;
}
```

# 参考目录

[https://automaticaddison.com/how-to-set-up-a-camera-for-nvidia-jetson-nano/](https://automaticaddison.com/how-to-set-up-a-camera-for-nvidia-jetson-nano/)