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
```sh
ros-autocar@ros-autocar:~$ ls /dev/ttyUSB*
/dev/ttyUSB1  /dev/ttyUSB2  /dev/ttyUSB3
ros-autocar@ros-autocar:~$ ls /dev/ttyUSB*
/dev/ttyUSB0  /dev/ttyUSB1  /dev/ttyUSB2  /dev/ttyUSB3
```
有四个设备，拔掉imu之后再次查看少了那个就知道imu的地址了。给IMU的设备添加读写权限：
```sh
sudo chmod  +777 /dev/ttyUSB0
```
然后使用资料给的命令就可以读取imu的数值了
```sh
roslaunch imu_launch  imu_msg.launch
```
```sh
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

***

> 激光雷达

资料给了驱动雷达的功能包，启动雷达的launch命令：
这个launch文件有一个地方需要改动就是雷达串口设备号，我这一次改成是`/dev/ttyUSB2`如果启动文件之后雷达不转可以考虑是这个问题
```sh
roslaunch ls01b_v2 ls01b_v2.launch
```
然后在另外一个终端查看雷达数据：应该会看到满屏的数字hhh都是雷达扫描到的距离信息
```sh
rostopic echo /scan
```

***

> 编码器

同样，使用资料的驱动功能包：

```sh
roscore
rosrun encoder_driver Encoder_vel.py
rostopic echo /encoder
```
输出：
```sh
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

***

> 整合及运动控制

```sh
roslaunch racecar Run_car.launch 
rosrun racecar racecar_teleop.py
rviz rviz
```
第一个启动了上述传感器的程序：
![](微信截图_20230605212637.png)
第二个用于使用键盘发布信息控制底盘，最后是rviz用于显示雷达等信息。

其中雷达需要设置坐标变换，雷达的坐标是相对于底盘坐标定义的，rviz默认使用map坐标系显示，需要定义底盘坐标系和map坐标系的相对关系。
```xml
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
```xml
<launch>
 
  <node pkg="rf2o_laser_odometry" type="rf2o_laser_odometry_node" name="rf2o_laser_odometry" output="screen">
    <param name="laser_scan_topic" value="/scan"/>        # 雷达发布数据的话题
    <param name="odom_topic" value="/odom" />             # 发布测程估计的topic
    <param name="publish_tf" value="true" />              # 是否发布tf变换(base->odom)
    <param name="base_frame_id" value="/base_footprint"/> # 底盘的frame_id (tf) 必须要雷达laser_frame到base_frame的tf变换
    <param name="odom_frame_id" value="/odom" />          # 发布测程估计的frame_id (tf) 
    <param name="init_pose_from_topic" value="" />        # Odom 话题初始点，留空从(0,0)开始
    <param name="freq" value="10.0"/>                     # 执行频率
    <param name="verbose" value="true"/>                  # verbose
  </node>
</launch>
```

接下来改一些bug：修改`src/CLaserOdometry2D.cpp`

292-298行修改:
```cpp
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
```cpp
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
```sh
roslaunch rf2o_laser_odometry rf2o_laser_odometry.launch 
```
就可以看见终端输出的里程数据。这个方法得到的数据有些缺陷，就是在车子发生旋转时候是不准确的，因为雷达和车子都在旋转相对位置不准确。

![](rf2o.png)

还有一个bug，如果你把车子往前推，结果rf2o输出为负方向，那么需要改一下代码：
`CLaserOdometry2D.cpp`：
```
Line 923:
  pose_aux_2D.translation()(0) = -acu_trans(0,2);
  pose_aux_2D.translation()(1) = -acu_trans(1,2);
Line 956:
  lin_speed = -acu_trans(0,2) / time_inc_sec;
```
## 使用编码器和imu定位
这段代码是志伟学长写的，我就一边学习理解一边写写注释吧
```python
import rospy
from nav_msgs.msg import Odometry
from sensor_msgs.msg import Imu
from scipy.spatial.transform import Rotation
import numpy as np
import time
import tf2_ros
from geometry_msgs.msg import Pose, Point, Quaternion, TransformStamped, PoseArray
import tf_conversions
import message_filters
from math import sin, cos, pi,sqrt,fabs

last_angle = None
last_vel = None
last_time = None
pos_x = 0
pos_y = 0
br = tf2_ros.TransformBroadcaster()
encoder_dx = 27.7/1000.0

