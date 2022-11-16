# 在Jetson Nano上安装TensorFlow插件

# 1.简介

TensorFlow插件为你提供了还没有包括在TensorFlow核心的额外的功能。如果它被证明是有用的，插件就会合并到TensorFlow包中。其他算法仍停留在插件群体中，只被TensorFlow社区中的少数人所使用。

由于包的实验性质，版本的生命周期很短。仅在2020年就发布了14个版本。我们希望能跟上发展的步伐。如果没有，请让我们知道，我们会尽量在我们的GitHub页面上提供缺失的版本。

# 2. TensorFlow

如前所述，插件与TensorFlow框架一起工作，因此您需要在您的系统上有一个工作的TensorFlow版本。如果需要，你可以根据我们的指南安装一个最新的TensorFlow版本。你可以在这里用不同的插件版本检查你的TensorFlow版本。

## 2.1轮子安装

一旦你启动并运行TensorFlow，你就可以安装插件了。安装TensorFlow-Addons最简单的方法是使用我们放置在GitHub页面上的轮子。这是Bazel从头开始耗时安装的结果，详见下一段。请按照说明操作。

```shell
# download the wheel
$ wget https://github.com/Qengineering/TensorFlow-Addons-Jetson-Nano/raw/main/tensorflow_addons-0.13.0.dev0-cp36-cp36m-linux_aarch64.whl
# install the wheel
$ sudo -H pip3 install tensorflow_addons-0.13.0.dev0-cp36-cp36m-linux_aarch64.whl
```

现在可以检查安装，如下面的屏幕中所示。

![tfa_succes_nano](https://qengineering.eu/images/tfa_succes_nano.webp)

## 2.2 从源码安装

从零开始构建TensorFlow-Addons并不困难。编译所有代码只需要时间。最后，你得到的轮子和我们放在GitHub上的一样。如果你想节省一些时间，请随意使用这个轮子。如果你想自己构建插件，这里有完整的指南。

### 2.2.1 Bazel

首先，您需要bazel，这是一种常用的构建工具，如CMake。我们已经在从头开始构建TensorFlow的过程中使用了bazel。

### 2.2.2 TensorFlow代码

插件使用TensorFlow代码进行CUDA加速。您需要下载TensorFlow并发出一个./configure命令。你不需要真正构建TensorFlow，你只需要设置环境，这样Bazel就可以确定它面对的是哪种CUDA架构。我们在说明中使用了TensorFlow 2.4.1，但它可以是你选择的任何版本。你只需要改变编号。

```shell
# download TensorFlow 2.4.1
$ wget -O tensorflow.zip https://github.com/tensorflow/tensorflow/archive/v2.4.1.zip
# unpack and give the folder a convenient name
$ unzip tensorflow.zip
$ mv tensorflow-2.4.1 tensorflow
$ cd tensorflow
# reveal the CUDA location
$ sudo sh -c "echo '/usr/local/cuda/lib64' >> /etc/ld.so.conf.d/nvidia-tegra.conf"
$ sudo ldconfig
# give the settings
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

### 2.2.3 构建插件

安装bazel并准备好TensorFlow代码后，现在可以下载Addons。

在开始编译**Addons**之前，必须修改**configure.py**脚本，为bazel构建设置正确的环境变量。另一个需要注意的脚本是**build_pip_pkg.sh**。在这里，我们必须将“旧的”python更改为python3，以使轮子生成工作。

我们已经向TensorFlow-Addon社区发出了pull请求，以修改**Jetson Nano**的**configuration .py**和**build_pip_pkg.sh**。在他们批准之前，请使用我们GitHub页面上的脚本。

```shell
# get the aadons
$ cd ~
$ git clone --depth=1 https://github.com/tensorflow/addons.git
# get the modified configure script and replace the original one
$ wget https://github.com/Qengineering/TensorFlow-Addons-Jetson-Nano/raw/main/configure.py
$ mv configure.py ./addons/
# get the modified build_pip_pkg.sh script and replace the original one
$ wget https://github.com/Qengineering/TensorFlow-Addons-Jetson-Nano/raw/main/build_pip_pkg.sh
$ mv build_pip_pkg.sh ./addons/build_deps
# symlink the tensorflow lib to /usr/lib
$ sudo ln -s /usr/local/lib/python3.6/dist-packages/tensorflow/python/_pywrap_tensorflow_internal.so /usr/lib/lib_pywrap_tensorflow_internal.so
# the Jetson Nano configuration will be set
$ cd ~/addons
$ python3 ./configure.py
# and start the build
$ bazel clean
$ bazel build build_pip_pkg
# finish with the wheel generation
$ sudo bazel-bin/build_pip_pkg /tmp/tensoraddons_pkg
# install the wheel
$ cd /tmp/tensoraddons_pkg
$ sudo -H pip3 install tensorflow_addons-0.13.0.dev0-cp36-cp36m-linux_aarch64.whl
# you can remove the addons folder, as it is no longer needed
$ sudo rm -rf ~/addons
```

# 参考目录

[https://qengineering.eu/install-tensorflow-addons-on-jetson-nano.html](https://qengineering.eu/install-tensorflow-addons-on-jetson-nano.html)