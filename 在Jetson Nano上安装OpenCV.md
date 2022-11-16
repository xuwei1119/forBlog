# 在Jetson Nano上安装OpenCV

Jetson Nano自带OpenCV，但是不支持CUDA，先要使用CUDA功能，需要重新安装OpenCV。

# 1. 自带OpenCV3版本安装OpenCV4

如果Jetson Nano自带的是OpenCV3版本，则需要先删除旧版本再安装新版本。下面的**install_opencv4.5.0_Jetson.sh**、**install_opencv4.6.0_Jetson.sh**或者**install_opencv-4.5.5.sh**脚本可以实现这个功能。

## 1.1.在Jetson Nano上安装OpenCV4.5.0

创建**install_opencv4.5.0_Jetson.sh**文本，使用编辑器（vim或者nano）打开文件，在文件中输入以下内容

```shell
#!/bin/bash
#
# Copyright (c) 2020, NVIDIA CORPORATION.  All rights reserved.
#
# NVIDIA Corporation and its licensors retain all intellectual property
# and proprietary rights in and to this software, related documentation
# and any modifications thereto.  Any use, reproduction, disclosure or
# distribution of this software and related documentation without an express
# license agreement from NVIDIA Corporation is strictly prohibited.
#

version="4.5.0"
folder="workspace"

echo "** Remove other OpenCV first"
sudo sudo apt-get purge *libopencv*


echo "** Install requirement"
sudo apt-get update
sudo apt-get install -y build-essential cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install -y libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
sudo apt-get install -y python2.7-dev python3.6-dev python-dev python-numpy python3-numpy
sudo apt-get install -y libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev
sudo apt-get install -y libv4l-dev v4l-utils qv4l2 v4l2ucp
sudo apt-get install -y curl


echo "** Download opencv-"${version}
mkdir $folder
cd ${folder}
curl -L https://github.com/opencv/opencv/archive/${version}.zip -o opencv-${version}.zip
curl -L https://github.com/opencv/opencv_contrib/archive/${version}.zip -o opencv_contrib-${version}.zip
unzip opencv-${version}.zip
unzip opencv_contrib-${version}.zip
cd opencv-${version}/


echo "** Building..."
mkdir release
cd release/
cmake -D WITH_CUDA=ON -D WITH_CUDNN=ON -D CUDA_ARCH_BIN="5.3,6.2,7.2" -D CUDA_ARCH_PTX="" -D OPENCV_GENERATE_PKGCONFIG=ON -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-${version}/modules -D WITH_GSTREAMER=ON -D WITH_LIBV4L=ON -D BUILD_opencv_python2=ON -D BUILD_opencv_python3=ON -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_EXAMPLES=OFF -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
make -j$(nproc)
sudo make install


echo "** Install opencv-"${version}" successfully"
echo "** Bye :)"
```

## 1.2.在Jetson Nano上安装OpenCV4.6.0

创建**install_opencv4.6.0_Jetson.sh**文本，使用编辑器（vim或者nano）打开文件，在文件中输入以下内容

