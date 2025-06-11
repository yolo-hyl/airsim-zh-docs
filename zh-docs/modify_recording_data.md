# 修改录制数据

AirSim 具有一个 [录制功能](settings.md#recording)，可以轻松收集数据和图像。 [录制 API](apis.md#recording-apis) 也允许通过 API 启动和停止录制。

然而，默认录制的数据可能不足以满足你的使用案例，因此最好额外录制如 IMU、GPS 传感器、无人机转速等数据。你可以使用现有的 Python 和 C++ API 获取信息并按照需要存储，特别是 Lidar。通过修改 AirSim 内的录制方法，添加小字段（如 GPS）或内部数据（如 Unreal 位置或其他信息）也是可行的。此页面描述了需要更改的具体方法。

录制的数据以制表符分隔的格式写入 `airsim_rec.txt` 文件，图像则存放在 `images/` 文件夹中。默认情况下，整个文件夹位于 `Documents` 文件夹中（或在设置中指定），并以 `%Y-%M-%D-%H-%M-%S` 格式标记录制开始的时间戳。

车辆记录的以下字段 -

```text
VehicleName TimeStamp   POS_X   POS_Y   POS_Z   Q_W Q_X Q_Y Q_Z Throttle    Steering    Brake   Gear    Handbrake   RPM Speed   ImageFile
```

对于多旋翼 -

```text
VehicleName TimeStamp   POS_X   POS_Y   POS_Z   Q_W Q_X Q_Y Q_Z ImageFile
```

## 代码更改

注意，这需要从源代码构建并使用 AirSim。如果需要，您可以在修改后自己编译一个二进制文件。

填充待存储数据的主要方法是 [`PawnSimApi::getRecordFileLine`](https://github.com/microsoft/AirSim/blob/880c5541fd4824ee2cd9bb82ca5f611eb1ab236a/Unreal/Plugins/AirSim/Source/PawnSimApi.cpp#L544)，这是所有车辆的基础方法，而 Car 重写了它以记录额外的数据，如 [`CarPawnSimApi::getRecordFileLine`](https://github.com/microsoft/AirSim/blob/880c5541fd4824ee2cd9bb82ca5f611eb1ab236a/Unreal/Plugins/AirSim/Source/Vehicles/Car/CarPawnSimApi.cpp#L34) 所示。

要为多旋翼录制额外数据，你可以在 [MultirotorPawnSimApi.cpp/h](https://github.com/microsoft/AirSim/tree/main/Unreal/Plugins/AirSim/Source/Vehicles/Multirotor) 文件中添加类似的方法，重写父类实现并附加其他数据。当前记录的数据也可以根据需要进行修改和删除。

例如，同时录制 GPS、IMU 和气压计数据的多旋翼 -

```cpp
// MultirotorPawnSimApi.cpp
std::string MultirotorPawnSimApi::getRecordFileLine(bool is_header_line) const
{
    std::string common_line = PawnSimApi::getRecordFileLine(is_header_line);
    if (is_header_line) {
        return common_line +
               "Latitude\tLongitude\tAltitude\tPressure\tAccX\tAccY\tAccZ\t";
    }

    const auto& state = vehicle_api_->getMultirotorState();
    const auto& bar_data = vehicle_api_->getBarometerData("");
    const auto& imu_data = vehicle_api_->getImuData("");

    std::ostringstream ss;
    ss << common_line;
    ss << state.gps_location.latitude << "\t" << state.gps_location.longitude << "\t"
       << state.gps_location.altitude << "\t";

    ss << bar_data.pressure << "\t";

    ss << imu_data.linear_acceleration.x() << "\t" << imu_data.linear_acceleration.y() << "\t"
       << imu_data.linear_acceleration.z() << "\t";

    return ss.str();
}
```

```cpp
// MultirotorPawnSimApi.h
virtual std::string getRecordFileLine(bool is_header_line) const override;
```