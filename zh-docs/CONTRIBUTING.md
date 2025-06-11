# 贡献指南

## 快速开始
- 请阅读我们的 [简短且清晰的编码指南](coding_guidelines.md)。
- 对于重大更改，例如添加新功能或重构，请 [先提一个问题](https://github.com/Microsoft/AirSim/issues)。
- 使用我们的 [推荐开发工作流程](dev_workflow.md) 进行更改和测试。
- 像其他 GitHub 项目一样，通过 [常规步骤](https://www.dataschool.io/how-to-contribute-on-github/) 进行贡献。如果您不熟悉 Git 的 Branch-Rebase-Merge 工作流程，请 [先阅读这个](https://git-rebase.io/)。

## 检查清单
- 使用与代码其余部分相同的风格和格式，即使这不是您偏好的风格。
- 任何与代码更改相关的文档都要进行相应修改。
- 不要包含特定于操作系统的头文件或新的第三方依赖。
- 保持您的 Pull Request 小，理想情况下不超过 10 个文件。
- 确保您没有包含大型二进制文件。
- 当添加新的包含文件时，请确保依赖是绝对必要的。
- 定期将您的分支与主分支进行变基（理想情况下每 2-3 天一次）。
- 确保您的代码可以在 Windows、Linux 和 OSX 上编译。