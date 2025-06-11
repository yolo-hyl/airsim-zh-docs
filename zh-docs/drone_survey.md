# 实现无人机测量脚本

从 [https://github.com/microsoft/AirSim/wiki/Implementing-a-Drone-Survey-script](https://github.com/microsoft/AirSim/wiki/Implementing-a-Drone-Survey-script) 移动到这里

你是否曾想过捕捉某个地点的顶视图照片？好消息是，Python API 让这变得非常简单。请查看 [这里的代码](https://github.com/microsoft/AirSim/blob/main/PythonClient/multirotor/survey.py)。

![survey](images/survey.png)

假设我们想要以下变量：

| 变量        | 描述                         |
| ----------- | ---------------------------- |
| boxsize     | 测量的正方形区域的整体大小   |
| stripewidth | 选择游泳道之间的间距，这可能取决于相机镜头的类型，例如。 |
| altitude    | 测量时的飞行高度           |
| speed       | 测量的速度可能依赖于相机的拍摄速度。 |

有了这些变量，我们可以使用以下代码计算一个正方形路径：

```python
        path = []
        distance = 0
        while x < self.boxsize:
            distance += self.boxsize
            path.append(Vector3r(x, self.boxsize, z))
            x += self.stripewidth
            distance += self.stripewidth
            path.append(Vector3r(x, self.boxsize, z))
            distance += self.boxsize
            path.append(Vector3r(x, -self.boxsize, z))
            x += self.stripewidth
            distance += self.stripewidth
            path.append(Vector3r(x, -self.boxsize, z))
            distance += self.boxsize
```

假设我们从正方形的一个角开始，按游泳道宽度增加 x，然后在完整的 y 维度上飞行 `-boxsize` 到 `+boxsize`，所以在这个例子中，`boxsize` 是我们将要覆盖的实际正方形的一半大小。

一旦我们有了这个 Vector3r 对象的列表，我们可以通过以下调用非常简单地飞行这个路径：
```python
result = self.client.moveOnPath(path, self.velocity, trip_time, DrivetrainType.ForwardOnly,
                                YawMode(False,0), lookahead, 1)
```

我们可以通过将路径的距离除以飞行速度来计算合适的 `trip_time` 超时。

在这里所需的 `lookahead` 可以通过 `self.velocity + (self.velocity/2)` 从速度计算得出。lookahead 越多，转弯越平滑。这就是为什么你在截图中看到每条游泳道的末端是平滑的转弯，而不是方形盒子模式。这也可能导致你相机视频的平滑性。

就这些，非常简单，对吧？

当然，你可以对此添加更多智能，让它避开地图上的已知障碍，使其上下爬升以便测量斜坡等等。非常有趣的体验。