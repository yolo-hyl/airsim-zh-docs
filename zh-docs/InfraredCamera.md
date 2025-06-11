这是一个使用 AirSim 和 AirSim Africa 环境生成模拟热红外 (IR) 图像的教程。

预编译的 Africa 环境可以从这个 GitHub 仓库的 Releases 标签下载：
[Windows 预编译二进制文件](https://github.com/Microsoft/AirSim/releases/tag/v1.2.1)

要生成您自己的数据，您可以使用两个 Python 文件：[create_ir_segmentation_map.py](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/create_ir_segmentation_map.py) 和 
[capture_ir_segmentation.py](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/capture_ir_segmentation.py)。

[create_ir_segmentation_map.py](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/create_ir_segmentation_map.py) 使用温度、发射率和相机响应信息来估算环境中物体可能的热数字计数，然后重新分配 AirSim 中的分割 ID，以匹配这些数字计数。它应在开始捕获热 IR 数据之前运行。否则，IR 图像中的数字计数将不正确。相机响应、温度和发射率数据都已包含在 Africa 环境中。

[capture_ir_segmentation.py](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/capture_ir_segmentation.py) 在重新分配分割 ID 后运行。它跟踪感兴趣的对象，并记录多旋翼的红外和场景图像。它使用计算机视觉模式。

最后，关于如何估计 Africa 环境中植物和动物的温度等详细信息可以在这篇论文中找到：

```plaintext
@inproceedings{bondi2018airsim,
  title={AirSim-W: A Simulation Environment for Wildlife Conservation with UAVs},
  author={Bondi, Elizabeth and Dey, Debadeepta and Kapoor, Ashish and Piavis, Jim and Shah, Shital and Fang, Fei and Dilkina, Bistra and Hannaford, Robert and Iyer, Arvind and Joppa, Lucas and others},
  booktitle={Proceedings of the 1st ACM SIGCAS Conference on Computing and Sustainable Societies},
  pages={40},
  year={2018},
  organization={ACM}
}
```