# 在 Linux 上构建 AirSim

当前推荐并经过测试的环境是 **Ubuntu 18.04 LTS**。理论上你可以在其他发行版上构建，但我们尚未进行测试。

我们提供两种构建方式 —— 你可以选择在 Docker 容器中构建，也可以在宿主机上构建。

## Docker

请参阅[此处说明](docker_ubuntu.md)

## 宿主机

### 构建前准备

#### 构建 Unreal Engine

- 请确保你已[注册 Epic Games 帐号](https://docs.unrealengine.com/en-US/SharingAndReleasing/Linux/BeginnerLinuxDeveloper/SettingUpAnUnrealWorkflow/index.html)。获取 Unreal Engine 源代码需要此注册。

- 在你喜欢的文件夹中克隆 Unreal，并进行构建（这可能需要一段时间！）。**注意**：目前我们仅支持 Unreal >= 4.27。推荐使用 4.27 版本。

```bash
# 前往你用于克隆 GitHub 项目的文件夹
git clone -b 4.27 git@github.com:EpicGames/UnrealEngine.git
cd UnrealEngine
./Setup.sh
./GenerateProjectFiles.sh
make
```

### 构建 AirSim

* 克隆 AirSim 并进行构建：

```bash
# 前往你用于克隆 GitHub 项目的文件夹
git clone https://github.com/Microsoft/AirSim.git
cd AirSim
```

默认情况下，AirSim 使用 clang 8 进行构建，以兼容 UE 4.27。`setup.sh` 脚本将安装正确版本的 cmake、llvm 和 eigen。

```bash
./setup.sh
./build.sh
# 使用 ./build.sh --debug 可在调试模式下构建
```

### 构建 Unreal 环境

最后，你需要一个 Unreal 项目来承载你的车辆环境。AirSim 自带一个内置的 "Blocks Environment" 可供使用，你也可以创建自己的环境。若你希望搭建自定义环境，请参阅[设置 Unreal 环境](unreal_proj.md)。

## 如何使用 AirSim

一旦 AirSim 设置完成：

* 进入 `UnrealEngine` 安装目录，运行 `./Engine/Binaries/Linux/UE4Editor` 启动 Unreal。
* 当 Unreal Engine 提示打开或创建项目时，选择 Browse 并定位到 `AirSim/Unreal/Environments/Blocks`（或你的[自定义](unreal_custenv.md) Unreal 项目）。
* 或者，也可以通过命令行参数传入项目文件。对于 Blocks：`./Engine/Binaries/Linux/UE4Editor <AirSim_path>/Unreal/Environments/Blocks/Blocks.uproject`
* 若出现转换项目的提示，选择 “More Options” 或 “Convert-In-Place”。若提示是否构建，选择 Yes。若提示是否禁用 AirSim 插件，选择 No。
* Unreal Editor 加载完成后，点击 Play 按钮。

参阅 [使用 APIs](apis.md) 和 [settings.json 配置](settings.md) 以了解 AirSim 的多种使用选项。

!!! tip
    进入 'Edit -> Editor Preferences'，在搜索框中输入 'CPU'，确保取消勾选 'Use Less CPU when in Background'。

### \[可选] 设置遥控器（仅限多旋翼）

如果你希望手动飞行，则需要使用遥控器。详情请参阅[遥控器设置](remote_control.md)。

或者，你也可以使用 [APIs](apis.md) 进行编程控制，或使用所谓的 [Computer Vision 模式](image_apis.md) 通过键盘移动。
