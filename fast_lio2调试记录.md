# fast_lio2调试记录

## 一、编译imu

### handsfree_ros_imu功能包代码地址： https://gitee.com/HANDS-FREE/handsfree_ros_imu.git

```c++
mkdir -p ~/handsfree_ros_ws/src/
cd ~/handsfree_ros_ws/src/
git clone https://gitee.com/HANDS-FREE/handsfree_ros_imu.git
cd ~/handsfree_ros_ws/
catkin_make
cd ~/handsfree_ros_ws/src/handsfree_ros_imu/scripts/
sudo chmod 777 *.py
```
## 二、编译livor_ros_driver

代码地址：https://github.com/Livox-SDK/livox_ros_driver

照着官方readme就能编译

## 三、编译fast_lio2

代码地址：https://github.com/98huan/FAST_LIO

按着官方readme就能编译成功

## 四、启动

### 4.1  启动导远组合导航（最先启动）
```c++
roslaunch dy_rec dy_rec.launch
```

### 4.2  启动小惯导

### 4.2.1 前面没启动导远时

`ls /dev/ttyUSB0`
`sudo chmod 777 /dev/ttyUSB0`

### 4.2.2 前面启动导远时

`ls /dev/ttyUSB1`
`sudo chmod 777 /dev/ttyUSB1`

### 4.2.3 启动imu驱动

```c++
roslaunch handsfree_ros_imu rviz_and_imu.launch imu_type:=a9	//rviz版本，**仅用于检查**。移动imu，在rviz里会动
roslaunch handsfree_ros_imu handsfree_imu.launch  imu_type:=a9	//没有rviz版本，用于SLAM
```

```
陀螺仪和加速计的发布话题：/handsfree/imu 
磁力计的发布话题：/handsfree/mag 
```

## 4.2 启动livox

### 4.1.1 雷达时间同步

参考https://blog.csdn.net/m0_38144614/article/details/124862734?spm=1001.2014.3001.5506

```
sudo ptpd -M -i enp92s0 -C	//enp92s0是lwh的y9000k上的livox网卡，ip是192.168.1.50
sudo ptpd -M -i enp49s0 -C	//enp49s0是zh的y9000p上的livox网卡，ip是192.168.1.50
```

 可以启动livox_iewer查看有没有时间同步（在官网https://www.livoxtech.com/cn/downloads可以下载） 

### 4.1.2 启动livox驱动

该livox mid-70广播码： bd_list:="3GGDHAD0010427"，样例：

```
roslaunch livox_ros_driver livox_lidar_rviz.launch bd_list:="3GGDHAD0010427" //PointCloud2格式点云loam系使用的格式
roslaunch livox_ros_driver livox_lidar_msg.launch bd_list:="3GGDHAD0010427"	 //livox自定义格式。fast-lio2, faster_lio使用的格式
```

```
livox_lidar_rviz.launch	//向外发布pointcloud2格式的点云数据,自动加载rviz
livox_lidar.launch	//向外发布pointcloud2格式的点云数据,无rviz
livox_lidar_msg.launch	//向外发布览沃自定义点云数据
```

## 5 录包

### 5.1 启动相机（需要在rviz中显示时可选，加了这个包会很大）

`roslaunch zed_wrapper zed2i.launch`		//显示success

### 5.2 在指定路径下录制话题
推荐用rqt_bag录包

```
rosbag rosbag record -a	//录全部话题
rosbag record /livox/lidar /handsfree/imu /zed2i/zed_node/right_raw/image_raw_color	//录指定话题
```

## 6 运行激光惯导SLAM

```
roslaunch faster_lio mapping_avia.launch  //高翔开发的faster_lio，有论文
roslaunch fast_lio mapping_avia.launch	//港大开发的FAST_LIO2，有论文
```

