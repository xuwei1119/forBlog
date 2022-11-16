ubuntu常用软件安装QQ，wechat

首先要安装`deepin-wine`

解压`deepin-wine-ubuntu-master.zip`文件夹

然后进入运行 `./install.sh`



类似,`QQ, WeChat`下载目录如下：

http://packages.deepin.com/deepin/pool/non-free/d/deepin.com.wechat/

http://packages.deepin.com/deepin/pool/non-free/d/deepin.com.qq.im/

执行安装命令

```shell
sudo dpkg -i deepin.com.qq.im_8.9.19983deepin23_i386.deb
```

卸载

首先查找deb软件名称

```shell
sudo dpkg -l | grep qq

sudo dpkg -r deepin.com.qq.im:i386
```

