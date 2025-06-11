# 在 macOS 上构建 AirSim

目前仅在 **macOS Catalina (10.15)** 上进行了测试。理论上，AirSim 应该也可以在更高版本的 macOS 和 Apple Silicon 硬件上运行，但这些平台暂不提供官方支持。

我们提供两种构建方式 —— 你可以选择在 Docker 容器中构建，也可以在宿主机上构建。

## Docker

请参阅[此处说明](docker_ubuntu.md)

## 宿主机

### 构建前准备

#### 下载 Unreal Engine

1. [下载](https://www.unrealengine.com/download) Epic Games Launcher。尽管 Unreal Engine 是开源且可免费下载的，但仍需注册帐号。
2. 启动 Epic Games Launcher，打开左侧面板中的 `Library` 标签页。
   点击 `Add Versions`，应该会出现下载 **Unreal 4.27** 的选项，如下所示。如果你安装了多个 Unreal 版本，请**确保将 4.27 设置为当前版本**，方法是点击该版本“Launch”按钮旁边的下拉箭头。

   **注意**：AirSim 也支持 UE >= 4.24，但我们推荐使用 4.27。
   **注意**：如果你有 UE 4.16 或更早版本的项目，请参考[升级指南](unreal_upgrade.md)以升级你的项目。

### 构建 AirSim

- 克隆 AirSim 并进行构建：

```bash
# 前往你用于克隆 GitHub 项目的文件夹
git clone https://github.com/Microsoft/AirSim.git
cd AirSim
````

默认情况下，AirSim 使用 clang 8 进行构建，以兼容 UE 4.25。`setup.sh` 脚本将安装合适版本的 cmake、llvm 和 eigen。

在 Apple Silicon 上构建时，需使用 CMake 3.19.2。

```bash
./setup.sh
./build.sh
# 使用 ./build.sh --debug 可在调试模式下构建
```

### 构建 Unreal 环境

最后，你需要一个 Unreal 项目来承载你的车辆环境。AirSim 自带一个内置的 "Blocks Environment" 可供使用，你也可以创建自己的环境。若你希望搭建自定义环境，请参阅[设置 Unreal 环境](unreal_proj.md)。

## 如何使用 AirSim

* 浏览至 `AirSim/Unreal/Environments/Blocks`。
* 在终端中运行 `./GenerateProjectFiles.sh <UE_PATH>`，其中 `<UE_PATH>` 是 Unreal 安装目录的路径。（默认情况下为 `/Users/Shared/Epic\ Games/UE_4.27/`）该脚本会生成名为 `Blocks.xcworkspace` 的 XCode 工作区。
* 打开 XCode 工作区，并点击左上角的构建并运行按钮。
* Unreal Editor 加载完成后，点击 Play 按钮。

参阅 [使用 APIs](apis.md) 和 [settings.json 配置](settings.md) 以了解 AirSim 的多种使用选项。

!!! tip
进入 'Edit -> Editor Preferences'，在搜索框中输入 'CPU'，确保取消勾选 'Use Less CPU when in Background'。

### \[可选] 设置遥控器（仅限多旋翼）

如果你希望手动飞行，则需要使用遥控器。详情请参阅[遥控器设置](remote_control.md)。

或者，你也可以使用 [APIs](apis.md) 进行编程控制，或使用所谓的 [Computer Vision 模式](image_apis.md) 通过键盘移动。
