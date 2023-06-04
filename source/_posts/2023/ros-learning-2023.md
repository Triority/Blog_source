---
title: ros学习笔记-2023重制版
tags:
  - 笔记
  - linux
  - python
  - ROS
index_img: /img/ros.png
categories:
- [计算机, 知识整理]
notshow: false
date: 2023-02-21 16:37:19
excerpt: 重写一篇笔记
---
# 前言
之前写过一篇关于[ROS学习笔记](https://triority.cn/root/ros学习笔记/)，那时候刚开始学感觉不适合现在使用所以重写一篇。

这篇文章缩减了一部分不需要的内容，并且加入了实操应用的记录
# 基础知识
## 工作空间和功能包的基本操作
### 创建工作空间目录和代码空间
```
mkdir ros_learning
cd ros_learning
mkdir src
```
### 创建工作空间并编译和生成install空间
```
catkin_init_workspace
cd ..
catkin_make
catkin_make install
```
### 环境变量设置
在工作空间目录内：
```
source devel/setup.bash
```
检查环境变量：
```
echo $ROS_PACKAGE_PATH
```
每次打开终端都要设置环境变量，如果想要免除这一步骤可以修改用户目录内的(`/home/triority`)文件`.bashrc`。

在`source /opt/ros/melodic/setup.bash`下一行加入:
```
source /home/triority/desktop/ROS_learning/devel/setup.bash
```
工作空间路径改成自己的。

### 创建功能包:catkin_create_pkg
在`src`文件夹内：
```
catkin_create_pkg <包名> <依赖>
```
依赖：roscpp；用于编写c++程序，rospy：用于编写python程序，std_msgs；ros标准消息结构，等依赖的的一个名为`lab`的功能包(应创建在src文件夹内)
```
catkin_create_pkg lab roscpp rospy std_msgs geometry_msgs turtlesim message_generation
```

## 话题：发布与订阅
### 自定义消息
##### 消息定义
在功能包文件夹内创建`msg`文件夹，并新建`lab.msg`文件，在里面写入：
```
string str
```
##### 添加依赖
在`package.xml`文件内添加依赖：
```
<exec_depend>message_runtime</exec_depend>
<build_export_depend>message_generation</build_export_depend>
```
在`CMakeLists.txt`内加入：
```
add_message_files(
  FILES
  Person.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
)
```
并把
```
#  CATKIN_DEPENDS geometry_msgs roscpp rospy std_msgs turtlesim
```
改为
```
   CATKIN_DEPENDS geometry_msgs roscpp rospy std_msgs turtlesim message_runtime
```
### Publisher
publisher.py
记得给.py文件添加可执行权限！！！
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import rospy
from lab.msg import lab


def publisher():
	# 初始化ROS节点
    rospy.init_node('lab_publisher_node', anonymous=True)

	# 创建一个Publisher，发布名为/publisher_info的topic，消息类型为lab，队列长度10
    publisher_info = rospy.Publisher('/publisher_info', lab, queue_size=10)

	#设置循环的频率
    rate = rospy.Rate(1) 

    while not rospy.is_shutdown():
        # 初始化消息类型
        lab_msg = lab()
        lab_msg.str = "Tom"

		    # 发布消息
        publisher_info.publish(lab_msg)
        rospy.loginfo("Publsh person message[%s]", lab.str)

		    # 按照循环频率延时
        rate.sleep()


if __name__ == '__main__':
    try:
        publisher()
    except rospy.ROSInterruptException:
        pass
```
### Subscriber
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import rospy
from lab.msg import lab


def InfoCallback(msg):
    rospy.loginfo("Subcribe Info: str:%s", msg.str)


def subscriber():
	# ROS节点初始化
    rospy.init_node('lab_subscriber', anonymous=True)

	# 创建一个Subscriber，订阅名为/publisher_info的topic，注册回调函数InfoCallback
    rospy.Subscriber("/publisher_info", lab, InfoCallback)

	# 循环等待回调函数
    rospy.spin()


if __name__ == '__main__':
    subscriber()
```
### rosrun
```
rosrun lab publisher.py
```
```
rosrun lab subscriber.py
```
## 服务：服务端与客户端
### 自定义消息
新建`srv`文件夹并写入`labs.srv`(不能与前面的lab.msg同名)：
```
string str
---
string res
```
在`package.xml`添加的依赖和前面的话题部分一样，写过就不再写了：
```
<exec_depend>message_runtime</exec_depend>
<build_export_depend>message_generation</build_export_depend>
```
在`CMakeList.txt`添加：
```
add_service_files(
  FILES
  Person.srv
)
```
还需要添加的另一部分也已经在前面的话题部分一样：
```
generate_messages(
  DEPENDENCIES
  std_msgs
)
```
### Client
client.py
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import rospy
from lab.srv import labs, labsResponse


def client():
	  # ROS节点初始化
    rospy.init_node('lab_client')

	  # 发现/lab_server服务后，创建客户端连接名为/lab_server的service
    rospy.wait_for_service('/lab_server')
    try:
        person_client = rospy.ServiceProxy('/lab_server', labs)

		    # 请求服务调用，输入请求数据
        response = person_client("str")
        return response.res
    except rospy.ServiceException as e:
        print ("Service call failed: %s"%e)

if __name__ == "__main__":
	  #服务调用并显示调用结果
    print ("Show result : %s" %(client()))
```
### Server
server.py
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import rospy
from lab.srv import labs, labsResponse


def Callback(req):
	  # 显示请求数据
    rospy.loginfo("string: str:%s", req.str)

	  # 反馈数据
    return labsResponse("OK")


def person_server():
	  # ROS节点初始化
    rospy.init_node('lab_server')

	  # 创建一个名为/lab_server的server，注册回调函数Callback
    s = rospy.Service('/lab_server', labs, Callback)

	  # 循环等待回调函数
    print ("Ready to show.")
    rospy.spin()

if __name__ == "__main__":
    person_server()
```
### rosrun
```
rosrun lab client.py
```
```
rosrun lab server.py
```

## 参数服务器：全局字典
### 参数操作命令行
1.列出参数
```
rosparam list
```
2.获取参数值
```
rosparam get <参数>
```
3.修改参数
```
rosparam set <参数>
```
4.保存参数为文件
```
rosparam dump xxx.yaml
```
5.从文件加载参数
```
rosparam load xxx.yaml
```
6.删除参数
```
rosparam delete xxx.yaml
```
### 使用python操作参数
parameter_config.py
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import sys
import rospy
from std_srvs.srv import Empty


def parameter_config():
	# ROS节点初始化
    rospy.init_node('parameter_config', anonymous=True)

	# 读取背景颜色参数
    red   = rospy.get_param('/turtlesim/background_r')
    green = rospy.get_param('/turtlesim/background_g')
    blue  = rospy.get_param('/turtlesim/background_b')

    rospy.loginfo("Get Backgroud Color[%d, %d, %d]", red, green, blue)

	# 设置背景颜色参数
    rospy.set_param("/turtlesim/background_r", 255);
    rospy.set_param("/turtlesim/background_g", 255);
    rospy.set_param("/turtlesim/background_b", 255);

    rospy.loginfo("Set Backgroud Color[255, 255, 255]");

	# 读取背景颜色参数
    red   = rospy.get_param('/turtlesim/background_r')
    green = rospy.get_param('/turtlesim/background_g')
    blue  = rospy.get_param('/turtlesim/background_b')

    rospy.loginfo("Get Backgroud Color[%d, %d, %d]", red, green, blue)

	# 发现/spawn服务后，创建一个服务客户端，连接名为/spawn的service
    rospy.wait_for_service('/clear')
    try:
        clear_background = rospy.ServiceProxy('/clear', Empty)

		# 请求服务调用，输入请求数据
        response = clear_background()
        return response
    except rospy.ServiceException as e:
        print ("Service call failed: %s"%e)

if __name__ == "__main__":
    parameter_config()
```

## 使用launch文件启动
### luanch文件语法
##### <launch>根元素定义标签
##### <node>启动节点
```
<node pkg="package-name" type="executable-name" name="node-name" />
```
可选属性：
+ `pkg`：节点功能包名称
+ `type`：节点可执行文件名称
+ `name`：节点运行时的名称，取代原有节点名，避免冲突
+ `output`：控制终端使出
+ `respawn`：挂掉后是否自动重启
+ `required`：是否为必须项
+ `args`：输入参数
+ ......

##### <param>设置参数服务器的参数
```
<param name="output_frame" value="odom" />
```
`name`：参数名；`value`：参数值

加载多个参数：
```
<rosparam file+"param.yaml" command="load" ns="params" />
```
##### <arg>launch文件内局部变量
```
<arg name="arg-name" default="arg-value" />
```
`name`：参数名；`value`：参数值
##### <remap>重命名
```
<remap from="/turtlebot/cmd_vel" to+"/cmd_vel" />
```
##### <include>包含其他launch文件
### 运行launch文件
```
roslaunch <package_name> <launch_name>
```

## 命令行及可视化工具
##### 可视化查看节点关系`rqt_graph`
```
rqt_graph
```
##### 查看节点信息`rosnode`
查看全部节点
```
rosnode list
```
查看节点详细信息
```
rosnode info <节点名>
```
##### 查看话题`rostopic`
打印话题列表
```
rostopic list
```
给话题发布数据
```
rostopic pub <话题名> <数据内容>
```
> 数据内容可用两次tab补全默认格式。可加参数 -r <频率(Hz)> 来连续发布

##### 查看消息`rosmsg`
查看消息数据结构
```
rosmsg show <话题名>
```
##### 查看服务`rosservice`
#查看所有服务
```
rosservice list
```
发布服务请求
```
rosservice call <服务名>
```
##### 记录工具
记录话题数据
```
rosbag record -a -O <压缩数据包名>
```
> 其中-a表示all保存全部数据，-O表示保存成压缩包

复现话题数据
```
rosbag paly <复现文件名>
```
##### rqt可视化
显示tf坐标变换关系：
```
rosrun rqt_tf_tree rqt_tf_tree
```

日志输出工具`rqt_console`
计算图可视化工具`rqt_graph`
数据绘图工具`rqt_plot`
图像渲染工具`rqt_image_view`



# 实际应用记录
这部分记录2023年的人工智能竞赛学习开发过程
## 前置任务[获取传感器信息]
新建工作空间这部分就不说了，首先按照资料给出的内容测试一下车载传感器。

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

## 激光雷达建图：gmapping
