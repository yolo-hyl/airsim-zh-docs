# 如何在 AIRSIM 中访问网格

AirSim 支持访问构成场景的静态网格。

## 网格结构
每个网格由以下结构表示。
```cpp
struct MeshPositionVertexBuffersResponse {

	Vector3r position;
	Quaternionr orientation;

	std::vector<float> vertices;
	std::vector<uint32_t> indices;
	std::string name;
};
```

* 位置和方向采用 Unreal 坐标系。
* 网格本身是一个由顶点和索引表示的三角网格。
	* 三角网格类型通常称为 [Face-Vertex](https://en.wikipedia.org/wiki/Polygon_mesh#Face-vertex_meshes) 网格。这意味着每组三个索引代表构成三角形/面的顶点索引。
	* 顶点的 x、y、z 坐标都存储在一个单一的向量中。这意味着顶点向量是 Nx3，其中 N 是顶点的数量。
    * 顶点的位置是 Unreal 坐标系中的全局位置。这意味着它们已经通过位置和朝向进行了变换。

## 如何使用
获取场景中的网格的 API 非常简单。然而，应该注意到函数调用的开销很大，应该极少调用。一般来说，这是可以的，因为此函数仅访问静态网格，在大多数应用程序中，它们在程序运行期间不会发生变化。

请注意，您将需要使用第三方库或您自己的自定义代码来实际与接收到的网格进行交互。以下我利用 [libigl](https://github.com/libigl/libigl) 的 Python 绑定来可视化接收到的网格。

```python
import airsim

AIRSIM_HOST_IP='127.0.0.1'

client = airsim.VehicleClient(ip=AIRSIM_HOST_IP)
client.confirmConnection()

# 通过此函数接收返回的网格列表
meshes=client.simGetMeshPositionVertexBuffers()

index=0
for m in meshes:
    # 查找 Blocks 环境中的一个立方体网格
    if 'cube' in m.name:

        # 从这里开始的代码依赖于 libigl。Libigl 使用 pybind11 封装 C++ 代码。因此这里构建的 pyigl.so
        # 库与这个示例代码在同一目录中。
        # 这部分是您自己的网格库所需的类似代码
        from pyigl import *
        from iglhelpers import *

        # 将列表转换为 numpy 数组
        vertex_list=np.array(m.vertices,dtype=np.float32)
        indices=np.array(m.indices,dtype=np.uint32)

        num_vertices=int(len(vertex_list)/3)
        num_indices=len(indices)

        # Libigl 要求形状为 Nx3，其中 N 是顶点或索引的数量
        # 还要求实际类型为 double(float64) 以便于顶点，int64 以便于三角形/索引
        vertices_reshaped=vertex_list.reshape((num_vertices,3))
        indices_reshaped=indices.reshape((int(num_indices/3),3))
        vertices_reshaped=vertices_reshaped.astype(np.float64)
        indices_reshaped=indices_reshaped.astype(np.int64)

        # Libigl 函数转换为内部 Eigen 格式
        v_eig=p2e(vertices_reshaped)
        i_eig=p2e(indices_reshaped)

        # 查看网格
        viewer = igl.glfw.Viewer()
        viewer.data().set_mesh(v_eig,i_eig)
        viewer.launch()
        break
```