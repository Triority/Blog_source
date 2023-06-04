---
title: ros实操记录
tags:
  - 笔记
  - linux
  - python
  - ROS
index_img: /img/ros.png
categories:
- [计算机, 知识整理]
- 值得一提的文章
notshow: false
date: 2023-06-05 00:11:24
excerpt: 这部分文章是从最基础那篇文章分离出来的，不然那个太长了hhh
---
# 前言
这篇文章是从最基础那篇文章分离出来的，不然那个太长了找起东西太麻烦

同时也部分记录2023年的人工智能竞赛学习开发过程
# 前置任务(在比赛的车上获取传感器信息)
这一部分除了比赛那辆车以外都用不到，也就不详细分目录了，不想占用目录太多空间

> IMU

首先查看一下IMU的地址
```
ros-autocar@ros-autocar:~$ ls /dev/ttyUSB*
/dev/ttyUSB1  /dev/ttyUSB2  /dev/ttyUSB3
ros-autocar@ros-autocar:~$ ls /dev/ttyUSB*
/dev/ttyUSB0  /dev/ttyUSB1  /dev/ttyUSB2  /dev/ttyUSB3
```
有四个设备，拔掉imu之后再次查看少了那个就知道imu的地址了。给IMU的设备添加读写权限：
```
sudo chmod  +777 /dev/ttyUSB0
```
然后使用资料给的命令就可以读取imu的数值了
```
roslaunch imu_launch  imu_msg.launch
```
```
header: 
  seq: 18691
  stamp: 
    secs: 1685621438
    nsecs: 130887727
  frame_id: "base_link"
orientation: 
  x: 0.03581126779317856
  y: -0.006689701694995165
  z: -0.002406603889539838
  w: 0.999333381652832
orientation_covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
angular_velocity: 
  x: 5.624821674786508e-05
  y: -0.0020166519167128206
  z: -0.001579628805375099
angular_velocity_covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
linear_acceleration: 
  x: 0.19336825460195542
  y: 0.6750088781118393
  z: 9.727661895751954
linear_acceleration_covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
---
```

> 激光雷达

资料给了驱动雷达的功能包，启动雷达的launch命令：
这个launch文件有一个地方需要改动就是雷达串口设备号，我这一次改成是`/dev/ttyUSB2`如果启动文件之后雷达不转可以考虑是这个问题
```
roslaunch ls01b_v2 ls01b_v2.launch
```
然后在另外一个终端查看雷达数据：应该会看到满屏的数字hhh都是雷达扫描到的距离信息
```
rostopic echo /scan
```

> 编码器

同样，使用资料的驱动功能包：

```
roscore
rosrun encoder_driver Encoder_vel.py
rostopic echo /encoder
```
输出：
```
header: 
  seq: 11316
  stamp: 
    secs: 1685623435
    nsecs: 710257768
  frame_id: "odom"
child_frame_id: "base_footprint"
pose: 
  pose: 
    position: 
      x: 0.0
      y: 0.0
      z: 0.0
    orientation: 
      x: 0.0
      y: 0.0
      z: 0.0
      w: 0.0
  covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
twist: 
  twist: 
    linear: 
      x: 0.05552978515624999
      y: 0.0
      z: 0.0
    angular: 
      x: 0.0
      y: 0.0
      z: 0.0
  covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
---
```
其中`x: 0.05552978515624999`表示前进速度

> 整合及运动控制

```
roslaunch racecar Run_car.launch 
rosrun racecar racecar_teleop.py
rviz rviz
```
第一个启动了上述传感器的程序，第二个用于使用键盘发布信息控制底盘，最后是rviz用于显示雷达等信息。

其中雷达需要设置坐标变换，雷达的坐标是相对于底盘坐标定义的，rviz默认使用map坐标系显示，需要定义底盘坐标系和map坐标系的相对关系。
```
<node pkg="tf" type="static_transform_publisher" name="map_odom_broadcaster" args="0 0 0 0 0 0 /map /odom 100" />

```
其实临时也可以把rviz的frame改为雷达坐标系解决。

