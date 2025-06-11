AirSim 提供了一个基于 Python 的事件相机模拟器，旨在实现高性能并能够与模拟环境实时运行。

#### 事件相机
事件相机是一种特殊的视觉传感器，它测量对数亮度的变化，并仅报告“事件”。每个事件是一组四个值，当对数亮度的绝对变化超过某个阈值时，就会生成一个事件。事件包含测量的时间戳、像素位置（x 和 y 坐标）以及极性：根据对数亮度是增加还是减少的情况，其值为 +1 或 -1。大多数事件相机的时间分辨率在微秒级别，这使得它们的速度显著快于 RGB 传感器，并且具有高动态范围和低运动模糊。有关事件相机的更多详细信息，请参阅 [RPG-UZH 的这个教程](http://rpg.ifi.uzh.ch/docs/scaramuzza/Tutorial_on_Event_Cameras_Scaramuzza.pdf)。

#### AirSim 事件模拟器

AirSim 事件模拟器使用两幅连续的 RGB 图像（转换为灰度），并根据图像之间的对数亮度变化计算“过去事件”。这些事件作为字节流报告，格式如下：

`<x> <y> <timestamp> <pol>`

x 和 y 是事件触发的像素位置，timestamp 是以微秒为单位的全局时间戳，pol 是根据亮度是增加还是减少而定的 +1/-1。除了这个字节流，还构建了一个二维帧的事件累积，称为“事件图像”，它将 +1 事件可视化为红色像素，将 -1 事件可视化为蓝色像素。下面是一个事件图像的示例：

![image](images/event_sim.png)

#### 用法
用于与 AirSim 一起运行事件模拟器的示例脚本位于 https://github.com/microsoft/AirSim/blob/main/PythonClient/eventcamera_sim/test_event_sim.py。可以将以下可选命令行参数传递给该脚本。

```
args.width, args.height (float): 模拟事件相机分辨率
args.save (bool): 是否将事件数据保存到文件，args.debug (bool): 是否将模拟事件显示为图像
```

实际事件模拟的实现，使用 Python 和 numba 编写，位于 https://github.com/microsoft/AirSim/blob/main/PythonClient/eventcamera_sim/event_simulator.py。事件模拟器的初始化如下，参数控制相机的分辨率。

```
from event_simulator import *
ev_sim = EventSimulator(W, H)
```

事件的实际计算通过 `image_callback` 函数触发，每当获取到新的 RGB 图像时，该函数就会被执行。第一次调用该函数时，由于没有“前一”图像，它充当事件模拟的初始化。

```
event_img, events = ev_sim.image_callback(img, ts_delta)
```
该函数的行为类似于回调（每次接收到新图像时调用），返回的事件图像为一维的 +1/-1 值数组，指示每个像素处是否看到了事件，但不指示事件的时间/数量。这个一维数组可以通过函数 `convert_event_img_rgb` 转换为红/蓝事件图像。`events` 是一个包含事件的 numpy 数组，每个事件的格式为 `<x> <y> <timestamp> <pol>`。

通过这个函数，事件模拟计算过去图像和当前图像之间的差异，并计算出一系列事件，这些事件随后作为一个 numpy 数组返回。然后，可以将其附加到文件中。

有许多参数可以调整，以实现事件模拟的视觉保真度/性能。主要需要调整的因素包括：

1. 相机的分辨率。
2. 对数亮度阈值 `TOL`，用于确定检测到的变化是否算作事件。

注意：目前每对图像生成的事件数量也有一个最大限制，这个限制也可以调整。

#### 算法
事件模拟器的工作大致遵循以下操作步骤：
1. 获取当前帧与前一帧的对数亮度差。  
2. 遍历所有像素，根据对数亮度变化的阈值计算每个像素的极性。  
3. 根据超出阈值的亮度变化程度确定每个像素要触发的事件数量。设 $N_{max}$ 为单个像素最多可以发生的事件数量，则在像素位置 $u$ 处模拟的总触发次数为 $N_e(u) = min(N_{max}, \frac{\Delta L(u)}{TOL})$。  
4. 通过插值计算每个事件的时间戳，以便在前后的图像捕获之间的经过时间插值。  
$t = t_{prev} + \frac{\Delta T}{N_e(u)}$  
5. 通过在每个像素上模拟事件并按时间戳排序来生成输出字节流。