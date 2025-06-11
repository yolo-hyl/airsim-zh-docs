# PX4 对 AirSim 的设置

[PX4 软件栈](http://github.com/px4/firmware) 是一个开源的非常流行的飞行控制器，支持多种电路板和传感器，并具备进行更高阶任务（如任务规划）的内置能力。欲了解更多信息，请访问 [px4.io](http://px4.io)。

**Warning**: 尽管 AirSim 的所有版本始终会与 PX4 进行测试以确保支持，但设置 PX4 并不是一项简单的任务。除非您对 PX4 栈具有至少中级水平的经验，否则我们建议您使用 [simple_flight](simple_flight.md)，它现在是 AirSim 的默认设置。

## 支持的硬件

以下 Pixhawk 硬件已经与 AirSim 进行了测试：

1. [Pixhawk PX4 2.4.8](http://www.banggood.com/Pixhawk-PX4-2_4_8-Flight-Controller-32-Bit-ARM-PX4FMU-PX4IO-Combo-for-Multicopters-p-1040416.html)
2. [PixFalcon](https://hobbyking.com/en_us/pixfalcon-micro-px4-autopilot.html?___store=en_us)
3. [PixRacer](https://www.banggood.com/Pixracer-Autopilot-Xracer-V1_0-Flight-Controller-Mini-PX4-Built-in-Wifi-For-FPV-Racing-RC-Multirotor-p-1056428.html?utm_source=google&utm_medium=cpc_ods&utm_content=starr&utm_campaign=Smlrfpv-ds-FPVracer&gclid=CjwKEAjw9MrIBRCr2LPek5-h8U0SJAD3jfhtbEfqhX4Lu94kPe88Zrr62A5qVgx-wRDBuUulGzHELRoCRVTw_wcB)
4. [Pixhawk 2.1](http://www.proficnc.com/)
5. [Holybro 的 Pixhawk 4 mini](https://shop.holybro.com/pixhawk4-mini_p1120.html)
6. [Holybro 的 Pixhawk 4](https://shop.holybro.com/pixhawk-4beta-launch_p1089.html)

版本 1.11.2 的 PX4 固件也适用于 Pixhawk 4 设备。

## 设置 PX4 硬件闭环

为此，您需要上述支持的设备之一。对于手动飞行，您还需要 RC + 发射器。

1. 确保您的 RC 接收器已与其 RC 发射器绑定。将 RC 发射器连接到飞行控制器的 RC 端口。有关更多信息，请参阅您的 RC 手册和 [PX4 文档](https://docs.px4.io/en/getting_started/rc_transmitter_receiver.html)。
2. 下载 [QGroundControl](http://qgroundcontrol.com/)，启动它并将飞行控制器连接到 USB 端口。
3. 使用 QGroundControl 刷新最新的 PX4 飞行栈。另见 [初始固件设置视频](https://docs.px4.io/master/en/config/)。
4. 在 QGroundControl 中，通过选择 HIL Quadrocopter X 机架来配置您的 Pixhawk 进行 HIL 模拟。 PX4 重启后，请检查是否确实选择了 "HIL Quadrocopter X"。
5. 在 QGroundControl 中，转到无线电选项卡并进行校准（确保遥控器开启，并且接收器显示绑定指示器）。
6. 转到飞行模式选项卡，选择一个遥控开关作为 "模式通道"。然后设置（例如）稳定飞行和姿态飞行模式为开关的两个位置。
7. 转到 QGroundControl 的调试部分并设置适当的值。例如，对于 Fly Sky 的 FS-TH9X 遥控器，以下设置会带来更真实的感觉：悬停油门 = 中 + 1，横滚和俯仰灵敏度 = 中 - 3，高度和位置控制灵敏度 = 中 - 2。
8. 在 [AirSim 设置](settings.md) 文件中，像下面这样指定 PX4 作为您的车辆配置：
```
    {
        "SettingsVersion": 1.2,
        "SimMode": "Multirotor",
        "ClockType": "SteppableClock",
        "Vehicles": {
            "PX4": {
                "VehicleType": "PX4Multirotor",
                "UseSerial": true,
                "LockStep": true,
                "Sensors":{
                    "Barometer":{
                        "SensorType": 1,
                        "Enabled": true,
                        "PressureFactorSigma": 0.0001825
                    }
                },
                "Parameters": {
                    "NAV_RCL_ACT": 0,
                    "NAV_DLL_ACT": 0,
                    "COM_OBL_ACT": 1,
                    "LPE_LAT": 47.641468,
                    "LPE_LON": -122.140165
                }
            }
        }
    }
```

注意 PX4 的 `[simulator]` 使用 TCP，因此我们需要添加 `"UseTcp": true,`。注意我们还启用了 `LockStep`，有关更多信息，请参见 [PX4 LockStep](px4_lockstep.md)。`Barometer` 设置保持了 PX4 的愉悦，因为默认的 AirSim 气压计有点过多的噪声生成。这个设置可以减少噪声，从而使 PX4 更快地取得 GPS 锁定。

完成以上设置后，您应该能够使用遥控器（RC）在 AirSim 中飞行。您通常可以通过将 RC 的两个杆向下和向内移动来为飞行器解锁。初始设置后无需再使用 QGroundControl。通常，稳定模式（而非手动模式）为初学者提供了更好的体验。请参见 [PX4 基础飞行指南](https://docs.px4.io/master/en/flying/basic_flying.html)。

您还可以通过 [Python APIs](apis.md) 控制无人机。

请参见 [演示视频](https://youtu.be/HNWdYrtw3f0) 和 [Unreal AirSim 设置视频](https://youtu.be/1oY8Qu5maQQ)，展示了本文件中的所有设置步骤。

## 设置 PX4 软件闭环

PX4 SITL 模式不需要您拥有 Pixhawk 或 Pixracer 等独立设备。这实际上是 PX4 团队推荐的与模拟器一起使用 PX4 的方式。然而，这确实更难以设置。请参见 [这个专门页面](px4_sitl.md) 来设置 PX4 在 SITL 模式下。

## 常见问题

#### 无人机飞行不正常，只是“失控”。

造成这种情况的原因有几个。首先，请确保您的无人机在启动模拟器时没有从高处坠落。这可能发生在您创建自定义 Unreal 环境并且玩家起始位置安置得离地面太高时。当发生这种情况时，PX4 内部校准会出错。

您还应[使用 QGroundControl](#setting-up-px4-hardware-in-loop)，确保您能在 QGroundControl 中正常解锁和起飞。

最后，这在某些罕见情况下也可能是机器性能问题，请检查您的 [硬盘性能](hard_drive.md)。

#### 我可以使用 Arducopter 或其他 MavLink 实现吗？

我们的代码经过 [PX4 固件](https://dev.px4.io/) 测试。我们未测试 Arducopter 或其他 mavlink 实现。一些飞行 API 确实在 MAV_CMD_DO_SET_MODE 消息中使用 PX4 自定义模式（如 PX4_CUSTOM_MAIN_MODE_AUTO）。

#### 找不到我的 Pixhawk 硬件

检查您的 settings.json 文件中这一行 "SerialPort":"*,115200"。这里的星号表示“查找任何看起来像 Pixhawk 设备的串口，但这并不总是对所有类型的 Pixhawk 硬件有效”。在 Windows 上，您可以通过设备管理器找到实际的 COM 端口，查看“端口（COM 和 LPT）”，插入设备，查看显示的新 COM 端口。假设您看到一个新端口名为 "USB Serial Port (COM5)"。那么，您可以将 SerialPort 设置更改为： "SerialPort":"COM5,115200"。

在 Linux 上，可以通过运行 "ls /dev/serial/by-id" 找到设备，如果您看到一个看起来像 `usb-3D_Robotics_PX4_FMU_v2.x_0-if00` 的设备名称，则可以使用该名称连接，如下所示：
"SerialPort":"/dev/serial/by-id/usb-3D_Robotics_PX4_FMU_v2.x_0-if00"。请注意，此长名称实际上是指向真实名称的符号链接，如果您使用 "ls -l ..."，可以找到该符号链接，通常类似于 "/dev/ttyACM0"，因此这也会工作 "SerialPort":"/dev/ttyACM0,115200"。但该映射与 Windows 类似，它是自动分配的并可能会更改，而长名称将即使在实际 TTY 串行设备映射更改的情况下也能工作。

#### WARN  [commander] 起飞被拒绝，解除武装再重试

如果您在 PX4 仍未计算家庭位置时尝试起飞，就会发生这种情况。PX4 会在对 GPS 信号满意时报告家庭位置，您将看到以下消息：

```
INFO  [commander] home: 47.6414680, -122.1401672, 119.99
INFO  [tone_alarm] home_set
```

在这之前，PX4 将拒绝起飞命令。

#### 当我告诉无人机做某事时，它总是降落

例如，您使用 DroneShell `moveToPosition -z -20 -x 50 -y 0`，它确实如此，但当到达目标位置时，无人机开始降落。这是 PX4 在离线模式完成时的默认行为。要使无人机悬停，请设置此 PX4 参数：
```
param set COM_OBL_ACT 1
```

#### 我收到消息长度不匹配的错误

您可能需要在 QGC 中将 MAV_PROTO_VER 参数设置为 "始终使用版本 1"。有关更多详细信息，请参见 [此问题](https://github.com/Microsoft/AirSim/issues/546)。