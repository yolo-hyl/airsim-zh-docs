# 下载二进制文件

您可以简单地下载预编译的二进制文件并运行，以立即开始使用。如果您想要设置自己的 Unreal 环境，请查看 [这些说明](https://github.com/Microsoft/AirSim/#how-to-get-it)。

### Unreal 引擎

**Windows, Linux**：从 [最新版本](https://github.com/Microsoft/AirSim/releases) 下载您选择的环境的二进制文件。

一些预编译的环境二进制文件可能包含多个文件（即 City.zip.001、City.zip.002）。确保在启动环境之前下载所有文件。
使用 [7zip](https://www.7-zip.org/download.html) 解压这些文件。在 Linux 上，将第一个 zip 文件名作为参数传递，它应该能够检测到所有其他部分 - `7zz x TrapCamera.zip.001`

**macOS**：您需要 [自己构建](build_linux.md)

### Unity（实验性）

一个名为 Windridge City 的免费环境可在 [Unity Asset Store](https://assetstore.unity.com/) 作为 AirSim 在 Unity 上的实验性发布。**注意**：这是一个旧版本，许多功能和 API 可能无法使用。

## 控制车辆

我们的大多数用户通常使用 [API](apis.md) 控制车辆。然而，您也可以手动控制车辆。您可以使用键盘、游戏手柄或 [方向盘](steering_wheel_installation.md) 驾驶汽车。要手动飞行无人机，您需要使用 XBox 控制器或遥控器（欢迎随时 [贡献](CONTRIBUTING.md) 键盘支持）。有关更多详细信息，请参见 [遥控器设置](remote_control.md)。或者，您可以使用 [API](apis.md) 进行编程控制，或者使用所谓的 [计算机视觉模式](image_apis.md) 使用键盘在环境中移动。

## 没有好的 GPU？

AirSim 二进制文件，如 CityEnviron，需要一块强劲的 GPU 以顺利运行。您可以通过编辑 Windows 上的 `run.bat` 文件（如果不存在，则使用以下内容创建它）以低分辨率模式运行它：

```batch
start CityEnviron -ResX=640 -ResY=480 -windowed
```

对于 Linux 二进制文件，可以使用 `Blocks.sh` 或相应的 shell 脚本，如下所示：

```shell
./Blocks.sh -ResX=640 -ResY=480 -windowed
```

查看所有其他 [命令行选项](https://docs.unrealengine.com/en-US/ProductionPipelines/CommandLineArguments/index.html)

UE 4.24 默认使用 Vulkan 驱动程序，但它们可能会消耗更多 GPU 内存。如果出现内存分配错误，您可以尝试通过使用 `-opengl` 切换到 OpenGL。

您还可以使用 `simRunConsoleCommand()` API 限制最大 FPS，方法如下：

```python
>>> import airsim
>>> client = airsim.VehicleClient()
>>> client.confirmConnection()
Connected!
Client Ver:1 (Min Req: 1), Server Ver:1 (Min Req: 1)

>>> client.simRunConsoleCommand("t.MaxFPS 10")
True
```