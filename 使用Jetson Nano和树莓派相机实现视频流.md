# 使用Jetson Nano和树莓派相机实现视频流

了解如何用树莓派相机和Jetson Nano使用Flask框架实时视频流!

Jetson Nano是一个边缘计算平台，适合低功耗、不受监控和独立使用。它非常适合在没有显示器或键盘等外设连接的情况下使用。

本文展示了如何从树莓派(Raspberry Pi)摄像机实时传输视频到web浏览器，并在连接到同一网络的任何其他设备上访问该视频流。

# 1.先决条件

+ Jetson Nano开发工具包
+ Raspberry Pi Camera v2相机
+ 开发者套件的电源

一切都设置好之后，我们创建一个简单的Python应用程序，它使用OpenCV从摄像机捕获视频，调整每一帧的大小，并使用Python Flask框架将其传输到HTML网页。

# 2. Flask

Python有几个用于创建可靠和高性能web应用程序的web开发框架。Django、Flask和Pyramid是最常见的框架。

Flask是一个微框架，不需要外部库或工具。它比Django更灵活，并且允许用户包含不同的插件和或扩展。它的数据抽象层或数据验证层可以在其他框架中找到。用户可以为身份验证、数据格式验证、SQL管理、请求处理和用户权限部署第三方扩展。

在Jetson Nano上安装Falsk

```shell
pip3 install flask
```

# 3.简单的视频流到Web浏览器应用程序

Jetson Nano板使用GStreamer管道来处理媒体应用程序。GStreamer管道利用appsink接收器插件来访问原始缓冲区数据。

![image-20221102171316073](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102171316073.png)

当用户发送kill命令(Ctrl + C)时，它会在退出前等待所有数据缓冲区处理完毕，这将阻塞应用程序中的流线程。因此，这里创建的Gstreamer对象额外被赋予了`wait-on-eos=false`和只有一个buffer (`max-buffers=true`)参数，以避免脏杀和挂起管道。

```python
import cv2
import time
import threading
from flask import Response, Flask

# Image frame sent to the Flask object
global video_frame
video_frame = None

# Use locks for thread-safe viewing of frames in multiple browsers
global thread_lock 
thread_lock = threading.Lock()

# GStreamer Pipeline to access the Raspberry Pi camera
GSTREAMER_PIPELINE = 'nvarguscamerasrc ! video/x-raw(memory:NVMM), width=3280, height=2464, format=(string)NV12, framerate=21/1 ! nvvidconv flip-method=0 ! video/x-raw, width=960, height=616, format=(string)BGRx ! videoconvert ! video/x-raw, format=(string)BGR ! appsink wait-on-eos=false max-buffers=1 drop=True'

# Create the Flask object for the application
app = Flask(__name__)

def captureFrames():
    global video_frame, thread_lock

    # Video capturing from OpenCV
    video_capture = cv2.VideoCapture(GSTREAMER_PIPELINE, cv2.CAP_GSTREAMER)

    while True and video_capture.isOpened():
        return_key, frame = video_capture.read()
        if not return_key:
            break

        # Create a copy of the frame and store it in the global variable,
        # with thread safe access
        with thread_lock:
            video_frame = frame.copy()
        
        key = cv2.waitKey(30) & 0xff
        if key == 27:
            break

    video_capture.release()
        
def encodeFrame():
    global thread_lock
    while True:
        # Acquire thread_lock to access the global video_frame object
        with thread_lock:
            global video_frame
            if video_frame is None:
                continue
            return_key, encoded_image = cv2.imencode(".jpg", video_frame)
            if not return_key:
                continue

        # Output image as a byte array
        yield(b'--frame\r\n' b'Content-Type: image/jpeg\r\n\r\n' + 
            bytearray(encoded_image) + b'\r\n')

@app.route("/")
def streamFrames():
    return Response(encodeFrame(), mimetype = "multipart/x-mixed-replace; boundary=frame")

# check to see if this is the main thread of execution
if __name__ == '__main__':

    # Create a thread and attach the method that captures the image frames, to it
    process_thread = threading.Thread(target=captureFrames)
    process_thread.daemon = True

    # Start the thread
    process_thread.start()

    # start the Flask Web Application
    # While it can be run on any feasible IP, IP = 0.0.0.0 renders the web app on
    # the host machine's localhost and is discoverable by other machines on the same network 
    app.run("0.0.0.0", port="8000")
```

代码的重要部分带有注释。该应用程序是用线程安全的视频帧捕获和显示实现编写的，以避免数据损坏。

+ 将上面显示的程序保存为 web_streaming.py 并通过python web_streaming.py 运行。这将启动 Flask 应用程序并启动视频流。
+ 运行 ifconfig 获取 Jetson Nano Developer Kit 的 IP 地址。

您可以通过访问 IP 地址在连接到同一网络的任何设备的浏览器窗口中访问视频源。例如输入 192.168.1.2:8000 就可以看到直播了。

# 4.结果和结论

当 Jetson Nano 板部署在机器人或监控站点等远程操作平台上时，实时查看摄像头操作至关重要。这是一种非常简单的技术，可以通过网络传输摄像头实时视频流并在其他机器上查看。

# 参考目录

[https://maker-pro.translate.goog/nvidia-jetson/tutorial/streaming-real-time-video-from-rpi-camera-to-browser-on-jetson-nano-with-flask](https://maker-pro.translate.goog/nvidia-jetson/tutorial/streaming-real-time-video-from-rpi-camera-to-browser-on-jetson-nano-with-flask)