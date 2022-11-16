# 在Jetson Nano上安装PyTorch

# 1.简介

本页面将指导您在Jetson Nano上安装PyTorch, TorchVision, LibTorch和caff2。

PyTorch是一个专门为深度学习开发的软件库。它会消耗你的Jetson Nano的大量资源。所以，不要期待奇迹。它可以运行你的模型，但它不能训练新的模型。由于可用RAM的数量有限，所谓的迁移学习可能会导致问题。

我们将讨论两个安装，其中一个使用Python 3轮子。另一种方法是从头构建。不幸的是，Jetson Nano没有官方的pip3轮子。不过，我们创建了这些轮子，并把它们放在GitHub上，以方便您使用。

（1）PyTorch 1.12, 1.11。

PyTorch 1.11及以上版本需要JetPack 5.0中的Python 3.7。

因为JetPack 4.6有Python 3.6，所以你不能在Jetson Nano上安装PyTorch 1.11.0。

看起来英伟达目前还没有计划为Jetson Nano发布新的JetPack 5.0。它只适用于Xavier系列。

但是，您可以在Ubuntu 20.04中使用当前版本的Jetson Nano。我们在GitHub上为这个版本提供轮子。

（2）PyTorch 1.10。

PyTorch 1.10进行了通常的改进和bug修复。请注意，与版本1.9相比，有些操作具有不同的行为。

（3）PyTorch 1.9。

关于1.9.0版本的一些警告。从这里可以看到，与上一个版本相比，软件做了相当多的更改。不再支持所有的操作和声明。当1.8网络在这个新版本上运行时，它可能会导致向后兼容性问题。

# 2.使用轮子安装

PyTorch是由Ninja构建的。它需要超过5个小时来完成整个构建。我们已经在我们的GitHub页面上发布了轮子。请随意使用这些。所有繁琐的工作已经完成，现在只需要几分钟就可以在Nano上安装PyTorch。

整个快捷程序如下所示。轮子太大，无法存储在GitHub中，所以使用谷歌驱动器。请确保您安装了最新的pip3和python3版本，否则，pip可能会附带消息`.whl is not a supported wheel on this platform`。

