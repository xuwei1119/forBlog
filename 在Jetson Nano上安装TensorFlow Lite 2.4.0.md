# 在Jetson Nano上安装TensorFlow Lite 2.4.0

# 1.简介

本指南的第一部分将引导您在Jetson Nano上安装TensorFlow Lite。

第二部分将指导您使用GPU安装TensorFlow Lite。必须说，预期的加速有点令人失望。

第三部分介绍了用于了解Nano上TensorFlow Lite性能的c++示例。

TensorRT默认与Jetson Nano一起作为深度学习框架。它是一个基于CUDA和cuDNN的c++库。由于它的底层结构，它需要相当熟练的编程技能。这就是为什么我们不介绍TensorRT框架，尽管它的执行速度比TensorFlow Lite快一点。

# 2.准备

如果你想运行我们提供的TensorFlow Lite示例，请确保在Jetson Nano上安装了OpenCV。它可能是不支持CUDA的默认版本，或者您可以根据我们的指南重新安装带有CUDA的OpenCV 4.5.0。

# 3.安装TensorFlow Lite

如果你想要构建快速的深度学习应用程序，你必须使用c++。这就是为什么你需要构建TensorFlow Lite的c++ API库的原因。过程很简单，只需复制最新的GitHub存储库并运行这两个脚本。下面列出了这些命令。这个安装忽略了Jetson Nano上的CUDA GPU。它纯粹是基于CPU的。

（1）TensorFlow Lite 2.4.1

```shell
# the tools needed
$ sudo apt-get install cmake curl
# download TensorFlow version 2.4.1
$ wget -O tensorflow.zip https://github.com/tensorflow/tensorflow/archive/v2.4.1.zip
# unpack and give the folder a convenient name
$ unzip tensorflow.zip
$ mv tensorflow-2.4.1 tensorflow
$ cd tensorflow
# get the dependencies
$ ./tensorflow/lite/tools/make/download_dependencies.sh
# run the C++ installation
$ ./tensorflow/lite/tools/make/build_aarch64_lib.sh
```

（1）TensorFlow Lite 2.4.0

```shell
# the tools needed
$ sudo apt-get install cmake curl
# download TensorFlow version 2.4.0
$ wget -O tensorflow.zip https://github.com/tensorflow/tensorflow/archive/v2.4.0.zip
# unpack and give the folder a convenient name
$ unzip tensorflow.zip
$ mv tensorflow-2.4.0 tensorflow
$ cd tensorflow
# get the dependencies
$ ./tensorflow/lite/tools/make/download_dependencies.sh
# run the C++ installation
$ ./tensorflow/lite/tools/make/build_aarch64_lib.sh
```

