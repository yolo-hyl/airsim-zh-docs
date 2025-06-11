# 升级到 Unreal Engine 4.27

这些说明适用于已经在 Unreal Engine 4.25 上使用 AirSim 的用户。如果您从未安装过 AirSim，请参见 [如何获取](https://github.com/microsoft/airsim#how-to-get-it)。

**注意：** 以下步骤将删除您在 AirSim 或 Unreal 文件夹中未保存的任何工作。

## 首先执行以下操作

### 对于 Windows 用户
1. 安装 Visual Studio 2022，包括 VC++、Python 和 C#。
2. 通过 Epic Games Launcher 安装 UE 4.27。
3. 启动 `x64 Native Tools Command Prompt for VS 2022` 并导航到 AirSim 仓库。
4. 运行 `clean_rebuild.bat`，以删除所有未选中/额外的内容并重建一切。
5. 有关更多信息，请参见 [在 Windows 上构建 AirSim](build_windows.md)。

### 对于 Linux 用户
1. 从您的 AirSim 仓库文件夹，运行 `clean_rebuild.sh`。
2. 重命名或删除您现有的 Unreal Engine 文件夹。
3. 按照步骤 1 和 2 [安装 Unreal Engine 4.27](build_linux.md)。
4. 有关更多信息，请参见 [在 Linux 上构建 AirSim](build_linux.md)。

## 升级您的自定义 Unreal 项目
如果您在旧版本的 Unreal Engine 中创建了自己的 Unreal 项目，则需要将项目升级到 Unreal 4.27。为此，

1. 打开 .uproject 文件，查找 `"EngineAssociation"` 行，确保其内容为 `"EngineAssociation": "4.27"`。
2. 删除您 Unreal 项目文件夹中的 `Plugins/AirSim` 文件夹。
3. 转到您的 AirSim 仓库文件夹，将 `Unreal\Plugins` 文件夹复制到您的 Unreal 项目文件夹中。
4. 从 `Unreal\Environments\Blocks` 中复制 *.bat（或对于 Linux 使用 *.sh）到您的项目文件夹。
5. 运行 `clean.bat`（或对于 Linux 使用 `clean.sh`），然后运行 `GenerateProjectFiles.bat`（仅适用于 Windows）。

## 常见问题解答

### 我有一个早于 4.16 的 Unreal 项目。我该如何升级？

#### 选项 1：重新创建项目
如果您的项目除了您下载的环境外没有任何代码或资产，那么您可以简单地 [在 Unreal 4.27 编辑器中重新创建项目](unreal_custenv.md)，然后从 `AirSim/Unreal/Plugins` 文件夹复制 Plugins 文件夹。

#### 选项 2：修改几个文件
比 Unreal 4.15 更新的版本有破坏性更改。因此，您需要修改您在 Unreal 项目 `Source` 文件夹中找到的 *.Build.cs 和 *.Target.cs 文件。那是什么更改呢？以下是其要点，但您应该查看 [Unreal 官方 4.16 过渡帖子](https://forums.unrealengine.com/showthread.php?145757-C-4-16-Transition-Guide)。

##### 在您项目的 *.Target.cs 中
1. 将构造函数从 `public MyProjectTarget(TargetInfo Target)` 更改为 `public MyProjectTarget(TargetInfo Target) : base(Target)`。

2. 如果您有 `SetupBinaries` 方法，请删除它，并在上述构造函数中添加以下行：`ExtraModuleNames.AddRange(new string[] { "MyProject" });`。

##### 在您项目的 *.Build.cs 中
将构造函数从 `public MyProject(TargetInfo Target)` 更改为 `public MyProject(ReadOnlyTargetRules Target) : base(Target)`。

##### 最后...
按照上述步骤继续升级。警告框可能只显示“打开副本”按钮。不要点击那个。相反，点击更多选项，这将显示更多按钮。选择 `Convert-In-Place option`。*注意：* 始终先备份您的项目！如果您没有任何问题，就地转换应该会正常进行，您现在已经使用了新版 Unreal。