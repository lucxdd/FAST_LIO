# fast_lio2调试记录

# 编译

## 1.按着官方readme就能编译成功



# 启动

## 1 启动惯导

### 1.1 连接IMU的USB，检查电脑能否识别到ttyUSB0

`ls /dev/ttyUSB0`

### 1.2 检测到ttyUSB0后，给ttyUSB0赋权限

`sudo chmod 777 /dev/ttyUSB0`

### 1.3 启动imu

`roslaunch handsfree_ros_imu rviz_and_imu.launch imu_type:=a9`	//自带rviz版本，**仅用于检查**。移动imu，在rviz里是否跟着一起动
`roslaunch handsfree_ros_imu handsfree_imu.launch  imu_type:=a9`	//没有rviz版本，用于激光惯导SLAM

<!--陀螺仪和加速计的发布话题：/handsfree/imu -->
<!-- 磁力计的发布话题：/handsfree/mag -->

## 2 启动固态激光雷达livox

### 2.1 雷达时间同步

参考https://blog.csdn.net/m0_38144614/article/details/124862734?spm=1001.2014.3001.5506

sudo ptpd -M -i enp49s0 -C	//enp49s0是zh的y9000p上的livox网卡，ip是192.168.1.50
<!-- 可以启动livox_iewer查看有没有时间同步（在官网可以下载） -->

### 2.2 启动激光雷达驱动

样例：`roslaunch livox_ros_driver xxx.launch bd_list:="3GGDHAD0010427"`	//该livox广播码： bd_list:="3GGDHAD0010427"
`roslaunch livox_ros_driver livox_lidar_rviz.launch bd_list:="3GGDHAD0010427"`	//PointCloud2格式点云
`roslaunch livox_ros_driver livox_lidar_msg.launch bd_list:="3GGDHAD0010427"`	  //livox自定义格式。fast-lio2, faster_lio使用的格式

<!--livox_lidar_rviz.launch	//向外发布pointcloud2格式的点云数据,自动加载rviz-->
<!--livox_lidar.launch	//向外发布pointcloud2格式的点云数据,无rviz-->
<!--livox_lidar_msg.launch	//向外发布览沃自定义点云数据-->

## 3 录包

### 3.1 启动相机（需要在rviz中显示时可选，加了这个包会很大）

`roslaunch zed_wrapper zed2i.launch`		//显示success

### 3.2 在指定路径下录制话题

`rosbag rosbag record -a`	//录全部话题
`rosbag record -O livox_imu.bag /livox/lidar /handsfree/imu  /zed2i/zed_node/right_raw/image_raw_color`	//录指定话题

## 4 运行激光惯导SLAM

`roslaunch faster_lio mapping_avia.launch`		//高翔开发的faster_lio

`roslaunch fast_lio mapping_avia.launch`			//港大开发的FAST_LIO2，有论文
