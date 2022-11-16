# 在Jetson Nano上安装Ubuntu 20.04

# 1.简介

Jetson Nano配备Ubuntu 18.04版本。在某些情况下，你需要Ubuntu 20.04。特别是ROS2需要Python 3.8。本指南讨论在Jetson Nano上升级到Ubuntu 20.04。我们将使用Ubuntu操作系统的标准升级机制。

请在[GitHub](https://github.com/Qengineering/Jetson-Nano-Ubuntu-20-image)页面上找到一个完整的带有预装Ubuntu 20.04, OpenCV, TensorFlow和PyTorch的Jetson Nano镜像。下载zip文件，解压缩镜像在32GB sd卡上。

# 2. 准备工作

首先，确保SD卡上有足够的空间执行升级。您至少需要16GB的空闲空间。

下面的操作是删除您不需要立即使用的软件包，例如LibreOffice。它减慢了整个过程，如果需要，你可以很容易地重新安装它们。

Chromium 也应该去除，因为它会干扰Snap安装。您可以通过使用软件更新程序或命令行来完成此操作。

![Ubuntu software](https://qengineering.eu/images/SoftwareUb20.webp)

一旦删除，更新，升级和清理您的系统。

```shell
# remove Chromium and OpenCV
$ sudo apt-get remove --purge chromium-browser chromium-browser-l10n
# refresh your system
$ sudo apt-get update
# need nano for editing some files
$ sudo apt-get install nano
$ sudo apt-get upgrade
$ sudo apt-get autoremove
```

接下来，您需要通过在**/etc/update-manager/release- updates**文件中设置p**rompt=normal**来启用更新管理器中的分发升级。

像往常一样，以<Ctrl>+<X>， <Y>和<Enter>结束。

```shell
$ sudo nano /etc/update-manager/release-upgrades
```

![Release updates](https://qengineering.eu/images/ReleaseUb20.webp)

设置了更新管理器后，我们需要再次刷新软件数据库。完成之后，就可以重新启动了。

```SHELL
# refresh your system again
$ sudo apt-get update
$ sudo apt-get dist-upgrade
$ sudo reboot
```

# 3.升级到Ubuntu 20.04

准备就绪后，是时候升级到Ubuntu 20.04了。这需要几个小时。不幸的是，在整个过程中需要一些输入，因为有一些问题需要回答。不时查看屏幕。用建议的默认值回答所有问题。

```SHELL
# upgrade to Ubuntu 20.04
$ sudo do-release-upgrade
```

![StartUpgrade](https://qengineering.eu/images/StartUpgradeUb20.webp)

在输入<y>之后，转换将发生。如前所述，在这个过程中会提出一些问题。始终确认默认值。也可能会有一个新的Ubuntu版本可用的通知，Hirsute Hippo 21.04版本。请取消该通知，因为我们暂时不会安装该版本。

当Ubuntu 20.04安装后，所有过时的文件都可以从你的驱动器中删除，就像屏幕上显示的那样。

![none](https://qengineering.eu/images/RemoveObsoleteUb20.webp)

现在这些包已经被删除了，Ubuntu将要求重新启动。现在不重新启动!这是非常重要的。

![Don't reboot](https://qengineering.eu/images/NoRebootUb20.webp)

在重启Ubuntu之前，您需要检查一些文件并做一些更改。如果你忘记了这一点，每次Ubuntu启动时你都会得到一个警告。所以马上行动吧。

首先，检查**/etc/gdm3/custom.conf**文件中是否没有注释**WaylandEnable=false**。

![Wayland](https://qengineering.eu/images/WaylandUb20.webp)

取消**/etc/ x11/xorg.conf**文件中的驱动程序**“nividia”**的注释。

![Driver nvidia](https://qengineering.eu/images/nvidiaUb20.webp)

最后，将升级管理器重置为never。

![none](https://qengineering.eu/images/ReleaseNeverUb20.webp)

现在您可以安全地重新启动系统了。

```SHELL
# check and editing some files
$ sudo nano /etc/gdm3/custom.conf
$ sudo nano /etc/X11/xorg.conf
$ sudo nano /etc/update-manager/release-upgrades
$ sudo reboot
```

# 4.在Nano上启动Ubuntu 20.04

在您可以在Jetson Nano上完全享受Ubuntu 20.04之前，需要一些准备工作。

从众所周知的更新、升级和自动删除循环开始。你可能不会得到升级，因为Ubuntu刚刚安装。然而，自动删除将通过删除过时的文件释放大量内存。

接下来，在使用Jtop时为防止出现lavapipe警告删除目录**/usr/share/vulkan/icd.d**。在最新的Jetpack中，这个文件夹被移除了。

```shell
# remove icd.d
$ sudo rm -rf /usr/share/vulkan/icd.d
```

同时删除**/usr/share/applications**中恼人的循环符号链接，它会使同一个应用程序在软件概述中出现86次。

```shell
# prepare your system
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get autoremove
# remove circular symlink
$ sudo rm /usr/share/applications/vpi1_demos
# remove distorted nvidia logo in top bar
$ cd /usr/share/nvpmodel_indicator
$ sudo mv nv_logo.svg no_logo.svg
```

最后一个操作是重新启用在升级期间禁用的原始NVIDIA存储库。在**/etc/apt/sources.list.d/**文件夹中。你会发现需要修改的五个文件。用**$ sudo nano**打开每个文件，并删除行前的注释符，以激活存储库。

![none](https://qengineering.eu/images/RepoUb20.webp)

# 5.gcc-8

Ubuntu 20.04自带gcc 9.3.0版本。

![Gcc9Ub20](https://qengineering.eu/images/Gcc9Ub20.png)

有些软件包，特别是CUDA软件，需要gcc版本8。我们将在已有的9版本之外安装这个版本。通过一个简单的命令，您现在可以在两个版本之间切换。gcc编译器总是伴随着相应的g++编译器。后者也将安装。

```shell
# install gcc and g++ version 8
$ sudo apt-get install gcc-8 g++-8
# setup the gcc selector
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 9
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 8
# setup the g++ selector
$ sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-9 9
$ sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 8
# if you want to make a selection use these commands
$ sudo update-alternatives --config gcc
$ sudo update-alternatives --config g++
```

总是同时选择两个对应的版本。否则，编译过程中可能会出现非常模糊的错误。如前所述，如果您想编译需要CUDA或cuDNN的软件，如OpenCV，请选择版本8。

您的选择被保存并应用于系统范围。即使在重新启动之后，所选的版本对仍然是活动的。更改版本的唯一方法是重新提交命令。

![SelectorUb20](https://qengineering.eu/images/SelectorUb20.webp)

clang编译器也是如此。在这种情况下，CUDA和cuDNN需要版本8，而Ubuntu安装默认版本10。您可以使用**$ sudo apt-get install clang-8**立即安装版本8，也可以像上面那样安装更多版本并创建**sudo update-alternatives  --config**。

**警告**

不要安装**Chromium**，因为它会干扰Snap安装。使用预安装的**Morzilla Firefox**。

# 6.升级

在Jetson Nano上升级Ubuntu 20.04一段时间后，可能会遇到问题。软件更新程序无法安装列出的所有软件包。当您运行**$ sudo apt-get upgrade**时，您将看到以下屏幕。

![ErrorUpgradeUb20](https://qengineering.eu/images/ErrorUpgradeUb20.webp)

问题是**nvidia-l4t-init**文件的版本错误。由于许多其他包都依赖于此文件，因此整个升级过程将终止。

您必须首先安装正确的版本，才能成功进行任何进一步升级。

命令**$ sudo apt——fix-broken install**提供以下信息。

![FixBrokenUb20](https://qengineering.eu/images/FixBrokenUb20.webp)

看起来**/etc/systemd/sleep.conf**正在阻止升级。最简单的解决方案是使用命令**$ sudo dpkg -i --force-overwrite**强制升级。注意，**nvidia-l4t-init**文件的位置在上一个**fix-broken**命令的输出中给出。

![SuccessForceUb20](https://qengineering.eu/images/SuccessForceUb20.webp)

如您所见，**sleep.conf**被覆盖，**nvidia-l4t-init**被成功安装。

现在你可以用通常的命令**$ sudo apt-get upgrade**升级你的Ubuntu 20.04。

# 7.结束语

Ubuntu 20.04现在可以在你的Jetson Nano上完全运行了。您可以根据自己的需要进一步定制安装。

我们对完成某些任务所需的时间感到惊讶。当我们为OpenCV安装设置dphys交换机制时，花了令人伤脑筋的五分钟才完成重新启动。我们总是盯着一个空白的屏幕。所以在这种情况下要有耐心。

我们还必须在设置中禁用省电的空白屏幕选项，因为当从休眠模式重新启动时，它会给出不稳定的结果。

![SleepUb20](https://qengineering.eu/images/SleepUb20.webp)

Nvidia 没有将Ubuntu 20.04与它的JetPack一起发布是有原因的。这当然与与18.04版本相比增加的价值很少有关。但也会有其他原因。

# 参考目录

[https://qengineering.eu/install-ubuntu-20.04-on-jetson-nano.html](https://qengineering.eu/install-ubuntu-20.04-on-jetson-nano.html)