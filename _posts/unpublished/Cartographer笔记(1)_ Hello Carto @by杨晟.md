# Cartographer笔记(1)_ Hello Carto @by杨晟
标签（空格分隔）： cartographer

---
# 目录
[TOC]

## 任务目标
1. 搞定三种(四种)类型的输入。
  - VLP-16 转 MultiEchoLaserScan / PointCloud2
  - depthimage 转 MultiEchoLaserScan / PointCloud2
  - VLP-16 转 LaserScan
  - depthimage 转 LaserScan

2. 比一比不同情况下的结果。

## 数据
- 目前可以用的数据。
  - cartographer提供的数据，例如cartographer_turtlebot这个demo bag包含LaserScan和depthimage，这份数据就能做2D(Depth->LaserScan), 3D(Depth->PointCloud2), 原生态LaserScan一共三类输入的比较。
  - cartographer提供的的其他数据中，topic为/horizontal_laser_2d和/vertical_laser_2d的均为MultiEchoLaserScan的类型。

- 下次组会我问问松老师看能不能直接把VLP-16设备拿过来，绑在turtlebot上扫几个bag下来做实验。

# ROS的基本机制
官方tutorial不算太臭但是很长，挑我们最关心的说：
## Node和Topic
ros平台中，每一个程序都会作为一个node，node和node之间**广义**上有三种传递机制：
- Topic: 最为广泛重要常用的**单向异步**通讯模式。在两个或者多个node之间架"桥"，数据总是由publisher到subscriber，通过队列缓冲。一个Topic为一种数据类型，像上面的/laser_scan, /camera/depth/image_raw全是Topic，它们的类型为sensor_msgs/LaserScan, sensor_msgs/MultiEchoLaserScan等。
```bash
roscore                  # [新终端]不用roslaunch的情况下，需要手动启动，全平台唯一
rosbag play <bag_file>   # [新终端]回放，会把录制时发过的Topics全部照搬
rosnode list             # [一次性]查看当前活动的所有Nodes
rostopic list            # [一次性]查看当前活动的所有Topics
rostopic type /horizontal_laser_2d               # [一次]查看数据类型
rostopic type /horizontal_laser_2d | rosmsg show # [一次]查看数据类型细节
```
- Service: **双向同步**通讯模式，request方会一直苦苦等待response。常用于发一些指令，如紧急刹车，Reset等。
- Action: **双向异步**通讯模式，request方不会阻塞，可以问询处理状态和撤销，会开出相当多的Nodes和Topics实现这个通讯模式。
后面两种相对于我们的目标比较深，只做了解，查看当前的所有Nodes和Topics:
```bash
rqt_graph    # 不嫌麻烦敲rosrun rqt_graph rqt_graph也行，其他rqt_类似。
```

