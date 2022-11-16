在这个文章中，我们将学习如何从基于OpenCV和imageio实现将视频转为GIF。

# 1.步骤

+ 使用`cv2.VideoCapture`打开视频文件
+ 使用`cap.read()`方法一帧一帧读取视频帧
+ 将BGR转为RGB。`imageio`接受RGB格式的图像
+ 将帧保存到列表中，并关闭视频文件
+ 使用`imageio.mimsave()`方法将帧列表转换为GIF。根据需要设置FPS

代码如下：

```python
import cv2

import imageio

cap = cv2.VideoCapture('D:/downloads/video.mp4')
image_lst = []

while True:
    ret, frame = cap.read()
    frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    image_lst.append(frame_rgb)
    
    cv2.imshow('a', frame)
    key = cv2.waitKey(1)
    if key == ord('q'):
        break
        
cap.release()
cv2.destroyAllWindows()

# Convert to gif using the imageio.mimsave method
imageio.mimsave('D:/downloads/video.gif', image_lst, fps=60)
```

接下来让我们看看如何定制化视频部分转为GIF。

# 2. 定制化视频部分转为GIF

## 2.1 方法一

使用视频的FPS，我们可以很容易的计算出开始与结束帧。然后提取之间的所有帧。一旦需要的帧提取完，我们就可以将它们转为GIF。以下显示了提取20秒到25秒之间的帧并将其转为GIF。

```python
import cv2

import imageio

cap = cv2.VideoCapture('D:/downloads/video.mp4')
fps = cap.get(cv2.CAP_PROP_FPS)
start_time = 20*fps
end_time = 25*fps
image_lst = []
i = 0

while True:
    ret, frame = cap.read()
    if ret == False:
        break
    if (i>=start_time and i<=end_time):
        frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        image_lst.append(frame_rgb)

        cv2.imshow('a', frame)
        key = cv2.waitKey(1)
        if key == ord('q'):
            break
    i +=1

cap.release()
cv2.destroyAllWindows()

# Convert to gif using the imageio.mimsave method
imageio.mimsave('D:/downloads/video.gif', image_lst, fps=60)
```

## 2.2 方法二

也可以根据控件手动的保存需要的帧图像。例如，当按`s`时候保存图像到列表中，当按`q`时候退出。

```python
import cv2

import imageio

cap = cv2.VideoCapture('D:/downloads/video.mp4')
image_lst = []

prev_key = -1
while True:
    ret, frame = cap.read()
    cv2.imshow('a', frame)
    key = cv2.waitKey(1)
    
    if key == ord('s'):
        key = -1
        prev_key = ord('s')
    
    if key == -1 and prev_key == ord('s'):
        frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        image_lst.append(frame_rgb)
    
    if key == ord('q'):
        break
        
cap.release()
cv2.destroyAllWindows()

# Convert to gif using the imageio.mimsave method
imageio.mimsave('D:/downloads/video.gif', image_lst, fps=60)
```

## 2.2 方法三

这种方法相对来说比较繁琐。在这里，你一帧一帧地浏览每一帧，如果你想包括在GIF中你按下`a`键。要退出，按`q`键。一旦特定的帧被提取出来，我们可以很容易地使用`imageio`将它们转换成上面讨论过的动图。下面是它的代码。

```python
import cv2
import imageio

cap = cv2.VideoCapture('D:/downloads/video.mp4')
image_lst = []

while True:
    ret, frame = cap.read()
    cv2.imshow('a', frame)
    key = cv2.waitKey(0)

    
    if key == ord('s'):
        frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        image_lst.append(frame_rgb)
    
    if key == ord('q'):
        break
        
cap.release()
cv2.destroyAllWindows()

# Convert to gif using the imageio.mimsave method
imageio.mimsave('D:/downloads/video.gif', image_lst, fps=60)
```

# 参考目录

[https://theailearner.com/2021/05/29/creating-gif-from-video-using-opencv-and-imageio/](https://theailearner.com/2021/05/29/creating-gif-from-video-using-opencv-and-imageio/)