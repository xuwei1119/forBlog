#  calHist()-使用OpenCV和C++计算直方图

在计算机视觉中，几乎处处都使用直方图。对于阈值计算，我们使用灰度直方图。对于白平衡，我们使用直方图。对于图片中的对象跟踪，比如CamShift技术，我们使用颜色直方图，采用颜色直方图作为特征。

在更抽象的意义上，从梯度直方图形成 HOG 和 SIFT 描述符。

直方图也是一种视觉词袋表示，广泛用于图像搜索引擎和机器学习中。而且，这很可能不是您第一次在研究中看到直方图。

那么，为什么直方图会派上用场呢？

因为直方图描绘了一组数据频率分布。事实证明，查看这些频率分布是开发简单图像处理技术的主要方法......以及真正强大的机器学习算法。

这篇博文将总结图像直方图，以及如何使用 OpenCV 和 C++ 从视频中计算颜色直方图。

# 1. 什么是直方图

可以将直方图视为显示图像强度分布的图形。X 轴为像素值（通常范围为 0 到 255），Y 轴为图片中的像素数。

这只是查看图像的不同方式。当您查看图像的直方图时，您可能会感觉到图像的对比度、亮度、强度分布等。

今天几乎所有的图像处理软件都包含直方图功能。

![image-20221026111011917](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221026111011917.png)

# 2. OpenCV  C++实现

## 2.1 OpenCV 中的 calHist() 函数

```c++
cv.calcHist(images, channels, mask, histSize, ranges[, hist[, accumulate]])
```

使用**calHist**函数来实现直方图，参数解析：

    1. images:这是`uint8`或`float32`源图像。
    1. channels:它是计算直方图的通道索引。如果输入是灰度图像，则值为[0]。要计算彩色图像中蓝色、绿色或红色通道的直方图，请传递[0] 、[1]或[2] 。
    1. mask:计算直方图的区域，None表示整幅图像区域
    1. histSize:Bin数目，必须用方括号括起来。传递[256]表示全像素范围
    1. range:通常是[0,256];

## 2.2  代码

```c++
#include "opencv2/highgui.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/imgproc.hpp"
#include <iostream>


const int histSize = 256;

void drawHistogram(cv::Mat& b_hist,cv::Mat& g_hist,cv::Mat& r_hist)
{
    int hist_w = 512;
    int hist_h = 400;
    int bin_w = cvRound((double)hist_w / histSize);

    cv::Mat histImage(hist_h, hist_w, CV_8UC3, cv::Scalar(0, 0, 0));

    cv::normalize(b_hist, b_hist, 0, histImage.rows, cv::NORM_MINMAX, -1, cv::Mat());
    cv::normalize(g_hist, g_hist, 0, histImage.rows, cv::NORM_MINMAX, -1, cv::Mat());
    cv::normalize(r_hist, r_hist, 0, histImage.rows, cv::NORM_MINMAX, -1, cv::Mat());

    for (int i = 1; i < histSize; i++) 
    {
      cv::line(
          histImage,
          cv::Point(bin_w * (i - 1), hist_h - cvRound(b_hist.at<float>(i - 1))),
          cv::Point(bin_w * (i), hist_h - cvRound(b_hist.at<float>(i))),
          cv::Scalar(255, 0, 0), 2, 8, 0);
      cv::line(
          histImage,
          cv::Point(bin_w * (i - 1), hist_h - cvRound(g_hist.at<float>(i - 1))),
          cv::Point(bin_w * (i), hist_h - cvRound(g_hist.at<float>(i))),
          cv::Scalar(0, 255, 0), 2, 8, 0);
      cv::line(
          histImage,
          cv::Point(bin_w * (i - 1), hist_h - cvRound(r_hist.at<float>(i - 1))),
          cv::Point(bin_w * (i), hist_h - cvRound(r_hist.at<float>(i))),
          cv::Scalar(0, 0, 255), 2, 8, 0);
    }

    cv::namedWindow("calcHist Demo", cv::WINDOW_AUTOSIZE);
    cv::imshow("calcHist Demo", histImage);

}
int main(int argc, char **argv) 
{
  cv::Mat src, dst;

  cv::VideoCapture cap;
  if (argc != 2)
    cap.open(0);
  else
    cap.open(argv[1]);

  if (!cap.isOpened()) {
    std::cerr << "Failed to load webcam/Video ...\n";
    return -1;
  }

  for (;;) 
  {
    if(!cap.read(src)) 
    {
      std::cerr << "Cannot read file\n";
      break;
    }
    cv::imshow("Src", src);
    std::vector<cv::Mat> bgr_planes;
    cv::split(src, bgr_planes);


    float range[] = {0, 256};
    const float *histRange = {range};

    bool uniform = true;
    bool accumulate = false;

    cv::Mat b_hist, g_hist, r_hist;

    cv::calcHist(&bgr_planes[0], 1, 0, cv::Mat(), b_hist, 1, &histSize,
                 &histRange, uniform, accumulate);
    cv::calcHist(&bgr_planes[1], 1, 0, cv::Mat(), g_hist, 1, &histSize,
                 &histRange, uniform, accumulate);
    cv::calcHist(&bgr_planes[2], 1, 0, cv::Mat(), r_hist, 1, &histSize,
                 &histRange, uniform, accumulate);

    drawHistogram(b_hist,g_hist,r_hist);

    if (cv::waitKey(30) == 27)
      break;
  }

  return 0;
}
```