## 搞定第一步的关键——launch文件
使用launch文件可以一次性launch一大堆的node，也就是rosrun一系列程序。并且可以使用remap修改Topic的名字使得大家可以链接起来(一般而言Topic名字是写死在程序文件里面的。)
[[官方Wiki，重要性：2]](http://wiki.ros.org/roslaunch)

## 写与调试launch文件
完成输入对接应该不需要写源码，只需要写个比较出色的launch文件就可以，如果需要改动源码，在src的上一级目录(catkin workspace)执行catkin_make就行，cartographer项目需要catkin_make_isolated。
改好launch文件后，测试：
```bash
roslaunch cartographer_ros <launch_file>  # 每一个都可以TAB补全，前提见下面。
```
在执行上一步之前，要在~/.bashrc末尾加上：
```bash
export EDITOR='subl'    # 默认gedit

source /opt/ros/indigo/setup.bash
source /home/shengyang/workspace/ros/cartographer/install_isolated/setup.bash
```
这样就可以使用roscd rosls roscat rosed(打开编辑器，其他命令去掉ros即可理解)等命令，所有命令的格式都是
```bash
rosed <package_name> <file> # 均可TAB补全
```

## package组织方式(科普向，不重要)
因为暂时不会深入改cartographer的代码，所以只是列举一下：
```text
catkin_ws/         # catkin workspace, 
  ...              # catkin_make的产物，记得要source上，一般为devel/setup.bash。
  src/             # packages源码呆的地方
    CMakeLists.txt # catkin_init_workspace会在这里出现一个CMakeLists.txt。方便在上一级catkin_make
    ...            # 其他包
    package_i/
      package.xml     # package info，方便ROS和PM组织管理
      CMakeLists.txt  # 编译向，使用ros catkin初始化package会自动生成一个模板      
      各种文件夹        # 比如launch下的launch，src下的src等。
```

## TF什么鬼
[[Wiki，重要性：2]](https://wiki.ros.org/tf) 。意思就是local-coordinates。用turtlebot会自带，关键是他们给的pcap没有这个。

## Nodelet
[[Wiki，重要性：3]](https://wiki.ros.org/nodelet)。这个可以减少Topic拷贝内存的开销，launch中应该带这个，Velodyne系中就用到了nodelet，可以参考，具体见下节。

# launch可以用到的包
这部分需要一起找一找，主要是会用到一些倒数据格式的包，已经找到的有：
## [[Cartographer]](https://google-cartographer-ros.readthedocs.io/en/latest/ros_api.html)
  - Subscriber: scan(**LaserScan**, 2D, 单线"激光"), echoes(**MultiEchoLaserScan**, "2D", 多线"激光"), cloud(**PointCloud2**, 3D, 点云)。
  - 附加Subs: imu(惯导，3D必备), odom(测距)
  - TF-trans: 
  - Publisher: map, matched_points, submap_list
  - 说明: 我们主要是需要完成Subscriber的对接，把正确的Message Type通过launch给remap通。主源代码是node_main.cc，可以看出当输入为scan, echoes时，都会作为2D Scenario处理，否则为3D Scenario。

## [[Velodyne]](http://wiki.ros.org/velodyne)
包含两个关键node： velodyne_driver和velodyne_pointcloud，目前来看：
[[velodyne_driver]](http://wiki.ros.org/velodyne_driver)：
 - 输入（无Subscriber）: 激光数据(pcap文件或者硬件)。
 - Publisher: velodyne_packets(类型为VelodyneScan)。

[[velodyne_pointcloud]](http://wiki.ros.org/velodyne_pointcloud)：
 - Subscriber: velodyne_packets(driver的输出)
 - Publisher: points(**PointCloud2**)
 - 这部分用到的Nodelet可以参考链接中的第3节(launch File Examples)

## DepthMap
跟上面一样，有两部：第一步是拿到/camera/depth/image_raw和cameraInfo，深度图，turtlebot自带launchfile包干。第二步:
[[depthimage_to_laserscan]](http://wiki.ros.org/depthimage_to_laserscan)
 - Subscriber: image, camerainfo。
 - Publisher: scan(**LaserScan**)。

# 总结
写一个或者几个launch，搞定如下几种输入：
1. 在线(turtlebot) /离线(bag, pcap, video)
2. 基于VLP-16和DepthMap，给cartographer传三种数据：**LaserScan**, **MultiEchoLaserScan**, **PointCloud2**。

launch中用到的packages:
- Cartographer系列(包含ros和turtlebot)
- turtlebot_bringup (在线 / 离线DepthCamera)
- Velodyne_driver and Velodyne_pointcloud (在线，离线VLP-16)
- pointcloud_to_laserscan
- depthimage_to_laserscan
- *待查*  ?_to_multiEchoLaserScan

方法:
```text
VLP-16 -> velodyne_driver -> velodyne_pointcloud
                           ... -> pointcloud_to_laserscan
                           ... -> ? -> MultiEchoLaserScan # 调研
Depth -> bringup -> pointcloud2
                 -> laserScan
                 -> ? -> MultiEchoLaserScan               # 调研
```




