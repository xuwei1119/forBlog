# 在树莓派4或Jetson Nano上安装带有OpenCV的Qt5

# 1.简介

本文将帮助您在Raspberry Pi 4或Jetson Nano上安装Qt5。安装之后，我们将构建一个带有OpenCV接口的GUI。最后，你将拥有一个原始的Raspicam或Tegra UI风格的实时Raspicam或网络摄像头界面。

Qt5是一个免费的、开源的、跨平台的软件，特别适合设计图形用户界面。一些著名的用Qt5编写的GUI应用程序是Google-Earth和VLC媒体播放器。当然，您总是可以将Qt5用于您的简单终端程序。Qt5的一个很好的特性是它出色的跨平台可移植性。一旦你在Linux机器上编写了应用程序，将同样的应用程序移植到苹果或windows机器上就意味着几乎不需要额外的代码。

Qt5不是像Geany或类似的Code::Blocks那样简单的编程工具。在Qt5中开始编码时，您将面临陡峭的学习曲线。即使是一个简单的GUI程序，也必须理解许多概念。当你脑子里只有一些简单的想法时，问问你自己，这是否值得花费所有的时间和挫折。另一方面，一旦征服了Qt5，你就可以称自己为真正的专业人士。而且，网上有很多友好的Qt5论坛和教程。

本指南不会解释潜在的Qt5机制。这不是一个Qt5教程。这里的主要目标是Qt5的初始安装以及OpenCV和Qt5“世界”之间的连接。Qt5有它自己的对象，比如字符串和位图。它们与标准库和OpenCV实现略有不同。顺便说一下，这是使用Qt5时必须克服的一个众所周知的障碍。

在过去，嵌入式系统几乎没有RAM，时钟速度最高可达10 MHz，因此必须进行交叉编译。首先，软件是在桌面PC上开发的，然后转移到嵌入式系统。这是一种编写软件的繁琐方式，调试的可能性有限。当专门的I/O必须被处理时，它可能会以真正的噩梦告终。

今天，四核ARM的运行频率为1.5 GHz或更高，RAM为2,4甚至8GByte，幸运的是，不再需要交叉编译了。只需将整个Qt5平台安装到树莓派4或Jetson Nano上，然后立即开始编写代码。

# 2.安装

安装很简单。不需要首先安装依赖项。如果您想再现给定的示例，您需要在您的板上安装OpenCV。

```shell

# install Qt5
$ sudo apt-get update
$ sudo apt-get upgrade
Buster OS
$ sudo apt-get install qt5-default
$ sudo apt-get install qtcreator
$ sudo apt-get install qtdeclarative5-dev
or Bullseye OS
$ sudo apt-get install qtbase5-dev qtchooser
$ sudo apt-get install qt5-qmake qtbase5-dev-tools
$ sudo apt-get install qtcreator
$ sudo apt-get install qtdeclarative5-dev
```

Raspberry Pi Buster用户必须修改GTK-toolkit版本。如果你有一台Jetson Nano，你可以跳过这一步。

用nano编辑器打开文件`/etc/xdg/qt5ct/qt5ct.conf`，并设置样式为`gtk3`。保存并退出<Ctrl> + <X>， <Y>和<Enter>。

```shell
# edit qt5ct.conf on a Raspberry Pi Buster.
$ sudo nano /etc/xdg/qt5ct/qt5ct.conf
```

![image-20221102143045777](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102143045777.png)



如果您忘记设置GTK工具包，您可能会遇到以下警告和错误。

> \- cannot register existing type 'GtkWidget'
>
> \- g_type_add_interface_static: assertion 'G_TYPE_IS_INSTANTIATABLE (instance_type)' failed
>
> \- g_type_register_static: assertion 'parent_type > 0' failed

如果一切顺利，在启动Qt5创建器时，应该会看到下面的欢迎屏幕。这次是在Jetson Nano上。

![image-20221102143145299](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102143145299.png)

# 3.OpenCV案例

