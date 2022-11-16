anaconda环境配置

直接安装

```bash
bash Anaconda3.7-Linux-x86_64.sh
```

安装了anaconda之后打开terminal

```shell
gedit  ~/.bashrc
```

打开以后，在最后一行下面输入以下内容，用户名替换为自己的用户名

```bash
export PATH="/home/用户名/anaconda3/bin:$PATH"
```

保存退出后，再终端输入

```shell
source ~/.bashrc
```

