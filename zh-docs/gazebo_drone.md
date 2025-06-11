# 欢迎使用 GazeboDrone

GazeboDrone 允许将 Gazebo 无人机连接至 AirSim 无人机，利用 Gazebo 无人机作为飞行动力学模型 (FDM)，而 AirSim 用于生成环境传感器数据。它可用于 **多旋翼**、**固定翼** 或其他任何车辆。

## 依赖项

### Gazebo

请确保已安装 Gazebo 的依赖项：

```
sudo apt-get install libgazebo9-dev
```

### AirLib

该项目使用 GCC 8 进行构建，因此 AirLib 也需要使用 GCC 8 进行构建。  
从你的 AirSim 根目录运行：  
```
./clean.sh
./setup.sh
./build.sh --gcc
```

## AirSim 模拟器

AirSim UE 插件需要使用 clang 进行构建，因此你不能使用上一步中编译的版本。你可以使用 [我们的二进制文件](https://github.com/microsoft/AirSim/releases)，或者可以在另一个文件夹中再次克隆 AirSim 并不带上述选项进行构建，然后你可以 [运行 Blocks](build_linux.md#how-to-use-airsim) 或你的自定义环境。

### AirSim 设置

在 `settings.json` 文件中，你需要添加这一行：  
`"PhysicsEngineName":"ExternalPhysicsEngine"`。  
你可能想要更改 AirSim 无人机的视觉模型，对于此你可以参考 [这个教程](https://youtu.be/Bp86WiLUC80)。

## 构建

从你的 AirSim 根目录执行以下命令：  
```
cd GazeboDrone
mkdir build && cd build
cmake -DCMAKE_C_COMPILER=gcc-8 -DCMAKE_CXX_COMPILER=g++-8 ..
make
```

## 运行

首先运行 AirSim 模拟器和你的 Gazebo 模型，然后从你的 AirSim 根目录执行以下命令：

```
cd GazeboDrone/build
./GazeboDrone
```