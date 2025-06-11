# 开发工作流程

以下是使用 AirSim 进行不同开发活动的指南。如果您对基于 Unreal Engine 的项目不熟悉，想要为 AirSim 贡献代码或根据自己的需求制作分支，这可能会为您节省一些时间。

## 开发环境
### 操作系统
我们强烈推荐使用 Windows 10 和 Visual Studio 2019 作为开发环境。其他操作系统和 IDE 的支持在 Unreal Engine 方面不够成熟，您在尝试解决问题和适应环境时可能会严重影响生产力。

### 硬件
我们推荐使用 NVidia 1080 或 NVidia Titan 系列的 GPU 和搭载 64GB RAM、6 个以上核心、SSD 和 2-3 个显示器（理想情况下是 4K）的强大台式机。我们发现 HP Z840 非常符合我们的需求。高端笔记本的开发体验通常不如强大的台式机，但在紧急情况下可能会有所帮助。您通常需要配备离散 NVidia GPU（至少是 M2000 或更好）的笔记本，64GB RAM，SSD，并且希望有 4K 显示器。我们发现联想 P50 型号非常符合我们的需求。仅配备集成显卡的笔记本可能表现不佳。

## 更新和更改 AirSim 代码

### 概述
AirSim 被设计为插件。这意味着它不能单独运行，您需要将其放入 Unreal 项目中（我们称之为“环境”）。因此，构建和测试 AirSim 有两个步骤：（1）构建插件，（2）在 Unreal 项目中部署插件并运行项目。

第一步可以通过位于 AirSim 根目录下的 build.cmd 完成。该命令将更新您在 `Unreal\Plugins` 文件夹中所需的所有内容。因此，要部署插件，您只需将 `Unreal\Plugins` 文件夹复制到您的 Unreal 项目文件夹中。接下来，您应该删除 Unreal 项目中的所有中间文件，然后重新生成 .sln 文件。为此，我们在 `Unreal\Environments\Blocks` 文件夹中有两个方便的 .bat 文件：`clean.bat` 和 `GenerateProjectFiles.bat`。所以只需从您的 Unreal 项目根目录依次运行这些 bat 文件。现在，您可以在 Visual Studio 中打开新的 .sln 并按 F5 运行它。

### 步骤
以下是我们用来更改 AirSim 并测试其步骤。使用 [Blocks project](unreal_blocks.md) 进行 AirSim 代码开发是最佳方法。这是一个轻量级项目，因此编译时间相对较快。通常工作流程如下：

```
REM //使用 VS 2019 的 x64 Native Tools 命令提示符
REM //导航到 AirSim 仓库文件夹

git pull                          
build.cmd                        
cd Unreal\Environments\Blocks         
update_from_git.bat
start Blocks.sln
```

上述命令首先构建 AirSim 插件，然后通过方便的 `update_from_git.bat` 将其部署到 Blocks 项目中。现在您可以在 Visual Studio 解决方案中工作，修改代码，只需按 F5 来构建、运行和测试您的更改。调试、断点等应照常工作。

完成代码更改后，您可能希望将更改推送回 AirSim 仓库或您自己的分支，或者将新插件部署到您自定义的 Unreal 项目中。为此，返回命令提示符，首先更新 AirSim 仓库文件夹：

```
REM //使用 VS 2019 的 x64 Native Tools 命令提示符
REM //从 Unreal\Environments\Blocks 运行

update_to_git.bat
build.cmd
```

上述命令将您的代码更改从 Unreal 项目文件夹传输回 `Unreal\Plugins` 文件夹。现在，您的更改准备好推送到 AirSim 仓库或您自己的分支。您还可以将 `Unreal\Plugins` 复制到您的自定义 Unreal 引擎项目中，查看一切是否在您的自定义项目中正常工作。

### 重点提示
一旦您理解了 Unreal Build 系统和插件模型是如何工作的，以及我们为什么要执行上述步骤，您应该会在遵循这个工作流程时感到相当舒服。不要害怕打开 .bat 文件查看其内容，它们非常简单明了（当然，build.cmd 除外 - 不要去看那个）。

## 常见问题解答

#### 我在 Blocks 项目中做了代码更改，但它不起作用。
当您在 Visual Studio 中按 F5 或 F6 开始构建时，Unreal Build 系统会启动，并尝试找出是否有任何文件需要构建。不幸的是，它通常无法识别未使用 Unreal 头文件和对象层次结构的脏文件。因此，诀窍是使某些文件被标记为脏文件，这是 Unreal Build 系统总能识别的。我最喜欢的文件是 AirSimGameMode.cpp。只需插入一行，删除它并保存文件。

#### 我在 Visual Studio 外部做了代码更改，但它不起作用。
不要那样做！Unreal Build 系统 *假设* 您正在使用 Visual Studio，并且与 Visual Studio 集成时做了一些工作。如果您坚持使用其他编辑器，那么请查阅如何在 Unreal 项目中进行命令行构建，或查看您编辑器的文档以了解如何与 Unreal Build 系统集成，或运行 `clean.bat` + `GenerateProjectFiles.bat` 以确保 VS 解决方案同步。

#### 我在 Unreal 项目中尝试添加新文件，但它不起作用。
不会！尽管您确实使用了 Visual Studio 解决方案，但请记住，该解决方案实际上是由 Unreal Build 系统生成的。如果您想在项目中添加新文件，首先要关闭 Visual Studio，在所需位置添加一个空文件，然后运行 `GenerateProjectFiles.bat`，它将扫描项目中的所有文件并重新创建 .sln 文件。现在打开这个新 .sln 文件，您就可以开始工作了。

#### 我复制了 Unreal\Plugins 文件夹，但在 Unreal 项目中没有任何效果。
首先确保您的项目 .uproject 文件引用了该插件。然后确保您已经运行了 `clean.bat` 和 `GenerateProjectFiles.bat`，如上文概述所述。

#### 我有多个包含 AirSim 插件的 Unreal 项目。如何轻松更新它们？
您运气很好！我们有一个 `build_all_ue_projects.bat` 文件，正好可以做到这一点。不要把它当作黑箱（至少现在不要），打开它看看它在做什么。它有 4 个从命令行参数设置的变量。如果没有提供这些参数，它们将在下一组语句中设置为默认值。您可能需要更改路径的默认值。这个批处理文件构建 AirSim 插件，将其部署到所有列出的项目（请参见之后的 CALL 语句），为这些项目运行打包，并将最终二进制文件放在指定的文件夹中——一次性完成！这就是我们创建自己二进制发布的方式。

#### 我该如何向 AirSim 贡献代码？
在进行任何更改之前，确保您创建了功能分支。在 Blocks 环境中测试代码更改后，请遵循 [常规步骤](https://akrabat.com/the-beginners-guide-to-contributing-to-a-github-project/) 进行贡献，就像其他 GitHub 项目那样。请使用 rebase 和 squash merge，更多信息请参见 [An introduction to Git merge and rebase: what they are, and how to use them](https://www.freecodecamp.org/news/an-introduction-to-git-merge-and-rebase-what-they-are-and-how-to-use-them-131b863785f/)。