```shell
#!/bin/bash
#
# Copyright (c) 2022, NVIDIA CORPORATION.  All rights reserved.
#
# NVIDIA Corporation and its licensors retain all intellectual property
# and proprietary rights in and to this software, related documentation
# and any modifications thereto.  Any use, reproduction, disclosure or
# distribution of this software and related documentation without an express
# license agreement from NVIDIA Corporation is strictly prohibited.
#

version="4.6.0"
folder="workspace"

set -e

for (( ; ; ))
do
    echo "Do you want to remove the default OpenCV (yes/no)?"
    read rm_old

    if [ "$rm_old" = "yes" ]; then
        echo "** Remove other OpenCV first"
        sudo apt -y purge *libopencv*
	break
    elif [ "$rm_old" = "no" ]; then
	break
    fi
done


echo "------------------------------------"
echo "** Install requirement (1/4)"
echo "------------------------------------"
sudo apt-get update
sudo apt-get install -y build-essential cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install -y libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
sudo apt-get install -y python3.8-dev python-dev python-numpy python3-numpy
sudo apt-get install -y libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev
sudo apt-get install -y libv4l-dev v4l-utils qv4l2 v4l2ucp
sudo apt-get install -y curl


echo "------------------------------------"
echo "** Download opencv "${version}" (2/4)"
echo "------------------------------------"
mkdir $folder
cd ${folder}
curl -L https://github.com/opencv/opencv/archive/${version}.zip -o opencv-${version}.zip
curl -L https://github.com/opencv/opencv_contrib/archive/${version}.zip -o opencv_contrib-${version}.zip
unzip opencv-${version}.zip
unzip opencv_contrib-${version}.zip
rm opencv-${version}.zip opencv_contrib-${version}.zip
cd opencv-${version}/


echo "------------------------------------"
echo "** Build opencv "${version}" (3/4)"
echo "------------------------------------"
mkdir release
cd release/
cmake -D WITH_CUDA=ON -D WITH_CUDNN=ON -D CUDA_ARCH_BIN="7.2,8.7" -D CUDA_ARCH_PTX="" -D OPENCV_GENERATE_PKGCONFIG=ON -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-${version}/modules -D WITH_GSTREAMER=ON -D WITH_LIBV4L=ON -D BUILD_opencv_python3=ON -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_EXAMPLES=OFF -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
make -j$(nproc)


echo "------------------------------------"
echo "** Install opencv "${version}" (4/4)"
echo "------------------------------------"
sudo make install
echo 'export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export PYTHONPATH=/usr/local/lib/python3.8/site-packages/:$PYTHONPATH' >> ~/.bashrc
source ~/.bashrc


echo "** Install opencv "${version}" successfully"
echo "** Bye :)"
```

## 1.3.在Jetson Nano上安装OpenCV4.5.5

创建**install_opencv4.5.5.sh**文本，使用编辑器（vim或者nano）打开文件，在文件中输入以下内容