![TF Lite Rdy Jetson](https://qengineering.eu/images/TF-Lite-Rdy-Jetson.webp)

TensorFlow Lite缓冲区也是必需的。请使用以下命令。

```shell
# install the flatbuffers
$ cd ~/tensorflow/tensorflow/lite/tools/make/downloads/flatbuffers
$ mkdir build
$ cd build
$ cmake ..
$ make -j4
$ sudo make install
$ sudo ldconfig
# clean up
$ cd ~
$ rm tensorflow.zip
```

如果一切顺利，您应该拥有两个库和两个带有头文件的文件夹，如幻灯片所示。

![TFliteNanoLib.png](https://qengineering.eu/my_gallery/TFliteNanoLib.webp)

从TensorFlow Lite 2.3.0版本开始，Tensorflow Lite使用动态链接。在运行时，库被复制到RAM中，指针在TF Lite可以运行之前被重新定位。这种策略提供了更大的灵活性。这一切都意味着TensorFlow Lite现在需要glibc 2.28或更高版本才能运行。从现在开始，在构建应用程序时链接libdl库，否则，您将得到对符号**dlsym@@GLIBC_2.17**链接器错误的未定义引用。符号链接可以在64位Linux操作系统** /lib/aarch64-linux-gnu/libdl.so.2**找到。请在GitHub上查看我们的例子。

现在，您已经在Jetson Nano上拥有了TensorFlow Lite 2.3.1的完整操作版本。正如您可能已经发现的那样，安装与使用64位操作系统与树莓派4几乎没有区别。



# 4.安装GPU支持的TensorFlow Lite

TensorFlow Lite最初是为智能手机和其他小型设备开发的，它永远不会遇到CUDA GPU。因此，它不支持CUDA或cuDNN。另一方面，它可以包括所谓的GPU。手机中的GPU硬件，如MALI GPU，被用来加速张量计算，希望获得速度。大多数硬件由OpenCL或OpenGL ES软件提供支持。Jetson Nano没有OpenCL，但OpenGL ES API与JetPack一起提供。

如前所述，最终的性能比单独使用四核CPU略差。这可能与TensorFlow Lite实际上将所有计算都转移到GPU有关。GPU和CPU之间没有像如ncnn, MNN或Paddle Lite平衡的混合。TensorFlow Lite所有东西都要在GPU上处理。它可以很好地完成特定的任务，正如这里所解释的，但其他任务可能会非常糟糕。甚至有些操作GPU不能执行。例如，在MobileNetV1中发现的像Concatenation或Logistic这样的操作。

![errorGPUdelegate](https://qengineering.eu/images/errorGPUdelegate.png)

第二个令人失望的原因是TensorFlow Lite被迫选择OpenGL ES，因为Jetson Nano上缺乏更强大的OpenCL。

注意，为了GPU支持，您还需要先前构建的**libtensorflow-lite.a**和**libflatbuffers.a**库。安装需要很多资源。为了编译GPU C++ API，必须首先安装bazel。

## 4.1 增大内存交换大小

构建完整的TensorFlow Lite 2.4.1包需要超过6Gbyte的RAM。最好临时重新安装dphysi -swapfile，以获得SD卡的额外空间。安装完成后，我们将删除dphysics -swapfile。执行以下命令。

```shell
# install dphys-swapfile
$ sudo apt-get install dphys-swapfile
# give the required memory size
$ sudo nano /etc/dphys-swapfile
# reboot afterwards
$ sudo reboot.
```

![2 GB swap space](https://qengineering.eu/images/SwapJetson.webp)

如果一切顺利，你应该会得到这样的东西。

![4 GB swap memory](https://qengineering.eu/images/FreeJetsonSwap.webp)

为了记录，所示的图是由dphysi -swapfile和zram分配的交换空间总量。完成后不要忘记删除dphysics -swapfile。

## 4.2 Bazel

Bazel是谷歌提供的一个免费软件工具，用于自动构建和测试软件包。您可以将它与OpenCV使用的CMake进行比较，但后者只构建软件，没有测试工具。Bazel是用Java编写的，Java是一种平台独立的语言，在语法方面主要基于c++。要编译Bazel，首先必须使用以下命令安装Java和其他依赖项。

```shell
# get a fresh start
$ sudo apt-get update
$ sudo apt-get upgrade
# install pip and pip3
$ sudo apt-get install python-pip python3-pip
# install some tools
$ sudo apt-get install build-essential zip unzip curl
# install Java
$ sudo apt-get install openjdk-11-jdk
```

接下来，我们可以下载并解压Bazel软件。我们安装TensorFlow Lite 2.4.1需要的Bazel 3.1.0版本，所以请确保您安装了正确的版本。

```shell
$ wget https://github.com/bazelbuild/bazel/releases/download/3.1.0/bazel-3.1.0-dist.zip
$ unzip -d bazel bazel-3.1.0-dist.zip
$ cd bazel
```

在安装期间，Bazel使用预定义的可用工作内存比例。由于Jetson Nano的RAM大小有限，这个比例太小了。为了防止崩溃，必须在过程中将该内存的大小定义为最大1600 Mbyte。这是通过向脚本文件**compile.sh**添加一些额外的信息来实现的。可以在以run..开始的行中添加文本**-J-Xmx1600M**。(约154行)。请参阅下面的屏幕。使用众所周知的<Ctrl + X>， <Y>， <Enter>保存更改。

```shell
$ nano scripts/bootstrap/compile.sh -c
```

![Bazel 1600 Mb heap](https://qengineering.eu/images/BazelConfigJetson.webp)

一旦Bazel的Java环境最大化到1600 Mb，就可以开始使用下面的命令构建Bazel软件了。完成后，将二进制文件复制到/usr/local/bin位置，以便bash可以在任何地方找到可执行文件。最后一个操作是删除zip文件。整个构建过程大约需要33分钟。

```shell
# start the build
$ env EXTRA_BAZEL_ARGS="--host_javabase=@local_jdk//:jdk" bash ./compile.sh
# copy the binary
$ sudo cp output/bazel /usr/local/bin/bazel
# clean up
$ cd ~
$ rm bazel-3.1.0-dist.zip
# if you have a copied bazel to /usr/local/bin you may also
# delete the whole bazel directory, freeing another 500 MByte
$ sudo rm -rf bazel
```

## 4.3 构建TensorFlow Lite GPU

随着Bazel的启动和运行，我们可以开始在Jetson Nano上为TensorFlow Lite 2.4.1构建GPU。从GitHub下载TensorFlow并解压软件。

(1) TensorFlow Lite 2.4.1

```shell
# download TensorFlow 2.4.1
$ wget -O tensorflow.zip https://github.com/tensorflow/tensorflow/archive/v2.4.1.zip
# unpack and give the folder a convenient name
$ unzip tensorflow.zip
$ mv tensorflow-2.4.1 tensorflow
```

(1) TensorFlow Lite 2.4.0

```shell
# download TensorFlow 2.4.0
$ wget -O tensorflow.zip https://github.com/tensorflow/tensorflow/archive/v2.4.0.zip
# unpack and give the folder a convenient name
$ unzip tensorflow.zip
$ mv tensorflow-2.4.0 tensorflow
```

编译TensorFlow Lite GPU之前的下一步是配置Bazel。这是通过脚本文件和命令行选项完成的。让我们从脚本文件开始。使用以下命令，Bazel会问您几个问题。将Python 3定义为默认的Python版本。

```shell
$ cd tensorflow
$ ./configure
```

```shell
jetson@nano:~/tensorflow$ ./configure
You have bazel 3.1.0- (@non-git) installed.
Please specify the location of python. [Default is /usr/bin/python3]: <enter>

Found possible Python library paths:
 /usr/local/lib/python3.6/dist-packages
 /usr/lib/python3.6/dist-packages
 /usr/lib/python3/dist-packages
Please input the desired Python library path to use.  Default is [/usr/local/lib/python3.6/dist-packages] <enter>
/usr/lib/python3/dist-packages

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: n
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with ROCm support? [y/N]: n
No ROCm support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: y
CUDA support will be enabled for TensorFlow.

Do you wish to build TensorFlow with TensorRT support? [y/N]: y
TensorRT support will be enabled for TensorFlow.

Found CUDA 10.2 in:
   /usr/local/cuda-10.2/targets/aarch64-linux/lib
   /usr/local/cuda-10.2/targets/aarch64-linux/include
Found cuDNN 8 in:
   /usr/lib/aarch64-linux-gnu
   /usr/include
Found TensorRT 7 in:
   /usr/lib/aarch64-linux-gnu
   /usr/include/aarch64-linux-gnu

Please specify a list of comma-separated CUDA compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus. Each capability can be specified as "x.y" or "compute_xy" to include both virtual and binary GPU code, or as "sm_xy" to only include the binary code.
Please note that each additional compute capability significantly increases your build time and binary size, and that TensorFlow only supports compute capabilities >= 3.5 [Default is: 3.5,7.0]: 5.3

Do you want to use clang as CUDA compiler? [y/N]: n
nvcc will be used as CUDA compiler.

Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]: <enter>

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native -Wno-sign-compare]: <enter>

Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: n
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See .bazelrc for more details.
--config=mkl          # Build with MKL support.
--config=monolithic   # Config for mostly static monolithic build.
--config=ngraph       # Build with Intel nGraph support.
--config=numa         # Build with NUMA support.
--config=dynamic_kernels # (Experimental) Build kernels into separate shared objects.
--config=v2           # Build TensorFlow 2.x instead of 1.x.
Preconfigured Bazel build configs to DISABLE default on features:
--config=noaws        # Disable AWS S3 filesystem support.
--config=nogcp        # Disable GCP support.
--config=nohdfs       # Disable HDFS support.
--config=nonccl       # Disable NVIDIA NCCL support.
Configuration finished
```

脚本文件现在都设置好了，可以使用下面的命令开始构建。为了清晰起见，这条命令很长。

```shell
$ sudo bazel build -s -c opt --copt="-DMESA_EGL_NO_X11_HEADERS" tensorflow/lite/delegates/gpu:libtensorflowlite_gpu_delegate.so
```

几分钟以后你将得到以下屏幕信息

![GPU delegate compiled](https://qengineering.eu/images/BazelGPU_TF_LiteRdy.webp)

你会在上面提到的地方找到库。

![GPU delegate folder](https://qengineering.eu/images/GPUdelegateFolder.webp)

如果必须重新安装dphysi -swapfile，那么是时候重新卸载它了。这样可以延长SD卡的使用寿命。

```shell
# remove the dphys-swapfile (if installed)
$ sudo /etc/init.d/dphys-swapfile stop
$ sudo apt-get remove --purge dphys-swapfile
```

# 5.基准测试

有了GPU库之后，是时候进行一些测试了。四种著名的TensorFlow Lite模型已经在两种不同的时钟速度下部署了GPU版本和非GPU版本。一个超频，另一个默认速度。此外，一些来自超频树莓派4的数据也被添加到表中。结果不言自明。所有代码都在我们的GitHub页面。只需单击模型的名称，就会显示相应的C++示例。

![image-20221102091009784](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221102091009784.png)

# 参考目录

[https://qengineering.eu/install-tensorflow-2-lite-on-jetson-nano.html](https://qengineering.eu/install-tensorflow-2-lite-on-jetson-nano.html)