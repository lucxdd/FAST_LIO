# ZH调试记录

## 一、编译imu

### 1.1 handsfree_imu 
功能包代码地址： https://gitee.com/HANDS-FREE/handsfree_ros_imu.git

```bash
mkdir -p ~/handsfree_ros_ws/src/
cd ~/handsfree_ros_ws/src/
git clone https://gitee.com/HANDS-FREE/handsfree_ros_imu.git
cd ~/handsfree_ros_ws/
catkin_make
cd ~/handsfree_ros_ws/src/handsfree_ros_imu/scripts/
sudo chmod 777 *.py
```

### 1.2 导远组合导航
代码地址：https://github.com/98huan/gnss_ins.git
```bash
mkdir -p ~/daoyuan_ws/src/
cd ~/daoyuan_ws/src/
catkin_make
```
## 二、编译livox的ros驱动
### 2.1 livox_ros_driver

代码地址：https://github.com/Livox-SDK/livox_ros_driver

照着官方readme就能编译

### 2.2 livox_ros_driver2

代码地址：https://github.com/Livox-SDK/livox_ros_driver2
照着官方readme编译完需要修改livox_ros_driver2/config/MID360_config.json（可参考
站视频https://www.bilibili.com/video/BV1s24y1B7gh/?spm_id_from=333.788&vd_source=192eb451247aa7a4011ed5bfff4d78b2）
host_net_info中所有ip改成网络连接的ip：192.168.1.50
lidar_configs中的ip改成雷达的固定ip：192.168.1.137（每个雷达都不一样）

## 三、编译fast_lio2

代码地址：https://github.com/98huan/FAST_LIO

按着官方readme就能编译成功

## 四、启动

### 4.1  启动惯导

### 4.1.1 检查

`ls /dev/ttyUSB*`

### 4.1.2 给所有串口赋权限

`sudo chmod 777 /dev/ttyUSB*`

### 4.1.3 启动imu节点

#### 4.1.3.1 导远组合导航
```bash
roslaunch daoyuan_integrated_navigation daoyuan_integrated_navigation.launch
```
#### 4.1.3.2 handsfree_imu
```c++
roslaunch handsfree_ros_imu rviz_and_imu.launch imu_type:=a9	//有rviz，默认USB0（前面没启动导远时用这个检查imu是否正常）。在rviz里会动
roslaunch handsfree_ros_imu handsfree_imu.launch  imu_type:=a9	//无rviz，默认USB1（先启动GPS时会先占用USB0），用于SLAM
// 陀螺仪和加速计的发布话题：/handsfree/imu 
// 磁力计的发布话题：/handsfree/mag 
```

## 4.2 启动livox

### 4.2.1 雷达时间同步

参考https://blog.csdn.net/m0_38144614/article/details/124862734?spm=1001.2014.3001.5506

```c++
// 同时连两个雷达不能在一个网段下，因此把livox静态ip改成了192.168.2.1
sudo ptpd -M -i enp49s0 -C	//zh的y9000p上的livox网卡，ip是192.168.2.50
sudo ptpd -M -i enx00e04c36036a -C      //zh的y9000p上的rslidar网卡，ip时192.168.1.102
```

 可以启动livox_iewer查看有没有时间同步（在官网https://www.livoxtech.com/cn/downloads可以下载） 

### 4.2.2 启动雷达驱动

#### livox mid-70
livox mid-70广播码： bd_list:="3GGDHAD00104271"，样例：
```c++
//PointCloud2格式点云loam系使用的格式
roslaunch livox_ros_driver livox_lidar_rviz.launch bd_list:="3GGDHAD00104271"
//livox自定义格式。fast-lio2, faster_lio使用的格式
roslaunch livox_ros_driver livox_lidar_msg.launch bd_list:="3GGDHAD00104271"
```
#### livox mid-360
livox mid360 广播码：47MDKBV0010037
```c++
//PointCloud2格式点云loam系使用的格式
roslaunch livox_ros_driver2 rviz_MID360.launch
//livox自定义格式。fast-lio2, faster_lio使用的格式
roslaunch livox_ros_driver2 msg_MID360.launch
```
#### rslidar
```c++
roslaunch rslidar_sdk start.launch	//有rviz
roslaunch rslidar_sdk start_noRviz.launch	//无rviz
roslaunch rslidar_sdk start_with_rs_converter.launch	//无rviz，有velodyne格式的点云
```
## 5 录包

### 5.1 启动相机（需要在rviz中显示时可选，加了这个包会很大）

```c++
roslaunch zed_wrapper zed2i.launch		//zed2i
roslaunch azure_kinect_ros_driver driver.launch	//Azure_Kinect_DK
```

### 5.2 启动小车底盘
```c++
rosrun scout_bringup bringup_can2usb.bash       //启动can设备
roslaunch scout_bringup scout_robot_base.launch //发布scout_odom里程计话题
roslaunch scout_bringup scout_teleop_keyboard.launch //键盘控制节点，通过向/cmd_vel发布指令控制小车
```

### 5.3 在指定路径下录制话题

```
//录全部话题
rosbag rosbag record -a
//录指定话题zed2i
rosbag record /handsfree/imu /zed2i/zed_node/right_raw/image_raw_color /scout_odom /livox/lidar /rslidar_points /velodyne_points /gps /imu_correct /odometry/gpsz
//录指定话题kinect相机
rosbag record /handsfree/imu /rgb/image_raw/compressed /scout_odom /livox/lidar /rslidar_points /velodyne_points /gps /imu_correct /odometry/gpsz
```

## 6 运行SLAM

```
roslaunch fast_lio mapping_avia.launch	//FAST_LIO2
```