```shell
#!/bin/bash
#
# Copyright (c) 2018, NVIDIA CORPORATION.  All rights reserved.
#
# NVIDIA Corporation and its licensors retain all intellectual property
# and proprietary rights in and to this software, related documentation
# and any modifications thereto.  Any use, reproduction, disclosure or
# distribution of this software and related documentation without an express
# license agreement from NVIDIA Corporation is strictly prohibited.
#

# The orginal script 'install_opencv4.0.0_Nano.sh' could be found here:
# https://github.com/AastaNV/JEP/blob/master/script/install_opencv4.0.0_Nano.sh
#
# I did some modification so that it installs opencv-3.4.8 instead.  Refer
# to my blog posts for more details.
#
# JK Jung, jkjung13@gmail.com

set -e

chip_id=$(cat /sys/module/tegra_fuse/parameters/tegra_chip_id)
case ${chip_id} in
  "33" )  # Nano and TX1
    cuda_compute=5.3
    ;;
  "24" )  # TX2
    cuda_compute=6.2
    ;;
  "25" )  # AGX Xavier
    cuda_compute=7.2
    ;;
  * )     # default
    cuda_compute=5.3,6.2,7.2
    ;;
esac

py3_ver=$(python3 -c "import sys; print(sys.version_info[1])")

folder=${HOME}/src
mkdir -p $folder

echo "** Purge old opencv installation"
sudo apt-get purge -y libopencv*

echo "** Install requirements"
sudo apt-get update
sudo apt-get install -y build-essential make cmake cmake-curses-gui git g++ pkg-config curl
sudo apt-get install -y libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libeigen3-dev libglew-dev libgtk2.0-dev
sudo apt-get install -y libtbb2 libtbb-dev libv4l-dev v4l-utils qv4l2 v4l2ucp
sudo apt-get install -y libdc1394-22-dev libxine2-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
# sudo apt-get install -y libjasper-dev
sudo apt-get install -y libjpeg8-dev libjpeg-turbo8-dev libtiff-dev libpng-dev
sudo apt-get install -y libxvidcore-dev libx264-dev libgtk-3-dev
sudo apt-get install -y libatlas-base-dev libopenblas-dev liblapack-dev liblapacke-dev gfortran
sudo apt-get install -y qt5-default

sudo apt-get install -y python3-dev python3-testresources
rm -f $folder/get-pip.py
wget https://bootstrap.pypa.io/get-pip.py -O $folder/get-pip.py
sudo python3 $folder/get-pip.py
sudo pip3 install protobuf
sudo pip3 install -U numpy matplotlib

if [ ! -f /usr/local/cuda/include/cuda_gl_interop.h.bak ]; then
  sudo cp /usr/local/cuda/include/cuda_gl_interop.h /usr/local/cuda/include/cuda_gl_interop.h.bak
fi
sudo patch -N -r - /usr/local/cuda/include/cuda_gl_interop.h < opencv/cuda_gl_interop.h.patch && echo "** '/usr/local/cuda/include/cuda_gl_interop.h' appears to be patched already.  Continue..."

echo "** Download opencv-4.5.5"
cd $folder
if [ ! -f opencv-4.5.5.zip ]; then
  wget https://github.com/opencv/opencv/archive/4.5.5.zip -O opencv-4.5.5.zip
  wget https://github.com/opencv/opencv_contrib/archive/4.5.5.zip -O opencv_contrib.zip 
fi
if [ -d opencv-4.5.5 ]; then
  echo "** ERROR: opencv-4.5.5 directory already exists"
  exit
fi
unzip opencv-4.5.5.zip
unzip opencv_contrib.zip
mv opencv-4.5.5 opencv
mv opencv_contrib-4.5.5 opencv_contrib
cd opencv/

echo "** Building opencv..."
mkdir build
cd build/

cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_EXTRA_MODULES_PATH=~/src/opencv_contrib/modules \
      -D WITH_CUDA=ON -D CUDA_ARCH_BIN=${cuda_compute} -D CUDA_ARCH_PTX="" \
      -D WITH_CUBLAS=ON -D ENABLE_FAST_MATH=ON -D CUDA_FAST_MATH=ON \
      -D ENABLE_NEON=ON -D WITH_GSTREAMER=ON -D WITH_LIBV4L=ON \
      -D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
      -D BUILD_opencv_python2=OFF -D BUILD_opencv_python3=ON \
      -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_EXAMPLES=OFF \
      -D WITH_QT=ON -D WITH_OPENGL=ON ..
make -j$(nproc)
sudo make install
sudo ldconfig

python3 -c 'import cv2; print("python3 cv2 version: %s" % cv2.__version__)'

echo "** Install opencv-4.5.5 successfully"
```



# 2.自带OpenCV4安装OpenCV4

如果Jetson Nano自带的是OpenCV4版本，则下面的**OpenCV-4-5-5.sh**或者**OpenCV-4-6-0.sh**脚本可以实现这个功能。

## 2.1.在Jetson Nano上安装OpenCV4.5.5

创建**OpenCV-4-5-5.sh**文本，使用编辑器（vim或者nano）打开文件，在文件中输入以下内容

