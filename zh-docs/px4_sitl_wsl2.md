# PX4 软件在环测试与 WSL 2

[Windows 子系统 Linux 版本 2](https://docs.microsoft.com/en-us/windows/wsl/install-win10) 使用虚拟机器，这与您的 Windows 主机有一个独立的 IP 地址。这意味着 PX4 不能使用 "localhost" 来找到 AirSim，这是 PX4 的默认行为。

您会注意到在 Windows 上，`ipconfig` 返回一个新的 WSL 以太网适配器，类似如下（请注意 vEthernet 的名称中有 `(WSL)`）：

```plain
Ethernet adapter vEthernet (WSL):

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::1192:f9a5:df88:53ba%44
   IPv4 Address. . . . . . . . . . . : 172.31.64.1
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . :
```

这个地址 `172.31.64.1` 是 WSL 2 用于访问您的 Windows 主机的地址。

从这个 [PX4 更改请求](https://github.com/PX4/PX4-Autopilot/commit/1719ff9892f3c3d034f2b44e94d15527ab09cec6) 开始（对应于版本 v1.12.0-beta1 或更新版本），PX4 在 SITL 模式下现在可以连接到不同（远程）的 IP 地址上的 AirSim。要启用此功能，请确保您使用的是包含此修复的 PX4 版本，并在 Linux 中设置以下环境变量：

```shell
export PX4_SIM_HOST_ADDR=172.31.64.1
```

**注意：** 请确保更新上述地址 `172.31.64.1` 以匹配您从 `ipconfig` 命令中看到的地址。

使用防火墙配置打开传入的 TCP 端口 4560 和传入的 UDP 端口 14540。

现在在 Linux 侧运行 `ip address show` 并复制 `eth0 inet` 地址，它应该类似于 `172.31.66.156`。这是 Windows 需要知道的，以便找到 PX4。

编辑您的 [AirSim 设置](settings.md) 文件，并添加 `LocalHostIp` 来告诉 AirSim 使用 WSL 以太网适配器地址，而不是默认的 `localhost`。这将使 AirSim 在该适配器上打开 TCP 端口，这是 PX4 应用程序将要查找的地址。同时告诉 AirSim 通过将 `ControlIp` 设置为魔法字符串 `remote` 来连接 `ControlIp` UDP 通道。这个设置解析为 TCP 套接字中找到的 WSL 2 远程 IP 地址。

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
            "ControlIp": "remote",
            "ControlPortLocal": 14540,
            "ControlPortRemote": 14580,
            "LocalHostIp": "172.31.64.1",
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
请参见 [PX4 LockStep](px4_lockstep.md) 获取更多信息。"Barometer" 设置使 PX4 处于正常状态，因为默认的 AirSim 气压计产生的噪音有点太多。此设置稍微限制了噪音。

如果您的本地库不包含 [此 PX4 提交](https://github.com/PX4/PX4-Autopilot/commit/292a66ce417c9769e1a7845fbc9b8d5e68e1cf0b)，请编辑 `ROMFS/px4fmu_common/init.d-posix/rcS` 中的 Linux 文件，并确保它正在查找 `PX4_SIM_HOST_ADDR` 环境变量，并将其传递给 PX4 模拟器，如下所示：

```shell
# 如果 PX4_SIM_HOST_ADDR 环境变量为空，则使用 localhost。
if [ -z "${PX4_SIM_HOST_ADDR}" ]; then
    echo "PX4 SIM HOST: localhost"
    simulator start -c $simulator_tcp_port
else
    echo "PX4 SIM HOST: $PX4_SIM_HOST_ADDR"
    simulator start -t $PX4_SIM_HOST_ADDR $simulator_tcp_port
fi
```

**注意：** 根据您使用的 PX4 版本，这段代码可能已经存在。

**注意：** 在等待消息时请耐心：

```
INFO  [simulator] Simulator connected on TCP port 4560.
```

建立远程连接可能比使用 `localhost` 需要更长的时间。

现在您可以继续进行 [设置 PX4 软件在环测试](px4_sitl.md) 中显示的步骤。