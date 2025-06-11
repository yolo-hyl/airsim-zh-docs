# 遥控

要手动飞行，您需要遥控器或 RC。如果您没有遥控器，可以使用 [APIs](apis.md) 进行编程飞行，或使用所谓的 [计算机视觉模式](image_apis.md) 通过键盘移动。

## 默认配置的 RC 设置

默认情况下，AirSim 使用 [simple_flight](simple_flight.md) 作为其飞行控制器，通过 USB 端口连接到计算机的 RC。

您可以使用 Xbox 控制器或 [FrSky Taranis X9D Plus](https://hobbyking.com/en_us/frsky-2-4ghz-accst-taranis-x9d-plus-and-x8r-combo-digital-telemetry-radio-system-mode-2.html)。请注意，Xbox 360 控制器不够精确，不建议使用，如果您想要更真实的体验。如果有问题，请查看下面的常见问题解答。

### 其他设备

AirSim 可以检测各种设备，但除上述之外的设备 *可能* 需要额外配置。未来我们将添加通过 settings.json 设置此配置的能力。目前，如果无法正常工作，您可以尝试一些变通办法，如 [x360ce](http://www.x360ce.com/) 或修改 [SimJoystick.cpp 文件](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Source/SimJoyStick/SimJoyStick.cpp#L50) 中的代码。

### 关于 FrSky Taranis X9D Plus 的说明

[FrSky Taranis X9D Plus](https://hobbyking.com/en_us/frsky-2-4ghz-accst-taranis-x9d-plus-and-x8r-combo-digital-telemetry-radio-system-mode-2.html) 是一款真正的无人机遥控器，具有 USB 端口的优势，因此可以直接连接到 PC。您可以 [下载 AirSim 配置文件](misc/AirSim_FrSkyTaranis.bin) 并 [按照此教程](https://www.youtube.com/watch?v=qe-13Gyb0sw) 进行配置导入。然后，您应该在 RC 中看到 "sim" 模型，并且所有通道都配置正确。

### 关于 Linux 的说明

当前，Linux 上的默认配置是使用 Xbox 控制器。这意味着其他设备可能无法正常工作。未来我们将添加在 settings.json 中配置 RC 的能力，但目前您 *可能* 需要修改 [SimJoystick.cpp 文件](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Source/SimJoyStick/SimJoyStick.cpp#L340) 中的代码以使用其他设备。

## PX4 的 RC 设置

AirSim 支持 PX4 飞行控制器，但需要不同的设置。您可以与四旋翼配合使用多种遥控选项。我们已成功使用 FrSky Taranis X9D Plus、FlySky FS-TH9X 和 Futaba 14SG 与 AirSim。以下是配置 RC 的高层步骤：

1. 如果您打算使用硬件在环模式，您需要特定品牌 RC 的发射器并进行绑定。您可以在 RC 的用户手册中找到这些信息。
2. 对于硬件在环模式，您需要将发射器连接到 Pixhawk。通常，您可以在网上找到文档或 YouTube 视频教程来了解如何操作。
3. [在 QGroundControl 中校准您的 RC](https://docs.qgroundcontrol.com/en/SetupView/Radio.html)。

请参阅 [PX4 RC 配置](https://docs.px4.io/en/getting_started/rc_transmitter_receiver.html) 及 [此指南](https://docs.px4.io/master/en/getting_started/rc_transmitter_receiver.html#px4-compatible-receivers) 获取更多信息。

### 使用 Xbox 360 USB 游戏手柄

您还可以在 SITL 模式下使用 Xbox 控制器，但其精度不如真正的 RC 控制器。
有关如何设置的详细信息，请参见 [xbox controller](xbox_controller.md)。

### 使用 Playstation 3 控制器

确认 Playstation 3 控制器可作为 AirSim 控制器使用。然而，在 Windows 上，需要一个模拟器使其看起来像 Xbox 360 控制器。许多不同的解决方案可以在网上找到，例如 [x360ce Xbox 360 Controller Emulator](https://github.com/x360ce/x360ce)。

### DJI 控制器

Nils Tijtgat 写了一篇优秀的博客，介绍如何让 [DJI 控制器在 AirSim 中工作](https://timebutt.github.io/static/using-a-phantom-dji-controller-in-airsim/)。

## 常见问题解答

#### 我正在使用默认配置，AirSim 说我的 RC 没有在 USB 上被检测到。

通常情况下，如果您连接了多个 RC 和/或 Xbox/Playstation 游戏手柄等，会发生这种情况。在 Windows 中，按 Windows+S 键搜索 "设置 USB 游戏控制器"（在旧版本的 Windows 中，请尝试 "操纵杆"）。这将显示您连接到 PC 的所有游戏控制器。如果您没有看到自己的控制器，则 Windows 尚未检测到它，因此您需要先解决此问题。如果您看到了控制器，但它不在列表的顶部（即索引为 0），则需要告诉 AirSim，因为 AirSim 默认会尝试使用索引为 0 的 RC。为此，请导航到 `~/Documents/AirSim` 文件夹，打开 `settings.json` 并添加/修改以下设置。如下所示将告诉 AirSim 使用索引为 2 的 RC。
```
{
    "SettingsVersion": 1.2,
    "SimMode": "Multirotor",
    "Vehicles": {
        "SimpleFlight": {
            "VehicleType": "SimpleFlight",
            "RC": {
              "RemoteControlID": 2
            }
        }
    }
}
```

#### 使用 Xbox/PS3 控制器时，飞行器似乎不稳定

常规游戏手柄的精度较低，噪声较大。大多数情况下，您可能会看到显著的偏差（即当摇杆在零位时，输出不是零）。因此这种行为是预期的。

#### AirSim 中的 RC 校准在哪里？

我们还没有实现这一功能。这意味着您目前的 RC 固件需要具备进行校准的能力。

#### 我的 RC 在 PX4 设置下不起作用。

首先，您需要确保您的 RC 在 [QGroundControl](https://docs.qgroundcontrol.com/en/SetupView/Radio.html) 中正常工作。如果不工作，那么在 AirSim 中肯定也不会工作。PX4 模式适合那些至少具备中级水平经验的用户，以应对与 PX4 相关的各种问题，通常建议您寻求来自 PX4 论坛的帮助。