# 常用算法
## 激光里程计：rf2o_laser_odometry
这个功能包可以根据激光雷达两帧之间的位置差距计算车子实际走的距离。

[github链接](https://github.com/MAPIRlab/rf2o_laser_odometry)

下载的功能包放在src文件夹内，需要做一些配置和修改一些bug：

首先是修改launch文件的话题节点等名称，这里我是改成这样

`rf2o_laser_odometry.launch`
```
<launch>
 
  <node pkg="rf2o_laser_odometry" type="rf2o_laser_odometry_node" name="rf2o_laser_odometry" output="screen">
    <param name="laser_scan_topic" value="/scan"/>                         # 雷达发布数据的话题
    <param name="odom_topic" value="/odom" />                         # topic where tu publish the odometry estimations
    <param name="publish_tf" value="true" />                              # wheter or not to publish the tf::transform (base->odom)
    <param name="base_frame_id" value="/base_footprint"/>                       # frame_id (tf) of the mobile robot base. A tf transform      from the laser_frame to the base_frame is mandatory
    <param name="odom_frame_id" value="/odom" />                           # frame_id (tf) to publish the odometry estimations
    <param name="init_pose_from_topic" value="" />  # (Odom topic) Leave empty to start at point (0,0)
    <param name="freq" value="10.0"/>                                       # Execution frequency.
    <param name="verbose" value="true"/>                                  # verbose
  </node>
</launch>
```

接下来改一些bug：修改`src/CLaserOdometry2D.cpp`

292-298行修改:
```
//Inner pixels
if ((u>1)&&(u<cols_i-2))
{
if (dcenter > 0.f)
if (std::isfinite(dcenter) && dcenter > 0.f)
{
float sum = 0.f;
float weight = 0.f;

```
316-322行修改:
```
//Boundary
else
{
if (dcenter > 0.f)
if (std::isfinite(dcenter) && dcenter > 0.f)
{
float sum = 0.f;
float weight = 0.f;

```

打开底盘lanunch文件，并启动rf2o：
```
roslaunch rf2o_laser_odometry rf2o_laser_odometry.launch 
```
就可以看见终端输出的里程数据。这个方法得到的数据有些缺陷，就是在车子发生旋转时候是不准确的，因为雷达和车子都在旋转相对位置不准确。

## 传感器数据融合：robot localization/ekf

## 激光雷达建图：gmapping/cartographer

## 重定位：amcl

## 路径规划：全局/局部

### 全局：A*

### 局部：DWA，teb

#### 动态窗口法：DWA
> 全称为`dynamic window approach`，其原理主要是在速度空间（v,w）中采样多组速度，并模拟出这些速度在一定时间内的运动轨迹，并通过评价函数对这些轨迹进行评价，选取最优轨迹对应的速度驱动机器人运动

优缺点：考虑到速度和加速度的限制只有安全的轨迹会被考虑；可以实时避障但是避障效果一般；不适用于阿克曼模型车模；每次都选择下一步的最佳路径而非全局最优路径

#### 时间弹性带：teb
> 全称为`Time Elastic Band`，连接起始、目标点，并让这个路径可以变形，变形的条件就是将所有约束当做橡皮筋的外力。起始点、目标点状态由用户/全局规划器指定，中间插入N个控制橡皮筋形状的控制点（机器人姿态），当然，为了显示轨迹的运动学信息，我们在点与点之间定义运动时间Time，即为`Timed-Elastic-Band`算法

优缺点：适用于各种常见车模；有很强的前瞻性；对动态障碍有较好的避障效果；计算复杂度较大但是可通过牺牲预测距离来降低；速度和角度波动较大控制不稳定但是提高控制频率可以改善

## 区域搜索：RRT
