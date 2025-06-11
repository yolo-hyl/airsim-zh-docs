```markdown
# 构建 PX4

## 源代码

获取 PX4 源代码非常简单：
```
sudo apt-get install git
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh --no-sim-tools
cd PX4-Autopilot
```

现在要构建它，你需要合适的工具。

## PX4 构建工具

完整的说明可以在 [dev.px4.io](https://docs.px4.io/master/en/dev_setup/building_px4.html) 网站找到，
但我们在这里复制了相关的部分以方便你。

（请注意， [BashOnWindows](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide) 可以用来构建
PX4 固件，只需按照该页面底部的 BashOnWindows 指示操作，然后继续进行 PX4 的 Ubuntu 设置。）

## 构建 SITL 版本

现在你可以从你之前创建的 Firmware 文件夹中构建运行在 posix 上的 SITL 版本：
```
make px4_sitl_default none_iris
```

注意：这个构建系统相当特别，它知道如何更新 git 子模块（而且有很多），然后它运行 cmake（如果需要），接着运行构建。因此，从某种意义上说，根 Makefile 是一个元元 makefile :-) 你可能会看到如下提示：

```shell
 *******************************************************************************
 *   IF YOU DID NOT CHANGE THIS FILE (OR YOU DON'T KNOW WHAT A SUBMODULE IS):  *
 *   Hit 'u' and <ENTER> to update ALL submodules and resolve this.            *
 *   (performs git submodule sync --recursive                                  *
 *    and git submodule update --init --recursive )                            *
 *******************************************************************************
```
每次看到这个提示时，请在键盘上输入 'u'。

这不会花很长时间，大约 2 分钟。如果所有操作成功，最后一行将会链接 `px4` 应用程序，
然后你可以使用以下命令运行它：

```
make px4_sitl_default none_iris
```

你应该会看到类似如下的输出：

```
creating new parameters file
creating new dataman file

______  __   __    ___ 
| ___ \ \ \ / /   /   |
| |_/ /  \ V /   / /| |
|  __/   /   \  / /_| |
| |     / /^\ \ \___  |
\_|     \/   \/     |_/

px4 starting.

18446744073709551615 WARNING: setRealtimeSched failed (not run as root?)
ERROR [param] importing from 'rootfs/eeprom/parameters' failed (-1)
Command 'param' failed, returned 1
  SYS_AUTOSTART: curr: 0 -> new: 4010
  SYS_MC_EST_GROUP: curr: 2 -> new: 1
INFO  [dataman] Unkown restart, data manager file 'rootfs/fs/microsd/dataman' size is 11797680 bytes
  BAT_N_CELLS: curr: 0 -> new: 3
  CAL_GYRO0_ID: curr: 0 -> new: 2293768
  CAL_ACC0_ID: curr: 0 -> new: 1376264
  CAL_ACC1_ID: curr: 0 -> new: 1310728
  CAL_MAG0_ID: curr: 0 -> new: 196616

```

这很好，第一次运行为 SITL 模式设置了 px4 参数。第二次运行时输出会更少。
这个应用程序也是一个交互式控制台，你可以输入命令。输入 'help' 以查看有哪些命令，输入 ctrl-C 可以停止它。你可以随时这样做并重新启动，这是重置任何异常状态的一种很好的方法（相当于 Pixhawk 硬件重启）。

## ARM 嵌入式工具

如果你打算为实际的 Pixhawk 硬件构建 PX4 固件，那么你将需要用于 ARM Cortex-M4 芯片组的 gcc 交叉编译器。你可以通过 PX4 DevGuide 获取这个编译器，具体是在它们的 `ubuntu_sim_nuttx.sh` 设置脚本中。

按照这些设置说明后，你可以通过输入命令 `arm-none-eabi-gcc --version` 来验证安装。你应该看到如下的输出：

```
arm-none-eabi-gcc (GNU Tools for Arm Embedded Processors 7-2017-q4-major) 7.2.1 20170904 (release) [ARM/embedded-7-branch revision 255204]
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

## 为 ARM 硬件构建 PX4

现在你可以为实际的 Pixhawk 硬件构建 PX4 固件：

```
make px4_fmu-v4
```

这个构建将花费更长时间，因为它构建的内容更多，包括 NuttX 实时操作系统、Pixhawk 飞控中的所有传感器驱动程序等等。它还在超级压缩模式下运行编译器，以便可以将所有这些内容适配到 1 兆字节的 ROM 中！！

一个好的小提示是，你可以插入你的 Pixhawk USB，并输入 `make px4fmu-v2_default upload` 将这些全新的 bits 刷写到硬件中，因此你无需使用 QGroundControl。

## 一些有用的参数

PX4 有许多可自定义的参数（实际上超过 700 个），为了与 AirSim 取得最佳效果，我们发现以下参数很有用：

```
// 一定要启用新的位置估算模块：
param set SYS_MC_EST_GROUP 2

// 增加巡航速度的默认限制，以便你可以更快地在大地图上移动。
param MPC_XY_CRUISE 10             
param MPC_XY_VEL_MAX 10
param MPC_Z_VEL_MAX_DN 2

// 增加着陆时自动解锁的超时时间，以便任何长期运行的应用程序无须担心这个问题
param COM_DISARM_LAND 60

// 使飞行时不需要无线电控制（切勿在真实无人机上这样做）
param NAV_RCL_ACT 0

// 启用新的系统日志记录器，以获取 PX4 日志中的更多信息
param set SYS_LOGGER 1
```

## 使用 BashOnWindows

请参见 [Bash on Windows Toolchain](https://dev.px4.io/en/setup/dev_env_windows_bash_on_win.html)。
```