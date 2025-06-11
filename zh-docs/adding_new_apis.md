# 在 AirSim 中添加新 API

添加新 API 需要修改源代码。许多更改是机械性的，并且是为 AirSim 支持的各种抽象级别所必需的。下面描述了需要修改的主要文件，以及一些提交和 PR 的示例。在某些地方可能会链接到 PR 或提交的特定部分，但查看整个 diff 会对理解工作流程更有帮助。也请不要犹豫，如果不确定如何进行更改或需要反馈，可以随时打开一个 issue 或草稿 PR。

## 实现 API

在添加调用和处理 API 的包装代码之前，首先需要实现 API。确切的实现文件根据 API 的功能而异。以下提供了一些示例，可能会帮助你入门。

### 基于车辆的 API

`moveByVelocityBodyFrameAsync` API 用于在多旋翼的 X-Y 坐标系中进行基于速度的运动。

主要实现位于 [MultirotorBaseApi.cpp](https://github.com/microsoft/AirSim/pull/3169/files#diff-29ac01a05077b6e8e1f09221a113f779c952a80dc8823725eb451a9fc5d7de5f) 中，这里实现了大部分多旋翼 API。

在某些情况下，可能需要额外的结构来存储数据，[`getRotorStates` API](https://github.com/microsoft/AirSim/pull/3242) 就是一个很好的例子，此处的 `RotorStates` 结构在两个地方定义，以便从 RPC 转换到内部代码。这也需要在 AirLib 以及 Unreal/Plugins 中进行修改。

### 环境相关 API

这些 API 需要与仿真环境本身进行交互，因此它们很可能会在 `Unreal/Plugins` 文件夹内实现。

- `simCreateVoxelGrid` API 用于生成并保存环境的 binvox 格式的网格 - [WorldSimApi.cpp](https://github.com/microsoft/AirSim/pull/3209/files#diff-89d4ec9b62486b1322e5ba2dd9936b13962f9ed113ec5e35a0678846889c7e2d)

- `simAddVehicle` API 在运行时创建车辆 - [SimMode*, WorldSimApi 文件](https://github.com/microsoft/AirSim/pull/2390/files#diff-fcc0aa1fbc74a924fccd12589295aceeea59074c94256eccba7df3ce85d3a26c)

### 物理相关 API

`simSetWind` API 显示了修改物理行为并为此添加 API + 设置字段的示例。有关代码的详细信息，请参见 [该 PR](https://github.com/microsoft/AirSim/pull/2867)。

## RPC 包装器

这些 API 使用 [msgpack-rpc 协议](https://github.com/msgpack-rpc/msgpack-rpc) 通过 [rpclib](http://rpclib.net/) 进行 TCP/IP 通信，该库由 [Tamás Szelei](https://github.com/sztomi) 开发，允许你使用多种编程语言，包括 C++、C#、Python、Java 等。当 AirSim 启动时，会打开 41451 端口（此端口可以通过 [settings](settings.md) 修改），并监听传入请求。Python 或 C++ 客户端代码连接到此端口并使用 [msgpack 序列化格式](https://msgpack.org) 发送 RPC 调用。

要添加 RPC 代码以调用新 API，请按照以下步骤操作。请遵循文件中定义的其他 API 的实现。

1. 在服务器中添加一个 RPC 处理程序，该处理程序调用你在 [RpcLibServerBase.cpp](https://github.com/microsoft/AirSim/blob/main/AirLib/src/api/RpcLibServerBase.cpp) 中实现的方法。特定于车辆的 API 位于各自的车辆子文件夹内。

2. 在 [RpcClientBase.cpp](https://github.com/microsoft/AirSim/blob/main/AirLib/src/api/RpcLibClientBase.cpp) 中添加 C++ 客户端 API 方法。

3. 在 [client.py](https://github.com/microsoft/AirSim/blob/main/PythonClient/airsim/client.py) 中添加 Python 客户端 API 方法。如有需要，请在 [types.py](https://github.com/microsoft/AirSim/blob/main/PythonClient/airsim/types.py) 中添加或修改结构定义。

## 测试

测试是确保 API 按预期工作所必需的。为此，正如预期的那样，你需要使用源构建的 AirSim 和 Blocks 环境。除此之外，如果使用 Python API，必须使用源中的 `airsim` 包，而不是 PyPI 包。下面描述了使用源 pacote 的两种方式 –

1. 使用 [setup_path.py](https://github.com/microsoft/AirSim/blob/main/PythonClient/multirotor/setup_path.py)。它将设置路径，以便使用本地的 airsim 模块，而不是 pip 安装的包。这是许多脚本中使用的方法，因为用户只需运行脚本即可。
    将示例脚本放在 `PythonClient` 中的一个文件夹内，例如 `multirotor`、`car` 等。你也可以创建一个文件夹以保持分开，并从其他文件夹复制 `setup_path.py` 文件。
    在你的文件中，在 `import airsim` 之前添加 `import setup_path`。现在将使用最新的主 API（或当前检出的任何分支）。

2. 使用 [本地项目 pip 安装](https://pip.pypa.io/en/stable/cli/pip_install/#local-project-installs)。常规安装将创建当前源的副本并使用，而可编辑安装（在 `PythonClient` 文件夹内执行 `pip install -e .`）将在每次更改 Python API 文件时更新包。在处理多个分支或 API 尚未最终确定时，可编辑安装具有优势。

建议使用虚拟环境处理 Python 打包，以避免破坏任何现有设置。

在打开 PR 时，请确保遵循 [编码指南](coding_guidelines.md)。还请在 Python 文件中为 API 添加文档字符串，并包含脚本中所需的任何示例脚本和设置。