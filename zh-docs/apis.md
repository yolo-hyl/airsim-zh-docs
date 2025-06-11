# AirSim API

## 简介
AirSim 提供 API，让您可以以编程方式与模拟中的车辆进行交互。您可以使用这些 API 来检索图像、获取状态、控制车辆等等。

## Python 快速入门
如果您想使用 Python 调用 AirSim API，建议使用 Anaconda 和 Python 3.5 或更高版本，然而某些代码也可能在 Python 2.7 上运行（[帮助我们](CONTRIBUTING.md) 改善兼容性！）。

首先安装该软件包：

```
pip install msgpack-rpc-python
```

您可以从 [releases](https://github.com/Microsoft/AirSim/releases) 获取 AirSim 二进制文件，或从源代码编译（[Windows](build_windows.md)，[Linux](build_linux.md)）。一旦可以运行 AirSim，选择“Car”作为车辆，然后导航到 `PythonClient\car\` 文件夹，运行：

```
python hello_car.py
```

如果您使用的是 Visual Studio 2019，只需打开 AirSim.sln，将 PythonClient 设置为启动项目，并选择 `car\hello_car.py` 作为启动脚本。

### 安装 AirSim 包
您还可以通过以下命令简单地安装 `airsim` 包：

```
pip install airsim
```

您可以在您的代码库中的 `PythonClient` 文件夹中找到该包的源代码和示例。

**注意**
1. 您可能会在我们的示例文件夹中注意到一个名为 `setup_path.py` 的文件。该文件具有简单代码，用于检测父文件夹中是否可用 `airsim` 包，并在这种情况下使用它而不是通过 pip 安装的包，因此您始终使用最新的代码。
2. AirSim 仍在大规模开发中，这意味着您可能需要频繁更新该软件包以使用新 API。

## C++ 用户
如果您想使用 C++ API 和示例，请参见 [C++ API 指南](apis_cpp.md)。

## Hello Car
以下是如何使用 Python 的 AirSim API 控制模拟汽车的示例（也参见 [C++ 示例](apis_cpp.md#hello_car)）：

```python
# 准备运行示例: PythonClient/car/hello_car.py
import airsim
import time

# 连接到 AirSim 模拟器
client = airsim.CarClient()
client.confirmConnection()
client.enableApiControl(True)
car_controls = airsim.CarControls()

while True:
    # 获取汽车状态
    car_state = client.getCarState()
    print("Speed %d, Gear %d" % (car_state.speed, car_state.gear))

    # 设置汽车控制
    car_controls.throttle = 1
    car_controls.steering = 1
    client.setCarControls(car_controls)

    # 让汽车行驶一段时间
    time.sleep(1)

    # 从汽车获取摄像头图像
    responses = client.simGetImages([
        airsim.ImageRequest(0, airsim.ImageType.DepthVis),
        airsim.ImageRequest(1, airsim.ImageType.DepthPlanar, True)])
    print('Retrieved images: %d', len(responses))

    # 处理图像
    for response in responses:
        if response.pixels_as_float:
            print("Type %d, size %d" % (response.image_type, len(response.image_data_float)))
            airsim.write_pfm('py1.pfm', airsim.get_pfm_array(response))
        else:
            print("Type %d, size %d" % (response.image_type, len(response.image_data_uint8)))
            airsim.write_file('py1.png', response.image_data_uint8)
```

## Hello Drone
以下是如何使用 Python 的 AirSim API 控制模拟四旋翼的示例（也参见 [C++ 示例](apis_cpp.md#hello_drone)）：

```python
# 准备运行示例: PythonClient/multirotor/hello_drone.py
import airsim
import os

# 连接到 AirSim 模拟器
client = airsim.MultirotorClient()
client.confirmConnection()
client.enableApiControl(True)
client.armDisarm(True)

# 异步方法返回 Future。调用 join() 等待任务完成。
client.takeoffAsync().join()
client.moveToPositionAsync(-10, 10, -10, 5).join()

# 拍摄图像
responses = client.simGetImages([
    airsim.ImageRequest("0", airsim.ImageType.DepthVis),
    airsim.ImageRequest("1", airsim.ImageType.DepthPlanar, True)])
print('Retrieved images: %d', len(responses))

# 处理图像
for response in responses:
    if response.pixels_as_float:
        print("Type %d, size %d" % (response.image_type, len(response.image_data_float)))
        airsim.write_pfm(os.path.normpath('/temp/py1.pfm'), airsim.get_pfm_array(response))
    else:
        print("Type %d, size %d" % (response.image_type, len(response.image_data_uint8)))
        airsim.write_file(os.path.normpath('/temp/py1.png'), response.image_data_uint8)
```

## 常用 API

* `reset`：重置车辆至其原始起始状态。请注意，在调用 `reset` 后，您必须再次调用 `enableApiControl` 和 `armDisarm`。
* `confirmConnection`：每秒检查连接状态并在控制台中报告，以便用户看到连接的进度。
* `enableApiControl`：出于安全原因，默认情况下，不启用自动车辆的 API 控制，人类操作员具有完全控制权（通常通过遥控器或模拟器中的操纵杆）。客户端必须调用此方法以请求通过 API 控制。人类操作员可能已禁止 API 控制，这意味着 `enableApiControl` 将无效。这可以通过 `isApiControlEnabled` 检查。
* `isApiControlEnabled`：如果建立了 API 控制，则返回 true。否则（默认值为 false），API 调用将被忽略。在成功调用 `enableApiControl` 后，`isApiControlEnabled` 应返回 true。
* `ping`：如果连接已建立，则此调用将返回 true，否则将阻塞直到超时。
* `simPrintLogMessage`：在模拟器窗口中打印指定消息。如果提供了 `message_param`，则其将在消息旁边打印；在这种情况下，如果再次使用相同消息值但不同 `message_param` 调用此 API，则上一行将替换为新行（而不是 API 在显示上创建新行）。例如，`simPrintLogMessage("Iteration: ", to_string(i))` 在调用 API 时会在显示上不断更新相同的行。
* `simGetObjectPose`，`simSetObjectPose`：获取和设置 Unreal 环境中指定对象的位姿。这里的对象是 Unreal 术语中的 “actor”。它们通过标签以及名称进行搜索。请注意，UE 编辑器中显示的名称在每次运行时都是 *自动生成* 的，并不是永久性的。因此，如果您想按名称引用 actor，必须在 UE 编辑器中更改其自动生成的名称。或者，您可以向 actor 添加标签，可以通过在 Unreal 编辑器中单击该 actor 然后转到 [Tags 属性](https://answers.unrealengine.com/questions/543807/whats-the-difference-between-tag-and-tag.html)，单击 "+" 符号并添加一些字符串值。如果多个 actor 具有相同标签，则返回第一个匹配项。如果找不到匹配项，则返回 NaN 位姿。返回的位姿是世界坐标系中的 NED 坐标，单位为 SI。对于 `simSetObjectPose`，指定的 actor 必须将 [Mobility](https://docs.unrealengine.com/en-us/Engine/Actors/Mobility) 设置为 Movable，否则会产生未定义行为。`simSetObjectPose` 有参数 `teleport`，表示对象在移动过程中会 [穿过其他物体](https://www.unrealengine.com/en-US/blog/moving-physical-objects)，并且如果移动成功则返回 true。

### 图像 / 计算机视觉 API
AirSim 提供全面的图像 API，以从多个摄像头同步获取图像，同时包括深度、视差、表面法线和视觉的地面真实数据。您可以在 [settings.json](settings.md) 中设置分辨率、视场、运动模糊等参数。还有 API 用于检测碰撞状态。参见 [完整代码](https://github.com/Microsoft/AirSim/tree/main/Examples/DataCollection/StereoImageGenerator.hpp)，该代码生成指定数量的立体图像和深度的地面真实数据，并进行相机平面的归一化计算视差图像并将其保存为 [pfm 格式](pfm.md)。

有关图像 API 和计算机视觉模式的更多信息，请参见 [image APIs and Computer Vision mode](image_apis.md)。
对于可以从领域随机化中受益的视觉问题，还有一个 [object retexturing API](retexturing.md)，可用于受支持的场景中。

### 暂停和继续 API
AirSim 允许通过 `pause(is_paused)` API 暂停和继续模拟。要暂停模拟，调用 `pause(True)`，要继续模拟，调用 `pause(False)`。您可能会遇到这样的场景，特别是在使用强化学习时，运行模拟指定时间后自动暂停。在模拟暂停时，您可以进行一些耗时的计算，发送新的命令，然后再次运行模拟指定时间。这可以通过 API `continueForTime(seconds)` 实现。此 API 在指定秒数内运行模拟，然后暂停模拟。例如用法，请参见 [pause_continue_car.py](https://github.com/Microsoft/AirSim/tree/main/PythonClient//car/pause_continue_car.py) 和 [pause_continue_drone.py](https://github.com/Microsoft/AirSim/tree/main/PythonClient//multirotor/pause_continue_drone.py)。

### 碰撞 API
可以使用 `simGetCollisionInfo` API 获取碰撞信息。此调用返回一个结构，不仅包含碰撞是否发生的信息，还包含碰撞位置、表面法线、渗透深度等信息。

### 时间 API
AirSim 假设您的环境中存在类 `EngineSky/BP_Sky_Sphere` 的天空球，并带有 [ADirectionalLight actor](https://github.com/microsoft/AirSim/blob/v1.4.0-linux/Unreal/Plugins/AirSim/Source/SimMode/SimModeBase.cpp#L224)。默认情况下，场景中太阳的位置不会随时间移动。您可以使用 [settings](settings.md#timeofday) 设置纬度、经度、日期和时间，AirSim 将据此计算场景中太阳的位置。

您还可以使用以下 API 调用根据给定的日期时间设置太阳位置：

```
simSetTimeOfDay(self, is_enabled, start_datetime = "", is_start_datetime_dst = False, celestial_clock_speed = 1, update_interval_secs = 60, move_sun = True)
```

`is_enabled` 参数必须为 `True` 以启用昼夜效果。如果为 `False`，则太阳位置将重置为环境中的原始位置。

其他参数与 [settings](settings.md#timeofday) 中相同。

### 视线和世界范围 API
要测试模拟中车辆到某个点或两个点之间的视线，请参见 simTestLineOfSightToPoint(point, vehicle_name) 和 simTestLineOfSightBetweenPoints(point1, point2)。
模拟的世界范围，可以通过 simGetWorldExtents() 获取，以两 GeoPoints 的向量形式表示。

### 天气 API
默认情况下，所有天气效果都被禁用。要启用天气效果，首先调用：

```
simEnableWeather(True)
```

可以通过使用 `simSetWeatherParameter` 方法启用各种天气效果，该方法接受 `WeatherParameter`，例如：

```
client.simSetWeatherParameter(airsim.WeatherParameter.Rain, 0.25);
```
第二个参数的值范围为 0 到 1。第一个参数提供以下选项：

```
class WeatherParameter:
    Rain = 0
    Roadwetness = 1
    Snow = 2
    RoadSnow = 3
    MapleLeaf = 4
    RoadLeaf = 5
    Dust = 6
    Fog = 7
```

请注意，`Roadwetness`、`RoadSnow` 和 `RoadLeaf` 效果需要在场景中添加 [materials](https://github.com/Microsoft/AirSim/tree/main/Unreal/Plugins/AirSim/Content/Weather/WeatherFX)。

有关更多详情，请参见 [示例代码](https://github.com/Microsoft/AirSim/blob/main/PythonClient/environment/weather.py)。

### 录制 API

录制 API 可用于通过 API 开始录制数据。要录制的数据可以通过 [settings](settings.md#recording) 指定。要开始录制，请使用 -

```
client.startRecording()
```

同样，要停止录制，请使用 `client.stopRecording()`。要检查录制是否正在运行，请调用 `client.isRecording()`，返回一个 `bool`。

此 API 与使用 R 按钮切换录制同时工作，因此如果使用 R 键启用，`isRecording()` 将返回 `True`，并且可以通过 API 使用 `stopRecording()` 停止录制。同样，如果通过 API 启动的录制将在视口中按下 R 键时停止。如果通过 API 启动或停止录制，将在视口左上角显示 LogMessage。

请注意，这仅会按设置中指定的内容保存数据。要完全自由地存储数据，例如某些传感器信息，或以不同的格式或布局，请使用其他 API 来获取数据并按所需方式保存。有关如何修改被录制的运动学数据的详细信息，请查看 [Modifying Recording Data](modify_recording_data.md)。

### 风 API

可以通过 `simSetWind()` 在模拟期间更改风。风是在世界坐标系中指定的 NED 方向和 m/s 值。

例如，要设置 20m/s 风朝北（前方）方向 -

```python
# 将风设置为 (20,0,0) 在 NED（前方方向）
wind = airsim.Vector3r(20, 0, 0)
client.simSetWind(wind)
```

另请参见 [set_wind.py](https://github.com/Microsoft/AirSim/blob/main/PythonClient/multirotor/set_wind.py) 中的示例脚本。

### Lidar API
AirSim 提供 API 从车辆上的 Lidar 传感器获取点云数据。您可以在 [settings.json](settings.md) 中设置通道数量、每秒点数、水平和垂直视场等参数。

更多有关 [lidar API 和设置](lidar.md) 及 [sensor settings](sensors.md)。

### 灯光控制 API

可以通过 `simSpawnObject()` API 在 AirSim 内部创建可操作的灯光，方法是将 `PointLightBP` 或 `SpotLightBP` 作为 `asset_name` 参数传递，并将 `True` 作为 `is_blueprint` 参数。一旦灯光被生成，可以使用以下 API 进行操作：

* `simSetLightIntensity`：这允许您编辑灯光的亮度或强度。它接受两个参数，`light_name`，这是先前调用 `simSpawnObject()` 返回的灯光对象的名称，以及 `intensity`，一个浮动值。

### 纹理 API

可以通过以下 API 动态设置对象的纹理：

* `simSetObjectMaterial`：这使用现有的 Unreal 材料资产设置对象的材料。它接受两个字符串参数，`object_name` 和 `material_name`。
* `simSetObjectMaterialFromTexture`：这使用纹理路径设置对象的材料。它接受两个字符串参数，`object_name` 和 `texture_path`。

### 多辆车辆
AirSim 支持多辆车辆，并通过 API 进行控制。请参见 [多辆车辆](multi_vehicle.md) 文档。

### 坐标系统
所有 AirSim API 使用 NED 坐标系统，即 +X 为北，+Y 为东，+Z 为下。所有单位均为 SI 制。请注意，这与 Unreal Engine 内部使用的坐标系统不同。在 Unreal Engine 中，+Z 是向上而不是向下，长度单位为厘米而不是米。AirSim API 处理适当的转换。车辆的起始点始终为 NED 系统中的坐标 (0, 0, 0)。因此，在从 Unreal 坐标转换为 NED 之前，我们首先减去起始偏移量，然后将厘米转换为米乘以 100。车辆是在 Unreal 环境中生成的，玩家起始组件被放置。`OriginGeopoint` 在 [settings.json](settings.md) 中是一个设置，它将地理经度、纬度和海拔分配给玩家起始组件。

## 车辆特定 API
### 汽车的 API
汽车可用的 API 如下：

* `setCarControls`：这允许您设置油门、转向、手刹和自动或手动挡。
* `getCarState`：这获取状态信息，包括速度、当前挡位和 6 个运动学量：位置、方向、线速度、角速度、线加速度和角加速度。所有量均在 NED 坐标系统中，单位为 SI，但角速度和加速度在机体坐标系中。
* [图像 API](image_apis.md)。

### 多旋翼的 API
多旋翼可以通过指定角度、速度向量、目标位置或这些的某种组合进行控制。为此，均有相应的 `move*` API。当进行位置控制时，我们需要使用某些路径跟踪算法。默认情况下，AirSim 使用胡萝卜跟踪算法。这通常被称为“高水平控制”，因为您只需指定高水平目标，固件将处理其余部分。当前在 AirSim 中可用的最低级别控制是 `moveByAngleThrottleAsync` API。

#### getMultirotorState
此 API 在一次调用中返回车辆的状态。状态包括、碰撞、估算运动学（即通过传感器融合计算的运动学）和时间戳（自纪元以来的纳秒数）。这里的运动学意味着 6 个量：位置、方向、线速度、角速度、线加速度和角加速度。请注意，simple_slight 当前不支持状态估计，这意味着 estimated 和 ground truth 运动学值对 simple_flight 来说是相同的。除了角加速度外，PX4 仍然会提供估计的运动学。所有量均在 NED 坐标系统中，单位为 SI，但角速度和加速度在机体坐标系中。

#### Async 方法、持续时间和最大等待时间
许多 API 方法有命名为 `duration` 或 `max_wait_seconds` 的参数，并且以 *Async* 作为后缀，例如 `takeoffAsync`。这些方法将在任务在 AirSim 中启动后立即返回，以便您的客户端代码可以在任务执行时做其他事情。如果您希望等待此项任务完成，则可以像这样调用 `waitOnLastTask`：

```cpp
//C++
client.takeoffAsync()->waitOnLastTask();
```

```cpp
# Python
client.takeoffAsync().join()
```

如果您开始另一个命令，则将自动取消上一个任务并开始新命令。这允许您使用模式，其中您的代码不断进行传感，计算要跟随的新轨迹，并将该路径发布到 AirSim 中的车辆。每条新发布的轨迹都会取消先前的轨迹，使您的代码能够随着新传感器数据的到来不断更新。

所有 *Async* 方法在 Python 中返回 `concurrent.futures.Future` （在 C++ 中为 `std::future`）。请注意，这些 future 类目前不允许检查状态或取消任务；它们仅允许等待任务完成。AirSim 确实提供 API `cancelLastTask`，但。

#### drivetrain
您可以在两种模式中操作车辆：`drivetrain` 参数设置为 `airsim.DrivetrainType.ForwardOnly` 或 `airsim.DrivetrainType.MaxDegreeOfFreedom`。当您指定 ForwardOnly 时，您表示车辆的前部应始终指向移动车辆的方向。所以如果您希望无人机左转，它首先会旋转以使前部指向左侧。此模式在您只有前置摄像头并且使用 FPV 视图操作车辆时非常有用。这或多或少像在汽车中旅行，您总是有前视图。MaxDegreeOfFreedom 意味着您不在乎前方指向何处。因此，当您向左转时，只需开始向左移动，就像蟹一样。四旋翼可以在任何方向上移动，而无需考虑前方的位置。MaxDegreeOfFreedom 启用此模式。

#### yaw_mode
`yaw_mode` 是一个结构 `YawMode`，具有两个字段，`yaw_or_rate` 和 `is_rate`。如果 `is_rate` 字段为 True，则 `yaw_or_rate` 字段被解释为以度/秒为单位的角速度，这意味着您希望车辆在移动时以该角速度持续旋转其轴。如果 `is_rate` 为 False，则 `yaw_or_rate` 被解释为以度为单位的角度，这意味着您希望车辆旋转到特定角度（即偏航）并在移动时保持该角度。

您可能会发现当 `yaw_mode.is_rate == true` 时，`drivetrain` 参数不应该设置为 `ForwardOnly`，因为您在相互矛盾，既然您说保持前方指向前方，同时又想持续旋转。然而如果您在 `ForwardOnly` 模式下有 `yaw_mode.is_rate = false`，那么您可以做一些奇怪的事情。例如，您可以让无人机做圆圈，且 `yaw_or_rate` 设置为 90 以便摄像头始终指向中心（“超酷的自拍模式”）。在 `MaxDegreeofFreedom` 中，您也可以通过将 `yaw_mode.is_rate = true` 和 `yaw_mode.yaw_or_rate = 20` 设置为一些奇怪的事情。这将导致无人机在其路径上移动，同时旋转，这可能允许执行 360 度扫描。

在大多数情况下，您只希望偏航不发生变化，您可以通过将偏航速度设置为 0 来实现。对此的简写为 `airsim.YawMode.Zero()` (或在 C++ 中： `YawMode::Zero()` )。

#### lookahead 和 adaptive_lookahead
当您要求车辆跟随路径时，AirSim 使用“胡萝卜跟随”算法。此算法通过在路径上向前查看来运行并调整其速度向量。此算法的参数由 `lookahead` 和 `adaptive_lookahead` 指定。在大多数情况下，您希望算法自动决定值，方法是简单地将 `lookahead = -1` 和 `adaptive_lookahead = 0`。

## 在真实车辆上使用 API
我们希望能够在模拟与真实车辆上运行 *相同的代码*。这使您能够在模拟器中测试代码并部署到真实车辆上。

一般来说，因此 API 不应该允许您做一些在真实车辆上无法完成的事情（例如，获取地面真实数据）。但当然，模拟器具有更多信息，这在可能不关心在真实车辆上运行的应用中会很有用。因此，我们明确区分仅限模拟的 API，通过添加 `sim` 前缀，例如 `simGetGroundTruthKinematics`。这样，如果您关心在真实车辆上运行代码，就可以避免使用这些仅限模拟的 API。

AirLib 是一个自包含库，您可以将其放置在像 Gigabyte Barebone Mini PC 这样的离线计算模块上。这个模块可以使用相同的代码与 PX4 等飞行控制器进行通信，飞行控制协议保持不变。您在模拟器中测试的代码保持不变。请参见 [AirLib 在自定义无人机上的应用](custom_drone.md)。

## 向 AirSim 添加新 API

请参见 [添加新 API](adding_new_apis.md) 页面。

## 参考和示例

* [C++ API 示例](apis_cpp.md)
* [汽车示例](https://github.com/Microsoft/AirSim/tree/main/PythonClient//car)
* [多旋翼示例](https://github.com/Microsoft/AirSim/tree/main/PythonClient//multirotor)
* [计算机视觉示例](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision)
* [沿路径移动](https://github.com/Microsoft/AirSim/wiki/moveOnPath-demo) 演示视频，展示在 Modular Neighborhood 环境中快速飞行的多旋翼
* [构建六旋翼](https://github.com/Microsoft/AirSim/wiki/hexacopter)
* [构建点云](https://github.com/Microsoft/AirSim/wiki/Point-Clouds)

## 常见问题

#### 当我运行 API 时，Unreal 突然变得很慢
如果您看到在 Unreal Engine 窗口失去焦点时，Unreal 的运行速度显著变慢，请转到 Unreal 编辑器的“Edit->Editor Preferences”，在“Search”框中输入“CPU”，确保“Use Less CPU when in Background”未选中。

#### 在 Windows 上我还需要其他东西吗？
您应该安装 VS2019 和 VC++，Windows SDK 10.0 和 Python。要使用 Python API，您需要 Python 3.5 或更高版本（使用 Anaconda 安装）。

#### 我应该使用哪个版本的 Python？
我们建议使用 [Anaconda](https://www.anaconda.com/download/) 来获取 Python 工具和库。我们的代码在 Python 3.5.3 :: Anaconda 4.4.0 中进行了测试。这一点重要，因为较旧版本已知存在 [问题](https://stackoverflow.com/a/45934992/207661)。

#### 我在 `import cv2` 时遇到错误
您可以使用以下命令安装 OpenCV：
```
conda install opencv
pip install opencv-python
```

#### TypeError: unsupported operand type(s) for *: 'AsyncIOLoop' and 'float'

这个错误在您安装 Jupyter 时发生，某种原因使 msgpackrpc 库出现问题。创建一个新的 python 环境，尽量使用最小所需的软件包。