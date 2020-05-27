---
layout: post
title: "谷歌云(GCP,Google Cloud Platform)挂载新磁盘"
date: 2020-05-27
categories: tutorial
tags: [GCP,Google Cloud Platform,谷歌,Google,教程]
image: 
---
> 本文参考文章 [谷歌云gcp（centos7）挂载额外的新磁盘](https://www.nmbhost.com/archives/5063)并进行了一些改动。

## 一、前言
去年薅羊毛薅了谷歌云一年300美元的免费试用金，一直没怎么用，最近因为[COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019)疫情在家无聊，所以又开始捣鼓起来了。现在想给GCP中的虚拟机实例添加新硬盘，因为之前新建实例的时候没想到需要用来下载视频资源，实例中只有默认的10GB容量，于是需要新建一个磁盘并且挂载在这个实例下，用于下载的视频资源的保存。

## 二、新建磁盘

1、先在Computer Engine里选“磁盘”，然后选择“创建磁盘”。

![](https://wx1.sinaimg.cn/large/6a8c0fe1gy1gf68y7uq4vj20tw0an0u4.jpg)

2、 来到Computer Engine里的虚拟机(VM,Virtual Machine)实例，点开你需要挂载硬盘的实例，然后选择修改，并在“额外磁盘”中选择“附加现有磁盘”以添加这个新磁盘，最后重启（貌似不用重启也可以）。挂载之后新磁盘将隶属于这个实例。

![](https://wx1.sinaimg.cn/large/6a8c0fe1gy1gf68xxwle9j20mk07njrt.jpg)

## 三、配置并初始化新盘

1、 使用SSH连接添加了新磁盘的实例，并查看实例的文件系统。

```
root@instance-1:~# parted# 使用parted工具进入查看
GNU Parted 3.2
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print list# 输入 print list 列出磁盘信息
Model: Google PersistentDisk (scsi)
Disk /dev/sda: 10.7GB# 默认的、容量为10GB的磁盘sda
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
14      1049kB  5243kB  4194kB                     bios_grub
15      5243kB  116MB   111MB   fat32              boot, esp
 1      116MB   10.7GB  10.6GB  ext4

Error: /dev/sdb: unrecognised disk label
Model: Google PersistentDisk (scsi)
Disk /dev/sdb: 53.7GB# 新建的、容量为50GB的磁盘sdb
Sector size (logical/physical): 512B/4096B
Partition Table: unknown
Disk Flags:

(parted) quit# 输入 quit 退出parted
```

从上面可以看出默认的实例的磁盘为sda，容量为10GB；新建的磁盘为sdb，容量为50GB。

*PS：也可以用lsblk来查看磁盘信息。*

```
root@instance-1:~# lsblk# 使用lsblk查看磁盘信息
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0     7:0    0 114.5M  1 loop /snap/google-cloud-sdk/132
loop1     7:1    0  93.9M  1 loop /snap/core/9066
loop2     7:2    0  98.4M  1 loop /snap/google-cloud-sdk/129
loop3     7:3    0    55M  1 loop /snap/core18/1754
sda       8:0    0    10G  0 disk        # 默认的、容量为10GB的磁盘sda
├─sda1    8:1    0   9.9G  0 part /
├─sda14   8:14   0     4M  0 part
└─sda15   8:15   0   106M  0 part /boot/efi
sdb       8:16   0    50G  0 disk# 新建的、容量为50GB的磁盘sdb
```

2、 将磁盘sdb格式化为ext4格式。

## 四、将磁盘挂载为分区

1、格式化后就是挂载分区，由于打算当做下载盘使用，用于文件的保存以及实现Google Drive文件的上传，所以将该磁盘挂载到/root/Download0/文件夹。

`root@instance-1:~# mount /dev/sdb /root/Download0/`

2、修改读写权限。

`root@instance-1:~# chmod a+w /root/Download0/`

3、查看是否成功。

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
/dev/sdb         50G   84M   50G   1% /root/Download0# sdb已经挂在到文件夹Download0上
```

4、设置开机自动挂载sdb分区。

使用vim工具对文件fstab进行编辑：

`root@instance-1:~# vim /etc/fstab`

在文件fstab的末位加入下面一行代码：

`/dev/sdb /root/Download0/ xfs defaults 0 0# 各字段分别是分区、挂载点、文件格式、挂载参数。后两个一般用0 0`


























Ubuntu版本：16.04 LTS

## Jetson TX2 安装 ROS kinetic

[TX2 入门教程软件篇 - 安装 ROS kinetic](http://www.ncnynl.com/archives/201706/1750.html)

## Jetson TX2 安装 cartographer

google的官方教程：[https://google-cartographer-ros.readthedocs.io/en/latest/index.html](https://google-cartographer-ros.readthedocs.io/en/latest/index.html)

## 安装Rplidar激光雷达驱动

首先必须给Jetson TX2 编译内核，才可读取出雷达，编译方法参考 [http://m.blog.csdn.net/gzj2013/article/details/77069803](http://m.blog.csdn.net/gzj2013/article/details/77069803) 

注意：<br />
*1、要选择 CP210x 驱动，而不是例子中的 CH341*<br />
*2、必须保证Jetson TX2 中有足够的空间（大于 5G）才可正常安装*

#### 开始安装雷达驱动

在[https://github.com/robopeak/rplidar_ros](https://github.com/robopeak/rplidar_ros) 下载安装 rplidar 的 ROS 驱动：

1、在 `~/catkin_ws/src`下运行：<br />
`nvidia@tegra-ubuntu:~$ git clone https://github.com/robopeak/rplidar_ros`

2、重新编译，执行以下代码：<br />
`catkin_make_isolated --install --use-ninja `

3、设置 USB 口的权限：<br />
`nvidia@tegra-ubuntu:~$ ls -l /dev |grep ttyUSB`<br />
`nvidia@tegra-ubuntu:~$ sudo chmod 666 /dev/ttyUSB0`

每次插上 rplidar 都要设置一遍 USB 口的权限，可以将权限写在 rules 里（[参考这个](http://cyy4409.blog.163.com/blog/static/554042892013113153739289/)）：
root 权限新建文件` /etc/udev/rules.d/70-ttyusb.rules`

写入 `KERNEL=="ttyUSB[0-9]*", MODE="0666"`，保存，重新插上设备，在运行一次·`ls -l /dev |grep ttyUSB*` 查看权限，有两个 rw 就可以了

4、运行<br />
驱动里提供了三个 launch 文件，跑 cartographer 的话用 rplidar.launch 即可<br />
`nvidia@tegra-ubuntu:~$ roslaunch rplidar_ros rplidar.launch`<br />
运行完可以先关掉

## 测试 cartographer

 运行 cartographer，首先 编辑用于运行 rplidar 的配置文件：

1、 修改revo_lds_.lua：<br />
位于`/catkin_ws/src/cartographer_ros/cartographer_ros/configuration_files `文件夹下，修改了 22 和 23 行。
```
-- Copyright 2016 The Cartographer Authors
--
-- Licensed under the Apache License, Version 2.0 (the "License");
-- you may not use this file except in compliance with the License.
-- You may obtain a copy of the License at
--
--      http://www.apache.org/licenses/LICENSE-2.0
--
-- Unless required by applicable law or agreed to in writing, software
-- distributed under the License is distributed on an "AS IS" BASIS,
-- WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
-- See the License for the specific language governing permissions and
-- limitations under the License.
 
include "map_builder.lua"
include "trajectory_builder.lua"
 
options = {
  map_builder = MAP_BUILDER,
  trajectory_builder = TRAJECTORY_BUILDER,
  map_frame = "map",
  tracking_frame = "laser",
  published_frame = "laser",
  odom_frame = "odom",
  provide_odom_frame = true,
  use_odometry = false,
  num_laser_scans = 1,
  num_multi_echo_laser_scans = 0,
  num_subdivisions_per_laser_scan = 1,
  num_point_clouds = 0,
  lookup_transform_timeout_sec = 0.2,
  submap_publish_period_sec = 0.3,
  pose_publish_period_sec = 5e-3,
  trajectory_publish_period_sec = 30e-3,
  rangefinder_sampling_ratio = 1.,
  odometry_sampling_ratio = 1.,
  imu_sampling_ratio = 1.,
}
 
MAP_BUILDER.use_trajectory_builder_2d = true
 
TRAJECTORY_BUILDER_2D.submaps.num_range_data = 35
TRAJECTORY_BUILDER_2D.min_range = 0.3
TRAJECTORY_BUILDER_2D.max_range = 8.
TRAJECTORY_BUILDER_2D.missing_data_ray_length = 1.
TRAJECTORY_BUILDER_2D.use_imu_data = false
TRAJECTORY_BUILDER_2D.use_online_correlative_scan_matching = true
TRAJECTORY_BUILDER_2D.real_time_correlative_scan_matcher.linear_search_window = 0.1
TRAJECTORY_BUILDER_2D.real_time_correlative_scan_matcher.translation_delta_cost_weight = 10.
TRAJECTORY_BUILDER_2D.real_time_correlative_scan_matcher.rotation_delta_cost_weight = 1e-1
 
POSE_GRAPH.optimization_problem.huber_scale = 1e2
POSE_GRAPH.optimize_every_n_nodes = 35
POSE_GRAPH.constraint_builder.min_score = 0.65
 
return options
```

2、修改demo_revo_lds.launch ：<br />
位于`~/catkin_ws/src/cartographer_ros/cartographer_ros/launch `文件夹下，注意 23 和 25 行。<br />
主要修改两个地方，`<remap from="scan"to="horizontal_laser_2d"/>` 这一句后面的 `horizontal_laser_2d `改成 `scan `；最后的 playbag 节点删掉。
```
<!--
  Copyright 2016 The Cartographer Authors
 
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at
 
       http://www.apache.org/licenses/LICENSE-2.0
 
  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
 
<launch>
  <param name="/use_sim_time" value="true" />
 
  <node name="cartographer_node" pkg="cartographer_ros"
      type="cartographer_node" args="
          -configuration_directory $(find cartographer_ros)/configuration_files
          -configuration_basename revo_lds.lua"
      output="screen">
    <remap from="scan" to="scan" />
  </node>
 
  <node name="cartographer_occupancy_grid_node" pkg="cartographer_ros"
      type="cartographer_occupancy_grid_node" args="-resolution 0.05" />
 
  <node name="rviz" pkg="rviz" type="rviz" required="true"
      args="-d $(find cartographer_ros)/configuration_files/demo_2d.rviz" />
 
</launch>
```
3、重新编译
```
nvidia@tegra-ubuntu:~$ cd ~/catkin_ws
nvidia@tegra-ubuntu:~/catkin_ws$ catkin_make_isolated --install --use-ninja
```
4、运行<br />

启动两个终端，分别运行 rplidar 的节点和 cartographer_ros 节点：<br />
第一个终端：<br />
`nvidia@tegra-ubuntu:~$ roslaunch rplidar_ros rplidar.launch `<br />
第二个终端：<br />
`nvidia@tegra-ubuntu:~$ roslaunch cartographer_ros demo_revo_lds.launch`



<iframe src="//player.bilibili.com/player.html?aid=27473804&cid=47390597&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="1024" height="720"> </iframe>

