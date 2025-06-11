# 常见问题解答 (FAQ)

---

## Windows 构建

* [如何强制 Unreal 使用 Visual Studio 2019?](#如何强制-unreal-使用-visual-studio-2019)
* [我收到错误：'where' 未被识别为内部或外部命令](#我收到错误where-未被识别为内部或外部命令)
* [我收到错误 `<MyProject> 无法编译。请尝试从源代码手动重建`](#我收到错误-myproject-无法编译请尝试从源代码手动重建)
* [我在运行 build.cmd 时收到 `error C100 : 编译器发生内部错误`](#我在运行-buildcmd-时收到-error-c100--编译器发生内部错误)
* [我收到错误 "'corecrt.h': 没有这样的文件或目录" 或 "Windows SDK 版本 8.1 未找到"](##我收到错误-corecrth-没有这样的文件或目录-或-windows-sdk-版本-81-未找到)
* [如何在 AirSim 中使用 PX4 固件?](#如何在-airsim-中使用-px4-固件)
* [我在 Visual Studio 中做了更改，但没有效果](#我在-visual-studio-中做了更改但没有效果)
* [Unreal 仍在使用 VS2015 或我收到一些链接错误](#unreal-仍在使用-vs2015-或我收到一些链接错误)

---

## Linux 构建

* [我收到错误 `<MyProject> 无法编译。请尝试从源代码手动重建`](#我收到错误-myproject-无法编译请尝试从源代码手动重建)
* [Unreal 崩溃了！我如何知道出了什么问题？](#unreal-崩溃了我如何知道出了什么问题)
* [我如何在 Linux 上使用 IDE?](#我如何在-linux-上使用-ide)
* [我可以在 Windows 机器上交叉编译 Linux 吗?](#我可以在-windows-机器上交叉编译-linux-吗)
* [AirSim 使用哪个编译器和标准库?](#airsim-使用哪个编译器和标准库)
* [AirSim 构建使用哪个版本的 CMake?](#airsim-构建使用哪个版本的-cmake)
* [我可以在 BashOnWindows 中编译 AirSim 吗?](#我可以在-bashonwindows-中编译-airsim-吗)
* [我在哪里可以找到更多关于在 Linux 上运行 Unreal 的信息?](#我在哪里可以找到更多关于在-linux-上运行-unreal-的信息)

---

## 其他

* [打包包含 AirSim 插件的二进制文件](#打包包含-airsim-插件的二进制文件)

---

<!-- ======================================================================= -->
## Windows 构建
<!-- ======================================================================= -->

###### 如何强制 Unreal 使用 Visual Studio 2019?

>如果默认的 `update_from_git.bat` 文件生成 VS 2017 项目，则可能需要手动运行 `C:\Program Files\Epic Games\UE_4.25\Engine\Binaries\DotNET\UnrealBuildTool.exe` 工具，使用命令行选项 `-projectfiles -project=<your.uproject>  -game -rocket -progress -2019`。
>
>如果您是从 4.18 升级到 4.25，您可能还需要在 `*.Target.cs` 和 `*Editor.Target.cs` 构建文件中添加 `BuildSettingsVersion.V2`，像这样：
>
>```c#
>	public AirSimNHTestTarget(TargetInfo Target) : base(Target)
>	{
>		Type = TargetType.Game;
>		DefaultBuildSettings = BuildSettingsVersion.V2;
>		ExtraModuleNames.AddRange(new string[] { "AirSimNHTest" });
>	}
>```
>
>您还可能需要编辑以下文件：
>
>```
>"%APPDATA%\Unreal Engine\UnrealBuildTool\BuildConfiguration.xml
>```
>
>并添加此编译器版本设置：
>
>```xml
><Configuration xmlns="https://www.unrealengine.com/BuildConfiguration">
>  <WindowsPlatform>
>    <Compiler>VisualStudio2019</Compiler>
>  </WindowsPlatform>
></Configuration>
>```

<!-- ======================================================================= -->

###### 我收到错误：'where' 未被识别为内部或外部命令

>您必须将 `C:\WINDOWS\SYSTEM32` 添加到您的 PATH 环境变量中。

<!-- ======================================================================= -->

###### 我收到错误 `<MyProject> 无法编译。请尝试从源代码手动重建`

>当出现编译错误时，将发生这种情况。日志保存在 `<My-Project>\Saved\Logs` 中，可以用来找出问题。
>
>一个常见的问题可能是 Visual Studio 版本冲突，AirSim 使用 VS 2019，而 UE 使用 VS 2017，这可以通过在日志文件中搜索 `2017` 来发现。在这种情况下，请参见上面的答案。
>
>如果您修改了 AirSim 插件文件，您可以右键单击 `.uproject` 文件，选择 `生成 Visual Studio 解决方案文件`，然后在 VS 中打开 `.sln` 文件以修复错误并重新构建。

<!-- ======================================================================= -->

###### 我在运行 build.cmd 时收到 `error C100 : 编译器发生内部错误`

>我们注意到 VS 版本 `15.9.0` 会导致此问题，并在 AirSim 代码中检查到了一种解决方法。请确保您拉取最新的 AirSim 代码。

<!-- ======================================================================= -->

###### 我收到错误 "'corecrt.h': 没有这样的文件或目录" 或 "Windows SDK 版本 8.1 未找到"

>您很可能没有安装与 Visual Studio 一起的 [Windows SDK](https://developercommunity.visualstudio.com/content/problem/3754/cant-compile-c-program-because-of-sdk-81cant-add-a.html)。

<!-- ======================================================================= -->

###### 如何在 AirSim 中使用 PX4 固件?

>根据默认设置，AirSim 使用其内置的固件，称为 [simple_flight](simple_flight.md)。如果您只想使用它，则无需额外设置。如果您想切换到使用 PX4，请参见 [本指南](px4_setup.md)。

<!-- ======================================================================= -->

###### 我在 Visual Studio 中做了更改，但没有效果

>有时，Unreal + VS 构建系统不会在您只修改头文件时重新编译。要确保重新编译，可以使某个基于 Unreal 的 cpp 文件“变脏”，例如 AirSimGameMode.cpp。

<!-- ======================================================================= -->

###### Unreal 仍在使用 VS2015 或我收到一些链接错误

>运行多个版本的 VS 可能会导致在编译 UE 项目时出现问题。一个可能出现的问题是，UE 会尝试使用较旧版本的 VS 进行编译，这可能会工作也可能不会。Unreal 中有两个设置，一个用于引擎，一个用于项目，用于调整使用的 VS 版本。
>
>1. 编辑 -> 编辑器首选项 -> 常规 -> 源代码 -> 源代码编辑器
>2. 编辑 -> 项目设置 -> 平台 -> Windows -> 工具链 -> 编译器版本
>
>在某些情况下，这些设置仍然不会导致所需结果，可能会产生如下错误：LINK : fatal error LNK1181: 无法打开输入文件 'ws2_32.lib'。
>
>解决此类问题可以应用以下程序：
>
>1. 使用 [VisualStudioUninstaller](https://github.com/Microsoft/VisualStudioUninstaller/releases) 卸载所有旧版本的 VS
>2. 修复/安装 VS 2019
>3. 重新启动计算机并安装 Epic 启动器及所需版本的引擎

---

## Linux 构建
<!-- ======================================================================= -->

###### 我收到错误 `<MyProject> 无法编译。请尝试从源代码手动重建`。

>这可能由于编译错误或 gch 文件过时。请查看控制台窗口。您是否看到类似以下内容？
>
>`fatal error: file '/usr/include/linux/version.h''/usr/include/linux/version.h' 自上次预编译头后已更改`
>
>如果是这种情况，请查找在该消息后跟随的 *.gch 文件，并将它们删除，然后重试。以下是 [相关讨论](https://answers.unrealengine.com/questions/412349/linux-ue4-build-precompiled-header-fatal-error.html) 在 Unreal Engine 论坛上的链接。
>
>如果您在控制台中看到其他编译错误，请打开这些源文件，查看是否是由于您所做的更改。如果不是，则在 GitHub 上报告它作为问题。

<!-- ======================================================================= -->

###### Unreal 崩溃了！我如何知道出了什么问题？

>请进入 `MyUnrealProject/Saved/Crashes` 文件夹，并在其子目录中搜索文件 `MyProject.log`。在该文件的末尾，您将看到堆栈跟踪和消息。
>您还可以查看 `Diagnostics.txt` 文件。

<!-- ======================================================================= -->

###### 我如何在 Linux 上使用 IDE?

>您可以使用 Qt Creator 或 CodeLite。有关 Qt Creator 的说明可在 [此处](https://docs.unrealengine.com/en-US/SharingAndReleasing/Linux/BeginnerLinuxDeveloper/SettingUpQtCreator/index.html) 找到。

<!-- ======================================================================= -->

###### 我可以在 Windows 机器上交叉编译 Linux 吗?

>是的，您可以，但我们尚未进行测试。您可以在 [此处](https://docs.unrealengine.com/latest/INT/Platforms/Linux/GettingStarted/index.html) 找到说明。

<!-- ======================================================================= -->

###### AirSim 使用哪个编译器和标准库?

>我们使用 Unreal Engine 使用的相同编译器，**Clang 8**，以及标准库，**libc++**。AirSim 的 `setup.sh` 将自动下载它们。

<!-- ======================================================================= -->

###### AirSim 构建使用哪个版本的 CMake?

>3.10.0 或更高版本。这在 Ubuntu 16.04 中 *不* 是默认值，因此 `setup.sh` 为您安装它。您可以使用 `cmake --version` 检查您的 CMake 版本。如果您有较旧的版本，请按照 [这些说明](cmake_linux.md) 或查看 [CMake 网站](https://cmake.org/install/)。

<!-- ======================================================================= -->

###### 我可以在 BashOnWindows 中编译 AirSim 吗?

>是的，但您无法从 BashOnWindows 运行 Unreal。因此，这对于检查 Linux 编译很有用，但不适合端到端执行。
>请参阅 [BashOnWindows 安装指南](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide)。
>确保拥有最新版本（Windows 10 Creators Edition），因为以前版本存在各种问题。
>另外，不要从 `Visual Studio Command Prompt` 调用 `bash`，否则 CMake 可能会找到 VC++ 并尝试使用它！

<!-- ======================================================================= -->

###### 我在哪里可以找到更多关于在 Linux 上运行 Unreal 的信息？
>从这里开始： [Unreal on Linux](https://docs.unrealengine.com/latest/INT/Platforms/Linux/index.html)
>[Building Unreal on Linux](https://wiki.unrealengine.com/Building_On_Linux#Clang)
>[Unreal Linux Support](https://wiki.unrealengine.com/Linux_Support)
>[Unreal Cross Compilation](https://wiki.unrealengine.com/Compiling_For_Linux)

---

## 其他
<!-- ======================================================================= -->

###### 打包包含 AirSim 插件的二进制文件

>为了打包具有 AirSim 插件的自定义环境，需要确保满足一些项目设置，以确保所有 AirSim 所需的资产都包含在包内。在 `Edit -> Project Settings... -> Project -> Packaging` 下，请确保正确配置以下设置：
>
>- `要包含在打包构建中的地图列表`：确保存在一个条目 `/AirSim/AirSimAssets`
>- `额外的资产目录进行烹饪`：确保存在一个条目 `/AirSim/HUDAssets`