# 如何在 AirSim 中使用 Lidar

AirSim 支持多旋翼和汽车的 Lidar。

Lidar 的启用和其他 Lidar 设置可以通过 AirSimSettings json 配置。有关一般/共享传感器设置的配置，请参见 [general sensors](sensors.md)。

## 在车辆上启用 Lidar
* 默认情况下，Lidar 是未启用的。要启用 Lidar，请在设置 json 中设置 SensorType 和 Enabled 属性。

```json
    "Lidar1": {
         "SensorType": 6,
         "Enabled" : true,
    }
```

* 可以在一辆车辆上启用多个 Lidar。

## Lidar 配置
目前，可以通过设置 json 配置以下参数。

参数                      | 描述
--------------------------| ------------
NumberOfChannels          | Lidar 的通道数/激光器数量
Range                     | 范围，以米为单位
PointsPerSecond           | 每秒捕获的点数
RotationsPerSecond        | 每秒旋转次数
HorizontalFOVStart        | Lidar 的水平视场起始角度，以度为单位
HorizontalFOVEnd          | Lidar 的水平视场结束角度，以度为单位
VerticalFOVUpper          | Lidar 的垂直视场上限，以度为单位
VerticalFOVLower          | Lidar 的垂直视场下限，以度为单位
X Y Z                     | Lidar 相对于车辆的位置（以米为单位，按 NED 坐标系）
Roll Pitch Yaw            | Lidar 相对于车辆的朝向（以度为单位，按航向-俯仰-滚转顺序，前向向量为 +X）
DataFrame                 | 输出点的框架（“VehicleInertialFrame”或“SensorLocalFrame”）
ExternalController        | 是否将数据发送到外部控制器，例如 ArduPilot 或 PX4 （如果使用中，默认是 `true`）（PX4 目前不发送 Lidar 数据）

例如：

```json
{
    "SeeDocsAt": "https://microsoft.github.io/AirSim/settings/",
    "SettingsVersion": 1.2,

    "SimMode": "Multirotor",

     "Vehicles": {
		"Drone1": {
			"VehicleType": "simpleflight",
			"AutoCreate": true,
			"Sensors": {
			    "LidarSensor1": {
					"SensorType": 6,
					"Enabled" : true,
					"NumberOfChannels": 16,
					"RotationsPerSecond": 10,
					"PointsPerSecond": 100000,
					"X": 0, "Y": 0, "Z": -1,
					"Roll": 0, "Pitch": 0, "Yaw" : 0,
					"VerticalFOVUpper": -15,
					"VerticalFOVLower": -25,
					"HorizontalFOVStart": -20,
					"HorizontalFOVEnd": 20,
					"DrawDebugPoints": true,
					"DataFrame": "SensorLocalFrame"
				},
				"LidarSensor2": {
				   "SensorType": 6,
					"Enabled" : true,
					"NumberOfChannels": 4,
					"RotationsPerSecond": 10,
					"PointsPerSecond": 10000,
					"X": 0, "Y": 0, "Z": -1,
					"Roll": 0, "Pitch": 0, "Yaw" : 0,
					"VerticalFOVUpper": -15,
					"VerticalFOVLower": -25,
					"DrawDebugPoints": true,
					"DataFrame": "SensorLocalFrame"
				}
			}
		}
    }
}
```

## 服务器端调试的可视化

默认情况下，Lidar 点不会在视口中绘制。要启用在视口中绘制命中激光点，请通过设置 json 启用 `DrawDebugPoints`。

```json
    "Lidar1": {
         ...
         "DrawDebugPoints": true
    },
```

**注意：** 启用 `DrawDebugPoints` 可能会导致过高的内存使用，并在版本 `v1.3.1`、`v1.3.0` 中崩溃。该问题已在主分支中修复，并应在后续版本中正常工作。

## 客户端 API

使用 `getLidarData()` API 检索 Lidar 数据。

* 该 API 返回一个点云，包含一个平坦的浮点数组，以及捕获的时间戳和 Lidar 姿态。
* 点云：
    * 浮点数表示在最后一次扫描范围内每个命中点的 [x,y,z] 坐标。
    * 输出点的框架可以使用“DataFrame”属性进行配置 -
        * "" 或 `VehicleInertialFrame` -- 默认；返回的点在车辆惯性框架中（以米为单位，按 NED 坐标系）
        * `SensorLocalFrame` -- 返回的点在 Lidar 本地框架中（以米为单位，按 NED 坐标系）
* Lidar 姿态：
    * Lidar 在车辆惯性框架中的姿态（以米为单位，按 NED 坐标系）
    * 可用于将点转换为其他框架。
* 分割：每个 Lidar 点的碰撞物体的分割信息。

### Python 示例
- [drone_lidar.py](https://github.com/microsoft/AirSim/blob/main/PythonClient/multirotor/drone_lidar.py)
- [car_lidar.py](https://github.com/microsoft/AirSim/blob/main/PythonClient/car/car_lidar.py)
- [sensorframe_lidar_pointcloud.py](https://github.com/microsoft/AirSim/blob/main/PythonClient/multirotor/sensorframe_lidar_pointcloud.py)
- [vehicleframe_lidar_pointcloud.py](https://github.com/microsoft/AirSim/blob/main/PythonClient/multirotor/vehicleframe_lidar_pointcloud.py)

## 即将推出
* 客户端侧的 Lidar 数据可视化。