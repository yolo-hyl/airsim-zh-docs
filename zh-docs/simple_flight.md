# simple_flight

如果你不知道飞行控制器的作用，请参见 [What is Flight Controller?](flight_controller.md)。

AirSim 内置了名为 simple_flight 的飞行控制器，默认使用。你无需做任何事情来使用或配置它。AirSim 还支持 [PX4](px4_setup.md) 作为高级用户的另一种飞行控制器。未来，我们还计划支持 [ROSFlight](https://rosflight.org/) 和 [Hackflight](https://github.com/simondlevy/hackflight)。

## 优点

使用 simple_flight 的优点在于你无需额外的设置，它“就能工作”。此外，simple_flight 使用可步进的时钟，这意味着你可以暂停仿真，且不受操作系统提供的高方差低精度时钟的影响。此外，simple_flight 设计简单，跨平台，包含 100% 头文件唯一并且无依赖的 C++ 代码，这意味着你可以在同一个代码库中轻松切换模拟器和飞行控制器代码！

## 设计

通常，飞行控制器是设计用在实际车辆硬件上的，它们在模拟器中的支持程度差异很大。对于非专业用户来说，它们通常相当难以配置，且通常构建复杂，通常缺乏跨平台支持。所有这些问题在设计 simple_flight 时都起了重要作用。

simple_flight 从一开始就被设计为一个库，具有干净的接口，既可以在车辆上运行，也可以在模拟器中运行。核心原则是，飞行控制器没有办法指定特殊的模拟模式，因此它无法判断自己是作为模拟运行还是作为真实车辆运行。因此，我们视飞行控制器为一组包装在库中的算法。另一个重点是开发这段代码为无依赖的头文件纯标准 C++11 代码。这意味着编译 simple_flight 不需要特殊的构建。你只需将其源代码复制到任何你希望的项目中，它就能正常工作。

## 控制

simple_flight 可以通过接收期望的输入（如角速度、角度、速度或位置）来控制车辆。每个控制轴可以用这些模式中的一个来指定。在内部，simple_flight 使用级联 PID 控制器来最终生成致动器信号。这意味着位置 PID 驱动速度 PID，进而驱动角度 PID，最终驱动角速度 PID。

## 状态估计

在当前版本中，我们使用来自模拟器的真实状态进行状态估计。我们计划在不久的将来添加基于互补滤波的状态估计器，用于角速度和方向，使用两个传感器（陀螺仪，加速度计）。从长远来看，我们计划整合另一个库，通过使用 4 个传感器（陀螺仪，加速度计，磁力计和气压计）及扩展卡尔曼滤波器（EKF）进行速度和位置估计。如果你在这个领域有经验，我们鼓励你与我们互动并贡献！

## 支持的板卡

目前，我们已经为模拟板实现了 simple_flight 接口。我们计划为 Pixhawk V2 板实现它，可能还会为 Naze32 板实现。我们希望我们所有的代码保持不变，实施过程主要涉及为各种传感器添加驱动程序，处理ISR和管理其他特定于板卡的细节。如果你在这一领域有经验，我们鼓励你与我们互动并贡献！

## 配置

要让 AirSim 使用 simple_flight，你可以在 [settings.json](settings.md) 中这样指定。请注意，这是默认设置，因此你不必明确指定。

```
"Vehicles": {
    "SimpleFlight": {
      "VehicleType": "SimpleFlight",
    }
}
```

默认情况下，使用 simple_flight 的车辆已经被解除锁定，因此你会看到其螺旋桨旋转。然而，如果你不希望这样，请将 `DefaultVehicleState` 设置为 `Inactive`，如下所示：

```
"Vehicles": {
    "SimpleFlight": {
      "VehicleType": "SimpleFlight",
      "DefaultVehicleState": "Inactive"
    }
}
```

在这种情况下，你需要通过将遥控杆放在向下内侧的位置或使用 API 手动解除锁定。

出于安全原因，飞行控制器禁止 API 控制，除非人类操作员通过其遥控器上的开关同意使用。此外，当遥控控制丢失时，车辆应禁用 API 控制并进入悬停模式以确保安全。为简化操作，simple_flight 默认情况下允许在没有人类同意的情况下使用 RC 的 API 控制，甚至在未检测到 RC 时也是如此。然而，你可以使用以下设置进行更改：

```
"Vehicles": {
    "SimpleFlight": {
      "VehicleType": "SimpleFlight",

      "AllowAPIAlways": true,
      "RC": {
        "RemoteControlID": 0,      
        "AllowAPIWhenDisconnected": true
      }
    }
}
```

最后，simple_flight 默认使用可步进时钟，这意味着时钟在模拟器告诉它前进时才会前进（与严格按时间流逝前进的墙壁时钟不同）。这意味着时钟可以暂停，例如，当代码命中断点时，时钟没有方差（操作系统提供的时钟 API 可能有显著方差，除非它是“实时”操作系统）。如果你希望 simple_flight 使用墙壁时钟，则使用以下设置：

```
  "ClockType": "ScalableClock"
```