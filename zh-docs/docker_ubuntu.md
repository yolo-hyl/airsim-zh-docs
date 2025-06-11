# 在 Linux 上使用 Docker 的 AirSim
我们有两个 Docker 选项。您可以构建一个用于运行 [airsim linux binaries](#binaries) 的镜像，或用于从源代码 [编译 Unreal Engine + AirSim](#source)。

## 二进制文件
#### 要求：
- 安装 [nvidia-docker2](https://github.com/NVIDIA/nvidia-docker#quickstart)

#### 构建 Docker 镜像
- 以下是默认参数。
  `--base_image`：这是我们将安装 airsim 的基础镜像。我们在 Ubuntu 18.04 和 CUDA 10.0 上进行了测试。
   您可以自担风险指定任何 [NVIDIA cudagl](https://hub.docker.com/r/nvidia/cudagl/)。
   `--target_image` 是您希望的 Docker 镜像名称。
   默认设置为 `airsim_binary`，与基础镜像使用相同标签。

```bash
$ cd Airsim/docker;
$ python build_airsim_image.py \
   --base_image=nvidia/cudagl:10.0-devel-ubuntu18.04 \
   --target_image=airsim_binary:10.0-devel-ubuntu18.04
```

- 通过以下命令验证您是否有镜像：
 `$ docker images | grep airsim`

#### 在 Docker 容器中运行 Unreal 二进制文件
- 获取 [Linux 二进制文件](https://github.com/Microsoft/AirSim/releases) 或在 Ubuntu 中打包您自己的项目。
以 Blocks 二进制文件为例。
您可以通过运行以下命令下载：

```bash
   $ cd Airsim/docker;
   $ ./download_blocks_env_binary.sh
```

修改它以获取所需的特定二进制文件。

- 在 Docker 容器中运行 Unreal 二进制文件
   语法为：

```bash
   $ ./run_airsim_image_binary.sh DOCKER_IMAGE_NAME UNREAL_BINARY_SHELL_SCRIPT UNREAL_BINARY_ARGUMENTS -- headless
```

   对于 Blocks，您可以运行：
   `$ ./run_airsim_image_binary.sh airsim_binary:10.0-devel-ubuntu18.04 Blocks/Blocks.sh -windowed -ResX=1080 -ResY=720`

   * `DOCKER_IMAGE_NAME`：与上一步骤中的 `target_image` 参数相同。默认输入为 `airsim_binary:10.0-devel-ubuntu18.04`。
   * `UNREAL_BINARY_SHELL_SCRIPT`：对于 Blocks 环境，它将是 `Blocks/Blocks.sh`。
   * [`UNREAL_BINARY_ARGUMENTS`](https://docs.unrealengine.com/en-us/Programming/Basics/CommandLineArguments)：
      对于 airsim，最相关的参数是 `-windowed`、`-ResX`、`-ResY`。点击链接查看所有选项。

  * 在无头模式下运行：
      在末尾加上后缀 `-- headless`：
```bash
$ ./run_airsim_image_binary.sh Blocks/Blocks.sh -- headless
```

- [指定一个 `settings.json`](#specifying-settingsjson)

## 源代码
#### 要求：
- 安装 [nvidia-docker2](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)
- 安装 [ue4-docker](https://docs.adamrehn.com/ue4-docker/configuration/configuring-linux)

#### 在 Docker 中构建 Unreal Engine：
- 要访问 Unreal Engine 的源代码，请在 Epic Games 网站上注册，并将其链接到您的 GitHub 账户，如 `Required Steps` 部分 [这里](https://docs.unrealengine.com/en-us/Platforms/Linux/BeginnerLinuxDeveloper/SettingUpAnUnrealWorkflow) 所述。

    请注意，您不需要执行 `步骤 2：在 Linux 上下载 UE4`！

- 构建 Unreal Engine 4.19.2 Docker 镜像。在我们的示例中，我们将使用 CUDA 10.0。
    `$ ue4-docker build 4.19.2 --cuda=10.0 --no-full`
    - [可选] `$ ue4-docker clean` 以释放一些空间。 [详情在此](https://docs.adamrehn.com/ue4-docker/commands/clean)
    - `ue4-docker` 支持 NVIDIA 的 cudagl dockerhub 上列出的所有 CUDA 版本 [这里](https://hub.docker.com/r/nvidia/cudagl/)。
    - 有关使用 `ue4-docker` 的高级配置，请参见 [此页面](https://docs.adamrehn.com/ue4-docker/building-images/advanced-build-options)。

- 磁盘空间：
    - Unreal 镜像和容器可能占用很多空间，尤其是如果您尝试多个版本。
    - 这是一个监视 Docker 使用空间并清理中间构建的有用链接列表：
        - [大型容器镜像入门](https://docs.adamrehn.com/ue4-docker/read-these-first/large-container-images-primer)
        - [`docker system df`](https://docs.docker.com/engine/reference/commandline/system_df/)
        - [`docker container prune`](https://docs.docker.com/engine/reference/commandline/container_prune/)
        - [`docker image prune`](https://docs.docker.com/engine/reference/commandline/image_prune/)
        - [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/)

#### 在 UE4 Docker 容器中构建 AirSim：
* 构建 AirSim Docker 镜像（基于我们刚刚构建的 Unreal 镜像）。
  以下是默认参数。
    - `--base_image`：这是我们将安装 airsim 的基础镜像。我们在 `adamrehn/ue4-engine:4.19.2-cudagl10.0` 上进行了测试。有关其他版本，请参见 [ue4-docker](https://docs.adamrehn.com/ue4-docker/building-images/available-container-images)。
    - `--target_image` 是您希望的 Docker 镜像名称。
   默认设置为 `airsim_source`，与基础镜像使用相同标签。

```bash
$ cd Airsim/docker;
$ python build_airsim_image.py \
   --source \
   --base_image adamrehn/ue4-engine:4.19.2-cudagl10.0 \
   --target_image=airsim_source:4.19.2-cudagl10.0
```

#### 运行 AirSim 容器
* 通过以下命令运行我们构建的 airsim 源镜像：

```bash
   ./run_airsim_image_source.sh airsim_source:4.19.2-cudagl10.0
```

   语法为 `./run_airsim_image_source.sh DOCKER_IMAGE_NAME -- headless`
   `-- headless`：后缀此命令以在可选的无头模式下运行。

* 在容器内，您可以在 `/home/ue4` 下看到 `UnrealEngine` 和 `AirSim`。
* 在容器内启动 Unreal Engine：
   `ue4@HOSTMACHINE:~$ /home/ue4/UnrealEngine/Engine/Binaries/Linux/UE4Editor`
* [指定一个 airsim settings.json](#specifying-settingsjson)
* 继续 [AirSim 的 Linux 文档](build_linux.md#build-unreal-environment)。

#### [杂项] 在 `airsim_source` 容器中打包 Unreal 环境
* 让我们以 Blocks 环境为例。
    在以下脚本中，通过 `project` 指定您 Unreal 项目文件的完整路径，并通过 `archivedirectory` 指定您希望放置二进制文件的目录。

```bash
$ /home/ue4/UnrealEngine/Engine/Build/BatchFiles/RunUAT.sh BuildCookRun -platform=Linux -clientconfig=Shipping -serverconfig=Shipping -noP4 -cook -allmaps -build -stage -prereqs -pak -archive \
-archivedirectory=/home/ue4/Binaries/Blocks/ \
-project=/home/ue4/AirSim/Unreal/Environments/Blocks/Blocks.uproject
```

这将创建一个 Blocks 二进制文件在 `/home/ue4/Binaries/Blocks/`。
您可以通过运行 `/home/ue4/Binaries/Blocks/LinuxNoEditor/Blocks.sh -windowed` 进行测试。

### 指定 settings.json
#### `airsim_binary` Docker 镜像：
  - 我们将宿主机的 `PATH/TO/Airsim/docker/settings.json` 映射到 Docker 容器的 `/home/airsim_user/Documents/AirSim/settings.json`。
  - 因此，我们可以通过简单修改 `PATH_TO_YOUR/settings.json` 来加载任何设置文件，修改 [`run_airsim_image_binary.sh`](https://github.com/Microsoft/AirSim/blob/main/docker/run_airsim_image_binary.sh) 中的以下代码片段。

```bash
nvidia-docker run --runtime=nvidia -it \
      -v $PATH_TO_YOUR/settings.json:/home/airsim_user/Documents/AirSim/settings.json \
      -v $UNREAL_BINARY_PATH:$UNREAL_BINARY_PATH \
      -e SDL_VIDEODRIVER=$SDL_VIDEODRIVER_VALUE \
      -e SDL_HINT_CUDA_DEVICE='0' \
      --net=host \
      --env="DISPLAY=$DISPLAY" \
      --env="QT_X11_NO_MITSHM=1" \
      --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
      -env="XAUTHORITY=$XAUTH" \
      --volume="$XAUTH:$XAUTH" \
      --rm \
      $DOCKER_IMAGE_NAME \
      /bin/bash -c "$UNREAL_BINARY_COMMAND"
```

**注意：** Docker 版本 >=19.03（使用 `docker -v` 检查），原生支持 Nvidia GPU，因此使用给定的 `--gpus all` 标志运行 -

```bash
docker run --gpus all -it \
    ...
```

#### `airsim_source` Docker 镜像：

  * 我们将宿主机的 `PATH/TO/Airsim/docker/settings.json` 映射到 Docker 容器的 `/home/airsim_user/Documents/AirSim/settings.json`。
  * 因此，我们可以通过简单修改 `PATH_TO_YOUR/settings.json` 来加载任何设置文件，修改 [`run_airsim_image_source.sh`](https://github.com/Microsoft/AirSim/blob/main/docker/run_airsim_image_source.sh) 中的以下代码片段：

```bash
   nvidia-docker run --runtime=nvidia -it \
      -v $(pwd)/settings.json:/home/airsim_user/Documents/AirSim/settings.json \
      -e SDL_VIDEODRIVER=$SDL_VIDEODRIVER_VALUE \
      -e SDL_HINT_CUDA_DEVICE='0' \
      --net=host \
      --env="DISPLAY=$DISPLAY" \
      --env="QT_X11_NO_MITSHM=1" \
      --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
      -env="XAUTHORITY=$XAUTH" \
      --volume="$XAUTH:$XAUTH" \
      --rm \
   $DOCKER_IMAGE_NAME
```