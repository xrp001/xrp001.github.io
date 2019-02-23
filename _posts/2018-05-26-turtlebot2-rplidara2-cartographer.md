---
layout: post
title: "使用Turtlebot 2和Rplidar A2实现Cartographer"
date: 2018-05-26
categories: tutorial
tags: [cartographer,ros,turtlebot,rplidar,教程]
image: 
---
参考：<br />
[TX2+rplidar+cartographer](https://blog.csdn.net/zong596568821xp/article/details/77678693)<br />
[创客智造-ROS 与 SLAM 入门教程 - cartographer 在 Turltlebot 的应用 3 - 构建地图](https://www.ncnynl.com/archives/201702/1371.html)<br />
[Turtlebot 入门教程 - 激光雷达 (Rplidar)gmapping 构建地图](https://www.ncnynl.com/archives/201611/1095.html)<br />
[创客智造-Turtlebot 入门 - 创建地图](https://www.ncnynl.com/archives/201609/797.html)<br />
[创客智造-Turtlebot 入门 - 遥控](https://www.ncnynl.com/archives/201609/795.html)<br />

主机：Jetson TX2
Ubuntu版本：16.04 LTS

Jetson TX2主要的软件安装在[Jetson TX2 使用 Rplidar A2 实现 Cartographer](https://www.jianshu.com/p/07700e0ba224)已经介绍了，这篇就记录一下如何和Turtlebot2搭配使用。
## 一：安装cartographer_turtlebot
*前提：ROS、cartographer、cartographer_ros、rplidar_ros等都已经安装*

官方教程：https://google-cartographer-ros-for-turtlebots.readthedocs.io/en/latest/
```
# Install wstool and rosdep.
sudo apt-get update
sudo apt-get install -y python-wstool python-rosdep ninja-build

# Create a new workspace in 'catkin_ws'.
mkdir catkin_ws
cd catkin_ws
wstool init src

# Merge the cartographer_turtlebot.rosinstall file and fetch code for dependencies.
wstool merge -t src https://raw.githubusercontent.com/googlecartographer/cartographer_turtlebot/master/cartographer_turtlebot.rosinstall
wstool update -t src

# Install deb dependencies.
# The command 'sudo rosdep init' will print an error if you have already
# executed it since installing ROS. This error can be ignored.
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro=${ROS_DISTRO} -y

# Build and install.
catkin_make_isolated --install --use-ninja
source install_isolated/setup.bash
```
## 二：安装turtlebot_apps
需要用到turtlebot_navigation,turtlebot_navigation在turtlebot_apps里面，所以需要安装turtlebot_apps
```
$ cd   ~/catkint_ws/src
# turtlebot建图依赖包
$ git clone https://github.com/turtlebot/turtlebot_apps 
#编译
$ cd  ~/catkin_ws
$ catkin_make_isolated --install --use-ninja
```
添加环境变量，在~/.bashrc 最后添加一行：
`source ~/catkin_ws/install_isolated/setup.bash`
刷新配置
`$ source ~/.bashrc`
或
`$ rospack profile`
## 三、制作雷达驱动启动文件(重点)
1、复制 rplidar.launch 到 rplidar-laser.launch
```
$ roscd turtlebot_navigation
$ mkdir -p laser/driver
$ sudo cp ~/catkin_ws/src/rplidar_ros/launch/rplidar.launch laser/driver/rplidar_laser.launch
```
2、打开 rplidar_laser.launch进行修改

检查 frame_id 是否指定为 laser:
`<param name="frame_id" type="string" value="laser"/>`
<br />
3、增加 TF 定义
```
<node pkg="tf" type="static_transform_publisher" name="base_to_laser" args="0.0 0.0 0.18 0 0.0 0.0 base_link laser 100"/>
```
4、重新编译
```
#重新编译
$ cd ~/catkin_ws
$ catkin_make_isolated --install --use-ninja
```
*  修改为 args="0.0 0.0 0.18 0 0.0 0.0 为自己的实际安装位置。[详情查看](http://wiki.ros.org/tf/),static_transform_publisher 部分

*   static_transform_publisher x y z qx qy qz qw frame_id child_frame_id period_in_ms

*   这里我假设底盘的中心点为 0，雷达放在机器人托盘中心位置，X 为 0，高度为 18CM,Z 为 0.18m

*   TF 的单位使用米的，测量单位是 CM

*   rplidar_laser.launch修改后的完整代码如下：

```
<launch>
  <node name="rplidarNode"          pkg="rplidar_ros"  type="rplidarNode" output="screen">
  <param name="serial_port"         type="string" value="/dev/ttyUSB0"/>  
  <param name="serial_baudrate"     type="int"    value="115200"/>
  <param name="frame_id"            type="string" value="laser"/>
  <param name="inverted"            type="bool"   value="false"/>
  <param name="angle_compensate"    type="bool"   value="true"/>
  </node>

  <node pkg="tf" type="static_transform_publisher" name="base_to_laser" args="0.0 0.0 0.18 0 0.0 0.0 base_link laser 100"/>
</launch>
```
## 四：使用cartographer_turtlebot构建地图
1、增加 turtlebot_lidar.launch
```
$ cd ~/catkin_ws/src/cartographer_turtlebot/cartographer_turtlebot/launch
$ touch turtlebot_lidar.launch 
$ vim turtlebot_lidar.launch 
```
写入如下代码：
```
<launch>
  <arg name="configuration_basename"/>

  <include file="$(find turtlebot_bringup)/launch/minimal.launch" />

  <node name="cartographer_node" pkg="cartographer_ros"
      type="cartographer_node" args="
          -configuration_directory
              $(find cartographer_turtlebot)/configuration_files
          -configuration_basename $(arg configuration_basename)"
      output="screen">
    <remap from="scan" to="/scan" />
  </node>

  <node name="flat_world_imu_node" pkg="cartographer_turtlebot"
      type="cartographer_flat_world_imu_node" output="screen">
    <remap from="imu_in" to="/mobile_base/sensors/imu_data_raw" />
    <remap from="imu_out" to="/imu" />
  </node>

  <node name="rviz" pkg="rviz" type="rviz" required="true"
      args="-d $(find cartographer_turtlebot
          )/configuration_files/demo_turtlebot.rviz" />

</launch>
```
2、新增启动文件 turtlebot_lidar_2d.launch
```
$ cd ~/catkin_ws/src/cartographer_turtlebot/cartographer_turtlebot/launch
$ touch turtlebot_lidar_2d.launch
$ vim turtlebot_lidar_2d.launch
```
写入如下代码：
```
<launch>
  <include file="$(find cartographer_turtlebot)/launch/turtlebot_lidar.launch">
    <arg name="configuration_basename" value="turtlebot_urg_lidar_2d.lua" />
  </include>
</launch>
```
3、重新编译
```
#重新编译
$ cd ~/catkin_ws
$ catkin_make_isolated --install --use-ninja
```
4、启动Rplidar A2雷达<br />
打开一个新终端，输入：<br />
`$ roslaunch turtlebot_navigation rplidar_laser.launch`

5、启动cartographer_turtlebot建地图<br />
再打开一个新终端，输入：<br />
`$ roslaunch cartographer_turtlebot turtlebot_lidar_2d.launch`

6、使用键盘操作turtlebot2移动<br />
又打开一个新终端，输入：<br />
`$ roslaunch turtlebot_teleop keyboard_teleop.launch`<br />
按提示利用键盘控制turtlebot2移动建图。<br />
具体以及更多控制方式可参考：[创客智造-Turtlebot 入门 - 遥控](https://www.ncnynl.com/archives/201609/795.html)

## 五、保存地图
```
#新建map文件夹用于保存地图
$ mkdir -p ~/map
#启动存图，并将名为lidar_2d的地图文件保存在map文件夹
$ rosrun map_server map_saver -f ~/map/lidar_2d
#查看内容，包含lidar_2d.pgm  lidar_2d.yaml
$ ls ~/map   
```
大功告成！！！！
