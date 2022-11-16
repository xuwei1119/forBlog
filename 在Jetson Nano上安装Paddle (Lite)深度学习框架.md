# 在Jetson Nano上安装Paddle (Lite)深度学习框架

[TOC]

# 1.简介

本页面将指导您在Jetson Nano上安装百度的Paddle-Lite框架。

PaddlePaddle是中国版的TensorFlow。它广泛应用于工业、机构和大学。它支持多种硬件环境，包括使用Rockchip RK3399的NPU加速，在许多单板计算机上都可以找到。软件加速可以通过CUDA和cuDNN库完成。有关Paddle库的更多信息，请参见GitHub页面或中文教程https://paddle-lite.readthedocs.io/zh/latest/。

# 2. Paddle-Lite安装

我们将安装Paddle-Lite 2.7.0版本，因为它支持cuDNN 8.0，可以在Jetson Nano的Jetpack 4.5中找到。它使Paddle-Lite和MNN成为支持CUDA和cuDNN的小型设备仅有的两个Lite框架。所有其他框架要么没有GPU加速，要么有某种形式的Vulkan支持，比如ncnn。

注意，这是一个C++安装，不适合Python。Paddle-Lite的Python接口依赖于PaddlePaddle框架。因为我们不希望仅为Python接口使用额外的3GByte磁盘空间，所以我们将不使用它。而且，正如您所知，速度和Python并不是齐头并进的。

Paddle-Lite框架几乎没有依赖关系。所需的库会在安装期间自动下载和编译。如果您希望OpenCV也支持CUDA，请先重新安装OpenCV，大约需要一个半小时。这不是强制性的。最新版本的Paddle-Lite在Jetson Nano上的全部安装如下。

```shell
# check for updates
$ sudo apt-get update
$ sudo apt-get upgrade
# install dependencies
$ sudo apt-get install cmake wget
$ sudo apt-get install patchelf
# download Paddle Lite
$ git clone https://github.com/PaddlePaddle/Paddle-Lite.git
$ cd Paddle-Lite
# build Paddle Lite (± 2 Hours)
$ ./lite/tools/build.sh \
  --build_cv=ON \
  --build_extra=ON \
  --arm_os=armlinux \
  --arm_abi=armv8 \
  --arm_lang=gcc \
  cuda \
  tiny_publish
# copy the headers and library to /usr/local/
$ sudo mkdir -p /usr/local/include/paddle-lite
$ sudo cp -r build.lite.armlinux.armv8.gcc/inference_lite_lib.armlinux.armv8/cxx/include/*.* /usr/local/include/paddle-lite

$ sudo mkdir -p /usr/local/lib/paddle-lite
$ sudo cp -r build.lite.armlinux.armv8.gcc/inference_lite_lib.armlinux.armv8/cxx/lib/*.* /usr/local/lib/paddle-lite
```

两个小时后编译完成。它占用您磁盘上大约7.2 GByte的空间。

