## 距离传感器

默认情况下，距离传感器朝向车辆的前方。通过修改设置，可以将其指向任何方向。

可配置参数 -

参数                | 描述
--------------------|------------
X Y Z               | 传感器相对于车辆的位置（以NED坐标系，单位为米）（默认 (0,0,0) - 多旋翼，(0,0,-1) - 汽车）
Yaw Pitch Roll      | 传感器相对于车辆的方向（单位：度）（默认 (0,0,0)）
MinDistance         | 距离传感器测量的最小距离（米，仅用于填充PX4的Mavlink消息）（默认 0.2m）
MaxDistance         | 距离传感器测量的最大距离（米）（默认 40.0m）
ExternalController  | 是否将数据发送到外部控制器，如 ArduPilot 或 PX4（默认 `true`）

例如，为了使传感器朝向地面（进行类似气压计的高度测量），可以如下修改方向 -

```json
"Distance": {
    "SensorType": 5,
    "Enabled" : true,
    "Yaw": 0, "Pitch": -90, "Roll": 0
}
```

**注意：** 对于汽车，传感器默认安置在车辆中心上方1米处。这是必需的，因为否则，传感器会因为位于车辆内部而输出奇怪的数据。这不会影响传感器的数值，例如在测量两辆汽车之间的距离时。有关示例用法，请参见 [`PythonClient/car/distance_sensor_multi.py`](https://github.com/Microsoft/AirSim/blob/main/PythonClient/car/distance_sensor_multi.py)。