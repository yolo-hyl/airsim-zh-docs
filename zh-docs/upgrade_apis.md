# 升级 API 客户端代码
在 AirSim v1.2 中有多个 API 更改，我们希望这些更改能消除不一致性、增加未来扩展性并呈现更清晰的接口。然而，其中许多更改为 *破坏性更改*，这意味着您需要修改与 AirSim 通信的客户端代码。

## 更快速的方法
虽然您在客户端代码中需要进行的大多数更改相对简单，但更快捷的方法是查看示例代码，例如 [Hello Drone](https://github.com/Microsoft/AirSim/tree/main/PythonClient//multirotor/hello_drone.py) 或 [Hello Car](https://github.com/Microsoft/AirSim/tree/main/PythonClient//car/hello_car.py)，以了解更改的要点。

## 导入 AirSim
而不是，

```python
from AirSimClient import *
```
使用这个：

```python
import airsim
```

以上假设您已经使用以下命令安装了 AirSim 模块：
```
pip install --user airsim
```

如果您是从仓库的 PythonClient 文件夹运行代码，则也可以这样做：

```python
import setup_path 
import airsim
```

这里的 setup_path.py 应该存在于您的文件夹中，它将设置 `PythonClient` 仓库文件夹中 `airsim` 包的路径。PythonClient 文件夹中的所有示例都使用这种方法。

## 使用 AirSim 类
由于我们现在将所有内容放入包中，您需要像下面这样使用 AirSim 类的显式命名空间。

而不是，

```python
client1 = CarClient()
```

使用这个：

```python
client1 = airsim.CarClient()
```

## AirSim 类型

我们已将所有类型移至 `airsim` 命名空间。

而不是，

```python
image_type = AirSimImageType.DepthVis

d = DrivetrainType.MaxDegreeOfFreedom
```

使用这个：

```python
image_type = airsim.ImageType.DepthVis

d = airsim.DrivetrainType.MaxDegreeOfFreedom
```

## 获取图像

下面没有新内容，只是上面内容的组合。请注意，之前所有接受 `camera_id` 的 API，现在改为接受 `camera_name`。您可以查看 [可用摄像头](image_apis.md#avilable_cameras) 。

而不是，

```python
responses = client.simGetImages([ImageRequest(0, AirSimImageType.DepthVis)])
```

使用这个：

```python
responses = client.simGetImages([airsim.ImageRequest("0", airsim.ImageType.DepthVis)])
```

## 实用方法
在早期版本中，我们提供了多个作为 `AirSimClientBase` 一部分的实用方法。这些方法现已移至 `airsim` 命名空间，以提供更符合 Python 风格的接口。

而不是，

```python
AirSimClientBase.write_png(my_path, img_rgba) 

AirSimClientBase.wait_key('Press any key')
```

使用这个：

```python
airsim.write_png(my_path, img_rgba)

airsim.wait_key('Press any key')
```

## 相机名称
AirSim 现在使用 [名称](image_apis.md#available_cameras) 来引用相机，而不是索引号。然而，为了保持向后兼容，这些名称被与旧索引号作为字符串进行了别名。

而不是，

```python
client.simGetCameraInfo(0)
```

使用这个：

```python
client.simGetCameraInfo("0")

# 或

client.simGetCameraInfo("front-center")
```

## 异步方法
对于多旋翼飞行器，AirSim 具有多种方法，例如 `takeoff` 或 `moveByVelocityZ`，这些方法完成所需时间较长。现在所有此类方法都通过添加后缀 *Async* 进行重命名，如下所示。

而不是，

```python
client.takeoff()

client.moveToPosition(-10, 10, -10, 5)
```

使用这个：

```python
client.takeoffAsync().join()

client.moveToPositionAsync(-10, 10, -10, 5).join()
```

这里的 `.join()` 是 Python `Future` 类的调用，用于等待异步调用完成。您也可以选择在调用进行时进行其他计算。

## 仅限仿真方法
现在我们清楚区分了仅在仿真中可用的方法和在实际车辆上可能可用的方法。仅限仿真的方法以 `sim` 为前缀，如下所示。

```
getCollisionInfo()      改名为       simGetCollisionInfo()
getCameraInfo()         改名为       simGetCameraInfo()
setCameraOrientation()  改名为       simSetCameraOrientation()
```

## 状态信息
之前的 `CarState` 混合了诸如 `kinematics_true` 的仅仿真信息。今后，`CarState` 将仅包含在现实世界中可以获取的信息。

```python
k = car_state.kinematics_true
```

使用这个：

```python
k = car_state.kinematics_estimated

# 或

k = client.simGetGroundTruthKinematics()
```