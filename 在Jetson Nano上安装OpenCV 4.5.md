# 在Jetson Nano上安装OpenCV 4.5



# 1.简介

你的操作系统已经预装了一个版本，为什么还要在Jetson Nano上安装OpenCV呢?主要原因是附带的版本没有CUDA支持。毕竟，CUDA加速器不正是你当初购买Jetson Nano的主要原因吗?

通常，如果我们使用预先安装OpenCV, TensorFlow和PyTorch的Jetson Nano的SD镜像，我们会遇到问题。

我们很乐意满足这一要求。请在我们的[GitHub](https://github.com/Qengineering/Jetson-Nano-image)页面上找到完整的Jetson Nano镜像。从我们的Gdrive网站下载zip文件，解压在32GB SD卡上!

**小贴士**

在初始安装过程中，建议安装lightdm桌面而不是gdm3，这样可以节省大约500 MByte的宝贵内存空间。它可以使一切变得不同，特别是对于需要内存的深度学习模型。虽然你们中的一些人可能更喜欢Ubuntu的外观和感觉，而不是有点过时的lxdm体验。

![gdm2 desktop](https://qengineering.eu/images/gdm3.webp)![lightdm desktop](https://qengineering.eu/images/lightdm.webp)

在Jetson Nano上安装OpenCV 4.5.0之前，请考虑超频。当CUDA加速器在大多数日常应用中不使用时，Jetson Nano有一个四核ARM Cortex-A57内核，运行频率为1.4 GHz。与树莓派4的Cortex-A72 1.5 GHz频段相比，差别并不大。特别是当树莓派4超频时。在这种情况下，它将在大多数任务中击败Jetson。当然，CUDA生效的时候除外。

所以，是时候在支持CUDA和cuDNN的Jetson Nano上重新安装OpenCV了。

![SimpleOpenCV](https://qengineering.eu/images/SimpleOpenCV.webp)![FullOpenCV](https://qengineering.eu/images/FullOpenCV.webp)

# 2. 安装支持CUDA和cuDNN的OpenCV

## 2.1 扩大内存交换

构建完整的OpenCV 4.5包需要超过4Gb的RAM和通常可以在Jetson Nano上找到的由zram提供的2Gb交换空间。我们必须安装**dphysi -swapfile**来临时从SD卡获得额外的空间。编译完成后，该机制将被删除，从而消除对SD卡的交换。请执行下面的命令。还要注意nano的安装，这是一个微型文本编辑器。

## 2.2 OpenCV 4.5.2, 4.5.3, 4.5.4和4.5.5

从4.5.2版本开始，OpenCV甚至比以前的版本需要更多的内存。在这种情况下，您需要增加dphys交换超过常规的2048限制。首先将**/sbin/dphys-swapfile**中的最大边界更改为4096。接下来，设置**/etc/dphys-swapfile**。幻灯片会引导你。如果没有足够的交换内存，编译到100%时将在没有通知的情况下崩溃，特别是在同时使用所有4个内核时。

我们不建议增加zram交换限额。您不能为了获得一些额外的空间而一直压缩系统内存。最好是临时使用SD内存。安装OpenCV之后，您可以再次删除**dphysics -swapfile**。

```shell
# a fresh start, so check for updates
$ sudo apt-get update
$ sudo apt-get upgrade
# install nano
$ sudo apt-get install nano
# install dphys-swapfile
$ sudo apt-get install dphys-swapfile
# enlarge the boundary (4.5.2, 4.5.3, 4.5.4 and 4.5.5)
$ sudo nano /sbin/dphys-swapfile
# give the required memory size
$ sudo nano /etc/dphys-swapfile
# reboot afterwards
$ sudo reboot
```

![SwapMaxJetson.png](https://qengineering.eu/my_gallery/SwapMaxJetson.webp)

![Free -m](https://qengineering.eu/images/Free-m2.webp)

## 2.3 脚本安装

在你的Jetson Nano上安装OpenCV并不复杂。它有68个命令行，更多的是一项管理任务。这就是为什么我们创建了一个安装脚本，它一次执行本指南中的所有命令。如果你想使用它，它不会引起任何问题。整个安装需要两个小时才能完成。它从安装依赖项开始，到**ldconfig**结束。

如果您想用Qt5 GUI美化您的OpenCV，请遵循[GitHub](https://github.com/Qengineering/Install-OpenCV-Jetson-Nano)页面上的说明，或按照下面来做。

### 2.3.1 OpenCV 4.6.0

```shell
# check your memory first
$ free -m
# you need at least a total of 8.5 GB!
# if not, enlarge your swap space as explained in the guide
$ wget https://github.com/Qengineering/Install-OpenCV-Jetson-Nano/raw/main/OpenCV-4-6-0.sh
$ sudo chmod 755 ./OpenCV-4-6-0.sh
$ ./OpenCV-4-6-0.sh
# once the installation is done...
$ rm OpenCV-4-6-0.sh
# remove the dphys-swapfile now
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
# just a tip to save an additional 275 MB
$ sudo rm -rf ~/opencv
$ sudo rm -rf ~/opencv_contrib
```

### 2.3.2 OpenCV4.5.5

```shell
# check your memory first
$ free -m
# you need at least a total of 8.5 GB!
# if not, enlarge your swap space as explained in the guide
$ wget https://github.com/Qengineering/Install-OpenCV-Jetson-Nano/raw/main/OpenCV-4-5-5.sh
$ sudo chmod 755 ./OpenCV-4-5-5.sh
$ ./OpenCV-4-5-5.sh
# once the installation is done...
$ rm OpenCV-4-5-5.sh
# remove the dphys-swapfile now
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
# just a tip to save an additional 275 MB
$ sudo rm -rf ~/opencv
$ sudo rm -rf ~/opencv_contrib
```

### 2.3.3 OpenCV4.5.4

```shell
# check your memory first
$ free -m
# you need at least a total of 8.5 GB!
# if not, enlarge your swap space as explained in the guide
$ wget https://github.com/Qengineering/Install-OpenCV-Jetson-Nano/raw/main/OpenCV-4-5-4.sh
$ sudo chmod 755 ./OpenCV-4-5-4.sh
$ ./OpenCV-4-5-4.sh
# once the installation is done...
$ rm OpenCV-4-5-4.sh
# remove the dphys-swapfile now
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
# just a tip to save an additional 275 MB
$ sudo rm -rf ~/opencv
$ sudo rm -rf ~/opencv_contrib
```

### 2.3.4 OpenCV4.5.3

```shell
# check your memory first
$ free -m
# you need at least a total of 8.5 GB!
# if not, enlarge your swap space as explained in the guide
$ wget https://github.com/Qengineering/Install-OpenCV-Jetson-Nano/raw/main/OpenCV-4-5-3.sh
$ sudo chmod 755 ./OpenCV-4-5-3.sh
$ ./OpenCV-4-5-3.sh
# once the installation is done...
$ rm OpenCV-4-5-3.sh
# remove the dphys-swapfile now
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
# just a tip to save an additional 275 MB
$ sudo rm -rf ~/opencv
$ sudo rm -rf ~/opencv_contrib
```

### 2.3.5 OpenCV4.5.2

```shell
# check your memory first
$ free -m
# you need at least a total of 8.5 GB!
# if not, enlarge your swap space as explained in the guide
$ wget https://github.com/Qengineering/Install-OpenCV-Jetson-Nano/raw/main/OpenCV-4-5-2.sh
$ sudo chmod 755 ./OpenCV-4-5-2.sh
$ ./OpenCV-4-5-2.sh
# once the installation is done...
$ rm OpenCV-4-5-2.sh
# remove the dphys-swapfile now
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
# just a tip to save an additional 275 MB
$ sudo rm -rf ~/opencv
$ sudo rm -rf ~/opencv_contrib
```

### 2.3.6 OpenCV4.5.1

```shell
# check your memory first
$ free -m
# you need at least a total of 6.5 GB!
# if not, enlarge your swap space as explained in the guide
$ wget https://github.com/Qengineering/Install-OpenCV-Jetson-Nano/raw/main/OpenCV-4-5-1.sh
$ sudo chmod 755 ./OpenCV-4-5-1.sh
$ ./OpenCV-4-5-1.sh
# once the installation is done...
$ rm OpenCV-4-5-1.sh
# remove the dphys-swapfile now
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
# just a tip to save an additional 275 MB
$ sudo rm -rf ~/opencv
$ sudo rm -rf ~/opencv_contrib
```

### 2.3.7 OpenCV4.5.0

```shell
# check your memory first
$ free -m
# you need at least a total of 6.5 GB!
# if not, enlarge your swap space as explained in the guide
$ wget https://github.com/Qengineering/Install-OpenCV-Jetson-Nano/raw/main/OpenCV-4-5-0.sh
$ sudo chmod 755 ./OpenCV-4-5-0.sh
$ ./OpenCV-4-5-0.sh
# once the installation is done...
$ rm OpenCV-4-5-0.sh
# remove the dphys-swapfile now
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
# just a tip to save an additional 275 MB
$ sudo rm -rf ~/opencv
$ sudo rm -rf ~/opencv_contrib
```

## 2.4 手动安装

不使用脚本?这里是完整的指南。

OpenCV软件使用其他第三方软件库。先安装这些。其中一些带有Jetson Tegra操作系统，另一些你可能已经安装了。万无一失总比后悔好，下面是完整的清单。按照安装程序只安装最新的软件包。

安装将占用磁盘上额外的800MB(如果尚未存在的话)。

```shell
# reveal the CUDA location
$ sudo sh -c "echo '/usr/local/cuda/lib64' >> /etc/ld.so.conf.d/nvidia-tegra.conf"
$ sudo ldconfig
# third-party libraries
$ sudo apt-get install build-essential cmake git unzip pkg-config zlib1g-dev
$ sudo apt-get install libjpeg-dev libjpeg8-dev libjpeg-turbo8-dev
$ sudo apt-get install libpng-dev libtiff-dev libglew-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev
$ sudo apt-get install libgtk2.0-dev libgtk-3-dev libcanberra-gtk*
$ sudo apt-get install python-dev python-numpy python-pip
$ sudo apt-get install python3-dev python3-numpy python3-pip
$ sudo apt-get install libxvidcore-dev libx264-dev libgtk-3-dev
$ sudo apt-get install libtbb2 libtbb-dev libdc1394-22-dev libxine2-dev
$ sudo apt-get install gstreamer1.0-tools libgstreamer-plugins-base1.0-dev
$ sudo apt-get install libgstreamer-plugins-good1.0-dev
$ sudo apt-get install libv4l-dev v4l-utils v4l2ucp qv4l2
$ sudo apt-get install libtesseract-dev libxine2-dev libpostproc-dev
$ sudo apt-get install libavresample-dev libvorbis-dev
$ sudo apt-get install libfaac-dev libmp3lame-dev libtheora-dev
$ sudo apt-get install libopencore-amrnb-dev libopencore-amrwb-dev
$ sudo apt-get install libopenblas-dev libatlas-base-dev libblas-dev
$ sudo apt-get install liblapack-dev liblapacke-dev libeigen3-dev gfortran
$ sudo apt-get install libhdf5-dev libprotobuf-dev protobuf-compiler
$ sudo apt-get install libgoogle-glog-dev libgflags-dev
```

## 2.4.1 Qt5

Qt是一个开发跨平台图形用户界面的开源工具包。它也可以在Linux Tegra操作系统上运行。该软件可用于美化OpenCV窗口和其他用户界面，如滑块和复选框。这对于OpenCV的工作来说绝对不是强制性的，只是为了美化外观。必须指出的是，使用Qt5会使OpenCV的运行速度降低几个百分点。如果你想要最快的应用，那就不要用它。

![No Qt5](https://qengineering.eu/images/openCV-window.webp)![With Qt5](https://qengineering.eu/images/openCV-Qt5-window.webp)

如果您想要在OpenCV中启用Qt5支持，您必须下载如下命令所示的库。下一步是在构建期间设置**-D WITH_QT**标志。

```shell
# only install if you want Qt5
# to beautify your OpenCV GUI
$ sudo apt-get install qt5-default
```

## 2.4.2 下载OpenCV

当所有第三方软件安装完成后，OpenCV就可以下载了。需要两个包;基本版本和附加**contribute**。下载后，您可以解压缩文件。请注意文本框中的换行。这两个命令以**wget**开始，以**zip**结束。

(1) OpenCV4.6.0

```shell
# download the latest version
$ cd ~
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.6.0.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.6.0.zip
# unpack
$ unzip opencv.zip
$ unzip opencv_contrib.zip
# some administration to make live easier later on
$ mv opencv-4.6.0 opencv
$ mv opencv_contrib-4.6.0 opencv_contrib
# clean up the zip files
$ rm opencv.zip
$ rm opencv_contrib.zip
```

(2) OpenCV4.5.5

```shell
# download the latest version
$ cd ~
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.5.5.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.5.5.zip
# unpack
$ unzip opencv.zip
$ unzip opencv_contrib.zip
# some administration to make live easier later on
$ mv opencv-4.5.5 opencv
$ mv opencv_contrib-4.5.5 opencv_contrib
# clean up the zip files
$ rm opencv.zip
$ rm opencv_contrib.zip
```

(3) OpenCV4.5.4

```shell
# download the latest version
$ cd ~
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.5.4.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.5.4.zip
# unpack
$ unzip opencv.zip
$ unzip opencv_contrib.zip
# some administration to make live easier later on
$ mv opencv-4.5.4 opencv
$ mv opencv_contrib-4.5.4 opencv_contrib
# clean up the zip files
$ rm opencv.zip
$ rm opencv_contrib.zip
```

(4) OpenCV4.5.3

```shell
# download the latest version
$ cd ~
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.5.3.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.5.3.zip
# unpack
$ unzip opencv.zip
$ unzip opencv_contrib.zip
# some administration to make live easier later on
$ mv opencv-4.5.3 opencv
$ mv opencv_contrib-4.5.3 opencv_contrib
# clean up the zip files
$ rm opencv.zip
$ rm opencv_contrib.zip
```

(5) OpenCV4.5.2

```shell
# download the latest version
$ cd ~
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.5.2.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.5.2.zip
# unpack
$ unzip opencv.zip
$ unzip opencv_contrib.zip
# some administration to make live easier later on
$ mv opencv-4.5.2 opencv
$ mv opencv_contrib-4.5.2 opencv_contrib
# clean up the zip files
$ rm opencv.zip
$ rm opencv_contrib.zip
```

(6) OpenCV4.5.1

```shell
# download the latest version
$ cd ~
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.5.1.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.5.1.zip
# unpack
$ unzip opencv.zip
$ unzip opencv_contrib.zip
# some administration to make live easier later on
$ mv opencv-4.5.1 opencv
$ mv opencv_contrib-4.5.1 opencv_contrib
# clean up the zip files
$ rm opencv.zip
$ rm opencv_contrib.zip
```

(7) OpenCV4.5.0

```shell
# download the latest version
$ cd ~
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.5.0.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.5.0.zip
# unpack
$ unzip opencv.zip
$ unzip opencv_contrib.zip
# some administration to make live easier later on
$ mv opencv-4.5.0 opencv
$ mv opencv_contrib-4.5.0 opencv_contrib
# clean up the zip files
$ rm opencv.zip
$ rm opencv_contrib.zip
```



### 2.4.3 构建make

在我们开始实际构建库之前，还有一小步要走。您必须创建一个目录，其中可以存放所有构建文件。

```shell
$ cd ~/opencv
$ mkdir build
$ cd build
```

现在是时候迈出重要一步了。涉及到很多flag。您可能会注意到**-D WITH_QT=OFF**行。这里禁用了Qt5支持。如果选择将Qt5软件用于GUI，则设置**-D WITH_QT=ON**。我们通过排除任何(Python)示例或测试来节省空间。

在**-D**标记之前只有空白空格，而不是制表符。顺便说一下，最后两个点不是错别字。它告诉CMake在哪里可以找到它的CMakeLists.txt(大的配置文件)。

另外，请注意所有头文件和OpenCV库所在的/usr目录。我们稍微偏爱/usr/local文件夹。现在，我们将遵循Tegra安装目录，以保持现有的软件在重新安装的OpenCV中平稳运行。

(1) OpenCV 4.6.0

```shell
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
-D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
-D WITH_OPENCL=OFF \
-D WITH_CUDA=ON \
-D CUDA_ARCH_BIN=5.3 \
-D CUDA_ARCH_PTX="" \
-D WITH_CUDNN=ON \
-D WITH_CUBLAS=ON \
-D ENABLE_FAST_MATH=ON \
-D CUDA_FAST_MATH=ON \
-D OPENCV_DNN_CUDA=ON \
-D ENABLE_NEON=ON \
-D WITH_QT=OFF \
-D WITH_OPENMP=ON \
-D BUILD_TIFF=ON \
-D WITH_FFMPEG=ON \
-D WITH_GSTREAMER=ON \
-D WITH_TBB=ON \
-D BUILD_TBB=ON \
-D BUILD_TESTS=OFF \
-D WITH_EIGEN=ON \
-D WITH_V4L=ON \
-D WITH_LIBV4L=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D INSTALL_C_EXAMPLES=OFF \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D PYTHON3_PACKAGES_PATH=/usr/lib/python3/dist-packages \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D BUILD_EXAMPLES=OFF ..
```

(2) OpenCV 4.5.5

```shell
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
-D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
-D WITH_OPENCL=OFF \
-D WITH_CUDA=ON \
-D CUDA_ARCH_BIN=5.3 \
-D CUDA_ARCH_PTX="" \
-D WITH_CUDNN=ON \
-D WITH_CUBLAS=ON \
-D ENABLE_FAST_MATH=ON \
-D CUDA_FAST_MATH=ON \
-D OPENCV_DNN_CUDA=ON \
-D ENABLE_NEON=ON \
-D WITH_QT=OFF \
-D WITH_OPENMP=ON \
-D BUILD_TIFF=ON \
-D WITH_FFMPEG=ON \
-D WITH_GSTREAMER=ON \
-D WITH_TBB=ON \
-D BUILD_TBB=ON \
-D BUILD_TESTS=OFF \
-D WITH_EIGEN=ON \
-D WITH_V4L=ON \
-D WITH_LIBV4L=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D INSTALL_C_EXAMPLES=OFF \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D PYTHON3_PACKAGES_PATH=/usr/lib/python3/dist-packages \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D BUILD_EXAMPLES=OFF ..
```

(3) OpenCV 4.5.4

```shell
cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
-D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
-D WITH_OPENCL=OFF \
-D WITH_CUDA=ON \
-D CUDA_ARCH_BIN=5.3 \
-D CUDA_ARCH_PTX="" \
-D WITH_CUDNN=ON \
-D WITH_CUBLAS=ON \
-D ENABLE_FAST_MATH=ON \
-D CUDA_FAST_MATH=ON \
-D OPENCV_DNN_CUDA=ON \
-D ENABLE_NEON=ON \
-D WITH_QT=OFF \
-D WITH_OPENMP=ON \
-D BUILD_TIFF=ON \
-D WITH_FFMPEG=ON \
-D WITH_GSTREAMER=ON \
-D WITH_TBB=ON \
-D BUILD_TBB=ON \
-D BUILD_TESTS=OFF \
-D WITH_EIGEN=ON \
-D WITH_V4L=ON \
-D WITH_LIBV4L=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D INSTALL_C_EXAMPLES=OFF \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D BUILD_opencv_python3=TRUE \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D BUILD_EXAMPLES=OFF ..
```

(4)  OpenCV 4.5.3

```shell
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
-D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
-D WITH_OPENCL=OFF \
-D WITH_CUDA=ON \
-D CUDA_ARCH_BIN=5.3 \
-D CUDA_ARCH_PTX="" \
-D WITH_CUDNN=ON \
-D WITH_CUBLAS=ON \
-D ENABLE_FAST_MATH=ON \
-D CUDA_FAST_MATH=ON \
-D OPENCV_DNN_CUDA=ON \
-D ENABLE_NEON=ON \
-D WITH_QT=OFF \
-D WITH_OPENMP=ON \
-D BUILD_TIFF=ON \
-D WITH_FFMPEG=ON \
-D WITH_GSTREAMER=ON \
-D WITH_TBB=ON \
-D BUILD_TBB=ON \
-D BUILD_TESTS=OFF \
-D WITH_EIGEN=ON \
-D WITH_V4L=ON \
-D WITH_LIBV4L=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D INSTALL_C_EXAMPLES=OFF \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D BUILD_opencv_python3=TRUE \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D BUILD_EXAMPLES=OFF ..
```

(5)  OpenCV 4.5.2

```shell
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
-D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
-D WITH_OPENCL=OFF \
-D WITH_CUDA=ON \
-D CUDA_ARCH_BIN=5.3 \
-D CUDA_ARCH_PTX="" \
-D WITH_CUDNN=ON \
-D WITH_CUBLAS=ON \
-D ENABLE_FAST_MATH=ON \
-D CUDA_FAST_MATH=ON \
-D OPENCV_DNN_CUDA=ON \
-D ENABLE_NEON=ON \
-D WITH_QT=OFF \
-D WITH_OPENMP=ON \
-D BUILD_TIFF=ON \
-D WITH_FFMPEG=ON \
-D WITH_GSTREAMER=ON \
-D WITH_TBB=ON \
-D BUILD_TBB=ON \
-D BUILD_TESTS=OFF \
-D WITH_EIGEN=ON \
-D WITH_V4L=ON \
-D WITH_LIBV4L=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D INSTALL_C_EXAMPLES=OFF \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D BUILD_opencv_python3=TRUE \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D BUILD_EXAMPLES=OFF ..
```

(6) OpenCV 4.5.1

```shell
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
-D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
-D WITH_OPENCL=OFF \
-D WITH_CUDA=ON \
-D CUDA_ARCH_BIN=5.3 \
-D CUDA_ARCH_PTX="" \
-D WITH_CUDNN=ON \
-D WITH_CUBLAS=ON \
-D ENABLE_FAST_MATH=ON \
-D CUDA_FAST_MATH=ON \
-D OPENCV_DNN_CUDA=ON \
-D ENABLE_NEON=ON \
-D WITH_QT=OFF \
-D WITH_OPENMP=ON \
-D BUILD_TIFF=ON \
-D WITH_FFMPEG=ON \
-D WITH_GSTREAMER=ON \
-D WITH_TBB=ON \
-D BUILD_TBB=ON \
-D BUILD_TESTS=OFF \
-D WITH_EIGEN=ON \
-D WITH_V4L=ON \
-D WITH_LIBV4L=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D INSTALL_C_EXAMPLES=OFF \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D BUILD_NEW_PYTHON_SUPPORT=ON \
-D BUILD_opencv_python3=TRUE \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D BUILD_EXAMPLES=OFF ..
```

(7) OpenCV 4.5.0

```shell
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
-D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
-D WITH_OPENCL=OFF \
-D WITH_CUDA=ON \
-D CUDA_ARCH_BIN=5.3 \
-D CUDA_ARCH_PTX="" \
-D WITH_CUDNN=ON \
-D WITH_CUBLAS=ON \
-D ENABLE_FAST_MATH=ON \
-D CUDA_FAST_MATH=ON \
-D OPENCV_DNN_CUDA=ON \
-D ENABLE_NEON=ON \
-D WITH_QT=OFF \
-D WITH_OPENMP=ON \
-D BUILD_TIFF=ON \
-D WITH_FFMPEG=ON \
-D WITH_GSTREAMER=ON \
-D WITH_TBB=ON \
-D BUILD_TBB=ON \
-D BUILD_TESTS=OFF \
-D WITH_EIGEN=ON \
-D WITH_V4L=ON \
-D WITH_LIBV4L=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D INSTALL_C_EXAMPLES=OFF \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D BUILD_NEW_PYTHON_SUPPORT=ON \
-D BUILD_opencv_python3=TRUE \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D BUILD_EXAMPLES=OFF ..
```

希望一切都很顺利，CMake附带了一个报告，看起来像下面的截图。

![CMake OpenCV ready](https://qengineering.eu/images/cmake-opencv.webp)

### 2.4.4 Make

所有编译指令就绪后，您可以使用以下命令开始构建。这大约需要两个小时!

```shell
$ make -j4
```

在最少一个半小时后，你的构建就应该准备好了，你会看到下面的屏幕。

![Make Rdy](https://qengineering.eu/images/Make-ready.webp)

成功编译之后，您可以删除旧的包，并使用以下命令在系统的数据库中安装所有新生成的包。

```shell
$ sudo rm -r /usr/include/opencv4/opencv2
$ sudo make install
$ sudo ldconfig
# cleaning (frees 300 MB)
$ make clean
$ sudo apt-get update
```

### 2.4.5 检查

现在是时候检查您的安装了。

![Succes_4_6](https://qengineering.eu/images/Succes_4_6_Jetson.webp)

OpenCV将被安装到/usr目录下，所有文件将被复制到以下位置:

+ /usr/bin      可执行文件

+ /usr/lib/aarch64-linux-gnu      库文件(.so)

+ /usr/lib/aarch64-linux-gnu/cmake/opencv4      cmake包

+ /usr/include/opencv4       头文件

+ /usr/share/opencv4         其他文件(例如经过训练的XML格式级联)

### 2.4.6 清除

如果磁盘空间不足，可以考虑删除**~/opencv**和**~/opencv_contrib**文件夹。通过**sudo make install**命令，您已经将所有头文件和库复制到磁盘上相应的位置。不再需要**~/opencv/build/**文件夹中的原始文件。当然，如果您已经启用了示例的构建，那么显然不能删除文件夹，因为这也将删除这些示例。或者，如果你计划使用在OpenCV人脸检测模块中使用的Haar Cascades脚本，因为它们位于**~/ OpenCV /data/haarcascades**文件夹中。这将节省1.5 GByte空间。

在任何情况下，请删除本手册开头安装的SD内存交换软件。

```shell
# remove the dphys-swapfile now
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
# free the allocated memory
$ sudo rm /var/swap
# just a tip to save an additional 275 MB
$ sudo rm -rf ~/opencv
$ sudo rm -rf ~/opencv_contrib
```

## 2.5 cv::dnn::net

OpenCV附带了一个优秀的深度学习框架模块。不需要任何额外的安装，你可以运行TensorFlow, Caffe, ONNX, PyTorch或Darknet模型。更多信息请参见OpenCV页面。

现在我们已经安装了带有CUDA支持的OpenCV，看到这为深度学习模型带来的性能提升是很有趣的。为此目的进行了几次试验。结果如下表所示。

![image-20221101152643532](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221101152643532.png)

前两列是超频Jetson Nano的结果。中间两个是Jetson Nano在常规时钟速度下的结果。最后一列是为了比较而添加的，是一个超频的树莓派4结果。

还要注意，模型越大，CUDA的影响就越大。像UltraFace这样非常小的模型会变得更慢。这与CPU和GPU之间相对耗时的内存交换有关。GPU一次处理的张量越多，性能提升越好。

要为您的网络启用CUDA支持，还需要另外两行代码。如果没有这些行，OpenCV就会启用默认的CPU模式。

还要注意，在您的应用程序开始运行之前还需要一段时间。这与在后台寻找最佳张量计算策略的相当耗时的启发式搜索有关。

```C++
#include <opencv2/dnn/dnn.hpp>

.....
    // load some network
    net = cv::dnn::readNetFromCaffe("ResNet-50.prototxt", "ResNet-50.caffemodel");

    // check
    if(net.empty()){
        std::cout << "init the model net error" << std::endl;
        return -1;
    }

    // enable CUDA support
    net.setPreferableBackend(cv::dnn::DNN_BACKEND_CUDA);
    net.setPreferableTarget(cv::dnn::DNN_TARGET_CUDA);
.....
```

## 2.6 JTOP

您可能已经安装了**JTOP**。如果没有，你应该马上做。Raffaello Bonghi特别为Jetson家族制作的一个优秀的调试工具。它提供了开发过程中需要的所有信息，就像htop一样。您可以使用以下命令安装它。

```shell
$ sudo -H pip install -U jetson-stats
$ sudo reboot
# start the app with the simple command
$ jtop
```

![JTOP](https://qengineering.eu/images/jtop3_4_6.webp)

![Jtop_OpenCV](https://qengineering.eu/images/jtop_4_6_Opencv.webp)

# 参考目录

[https://qengineering.eu/install-opencv-4.5-on-jetson-nano.html](https://qengineering.eu/install-opencv-4.5-on-jetson-nano.html)