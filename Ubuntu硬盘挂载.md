Ubuntu硬盘挂载

https://www.jb51.net/article/176972.htm

https://blog.csdn.net/u011932817/article/details/102878605

命令：

```shell
sudo fdisk -l # 查看有哪些硬盘
```

在**/home/用户名**创建一个挂载点，即创建一个空的文件夹，比如**mkdir /home/用户名/local**

Ubuntu系统会自动将插入的硬盘挂载到某处，挂载前需要先解开挂载点，操作如下：

```shell
df -kh # 查看硬盘在哪里
sudo umount /dev/sdbx  # 解开挂载  此处的sdax应写入上面得到的内容,此处我的为sdb5
```

下面开始挂载

```shell
sudo blkid /dev/sdb5     # 首先得到盘符的信息
```

![image-20221111143736437](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221111143736437.png)

一定要记住这个`UUID`以及`文件类型`，这里的`UUID为0CB5853192855A30`，文件类型为`ntfs`。

然后，执行

```shell
sudo get /etc/fstab
```

在它的下面添加以下内容

```txt
UUID=0CB5853192855A30 /home/用户名/local ntfs   defaults    0     2
```

**第一列为UUID，第二列为挂载目录（该目录必须为一个空白目录），第三列为文件系统类型（一定要看清楚是什么类型，有可能是ntfs类型或者ext4类型等），第四列为参数，第五列0表示不备份，最后一列必须为2或者0（除非引导分区为1），其他情况一定要输入2**

修改好后，保存退出。

再执行

```shell
sudo mount -a # 最后进行自动挂载
# sudo nautilus
chmod 777 /home/用户名/local
# 下面是对根目录赋给权限
sudo chmod 777 /
```

