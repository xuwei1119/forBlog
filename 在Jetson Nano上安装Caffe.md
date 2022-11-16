# 在Jetson Nano上安装Caffe

# 1.简介

本页面将指导您在具有CUDA和cuDNN支持的Jetson Nano上安装Caffe。我们只指导您完成基础知识，所以最终您可以构建自己的应用程序。有关Caffe的更多信息，请访问:https://caffe.berkeleyvision.org/。

# 2.依赖

Caffe框架对其他库有一些依赖关系。根据我们的指南，我们假设您已经在您的Jetson Nano上安装了OpenCV。如果没有，最好先做。

```shell
$ sudo apt-get install cmake git unzip
$ sudo apt-get install libgoogle-glog-dev libgflags-dev
$ sudo apt-get install libprotobuf-dev libleveldb-dev liblmdb-dev
$ sudo apt-get install libsnappy-dev libhdf5-serial-dev protobuf-compiler
$ sudo apt-get install --no-install-recommends libboost-all-dev
$ sudo apt-get install libatlas-base-dev libopenblas-dev
$ sudo apt-get install the python3-dev python3-skimage
$ sudo -H pip3 install pydot
$ sudo apt-get install graphviz
```

# 3.下载Caffe

安装了所有必需的库之后，就可以下载Caffe了。

在我们的GitHub页面上，你会发现著名的Caffe克隆自WeiLui89。与最初的Caffe不同，该版本支持SSD操作。WeiLui 是SSD探测器的最初开发者之一。有关详细信息，请参阅他的[文章](https://arxiv.org/pdf/1512.02325.pdf)。

我们特别为OpenCV 4.4、cuDNN 8.0和Python3改编了他的fork。此外，我们还为树莓派、Jetson Nano和Ubuntu添加了一些额外的Makefile脚本。

```shell
$ cd ~
$ wget -O caffe.zip https://github.com/Qengineering/caffe/archive/ssd.zip
$ unzip caffe.zip
$ mv caffe-ssd caffe
```

# 4.cuDNN8.0

从8.0版本开始，NVIDIA已经放弃了对Caffe所依赖的一些cuDNN API调用的支持。

编译生成`cudnn_convolution_fwd_specificy_workspace_limit未在此作用域中声明`。

cudnnGetConvolutionForwardAlgorithm, cudnnGetConvolutionBackwardDataAlgorithm可以被其他调用替换。

然而，cudnnGetConvolutionBackwardFilterAlgorithm不再是一个合适的选择。

这给你留下了三种选择。

+ 降级到cuDNN版本7.6，我们强烈建议不要这样做。它会导致其他依赖性问题的。
+ 编译没有cuDNN加速的Caffe。保存和CUDA仍然工作，一个不太坏的选择。
+ 在GitHub Caffe中使用修复。我们已经将cudnnGetConvolutionBackwardFilterAlgorithm的结果设置为一个常量。该程序在给定层拓扑的情况下寻找最快的cuDNN解。使用一般的CUDNN_CONVOLUTION_BWD_FILTER_ALGO_0，您可能没有最佳的选择，但仍然是有效的。顺便说一下，该修复只适用于cuDNN发行版8.0。较老的cuDNN版本将使用原始代码编译。

# 5.构建Caffe

我们已经为Jetson Nano创建了一个配置文件。它应该没有错误。如果构建失败，请查看Caffe[配置](https://qengineering.eu/caffe-configuration.html)页面。

```shell
$ cd ~/caffe
# select the configuration
$ cp Makefile.config.JetsonNano_example Makefile.config
# start the build
$ make clean
$ make all -j4
$ make test -j4
$ make runtest -j4
```

希望您将在make runtest之后看到下面的屏幕，表明一切都很顺利。整个测试需要5分钟以上才能完成。与树莓派4相比，它需要更多的时间。有两个原因。首先，在启用CUDA和cuDNN的情况下，将运行更多的测试，分别为2301和1266。其次，在一些测试中，CUDA加速将执行启发式搜索，以找到最快的算法，稍后将在测试中使用。测试在几毫秒内完成，而准备工作可能需要几秒钟。

![Caffe test output](https://qengineering.eu/images/caffe-test-output.webp)

您可能还需要编译一些其他软件。首先是pycaffe。如果您想在Python 3中运行Caffe，您将需要这个接口。否则，Caffe只能从命令行或C++程序中运行。有关更多信息，请参阅原始的Caffe文档。

你也可以在MATLAB中运行Caffe。这次你需要`make matcaffe`。但是在成功构建matcaffe之前，需要设置Makefile.config中的一些行配置。同样，原始文档是您的朋友。在pytest之后，你的屏幕看起来就像下图一样。

```shell
# Assuming you're still in ~/caffe
$ make pycaffe
$ make pytest
```

![Pytest output](https://qengineering.eu/images/pytest-cudnn.webp)

最后的说明时间到了，您的Caffe安装已经准备好在Jetson Nano上运行。

您需要将Caffe添加到`PYTHONPATH`中。这可以在`~/.bashrc`或`~/.profile`中完成。我们选择`~/.bashrc`。(打开文件，并在文件的最后添加一行`export PYTHONPATH="$PYTHONPATH:$HOME/caffe/python"`。像往常一样用Crtl+X, Y和Enter键保存并关闭。

最后，您现在可以从任何位置导入Caffe，如截图所示。

```shell
$ cd ~
$ sudo nano ~/.bashrc
# at the end of the file add the next line
export PYTHONPATH="${PYTHONPATH}:$HOME/caffe/python"
# save and close with
Ctrl+X, Y, Enter
```

![.bashrc](https://qengineering.eu/images/bashrc-caffe.webp)

![Caffe Rdy](https://qengineering.eu/images/caffe-version-python.webp)

# 参考目录

[https://qengineering.eu/install-caffe-on-jetson-nano.html](https://qengineering.eu/install-caffe-on-jetson-nano.html)