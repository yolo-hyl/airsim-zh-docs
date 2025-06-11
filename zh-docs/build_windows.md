# 在 Windows 上构建 AirSim

## 安装 Unreal Engine

1. [下载](https://www.unrealengine.com/download) Epic Games Launcher。虽然 Unreal Engine 是开源且免费下载的，但仍需注册。
2. 运行 Epic Games Launcher，在左侧面板中打开 `Unreal Engine` 标签页。
点击右上角的 `Install` 按钮，应该会显示下载 **Unreal Engine >= 4.27** 的选项。根据需要选择安装位置，如下图所示。如果你安装了多个版本的 Unreal，请**确保你使用的版本被设置为 `current`**，方法是点击该版本的 Launch 按钮旁边的下拉箭头。

   **Note**: 如果你有 UE 4.16 或更早版本的项目，请查看 [升级指南](unreal_upgrade.md) 来升级你的项目。

![Unreal Engine Tab UI Screenshot](images/ue_install.png)

![Unreal Engine Install Location UI Screenshot](images/ue_install_location.png)

## 构建 AirSim
* 安装 Visual Studio 2022。
**确保**选择了 **使用 C++ 的桌面开发（Desktop Development with C++）** 和 **Windows 10 SDK 10.0.19041**（默认应该已经选择），并在安装 VS 2022 时，在“Individual Components”标签下选择最新的 .NET Framework SDK。
* 启动 `Developer Command Prompt for VS 2022`。
* 克隆代码仓库：`git clone https://github.com/Microsoft/AirSim.git`，然后通过 `cd AirSim` 进入 AirSim 目录。

    **Note:** 通常不建议将 AirSim 安装在 C 盘。这可能会导致脚本失败，并要求以管理员模式运行 VS。建议克隆到 D 或 E 盘等其他磁盘。

* 在命令行中运行 `build.cmd`。这将在 `Unreal\Plugins` 文件夹中创建可直接使用的插件文件，可将其复制到任意 Unreal 项目中。

## 构建 Unreal 项目

最后，你需要一个 Unreal 项目来承载你的载具环境。如果你尚未这样做，请务必在构建第一个环境前关闭并重新打开 Unreal Engine 和 Epic Games Launcher。重新启动 Epic Games Launcher 后，它会要求你将项目文件扩展名与 Unreal Engine 关联，点击“fix now”进行修复。AirSim 自带一个内置的 “Blocks Environment” 可供使用，当然你也可以创建自己的环境。请参考 [设置 Unreal 环境](unreal_proj.md)。

## 设置遥控器（仅限 Multirotor）

如果你想手动飞行，需要配置遥控器。详见 [遥控器设置](remote_control.md)。

你也可以使用 [APIs](apis.md) 进行编程控制，或使用所谓的 [计算机视觉模式](image_apis.md) 通过键盘控制移动。

## 如何使用 AirSim

按照以上步骤完成设置后，你可以：

1. 双击 .sln 文件以加载位于 `Unreal\Environments\Blocks` 的 Blocks 项目（或你自定义 [custom](unreal_custenv.md) Unreal 项目的 .sln 文件）。如果你未看到 .sln 文件，说明你可能还未完成上面“构建 Unreal 项目”部分的步骤。

    **Note**: Unreal 4.27 会自动生成针对 Visual Studio 2019 的 .sln 文件。Visual Studio 2022 仍然可以加载和运行该 .sln，但如果你希望完全支持 Visual Studio 2022，需要在 'Edit->Editor Preferences->Source Code' 中将 'Source Code Editor' 设置为 'Visual Studio 2022'。

2. 将你的 Unreal 项目设置为启动项目（例如 Blocks 项目），并确保 Build 配置设置为 "Develop Editor" 和 x64。
3. Unreal Editor 加载完成后，点击 Play 按钮。

!!! tip
    打开 'Edit->Editor Preferences'，在搜索框中输入 'CPU'，确保未勾选 'Use Less CPU when in Background'。

请查看 [使用 APIs](apis.md) 和 [settings.json](settings.md) 来了解各种可用选项。

# Unity 上的 AirSim（实验性）
[Unity](https://unity3d.com/) 是另一个优秀的游戏引擎平台，我们提供了 [AirSim 与 Unity 的实验性集成](Unity.md)。请注意，该功能仍在开发中，部分功能可能尚未完善。
