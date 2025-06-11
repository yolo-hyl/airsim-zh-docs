# 在 Linux 上安装 cmake

如果你的 cmake 版本不是 3.10（例如，Ubuntu 14 默认的是 3.2.2），你可以运行以下命令：

```

mkdir \~/cmake-3.10.2
cd \~/cmake-3.10.2
wget [https://cmake.org/files/v3.10/cmake-3.10.2-Linux-x86\_64.sh](https://cmake.org/files/v3.10/cmake-3.10.2-Linux-x86_64.sh)

```

现在你需要单独运行以下命令（它是交互式的）：
```

sh cmake-3.10.2-Linux-x86\_64.sh --prefix \~/cmake-3.10.2

```

在询问是否创建另一个 cmake-3.10.2-Linux-x86_64 文件夹时，回答 'n'，然后运行：

```

sudo update-alternatives --install /usr/bin/cmake cmake \~/cmake-3.10.2/bin/cmake 60

```

现在输入 `cmake --version` 来确认你的 cmake 版本是 3.10.2。