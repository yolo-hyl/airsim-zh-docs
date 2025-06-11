# AirSim 开发环境在 Azure 上的搭建

本文档解释了如何自动化创建 Azure 上的开发环境，并使用 Visual Studio Code 连接到 AirSim 的 Python 应用程序进行编码和调试。

## 自动部署您的 Azure 虚拟机
点击蓝色按钮开始 Azure 部署（模板已经预填充为以下两个教程推荐的虚拟机大小）。

<a href="https://aka.ms/AA8umgt" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>  
*注意：虚拟机的部署和配置过程可能需要超过 20 分钟的时间才能完成。*

### 关于 Azure 虚拟机的部署
- 当使用 Azure 试用账户时，默认的 vCPU 配额不足以分配所需的虚拟机来运行 AirSim。如果是这种情况，您在尝试创建虚拟机时会看到错误，并且需要提交配额增加的请求。**务必了解您将如何以及需要支付多少费用来使用虚拟机。**

- 为了避免在未使用虚拟机时仍产生费用，请记得从 [Azure 门户](https://portal.azure.com) 中解除其资源分配，或者使用以下 Azure CLI 命令：
```bash
az vm deallocate --resource-group MyResourceGroup --name MyVMName
```

## 从 Visual Studio Code 和远程 SSH 进行编码和调试
- 安装 Visual Studio Code
- 安装 *Remote - SSH* 扩展
- 按 `F1` 并运行 `Remote - SSH: Connect to host...` 命令
- 添加刚创建的虚拟机的详细信息，例如 `AzureUser@11.22.33.44`
- 再次运行 `Remote - SSH: Connect to host...` 命令，现在选择新添加的连接。
- 连接后，点击 Visual Studio Code 中的 `Clone Repository` 按钮，或者在远程虚拟机中克隆此存储库并仅打开 *`azure` 文件夹*，或创建一个全新的存储库，克隆它并从这个存储库中复制 `azure` 文件夹的内容。打开该目录非常重要，这样 Visual Studio Code 才能使用特定的 `.vscode` 目录而不是通用的 AirSim `.vscode` 目录。该目录包含推荐安装的扩展、启动 AirSim 的任务以及 Python 应用程序的启动配置。
- 安装所有推荐的扩展
- 按 `F1` 并选择 `Tasks: Run Task` 选项。然后，从 Visual Studio Code 中选择 `Start AirSim` 任务以执行 `start-airsim.ps1` 脚本。
- 打开 `app` 目录中的 `multirotor.py` 文件
- 开始用 Python 进行调试
- 完成后，请记得停止并解除 Azure 虚拟机的分配以避免额外费用。

## 从本地 Visual Studio Code 进行编码和调试，并通过转发端口连接到 AirSim

*注意：此场景将使用两个 Visual Studio Code 实例。第一个将用作通过 SSH 转发端口到 Azure 虚拟机和执行远程进程的桥梁，第二个则用于本地 Python 开发。在从本地 Python 代码访问虚拟机时，需要保持 `Remote - SSH` 的 Visual Studio Code 实例处于打开状态，同时在第二个实例中进行本地 Python 环境的工作。*

- 打开第一个 Visual Studio Code 实例
- 按照前一部分的步骤通过 `Remote - SSH` 连接
- 在 *Remote Explorer* 中，将端口 `41451` 添加为转发到 localhost 的端口
- 按照前一个场景中所述，在具有远程会话的 Visual Studio Code 中运行 `Start AirSim` 任务，或手动在虚拟机中启动 AirSim 二进制文件
- 打开第二个 Visual Studio Code 实例，且不需断开或关闭第一个实例
- 在本地克隆此存储库并仅在 Visual Studio Code 中打开 *`azure` 文件夹*，或创建一个全新的存储库，克隆它并复制此存储库中的 `azure` 文件夹的内容。
- 在 `app` 目录中运行 `pip install -r requirements.txt`
- 打开 `app` 目录中的 `multirotor.py` 文件
- 开始用 Python 进行调试
- 完成后，请记得停止并解除 Azure 虚拟机的分配以避免额外费用。

## 使用 Docker 运行
一旦 AirSim 环境和 Python 应用程序准备就绪，您可以将所有内容打包为 Docker 镜像。`azure` 目录中的示例项目已经准备好使用 Docker 运行预构建的 AirSim 二进制文件和 Python 代码。

这是一个非常适合进行大规模仿真的场景。例如，您可以为相同的仿真设置多个不同的配置，并使用 Azure 容器服务以并行、无人值守的方式执行它们。

由于 AirSim 需要访问主机的 GPU，因此需要使用支持 GPU 的 Docker 运行时。有关在 Docker 中运行 AirSim 的更多信息，请点击 [这里](docker_ubuntu.md)。

使用 Azure 容器服务运行此镜像时，唯一的额外要求是在将要部署的容器组中添加 GPU 支持。

它可以使用来自 DockerHub 的公共 Docker 镜像或部署到私有 Azure 容器注册表的镜像。

### 构建 Docker 镜像

```bash
docker build -t <your-registry-url>/<your-image-name> -f ./docker/Dockerfile .
```

## 使用不同的 AirSim 二进制文件

要使用不同的 AirSim 二进制文件，请首先查看官方文档 [如何在 Windows 上构建 AirSim](build_windows.md) 和 [如何在 Linux 上构建 AirSim](build_linux.md)，如果您还希望与 Docker 一起运行它。

一旦您有了包含新 AirSim 环境的 zip 文件（或更愿意使用 [官方发布](https://github.com/microsoft/AirSim/releases) 中的一个），您需要修改存储库中的 `azure` 目录中的一些脚本，使其指向新的环境：
- 在 [`azure/azure-env-creation/configure-vm.ps1`](https://github.com/microsoft/AirSim/blob/main/azure/azure-env-creation/configure-vm.ps1) 中，使用新值修改所有以 `$airSimBinary` 开头的参数
- 在 [`azure/start-airsim.ps1`](https://github.com/microsoft/AirSim/blob/main/azure/start-airsim.ps1) 中，使用新值修改 `$airSimExecutable` 和 `$airSimProcessName`

如果您使用的是 Docker 镜像，还需要一个 Linux 二进制 zip 文件，并修改以下与 Docker 相关的文件：
- 在 [`azure/docker/Dockerfile`](https://github.com/microsoft/AirSim/blob/main/azure/docker/Dockerfile) 中，修改 `AIRSIM_BINARY_ZIP_URL` 和 `AIRSIM_BINARY_ZIP_FILENAME` 的 ENV 声明为新值
- 在 [`azure/docker/docker-entrypoint.sh`](https://github.com/microsoft/AirSim/blob/main/azure/docker/docker-entrypoint.sh) 中，使用新值修改 `AIRSIM_EXECUTABLE` 

## 维护此开发环境

此开发环境的多个组件（ARM 模板、初始化脚本和 VSCode 任务）直接依赖于当前目录结构、文件名称和存储库位置。在计划修改/分叉这些内容时，请确保检查每个脚本和模板以进行必要的调整。