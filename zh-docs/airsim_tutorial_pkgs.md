# AirSim ROS 教程

这是一个用于在 ROS 中使用 AirSim 的示例 `settings.json`、roslaunch 和 rviz 文件的集合，提供了一个起点。     
请参见 [airsim_ros_pkgs](https://github.com/microsoft/AirSim/blob/main/ros/src/airsim_ros_pkgs/README.md) 获取 ROS API。

## 设置

确保已完成 [airsim_ros_pkgs 设置](airsim_ros_pkgs.md) 并安装了先决条件。

```shell
$ cd PATH_TO/AirSim/ros
$ catkin build airsim_tutorial_pkgs
```

如果你的默认 GCC 不是 8 或更高版本（使用 `gcc --version` 检查），则编译将会失败。在这种情况下，可以明确使用 `gcc-8`，如下所示-

```shell
catkin build airsim_tutorial_pkgs -DCMAKE_C_COMPILER=gcc-8 -DCMAKE_CXX_COMPILER=g++-8
```

!!! note

    为了运行示例，并且每次打开新的终端时，都需要源执行 `setup.bash` 文件。如果你经常使用 ROS 包装器，建议将 `source PATH_TO/AirSim/ros/devel/setup.bash` 添加到你的 `~/.profile` 或 `~/.bashrc` 中，以避免每次打开新终端时都需要运行该命令。

## 示例

### 单架无人机，配备单目和深度摄像头，以及激光雷达
 - Settings.json - [front_stereo_and_center_mono.json](https://github.com/microsoft/AirSim/blob/main/ros/src/airsim_tutorial_pkgs/settings/front_stereo_and_center_mono.json)
 ```shell
 $ source PATH_TO/AirSim/ros/devel/setup.bash
 $ roscd airsim_tutorial_pkgs
 $ cp settings/front_stereo_and_center_mono.json ~/Documents/AirSim/settings.json

 ## 在此启动你的 Unreal 包或二进制文件

 $ roslaunch airsim_ros_pkgs airsim_node.launch;

 # 在新面板/终端中
 $ source PATH_TO/AirSim/ros/devel/setup.bash
 $ roslaunch airsim_tutorial_pkgs front_stereo_and_center_mono.launch
 ```
 上述操作将启动 rviz，显示 tf 已注册的 RGBD 点云，使用 [depth_image_proc](https://wiki.ros.org/depth_image_proc)，基于 [`depth_to_pointcloud` 启动文件](https://github.com/microsoft/AirSim/blob/main/ros/src/airsim_tutorial_pkgs/launch/front_stereo_and_center_mono/depth_to_pointcloud.launch)，以及激光雷达点云。

### 两架无人机，配备摄像头、激光雷达和 IMU
- Settings.json - [two_drones_camera_lidar_imu.json](https://github.com/microsoft/AirSim/blob/main/ros/src/airsim_tutorial_pkgs/settings/two_drones_camera_lidar_imu.json)

 ```shell
 $ source PATH_TO/AirSim/ros/devel/setup.bash
 $ roscd airsim_tutorial_pkgs
 $ cp settings/two_drones_camera_lidar_imu.json ~/Documents/AirSim/settings.json

 ## 在此启动你的 Unreal 包或二进制文件

 $ roslaunch airsim_ros_pkgs airsim_node.launch;
 $ roslaunch airsim_ros_pkgs rviz.launch
 ```
你可以在 rviz 中查看 tf。并使用 `rostopic list` 和 `rosservice list` 检查可用的服务。

### 二十五架无人机以方形模式排列
- Settings.json - [twenty_five_drones.json](https://github.com/microsoft/AirSim/blob/main/ros/src/airsim_tutorial_pkgs/settings/twenty_five_drones.json)

 ```shell
 $ source PATH_TO/AirSim/ros/devel/setup.bash
 $ roscd airsim_tutorial_pkgs
 $ cp settings/twenty_five_drones.json ~/Documents/AirSim/settings.json

 ## 在此启动你的 Unreal 包或二进制文件

 $ roslaunch airsim_ros_pkgs airsim_node.launch;
 $ roslaunch airsim_ros_pkgs rviz.launch
 ```
你可以在 rviz 中查看 tf。并使用 `rostopic list` 和 `rosservice list` 检查可用的服务。