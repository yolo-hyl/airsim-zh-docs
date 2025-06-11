# LockStep

最新版本的 PX4 支持在通过 TCP 与模拟器通信时的新 [lockstep 特性](https://docs.px4.io/master/en/simulation/#lockstep-simulation)。Lockstep 是一个重要的特性，因为它同步了 PX4 和模拟器，使得它们本质上使用相同的时钟时间。这使得 PX4 在模拟器性能出现异常长延迟时仍能正常运行。

建议你在以 SITL 模式运行支持 lockstep 的 PX4 版本时，告诉 AirSim 使用 `SteppableClock`，并将 `UseTcp` 和 `LockStep` 均设置为 `true`。

```
    {
        "SettingsVersion": 1.2,
        "SimMode": "Multirotor",
        "ClockType": "SteppableClock",
        "Vehicles": {
            "PX4": {
                "VehicleType": "PX4Multirotor",
                "UseTcp": true,
                "LockStep": true,
                ...
```

这使得 AirSim 不使用“实时”时钟，而是随着每个发送到 PX4 的传感器更新逐步推进时钟。这样，PX4 认为时间在平稳前进，无论 AirSim 实际处理该更新循环花费多长时间。

这有以下优点：

- AirSim 可以在处理更新较慢的低配机器上使用。
- 你可以调试 AirSim 并触发断点，当你恢复时 PX4 将正常运行。
- 你可以启用一些非常慢的传感器，如大量模拟点的 Lidar，而 PX4 仍能正常运行。

`lockstep` 会有一些副作用，即在性能不足的机器上运行 AirSim 或使用高耗费的传感器（如 Lidar）会导致模拟飞行中出现一些明显的抖动，尤其是实时在屏幕上查看更新时。

# 禁用 LockStep

如果你在 cygwin 环境中运行 PX4，有一个关于 lockstep 的 [已知问题](https://github.com/microsoft/AirSim/issues/3415)。PX4 默认配置为使用 lockstep。要禁用此功能，首先 [在 PX4 中禁用它](https://docs.px4.io/master/en/simulation/#disable-lockstep-simulation)：

1. 在你的本地 PX4 仓库中导航到 `boards/px4/sitl/`
2. 编辑 `default.cmake` 并找到以下行：
    ```
    set(ENABLE_LOCKSTEP_SCHEDULER yes)
    ```
3. 将该行更改为：
    ```
    set(ENABLE_LOCKSTEP_SCHEDULER no)
    ```
4. 通过将 `LockStep` 设置为 `false` 并删除任何 `"ClockType": "SteppableClock"` 设置或将 `ClockType` 重置为默认值来在 AirSim 中禁用它：
    ```
        {
            ...
            "ClockType": "",
            "Vehicles": {
                "PX4": {
                    "VehicleType": "PX4Multirotor",
                    "LockStep": false,
                    ...
    ```
5. 现在你可以像往常一样运行 PX4 SITL（`make px4_sitl_default none_iris`），它将使用主机系统时间而无需等待 AirSim。