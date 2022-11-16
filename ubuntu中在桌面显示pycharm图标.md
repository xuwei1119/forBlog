ubuntu中在桌面显示pycharm图标

首先找到`pycharm`里面的`pycharm.sh`，然后执行命令`sh ./pycharm.sh`

首先进入到`other locations` 找到 `/usr/share/application`

然后再输入`sudo gedit pycharm.desktop`

在打开的文件里面输入以下内容

```shell
[Desktop Entry]
Version = 1.0
Type = Application
Name = Pycharm
Icon = /home/用户名/Pycharm/pycharm-2020.3.5/bin/pycharm.png
Exec = sh /home/用户名/Pycharm/pycharm-2020.3.5/bin/pycharm.sh
MimeType = applications/x-py;
Name[en_US] = pycharm
```

其中`Icon` 的路径就是`pycharm.png`的路径，后加`/pycharm.png`，

  `Exec` 的路径是 `sh` 加上`pycharm.png`的路径，后再加上`/pycharm.sh`

编写完成后，对文件进行保存，保存文件后，点击左下角的显示应用程序，就能看到PyCharm的图标。单击鼠标右键，选择添加到收藏夹，图标就能再左侧的显示栏中显示出来