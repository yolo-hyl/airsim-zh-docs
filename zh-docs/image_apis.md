# 图像 API

如果您不熟悉 AirSim API，请先阅读 [通用 API 文档](apis.md)。

## 获取单张图像

以下是从名为 "0" 的相机获取单张图像的示例代码。返回的值是 png 格式图像的字节。有关获取无压缩图像、其他格式以及可用相机的信息，请查看下一部分。

### Python

```python
import airsim #pip install airsim

# 对于汽车使用 CarClient() 
client = airsim.MultirotorClient()

png_image = client.simGetImage("0", airsim.ImageType.Scene)
# 对图像进行处理
```

### C++

```cpp
#include "vehicles/multirotor/api/MultirotorRpcLibClient.hpp"

int getOneImage() 
{
    using namespace msr::airlib;
    
    // 对于汽车使用 CarRpcLibClient
    MultirotorRpcLibClient client;

    std::vector<uint8_t> png_image = client.simGetImage("0", VehicleCameraBase::ImageType::Scene);
    // 对图像进行处理
}
```

## 更灵活地获取图像

`simGetImages` API 使用起来比 `simGetImage` 稍微复杂，例如，您可以在单个 API 调用中获取左侧相机视图、右侧相机视图和左侧相机的深度图像。`simGetImages` API 还允许您获取无压缩图像以及浮点单通道图像（而不是 3 通道（RGB），每个 8 位）。

### Python

```python
import airsim #pip install airsim

# 对于汽车使用 CarClient() 
client = airsim.MultirotorClient()

responses = client.simGetImages([
    # png 格式
    airsim.ImageRequest(0, airsim.ImageType.Scene), 
    # 无压缩 RGB 数组字节
    airsim.ImageRequest(1, airsim.ImageType.Scene, False, False),
    # 浮点无压缩图像
    airsim.ImageRequest(1, airsim.ImageType.DepthPlanar, True)])
 
# 处理包含图像数据、姿态、时间戳等的响应
```

#### 使用 AirSim 图像与 NumPy

如果您计划使用 numpy 进行图像处理，您应该获取无压缩 RGB 图像，然后像这样转换为 numpy：

```python
responses = client.simGetImages([airsim.ImageRequest("0", airsim.ImageType.Scene, False, False)])
response = responses[0]

# 获取 numpy 数组
img1d = np.fromstring(response.image_data_uint8, dtype=np.uint8) 

# 将数组重塑为 4 通道图像数组 H X W X 4
img_rgb = img1d.reshape(response.height, response.width, 3)

# 原始图像是垂直翻转的
img_rgb = np.flipud(img_rgb)

# 写入 png 
airsim.write_png(os.path.normpath(filename + '.png'), img_rgb) 
```

#### 快速提示
- API `simGetImage` 返回的是 `binary string literal`，这意味着您可以简单地将其转储到二进制文件中以创建 .png 文件。然而，如果您想以任何其他方式处理它，可以使用便捷函数 `airsim.string_to_uint8_array`。该函数将二进制字符串字面值转换为 NumPy uint8 数组。

- API `simGetImages` 可以接受来自任何相机的多种图像类型的请求。在单次调用中，您可以指定图像是 png 压缩的、RGB 无压缩的或浮点数组。对于 png 压缩图像，您会得到 `binary string literal`。对于浮点数组，您将获得 Python 的 float64 列表。您可以使用以下语句将此浮点数组转换为 NumPy 2D 数组：
    ```
    airsim.list_to_2d_float_array(response.image_data_float, response.width, response.height)
    ```
    您还可以使用 `airsim.write_pfm()` 函数将浮点数组保存为 .pfm 文件（可携带浮点图格式）。

- 如果您希望在调用其中一个图像 API 时查询位置信息和方向信息，可以使用 `client.simPause(True)` 和 `client.simPause(False)` 在调用图像 API 和查询所需物理状态时暂停仿真，确保物理状态在图像 API 调用后立即保持不变。

### C++

```cpp
int getStereoAndDepthImages() 
{
    using namespace msr::airlib;
    
    typedef VehicleCameraBase::ImageRequest ImageRequest;
    typedef VehicleCameraBase::ImageResponse ImageResponse;
    typedef VehicleCameraBase::ImageType ImageType;

    // 对于汽车使用
    // CarRpcLibClient client;
    MultirotorRpcLibClient client;

    // 获取右侧、左侧和深度图像。前两个为 png，后一个为 float16。
    std::vector<ImageRequest> request = { 
        // png 格式
        ImageRequest("0", ImageType::Scene),
        // 无压缩 RGB 数组字节
        ImageRequest("1", ImageType::Scene, false, false),       
        // 浮点无压缩图像  
        ImageRequest("1", ImageType::DepthPlanar, true) 
    };

    const std::vector<ImageResponse>& response = client.simGetImages(request);
    // 对包含图像数据、姿态、时间戳等的响应进行处理
}
```

