conda改为国内镜像源

（1）文件配置

```shell
sudo gedit ~/.condarc
```

在打开的文件中添加以下内容：

+ 阿里源

```shell
channels:
  - defaults
show_channel_urls: true
default_channels:
  - http://mirrors.aliyun.com/anaconda/pkgs/main
  - http://mirrors.aliyun.com/anaconda/pkgs/r
  - http://mirrors.aliyun.com/anaconda/pkgs/msys2
custom_channels:
  conda-forge: http://mirrors.aliyun.com/anaconda/cloud
  msys2: http://mirrors.aliyun.com/anaconda/cloud
  bioconda: http://mirrors.aliyun.com/anaconda/cloud
  menpo: http://mirrors.aliyun.com/anaconda/cloud
  pytorch: http://mirrors.aliyun.com/anaconda/cloud
  simpleitk: http://mirrors.aliyun.com/anaconda/cloud
```

+ 清华源

```shell
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

（2）命令行配置

也可以命令行配置

显示已配置镜像

```bash
#查看当前conda配置
conda config --show channels

channels:
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/fastai/
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - defaults
```

配置国内镜像

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud//pytorch/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
 
#设置搜索是显示通道地址
conda config --set show_channel_urls yes
```

当然也可以删除配置

```bash
conda config --remove-key channels #删除所有的镜像源，恢复到默认
conda config --remove channels [urls] #删除指定的镜像源
```

（3）国内常用镜像

```bash
http://mirrors.aliyun.com/pypi/simple/ #阿里
https://pypi.tuna.tsinghua.edu.cn/simple/ #清华
http://pypi.douban.com/ #豆瓣
http://pypi.mirrors.ustc.edu.cn/ #中国科学技术大学
```

（4）清除缓存

配置完成可运行 `conda clean -i` 清除索引缓存。