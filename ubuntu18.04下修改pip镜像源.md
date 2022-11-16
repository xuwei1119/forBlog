ubuntu18.04下修改pip镜像源

https://www.cnblogs.com/FlyerBird/p/10953790.html

在`/home/用户名`目录下使用命令`mkdir .pip`创建`.pip`文件夹

然后`cd .pip`

使用命令`touch pip.conf`创建`pip.conf`文件

使用命令`gedit pip.conf`编辑

在打开的`pip.conf`文件中输入以下内容：

```txt
[global]
timeout=6000
index-url=https://pypi.doubanio.com/simple
trusted-host=pypi.doubanio.com
```



注意，如果上面不行，就用下面的内容替换上面的文件内容

https://blog.csdn.net/zaf0516/article/details/103961944

```txt
[global]
timeout=6000
index-url=https://mirrors.aliyun.com/pypi/simple/
trusted-host=mirrors.aliyun.com
```

**此处列举国内常用pip安装镜像：**

清华：https://pypi.tuna.tsinghua.edu.cn/simple
阿里云：https://mirrors.aliyun.com/pypi/simple/
中国科技大学: https://pypi.mirrors.ustc.edu.cn/simple/
华中理工大学：https://pypi.hustunique.com/
山东理工大学：https://pypi.sdutlinux.org/
豆瓣：https://pypi.douban.com/simple/