让我们从OpenCV中一个简单的网络摄像头预览开始。之前我们已经开发了一个只使用OpenCV和网络摄像头的应用程序。然而，它是一个终端应用程序，带有一个显示视频图像的OpenCV窗口。请在[这里](https://qengineering.eu/opencv-c-examples-on-raspberry-pi.html)找到教程。现在我们使用真正的GUI来显示帧。这一切都是在Qt5环境中完成的。

第一步是设计适合显示图像的GUI。其次，OpenCV视频记录必须在一个单独的线程中启动，以处理来自网络摄像头的帧。第三，实现一个图像显示，将OpenCV cv::mat转换为QImage，即Qt5图像格式。最后一个操作是在GUI的“画布”上绘制图像。

让我们从从[GitHub下载](https://github.com/Qengineering/Qt5-OpenCV-Raspberry-Pi-Jetson-Nano)项目开始。假设您已经将项目解压到`~/software/Qt5`文件夹中，您将得到以下内容。

![image-20221102143449607](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102143449607.png)

现在打开Qt5创建器。您将看到下一个欢迎屏幕。

![image-20221102143521506](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102143521506.png)

接下来，单击`Open Project`按钮并加载示例项目文件`Viewer.pro`进入IDE。您将看到下一个屏幕，要求您通过选择一个所谓的套件来配置项目。Qt5与“套件”一起工作，套件是定义环境的参数集合，如设备、编译器、桌面等。它使跨平台开发变得容易。在我们的例子中，我们只有一个套件，树莓派桌面。只需单击Configure Project按钮，就可以开始工作了。

![image-20221102143707725](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102143707725.png)

Qt5现在在IDE环境中加载项目。我们展开了所有文件夹并双击Viewer.pro文件以在编辑器中打开它。这里最重要的是红框中的线条。这将包括项目中的OpenCV头文件和库。如果您需要在Qt5项目中使用OpenCV，请始终添加这些行。当然，文件的位置将取决于您的OpenCV安装。这里我们使用的是OpenCV 4.5安装中的目录。

![image-20221102143843433](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102143843433.png)

您可能想查看一下我们为这个应用程序创建的GUI。双击`mainwindow.ui `文件，GUI将打开与设计。在屏幕中，将选择绘制视频的QLabel。

![image-20221102144138355](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102144138355.png)

不需要过多的细节，OpenCV到Qt5位图的转换在myvidecapture .cpp文件的最后一部分，cvMatToQImage和cvMatToQPixmap例程中完成。QPixmap可以在QLabel画布上绘制。

该文件的第一部分创建了一个单独的线程，用于从WebCam捕获视频流。如您所知，GUI应用程序与事件一起工作。事件可以是鼠标点击、移动滚动条、以太网数据包的到达等等。每个事件调用一小段代码，描述要做什么。一旦完成，应用程序将处于空闲状态，等待下一个事件的发生。这是一个很好的机制，只要它不花费太多时间来处理事件。

例如，如果绘图需要一些耗时的图像渲染，您的应用程序将不得不等待完成这一操作。你可能会觉得应用已经崩溃了;它对你的鼠标或键盘没有反应。在这种情况下，最好创建一个单独的线程。主线程仍然可以处理常见的用户事件，而另一个线程在后台渲染图像。这个应用程序也是如此。一个线程不断地接收新的图像，并将它们转换为QPixmap在画布上绘图。另一个线程保持你的“正常”Gui的活力。

## **myvideocapture.h**

```
#ifndef MYVIDEOCAPTURE_H
#define MYVIDEOCAPTURE_H

#include <QPixmap>
#include <QImage>
#include <QThread>
#include <opencv2/opencv.hpp>

#define ID_CAMERA 0

class MyVideoCapture : public QThread
{
    Q_OBJECT
public:
    MyVideoCapture(QObject *parent = nullptr);
    QPixmap pixmap() const
    {
        return mPixmap;
    }
signals:
    void newPixmapCapture(); //capture a frame
protected:
    void run() override;
private:
    QPixmap mPixmap;              //Qt image
    cv::Mat mFrame;               //OpenCV image
    cv::VideoCapture mVideoCap;   //video capture

    QImage cvMatToQImage(const cv::Mat &inMat);
    QPixmap cvMatToQPixmap(const cv::Mat &inMat );
};

#endif // MYVIDEOCAPTURE_H
```

## **myvideocapture.cpp**

```C++
#include "myvideocapture.h"
#include <QDebug>

MyVideoCapture::MyVideoCapture(QObject *parent)
    :QThread { parent }
    ,mVideoCap { ID_CAMERA }
{
}

void MyVideoCapture::run()
{
    if(mVideoCap.isOpened())
    {
        while (true)
        {
            mVideoCap >> mFrame;
            if(!mFrame.empty())
            {
                mPixmap = cvMatToQPixmap(mFrame);
                emit newPixmapCapture();
            }
        }
    }
}

QImage MyVideoCapture::cvMatToQImage( const cv::Mat &inMat )
{
  switch ( inMat.type() )
  {
     // 8-bit, 4 channel
     case CV_8UC4:
     {
        QImage image( inMat.data,
                      inMat.cols, inMat.rows,
                      static_cast<int>(inMat.step),
                      QImage::Format_ARGB32 );

        return image;
     }

     // 8-bit, 3 channel
     case CV_8UC3:
     {
        QImage image( inMat.data,
                      inMat.cols, inMat.rows,
                      static_cast<int>(inMat.step),
                      QImage::Format_RGB888 );

        return image.rgbSwapped();
     }

     // 8-bit, 1 channel
     case CV_8UC1:
     {
#if QT_VERSION >= QT_VERSION_CHECK(5, 5, 0)
        QImage image( inMat.data,
                      inMat.cols, inMat.rows,
                      static_cast<int>(inMat.step),
                      QImage::Format_Grayscale8 );
#else
        static QVector<QRgb>  sColorTable;

        // only create our color table the first time
        if ( sColorTable.isEmpty() )
        {
           sColorTable.resize( 256 );

           for ( int i = 0; i < 256; ++i )
           {
              sColorTable[i] = qRgb( i, i, i );
           }
        }

        QImage image( inMat.data,
                      inMat.cols, inMat.rows,
                      static_cast<int>(inMat.step),
                      QImage::Format_Indexed8 );

        image.setColorTable( sColorTable );
#endif

        return image;
     }

     default:
        qWarning() << "ASM::cvMatToQImage() - cv::Mat image type not handled in switch:" << inMat.type();
        break;
  }

  return QImage();
}
QPixmap MyVideoCapture::cvMatToQPixmap( const cv::Mat &inMat )
{
  return QPixmap::fromImage( cvMatToQImage( inMat ) );
}
```

其余的代码或多或少是标准的。`mainwindow.cpp`文件包含类的构造函数和析构函数，它的父类是一个QMianWindow和我们的UI。它有一个事件处理程序，即启动OpenCV捕获线程的按钮。这个线程将无休止地运行，直到主窗口析构函数终止它。

```C++
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include "myvideocapture.h"

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    mOpenCV_videoCapture =  new MyVideoCapture(this);

    connect(mOpenCV_videoCapture, &MyVideoCapture::newPixmapCapture, this, [&]()
    {
       ui->opencvFrame->setPixmap(mOpenCV_videoCapture->pixmap().scaled(640,480));
    });
}

MainWindow::~MainWindow()
{
    delete ui;
    mOpenCV_videoCapture->terminate();
}

void MainWindow::on_InitOpenCV_button_clicked()
{
    mOpenCV_videoCapture->start(QThread::HighestPriority);
}
```

要开始GUI示例，只需使用锤子来编译代码。现在，您可以使用将以发布模式启动应用程序的绿色箭头，或者将带有错误的绿色箭头使用调试模式。如果您想了解更多信息，我们建议您访问互联网，在那里您可以找到关于Qt5的教程和论坛。

![image-20221102144848933](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102144848933.png)

最后结果:

![image-20221102144922790](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102144922790.png)

#  参考目录

[https://qengineering.eu/install-qt5-with-opencv-on-raspberry-pi-4.html](https://qengineering.eu/install-qt5-with-opencv-on-raspberry-pi-4.html)