def callback(imu_data, encoder_data):
    global last_angle, last_vel, last_time, pos_x, pos_y

    r = Rotation.from_quat([imu_data.orientation.x, 
                            imu_data.orientation.y, 
                            imu_data.orientation.z, 
                            imu_data.orientation.w])
    angle = r.as_euler('xyz')[2]
    if angle<0:
        angle = 2*np.pi + angle

    if encoder_data.twist.twist.linear.x>10 or encoder_data.twist.twist.linear.x<-10:
        return
    
    if last_time is None:
        last_time = encoder_data.header.stamp
        last_angle = angle
        last_vel = encoder_data.twist.twist.linear.x
        return
    
    dt = (encoder_data.header.stamp - last_time).to_sec()
    last_time = encoder_data.header.stamp


    d = (encoder_data.twist.twist.linear.x+last_vel)/2.0*dt
    last_vel = encoder_data.twist.twist.linear.x

    d_angle = np.fabs(angle - last_angle)
    if angle - last_angle>0.00001:
        l = d / d_angle+encoder_dx
        d = l * np.sqrt(2*(1-np.cos(d_angle)))
    elif angle - last_angle<-0.00001:
        l = d / d_angle-encoder_dx
        d = l * np.sqrt(2*(1-np.cos(d_angle)))
    
    last_angle = angle
    
    d_x = d * np.cos(angle)
    d_y = d * np.sin(angle)
    pos_x += d_x
    pos_y += d_y

    t = TransformStamped()
    t.header.stamp = rospy.Time.now()
    t.header.frame_id = "odom"
    t.child_frame_id = "base_link"
    t.transform.translation.x = pos_x
    t.transform.translation.y = pos_y
    t.transform.translation.z = 0.0

    q = tf_conversions.transformations.quaternion_from_euler(0, 0, angle)
    t.transform.rotation.x = q[0]
    t.transform.rotation.y = q[1]
    t.transform.rotation.z = q[2]
    t.transform.rotation.w = q[3]

    br.sendTransform(t)
    odom_data = Odometry()
    odom_data.header.stamp = rospy.Time.now()
    odom_data.header.frame_id = "odom"
    odom_data.child_frame_id = "base_link"
    odom_data.pose.pose.position.x = pos_x
    odom_data.pose.pose.position.y = pos_y
    odom_data.pose.pose.position.z = 0.0
    odom_data.pose.pose.orientation.x = q[0]
    odom_data.pose.pose.orientation.y = q[1]
    odom_data.pose.pose.orientation.z = q[2]
    odom_data.pose.pose.orientation.w = q[3]
    odom_pub.publish(odom_data)
    
# 初始化encoder_mix节点
rospy.init_node('encoder_mix')
# 创建Subscriber，订阅imu和编码器的topic
imu_sub = message_filters.Subscriber('/imu_data', Imu)
encoder_sub = message_filters.Subscriber('/encoder', Odometry)
# 使用自适应算法来匹配基于其时间戳的消息并执行回调函数，详见https://blog.csdn.net/chishuideyu/article/details/77479758
ts = message_filters.ApproximateTimeSynchronizer([imu_sub, encoder_sub], 20, 0.1)
ts.registerCallback(callback)
# 
odom_pub = rospy.Publisher('/odom', Odometry, queue_size=10)
rospy.spin()
```


## 传感器数据融合：robot localization/ekf



## 激光雷达建图：gmapping/cartographer
[gmapping的github](https://github.com/ros-perception/slam_gmapping)

gmapping安装：
```
sudo apt-get install ros-noetic-slam-gmapping
```
编写一个新的launch文件：
```xml
<launch>
    <arg name="scan_topic" default="scan" />                  <!-- 发布scan名称 -->
    <node pkg="gmapping" type="slam_gmapping" name="slam_gmapping" output="screen" clear_params="true">
        <param name="base_frame" value="base_link"/>     <!-- 基座标系名称 -->
        <param name="odom_frame" value="odom"/>               <!-- 里程计坐标系名称 -->
        <param name="map_update_interval" value="4.0"/>
        <!-- Set maxUrange < actual maximum range of the Laser -->
        <param name="maxRange" value="5.0"/>
        <param name="maxUrange" value="4.5"/>
        <param name="sigma" value="0.05"/>
        <param name="kernelSize" value="1"/>
        <param name="lstep" value="0.05"/>
        <param name="astep" value="0.05"/>
        <param name="iterations" value="5"/>
        <param name="lsigma" value="0.075"/>
        <param name="ogain" value="3.0"/>
        <param name="lskip" value="0"/>
        <param name="srr" value="0.01"/>
        <param name="srt" value="0.02"/>
        <param name="str" value="0.01"/>
        <param name="stt" value="0.02"/>
        <param name="linearUpdate" value="0.5"/>
        <param name="angularUpdate" value="0.436"/>
        <param name="temporalUpdate" value="-1.0"/>
        <param name="resampleThreshold" value="0.5"/>
        <param name="particles" value="80"/>
        <param name="xmin" value="-1.0"/>
        <param name="ymin" value="-1.0"/>
        <param name="xmax" value="1.0"/>
        <param name="ymax" value="1.0"/>
        <param name="delta" value="0.05"/>
        <param name="llsamplerange" value="0.01"/>
        <param name="llsamplestep" value="0.01"/>
        <param name="lasamplerange" value="0.005"/>
        <param name="lasamplestep" value="0.005"/>
        <remap from="scan" to="$(arg scan_topic)"/>
    </node>
</launch>
```
启动launch文件：
```sh
roslaunch test gmapping.launch
```
![](gmapping.png)

建图后，在想要保存的路径打开终端并输入使用mapserver保存命令：
```sh
ros-autocar@ros-autocar:~/Ros-autocar$ rosrun map_server map_saver -f 233
[ INFO] [1685971241.710265601]: Waiting for the map
[ INFO] [1685971241.931482169]: Received a 480 X 544 map @ 0.050 m/pix
[ INFO] [1685971241.931548164]: Writing map occupancy data to 233.pgm
[ INFO] [1685971241.942866551]: Writing map occupancy data to 233.yaml
[ INFO] [1685971241.943307197]: Done
```
`233`为保存地图的文件名

补充一个launch文件启动rviz的代码：
```xml
    <!-- rviz -->
    <node pkg="rviz" type="rviz" name="rviz" args="-d /home/ros-autocar/Ros-autocar/rviz.rviz" required="true" />
```
其中`/home/ros-autocar/Ros-autocar/rviz.rviz`是rviz配置文件的路径
## 重定位：amcl
与rf2o的雷达两帧之间比较不同，amcl是雷达数据和地图之间比较计算里程计误差。

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
