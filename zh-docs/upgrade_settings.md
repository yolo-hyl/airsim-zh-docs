# 更新设置

AirSim 1.2 中的设置架构发生了变化，以提供更大的灵活性和更清晰的界面。如果您有旧的 [settings.json](settings.md) 文件，您可以选择删除它并重启 AirSim，或者使用本指南进行手动升级。

## 更快的方式
我们建议简单地删除 [settings.json](settings.md) 并重新添加所需的设置。
有关可用设置的完整信息，请参见 [文档](settings.md)。

## 更改

### 使用场景
以前我们使用 `UsageScenario` 来指定 `ComputerVision` 模式。现在改为使用 `"SimMode": "ComputerVision"`。

### 摄像头默认值和更改摄像头设置
以前我们在根目录下有 `CaptureSettings` 和 `NoiseSettings`。现在这些设置被合并到新的 `CameraDefaults` 元素中。该 [元素的架构](settings.md#camera_settings) 后续用于配置车辆上的摄像头。

### 云台
[云台元素](settings.md#Gimbal)（取代旧的 Gimble 元素）现在已从 `CaptureSettings` 中移出。

### 摄像头ID到摄像头名称
所有设置现在通过 [名称](image_apis.md#available_cameras) 而不是 ID 来引用摄像头。

### 使用 PX4
新的 Vehicles 元素允许指定要创建哪些车辆。要使用 PX4，请参见 [此部分](settings.md#using_px4)。

### 附加摄像头
旧的 `AdditionalCameras` 设置现在被车辆设置中的 [Cameras 元素](settings.md#Common_Vehicle_Setting) 所替代。