```shell
#!/bin/bash
set -e

echo "Installing OpenCV 4.5.5 on your Jetson Nano"
echo "It will take 3 hours !"

# reveal the CUDA location
cd ~
sudo sh -c "echo '/usr/local/cuda/lib64' >> /etc/ld.so.conf.d/nvidia-tegra.conf"
sudo ldconfig

# install the dependencies
sudo apt-get install -y build-essential cmake git unzip pkg-config zlib1g-dev
sudo apt-get install -y libjpeg-dev libjpeg8-dev libjpeg-turbo8-dev libpng-dev libtiff-dev
sudo apt-get install -y libavcodec-dev libavformat-dev libswscale-dev libglew-dev
sudo apt-get install -y libgtk2.0-dev libgtk-3-dev libcanberra-gtk*
sudo apt-get install -y python-dev python-numpy python-pip
sudo apt-get install -y python3-dev python3-numpy python3-pip
sudo apt-get install -y libxvidcore-dev libx264-dev libgtk-3-dev
sudo apt-get install -y libtbb2 libtbb-dev libdc1394-22-dev libxine2-dev
sudo apt-get install -y gstreamer1.0-tools libv4l-dev v4l-utils qv4l2 
sudo apt-get install -y libgstreamer-plugins-base1.0-dev libgstreamer-plugins-good1.0-dev
sudo apt-get install -y libavresample-dev libvorbis-dev libxine2-dev libtesseract-dev
sudo apt-get install -y libfaac-dev libmp3lame-dev libtheora-dev libpostproc-dev
sudo apt-get install -y libopencore-amrnb-dev libopencore-amrwb-dev
sudo apt-get install -y libopenblas-dev libatlas-base-dev libblas-dev
sudo apt-get install -y liblapack-dev liblapacke-dev libeigen3-dev gfortran
sudo apt-get install -y libhdf5-dev protobuf-compiler
sudo apt-get install -y libprotobuf-dev libgoogle-glog-dev libgflags-dev

# remove old versions or previous builds
cd ~ 
sudo rm -rf opencv*
# download the latest version
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.5.5.zip 
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.5.5.zip 
# unpack
unzip opencv.zip 
unzip opencv_contrib.zip 
# some administration to make live easier later on
mv opencv-4.5.5 opencv
mv opencv_contrib-4.5.5 opencv_contrib
# clean up the zip files
rm opencv.zip
rm opencv_contrib.zip

# set install dir
cd ~/opencv
mkdir build
cd build

# run cmake
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
-D PYTHON3_PACKAGES_PATH=/usr/lib/python3/dist-packages \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D BUILD_EXAMPLES=OFF ..

# run make
FREE_MEM="$(free -m | awk '/^Swap/ {print $2}')"
# Use "-j 4" only swap space is larger than 5.5GB
if [[ "FREE_MEM" -gt "5500" ]]; then
  NO_JOB=4
else
  echo "Due to limited swap, make only uses 1 core"
  NO_JOB=1
fi
make -j ${NO_JOB} 

sudo rm -r /usr/include/opencv4/opencv2
sudo make install
sudo ldconfig

# cleaning (frees 300 MB)
make clean
sudo apt-get update

echo "Congratulations!"
echo "You've successfully installed OpenCV 4.5.5 on your Jetson Nano"
```

## 2.2.在Jetson Nano上安装OpenCV4.6.0

创建**OpenCV-4-6-0.sh**文本，使用编辑器（vim或者nano）打开文件，在文件中输入以下内容

