# 常见问题解答

---

## 一般性问题

* [当 Unreal 编辑器不是活动窗口时，它会变得缓慢](#unreal-editor-is-slow-when-it-is-not-the-active-window)
* [我的鼠标在 Unreal 中消失了](#my-mouse-disappears-in-unreal)
* [设置文件在哪里，如何修改？](#where-is-the-setting-file-and-how-do-i-modify-it)
* [如何启动我的无人机？](#how-do-i-arm-my-drone)
* [调用 API 时出现错误](#when-making-api-call-i-get-error)
* [编译 Unreal 项目时出现 Eigen not found 错误。](#im-getting-eigen-not-found-error-when-compiling-unreal-project)
* [出现问题了。我该如何调试？](#something-went-wrong-how-do-i-debug)
* [在分割视图中，颜色代表什么？](#what-do-the-colors-mean-in-the-segmentation-view)
* [Unreal 4.xx 看起来没 4.yy 好](#unreal-4xx-doesnt-look-as-good-as-4yy)
* [我可以使用 Xbox 控制器飞行吗？](#can-i-use-an-xbox-controller-to-fly)
* [我可以用 AirSim 构建六轴飞行器吗？](#can-i-build-a-hexacopter-with-airsim)
* [我如何使用 AirSim 多辆车辆？](#how-do-i-use-airsim-with-multiple-vehicles)
* [你需要什么样的电脑？](#what-computer-do-you-need)
* [我该如何报告问题？](#how-do-i-report-issues)

---

<!-- ======================================================================= -->
## 一般性问题
<!-- ======================================================================= -->

###### 当 Unreal 编辑器不是活动窗口时，它会变得缓慢

>前往 编辑/编辑器偏好，选择“所有设置”，在搜索框中输入“CPU”。 
>它应该找到标题为“在后台使用更少的 CPU”的设置，你需要取消勾选该复选框。

<!-- ======================================================================= -->

###### 我的鼠标在 Unreal 中消失了

>是的，Unreal 会锁定鼠标，我们不绘制鼠标。因此，要恢复你的鼠标，请使用 Alt+TAB 切换到其他窗口。为了完全避免这种情况，前往项目设置 > 在 Unreal 编辑器中，进入输入标签并禁用所有鼠标捕获设置。

<!-- ======================================================================= -->

###### 设置文件在哪里，如何修改？

>AirSim 将在 `~/Documents/AirSim/settings.json` 创建空设置文件。你可以查看可用的 [设置选项](settings.md)。

<!-- ======================================================================= -->

###### 如何启动我的无人机？

>如果你使用的是 simple_flight，你的飞行器已经启动，可以飞行。对于 PX4，你可以通过将遥控器上的两个摇杆向下并向中心推来启动。

<!-- ======================================================================= -->

###### 调用 API 时出现错误

>如果你收到这个错误，
>```
>TypeError: unsupported operand type(s) for *: 'AsyncIOLoop' and 'float'
>```
>这可能是由于 Python 中 tornado 包的版本升级到 > 5.0，导致与 `msgpack-rpc-python` 发生冲突，而后者需要 tornado 包 < 5.0。要解决此问题，你可以这样更新包：
>```
>pip install --upgrade msgpack-rpc-python
>```
>但这可能会导致其他问题（例如，PyTorch 0.4+），因为这将卸载较新的 tornado 并重新安装较旧的版本。为了避免这种情况，你应该创建新的 [conda 环境](https://conda.io/docs/user-guide/tasks/manage-environments.html)。

<!-- ======================================================================= -->

###### 编译 Unreal 项目时出现 Eigen not found 错误。

>这很可能是因为 AirSim 没有被构建并且插件文件夹被复制到 Unreal 项目文件夹中。为了解决这个问题，请确保你先 [构建了 AirSim](build_windows.md)（在 Windows 中运行 `build.cmd`）。

<!-- ======================================================================= -->

###### 出现问题了。我该如何调试？

>首先，从异常窗口启用 C++ 异常：

>![exceptions](images/exceptions.png)

>并复制在执行过程中你看到的所有相关异常的堆栈跟踪（例如，可能有一个来自 VSPerf140 的初始异常可以忽略），然后将这些调用栈粘贴到新的 AirSim GitHub 问题中，谢谢。

<!-- ======================================================================= -->

###### 在分割视图中，颜色代表什么？

>查看 [摄像机视图](camera_views.md) 以获取有关摄像机视图及其更改的信息。

<!-- ======================================================================= -->

###### Unreal 4.xx 看起来没 4.yy 好

>Unreal 4.15 添加了功能，可以根据具体情况禁用植被 LOD 抖动，通过在植被材质中取消勾选 `Dithered LOD Transition` 复选框。请注意，所有 LOD 使用的所有材质都需要勾选该复选框，以便抖动的 LOD 过渡能够正常工作。选中后，生成植被的过渡将更加平滑，看起来比 4.14 更好。

<!-- ======================================================================= -->

###### 我可以使用 Xbox 控制器飞行吗？

>有关详细信息，请参见 [Xbox 控制器](xbox_controller.md)。

<!-- ======================================================================= -->

###### 我可以用 AirSim 构建六轴飞行器吗？

>请查看 [如何构建六轴飞行器](https://github.com/microsoft/airsim/wiki/hexacopter)。

<!-- ======================================================================= -->

###### 我如何使用 AirSim 多辆车辆？

>这里是 [多车辆设置指南](multi_vehicle.md)。

<!-- ======================================================================= -->

###### 你需要什么样的电脑？

>这取决于你的 Unreal 环境有多大。AirSim 附带的 Blocks 环境非常基础，适用于典型的笔记本电脑。我们自己用于研究的 [模块化邻里包](https://www.unrealengine.com/marketplace/modular-neighborhood-pack) 需要至少 4GB 的 GPU 内存。 [开放世界环境](https://www.unrealengine.com/marketplace/open-world-demo-collection) 需要 8GB 的 GPU 内存。我们的典型开发机器有 32GB 内存和 NVIDIA TitanX 以及 [快速硬盘](hard_drive.md)。

<!-- ======================================================================= -->

###### 我该如何报告问题？

>最好包括你的配置，如下所示。如果你还可以包含日志，这也可以加快调查速度。

>```
>操作系统：Windows 10 64位
>CPU：Intel Core i7
>GPU：Nvidia GTX 1080
>内存：32 GB
>飞行控制器：Pixhawk v2
>遥控器：Futaba
>```

>如果你修改了默认的 `~/Document/AirSim/settings.json`，请一并包含你的设置。

>如果你使用的是 PX4，请尝试 [从 MavLink 或 PX4 捕获日志](px4_logging.md)。

>通过 [GitHub Issues](https://github.com/microsoft/airsim/issues) 提交问题。

<!-- ======================================================================= -->
## 其他
<!-- ======================================================================= -->

* [Linux 构建常见问题](build_linux.md#faq)
* [Windows 构建常见问题](build_windows.md#faq)
* [PX4 设置常见问题](px4_setup.md#faq)
* [遥控常见问题](remote_control.md#faq)
* [Unreal 块环境常见问题](unreal_blocks.md#faq)
* [Unreal 自定义环境常见问题](unreal_custenv.md#faq)
* [打包 AirSim](build_faq.md#packaging-a-binary-including-the-airsim-plugin)