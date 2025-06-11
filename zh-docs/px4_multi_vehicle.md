# 设置多车辆 PX4 仿真

[PX4 SITL 堆栈](px4_sitl.md) 附带了一个 `sitl_multiple_run.sh` Shell 脚本，可以运行多个 PX4 二进制实例。这将使 SITL 堆栈能够在多个 TCP 端口上监听来自多个 AirSim 车辆的连接，端口从 4560 开始。
然而，提供的脚本并不允许我们查看 PX4 控制台。如果您希望手动运行实例并能够查看每个实例的控制台（**推荐**），请参阅 [本节](px4_multi_vehicle.md#starting-sitl-instances-with-px4-console)。

## 设置多个 PX4 软件在环实例

**注意** 您需要使用 `make px4_sitl_default none_iris` 构建 PX4，如 [这里](px4_sitl.md#setting-up-px4-software-in-loop) 所示，然后才能尝试运行多个 PX4 实例。

1. 从您的 Bash（或 Cygwin）终端转到 PX4 固件目录并运行 `sitl_multiple_run.sh` 脚本，同时指定所需的车辆数量：
    ```    
    cd PX4-Autopilot
    ./Tools/sitl_multiple_run.sh 2    # 这里的 2 是车辆/实例的数量
    ```
    这将启动多个实例，监听从 4560 到 4560+i 的 TCP 端口，其中 'i' 是指定的车辆/实例数量。

2. 您应该收到一条确认消息，表示旧实例已停止，新实例已启动：
    ```
    killing running instances
    starting instance 0 in /cygdrive/c/PX4/home/PX4/Firmware/build/px4_sitl_default/instance_0
    starting instance 1 in /cygdrive/c/PX4/home/PX4/Firmware/build/px4_sitl_default/instance_1
    ```

3. 现在编辑 [AirSim 设置](settings.md) 文件，以确保您为设置的车辆数量匹配 TCP 端口设置，并确保两个车辆不会在同一点生成。

    例如，这些设置将生成两个 PX4Multirotors，其中一个尝试连接到 PX4 SITL 的端口 `4560`，另一个连接到端口 `4561`。它还确保车辆在 `0,1,0` 和 `0,-1,0` 处生成，以避免碰撞：
    ```json
    {
        "SettingsVersion": 1.2,
        "SimMode": "Multirotor",
        "Vehicles": {
            "Drone1": {
                "VehicleType": "PX4Multirotor",
                "UseSerial": false,
                "UseTcp": true,
                "TcpPort": 4560,
                "ControlPortLocal": 14540,
                "ControlPortRemote": 14580,
                "X": 0, "Y": 1, "Z": 0
            },
            "Drone2": {
                "VehicleType": "PX4Multirotor",
                "UseSerial": false,
                "UseTcp": true,
                "TcpPort": 4561,
                "ControlPortLocal": 14541,
                "ControlPortRemote": 14581,       
                "X": 0, "Y": -1, "Z": 0
            }
        }
      }
    ```
    您可以添加超过两个车辆，但需要确保调整每个车辆的 TCP 端口（即：车辆 3 的端口为 `4562`，依此类推），并调整生成点。

4. 现在运行您的 Unreal AirSim 环境，它应该通过 TCP 连接到 SITL PX4。
如果您在 [可见 PX4 控制台](px4_multi_vehicle.md#Starting-sitl-instances-with-px4-console) 的情况下运行实例，您应该会看到每个 SITL PX4 窗口中的一堆消息。
具体来说，以下消息告诉您 AirSim 已正确连接，GPS 融合稳定：
    ```
    INFO  [simulator] Simulator connected on UDP port 14560
    INFO  [mavlink] partner IP: 127.0.0.1
    INFO  [ecl/EKF] EKF GPS checks passed (WGS-84 origin set)
    INFO  [ecl/EKF] EKF commencing GPS fusion
    ```

    如果您没有看到这些消息，请检查您的端口设置。

5. 您还应该能够在 SITL 模式下使用 QGroundControl。确保没有连接 Pixhawk 硬件，否则 QGroundControl 会选择使用该硬件。请注意，由于我们没有物理板，遥控器无法直接连接。因此，可以选择使用 Xbox 360 控制器，或通过 USB 连接您的遥控器（例如，FrSky Taranis X9D Plus 的情况），或使用训练 USB 电缆连接到您的 PC。这使您的遥控器看起来像一个摇杆。您需要在 QGroundControl 中进行额外设置，以使用虚拟摇杆进行遥控控制。只有在您计划在 AirSim 中手动飞行无人机时才需要这样做。使用 Python API 的自主飞行不需要遥控器，详见 [`无遥控`](px4_sitl.md#No-Remote-Control)。

## 使用 PX4 控制台启动 SITL 实例

如果您希望在能够查看 PX4 控制台的情况下启动 SITL 实例，您需要运行 [这里](https://github.com/microsoft/AirSim/tree/main/PX4Scripts) 的 Shell 脚本，而不是 `sitl_multiple_run.sh`。
以下是如何操作：

**注意** 此脚本也假设 PX4 使用 `make px4_sitl_default none_iris` 构建，如 [这里](px4_sitl.md#setting-up-px4-software-in-loop) 所示，然后才能尝试运行多个 PX4 实例。

1. 从您的 Bash（或 Cygwin）终端转到 PX4 目录并获取脚本（将其放置在 PX4 目录中名为 Scripts 的子目录中，如下所示）：
    ```
    cd PX4
    mkdir -p Scripts
    cd Scripts
    wget https://github.com/microsoft/AirSim/raw/main/PX4Scripts/sitl_kill.sh
    wget https://github.com/microsoft/AirSim/raw/main/PX4Scripts/run_airsim_sitl.sh
    ```
    **注意** Shell 脚本期望 `Scripts` 和 `Firmware` 目录在同一父目录中。此外，您可能需要运行 `chmod +x sitl_kill.sh` 和 `chmod +x run_airsim_sitl.sh` 使脚本可执行。

2. 运行 `sitl_kill.sh` 脚本以杀死所有活动的 PX4 SITL 实例：
    ```
    ./sitl_kill.sh
    ```
    
3. 运行 `run_airsim_sitl.sh` 脚本，同时指定您想在当前终端窗口中运行的实例（第一个实例编号为 0）：
    ```
    ./run_airsim_sitl.sh 0 # 第一个实例 = 0
    ```
    
    您应该看到 PX4 实例启动并等待 AirSim 的连接，就像单个实例一样：
    ```
    ______  __   __    ___
    | ___ \ \ \ / /   /   |
    | |_/ /  \ V /   / /| |
    |  __/   /   \  / /_| |
    | |     / /^\ \ \___  |
    \_|     \/   \/     |_/

    px4 starting.
    INFO  [px4] Calling startup script: /bin/sh /cygdrive/c/PX4/home/PX4/Firmware/etc/init.d-posix/rcS 0
    INFO  [dataman] Unknown restart, data manager file './dataman' size is 11798680 bytes
    INFO  [simulator] Waiting for simulator to connect on TCP port 4560
    ```

4. 打开一个新终端并转到 Scripts 目录，启动下一个实例：
    ```
    cd PX4
    cd Scripts
    ./run_airsim_sitl.sh 1  # ,2,3,4,..,etc
    ```

5. 重复步骤 4，启动尽可能多的实例。

6. 运行您的 Unreal AirSim 环境，它应该通过 TCP 连接到 SITL PX4（假设您的 settings.json 文件具有正确的端口）。