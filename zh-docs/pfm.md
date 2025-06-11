# pfm 格式

Pfm（或可移植浮点图，Portable FloatMap）图像格式以浮点像素存储图像，因此不受通常的 0-255 像素值范围限制。这对于 HDR 图像或描述其他内容（如深度）的图像非常有用。

一种很好查看此文件格式的工具是 [PfmPad](https://sourceforge.net/projects/pfmpad/)。我们不推荐 Maverick photo viewer，因为它似乎无法正确显示深度图像。

AirSim 有代码可以为 [C++](https://github.com/Microsoft/AirSim/blob/main/AirLib/include/common/common_utils/Utils.hpp#L637) 写入 pfm 文件，并且可以为 [Python](https://github.com/Microsoft/AirSim/tree/main/PythonClient//airsim/utils.py#L122) 读取和写入 pfm 文件。