查看[GitHub页面](https://github.com/Qengineering/PyTorch-Jetson-Nano)的所有轮子。

![Python version check](https://qengineering.eu/images/Python3Jetson.webp)

(1) PyTorch1.12.0

```shell
Only for a Jetson Nano with Ubuntu 20.04

# install the dependencies (if not already onboard)
$ sudo apt-get install python3-pip libjpeg-dev libopenblas-dev libopenmpi-dev libomp-dev
$ sudo -H pip3 install future
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install Cython
# install gdown to download from Google drive
$ sudo -H pip3 install gdown
# download the wheel
$ gdown https://drive.google.com/uc?id=1MnVB7I4N8iVDAkogJO76CiQ2KRbyXH_e
# install PyTorch 1.12.0
$ sudo -H pip3 install torch-1.12.0a0+git67ece03-cp38-cp38-linux_aarch64.whl
# clean up
$ rm torch-1.12.0a0+git67ece03-cp38-cp38-linux_aarch64.whl
```

(2) PyTorch1.11.0

```shell
Only for a Jetson Nano with Ubuntu 20.04

# install the dependencies (if not already onboard)
$ sudo apt-get install python3-pip libjpeg-dev libopenblas-dev libopenmpi-dev libomp-dev
$ sudo -H pip3 install future
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install Cython
# install gdown to download from Google drive
$ sudo -H pip3 install gdown
# download the wheel
$ gdown https://drive.google.com/uc?id=1AQQuBS9skNk1mgZXMp0FmTIwjuxc81WY
# install PyTorch 1.11.0
$ sudo -H pip3 install torch-1.11.0a0+gitbc2c6ed-cp38-cp38-linux_aarch64.whl
# clean up
$ rm torch-1.11.0a0+gitbc2c6ed-cp38-cp38-linux_aarch64.whl
```

(3) PyTorch1.10.0

```shell
# install the dependencies (if not already onboard)
$ sudo apt-get install python3-pip libjpeg-dev libopenblas-dev libopenmpi-dev libomp-dev
$ sudo -H pip3 install future
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install Cython
# install gdown to download from Google drive
$ sudo -H pip3 install gdown
# download the wheel
$ gdown https://drive.google.com/uc?id=1TqC6_2cwqiYacjoLhLgrZoap6-sVL2sd
# install PyTorch 1.10.0
$ sudo -H pip3 install torch-1.10.0a0+git36449ea-cp36-cp36m-linux_aarch64.whl
# clean up
$ rm torch-1.10.0a0+git36449ea-cp36-cp36m-linux_aarch64.whl
```

(4) PyTorch1.9.0

```shell
# install the dependencies (if not already onboard)
$ sudo apt-get install python3-pip libjpeg-dev libopenblas-dev libopenmpi-dev libomp-dev
$ sudo -H pip3 install future
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install Cython
# install gdown to download from Google drive
$ sudo -H pip3 install gdown
# download the wheel
$ gdown https://drive.google.com/uc?id=1wzIDZEJ9oo62_H2oL7fYTp5_-NffCXzt
# install PyTorch 1.9.0
$ sudo -H pip3 install torch-1.9.0a0+gitd69c22d-cp36-cp36m-linux_aarch64.whl
# clean up
$ rm torch-1.9.0a0+gitd69c22d-cp36-cp36m-linux_aarch64.whl
```

(5) PyTorch1.8.0

```shell
# install the dependencies (if not already onboard)
$ sudo apt-get install python3-pip libjpeg-dev libopenblas-dev libopenmpi-dev libomp-dev
$ sudo -H pip3 install future
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install Cython
# install gdown to download from Google drive
$ sudo -H pip3 install gdown
# download the wheel
$ gdown https://drive.google.com/uc?id=1-XmTOEN0z1_-VVCI3DPwmcdC-eLT_-n3
# install PyTorch 1.8.0
$ sudo -H pip3 install torch-1.8.0a0+37c1f4a-cp36-cp36m-linux_aarch64.whl
# clean up
$ rm torch-1.8.0a0+37c1f4a-cp36-cp36m-linux_aarch64.whl
```

(6) PyTorch1.7.0

```shell
# install the dependencies (if not already onboard)
$ sudo apt-get install python3-pip libjpeg-dev libopenblas-dev libopenmpi-dev libomp-dev
$ sudo -H pip3 install future
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install Cython
# install gdown to download from Google drive
$ sudo -H pip3 install gdown
# download the wheel
$ gdown https://drive.google.com/uc?id=1aWuKu8eqkZwVzFFvguVuwkj0zdCir9qX
# install PyTorch 1.7.0
$ sudo -H pip3 install torch-1.7.0a0-cp36-cp36m-linux_aarch64.whl
# clean up
$ rm torch-1.7.0a0-cp36-cp36m-linux_aarch64.whl
```

成功安装之后，可以使用以下命令检查PyTorch。

![PyTorch 1.9.0 success](https://qengineering.eu/images/Torch_1_9_0_Jetson_Succes.webp)

# 3.源码安装

## 3.1 安装Python3 PyTorch

从头开始构建PyTorch相对容易。首先安装一些依赖项，然后从GitHub下载压缩包，最后构建软件。

注意，在超频Jetson Nano上，整个过程大约需要8个小时。

（1）PyTorch1.12.0

```shell
Only for a Jetson Nano with Ubuntu 20.04

# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# the dependencies
$ sudo apt-get install ninja-build git cmake
$ sudo apt-get install libjpeg-dev libopenmpi-dev libomp-dev ccache
$ sudo apt-get install libopenblas-dev libblas-dev libeigen3-dev
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install scikit-build
# download PyTorch 1.12.0 with all its libraries
$ git clone -b v1.12.0 --depth=1 --recursive https://github.com/pytorch/pytorch.git
$ cd pytorch
# one command to install several dependencies in one go
# installs future, numpy, pyyaml, requests
# setuptools, six, typing_extensions, dataclasses
$ sudo pip3 install -r requirements.txt
```

（2）PyTorch1.11.0

```shell
Only for a Jetson Nano with Ubuntu 20.04

# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# the dependencies
$ sudo apt-get install ninja-build git cmake
$ sudo apt-get install libjpeg-dev libopenmpi-dev libomp-dev ccache
$ sudo apt-get install libopenblas-dev libblas-dev libeigen3-dev
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install scikit-build
# download PyTorch 1.11.0 with all its libraries
$ git clone -b v1.11.0 --depth=1 --recursive https://github.com/pytorch/pytorch.git
$ cd pytorch
# one command to install several dependencies in one go
# installs future, numpy, pyyaml, requests
# setuptools, six, typing_extensions, dataclasses
$ sudo pip3 install -r requirements.txt
```

（3）PyTorch1.10.0

```shell
# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# the dependencies
$ sudo apt-get install ninja-build git cmake
$ sudo apt-get install libjpeg-dev libopenmpi-dev libomp-dev ccache
$ sudo apt-get install libopenblas-dev libblas-dev libeigen3-dev
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install scikit-build
# download PyTorch 1.10.0 with all its libraries
$ git clone -b v1.10.0 --depth=1 --recursive https://github.com/pytorch/pytorch.git
$ cd pytorch
# one command to install several dependencies in one go
# installs future, numpy, pyyaml, requests
# setuptools, six, typing_extensions, dataclasses
$ sudo pip3 install -r requirements.txt
```

（4）PyTorch1.9.0

```shell
# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# the dependencies
$ sudo apt-get install ninja-build git cmake
$ sudo apt-get install libjpeg-dev libopenmpi-dev libomp-dev ccache
$ sudo apt-get install libopenblas-dev libblas-dev libeigen3-dev
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install scikit-build
# download PyTorch 1.9.0 with all its libraries
$ git clone -b v1.9.0 --depth=1 --recursive https://github.com/pytorch/pytorch.git
$ cd pytorch
# one command to install several dependencies in one go
# installs future, numpy, pyyaml, requests
# setuptools, six, typing_extensions, dataclasses
$ sudo pip3 install -r requirements.txt
```

（5）PyTorch1.8.0

```shell
# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# the dependencies
$ sudo apt-get install ninja-build git cmake
$ sudo apt-get install libjpeg-dev libopenmpi-dev libomp-dev ccache
$ sudo apt-get install libopenblas-dev libblas-dev libeigen3-dev
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install scikit-build
# download PyTorch with all its libraries
$ git clone -b v1.8.0 --depth=1 --recursive https://github.com/pytorch/pytorch.git
$ cd pytorch
# one command to install several dependencies in one go
# installs future, numpy, pyyaml, requests
# setuptools, six, typing_extensions, dataclasses
$ sudo pip3 install -r requirements.txt
```

（6）PyTorch1.7.0

```shell
# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# the dependencies
$ sudo apt-get install ninja-build git cmake
$ sudo apt-get install libjpeg-dev libopenmpi-dev libomp-dev ccache
$ sudo apt-get install libopenblas-dev libblas-dev libeigen3-dev
$ sudo pip3 install -U --user wheel mock pillow
$ sudo -H pip3 install testresources
# above 58.3.0 you get version issues
$ sudo -H pip3 install setuptools==58.3.0
$ sudo -H pip3 install scikit-build
# download PyTorch with all its libraries
$ git clone -b v1.7.0 --depth=1 --recursive https://github.com/pytorch/pytorch.git
$ cd pytorch
# one command to install several dependencies in one go
# installs future, numpy, pyyaml, requests
# setuptools, six, typing_extensions, dataclasses
$ sudo pip3 install -r requirements.txt
```

## 3.2扩大内存交换

构建完整的PyTorch需要超过4Gb的RAM和通常在Jetson Nano上找到的由zram提供的2Gb交换空间。我们必须安装dphysi -swapfile来临时从SD卡获得额外的空间。编译完成后，该机制将被删除，从而消除对SD卡的交换。

您需要增加dphys交换超过常规的2048限制。首先将` /sbin/dphys-swapfile`中的最大边界更改为4096。接下来，设置`/etc/dphys-swapfile`。幻灯片会引导你。如果没有足够的交换内存，编译将产生模糊的`CalledProcessErrors`。

我们不建议增加zram交换限额。您不能为了获得一些额外的空间而一直压缩系统内存。最好是临时使用SD内存。安装PyTorch之后，您可以再次删除dphysics -swapfile。

请执行下面的命令。还要注意nano的安装，这是一个微型文本编辑器。

```shell
# a fresh start, so check for updates
$ sudo apt-get update
$ sudo apt-get upgrade
# install nano
$ sudo apt-get install nano
# install dphys-swapfile
$ sudo apt-get install dphys-swapfile
# enlarge the boundary
$ sudo nano /sbin/dphys-swapfile
# give the required memory size
$ sudo nano /etc/dphys-swapfile
# reboot afterwards
$ sudo reboot.
```

![SwapMaxJetsonPy1.png](https://qengineering.eu/my_gallery/SwapMaxJetsonPy1.webp)

## 3.3 Clang编译器

在开始构建之前，需要一些准备工作。首先，您必须在Jetson Nano上安装最新的clang编译器。在编译PyTorch时，GNU编译器和Jetson Nano总会出现一系列问题。通常，这与ARM核心的NEON架构支持差有关，导致浮点被截断。

![GNUerror](https://qengineering.eu/images/GNUerror.webp)

或者导致浮点错误。

![ClangNano](https://qengineering.eu/images/ClangNano.webp)

奇怪的是，clang编译器似乎对代码没有任何问题，所以这次应该使用clang。我们知道有人不喜欢clang。GNU编译器曾经比clang更优秀，但那些日子已经一去不复返了。如今，这两个编译器的性能几乎完全相同。

```shell
# install the clang compiler (version 8)
$ sudo apt-get install clang-8
# create symlinks to clang
$ sudo ln -s /usr/bin/clang-8 /usr/bin/clang
$ sudo ln -s /usr/bin/clang++-8 /usr/bin/clang++
```

接下来，必须修改刚刚从GitHub下载的PyTorch代码。所有更改都会限制运行时可用的CUDA线程的最大值。有四个地方需要我们注意。

**PyTorch >1.10:**    **~/pytorch/aten/src/ATen/cpu/vec/vec256/vec256_float_neon.h**

**PyTorch <1.10 :**  **~/pytorch/aten/src/ATen/cpu/vec256/vec256_float_neon.h**

在第28行添加` \#if defined(__clang__) ||(__GNUC__ > 8 || (__GNUC__ == 8 && __GNUC_MINOR__ > 3))`和匹配的闭包`#endif`。

![Prep1](https://qengineering.eu/images/Prep1.webp)

**~/pytorch/aten/src/ATen/cuda/CUDAContext.cpp**

在第24行左右添加额外的一行`device_prop.maxThreadsPerBlock = device_prop.maxThreadsPerBlock / 2;`

![Prep2](https://qengineering.eu/images/Prep2.webp)

**~/pytorch/aten/src/ATen/cuda/detail/KernelUtils.h**

在第26行中，将常量从1024更改为512。

![Prep3](https://qengineering.eu/images/Prep3.webp)

**PyTorch >1.10 :**  **common.h is no longer in use.**

**PyTorch <1.10 :** **~/pytorch/aten/src/THCUNN/common.h**

在第22行相同的修改中，将CUDA_NUM_THREADS从1024更改为512。

![Prep4](https://qengineering.eu/images/Prep4.webp)

所有准备工作完成后，我们现在可以设置环境参数，以便Ninja编译器获得关于我们希望如何构建PyTorch的正确指示。如您所知，这些指令仅在当前终端有效。如果在另一个终端中启动构建，则需要再次设置参数。

还要注意指令末尾的符号链接。NVIDIA已经将`cublas`库从`/usr/local/cuda/lib64/`移到了`/usr/lib/aarch64-linux-gnu/`文件夹中，留下了很多软件，比如PyTorch，有坏链接。符号链接是这里最好的解决方法。

另一个值得注意的点是arch_list。不仅给出Jetson Nano的5.3而且也给出Jetson Xavier的6.2或7.2。现在轮子也支持Xavier设备。

```shell
# set NINJA parameters
$ cd pytorch
$ export BUILD_CAFFE2_OPS=OFF
$ export USE_FBGEMM=OFF
$ export USE_FAKELOWP=OFF
$ export BUILD_TEST=OFF
$ export USE_MKLDNN=OFF
$ export USE_NNPACK=OFF
$ export USE_XNNPACK=OFF
$ export USE_QNNPACK=OFF
$ export USE_PYTORCH_QNNPACK=OFF
$ export USE_CUDA=ON
$ export USE_CUDNN=ON
$ export TORCH_CUDA_ARCH_LIST="5.3;6.2;7.2"
$ export USE_NCCL=OFF
$ export USE_SYSTEM_NCCL=OFF
$ export USE_OPENCV=OFF
$ export MAX_JOBS=4
# set path to ccache
$ export PATH=/usr/lib/ccache:$PATH
# set clang compiler
$ export CC=clang
$ export CXX=clang++
# set cuda compiler
$ export CUDACXX=/usr/local/cuda/bin/nvcc
# create symlink to cublas
$ sudo ln -s /usr/lib/aarch64-linux-gnu/libcublas.so /usr/local/cuda/lib64/libcublas.so
# clean up the previous build, if necessary
$ python3 setup.py clean
# start the build
$ python3 setup.py bdist_wheel
```

![BuildEnvironment](https://qengineering.eu/images/Environment.webp)

一旦Ninja完成构建，你可以在你的Jetson Nano上使用生成的轮子安装PyTorch。按照下面的说明操作。

```shell
# install the wheel found in the dist folder
$ cd dist
$ ls
$ sudo -H pip3 install torch-<version>-cp36-cp36m-linux_aarch64.whl
```

成功安装之后，您可以使用上一节末尾给出的命令检查PyTorch。

![none](https://qengineering.eu/images/Torch_1_10_0_Jetson_Succes.webp)

说说OpenCV。PyTorch可以选择使用OpenCV。然而，它与构建过程中发现的OpenCV版本硬编码链接。一旦你升级你的OpenCV, PyTorch将停止工作，因为它不能找到旧的OpenCV版本。鉴于OpenCV每年至少发布两到三个版本，将PyTorch与OpenCV联系起来似乎并不明智。否则，您将被迫重新编译PyTorch或手动创建一大堆到旧库的符号链接。

成功安装之后，许多文件就不再需要了。删除它们将为您提供大约3.6 GB的磁盘空间。

请确保删除本手册开头安装的SD内存交换软件。

```shell
# remove the dphys-swapfile now
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
# just a tip to save some space
$ sudo rm -rf ~/pytorch
```

# 4.**TorchVision**

## 4.1 在Jetson Nano上安装TorchVision

Torchvision是一个常用的数据集、架构和图像算法的集合。当你使用我们在GitHub上找到的轮子时，安装是简单的。Torchvision假设PyTorch已经安装在你的机器上。

（1）Vison0.13.0

```shell
Used with PyTorch 1.12.0

Only for a Jetson Nano with Ubuntu 20.04

# the dependencies
$ sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev
$ sudo pip3 install -U pillow
# install gdown to download from Google drive, if not done yet
$ sudo -H pip3 install gdown
# download TorchVision 0.13.0
$ gdown https://drive.google.com/uc?id=11DPKcWzLjZa5kRXRodRJ3t9md0EMydhj
# install TorchVision 0.13.0
$ sudo -H pip3 install torchvision-0.13.0a0+da3794e-cp38-cp38-linux_aarch64.whl
# clean up
$ rm torchvision-0.13.0a0+da3794e-cp38-cp38-linux_aarch64.whl
```

（2）Vison0.12.0

```shell
Used with PyTorch 1.11.0

Only for a Jetson Nano with Ubuntu 20.04

# the dependencies
$ sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev
$ sudo pip3 install -U pillow
# install gdown to download from Google drive, if not done yet
$ sudo -H pip3 install gdown
# download TorchVision 0.12.0
$ gdown https://drive.google.com/uc?id=1BaBhpAizP33SV_34-l3es9MOEFhhS1i2
# install TorchVision 0.12.0
$ sudo -H pip3 install torchvision-0.12.0a0+9b5a3fe-cp38-cp38-linux_aarch64.whl
# clean up
$ rm torchvision-0.12.0a0+9b5a3fe-cp38-cp38-linux_aarch64.whl
```

（3）Vison0.11.0

```shell
Used with PyTorch 1.10.0
# the dependencies
$ sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev
$ sudo pip3 install -U pillow
# install gdown to download from Google drive, if not done yet
$ sudo -H pip3 install gdown
# download TorchVision 0.11.0
$ gdown https://drive.google.com/uc?id=1C7y6VSIBkmL2RQnVy8xF9cAnrrpJiJ-K
# install TorchVision 0.11.0
$ sudo -H pip3 install torchvision-0.11.0a0+fa347eb-cp36-cp36m-linux_aarch64.whl
# clean up
$ rm torchvision-0.11.0a0+fa347eb-cp36-cp36m-linux_aarch64.whl
```

（4）Vison0.10.0

```shell
Used with PyTorch 1.9.0
# the dependencies
$ sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev
$ sudo pip3 install -U pillow
# install gdown to download from Google drive, if not done yet
$ sudo -H pip3 install gdown
# download TorchVision 0.10.0
$ gdown https://drive.google.com/uc?id=1Q2NKBs2mqkk5puFmOX_pF40yp7t-eZ32
# install TorchVision 0.10.0
$ sudo -H pip3 install torchvision-0.10.0a0+300a8a4-cp36-cp36m-linux_aarch64.whl
# clean up
$ rm torchvision-0.10.0a0+300a8a4-cp36-cp36m-linux_aarch64.whl
```

（5）Vison0.9.0

```SHELL
Used with PyTorch 1.8.0
# the dependencies
$ sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev
$ sudo pip3 install -U pillow
# install gdown to download from Google drive, if not done yet
$ sudo -H pip3 install gdown
# download TorchVision 0.9.0
$ gdown https://drive.google.com/uc?id=1BdvXkwUGGTTamM17Io4kkjIT6zgvf4BJ
# install TorchVision 0.9.0
$ sudo -H pip3 install torchvision-0.9.0a0+01dfa8e-cp36-cp36m-linux_aarch64.whl
# clean up
$ rm torchvision-0.9.0a0+01dfa8e-cp36-cp36m-linux_aarch64.whl
```

（6）Vison0.8.0

```shell
Used with PyTorch 1.7.0
# the dependencies
$ sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev
$ sudo pip3 install -U pillow
# install gdown to download from Google drive, if not done yet
$ sudo -H pip3 install gdown
# download TorchVision 0.8.0
$ gdown https://drive.google.com/uc?id=1P0xyPT-WIWglqmT195OSyazV_1LPaHDa
# install TorchVision 0.8.0
$ sudo -H pip3 install torchvision-0.8.0a0+291f7e2-cp36-cp36m-linux_aarch64.whl
# clean up
$ rm torchvision-0.8.0a0+291f7e2-cp36-cp36m-linux_aarch64.whl
```

安装后，您可能想通过验证发布版本来检查torchvision。

![TV_0_10_0_Success](https://qengineering.eu/images/TV_0_10_0_Succes.webp)



你也可以从头开始构建`torchvision`。在这种情况下，你必须从官方GitHub页面下载你选择的版本，在`version.txt`中修改版本号，并发出命令

```shell
$ python3 setup.py bdist_wheel
```

Bergen Robotics AS的Brett Ryland给我们发了一封邮件，指出CUDA代码中的一个常量需要更改，以处理可变形卷积。用编辑器打开`deform_conv2d_kernel.cu`文件并且降低线程数，如下所示。

![none](https://qengineering.eu/images/VisionCuda.webp)

# 5.LibTorch

## 5.1 在Jetson Nano上安装LibTorch

PyTorch的原生语言是Python。这是有原因的。

人工智能科学家希望塑造他们的深度学习模型，并分析结果，而不需要专门的软件编程。

Python非常适合这项工作。大多数人可以在几周内理解和修改一个Python程序，而掌握(低级)C++语言则需要数年时间。如果您是深度学习和PyTorch的新手，我们强烈建议您使用Python。

在开始C++冒险之前，还有很多事情需要了解。

+ 预编译的`LibTorch`只适用于`x86_64`机器。没有`aarch64`版本，我们必须从头开始构建它。
+ C++文档的质量很低。您对函数调用有一个简短的解释，但是缺少关于安装LibTorch以及出现错误时如何处理的良好指南。(顺便说一下，大多数框架都同样缺乏文档。)
+ 在Jetson Nano上编译时间非常长。稍后将显示一个简单的`example.cpp`，其构建时间大约为一分钟。如果您非常重视C++编程，那么最好使用交叉编译技术。等待超过一分钟来纠正一个简单的拼写错误是令人沮丧的。
+ PyTorch的核心是用一个旧的GCC编译器构建的，而不是使用2011年的C++字符串命名约定。要使用静态构建LibTorch，必须使用宏`_GLIBCXX_USE_CXX11_ABI`，这可能与C++程序的其他部分发生冲突。如果你想使用LibTorch，最好使用动态构建版本。它可以顺利地与其他C++包(如ROS)一起工作。
+ 经过三天的辛苦工作，我们仍然无法让静态构建LibTorch在Jetson Nano上工作。通过浏览论坛，你会看到相同的问题和相同的(糟糕的)解决方案。
+ ARM核的NEON寄存器的本征的GCC优化存在问题。它迫使我们使用clang编译器。因此，在使用LibTorch API时，您可能也会被迫使用clang编译器。

现在，让我们开始构建LibTorch C++ API。

在Jetson Nano上安装LibTorch有两种可能的方法。第一种方法是从我们的[GitHub](https://github.com/Qengineering/PyTorch-Jetson-Nano)下载tar.xz文件并提取它。安装了所有必要的库和头文件，如下面的截图所示。

![LibTorchTree](https://qengineering.eu/images/TreeLibTorchNano.webp)

这些文件放在名为pytorch的文件夹中。为了避免冲突，请确保在要解压缩tar.gz文件的目录中没有同名文件夹。该文件的结构与原来的`libtorch-cxx11-abi-share -with-deps-1.10.1+cu102.zip`相同，可以在[PyTorch安装页面](https://pytorch.org/get-started/locally/)找到。

（1）LibTorch1.12.0

```shell
Only for a Jetson Nano with Ubuntu 20.04

# install gdown to download from Google drive, if not done yet
$ sudo -H pip3 install gdown
# download LibTorch 1.12.0
$ gdown https://drive.google.com/uc?id=1t0MM1Bec2XIIKK8PhQbEDOrM1Z2Xym5-
# unpack the LibTorch 1.12.0 tar ball
$ sudo tar -xf libtorch-1.12.0-Jetson-aarch64-GPU.tar.gz
# clean up
$ rm libtorch-1.12.0-Jetson-aarch64-GPU.tar.gz
```

（2）LibTorch1.11.0

```shell
Only for a Jetson Nano with Ubuntu 20.04

# install gdown to download from Google drive, if not done yet
$ sudo -H pip3 install gdown
# download LibTorch 1.11.0
$ gdown https://drive.google.com/uc?id=1OSWB_Wv7rghpiBI3V9Rvj0ZR6bRcAZsY
# unpack the LibTorch 1.11.0 tar ball
$ sudo tar -xf libtorch-1.11.0-Jetson-aarch64-GPU.tar.gz
# clean up
$ rm libtorch-1.11.0-Jetson-aarch64-GPU.tar.gz
```

（3）LibTorch1.10.0

```shell
# install gdown to download from Google drive, if not done yet
$ sudo -H pip3 install gdown
# download LibTorch 1.10.0
$ gdown https://drive.google.com/uc?id=1izv6kmcnqXk9i7-Ey-vldjC-CGfHOGCl
# unpack the LibTorch 1.10.0 tar ball
$ sudo tar -xf libtorch-1.10.0-Jetson-aarch64-GPU.tar.gz
# clean up
$ rm libtorch-1.10.0-Jetson-aarch64-GPU.tar.gz
```

（4）LibTorch1.9.0

```shell
# install gdown to download from Google drive, if not done yet
$ sudo -H pip3 install gdown
# copy binairy
$ gdown https://drive.google.com/uc?id=1E4Hfz1cj6bwGz8a72OS5uH3SnlvRyrOi
# unpack the LibTorch 1.9.0 tar ball
$ sudo tar -xf libtorch-1.9.0-Jetson-aarch64-GPU.tar.gz
# clean up
$ rm libtorch-1.9.0-Jetson-aarch64-GPU.tar.gz
```

另一种方法是从头编译`LibTorch C++ API`。整个过程几乎与最初的Python安装相同。如果希望从头编译和安装库，请按照说明进行操作。如果需要静态库(`libtorch.a`)，请设置环境标志`BUILD_SHARED_LIBS=OFF`。如前所述，我们无法让静态库在Nano上工作。最好使用动态库(`libtorch.so`)。

```shell
# First, download and install the dependencies and
# your PyTorch version of your choice as specified above.
# Follow all steps up until the environment variables.
# Don't forget to modify the five files also.

$ cd ~/pytorch
$ mkdir build_libtorch
$ cd build_libtorch
# now set the temporary environment variables for LibTorch
# remember, don't close the window as it will delete these variables
$ export BUILD_PYTHON=OFF
$ export BUILD_CAFFE2_OPS=OFF
$ export USE_FBGEMM=OFF
$ export USE_FAKELOWP=OFF
$ export BUILD_TEST=OFF
$ export USE_MKLDNN=OFF
$ export USE_NNPACK=OFF
$ export USE_XNNPACK=OFF
$ export USE_QNNPACK=OFF
$ export USE_PYTORCH_QNNPACK=OFF
$ export USE_CUDA=ON
$ export USE_CUDNN=ON
$ export TORCH_CUDA_ARCH_LIST="5.3;6.2;7.2"
$ export MAX_JOBS=4
$ export USE_NCCL=OFF
$ export USE_OPENCV=OFF
$ export USE_SYSTEM_NCCL=OFF
$ export BUILD_SHARED_LIBS=ON
$ PATH=/usr/lib/ccache:$PATH
# set the compilers
$ export CC=clang
$ export CXX=clang++
$ export CUDACXX=/usr/local/cuda/bin/nvcc
# create symlink to cublas (if not done yet)
$ sudo ln -s /usr/lib/aarch64-linux-gnu/libcublas.so /usr/local/cuda/lib64/libcublas.so
# clean up the previous build, if necessary
$ python3 setup.py clean
# start the build
$ python3 ./tools/build_libtorch.py
```

![libTorchSuccessNano](https://qengineering.eu/images/libTorchSuccessNano.webp)



构建完成后，您可能需要剥离`~/pytorch`目录。这将节省大量的磁盘空间。你需要的唯一文件夹是`~pytorch/torch`文件夹。在这个目录中，您可以删除除`bin、include、lib和share`文件夹之外的所有内容。别忘了还有很多隐藏的文件。最终得到的结构与上面tar.gz安装中所示的相同。

![LibTorchFolder](https://qengineering.eu/images/TorchFolder.webp)

## 5.2 example-app.cpp

是时候使用来自[PyTorch C++站点](https://pytorch.org/cppdocs/installing.html)的著名示例应用程序来测试LibTorch安装了。

```c++
#include <iostream>
#include <torch/torch.h>

using namespace std;

int main()
{
    torch::Tensor tensor = torch::rand({2, 3});
    cout << tensor << endl;
    cout << torch::hypot(torch::tensor(1.),torch::tensor(1.))<< endl;

    auto t = torch::ones({3, 3}, torch::dtype(torch::kFloat32));
    cout << "t:\n" << t << endl;
    cout << "t.exp():\n" << t.exp() << endl;

    return 0;
}
```

我们在同一个PyTorch页面上使用CMake文件。我们只剥离了Windows MSVC分支，因为在Linux环境中不需要它。

将该文件保存为CMakeLists.txt，保存在与放置示例app.cpp的文件夹相同的文件夹中。

```bash
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(example-app)

set(CMAKE_C_COMPILER clang)
set(CMAKE_CXX_COMPILER clang++)

find_package(Torch REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

add_executable(example-app example-app.cpp)
target_link_libraries(example-app "${TORCH_LIBRARIES}")
set_property(TARGET example-app PROPERTY CXX_STANDARD 14)
```

然后创建一个名为`build`的文件夹，并在该目录中使用以下命令创建应用程序。

```shell
# make the build folder
$ mkdir build
$ cd build
$ cmake -D CMAKE_PREFIX_PATH=/home/pi/pytorch ..    # 此处的需要自己的安装路径CMAKE_PREFIX_PATH=/home/xxx/pytorch
$ cmake --build . --config Release
```

![LibTorchSuccesNano](https://qengineering.eu/images/LibTorchSuccesNano.webp)

# 6. Caffe2

## 6.1 在Jetson Nano上安装Caffe2

PyTorch自带Caffe2 。换句话说，如果您安装了PyTorch，那么您也在Jetson Nano上安装了支持CUDA的caff2。连同两个转换工具。在使用Caffe2之前，`protobuf`大部分时间都需要更新。我们现在就做吧。

```shell
# update protobuf (3.15.5)
$ sudo -H pip3 install -U protobuf
```

您可以使用一些Python指令检查caff2的安装情况。

![Caffe2_Nano](https://qengineering.eu/images/caffe2_succes.webp)

# 参考目录

[https://qengineering.eu/install-pytorch-on-jetson-nano.html](https://qengineering.eu/install-pytorch-on-jetson-nano.html)