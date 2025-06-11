# airsim_ros_pkgs

AirSim C++ 客户端库的 ROS 包装器。

## 设置

以下步骤适用于 Linux。如果您在 Windows 上运行 AirSim，可以使用 Windows 子系统 Linux (WSL) 来运行 ROS 包装器，详见 [下面的说明](#setting-up-the-build-environment-on-windows10-using-wsl1-or-wsl2)。如果由于某些问题无法或不愿在主机 Linux 上安装 ROS 和相关工具，您也可以尝试使用 Docker，详见 [使用 Docker 进行 ROS 包装器](#using-docker-for-ros) 中的步骤。

- 如果您的默认 GCC 版本低于 8（使用 `gcc --version` 检查）

    - 安装 gcc >= 8.0.0: `sudo apt-get install gcc-8 g++-8`
    - 通过 `gcc-8 --version` 验证安装

- Ubuntu 16.04
    * 安装 [ROS kinetic](https://wiki.ros.org/kinetic/Installation/Ubuntu)
    * 安装 tf2 sensor 和 mavros 包： `sudo apt-get install ros-kinetic-tf2-sensor-msgs ros-kinetic-tf2-geometry-msgs ros-kinetic-mavros*`

- Ubuntu 18.04
    * 安装 [ROS melodic](https://wiki.ros.org/melodic/Installation/Ubuntu)
    * 安装 tf2 sensor 和 mavros 包： `sudo apt-get install ros-melodic-tf2-sensor-msgs ros-melodic-tf2-geometry-msgs ros-melodic-mavros*`

- Ubuntu 20.04
    * 安装 [ROS noetic](https://wiki.ros.org/noetic/Installation/Ubuntu)
    * 安装 tf2 sensor 和 mavros 包： `sudo apt-get install ros-noetic-tf2-sensor-msgs ros-noetic-tf2-geometry-msgs ros-noetic-mavros*`

- 安装 [catkin_tools](https://catkin-tools.readthedocs.io/en/latest/installing.html)
    `sudo apt-get install python-catkin-tools` 或
    `pip install catkin_tools`。如果使用 Ubuntu 20.04，请使用 `pip install "git+https://github.com/catkin/catkin_tools.git#egg=catkin_tools"`

## 构建

- 构建 AirSim

```shell
git clone https://github.com/Microsoft/AirSim.git;
cd AirSim;
./setup.sh;
./build.sh;
```

- 确保您已按照上述安装页面的说明设置了 ROS 的环境变量。为了方便，可以将 `source` 命令添加到您的 `.bashrc` 中（将 `melodic` 替换为特定版本名称）-

```shell
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

- 构建 ROS 包

```shell
cd ros;
catkin build; # 或 catkin_make
```

如果您的默认 GCC 版本低于 8（使用 `gcc --version` 检查），则编译将失败。在这种情况下，请明确使用 `gcc-8`，如下所示-

```shell
catkin build -DCMAKE_C_COMPILER=gcc-8 -DCMAKE_CXX_COMPILER=g++-8
```

## 运行

```shell
source devel/setup.bash;
roslaunch airsim_ros_pkgs airsim_node.launch;
roslaunch airsim_ros_pkgs rviz.launch;
```

   **注意**: 如果运行 `roslaunch airsim_ros_pkgs airsim_node.launch` 时出现错误，请运行 `catkin clean` 然后再试。

## 使用 AirSim ROS 包装器

ROS 包装器由两个 ROS 节点组成 - 第一个是 AirSim 的多旋翼 C++ 客户端库的包装器，第二个是一个简单的 PD 位置控制器。让我们来看一下两个节点的 ROS API：

### AirSim ROS 包装器节点

#### 发布者：

- `/airsim_node/origin_geo_point` [airsim_ros_pkgs/GPSYaw](https://github.com/microsoft/AirSim/tree/main/ros/src/airsim_ros_pkgs/msg/GPSYaw.msg)
GPS 坐标对应于全球 NED 坐标系。此设置在 AirSim 的 [settings.json](https://microsoft.github.io/AirSim/settings/) 文件中的 `OriginGeopoint` 键下。

- `/airsim_node/VEHICLE_NAME/global_gps` [sensor_msgs/NavSatFix](https://docs.ros.org/api/sensor_msgs/html/msg/NavSatFix.html)
当前无人机在 AirSim 中的 GPS 坐标。

- `/airsim_node/VEHICLE_NAME/odom_local_ned` [nav_msgs/Odometry](https://docs.ros.org/api/nav_msgs/html/msg/Odometry.html)
相对于起飞点的 NED 坐标系下的里程计（默认名称：odom_local_ned，启动名称和帧类型可配置）。

- `/airsim_node/VEHICLE_NAME/CAMERA_NAME/IMAGE_TYPE/camera_info` [sensor_msgs/CameraInfo](https://docs.ros.org/api/sensor_msgs/html/msg/CameraInfo.html)

- `/airsim_node/VEHICLE_NAME/CAMERA_NAME/IMAGE_TYPE` [sensor_msgs/Image](https://docs.ros.org/api/sensor_msgs/html/msg/Image.html)
  根据 settings.json 中请求的图像类型返回 RGB 或浮点图像。

- `/tf` [tf2_msgs/TFMessage](https://docs.ros.org/api/tf2_msgs/html/msg/TFMessage.html)

- `/airsim_node/VEHICLE_NAME/altimeter/SENSOR_NAME` [airsim_ros_pkgs/Altimeter](https://github.com/microsoft/AirSim/blob/main/ros/src/airsim_ros_pkgs/msg/Altimeter.msg)
当前高度、压力和 [QNH](https://en.wikipedia.org/wiki/QNH) 的高度计读数。

- `/airsim_node/VEHICLE_NAME/imu/SENSOR_NAME` [sensor_msgs::Imu](http://docs.ros.org/api/sensor_msgs/html/msg/Imu.html)
IMU 传感器数据。

- `/airsim_node/VEHICLE_NAME/magnetometer/SENSOR_NAME` [sensor_msgs::MagneticField](http://docs.ros.org/api/sensor_msgs/html/msg/MagneticField.html)
  磁场矢量/指南针的测量值。

- `/airsim_node/VEHICLE_NAME/distance/SENSOR_NAME` [sensor_msgs::Range](http://docs.ros.org/api/sensor_msgs/html/msg/Range.html)
  活动探测器（如红外或 IR）的距离测量。

- `/airsim_node/VEHICLE_NAME/lidar/SENSOR_NAME` [sensor_msgs::PointCloud2](http://docs.ros.org/api/sensor_msgs/html/msg/PointCloud2.html)
  LIDAR 点云。

#### 订阅者：

- `/airsim_node/vel_cmd_body_frame` [airsim_ros_pkgs/VelCmd](https://github.com/microsoft/AirSim/tree/main/ros/src/airsim_ros_pkgs/msg/VelCmd.msg)
  忽略 `vehicle_name` 字段，留空。我们将在未来对多个无人机使用 `vehicle_name`。

- `/airsim_node/vel_cmd_world_frame` [airsim_ros_pkgs/VelCmd](https://github.com/microsoft/AirSim/tree/main/ros/src/airsim_ros_pkgs/msg/VelCmd.msg)
  忽略 `vehicle_name` 字段，留空。我们将在未来对多个无人机使用 `vehicle_name`。

- `/gimbal_angle_euler_cmd` [airsim_ros_pkgs/GimbalAngleEulerCmd](https://github.com/microsoft/AirSim/tree/main/ros/src/airsim_ros_pkgs/msg/GimbalAngleEulerCmd.msg)
  Euler 角度下的云台设定点。

- `/gimbal_angle_quat_cmd` [airsim_ros_pkgs/GimbalAngleQuatCmd](https://github.com/microsoft/AirSim/tree/main/ros/src/airsim_ros_pkgs/msg/GimbalAngleQuatCmd.msg)
  四元数下的云台设定点。

- `/airsim_node/VEHICLE_NAME/car_cmd` [airsim_ros_pkgs/CarControls](https://github.com/microsoft/AirSim/blob/main/ros/src/airsim_ros_pkgs/msg/CarControls.msg)
用于控制的油门、刹车、转向和档位选择。支持自动和手动变速控制，详见 [`car_joy.py`](https://github.com/microsoft/AirSim/blob/main/ros/src/airsim_ros_pkgs/scripts/car_joy) 脚本。

#### 服务：

- `/airsim_node/VEHICLE_NAME/land` [airsim_ros_pkgs/Takeoff](https://docs.ros.org/api/std_srvs/html/srv/Empty.html)

- `/airsim_node/takeoff` [airsim_ros_pkgs/Takeoff](https://docs.ros.org/api/std_srvs/html/srv/Empty.html)

- `/airsim_node/reset` [airsim_ros_pkgs/Reset](https://docs.ros.org/api/std_srvs/html/srv/Empty.html)
 重置 *所有* 无人机。

#### 参数：

- `/airsim_node/world_frame_id` [string]
  设置在: `$(airsim_ros_pkgs)/launch/airsim_node.launch`
  默认: world_ned
  设置为 "world_enu" 可以自动切换到 ENU 坐标系。

- `/airsim_node/odom_frame_id` [string]
  设置在: `$(airsim_ros_pkgs)/launch/airsim_node.launch`
  默认: odom_local_ned
  如果将 world_frame_id 设置为 "world_enu"，默认的 odom 名称将改为 "odom_local_enu"。

- `/airsim_node/coordinate_system_enu` [boolean]
  设置在: `$(airsim_ros_pkgs)/launch/airsim_node.launch`
  默认: false
  如果将 world_frame_id 设置为 "world_enu"，此设置将默认改为 true。

- `/airsim_node/update_airsim_control_every_n_sec` [double]
  设置在: `$(airsim_ros_pkgs)/launch/airsim_node.launch`
  默认: 0.01 秒。
  更新无人机里程计和状态及发送控制命令的计时器回调频率。
  目前 RPClib 接口与虚幻引擎的最大频率为 50 Hz。
  ROS 中的计时器回调以可能的最大速率运行，因此最好不要更改此参数。

- `/airsim_node/update_airsim_img_response_every_n_sec` [double]
  设置在: `$(airsim_ros_pkgs)/launch/airsim_node.launch`
  默认: 0.01 秒。
  从 AirSim 中所有摄像头接收图像的计时器回调频率。
  速度取决于请求的图像数量及其分辨率。
  ROS 中的计时器回调以可能的最大速率运行，因此最好不要更改此参数。

- `/airsim_node/publish_clock` [double]
  设置在: `$(airsim_ros_pkgs)/launch/airsim_node.launch`
  默认: false
  如果设置为 true，将发布 ROS /clock 主题。

### 简单 PID 位置控制器节点

#### 参数：

- PD 控制器参数：
  * `/pd_position_node/kd_x` [double],
    `/pd_position_node/kp_y` [double],
    `/pd_position_node/kp_z` [double],
    `/pd_position_node/kp_yaw` [double]
    比例增益。

  * `/pd_position_node/kd_x` [double],
    `/pd_position_node/kd_y` [double],
    `/pd_position_node/kd_z` [double],
    `/pd_position_node/kd_yaw` [double]
    微分增益。

  * `/pd_position_node/reached_thresh_xyz` [double]
    从当前位置到设定位置的欧拉距离阈值（米）。

  * `/pd_position_node/reached_yaw_degrees` [double]
    从当前位置到设定位置的偏航距离阈值（度）。

- `/pd_position_node/update_control_every_n_sec` [double]
  默认: 0.01 秒。

#### 服务：

- `/airsim_node/VEHICLE_NAME/gps_goal` [请求： [srv/SetGPSPosition](https://github.com/microsoft/AirSim/blob/main/ros/src/airsim_ros_pkgs/srv/SetGPSPosition.srv)]
  目标 GPS 位置 + 偏航。
  在 **绝对** 高度下。

- `/airsim_node/VEHICLE_NAME/local_position_goal` [请求： [srv/SetLocalPosition](https://github.com/microsoft/AirSim/blob/main/ros/src/airsim_ros_pkgs/srv/SetLocalPosition.srv)]
  目标本地位置 + 偏航，在全球 NED 坐标系中。

#### 订阅者：

- `/airsim_node/origin_geo_point` [airsim_ros_pkgs/GPSYaw](https://github.com/microsoft/AirSim/tree/main/ros/src/airsim_ros_pkgs/msg/GPSYaw.msg)
  监听由 `airsim_node` 发布的家里地理坐标。

- `/airsim_node/VEHICLE_NAME/odom_local_ned` [nav_msgs/Odometry](https://docs.ros.org/api/nav_msgs/html/msg/Odometry.html)
  监听由 `airsim_node` 发布的里程计。

#### 发布者：

- `/vel_cmd_world_frame` [airsim_ros_pkgs/VelCmd](https://github.com/microsoft/AirSim/tree/main/ros/src/airsim_ros_pkgs/msg/VelCmd.msg)
  向 `airsim_node` 发送速度命令。

#### 全局参数

- 动态约束。这些可以在 `dynamic_constraints.launch` 中更改：
    * `/max_vel_horz_abs` [double]
  无人机的最大水平速度（米/秒）。

    * `/max_vel_vert_abs` [double]
  无人机的最大垂直速度（米/秒）。

    * `/max_yaw_rate_degree` [double]
  最大偏航速率（度/秒）。

## 其他

### 在 Windows10 上使用 WSL1 或 WSL2 设置构建环境

这些设置说明描述了如何在 Windows 上设置 "Bash on Ubuntu"（即 "Windows 子系统 Linux"）。

它涉及到在 Windows10 中启用内置的 Windows Linux 环境 (WSL)、安装兼容的 Linux 操作系统镜像，最后安装构建环境，方式与普通 Linux 系统相同。

完成后，您将能够像在本地 Linux 机器上一样构建和运行 ROS 包装器。

##### WSL1 与 WSL2

WSL2 是 Windows10 的最新版本子系统 Linux。它比 WSL1 快很多倍（如果您使用 `/home/...` 中的本地文件系统，而不是 `/mnt/...` 下的 Windows 挂载文件夹），因此在构建代码时速度更快，推荐使用。

安装后，您可以根据需要在 WSL1 或 WSL2 版本之间切换。

##### WSL 设置步骤

1. 按照 [这里](https://docs.microsoft.com/en-us/windows/wsl/install-win10) 的说明进行操作。检查您要使用的 ROS 版本是否受到您要安装的 Ubuntu 版本的支持。

2. 恭喜您，现在在 Windows 下已成功设置工作 Ubuntu 子系统，您可以继续阅读 [Ubuntu 16 / 18 说明](#setup)，然后 [如何在 Windows 上运行 Airsim 和 WSL 上的 ROS 包装器](#how-to-run-airsim-on-windows-and-ros-wrapper-on-wsl)！

!!! note

    您可以通过在 Windows 上安装 [VcXsrv](https://sourceforge.net/projects/vcxsrv/) 来运行 XWindows 应用程序（包括 SITL）。
    为了使用它，查找并运行 Windows 开始菜单中的 `XLaunch`。
    在第一个弹出窗口中选择 `Multiple Windows`，在第二个弹出窗口中选择 `Start no client`，在第三个弹出窗口中仅选择 `Clipboard`。不要选择 `Native Opengl`（如果您无法连接，请选择 `Disable access control`）。
    您需要设置 DISPLAY 变量以指向您的显示器：在 WSL 中为 `127.0.0.1:0`，在 WSL2 中为计算机网络端口的 IP 地址，可以通过以下代码设置。此外，在 WSL2 中，您可能需要为公共网络禁用防火墙，或创建例外，以便 VcXsrv 能与 WSL2 通信：

    `export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0`

!!! tip

    - 如果将此行添加到您的 ~/.bashrc 文件中，则无需再次运行此命令。
    - 对于代码编辑，您可以在 WSL 中安装 VSCode。
    - Windows 10 包含 "Windows Defender" 病毒扫描器。它会显著降低 WSL 的速度。禁用它可以大大提高磁盘性能，但会增加病毒风险，因此请自行决定是否禁用。以下是许多资源/视频之一，展示了如何禁用它：[如何在 Windows 10 上禁用或启用 Windows Defender](https://youtu.be/FmjblGay3AM)

##### WSL 和 Windows10 之间的文件系统访问

在 WSL 内，Windows 驱动器在 /mnt 目录中引用。例如，要列出您 (<username>) 文档文件夹中的文档：

    `ls /mnt/c/'Documents and Settings'/<username>/Documents`
    或
    `ls /mnt/c/Users/<username>/Documents`


在 Windows 内，WSL 发行版的文件位于（在 Windows 资源管理器地址栏中输入）：

   `\\wsl$\<distribution name>`
   例如：
   `\\wsl$\Ubuntu-18.04`

##### 如何在 Windows 和 WSL 上运行 Airsim 及其 ROS 包装器

对于 WSL 1 执行：
`export WSL_HOST_IP=127.0.0.1`
对于 WSL 2：
`export WSL_HOST_IP=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')`
现在，按 [Linux 运行部分](#running) 中的步骤执行：

```shell
source devel/setup.bash
roslaunch airsim_ros_pkgs airsim_node.launch output:=screen host:=$WSL_HOST_IP
roslaunch airsim_ros_pkgs rviz.launch
```

### 使用 Docker 进行 ROS

在 [`tools`](https://github.com/microsoft/AirSim/tree/main/tools/Dockerfile-ROS) 目录中有一个 Dockerfile。要构建 `airsim-ros` 镜像 -

```shell
cd tools
docker build -t airsim-ros -f Dockerfile-ROS .
```

要运行，请替换下面 AirSim 文件夹的路径 -

```shell
docker run --rm -it --net=host -v <your-AirSim-folder-path>:/home/testuser/AirSim airsim-ros:latest bash
```

上述命令将 AirSim 目录挂载到容器内的主目录中。您在主机上的源文件中所做的任何更改都将在容器内部可见，这对开发和测试是有益的。现在按照 [构建](#build) 中的步骤编译并运行 ROS 包装器。