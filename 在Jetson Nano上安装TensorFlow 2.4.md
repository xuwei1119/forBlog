# 在Jetson Nano上安装TensorFlow 2.4

TensorFlow 2.5.0于2021年5月12日发布。TensorFlow 2.5.0依赖于CUDA 11.0和cuDNN 8.0.4版本，这两个版本还没有用于Jetson Nano。解决方案很麻烦，而且可能不太可靠。最好等待新的JetPack与CUDA和cuDNN所需版本一起发布。现在继续使用TensorFlow 2.4.1。

# 1.简介

本指南将指导您在Jetson Nano上安装TensorFlow 2.4.0或TensorFlow 2.4.1。

TensorFlow是一个专门为深度学习开发的大型软件库。它消耗大量的资源。你可以在Jetson Nano上执行TensorFlow，但不要期待奇迹。如果不是太复杂的话，它可以运行您的模型，但它将不能训练新的模型。它也不能进行所谓的迁移学习。除了运行你预先构建的深度学习模型，你可以使用这个库将冻结的TensorFlow模型转换为TensorFlow Lite模型。

如果你只想对深度学习有一些印象，请考虑安装TensorFlow Lite。它速度快得多，使用的资源也少得多，因为它是为手机和**Jetson Nano**等小型电脑设计的。您可以使用许多现成的构建模型。在[这里](https://qengineering.eu/install-tensorflow-2-lite-on-jetson-nano.html)查看我们的安装指南。

我们将讨论两个安装，一个用于Python 3，另一个用于c++ API库。

# 2. NUmpy

在撰写本文时(2020年12月)，我们正在经历Numpy的一些问题。当从零开始构建TensorFlow时，我们会遇到**'// TensorFlow /python:bfloat16_lib'**失败的错误。它与CUDA结合使用最新的Numpy版本有关，如GitHub ticket [#40688](https://github.com/tensorflow/tensorflow/issues/40688)所示。

我们强烈建议使用**pip3 list | grep Numpy**检查您的Numpy版本。如果您安装的版本高于1.18.5，请使用**pip3 uninstall numpy && pip3 install numpy==1.18.5**降级。

![none](https://qengineering.eu/images/Numpy.webp)

顺便说一下，只有在使用CUDA支持时才会出现问题。如果你构建的TensorFlow没有CUDA后端，那么最新版本的Numpy(1.19.4)就可以很好地工作。

在您的Jetson Nano上安装其他软件包时也要记住这一点。无意中，他们可能会将您的Numpy升级到后续版本。

# 3.快捷方式

TensorFlow是由一个叫做Bazel的谷歌软件安装程序安装的。最后，Bazel生成一个轮子来安装TensorFlow Python版本，或者在安装c++版本时生成一个tarball。我们已经在GitHub页面上发布了Bazel的结果。请随意使用这些快捷方式。整个TensorFlow安装过程从头到尾需要很多小时(Python为±50小时，c++库为±10小时，超频至1900 MHz)。尽管已经完成了所有繁琐的工作，在Nano上安装TensorFlow 2.4.0仍然需要1小时20分钟。

## 3.1 TensorFlow 2.4 for Python 3

整个快捷程序如下所示。轮子太大，无法存储在GitHub中，所以使用谷歌驱动器。请确保您安装了最新的pip3和python3版本，否则，pip可能会附带消息”WHL在这个平台上不是一个受支持的轮子”。安装将需要大约2.1 GByte的磁盘空间。

JetPack 4附带Python 3.6.9。毫无疑问，Python版本会随着时间的推移而升级，您将需要一个不同的轮子。查看GitHub页面的所有轮子。

![Python version check](https://qengineering.eu/images/Python3Jetson.webp)

（1）TensorFlow 2.4.1

```shell
# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# install pip and pip3
$ sudo apt-get install python-pip python3-pip
# remove old versions, if not placed in a virtual environment (let pip search for them)
$ sudo pip uninstall tensorflow
$ sudo pip3 uninstall tensorflow
# install the dependencies (if not already onboard)
$ sudo apt-get install gfortran
$ sudo apt-get install libhdf5-dev libc-ares-dev libeigen3-dev
$ sudo apt-get install libatlas-base-dev libopenblas-dev libblas-dev
$ sudo apt-get install liblapack-dev
$ sudo -H pip3 install Cython==0.29.21
# install h5py with Cython version 0.29.21 (± 6 min @1950 MHz)
$ sudo -H pip3 install h5py==2.10.0
$ sudo -H pip3 install -U testresources numpy
# upgrade setuptools 39.0.1 -> 53.0.0
$ sudo -H pip3 install --upgrade setuptools
$ sudo -H pip3 install pybind11 protobuf google-pasta
$ sudo -H pip3 install -U six mock wheel requests gast
$ sudo -H pip3 install keras_applications --no-deps
$ sudo -H pip3 install keras_preprocessing --no-deps
# install gdown to download from Google drive
$ sudo -H pip3 install gdown
# download the wheel
$ gdown https://drive.google.com/uc?id=1DLk4Tjs8Mjg919NkDnYg02zEnbbCAzOz
# install TensorFlow (± 12 min @1500 MHz)
$ sudo -H pip3 install tensorflow-2.4.1-cp36-cp36m-linux_aarch64.whl
```

（2） TensorFlow 2.4.0

```shell
# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# install pip and pip3
$ sudo apt-get install python-pip python3-pip
# remove old versions, if not placed in a virtual environment (let pip search for them)
$ sudo pip uninstall tensorflow
$ sudo pip3 uninstall tensorflow
# install the dependencies (if not already onboard)
$ sudo apt-get install gfortran
$ sudo apt-get install libhdf5-dev libc-ares-dev libeigen3-dev
$ sudo apt-get install libatlas-base-dev libopenblas-dev libblas-dev
$ sudo apt-get install liblapack-dev
$ sudo -H pip3 install Cython==0.29.21
# install h5py with Cython version 0.29.21 (± 6 min @1950 MHz)
$ sudo -H pip3 install h5py==2.10.0
$ sudo -H pip3 install -U testresources numpy
# upgrade setuptools 39.0.1 -> 51.0.0
$ sudo -H pip3 install --upgrade setuptools
$ sudo -H pip3 install pybind11 protobuf google-pasta
$ sudo -H pip3 install -U six mock wheel requests gast
$ sudo -H pip3 install keras_applications --no-deps
$ sudo -H pip3 install keras_preprocessing --no-deps
# install gdown to download from Google drive
$ sudo -H pip3 install gdown
# download the wheel
$ gdown https://drive.google.com/uc?id=1W-p9oIo37xT2rQzd6KPuJq1QOBRL7-wp
# install TensorFlow (± 12 min @1500 MHz)
$ sudo -H pip3 install tensorflow-2.4.0-cp36-cp36m-linux_aarch64.whl
```

当安装成功时，您应该会看到以下屏幕.

![TF_2_4_1_Rdy_Jetson](https://qengineering.eu/images/TF_2_4_1_Rdy_Jetson.webp)

## 3.2 TensorFlow 2.4 C++ API

如果你计划用c++编程，你将需要TensorFlow的c++ API构建而不是Python版本。从我们的GitHub页面使用预构建tarball安装c++库可以节省大量时间。请遵循以下程序。

（1）TensorFlow 2.4.1

```shell
# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# remove old versions (if found)
$ sudo rm -r /usr/local/lib/libtensorflow*
$ sudo rm -r /usr/local/include/tensorflow
# the dependencies
$ sudo apt-get install wget curl libhdf5-dev libc-ares-dev libeigen3-dev
$ sudo apt-get install libatlas-base-dev zip unzip
# install gdown to download from Google drive (if not already done)
$ pip install gdown
# download the tarball
$ gdown https://drive.google.com/uc?id=1zJ_EF2aFkr8JU8JgTLfKMxC6KxE3DRD4
# unpack the ball
$ sudo tar -C /usr/local -xzf libtensorflow-2.4.1-JetsonNano.tar.gz
```



（2）TensorFlow 2.4.0

```shell
# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# remove old versions (if found)
$ sudo rm -r /usr/local/lib/libtensorflow*
$ sudo rm -r /usr/local/include/tensorflow
# the dependencies
$ sudo apt-get install wget curl libhdf5-dev libc-ares-dev libeigen3-dev
$ sudo apt-get install libatlas-base-dev zip unzip
# install gdown to download from Google drive (if not already done)
$ pip install gdown
# download the tarball
$ gdown https://drive.google.com/uc?id=1ZDdkMhVu-hOt_bflVw373u1HVJQqMm51
# unpack the ball
$ sudo tar -C /usr/local -xzf libtensorflow-2.4.0-JetsonNano.tar.gz
```

你应该把你的TensorFlow库安装在**/usr/local/lib**位置，头文件安装在**usr/local/include/ TensorFlow /c**文件夹中。

![TF_API_LIB_Jetson](https://qengineering.eu/images/TF_API_LIB_Jetson.webp)

![TensorFlow C++ API headers folder](https://qengineering.eu/images/TFheadersMapJetson.webp)

# 参考目录

[https://qengineering.eu/install-tensorflow-2.4.0-on-jetson-nano.html](https://qengineering.eu/install-tensorflow-2.4.0-on-jetson-nano.html)