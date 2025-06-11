# 现代 C++ 编码指南

我们使用现代 C++11。智能指针、Lambda 表达式和 C++11 的多线程原语是你的好朋友。

## 快速提示

“标准”的伟大之处在于有很多可供选择：[ISO](https://isocpp.org/wiki/faq/coding-standards)、[Sutter & Stroustrup](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md)、[ROS](http://wiki.ros.org/CppStyleGuide)、[LINUX](https://www.kernel.org/doc/Documentation/process/coding-style.rst)、[Google](https://google.github.io/styleguide/cppguide.html)、[Microsoft](https://msdn.microsoft.com/en-us/library/888a6zcz.aspx)、[CERN](http://atlas-computing.web.cern.ch/atlas-computing/projects/qa/draft_guidelines.html)、[GCC](https://gcc.gnu.org/wiki/CppConventions)、[ARM](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0475c/CJAJAJCJ.html)、[LLVM](http://llvm.org/docs/CodingStandards.html) 以及可能还有数千个其他标准。不幸的是，这些标准之间甚至在如何命名类或常量这样的基本问题上也不能达成一致。这可能是因为这些标准通常带有许多遗留问题，因为它们需要支持现有的代码库。本文件的意图是创建一种指导方针，尽可能接近 ISO、Sutter & Stroustrup 和 ROS，同时解决它们之间的尽可能多的冲突、不利因素和不一致性。

## clang-format

C++ 语法的格式化由 clang-format 工具标准化，其设置已检查到本项目的 `.clang-format` 文件中。这些设置旨在与以下列出的格式化指南相匹配。你可以使用 clang-format 命令行格式化文件，或者通过启用 Visual Studio 的自动 clang 格式化功能，无论是在每次编辑时还是在保存文件时都这样做。所有文件都是这样格式化的，名为 `clang-format` 的 GitHub 工作流也会确保所有的拉取请求都是正确格式化的，因此代码应该保持整洁。显然，这不包括像 `Eigen` 或 `rpclib` 的外部代码。

如果你发现 clang-format 中的错误，你可以通过使用以下注释对代码块禁用 clang 格式化：

```c++
// clang-format off
...
// clang-format on
```

## 命名约定

避免在名称中使用任何类型的匈牙利命名法和 "_ptr" 后缀在指针上。

| **代码元素** | **风格** | **备注** |
| --- | --- | --- |
| 命名空间 | under_scored | 与类名区分 |
| 类名 | CamelCase | 与 ISO 推荐的 STL 类型区分（不要使用 "C" 或 "T" 前缀） |
| 函数名 | camelCase | 小写开头几乎是通用的，除了 .Net 世界 |
| 参数/局部变量 | under_scored | 绝大多数标准建议使用，因为 _ 对 C++ 用户更具可读性（尽管对 Java/.Net 用户而言不然） |
| 成员变量 | under_scored_with_ | 前缀 _ 被强烈不推荐，因为 ISO 有关于保留 _标识符的规则，因此我们建议使用后缀 |
| 枚举及其成员 | CamelCase | 大多数标准（除了非常老的标准）对此一致认同 |
| 全局变量 | g_under_scored | 你根本不应该有这些！ |
| 常量 | UPPER_CASE | 有争议，我们在此必须选择其中一种，除非是类或方法中的私有常量，则使用成员或局部变量的命名 |
| 文件名 | 与文件中的类名大小写匹配 | 各有利弊，但这样避免了自动生成代码的一致性问题（对 ROS 很重要） |

## 头文件

使用命名空间限定的 #ifdef 以防止多重包含：

```
#ifndef msr_airsim_MyHeader_hpp
#define msr_airsim_MyHeader_hpp

//--你的代码

#endif
```

我们不使用 #pragma once 的原因是，如果同一头文件存在于多个地方（在 ROS 构建系统下可能会发生这种情况），它将不受支持！

## 括号使用

函数或方法体内的大括号放在同一行。在命名空间、类和方法级别则使用单独一行。这被称为 [K&R 风格](https://en.wikipedia.org/wiki/Indent_style#K.26R_style)，其变体在 C++ 中广泛使用，而其他风格在其他语言中更为流行。请注意，如果你只有单个语句，则不需要花括号，但复杂语句在使用大括号时更容易保持正确。

```
int main(int argc, char* argv[])
{
     while (x == y) {
        f0();
        if (cont()) {
            f1();
        } else {
            f2();
            f3();
        }
        if (x > 100)
            break;
    }
}
```

## 常量和引用

严格审查你声明的所有非标量参数是否可以成为常量和引用。如果你来自 C#/Java/Python 等语言，你最常犯的错误就是按值传递参数，而不是 `const T&`；尤其是大多数字符串、向量和映射，你希望将其作为 `const T&`（如果它们是只读的）或 `T&`（如果它们是可写的）传递。此外，尽可能地向方法添加 `const` 后缀。

## 重写
重写虚拟方法时，使用 override 后缀。

## 指针

这实际上是关于内存管理的问题。模拟器有很多性能关键的代码，因此我们试图避免通过大量的 new/delete 调用来过载内存管理器。我们还希望避免在堆栈上复制过多的东西，因此尽可能通过引用传递事物。但是当对象确实需要存活超过调用栈时，通常需要在堆上分配该对象，因此你将有一个指针。现在，如果管理该对象的生命周期将变得棘手，我们建议使用 [C++ 11 智能指针](https://cppstyle.wordpress.com/c11-smart-pointers/)。但是智能指针确实有成本，所以不要到处盲目使用它们。对于性能至关重要的私有代码，可以使用原始指针。当与仅接受指针类型的遗留系统（例如，套接字 API）接口时，通常也需要使用原始指针。但我们尽量将这些遗留接口封装起来，避免这种编程风格渗透到更大的代码库中。

严格检查是否可以在任何地方使用 const，例如，`const float * const xP`。避免使用前缀或后缀来指示变量名中的指针类型，即使用 `my_obj` 而不是 `myobj_ptr`，除非在可能使变量区分更清楚的情况下，例如，`int mynum = 5; int* mynum_ptr = mynum;`

## 空值检查

在 Unreal C++ 代码中，当检查指针是否为空时，最好使用 `IsValid(ptr)`。除了检查指针是否为空，该函数还将返回一个 UObject 是否已正确初始化。这在一个 UObject 正在被垃圾回收但仍设置为非空值的情况下非常有用。

## 缩进

C++ 代码库使用四个空格进行缩进（不是制表符）。

## 换行

文件应使用 Unix 换行符提交。当在 Windows 上工作时，git 可以配置为以 Windows 换行符检出文件，并在提交时自动从 Windows 转换为 Unix 换行符，可以通过运行以下命令实现：

```
git config --global core.autocrlf true
```

在 Linux 上工作时，最好通过运行以下命令将 git 配置为以 Unix 换行符检出文件：

```
git config --global core.autocrlf input
```

有关此设置的更多详细信息，请参阅 [此文档](https://docs.github.com/en/get-started/getting-started-with-git/configuring-git-to-handle-line-endings)。

## 这太简短了，是吗？

是的，这故意而为，因为没有人喜欢阅读 200 页的编码指南。这里的目标是仅覆盖大部分重要的内容，而这些内容在 [GCC 的严格模式编译](http://shitalshah.com/p/how-to-enable-and-use-gcc-strict-mode-compilation/) 和 VC++ 中的 4 级错误警告中通常没有涉及。如果你想了解更多关于如何在 C++ 中编写更好的代码，请参见 [GotW](https://herbsutter.com/gotw/) 和 [Effective Modern C++](http://shop.oreilly.com/product/0636920033707.do) 一书。