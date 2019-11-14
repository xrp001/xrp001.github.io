---
layout: post
title: "TX2使用Rplidar A2实现Cartographer"
date: 2018-05-18
categories: tutorial
tags: [cartographer,ros,rplidar,教程]
image: 
---
参考：

[TX2+rplidar+cartographer](https://blog.csdn.net/zong596568821xp/article/details/77678693)<br />
[实时 Cartographer 测试 (1) - rplidar](http://www.cnblogs.com/yhlx125/p/8078697.html)<br />
[rplidar 跑 cartographer](http://www.cnblogs.com/liangyf0312/p/8028441.html)<br />
[ROS 下使用 rplidar 运行 google cartographer](https://blog.csdn.net/ywj447/article/details/52922487)

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


<iframe src="//player.bilibili.com/player.html?aid=27473804&cid=47390597&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="330px"></iframe>