```shell
#!/bin/bash
set -e

echo "Installing OpenCV 4.6.0 on your Jetson Nano"
echo "It will take 3 hours !"

# reveal the CUDA location
cd ~
sudo sh -c "echo '/usr/local/cuda/lib64' >> /etc/ld.so.conf.d/nvidia-tegra.conf"
sudo ldconfig

# install the dependencies
sudo apt-get install -y build-essential cmake git unzip pkg-config zlib1g-dev
sudo apt-get install -y libjpeg-dev libjpeg8-dev libjpeg-turbo8-dev libpng-dev libtiff-dev
sudo apt-get install -y libavcodec-dev libavformat-dev libswscale-dev libglew-dev
sudo apt-get install -y libgtk2.0-dev libgtk-3-dev libcanberra-gtk*
sudo apt-get install -y python-dev python-numpy python-pip
sudo apt-get install -y python3-dev python3-numpy python3-pip
sudo apt-get install -y libxvidcore-dev libx264-dev libgtk-3-dev
sudo apt-get install -y libtbb2 libtbb-dev libdc1394-22-dev libxine2-dev
sudo apt-get install -y gstreamer1.0-tools libv4l-dev v4l-utils qv4l2 
sudo apt-get install -y libgstreamer-plugins-base1.0-dev libgstreamer-plugins-good1.0-dev
sudo apt-get install -y libavresample-dev libvorbis-dev libxine2-dev libtesseract-dev
sudo apt-get install -y libfaac-dev libmp3lame-dev libtheora-dev libpostproc-dev
sudo apt-get install -y libopencore-amrnb-dev libopencore-amrwb-dev
sudo apt-get install -y libopenblas-dev libatlas-base-dev libblas-dev
sudo apt-get install -y liblapack-dev liblapacke-dev libeigen3-dev gfortran
sudo apt-get install -y libhdf5-dev protobuf-compiler
sudo apt-get install -y libprotobuf-dev libgoogle-glog-dev libgflags-dev

# remove old versions or previous builds
cd ~ 
sudo rm -rf opencv*
# download the latest version
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.6.0.zip 
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.6.0.zip 
# unpack
unzip opencv.zip 
unzip opencv_contrib.zip 
# some administration to make live easier later on
mv opencv-4.6.0 opencv
mv opencv_contrib-4.6.0 opencv_contrib
# clean up the zip files
rm opencv.zip
rm opencv_contrib.zip

# set install dir
cd ~/opencv
mkdir build
cd build

# run cmake
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
-D PYTHON3_PACKAGES_PATH=/usr/lib/python3/dist-packages \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D BUILD_EXAMPLES=OFF ..

# run make
FREE_MEM="$(free -m | awk '/^Swap/ {print $2}')"
# Use "-j 4" only swap space is larger than 5.5GB
if [[ "FREE_MEM" -gt "5500" ]]; then
  NO_JOB=4
else
  echo "Due to limited swap, make only uses 1 core"
  NO_JOB=1
fi
make -j ${NO_JOB} 

sudo rm -r /usr/include/opencv4/opencv2
sudo make install
sudo ldconfig

# cleaning (frees 300 MB)
make clean
sudo apt-get update

echo "Congratulations!"
echo "You've successfully installed OpenCV 4.6.0 on your Jetson Nano"
```

# 3.自定义安装OpenCV3

如果还想安装支持CUDA的OpenCV3版本，则下面的**install_opencv-3.4.8.sh**或者**install_opencv-3.4.6.sh**脚本可以实现这个功能。

## 3.1 在Jetson Nano上安装OpenCV3.4.8

创建**install_opencv-3.4.8.sh**文件，输入以下代码：

