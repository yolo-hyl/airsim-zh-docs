# 使用 C++ API 控制 AirSim

如果还没有阅读，请首先查看 [通用 API 文档](apis.md)。本文件描述了 C++ 示例和其他 C++ 相关的细节。

## 快速入门

最快的方法是打开 Visual Studio 2019 中的 AirSim.sln。您将在解决方案中看到 [Hello Car](https://github.com/Microsoft/AirSim/tree/main/HelloCar/) 和 [Hello Drone](https://github.com/Microsoft/AirSim/tree/main/HelloDrone/) 示例。这些示例将向您展示在 VC++ 项目中设置所需的 include 路径和 lib 路径。如果您使用的是 Linux，则需要在 [cmake 文件](https://github.com/Microsoft/AirSim/tree/main/cmake//HelloCar/CMakeLists.txt) 或在编译器命令行中指定这些路径。

#### Include 和 Lib 文件夹

* Include 文件夹: `$(ProjectDir)..\AirLib\deps\rpclib\include;include;$(ProjectDir)..\AirLib\deps\eigen3;$(ProjectDir)..\AirLib\include`
* 依赖项: `rpc.lib`
* Lib 文件夹: `$(ProjectDir)\..\AirLib\deps\MavLinkCom\lib\$(Platform)\$(Configuration);$(ProjectDir)\..\AirLib\deps\rpclib\lib\$(Platform)\$(Configuration);$(ProjectDir)\..\AirLib\lib\$(Platform)\$(Configuration)`
* 引用: 将 AirLib 和 MavLinkCom 引用到项目引用中。（右键点击您的项目，然后转到 `References`，`Add reference...`，然后选择 AirLib 和 MavLinkCom）

## Hello Car

以下是如何使用 C++ API 控制模拟汽车的方法（另见 [Python 示例](apis.md#hello_car)）：

```cpp

// ready to run example: https://github.com/Microsoft/AirSim/blob/main/HelloCar/main.cpp

#include <iostream>
#include "vehicles/car/api/CarRpcLibClient.hpp"

int main()
{
    msr::airlib::CarRpcLibClient client;
    client.enableApiControl(true); //这将禁用手动控制
    CarControllerBase::CarControls controls;

    std::cout << "按下 Enter 向前行驶" << std::endl; std::cin.get();
    controls.throttle = 1;
    client.setCarControls(controls);

    std::cout << "按下 Enter 以激活手刹" << std::endl; std::cin.get();
    controls.handbrake = true;
    client.setCarControls(controls);

    std::cout << "按下 Enter 以转弯并向后行驶" << std::endl; std::cin.get();
    controls.handbrake = false;
    controls.throttle = -1;
    controls.steering = 1;
    client.setCarControls(controls);

    std::cout << "按下 Enter 以停止" << std::endl; std::cin.get();
    client.setCarControls(CarControllerBase::CarControls());

    return 0;
}
```

## Hello Drone

以下是如何使用 C++ API 控制模拟四旋翼的方法（另见 [Python 示例](apis.md#hello_drone)）：

```cpp

// ready to run example: https://github.com/Microsoft/AirSim/blob/main/HelloDrone/main.cpp

#include <iostream>
#include "vehicles/multirotor/api/MultirotorRpcLibClient.hpp"

int main()
{
    msr::airlib::MultirotorRpcLibClient client;

    std::cout << "按下 Enter 启用 API 控制\n"; std::cin.get();
    client.enableApiControl(true);

    std::cout << "按下 Enter 启动无人机\n"; std::cin.get();
    client.armDisarm(true);

    std::cout << "按下 Enter 起飞\n"; std::cin.get();
    client.takeoffAsync(5)->waitOnLastTask();

    std::cout << "按下 Enter 以 1 m/s 的速度向 x 方向移动 5 米\n"; std::cin.get();
    auto position = client.getMultirotorState().getPosition(); // 从当前位置
    client.moveToPositionAsync(position.x() + 5, position.y(), position.z(), 1)->waitOnLastTask();

    std::cout << "按下 Enter 着陆\n"; std::cin.get();
    client.landAsync()->waitOnLastTask();

    return 0;
}
```

## 另请参见

* [示例](https://github.com/microsoft/AirSim/tree/main/Examples) 如何在其他项目中使用 AirSim 的内部基础设施
* [DroneShell](https://github.com/microsoft/AirSim/tree/main/DroneShell) 应用展示如何使用 C++ API 制作简单的界面来控制无人机
* [HelloSpawnedDrones](https://github.com/microsoft/AirSim/tree/main/HelloSpawnedDrones) 应用展示如何动态生成额外的车辆
* [Python APIs](apis.md)