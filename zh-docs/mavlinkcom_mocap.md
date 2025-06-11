# 欢迎使用 MavLinkMoCap

这个文件夹包含 MavLinkMoCap 库，旨在连接到 OptiTrack 相机系统以实现精确的室内定位。

## 依赖项：
* [OptiTrack Motive](http://www.optitrack.com/products/motive/)。
* [MavLinkCom](mavlinkcom.md)。

### 设置 RigidBody

首先，您需要使用 Motive 定义一个名为 'Quadrocopter' 的 RigidBody。
请参阅 [Rigid_Body_Tracking](http://wiki.optitrack.com/index.php?title=Rigid_Body_Tracking)。

### MavLinkTest

使用 MavLinkTest 与您的 PX4 无人机进行通信，使用“–server:addr:port”，例如，连接到无人机 WiFi 时使用：

    MavLinkMoCap -server:10.42.0.228:14590 "-project:D:\OptiTrack\Motive Project 2016-12-19 04.09.42 PM.ttp"

这会发布 ATT_POS_MOCAP 消息，您可以通过在无人机大脑上运行 MavLinkTest 将这些消息代理到 PX4：

    MavLinkTest -serial:/dev/ttyACM0,115200 -proxy:10.42.0.228:14590

现在无人机将接收到 ATT_POS_MOCAP 消息，您应该看到灯光变为绿色，表示它现在已经设定了家位置并准备起飞。