```shell
#!/bin/bash
#
# Copyright (c) 2018, NVIDIA CORPORATION.  All rights reserved.
#
# NVIDIA Corporation and its licensors retain all intellectual property
# and proprietary rights in and to this software, related documentation
# and any modifications thereto.  Any use, reproduction, disclosure or
# distribution of this software and related documentation without an express
# license agreement from NVIDIA Corporation is strictly prohibited.
#

# The orginal script 'install_opencv4.0.0_Nano.sh' could be found here:
# https://github.com/AastaNV/JEP/blob/master/script/install_opencv4.0.0_Nano.sh
#
# I did some modification so that it installs opencv-3.4.8 instead.  Refer
# to my blog posts for more details.
#
# JK Jung, jkjung13@gmail.com

set -e

chip_id=$(cat /sys/module/tegra_fuse/parameters/tegra_chip_id)
case ${chip_id} in
  "33" )  # Nano and TX1
    cuda_compute=5.3
    ;;
  "24" )  # TX2
    cuda_compute=6.2
    ;;
  "25" )  # AGX Xavier
    cuda_compute=7.2
    ;;
  * )     # default
    cuda_compute=5.3,6.2,7.2
    ;;
esac

py3_ver=$(python3 -c "import sys; print(sys.version_info[1])")

folder=${HOME}/src
mkdir -p $folder

echo "** Purge old opencv installation"
sudo apt-get purge -y libopencv*

echo "** Install requirements"
sudo apt-get update
sudo apt-get install -y build-essential make cmake cmake-curses-gui git g++ pkg-config curl
sudo apt-get install -y libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libeigen3-dev libglew-dev libgtk2.0-dev
sudo apt-get install -y libtbb2 libtbb-dev libv4l-dev v4l-utils qv4l2 v4l2ucp
sudo apt-get install -y libdc1394-22-dev libxine2-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
# sudo apt-get install -y libjasper-dev
sudo apt-get install -y libjpeg8-dev libjpeg-turbo8-dev libtiff-dev libpng-dev
sudo apt-get install -y libxvidcore-dev libx264-dev libgtk-3-dev
sudo apt-get install -y libatlas-base-dev libopenblas-dev liblapack-dev liblapacke-dev gfortran
sudo apt-get install -y qt5-default

sudo apt-get install -y python3-dev python3-testresources
rm -f $folder/get-pip.py
wget https://bootstrap.pypa.io/get-pip.py -O $folder/get-pip.py
sudo python3 $folder/get-pip.py
sudo pip3 install protobuf
sudo pip3 install -U numpy matplotlib

if [ ! -f /usr/local/cuda/include/cuda_gl_interop.h.bak ]; then
  sudo cp /usr/local/cuda/include/cuda_gl_interop.h /usr/local/cuda/include/cuda_gl_interop.h.bak
fi
sudo patch -N -r - /usr/local/cuda/include/cuda_gl_interop.h < opencv/cuda_gl_interop.h.patch && echo "** '/usr/local/cuda/include/cuda_gl_interop.h' appears to be patched already.  Continue..."

echo "** Download opencv-3.4.8"
cd $folder
if [ ! -f opencv-3.4.8.zip ]; then
  wget https://github.com/opencv/opencv/archive/3.4.8.zip -O opencv-3.4.8.zip
fi
if [ -d opencv-3.4.8 ]; then
  echo "** ERROR: opencv-3.4.8 directory already exists"
  exit
fi
unzip opencv-3.4.8.zip 
cd opencv-3.4.8/

echo "** Building opencv..."
mkdir build
cd build/

cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D WITH_CUDA=ON -D CUDA_ARCH_BIN=${cuda_compute} -D CUDA_ARCH_PTX="" \
      -D WITH_CUBLAS=ON -D ENABLE_FAST_MATH=ON -D CUDA_FAST_MATH=ON \
      -D ENABLE_NEON=ON -D WITH_GSTREAMER=ON -D WITH_LIBV4L=ON \
      -D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
      -D BUILD_opencv_python2=OFF -D BUILD_opencv_python3=ON \
      -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_EXAMPLES=OFF \
      -D WITH_QT=ON -D WITH_OPENGL=ON ..
make -j$(nproc)
sudo make install
sudo ldconfig

python3 -c 'import cv2; print("python3 cv2 version: %s" % cv2.__version__)'

echo "** Install opencv-3.4.8 successfully"
```

## 3.2 在Jetson Nano上安装OpenCV3.4.6

创建**install_opencv-3.4.6.sh**文件，输入以下代码：

