# 欢迎来到 MavLinkCom

MavLinkCom 是一个跨平台的 C++ 库，旨在帮助连接和与基于 [MavLink](https://github.com/mavlink/mavlink) 的车辆进行通信。具体来说，该库与基于 [PX4](https://github.com/PX4/Firmware) 的无人机协同工作良好。

## 设计

您可以在 Visual Studio 中查看和编辑 [Design.dgml](mavlinkcom_design/Design.dgml) 图。  
![概述](mavlinkcom_design/images/overview.png)

以下是此库中最重要的类。

### MavLinkNode

这是所有 MavLinkNodes 的基类（子类包括 MavLinkVehicle、MavLinkVideoClient 和 MavLinkVideoServer）。该节点通过 MavLinkConnection 连接到您的支持 mavlink 的车辆，并提供发送 MavLinkMessages 和 MavLinkCommands 的方法，以及订阅接收消息的方法。该基类还存储了您的应用程序希望用来识别自己与远程车辆的本地系统 ID 和组件 ID。您还可以调用 startHeartbeat 发送定期的心跳消息，以保持连接的活跃。

### MavLinkMessage

这是编码后的 MavLinkMessage。对于使用过 mavlink.h C API 的人来说，这类似于 mavlink_message_t。您不需要手动创建这些，它们是从强类型的 MavLinkMessageBase 子类编码而来的。

### 强类型消息和命令类

MavLinkComGenerator 解析 mavlink common.xml 消息定义，并生成所有的 MavLink* MavLinkMessageBase 子类以及许多方便的 mavlink 枚举和强类型的 MavLinkCommand 子类。

### MavLinkMessageBase

这是一个强类型消息类的基类，由 MavLinkComGenerator 项目代码生成。它替代了在 mavlink C API 中定义的 C 消息，并提供了一种稍微面向对象的方式，通过 MavLinkNode 的 sendMessage 方法发送和接收消息。这些类具有编码/解码方法，可以在 MavLinkMessage 类之间进行转换。

### MavLinkCommand

这是一个强类型命令类的基类，由 MavLinkComGenerator 项目代码生成。它替代了在 mavlink C API 中定义的 C 定义，并提供了一种更面向对象的方式，通过 MavLinkNode 的 sendCommand 方法发送命令。MavLinkNode 负责将这些转变为底层的 mavlink COMMAND_LONG 消息。

### MavLinkConnection

此类提供静态助手方法，用于通过串行端口、UDP 或 TCP 套接字创建与远程 MavLink 节点的连接。该类提供了一种以发布/订阅方式订阅接收来自该节点的消息的方法，因此您可以在同一连接上有多个订阅者。MavLinkVehicle 使用它来跟踪定义整体车辆状态的各种消息。

### MavLinkVehicle

MavLinkVehicle 是一个 MavLinkNode，跟踪定义整体车辆状态的各种消息，并提供包含该状态快照的 VehicleState 结构，包括 home 位置、当前位置、局部位置、全局位置等等。该类还提供许多包装常用命令的辅助方法，提供简单的方法调用来执行诸如解锁、上锁、起飞、降落、前往本地坐标以及通过位置或速度控制进行外部控制等操作。

### MavLinkTcpServer

此助手类提供了一种设置“服务器”的方式，该服务器接受来自远程节点的 MavLinkConnections。您可以使用此类获取一个连接，然后将其提供给 MavLinkVideoServer 以通过 MavLink 传输图像。

### MavLinkFtpClient

此助手类接受给定的 MavLinkConnection，并为支持 FTP 功能的车辆提供 MAVLINK_MSG_ID_FILE_TRANSFER_PROTOCOL 的 FTP 客户端支持。该类提供简单的方法来列出目录内容，以及获取和放置文件。

### MavLinkVideoClient

此助手类接受给定的 MavLinkConnection，并提供请求来自远程节点的视频的辅助方法，并将 MAVLINK_MSG_ID_DATA_TRANSMISSION_HANDSHAKE 和 MAVLINK_MSG_ID_ENCAPSULATED_DATA 消息打包成易于使用的 MavLinkVideoFrames。

### MavLinkVideoServer

此助手类接受给定的 MavLinkConnection，并提供 MavLinkVideoClient 协议的服务器端，包括在处理视频请求时通知的辅助方法（hasVideoRequest）和发送视频帧的方法（sendFrame），该方法将生成正确的 MAVLINK_MSG_ID_DATA_TRANSMISSION_HANDSHAKE 和 MAVLINK_MSG_ID_ENCAPSULATED_DATA 序列。

## 示例

以下代码来自 UnitTest 项目，展示了如何通过 USB 串行端口连接到 [Pixhawk](http://www.pixhawk.org/) 飞行控制器，然后等待接收第一个心跳消息：

```c++
auto connection = MavLinkConnection::connectSerial("drone", "/dev/ttyACM0", 115200, "sh /etc/init.d/rc.usb\n");
MavLinkHeartbeat heartbeat;
if (!waitForHeartbeat(10000, heartbeat)) {
	throw std::runtime_error("在 10 秒后未收到来自 PX4 的心跳");
}
```

以下代码连接到串行端口，然后将所有消息转发到和来自 QGroundControl 的无人机，使用另一个与无人机流加入的连接。
```c++
auto droneConnection = MavLinkConnection::connectSerial("drone", "/dev/ttyACM0", 115200, "sh /etc/init.d/rc.usb\n");
auto proxyConnection = MavLinkConnection::connectRemoteUdp("qgc", "127.0.0.1", "127.0.0.1", 14550);
droneConnection->join(proxyConnection);
```
以下代码利用该连接开启心跳并开始跟踪车辆信息，使用本地系统 ID 166 和组件 ID 1。
```c++
auto vehicle = std::make_shared<MavLinkVehicle>(166, 1);
vehicle->connect(connection);
vehicle->startHeartbeat();

std::this_thread::sleep_for(std::chrono::seconds(5));

VehicleState state = vehicle->getVehicleState();
printf("Home 位置是 %s, %f,%f,%f\n", state.home.is_set ? "已设定" : "未设定", 
	state.home.global_pos.lat, state.home.global_pos.lon, state.home.global_pos.alt);
```
以下代码使用车辆对象来解锁无人机并起飞，并等待达到起飞高度：
```c++
bool rc = false;
if (!vehicle->armDisarm(true).wait(3000, &rc) || !rc) {
	printf("解锁命令失败\n");
	return;
}
if (!vehicle->takeoff(targetAlt).wait(3000, &rc) || !rc) {
	printf("起飞命令失败\n");
	return;
}
int version = vehicle->getVehicleStateVersion();
while (true) {
	int newVersion = vehicle->getVehicleStateVersion();
	if (version != newVersion) {
		VehicleState state = vehicle->getVehicleState();
		float alt = state.local_est.pos.z;
		if (alt >= targetAlt - delta && alt <= targetAlt + delta)
		{			
			reached = true;
			printf("目标高度已达到\n");
			break;
		}
	} else {
		std::this_thread::sleep_for(std::chrono::milliseconds(10));
	}
}
```	
以下代码使用外部控制让无人机飞行成圆圈，镜头指向中心。这里我们使用 subscribe 方法检查每个新的位置消息，以便我们可以尽快计算新的速度向量。我们请求较高频率的这些消息，使用 setMessageInterval，以确保平滑的圆形轨道。
```c++
vehicle->setMessageInterval((int)MavLinkMessageIds::MAVLINK_MSG_ID_LOCAL_POSITION_NED, 30);
vehicle->requestControl();
int subscription = vehicle->getConnection()->subscribe(
	[&](std::shared_ptr<MavLinkConnection> connection, const MavLinkMessage& m) {
		if (m.msgid == (int)MavLinkMessageIds::MAVLINK_MSG_ID_LOCAL_POSITION_NED)
		{
			// 将通用消息转换为强类型消息。
			MavLinkLocalPositionNed localPos;
			localPos.decode(msg); 
			float x = localPos.x;
			float y = localPos.y;
			float dx = x - cx;
			float dy = y - cy;
			float angle = atan2(dy, dx);
			if (angle < 0) angle += M_PI * 2;
			float tangent = angle + M_PI_2;
			double newvx = orbitSpeed * cos(tangent);
			double newvy = orbitSpeed * sin(tangent);
			float heading = angle + M_PI;
			vehicle->moveByLocalVelocityWithAltHold(newvx, newvy, altitude, true, heading);
		}
	});
```
以下代码停止了无人机的外部飞行模式，并告诉无人机在当前位置盘旋。此版本的代码展示了如何使用 AsyncResult 而不在等待调用上阻塞。
```c++
	vehicle->releaseControl();
	if (vehicle->loiter().then([=](bool rc) {
		printf("盘旋命令 %s\n", rc ? "成功" : "失败");
	}
```

以下代码获取无人机的所有可配置参数并打印它们：

```c++
    auto list = vehicle->getParamList();
	auto end = list.end();
	int count = 0;
	for (auto iter = list.begin(); iter < end; iter++)
	{
		count++;
		MavLinkParameter p = *iter;
		if (p.type == MAV_PARAM_TYPE_REAL32 || p.type == MAV_PARAM_TYPE_REAL64) {
			printf("%s=%f\n", p.name.c_str(), p.value);
		}
		else {
			printf("%s=%d\n", p.name.c_str(), static_cast<int>(p.value));
		}
	}
```
以下代码设置 Pixhawk 的一个参数以禁用 USB 安全检查（如果您是通过另一台属于无人机本身的计算机通过 USB 控制 Pixhawk，这会很方便）。如果您是通过 USB 将您的 PC 或笔记本电脑连接到无人机，则**不应执行此操作**。

```c++
    MavLinkParameter  p;
    p.name = "CBRK_USB_CHK";
    p.value = 197848;
    if (!vehicle->setParameter(p).wait(3000,&rc) || !rc) {
		printf("设置 CBRK_USB_CHK 失败");
	}
```

MavLinkVehicle 实际上有一个辅助方法，名为 allowFlightControlOverUsb，因此你现在知道它是如何实现的 :-)

## 高级连接

您可以使用 MavLinkConnection 类的 "join" 方法，连接不同配置的 mavlink 管道，如下所示。

示例 1：我们通过串行连接到 PX4，并将这些消息通过远程端口转发给 QGroundControl 和 LogViewer。

![串行到 QGC](mavlinkcom_design/images/example1.png)

示例 2：模拟可以与 jMavSim 通信，而 jMavSim 连接到 PX4。jMavSim 还可以管理多个连接，因此它可以与虚幻模拟器通信。另一个 MavLinkConnection 可以加入以代理 jMavSim 不支持的连接，比如 LogViewer 或远程摄像头节点。

![串行到 QGC](mavlinkcom_design/images/example2.png)

示例 3：我们使用 MavLinkConnection 通过串行连接到 PX4，然后为所有远程节点包括 jMavSim 加入额外的连接。

![串行到 QGC](mavlinkcom_design/images/example3.png)

示例 4：我们还可以进行分布式系统以远程控制无人机：

![串行到 QGC](mavlinkcom_design/images/example4.png)