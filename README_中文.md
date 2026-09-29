# CS106L 作业：中文导航

这里保存了官方作业仓库的本地副本：7 份作业、环境检查、起始代码、测试器源码、数据和说明图片。你可以在 Obsidian 阅读中文指南，在 VS Code 编写作业。

每份作业的英文 README 均配有独立的 `README.zh-CN.md` 全文中文译文，包含背景、任务要求、提示、示例及提交说明。代码、命令和规定的输出格式保留；原文问题单独标为“译注”。以前的简明任务指南也保留，便于快速回顾。

## 作业目录

| 作业 | 全文中文译文 | 简明指南 | 主要练习 | 官方编译标准 |
|---|---|---|---|---|
| 环境检查 | [环境设置](assignment-setup/README.zh-CN.md) | [快速准备](assignment-setup/开始前的准备_中文.md) | 编译器、Python 与测试器 | C++23 |
| A1 | [选课数据处理](assignment1/README.zh-CN.md) | [任务清单](assignment1/任务指南_中文.md) | 结构体、引用、流、CSV | C++20 |
| A2 | [人名匹配](assignment2/README.zh-CN.md) | [任务清单](assignment2/任务指南_中文.md) | 集合、队列、指针 | C++20 |
| A3 | [自己设计一个类](assignment3/README.zh-CN.md) | [任务清单](assignment3/任务指南_中文.md) | 构造函数、封装、接口 | C++20 |
| A4 | [拼写检查器](assignment4/README.zh-CN.md) | [任务清单](assignment4/任务指南_中文.md) | 算法、迭代器、ranges | C++20 |
| A5 | [用户资料类](assignment5/README.zh-CN.md) | [任务清单](assignment5/任务指南_中文.md) | 运算符重载、复制与析构 | C++20 |
| A6 | [课程查询](assignment6/README.zh-CN.md) | [任务清单](assignment6/任务指南_中文.md) | optional 及其组合操作 | C++23 |
| A7 | [实现 unique_ptr](assignment7/README.zh-CN.md) | [任务清单](assignment7/任务指南_中文.md) | RAII、模板、移动语义 | C++20 |

## 按你的进度开始

先完成第 3 讲“初始化与引用”、第 4 讲“流”，再尝试 A1。你现在写的二次方程函数，是熟悉类型、函数返回值和数据组织的练习。

做题时先读全文中文译文，再对照英文 README 和源码中的 TODO。简明指南可用作完成情况清单。保留题目规定的函数接口、文件名和输出格式，完成一部分后再运行该题测试器。

每份作业应从自己的目录编译、运行，避免数据文件的相对路径出错。A3、A4、A5 需要编译多个指定源文件，命令见各自指南；不要把整个课程目录的所有 `.cpp` 一起编译。

## 下载状态

- 官方原始文件已下载并保留；题目尚未填写答案。
- 本次没有运行作业测试器，也没有安装其运行依赖。首次运行可能联网创建 Python 环境、安装依赖；A3 还会下载辅助分析工具。
- 你此前配置的 C++17 适用于眼前基础练习；这些作业使用的标准见上表。准备开做时，请按该题调整编译任务。
- 课程主页把 A1 称为 SimpleEnroll；当前仓库 README 使用 EnrollmentNavigator，二者指向同一份 A1。

来源：[官方作业仓库](https://github.com/cs106l/cs106l-assignments)。下载版本及原始文件校验信息见 [下载清单](docs/assignment_download_manifest.json)。GitHub 的 `main` 分支可能继续更新，这份本地副本记录了下载时的版本。
