---
title: 谷歌云(GCP,Google Cloud Platform)挂载新磁盘
date: 2020-05-27
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
```
root@instance-1:~# parted                # 使用parted工具进入查看
GNU Parted 3.2
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print list                      # 输入 print list 列出磁盘信息
Model: Google PersistentDisk (scsi)
Disk /dev/sda: 10.7GB                    # 默认的、容量为10GB的磁盘sda
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
14      1049kB  5243kB  4194kB                     bios_grub
15      5243kB  116MB   111MB   fat32              boot, esp
 1      116MB   10.7GB  10.6GB  ext4

Error: /dev/sdb: unrecognised disk label
Model: Google PersistentDisk (scsi)
Disk /dev/sdb: 53.7GB                    # 新建的、容量为50GB的磁盘sdb
Sector size (logical/physical): 512B/4096B
Partition Table: unknown
Disk Flags:

(parted) quit                            # 输入 quit 退出parted
```

从上面可以看出默认的实例的磁盘为sda，容量为10GB；新建的磁盘为sdb，容量为50GB。

PS：也可以用lsblk来查看磁盘信息。

```
root@instance-1:~# lsblk                 # 使用lsblk查看磁盘信息
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0     7:0    0 114.5M  1 loop /snap/google-cloud-sdk/132
loop1     7:1    0  93.9M  1 loop /snap/core/9066
loop2     7:2    0  98.4M  1 loop /snap/google-cloud-sdk/129
loop3     7:3    0    55M  1 loop /snap/core18/1754
sda       8:0    0    10G  0 disk        # 默认的、容量为10GB的磁盘sda
├─sda1    8:1    0   9.9G  0 part /
├─sda14   8:14   0     4M  0 part
└─sda15   8:15   0   106M  0 part /boot/efi
sdb       8:16   0    50G  0 disk        # 新建的、容量为50GB的磁盘sdb
```

2. 将磁盘sdb格式化为ext4格式。
```
root@instance-1:~# mkfs.xfs -f /dev/sdb  #“sdb”根据你挂载的磁盘名称不同而进行相应修改
meta-data=/dev/sdb               isize=512    agcount=4, agsize=3276800 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=0, rmapbt=0, reflink=0
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=6400, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

## 将磁盘挂载为分区

1. 格式化后就是挂载分区，由于打算当做下载盘使用，用于文件的保存以及实现Google Drive文件的上传，所以将该磁盘挂载到/root/Download0/文件夹。

`root@instance-1:~# mount /dev/sdb /root/Download0/`

2. 修改读写权限。

`root@instance-1:~# chmod a+w /root/Download0/`

3. 查看是否成功。
```
root@instance-1:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            274M     0  274M   0% /dev
tmpfs            59M  792K   58M   2% /run
/dev/sda1       9.6G  4.7G  4.9G  49% /
tmpfs           293M     0  293M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           293M     0  293M   0% /sys/fs/cgroup
/dev/loop1       94M   94M     0 100% /snap/core/9066
/dev/loop2       99M   99M     0 100% /snap/google-cloud-sdk/129
/dev/loop3       55M   55M     0 100% /snap/core18/1754
/dev/loop0      115M  115M     0 100% /snap/google-cloud-sdk/132
/dev/sda15      105M  3.6M  101M   4% /boot/efi
emby:           1.0P     0  1.0P   0% /home/gdrive
tmpfs            59M     0   59M   0% /run/user/0
/dev/sdb         50G   84M   50G   1% /root/Download0      # sdb已经挂在到文件夹Download0上
```

4. 设置开机自动挂载sdb分区。

`root@instance-1:~# vim /etc/fstab        # 使用vim工具对文件fstab进行编辑`

在文件fstab的末位加入下面一行代码：

`/dev/sdb /root/Download0/ xfs defaults 0 0      # 各字段分别是分区、挂载点、文件格式、挂载参数。后两个一般用0 0`
