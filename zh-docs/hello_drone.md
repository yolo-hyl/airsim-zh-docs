# Hello Drone

## Hello Drone 是如何工作的？

Hello Drone 使用 RPC 客户端连接到由 AirSim 自动启动的 RPC 服务器。  
RPC 服务器将所有命令路由到一个实现了 [MultirotorApiBase](https://github.com/Microsoft/AirSim/tree/main/AirLib//include/vehicles/multirotor/api/MultirotorApiBase.hpp) 的类。实质上，MultirotorApiBase 定义了我们从四旋翼获取数据和发送命令的抽象接口。目前，我们已经为基于 MavLink 的车辆提供了 MultirotorApiBase 的具体实现。针对 DJI 无人机平台，特别是 Matrice 的实现正在进行中。