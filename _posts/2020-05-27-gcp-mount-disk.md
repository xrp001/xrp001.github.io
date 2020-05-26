---
title: 谷歌云(GCP,Google Cloud Platform)挂载新磁盘
date: 2020-05-27
categories: tutorial
description: 本文主要讲述如何为GCP中的虚拟机实例添加新硬盘。
tags: [GCP,Google Cloud Platform,谷歌,Google]
---
> 1本文参考文章 [谷歌云gcp（centos7）挂载额外的新磁盘](https://www.nmbhost.com/archives/5063)并进行了一些改动。

## 前言
去年薅羊毛薅了谷歌云一年300美元的免费试用金，一直没怎么用，最近因为[COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019)疫情在家无聊，所以又开始捣鼓起来了。现在想给GCP中的虚拟机实例添加新硬盘，因为之前新建实例的时候没想到需要用来下载视频资源，实例中只有默认的10GB容量，于是需要新建一个磁盘并且挂载在这个实例下，用于下载的视频资源的保存。

## 新建磁盘

1. 先在Computer Engine里选“磁盘”，然后选择“创建磁盘”。

![](https://wx1.sinaimg.cn/large/6a8c0fe1gy1gf68y7uq4vj20tw0an0u4.jpg)

2. 去到Computer Engine里的虚拟机(VM,Virtual Machine)实例，点开你需要挂载硬盘的实例，然后选择修改，并在“额外磁盘”中选择“附加现有磁盘”以添加这个新磁盘，最后重启（貌似不用重启也可以）。挂载之后新磁盘将隶属于这个实例。

![](https://wx1.sinaimg.cn/large/6a8c0fe1gy1gf68xxwle9j20mk07njrt.jpg)

## 配置并初始化新盘

1. 使用SSH连接添加了新磁盘的实例，并查看实例的文件系统。
```
root@instance-1:~# parted# 使用parted工具进入查看
```

从上面可以看出默认的实例的磁盘为sda，容量为10GB；新建的磁盘为sdb，容量为50GB。

PS：也可以用lsblk来查看磁盘信息。

```
root@instance-1:~# lsblk# 使用lsblk查看磁盘信息
```

2. 将磁盘sdb格式化为ext4格式。
```
root@instance-1:~# mkfs.xfs -f /dev/sdb#“sdb”根据你挂载的磁盘名称不同而进行相应修改
```

## 将磁盘挂载为分区

1. 格式化后就是挂载分区，由于打算当做下载盘使用，用于文件的保存以及实现Google Drive文件的上传，所以将该磁盘挂载到/root/Download0/文件夹。

`root@instance-1:~# mount /dev/sdb /root/Download0/`

2. 修改读写权限。

`root@instance-1:~# chmod a+w /root/Download0/`

3. 查看是否成功。
```
root@instance-1:

```

4. 设置开机自动挂载sdb分区。

`root@instance-1:~# vim /etc/fstab# 使用vim工具对文件fstab进行编辑`

在文件fstab的末位加入下面一行代码：

`/dev/sdb /root/Download0/ xfs defaults 0 0# 各字段分别是分区、挂载点、文件格式、挂载参数。后两个一般用0 0`
