# 在Jetson Nano上安装MNN深度学习框架

# 1.简介

本页面将指导您在Jetson Nano上安装阿里巴巴的MNN框架。在最新的版本中，MNN框架还支持CUDA。这使得它成为Jetson Nano作为一个轻量级框架的理想选择。给出的C++代码示例是在Nano的code::Blocks IDE中编写的。我们只指导您完成基础知识，所以最终您可以构建自己的应用程序。有关MNN库的更多信息，请参阅[这里](https://www.yuque.com/mnn/en)的文档。也许没有必要，但是安装的是C++版本, 不适合Python。

# 2.依赖

MNN框架有一些依赖项。它需要protobuf。OpenCV被用于构建C++示例，而MNN不需要OpenCV。在使用Linux Tegra操作系统的Jetson Nano上安装MNN的步骤如下。

```shell
# check for updates
$ sudo apt-get update
$ sudo apt-get upgrade
# install dependencies
$ sudo apt-get install cmake wget
$ sudo apt-get install libprotobuf-dev protobuf-compiler
$ sudo apt-get install libglew-dev
```

在编译MNN软件之前，有一件事要做。MNN Vulkan接口使用OpenGL ES 3.0库。它是一个用于Android的低级图形渲染界面。幸运的是，它与Jetson Nano上的JetPack 4.4中的2.0版本库向后兼容。而且，据我们所知，MNN框架不使用任何独特的3.0版本调用。它使得使用符号链接将libGLES3.0重定向到libGLES2.0成为可能。这种策略非常有效，可以将您从安装3.0版本的繁琐过程中解放出来。

```shell
# make symlink
$ sudo ln -s /usr/lib/aarch64-linux-gnu/libGLESv2.so /usr/lib/aarch64-linux-gnu/libGLESv3.so
```

# 3.安装

安装了依赖项之后，就可以构建库了。

```shell
# download MNN
$ git clone https://github.com/alibaba/MNN.git
# common preparation (installing the flatbuffers)
$ cd MNN
$ ./schema/generate.sh
# install MNN
$ mkdir build
$ cd build
# generate build script
$ cmake -D CMAKE_BUILD_TYPE=Release \
        -D MNN_BUILD_QUANTOOLS=ON \
        -D MNN_BUILD_CONVERTER=ON \
       -D MNN_OPENGL=ON \
       -D MNN_VULKAN=ON \
       -D MNN_CUDA=ON \
        -D MNN_TENSORRT=OFF \
        -D MNN_BUILD_DEMO=ON \
        -D MNN_BUILD_BENCHMARK=ON ..
```

![CMake MNN Jetson](https://qengineering.eu/images/cmake-mnn-rdy-jetson2.webp)

是时候构建库并将其安装到适当的文件夹中了。

```shell
# build MNN (± 25 min)
$ make -j4
$ sudo make install
$ sudo cp ./source/backend/cuda/*.so /usr/local/lib/
# don't copy until MNN has solved the issues with the TensorRT backend
# $ sudo cp ./source/backend/tensorrt/*.so /usr/local/lib/
```

![image-20221102094924422](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102094924422.png)

然后**sudo make install**。

![MNN_build_rdy](https://qengineering.eu/images/install-rdy-mnn-jetson.webp)

如果一切顺利，您的Jetson Nano上有以下文件夹。

![MNN_include](https://qengineering.eu/images/include-MNN.webp)

![lib MNN Jetson](https://qengineering.eu/images/lib-MNN-Jetson.webp)

![none](https://qengineering.eu/images/convert-build-MNN.webp)

也请注意包含示例的文件夹。

![none](https://qengineering.eu/images/Examples-MNN-Jetson.webp)

如果您想下载一些示例深度学习模型，可以使用下面的命令。

```shell
# download some models
$ cd ~/MNN
$ ./tools/script/get_model.sh
```

# 4.基准测试

有了新的CUDA后端，看看MNN的性能有多好是很有趣的。以下是一些基准。平均而言，当MNN使用CUDA时，您将获得40%的性能提升。注意，在测试期间，Jetson Nano CPU被超频到2014.5 MHz, GPU被超频到998.4 MHz。

![image-20221102095306406](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102095306406.png)

# 参考目录

[https://qengineering.eu/install-mnn-on-jetson-nano.html](https://qengineering.eu/install-mnn-on-jetson-nano.html)