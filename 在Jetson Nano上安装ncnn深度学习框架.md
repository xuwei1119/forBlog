#  在Jetson Nano上安装ncnn深度学习框架

# 1.简介

本页面将指导您在Jetson Nano上安装腾讯的ncnn框架。因为ncnn框架的目标是移动设备，比如Android手机，所以它没有CUDA支持。然而，大多数Android手机使用Vulkan API对其GPU进行低级访问。ncnn框架可以使用Vulkan例程来加速深度学习模型的卷积。Jetson Nano有ncnn将使用的Vulkan支持，关于软件结构的更多信息可以在[这里](https://qengineering.eu/deep-learning-with-raspberry-pi-and-alternatives.html)和[这里](https://qengineering.eu/deep-learning-algorithms-for-raspberry-pi-and-alternatives.html)找到。给出的C++代码示例是在Nano的code::Blocks IDE中编写的。我们只指导您完成基础知识，所以最终您可以构建自己的应用程序。有关ncnn库的更多信息，请参见:https://github.com/Tencent/ncnn。也许没有必要，但是安装的是C++版本，它不适合Python。

# 2.RTTI运行时类型识别(Run Time Type Identification)

RTTI代表运行时类型标识。它是一种c++机制，在运行时用于获取对象的类型和内存大小，但尚未定义。通常情况下，程序员知道变量的类型，并可以提前分配存储对象的内存。从具有所有进程和线程的操作系统中获取内存可能是一项相对耗时的操作。现代C编译器知道需要多少内存，调用一次内存管理就足够了。这是相对于Python的主要优势之一，Python不受内存需求的影响，直到碰到带有变量的代码行。

如果您想要编写尽可能快的代码，最好不要使用RTTI。在ncnn框架中也是如此。

默认情况下，它使用**-fno-rtti**标志进行编译，以防止使用RTTI。使用此标志编译的自定义层(如YOLOV5中的层)仅在剩余程序仍未使用RTTI的情况下可用。

有时不可能避免RTTI。特别是在成熟的代码中，需要一个新函数而不需要重写所有代码。OpenCV在某些地方使用RTTI机制。

这里有个问题。当你用OpenCV编译ncnn时，编译器会在**-fno-rtti**标志上返回一个错误。移除标志有时也适用于ncnn代码，这取决于使用的DNN的类型。

此时，使用的构建标志**-D NCNN_DISABLE_RTTI=OFF**变得清晰起来。它告诉编译器ncnn将允许RTTI。这意味着您现在可以完整地运行ncnn，而不会遇到OpenCV的问题。Jetson Nano上你不会注意到任何性能方面不同的。

# 3. 依赖与安装

ncnn框架几乎没有依赖项。它需要protobuf加载ONNX模型，当然，还有Vulkan SDK。如果OpenCV尚未安装，请先安装它，大约需要两个小时。

在使用Linux Tegra操作系统的Jetson Nano上安装ncnn的整个过程如下。

```shell
# a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# install dependencies
$ sudo apt-get install cmake wget
$ sudo apt-get install libprotobuf-dev protobuf-compiler libvulkan-dev
# download ncnn
$ git clone --depth=1 https://github.com/Tencent/ncnn.git
# download glslang
$ cd ncnn
$ git submodule update --depth=1 --init
# prepare folders
$ mkdir build
$ cd build
# build 64-bit ncnn for Jetson Nano
$ cmake -D CMAKE_TOOLCHAIN_FILE=../toolchains/jetson.toolchain.cmake \
        -D NCNN_DISABLE_RTTI=OFF \
        -D NCNN_BUILD_TOOLS=ON \
        -D NCNN_VULKAN=ON \
        -D CMAKE_BUILD_TYPE=Release ..
$ make -j4
$ make install
# copy output to dirs
$ sudo mkdir /usr/local/lib/ncnn
$ sudo cp -r install/include/ncnn /usr/local/include/ncnn
$ sudo cp -r install/lib/*.a /usr/local/lib/ncnn/
# once you've placed the output in your /usr/local directory,
# you can delete the ncnn directory if you wish
$ cd ~
$ sudo rm -rf ncnn
```

![cmake ncnn](https://qengineering.eu/images/cmake-ncnn.webp)

如果一切顺利，您将得到两个文件夹。一个包含所有头文件，另一个包含如屏幕中所示的库。

![Include_ncnn](https://qengineering.eu/images/ncnn_include_nano.webp)

![Lib_ncnn](https://qengineering.eu/images/ncnn_lib_nano.webp)

也请注意包含示例的文件夹。这里涵盖了许多不同类型的深度学习。由于ncnn库的版本更改，对实际深度学习模型的引用有时会导致错误。我们最近从nihui收到了以下最新型号的仓库:https://github.com/nihui/ncnn-assets/tree/master/models.

![ncnn_Examples](https://qengineering.eu/images/examples-ncnn.webp)

# 4.基线测试

我们在支持和不支持Vulkan的情况下进行了一些基准测试，以了解ncnn的性能如何。平均来说，当ncnn使用Vulkan时，您将获得57%的性能提升。这比MNN增加CUDA的效果更好。注意，在测试期间，Jetson Nano CPU被超频到2014.5 MHz, GPU被超频到998.4 MHz。

![image-20221102093752066](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102093752066.png)

# 参考目录

[https://qengineering.eu/install-ncnn-on-jetson-nano.html](https://qengineering.eu/install-ncnn-on-jetson-nano.html)