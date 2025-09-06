配置港科大IMU标定程序的环境
（1）安装ceres
不过多介绍：去官网下载源码编译就好了。
https://github.com/ceres-solver/ceres-solver
http://www.ceres-solver.org/

（2）为标定IMU创建ROS工作空间
mkdir -p ~/catkin_imu_ws/src
cd ~/catkin_imu_ws/src
catkin_init_workspace
cd ~/catkin_imu_ws
catkin_make
source ~/catkin_imu_ws/devel/setup.bash

（可以将对应的source命令和ROS工作路径加入.bashrc文件中，按道理将应该会更好，实际上创建了这两个工作空间以后我原来的ros包都不能识别都，都要手动进行相应的ros工作目录进行
source devel/setup.bash 后才能自动补全）

（3）先下载code_utils并编译
cd ~/catkin_imu_ws/src/
git clone https://github.com/gaowenliang/code_utils.git
cd ~/catkin_imu_ws/src/
catkin_make 

（4）然后再下载imu_utils并编译
cd ~/catkin_imu_ws/src/
git clone https://github.com/gaowenliang/imu_utils.git
cd ~/catkin_imu_ws/src/
catkin_make

注：（3）和（4）注意先后顺序，一个一个编译，中间有点小问题，很简单的那种，一个是头文件可能要加一下路径，还有一个要包含一下vector这个头文件。

3.编写参数配置文件
cd ~/catkin_imu_ws/src/imu_utils/launch
touch d435i_imu_calibration.launch
gedit d435i_imu_calibration.launch

直接复制粘贴下面的就行了

<launch>
    <node pkg="imu_utils" type="imu_an" name="imu_an" output="screen">
        <param name="imu_topic" type="string" value= "/camera/imu"/>
        <param name="imu_name" type="string" value= "d435i_imu_calibration"/>
        <param name="data_save_path" type="string" value= "$(find imu_utils)/data/"/>
        <param name="max_time_min" type="int" value= "10"/>
        <param name="max_cluster" type="int" value= "100"/>
    </node>
</launch>

注：max_time_min代表的是标定时间，这里的单位是分钟，意思是填10就是代表10分钟，具体录制多久自行把握，反正时间越长越好。

4.离线录制IMU数据包
打开d435i相机

roscore 
roslaunch realsense2_camera rs_camera.launch

rqt_image_view	//查看相机是否正常打开
rostopic echo /camera/imu  //检查IMU话题时候有输出
rostopic hz /camera/imu //或者检查IMU打印频率

一切正常后不要移动相机，静置，进入想要保存bag的目录下调用终端开始录制。

rosbag record -O imu_calibration /camera/imu 

至少录制你刚刚设定的时间以上(max_time_min)
比如上面设定10，这里就要录足十分钟
录制完之后就按下ctrl+c，结束录制
你会发现当前目录有一个名为 imu_calibration.bag的文件

5.使用IMU工具箱标定
打开一个终端：

source ~/catkin_imu_ws/devel/setup.bash
roslaunch imu_utils d435i_imu_calibration.launch 

再调一个终端

source ~/catkin_imu_ws/devel/setup.sh 
cd ~/catkin_imu_ws //如果数据包在这个文件夹下
rosbag play -r 200 imu_calibration.bag

执行完毕后即可在这个目录下/imu_catkin_ws/src/imu_utils/data 找到一个名为 d435i_imu_calibration_imu_param.yaml的文件，打开即可查看标定结果，这是15分钟的标定结果。

%YAML:1.0
---
type: IMU
name: d435i_imu_calibration
Gyr:
   unit: " rad/s"
   avg-axis:
      gyr_n: 1.8282505899879821e-03
      gyr_w: 1.9435262017179183e-05
   x-axis:
      gyr_n: 1.4728528003952847e-03
      gyr_w: 1.8708803243080059e-05
   y-axis:
      gyr_n: 2.3392615939959523e-03
      gyr_w: 2.2671377122899810e-05
   z-axis:
      gyr_n: 1.6726373755727097e-03
      gyr_w: 1.6925605685557679e-05
Acc:
   unit: " m/s^2"
   avg-axis:
      acc_n: 1.2816582334747946e-02
      acc_w: 5.0028610709826078e-04
   x-axis:
      acc_n: 1.2169492333962395e-02
      acc_w: 3.9215145394702318e-04
   y-axis:
      acc_n: 1.5026867851384572e-02
      acc_w: 7.8886749202973451e-04
   z-axis:
      acc_n: 1.1253386818896876e-02
      acc_w: 3.1983937531802474e-04

等下只会用到其中几个参数用于相机和IMU之间的外参标定，分别是陀螺仪和加速度计高斯白噪声和随机游走噪声的平均值，是IMU噪声模型中的两种噪声。

   avg-axis:
      gyr_n: 1.8282505899879821e-03
      gyr_w: 1.9435262017179183e-05
   avg-axis:
      acc_n: 1.2816582334747946e-02
      acc_w: 5.0028610709826078e-04


————————————————

                            版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
                        
原文链接：https://blog.csdn.net/m0_64151866/article/details/137914028