## 2.3 代码解析

+ 首先读取我们的输入文件，使用`cap.read()`方法逐帧读取视频。
+ 使用[split()](https://docs.opencv.org/3.4/d2/de8/group__core__array.html#ga0547c7fed86152d7e9d0096029c8518a)函数，将多通道数组（即 BGR）划分为单独的单通道数组，并将其存储在`bgr_planes`.
+ 然后我们计算每个通道的直方图并将值存储在变量`b_hist`, `g_hist`, `r_hist`中。
+ 在直方图中，我们希望我们的Bin具有相同的大小，并且我们希望在开始时清除我们的直方图，因此，我们将`uniform`和`accumulate`设置为`true`。
+ 计算直方图后，我们创建一个图像`histImage`来显示我们的直方图。
+ 然后我们在每个通道的每个像素处使用[cv::line](https://docs.opencv.org/3.4/d6/d6e/group__imgproc__draw.html#ga7078a9fae8c7e7d13d24dac2520ae4a2)绘制线，即`b_hist`, `g_hist`, `r_hist`。\

## 2.4 输出

![image-20221026131646137](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221026131646137.png)

# 3. OpenCV Python实现

## 3.1 灰度直方图

`calHist`函数接收以下参数

cv2.calcHist([img], channels, mask, bins, ranges)

+ 图像列表
+ 通道列表
+ mask
+ bins数目
+ ranges，例如[0, 255]

![image-20221026141931097](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221026141931097.png)

```python
from matplotlib import pyplot as plt
import cv2 as cv

img = cv.imread('lego.png')
gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)

hist = cv.calcHist([gray], [0], None, [256], [0, 256])

plt.figure()
plt.title('Grayscale histogram')
plt.xlabel('Bins')
plt.ylabel('# of pixels')
plt.plot(hist)
plt.xlim([0, 256])
plt.ylim([0, 2000])
plt.show()

cv.waitKey(0)
```

## 3.2 彩色直方图

![image-20221026142030846](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221026142030846.png)



![image-20221026142044674](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221026142044674.png)



```python
# Color histogram
from matplotlib import pyplot as plt
import cv2 as cv

img = cv.imread('lego.png')
chans = cv.split(img)
colors = 'b', 'g', 'r'

plt.figure()
plt.title('Flattened color histogram')
plt.xlabel('Bins')
plt.ylabel('# of pixels')

for (chan, color) in zip(chans, colors):
    hist = cv.calcHist([chan], [0], None, [256], [0, 255])
    plt.plot(hist, color=color)
    plt.xlim([0, 256])
    plt.ylim([0, 1200])

plt.show()
cv.waitKey(0)
```



## 3.3 模糊案例

```python
# Blurring
import cv2 as cv

def trackbar(x):
    x = cv.getTrackbarPos('blur x','window')
    y = cv.getTrackbarPos('blur x','window')
    blurred = cv.blur(img, (x, y))
    cv.imshow('window', blurred)
    cv.displayOverlay('window', f'blur = ({x}, {y})')

img = cv.imread('lego.png')
cv.imshow('window', img)
cv.createTrackbar('blur x', 'window', 0, 4, trackbar)
cv.createTrackbar('blur y', 'window', 0, 4, trackbar)

cv.waitKey(0)
cv.destroyAllWindows()
```



# 参考目录

[https://anothertechs-com.translate.goog/programming/cpp/opencv/calculate-histogram/?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN#google_vignette](https://anothertechs-com.translate.goog/programming/cpp/opencv/calculate-histogram/?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN#google_vignette)