## 准备运行的完整示例

### Python

有关更完整的准备运行样例代码，请参见 [AirSimClient 项目中的示例代码](https://github.com/Microsoft/AirSim/tree/main/PythonClient//multirotor/hello_drone.py) （用于多旋翼）或 [HelloCar 示例](https://github.com/Microsoft/AirSim/tree/main/PythonClient//car/hello_car.py)。该代码还演示了简单的活动，如将图像保存到文件或使用 `numpy` 处理图像。

### C++

有关更完整的准备运行样例代码，请参见 [HelloDrone 项目中的示例代码](https://github.com/Microsoft/AirSim/tree/main/HelloDrone//main.cpp) （用于多旋翼）或 [HelloCar 项目](https://github.com/Microsoft/AirSim/tree/main/HelloCar//main.cpp)。

另请参见 [其他示例代码](https://github.com/Microsoft/AirSim/tree/main/Examples/DataCollection/StereoImageGenerator.hpp)，该代码生成指定数量的立体图像以及真实深度和视差，并将其保存为 [pfm 格式](pfm.md)。

## 可用相机

这些是每个车辆中默认可用的相机。除了这些，您还可以通过 [设置](settings.md) 向车辆和未连接任何车辆的外部相机添加更多相机。

### 汽车
汽车上的相机可以通过 API 调用中的以下名称访问：`front_center`、`front_right`、`front_left`、`fpv` 和 `back_center`。这里，FPV 相机是汽车中驾驶员的头部位置。

### 多旋翼
无人机上的相机可以通过 API 调用中的以下名称访问：`front_center`、`front_right`、`front_left`、`bottom_center` 和 `back_center`。

### 计算机视觉模式
相机名称与多旋翼相同。

### 相机名称的向后兼容性
在 AirSim v1.2 之前，使用 ID 编号而不是名称访问相机。为了向后兼容，您仍可以使用上述相机名称的 ID 编号，顺序如下：`"0"`、`"1"`、`"2"`、`"3"`、`"4"`。此外，相机名称 `""` 也可用来访问通常是相机 `"0"` 的默认相机。

## "计算机视觉" 模式

您可以在所谓的 "计算机视觉" 模式下使用 AirSim。在此模式下，物理引擎被禁用，且没有车辆，只有相机（如果您想使用车辆但没有其运动学，可以使用带有物理引擎的多旋翼模式 [ExternalPhysicsEngine](settings.md##physicsenginename)）。您可以使用键盘移动（使用 F1 查看键位帮助）。您可以按录制按钮持续生成图像。或者，您可以调用 APIs 移动相机并拍摄图像。

要激活此模式，请编辑您可以在 `Documents\AirSim` 文件夹中找到的 [settings.json](settings.md)（或者在 Linux 上为 `~/Documents/AirSim`），确保以下值在根级别存在：

```json
{
  "SettingsVersion": 1.2,
  "SimMode": "ComputerVision"
}
```

[这里是移动相机并捕获图像的 Python 示例代码](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/cv_mode.py)。

此模式受到了 [UnrealCV 项目](http://unrealcv.org/) 的启发。

### 在计算机视觉模式下设置姿态
要使用 APIs 在环境中移动，可以使用 `simSetVehiclePose` API。该 API 接受位置和方向，并将其设置在不可见的车辆上，前中心相机位于该位置。其他所有相机均会保持相对位置一起移动。如果您不想更改位置（或方向），则只需将位置（或方向）的组件设置为浮点 NaN 值。`simGetVehiclePose` 允许检索当前姿态。您还可以使用 `simGetGroundTruthKinematics` 获取运动的运动学量。许多其他与车辆无关的 API 也可用，例如分割 API、碰撞 API 和相机 API。

## 相机 API
`simGetCameraInfo` 返回指定相机的姿态（在世界坐标系中，NED 坐标，SI 单位）和视场（以度为单位）。请参见 [示例用法](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/cv_mode.py)。

`simSetCameraPose` 设置指定相机的姿态，同时将输入姿态作为相对位置和 NED 坐标系中的四元数的组合。便捷的 `airsim.to_quaternion()` 函数允许将俯仰、滚转、偏航转换为四元数。例如，要将相机-0 设置为 15 度的俯仰，同时保持相同的位置，您可以使用：
```
camera_pose = airsim.Pose(airsim.Vector3r(0, 0, 0), airsim.to_quaternion(0.261799, 0, 0))  # PRY 以弧度表示
client.simSetCameraPose(0, camera_pose);
```

- `simSetCameraFov` 允许在运行时更改相机的视场。
- `simSetDistortionParams`、`simGetDistortionParams` 允许设置和获取失真参数 K1、K2、K3、P1、P2。

所有相机 API 除了特定于 API 的参数外，还接受 3 个通用参数：`camera_name`（字符串）、`vehicle_name`（字符串）和 `external`（布尔值，指示 [外部相机](settings.md#external-cameras)）。相机和车辆名称用于获取特定相机，如果 `external` 设置为 `true`，则车辆名称将被忽略。有关这些 API 的示例用法，请参见 [external_camera.py](https://github.com/microsoft/AirSim/blob/main/PythonClient/computer_vision/external_camera.py)。

### 云台
您可以通过 [设置](settings.md#gimbal) 为任何相机设置俯仰、滚转或偏航的稳定性。

请参见 [示例用法](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/cv_mode.py)。

## 更改分辨率和相机参数
要更改分辨率、视场等，可以使用 [settings.json](settings.md)。例如，以下设置会在 settings.json 中添加参数以进行场景捕获，并使用上面描述的 "计算机视觉" 模式。如果您省略任何设置，则将使用以下默认值。有关更多信息，请参见 [设置文档](settings.md)。如果您使用立体摄像机，当前左右之间的距离固定为 25 厘米。

```json
{
  "SettingsVersion": 1.2,
  "CameraDefaults": {
      "CaptureSettings": [
        {
          "ImageType": 0,
          "Width": 256,
          "Height": 144,
          "FOV_Degrees": 90,
          "AutoExposureSpeed": 100,
          "MotionBlurAmount": 0
        }
    ]
  },
  "SimMode": "ComputerVision"
}
```

## 不同图像类型中像素值的含义是什么？
### 可用图像类型值
```cpp
  Scene = 0, 
  DepthPlanar = 1, 
  DepthPerspective = 2,
  DepthVis = 3, 
  DisparityNormalized = 4,
  Segmentation = 5,
  SurfaceNormals = 6,
  Infrared = 7,
  OpticalFlow = 8,
  OpticalFlowVis = 9
```                

### DepthPlanar 和 DepthPerspective
您通常希望以浮点形式检索深度图像（即，设置 `pixels_as_float = true`）并在 `ImageRequest` 中指定 `ImageType = DepthPlanar` 或 `ImageType = DepthPerspective`。对于 `ImageType = DepthPlanar`，您在相机平面中获得深度，即，与相机平行的所有点具有相同的深度。对于 `ImageType = DepthPerspective`，您使用命中该像素的投影光线从相机获取深度。根据您的用例，平面深度或透视深度可能是您想要的真实图像。例如，您可以将透视深度传输到 ROS 包，例如 `depth_image_proc` 以生成点云。或者平面深度可能更兼容于由立体算法（例如 SGM）生成的估计深度图像。

### DepthVis
当您在 `ImageRequest` 中指定 `ImageType = DepthVis` 时，您会获得一个有助于深度可视化的图像。在这种情况下，每个像素值根据相机平面中的深度（以米为单位）从黑色插值到白色。像素纯白表示深度为 100 米或更大，而纯黑表示深度为 0 米。

### DisparityNormalized
您通常希望以浮点形式检索视差图像（即，设置 `pixels_as_float = true` 并在 `ImageRequest` 中指定 `ImageType = DisparityNormalized`），在这种情况下，每个像素为 `(Xl - Xr)/Xmax`，因此被规范化为 0 到 1 之间的值。

### Segmentation
当您在 `ImageRequest` 中指定 `ImageType = Segmentation` 时，您会获得一个提供场景真实分割的图像。在启动时，AirSim 为环境中可用的每个网格分配值 0 到 255。该值随后映射到 [调色板](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Content/HUDAssets/seg_color_palette.png) 中的特定颜色。每个对象 ID 的 RGB 值可以在 [此文件](seg_rgbs.txt)中找到。

您可以使用 API 为特定网格分配特定值（限于 0-255 范围）。例如，以下 Python 代码将块环境中名为 "Ground" 的网格的对象 ID 设置为 20，从而改变其在分割视图中的颜色：

```python
success = client.simSetSegmentationObjectID("Ground", 20);
```

返回值为布尔类型，让您知道是否找到该网格。

请注意，典型的 Unreal 环境（如 Blocks）通常有许多其他网格组成同一对象，例如 "Ground_2"、"Ground_3" 等。因为为所有这些网格设置对象 ID 是繁琐的，AirSim 还支持正则表达式。例如，以下代码将所有名称以 "ground" 开头（不区分大小写）的网格设置为 21，只需一行代码：

```python
success = client.simSetSegmentationObjectID("ground[\w]*", 21, True);
```

返回值为 true 表示使用正则表达式匹配至少找到了一个网格。

建议您使用此 API 请求无压缩图像，以确保获得分割图像的精确 RGB 值：
```python
responses = client.simGetImages([ImageRequest(0, AirSimImageType.Segmentation, False, False)])
img1d = np.fromstring(response.image_data_uint8, dtype=np.uint8) #获取 numpy 数组
img_rgb = img1d.reshape(response.height, response.width, 3) #将数组重塑为 3 通道图像数组 H X W X 3
img_rgb = np.flipud(img_rgb) #原始图像是垂直翻转的

# 查找唯一颜色
print(np.unique(img_rgb[:,:,0], return_counts=True)) #红色
print(np.unique(img_rgb[:,:,1], return_counts=True)) #绿色
print(np.unique(img_rgb[:,:,2], return_counts=True)) #蓝色  
```

可以在 [segmentation.py](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/segmentation.py) 中找到完整的准备运行示例。

#### 取消设置对象 ID
可以将对象的 ID 设置为 -1，以使其在分割图像中不显示。

#### 如何找到网格名称？
要获得所需的真实分割，您需要知道 Unreal 环境中网格的名称。为此，您需要在 Unreal 编辑器中打开 Unreal 环境，然后使用世界大纲检查您感兴趣的网格的名称。例如，以下是 Blocks 环境中右侧面板中地面的网格名称：

![record screenshot](images/unreal_editor_blocks.png)

如果您不知道如何在 Unreal 编辑器中打开 Unreal 环境，请尝试按照 [从源构建](build_windows.md) 的指南进行操作。

一旦您确定感兴趣的网格，请记下它们的名称并使用上述 API 设置其对象 ID。可以通过 [一些设置](settings.md#segmentation-settings) 更改对象 ID 生成行为。

#### 更改对象 ID 的颜色
目前每个对象 ID 的颜色是固定的，如 [此调色板](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Content/HUDAssets/seg_color_palette.png) 所示。我们将很快添加更改对象 ID 颜色为所需值的功能。在此期间，您可以在您最喜欢的图像编辑器中打开分割图像，并获取您感兴趣的 RGB 值。

#### 启动对象 ID
在开始时，AirSim 为在环境中找到的每个类型为 `UStaticMeshComponent` 或 `ALandscapeProxy` 的对象分配对象 ID。然后，它使用网格名称或所有者名称（取决于设置），将其小写，删除 ASCII 97 以下的任何字符以去除数字和某些标点符号，求和所有字符的整数值并取模 255 来生成对象 ID。换句话说，所有具有相同字母字符的对象将获得相同对象 ID。这一启发式方法对许多 Unreal 环境有效，但可能不是您所需的。在这种情况下，请使用上述 APIs 将对象 ID 更改为您所需的值。有 [一些设置](settings.md#segmentation-settings) 可用于更改此行为。

#### 获取网格的对象 ID
`simGetSegmentationObjectID` API 允许您获取给定网格名称的对象 ID。

### 红外
当前这只是从对象 ID 到灰度 0-255 的映射。因此，任何对象 ID 为 42 的网格将显示为颜色 (42, 42, 42)。有关如何设置对象 ID 的更多详细信息，请参见 [分割部分](#segmentation)。通常可以对该图像类型应用噪声设置，以获得稍微更加真实的效果。我们仍在努力添加其他红外伪影，欢迎任何贡献。

### 光流和光流可视化
这些图像类型返回相机视角下感知运动的信息。光流返回一个 2 通道图像，通道对应于 vx 和 vy。光流可视化与光流相似，但将流数据转换为 RGB 以获得更“可视化”的输出。

## 示例代码
设置车辆在随机位置和方向的完整示例，以及拍摄图像的代码可以在 [GenerateImageGenerator.hpp](https://github.com/Microsoft/AirSim/tree/main/Examples/DataCollection/StereoImageGenerator.hpp) 中找到。该示例生成指定数量的立体图像和真实的视差图像，并将其保存为 [pfm 格式](pfm.md)。