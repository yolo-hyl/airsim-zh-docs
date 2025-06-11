# 代码结构

## AirLib

大部分代码位于 AirLib。这是一个自包含的库，您应该能够使用任何 C++11 编译器进行编译。

AirLib 包含以下组件：

1. [*物理引擎:*](https://github.com/microsoft/AirSim/tree/main/AirLib/include/physics) 这是一个仅包含头文件的物理引擎。它旨在快速且可扩展，以实现不同的车辆。
2. [*传感器模型:*](https://github.com/microsoft/AirSim/tree/main/AirLib/include/sensors) 这是用于气压计、IMU、GPS 和磁力计的仅包含头文件的模型。
3. [*车辆模型:*](https://github.com/microsoft/AirSim/tree/main/AirLib/include/vehiclesr) 这是用于车辆配置和模型的仅包含头文件的模型。目前我们为多旋翼模型和 PX4 QuadRotor 在 X 配置中实现了配置。多个不同的多旋翼模型在 MultiRotorParams.hpp 中定义，包括六旋翼。
4. [*API 相关文件:*](https://github.com/microsoft/AirSim/tree/main/AirLib/include/api) 这部分 AirLib 为我们的 API 提供了抽象基类和特定车辆平台（如 MavLink）的具体实现。它还包含 RPC 客户端和服务器的类。

除此之外，所有常用工具在 [`common/`](https://github.com/microsoft/AirSim/tree/main/AirLib/include/common) 子文件夹中定义。这里一个重要的文件是 [AirSimSettings.hpp](https://github.com/microsoft/AirSim/blob/main/AirLib/include/common/AirSimSettings.hpp)，如果要在 `settings.json` 中添加任何新字段，需要进行修改。

AirSim 支持多旋翼的不同固件，如其自己的 SimpleFlight、PX4 和 ArduPilot，与每个固件通信的文件存放在 [`multirotor/firmwares`](https://github.com/microsoft/AirSim/tree/main/AirLib/include/vehicles/multirotor/firmwares) 其各自的子文件夹中。

车辆特定的 API 在 `api/` 子文件夹中定义，附带所需的结构体。[`AirLib/src/`](https://github.com/microsoft/AirSim/tree/main/AirLib/src) 包含实现 .hpp 文件中定义的各种方法的 .cpp 文件。例如，[MultirotorApiBase.cpp](https://github.com/microsoft/AirSim/blob/main/AirLib/src/vehicles/multirotor/api/MultirotorApiBase.cpp) 包含多旋翼 API 的基本实现，如果需要，也可以在特定固件文件中重写。

## Unreal/Plugins/AirSim

这是项目中唯一依赖于 Unreal 引擎的部分。我们将其单独隔离，以便可以为其他平台实现模拟器，就像为 [Unity](https://microsoft.github.io/AirSim/Unity.html) 所做的那样。Unreal 代码利用了基于 UObject 的类，包括蓝图。`Source/` 文件夹包含 C++ 文件，而 `Content/` 文件夹则包含蓝图和资产。以下是一些主要组件的描述：

1. *SimMode_ 类:* SimMode 类帮助实现多种不同模式，例如纯计算机视觉模式，其中没有车辆或特定车辆的模拟（目前为汽车和多旋翼）。车辆类位于 [`Vehicles/`](https://github.com/microsoft/AirSim/tree/main/Unreal/Plugins/AirSim/Source/Vehicles)
2. *PawnSimApi:* 这是所有车辆角色可视化的 [基类](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Source/PawnSimApi.cpp)。每个车辆都有自己的子类（Multirotor|Car|ComputerVision）Pawn 类。
3. [UnrealSensors](https://github.com/microsoft/AirSim/tree/main/Unreal/Plugins/AirSim/Source/UnrealSensors): 包含距离和激光雷达传感器的实现。
4. *WorldSimApi:* 实现了大多数环境和与车辆无关的 API。

除此之外，[`PIPCamera`](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Source/PIPCamera.cpp) 包含相机初始化，[`UnrealImageCapture`](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Source/UnrealImageCapture.cpp) 和 [`RenderRequest`](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Source/RenderRequest.cpp) 包含图像渲染代码。[`AirBlueprintLib`](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Source/AirBlueprintLib.cpp) 包含许多用于与 UE4 引擎接口的实用程序和包装方法。

## MavLinkCom

这是由我们的团队成员 [Chris Lovett](https://github.com/lovettchris) 开发的库，提供与 MavLink 设备通信的 C++ 类。该库是独立的，可以在任何项目中使用。更多信息请参见 [MavLinkCom](mavlinkcom.md)。

## 示例程序

我们创建了一些示例程序，以演示如何使用 API。请查看 HelloDrone 和 DroneShell。DroneShell 演示如何使用 UDP 连接到模拟器。模拟器正在运行一个服务器（类似于 DroneServer）。

## PythonClient

[PythonClient](https://github.com/microsoft/AirSim/tree/main/PythonClient) 包含 Python API 包装文件和演示其用法的示例程序。

## 贡献

请参见 [贡献指南](CONTRIBUTING.md)。

## Unreal 框架

下图说明了 AirSim 如何被 Unreal 游戏引擎加载和调用：

![AirSimConstruction](images/airsim_startup.png)