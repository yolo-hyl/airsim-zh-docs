# AirLib 在真实无人机上的应用

AirLib 库可以在真实无人机的伴随计算机上编译和部署。为了进行测试，我们在无人机上安装了一台 Gigabyte Brix BXi7-5500 超紧凑型 PC，通过 USB 与 Pixhawk 飞行控制器连接。这台 Gigabyte PC 运行的是 Ubuntu，因此我们能够通过 Wi-Fi 进行 SSH 连接：

![Flamewheel](images/Flamewheel.png)

连接后，您可以使用以下命令运行 MavLinkTest：
```
MavLinkTest -serial:/dev/ttyACM0,115200 -logdir:. 
```
这将生成一个飞行日志文件，然后可以用于 [在模拟器中回放](playback.md)。

您还可以添加 `-proxy:192.168.1.100:14550`，将 MavLinkTest 连接到远程计算机，在那里您可以运行 QGroundControl 或我们的 [PX4 Log Viewer](log_viewer.md)，这是一种方便的方法来查看无人机的状态。

MavLinkTest 还有一些简单的命令用于测试您的无人机，以下是一些命令的简单示例：

```
arm
takeoff 5
orbit 10 2
```

这将解锁无人机，起飞 5 米，然后以 10 米的半径以 2 米/秒的速度进行环绕飞行。
键入 '?' 可以查看所有可用命令。

**注意：** 一些命令（例如 `orbit`）在 MavLinkTest 和 DroneShell 中的命名和语法是不同的（例如，`circlebypath -radius 10 -velocity 21`）。

当您降落无人机时，可以停止 MavLinkTest 并复制生成的 *.mavlink 日志文件。

# DroneServer 和 DroneShell

一旦您确认 MavLinkTest 正常工作，您还可以按如下方式运行 DroneServer 和 DroneShell。首先，使用本地代理运行 MavLinkTest，将所有内容发送到 DroneServer：

```
MavLinkTest -serial:/dev/ttyACM0,115200 -logdir:. -proxy:127.0.0.1:14560
```
将 ~/Documents/AirSim/settings.json 中的 "serial" 设置为 false，因为我们希望 DroneServer 查找此 UDP 连接。

```
DroneServer 0
```

最后，您现在可以将 DroneShell 连接到此实例的 DroneServer，并使用 DroneShell 命令来飞行您的无人机：

```
DroneShell
==||=>
        欢迎使用 DroneShell 1.0。
        输入 ? 获取帮助。
        微软研究院 (c) 2016。

等待无人机报告有效的 GPS 位置...
==||=> requestcontrol
==||=> arm
==||=> takeoff
==||=> circlebypath -radius 10 -velocity 2
```

## PX4 特定工具
您可以运行 MavlinkCom 库和 MavLinkTest 应用以测试伴随计算机与飞行控制器之间的连接。

## 这怎么运作？
AirSim 使用 @lovettchris 开发的 MavLinkCom 组件。MavLinkCom 具有代理架构，您可以通过串行或 UDP 打开到 PX4 的连接，然后其他组件共享此连接。当 PX4 发送 MavLink 消息时，所有组件都会接收到该消息。如果任何组件发送消息，则仅由 PX4 接收。这允许您将任意数量的组件连接到 PX4 [此代码](https://github.com/microsoft/AirSim/blob/main/AirLib/include/vehicles/multirotor/firmwares/mavlink/MavLinkMultirotorApi.hpp#L600) 为 LogViewer 和 QGC 打开连接。如果您愿意，可以添加更多内容。

如果您想将 QGC 与 AirSim 一起使用，则需要 QGC 拥有串口。QGC 打开一个 TCP 连接，作为代理，让任何其他组件可以连接到 QGC 并将 MavLinkMessage 发送到 QGC，然后 QGC 将该消息转发到 PX4。因此，您告知 AirSim 连接到 QGC，并让 QGC 拥有串口。

对于伴随板，我们之前的做法是在无人机上放置 Gigabyte Brix。这是一台与 PX4 通过 USB 连接的完整 x86 计算机。我们在 Brix 上安装了 Ubuntu，并运行 [DroneServer](https://github.com/Microsoft/AirSim/tree/main/DroneServer)。DroneServer 创建了一个 API 端点，我们可以通过 C++ 客户端代码（或 Python 代码）进行交互，并将 API 调用转换为 MavLink 消息。这样，您可以针对相同的 API 编写代码，在模拟器中进行测试，然后在实际设备上运行相同的代码。因此，伴随计算机上运行着 DroneServer 和客户端代码。