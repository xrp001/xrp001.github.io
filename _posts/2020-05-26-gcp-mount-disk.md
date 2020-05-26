---
title: # 谷歌云(GCP,Google Cloud Platform)挂载新磁盘
date: 2020-05-26
categories: tutorial
description: 本文主要讲述如何为GCP中的虚拟机实例添加新硬盘。
tags: [GCP,Google Cloud Platform,谷歌,Google]
---
>本文参考文章 [谷歌云gcp（centos7）挂载额外的新磁盘](https://www.nmbhost.com/archives/5063)并进行了一些改动。
## 前言
去年薅羊毛薅了谷歌云一年300美元的免费试用金，一直没怎么用，最近因为[COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019)疫情在家无聊，所以又开始捣鼓起来了。现在想给GCP中的虚拟机实例添加新硬盘，因为之前新建实例的时候没想到需要用来下载视频资源，实例中只有默认的10GB容量，于是需要新建一个磁盘并且挂载在这个实例下，用于下载的视频资源的保存。
## 新建磁盘
1. 先在Computer Engine里选“磁盘”，然后选择“创建磁盘”。
![new disk](https://wx1.sinaimg.cn/large/6a8c0fe1gy1gf68y7uq4vj20tw0an0u4.jpg)
2. 去到Computer Engine里的虚拟机(VM,Virtual Machine)实例，点开你需要挂载硬盘的实例，然后选择修改，并在“额外磁盘”中选择“附加现有磁盘”以添加这个新磁盘，最后重启（貌似不用重启也可以）。挂载之后新磁盘将隶属于这个实例。
![输入图片描述](https://wx1.sinaimg.cn/large/6a8c0fe1gy1gf68xxwle9j20mk07njrt.jpg)

## 配置并初始化新盘

1. 使用SSH连接添加了新磁盘的实例，并查看实例的文件系统。
`
root@instance-1:~# parted
GNU Parted 3.2
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print list
Model: Google PersistentDisk (scsi)
Disk /dev/sda: 10.7GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:
`

Number  Start   End     Size    File system  Name  Flags
14      1049kB  5243kB  4194kB                     bios_grub
15      5243kB  116MB   111MB   fat32              boot, esp
 1      116MB   10.7GB  10.6GB  ext4


Error: /dev/sdb: unrecognised disk label
Model: Google PersistentDisk (scsi)
Disk /dev/sdb: 53.7GB
Sector size (logical/physical): 512B/4096B
Partition Table: unknown
Disk Flags:

(parted) quit



[root@instance-5 ~]parted #使用 parted工具进入查看
print list 
Model: Google PersistentDisk (scsi)
Disk /dev/sda: 10.7GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags: 

Number Start End Size Type File system Flags
1 1049kB 10.7GB 10.7GB primary xfs boot


Error: /dev/sdb: unrecognised disk label
Model: Google PersistentDisk (scsi) 
Disk /dev/sdb: 537GB
Sector size (logical/physical): 512B/4096B
Partition Table: unknown
Disk Flags: 

quit #退出parted
从上面可以看出sda是10g大小的xfs，而sdb就是500g的unknown，接下来就是对sdb的操作

3.2.1由于已经知道了新的盘是sdb，当然你也可以用lsblk来查看

[root@instance-5 ~] lsblk
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sdb 8:16 0 500G 0 disk 
sda 8:0 0 10G 0 disk 
└─sda1 8:1 0 10G 0 part /

3.2.2格式化sdb  
我这里是直接格式化成ext4的格式

mkfs.xfs -f /dev/sdb #sdb根据你挂载的磁盘不同而不同
3.3格式化后就是挂载分区，由于我是打算用来做tr的缓存盘后上传到google drive的，所以我直接挂载到tr的temp文件夹上

mount /dev/sdb /home/transmission/tmp 
chmod a+w /home/transmission/tmp 
#[DEVICE_ID]：sdb
#/home/transmission/tmp tr的缓存目录
#chmod读写权限
df -h #查看是否成功
3.4 设置开机自动挂载sdb分区

[root@instance-5 ~]# vim /etc/fstab #然后在文件末位添加如下
/dev/sdb /home/transmission/tmp xfs defaults 0 0
#各字段分别是分区，挂载点，文件格式，挂载参数。后两个一般用0 0