```shell
#!/bin/bash
#
# Copyright (c) 2018, NVIDIA CORPORATION.  All rights reserved.
#
# NVIDIA Corporation and its licensors retain all intellectual property
# and proprietary rights in and to this software, related documentation
# and any modifications thereto.  Any use, reproduction, disclosure or
# distribution of this software and related documentation without an express
# license agreement from NVIDIA Corporation is strictly prohibited.
#

# The orginal script 'install_opencv4.0.0_Nano.sh' could be found here:
# https://github.com/AastaNV/JEP/blob/master/script/install_opencv4.0.0_Nano.sh
#
# I did some modification so that it installs opencv-3.4.6 instead.  Refer
# to my blog posts for more details.
#
# JK Jung, jkjung13@gmail.com

set -e

chip_id=$(cat /sys/module/tegra_fuse/parameters/tegra_chip_id)
case ${chip_id} in
  "33" )  # Nano and TX1
    cuda_compute=5.3
    ;;
  "24" )  # TX2
    cuda_compute=6.2
    ;;
  "25" )  # AGX Xavier
    cuda_compute=7.2
    ;;
  * )     # default
    cuda_compute=5.3,6.2,7.2
    ;;
esac

py3_ver=$(python3 -c "import sys; print(sys.version_info[1])")

folder=${HOME}/src
mkdir -p $folder

echo "** Purge old opencv installation"
sudo apt-get purge -y libopencv*

echo "** Install requirements"
sudo apt-get update
sudo apt-get install -y build-essential make cmake cmake-curses-gui git g++ pkg-config curl
sudo apt-get install -y libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libeigen3-dev libglew-dev libgtk2.0-dev
sudo apt-get install -y libtbb2 libtbb-dev libv4l-dev v4l-utils qv4l2 v4l2ucp
sudo apt-get install -y libdc1394-22-dev libxine2-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
# sudo apt-get install -y libjasper-dev
sudo apt-get install -y libjpeg8-dev libjpeg-turbo8-dev libtiff-dev libpng-dev
sudo apt-get install -y libxvidcore-dev libx264-dev libgtk-3-dev
sudo apt-get install -y libatlas-base-dev libopenblas-dev liblapack-dev liblapacke-dev gfortran
sudo apt-get install -y qt5-default

sudo apt-get install -y python3-dev python3-testresources
rm -f $folder/get-pip.py
wget https://bootstrap.pypa.io/get-pip.py -O $folder/get-pip.py
sudo python3 $folder/get-pip.py
sudo pip3 install protobuf
sudo pip3 install -U numpy matplotlib

if [ ! -f /usr/local/cuda/include/cuda_gl_interop.h.bak ]; then
  sudo cp /usr/local/cuda/include/cuda_gl_interop.h /usr/local/cuda/include/cuda_gl_interop.h.bak
fi
sudo patch -N -r - /usr/local/cuda/include/cuda_gl_interop.h < opencv/cuda_gl_interop.h.patch && echo "** '/usr/local/cuda/include/cuda_gl_interop.h' appears to be patched already.  Continue..."

echo "** Download opencv-3.4.6"
cd $folder
if [ ! -f opencv-3.4.6.zip ]; then
  wget https://github.com/opencv/opencv/archive/3.4.6.zip -O opencv-3.4.6.zip
fi
if [ -d opencv-3.4.6 ]; then
  echo "** ERROR: opencv-3.4.6 directory already exists"
  exit
fi
unzip opencv-3.4.6.zip 
cd opencv-3.4.6/

echo "** Building opencv..."
mkdir build
cd build/

cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D WITH_CUDA=ON -D CUDA_ARCH_BIN=${cuda_compute} -D CUDA_ARCH_PTX="" \
      -D WITH_CUBLAS=ON -D ENABLE_FAST_MATH=ON -D CUDA_FAST_MATH=ON \
      -D ENABLE_NEON=ON -D WITH_GSTREAMER=ON -D WITH_LIBV4L=ON \
      -D BUILD_opencv_python2=OFF -D BUILD_opencv_python3=ON \
      -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_EXAMPLES=OFF \
      -D WITH_QT=ON -D WITH_OPENGL=ON ..
make -j$(nproc)
sudo make install
sudo ldconfig

python3 -c 'import cv2; print("python3 cv2 version: %s" % cv2.__version__)'

echo "** Install opencv-3.4.6 successfully"
```

安装好以后使用以下代码检查

```shell
python3 -c "import cv2; print(cv2.__version__)"
```



# 参考目录

[https://github.com/Qengineering/Install-OpenCV-Jetson-Nano](https://github.com/Qengineering/Install-OpenCV-Jetson-Nano)

[https://github.com/AastaNV/JEP/tree/master/script](https://github.com/AastaNV/JEP/tree/master/script)

[https://github.com/jkjung-avt/jetson_nano](https://github.com/jkjung-avt/jetson_nano)