![PaddleLiteJetsonHeaders](https://qengineering.eu/images/PaddleLiteJetsonHeaders.webp)

![PaddleLiteJetsonLib](https://qengineering.eu/images/PaddleLiteJetsonLib.webp)

也请注意包含示例的文件夹。

![PaddleLiteJetsonExamples](https://qengineering.eu/images/PaddleLiteJetsonExamples.webp)

在将头文件和库复制到/usr/local/文件夹后，您可能想要删除整个Paddle-Lite文件夹。这个操作将释放大约7GByte的SD卡。请记住，您还删除了示例。如果需要的话，可以在GitHub上找到。一切都取决于你。

```shell
$ sudo rm -rf Paddle-Lite
```

# 3.OpenCV的依赖

首先，你需要在你的Jetson Nano上安装OpenCV。Paddle需要OpenCV进行图像渲染。它将从它的轮中安装opencv-python <= 4.2.0.32。除了众所周知的OpenCV pip安装令人沮丧之外，该版本在任何pypi和piwheels数据库中都不可用，因此退回到3.4.11.45版本，你不想要这么做，因为它有大量过时的依赖。

只要让事情简单就好。检查Python 3导入的OpenCV版本。如果需要，根据我们的指南安装一个更新的版本。

![OpenCV_Success_Nano](https://qengineering.eu/images/Succes_4_5_Jetson.webp)

我们甚至从轮子上删除了OpenCV要求，只是为了防止在Nano上安装两个版本。记住，SD卡上的空间也很宝贵。

# 4.Paddle pip3安装

在Jetson Nano上安装Paddle的最容易和最可靠安装方法是使用pip3安装。当然，如果您计划在Python3中使用Paddle。官方的Paddle[安装](https://www.paddlepaddle.org.cn/documentation/docs/en/2.0-alpha/install/install_Ubuntu_en.html)不支持AARCH64。这就是为什么我们在GitHub页面上放了一个轮子，使安装更容易。按照下面的说明操作。顺便说一下，如果需要的话，构建scipy库可能需要一段时间。

```shell
# a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# install dependencies
$ sudo apt-get install cmake wget
$ sudo apt-get install libatlas-base-dev libopenblas-dev libblas-dev
$ sudo apt-get install liblapack-dev patchelf gfortran
$ sudo -H pip3 install Cython
$ sudo -H pip3 install -U setuptools
$ pip3 install six requests wheel pyyaml
# upgrade version 3.0.0 -> 3.13.0
$ pip3 install -U protobuf
# download the wheel
$ wget https://github.com/Qengineering/Paddle-Jetson-Nano/raw/main/paddlepaddle_gpu-2.0.0-cp36-cp36m-linux_aarch64.whl
# install Paddle
$ sudo -H pip3 install paddlepaddle_gpu-2.0.0-cp36-cp36m-linux_aarch64.whl
# clean up
$ rm paddlepaddle_gpu-2.0.0-cp36-cp36m-linux_aarch64.whl
```

如果一切正常，您将得到以下屏幕。

![echo_paddle_nano](https://qengineering.eu/images/echo_paddle_nano.webp)

可能会出现这样的情况:安装被关闭时，会发出关于PATH的警告。如果是这样，在**.bashrc**文件的末尾添加位置(**~/.local/bin**)，使用**$ sudo nano .bashrc**，如下所示。

![bashrc_Paddle_Nano](https://qengineering.eu/images/bashrc_Paddle_Nano.webp)

您可以检查您Paddle安装如下。

![Paddle_Test_Succes](https://qengineering.eu/images/Paddle_Test_Succes.webp)



# 5.源码安装Paddle

如果不想使用Python轮，或者需要C++ API推理库，可以在Jetson Nano上从头构建Paddle深度学习框架。整个过程大约需要5个小时，将使用大约20GByte的磁盘。显然，至少需要一张64GB的SD卡。通常，第一步是安装依赖项。完整的列表如下所示。

```shell
# a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# install dependencies
$ sudo apt-get install cmake wget curl unrar swig
$ sudo apt-get install libjpeg8-dev libpng-dev libfreetype6-dev
$ sudo apt-get install libatlas-base-dev libopenblas-dev libblas-dev
$ sudo apt-get install liblapack-dev patchelf gfortran
$ sudo -H pip3 install Cython
$ sudo -H pip3 install -U setuptools
$ sudo -H pip3 install six requests wheel pyyaml
$ sudo -H pip3 install numpy
# upgrade version 3.0.0 -> 3.13.0
$ sudo -H pip3 install -U protobuf
$ sudo -H pip3 install matplotlib
$ sudo -H pip3 install nltk
# scipy may take a while to compile
$ sudo -H pip3 install scipy
$ sudo -H pip3 install graphviz funcsigs
$ sudo -H pip3 install decorator precd ttytable
$ sudo -H pip3 install pillow
```

在Jetson Nano上安装Paddle需要大量的内存。为了满足这个需求，使用dphysi -swapfile工具创建了一些额外的交换空间。安装完成后，工具将被移除。执行以下命令安装dphysi -swapfile。

```shell
# install dphys-swapfile
$ sudo apt-get install dphys-swapfile
# give the required memory size
$ sudo nano /etc/dphys-swapfile
# reboot afterwards
$ sudo reboot.
```

![2 GB swap space](https://qengineering.eu/images/SwapJetson.webp)

如果一切顺利，你应该会得到如下内容。

![4 GB swap memory](https://qengineering.eu/images/FreeJetsonSwap.webp)

为了记录，所示的图是由dphysi -swapfile和zram分配的交换空间总量。完成后不要忘记删除dphysics -swapfile。

## 5.1 nccl

下一步是在Jetson Nano上安装Nvidia的nccl包。当使用CUDA后端时，PaddlePaddle需要这个gpu间通信库。您可以在下面的说明中看到，安装只能从头开始。

```shell
$ cd ~
# download nccl
$ git clone https://github.com/NVIDIA/nccl.git
$ cd nccl
# compile (± 30 min)
$ make -j4
# install system wide
$ sudo make install
```

![nccl build rdy](https://qengineering.eu/images/nccl-build-rdy.webp)

所有依赖项就绪后，现在可以下载Paddle包了。在我们的GitHub页面上，我们发布了一个专为Jetson Nano (aarch64)改编的包。您可以通过比较按钮探索与原始软件的区别。

![GitHub Compare](https://qengineering.eu/images/GitHubCompare.png)

从概述中可以看出，这里做了三个修改。首先，已经提到的，在需求中包含OpenCV被删除了。第二，要让PaddleHub正常工作，版本号不应该是0.0.0。这就是为什么目前版本2.0.0是硬编码的。将来，当这个版本被标记为稳定时，将不再需要它，因为版本例程将生成一个可操作的数字。最后，OpenBLAS还需要cmake文件(cmake/external/ OpenBLAS .cmake)没有给出的额外指令。我们已经插入了。

```shell
# get Paddle
$ git clone https://github.com/Qengineering/Paddle.git
$ cd Paddle
# get a working directory
$ mkdir build
$ cd build
# run cmake
$ cmake -D PY_VERSION=3.6 \
        -D WITH_CONTRIB=OFF \
        -D WITH_MKL=OFF \
        -D WITH_MKLDNN=OFF \
        -D WITH_NV_JETSON=ON \
        -D WITH_XBYAK=OFF \
        -D WITH_GPU=ON \
        -D WITH_TESTING=OFF \
        -D ON_INFER=ON \
        -D CMAKE_CUDA_COMPILER=/usr/local/cuda/bin/nvcc \
        -D CMAKE_BUILD_TYPE=Release ..
```

正如您在上面的参数列表中看到的，将生成以下两个，一个Python轮和一个推理库。

![Paddle cmake rdy jetson](https://qengineering.eu/images/Paddle-cmake-rdy-jetson.webp)

在我们开始构建之前，同时打开的文件数量需要从默认的1024增加到2048。这是通过**ulimit**命令完成的。一旦make被调用，只有一个核心(**$ make -j1**)。即使有两个，由于令人窒息的内存交换，您最终也会崩溃。

```shell
# increase the number of open files
$ ulimit -n 2048
# run with one core
$ make -j1
```

![PAddle build Rdy Jetson](https://qengineering.eu/images/PAddle-build-Rdy-Jetson.webp)

在这里找到你的Python安装轮。它和我们在GitHub页面上的是一样的。

![LocPaddleJetson](https://qengineering.eu/images/Location-Paddle-Jetson2.webp)

现在可以安装Python Paddle版本。如果必须安装**dphys-swapfile**服务，那么现在应该删除它。这样可以延长SD卡的使用寿命。

```shell
# install Paddle Python
$ sudo -H pip3 install python/dist/paddlepaddle_gpu-2.0.0-cp36-cp36m-linux_aarch64.whl
# remove the dphys-swapfile
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
```

![pip3_paddle_gpu_ok](https://qengineering.eu/images/pip3_paddle_gpu_ok.webp)

您可以使用以下命令测试您的安装。

![Paddle_Test_Succes2](https://qengineering.eu/images/Paddle_Test_Succes.webp)

## 5.2 Paddle C++ API 库

安装Paddle C++ API库的步骤与安装Python版本的步骤完全相同。如果设置了参数`-D ON_INFER=ON`，则库位于以下位置。

![Paddle_API_lib_Nano](https://qengineering.eu/images/Paddle_API_lib_Nano.webp)

## 5.3 PaddleHub

PaddleHub是一个脚本文件的集合，在您的Jetson Nano上一键加载和部署预先训练的Paddle模型，正如我们将看到的。PaddleHub使用Paddle框架，因此必须先安装它。显然，您必须有一个工作的互联网连接访问Paddle服务器。

安装与树莓派4完全相同。请按照那一段的[说明](https://qengineering.eu/install-paddlepaddle-on-raspberry-pi-4.html#Paddle_Hub)去做。

安装只需一个pip就可以完成。然而，正如本指南前面所描述的，您将安装OpenCV3版本，它有许多不兼容的依赖项。最好是逐个安装所有依赖项和PaddleHub。

我们假设您已经为您的Paddle框架安装了OpenCV，因此不需要在这里再次安装它。

最好检查依赖项列表，以确保它仍然是最新版本的PaddleHub。requirements.txt位于GitHub页面的根目录。

PaddleHub支持多种深度学习模型。所有这些都附带一个小小的Python脚本，其中包括Paddle框架。代码自然也依赖于其他库。有些库一直在使用，比如protobuf。其他的只能由特定的深度学习模型调用，例如easydict。下面给出了完整的列表，这是我们从代码中所能得到的。这取决于您想要安装哪个库。将来当库丢失时，PaddleHub脚本将通知您。你可以在之后安装它。顺便说一下，鉴于更新速度很快，这个列表很快就会过时。它迫使您在运行脚本时安装缺失的依赖项。

```shell
# the dependencies
$ sudo apt-get install python3-matplotlib
$ pip3 install --user --upgrade pip
$ pip3 install packaging
$ pip3 install pre-commit
$ pip3 install protobuf
$ pip3 install yapf
$ pip3 install six
$ pip3 install flask
$ pip3 install flake8
$ pip3 install visualdl
$ pip3 install cma
$ pip3 install sentencepiece
$ pip3 install filelock
$ pip3 install easydict
$ pip3 install gitpython
$ pip3 install pyyaml
$ pip3 install pyzmq
$ pip3 install colorama
$ pip3 install colorlog
$ pip3 install tqdm
$ pip3 install nltk
$ pip3 install pandas
$ pip3 install gunicorn
$ pip3 install paddlenlp
```

在安装各种软件包的过程中，pip3抱怨缺少OpenCV。有时它甚至被称为错误。不要试图用pip安装OpenCV !只要您按照我们的指南安装了OpenCV，就不会出现任何问题。Python3将导入OpenCV。只有pip3数据库中缺失的条目才会生成这类警告。记住，在一台机器上使用两个不同版本的OpenCV会导致问题!

安装了大部分依赖项之后，现在是时候安装最新版本的PaddleHub了。在撰写本文时(2021年1月)，标准的pip3安装将为您提供1.8.3版本。它不能与新安装的PaddlePaddle 2.0.0版本完美地工作。您的PaddleHub至少需要2.0.0版本。最好的方法是提供一个不存在的数字，看看最新的版本是什么。然后就可以安装这个版本了。下面的屏幕显示了它的工作原理。

![Pip3_Latest_version](https://qengineering.eu/images/Pip3_Hub_Jetson.webp)

如果这种方法不起作用，你总是可以在GitHub PaddleHub页面上找到最新的版本号。

![none](https://qengineering.eu/images/PaddleHubVersion.webp)

## 5.4 转换为Paddle Lite

本节描述常规的，所谓的液体模型的转换，从PaddlePaddle框架到用于嵌入式系统(如树莓派4或Jetson Nano)的Paddle Lite模型。

与TensorFlow Lite一样，并非所有模型都可以移植到Lite框架，部分操作不支持。转换工具会让你知道情况是否如此。在构建转换工具之前，需要做一些准备工作。首先，我们需要至少6 GB的RAM，就像从零开始构建PaddlePaddle一样。由于程序与树莓派几乎相同，我们将使用一些树莓派屏幕内容。

![LessMemory](https://qengineering.eu/images/LessMemory.webp)



接下来，应该禁用`-m64`标志，因为aarch64系统不识别这个标志。

![M64error](https://qengineering.eu/images/M64error.webp)

`-m64`标志在flags中声明。Cmake在151行。该文件位于`~/Paddle-Lite/cmake`文件夹中。用文本`AND NOT(CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64.*|AARCH64.*)")`扩展第151行，如下所示。它有效地禁用了aarch64机器不知道的`-m64`标志。

![none](https://qengineering.eu/images/flags_cmake.webp)

完成这些步骤后，您现在可以开始编译Paddle Lite转换工具了。我们假设您已经成功构建了如上所述的Paddle Lite框架。转换工具的构建如下所示。

```shell
$ cd ~/Paddle-Lite
# build 64-bit Paddle Lite optimize tool
$ ./lite/tools/build.sh \
 --build_cv=ON \
 --build_extra=ON \
 --arm_os=armlinux \
 --arm_abi=armv8 \
 --arm_lang=gcc \
 cuda \
 build_optimize_tool
```

![Paddle_conversion_Nano](https://qengineering.eu/images/Paddle_conversion_Nano.webp)

有了优化器之后，现在就可以使用PaddleHub下载您感兴趣的深度学习模型了。作为一个例子，我们将使用口罩检测器。

这种被PaddlePaddle称为深度学习模型的液体模型有两种变体。要么它是一个带有参数(拓扑)和权重的组合模型，要么参数和权重分别存储在一个单独的文件中。优化器处理这两种类型。碰巧的是，口罩探测器有两个文件。

所以下一步将从网络中提取`__model__`和`__param__`。它是通过一些Python指令完成的。如你所见，我们同时使用PaddlePaddle和PaddleHub。

```shell
$ python3
>>> import paddlehub as hub 
>>> import paddle

>>> paddle.enable_static() 
>>> pyramidbox_lite_mobile_mask = hub.Module(name="pyramidbox_lite_mobile_mask") 
>>> pyramidbox_lite_mobile_mask.save_inference_model(dirname="test_program")
```

![Location test_program](https://qengineering.eu/images/Location-test_program.webp)

最后一个操作是将模型和参数通过刚刚构建的opt应用程序转换为一个Paddle Lite版本

这可以通过以下命令完成。

```shell
$ ./opt \
--model_file=/home/pi/test_program/mask_detector/__model__ \
--param_file=/home/pi/test_program/mask_detector/__params__ \
--valid_targets=arm \
--optimize_out_type=naive_buffer \
--optimize_out=opt_model_v2.7
```

![PaddleLiteOutput](https://qengineering.eu/images/PaddleLiteOutput.webp)

请注意版本号V2.7。大多数模型既不向后也不向前兼容。

关于优化工具的更多信息可以在wiki https://github.com/PaddlePaddle/Paddle-Lite/wiki/model_optimize_tool中找到。

如果必须重新安装dphysi -swapfile，那么是时候重新卸载它了。这样可以延长SD卡的使用寿命。

```shell
# remove the dphys-swapfile (if installed)
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
```

# 参考目录

[https://qengineering.eu/install-paddle-on-jetson-nano.html](https://qengineering.eu/install-paddle-on-jetson-nano.html)