# 设置 PX4 软件在环仿真

[PX4](http://dev.px4.io) 软件提供了一个在 Linux 上运行的 "软件在环"（SITL）版本。如果你使用的是 Windows，可以使用 [Cygwin 工具链](https://dev.px4.io/master/en/setup/dev_env_windows_cygwin.html)，或者使用 [Windows 子系统 Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) 并遵循 PX4 Linux 工具链的设置。

如果你使用的是 WSL2，请阅读这些 [附加说明](px4_sitl_wsl2.md)。

**注意**：每次停止 Unreal 应用后，你必须重新启动 `px4` 应用。

1. 从你的 Bash 终端按 [这些步骤进行配置 Linux](https://docs.px4.io/master/en/dev_setup/dev_env_linux.html)，并按照 `NuttX 基于硬件` 下的 **所有** 指示安装必要的前置条件。我们还包括了我们自己的 [PX4 构建说明](px4_build.md)，其中包含更简洁的具体需求。

2. 获取 PX4 源代码并构建 POSIX SITL 版本的 PX4：
    ```
    mkdir -p PX4
    cd PX4
    git clone https://github.com/PX4/PX4-Autopilot.git --recursive
    bash ./PX4-Autopilot/Tools/setup/ubuntu.sh --no-nuttx --no-sim-tools
    cd PX4-Autopilot
    ```
   并从 [https://github.com/PX4/PX4-Autopilot/releases](https://github.com/PX4/PX4-Autopilot/releases) 找到最新的稳定版本，并签出与该版本相匹配的源代码，例如：
    ```
    git checkout v1.11.3
    ```

3. 使用以下命令在 SITL 模式下构建并启动 PX4 固件：
    ```
    make px4_sitl_default none_iris
    ```
   如果你使用的是旧版本 v1.8.*，请改用此命令：`make posix_sitl_ekf2 none_iris`。

4. 你应该看到一条消息，表明 SITL PX4 应用正在等待模拟器（AirSim）连接。你还会看到关于哪个端口配置用于与 PX4 应用进行 mavlink 连接的信息。默认端口最近发生了变化，因此请仔细检查以确保 AirSim 设置正确。
    ```
    INFO  [simulator] Waiting for simulator to connect on TCP port 4560
    INFO  [init] Mixer: etc/mixers/quad_w.main.mix on /dev/pwm_output0
    INFO  [mavlink] mode: Normal, data rate: 4000000 B/s on udp port 14570 remote port 14550
    INFO  [mavlink] mode: Onboard, data rate: 4000000 B/s on udp port 14580 remote port 14540
    ```

    注意：这也是一个交互式的 PX4 控制台，输入 `help` 可以查看可以在这里输入的命令列表。它们大多是低级别的 PX4 命令，但其中一些在调试时可能会很有用。

5. 现在编辑 [AirSim 设置](settings.md) 文件，确保 UDP 和 TCP 端口设置匹配：
    ```json
    {
        "SettingsVersion": 1.2,
        "SimMode": "Multirotor",
        "ClockType": "SteppableClock",
        "Vehicles": {
            "PX4": {
                "VehicleType": "PX4Multirotor",
                "UseSerial": false,
                "LockStep": true,
                "UseTcp": true,
                "TcpPort": 4560,
                "ControlPortLocal": 14540,
                "ControlPortRemote": 14580,
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
    注意 PX4 的 `[simulator]` 正在使用 TCP，这就是我们需要添加 `"UseTcp": true,` 的原因。注意我们还启用了 `LockStep`，有关更多信息，请参见 [PX4 LockStep](px4_lockstep.md)。`Barometer` 设置使 PX4 感到满意，因为默认 AirSim 气压计产生了一些过多的噪声。这个设置稍微降低了噪声，使 PX4 更快地实现 GPS 锁定。

6. 使用防火墙配置打开入站 TCP 端口 4560 和入站 UDP 端口 14540。

7. 现在运行你的 Unreal AirSim 环境，它应该通过 TCP 连接到 SITL PX4。你应该会看到来自 SITL PX4 窗口的一系列消息。具体而言，以下消息告诉你 AirSim 已正确连接且 GPS 融合稳定：
    ```
    INFO  [simulator] Simulator connected on UDP port 14560
    INFO  [mavlink] partner IP: 127.0.0.1
    INFO  [ecl/EKF] EKF GPS checks passed (WGS-84 origin set)
    INFO  [ecl/EKF] EKF commencing GPS fusion
    ```

    如果你没有看到这些消息，请检查你的端口设置。

8. 你还应该能在 SITL 模式下使用 QGroundControl。确保没有连接 Pixhawk 硬件，否则 QGroundControl 将选择使用它。请注意，由于我们没有实际硬件，遥控器无法直接连接。因此，替代方案是使用 Xbox 360 控制器，或通过 USB 连接你的遥控器（例如，对于 FrSky Taranis X9D Plus）或通过培训 USB 电缆连接到你的 PC。这使得你的遥控器看起来像一个操纵杆。你需要在 QGroundControl 中做额外的设置，才能使用虚拟操纵杆进行遥控。如果你不打算手动飞行 AirSim 中的无人机，则不需要执行此操作。使用 Python API 进行自主飞行不需要遥控器，详见下面的 `无遥控`。

## 设置 GPS 原点

注意上述设置在 `settings.json` 文件的 `params` 部分提供：
```
    "LPE_LAT": 47.641468,
    "LPE_LON": -122.140165,
```

PX4 SITL 模式需要配置以正确获取家乡位置。
家乡位置需要设置为与 [OriginGeopoint](settings.md#origingeopoint) 中定义的坐标相同。

你也可以在 SITL PX4 控制台窗口中运行以下命令检查这些值是否设置正确。

```
param show LPE_LAT
param show LPE_LON
```

## 平滑的离线过渡

注意上述设置在 `settings.json` 文件的 `params` 部分提供：
```
    "COM_OBL_ACT": 1
```

这告诉无人机在每个离线控制命令完成后自动悬停（默认设置是降落）。悬停是在多个离线命令之间更平滑的过渡。你可以通过运行以下 PX4 控制台命令检查此设置：

```
param show COM_OBL_ACT
```

## 检查家乡位置

如果你使用 DroneShell 执行命令（解锁、起飞等），那么你应该等到家乡位置被设置。你会看到 PX4 SITL 控制台输出此消息：

```
INFO  [commander] home: 47.6414680, -122.1401672, 119.99
INFO  [tone_alarm] home_set
```

现在 DroneShell 的 'pos' 命令应该报告此位置，并且命令应该被 PX4 接受。
如果你尝试在没有家乡位置的情况下起飞，你会看到消息：

```
WARN  [commander] Takeoff denied, disarm and re-try
```

在设置家乡位置后，检查 'pos' 命令报告的本地位置：

```
Local position: x=-0.0326988, y=0.00656854, z=5.48506
```

如果 z 坐标大于这个值，起飞可能不会按预期工作。重置 SITL 和仿真应能解决此问题。

## WSL 2

Windows 子系统 Linux 版本 2 在虚拟机中运行。这需要额外设置 - 请参见 [附加说明](px4_sitl_wsl2.md)。

## 无遥控

注意上述设置在 `settings.json` 文件的 `params` 部分提供：
```
    "NAV_RCL_ACT": 0,
    "NAV_DLL_ACT": 0,
```

如果你打算使用 Python 脚本飞行 SITL 模式的 PX4 而不使用遥控器，这是必需的。这些参数阻止 PX4 在每次移动命令完成时触发“安全模式开启”。你可以使用以下 PX4 命令检查这些值是否设置正确：

```
param show NAV_RCL_ACT
param show NAV_DLL_ACT
```

注意：请勿在实际无人机上进行此操作，因为在没有这些安全措施的情况下飞行是非常危险的。

## 手动设置参数

你也可以在 PX4 控制台运行以下命令手动设置所有这些参数：

```
param set NAV_RCL_ACT 0
param set NAV_DLL_ACT 0
```

## 设置多无人机仿真

你可以使用 AirSim 在 SITL 模式下模拟多个无人机。但是，这需要设置多个 PX4 固件模拟器实例，以便能够在单独的 TCP 端口（4560、4561 等）上监听每个无人机的连接。请参见 [这个专用页面](px4_multi_vehicle.md) 获取设置多个 PX4 实例在 SITL 模式下的说明。

## 使用 VirtualBox Ubuntu

如果你想在 `VirtualBox Ubuntu` 机器上运行上述 posix_sitl，它将具有与 localhost 不同的 IP 地址。所以在这种情况下，你需要编辑 [设置文件](settings.md) 并将 UdpIp 和 SitlIp 更改为你的虚拟机的 IP 地址，将 LocalIpAddress 设置为运行 Unreal 引擎的主机机器的地址。

## 遥控器

有几种选项可以使用遥控器或操纵杆（如 Xbox 游戏手柄）来飞行模拟无人机。请参见 [遥控器](remote_control.md#rc-setup-for-px4)