![环境设置页横幅](docs/header.png)

# 作业环境设置！

> 本地 README 全文中文译文。对照：[英文原文](README.md)。代码、命令、网址与原稿规定的输出保留；额外解释明确标为“译注”。

截止时间：4 月 17 日（星期五）晚上 11:59。

## 概述

欢迎来到 CS106L！这项任务会帮你配置本学季所需的环境，让后续作业的准备工作简单、顺畅。完成后，你应当能够在 VS Code 中编译并运行 C++ 文件，也能够运行自动评分器；接下来的每份作业都会用到这些操作！

如果配置过程中遇到问题，请在 [EdStem](https://edstem.org/us/courses/81492/discussion) 联系课程团队，或来参加答疑！

## 第 1 部分：安装 Python

### 1.1：检查是否已经安装 Python

CS106L 每份作业的自动评分器都使用 Python。你必须安装 **3.8 或更高版本**。在终端中运行以下命令，查看 Python 版本。

Linux 或 Mac：

```sh
python3 --version
```

Windows：

```sh
python --version
```

如果显示的版本不低于 `3.8`，就满足要求，**可以继续第 2 部分**。否则，请按 1.2 节安装 Python。

### 1.2：安装 Python（如果尚未安装）

#### Mac 与 Windows

请从[这里](https://www.python.org/downloads/)下载最新版本的 Python，并运行安装程序。**注意：Windows 用户必须在安装程序中勾选 `Add python.exe to PATH`（将 python.exe 加入 PATH）**。安装后，按 **1.1 节**的方法确认安装成功。

#### Linux

以下说明适用于 Debian 系发行版，例如 Ubuntu；已在 Ubuntu 20.04 LTS 上测试。

1. 更新 Ubuntu 软件包列表：

   ```sh
   sudo apt-get update
   ```

2. 安装 Python：

   ```sh
   sudo apt-get install python3 python3-venv
   ```

3. 重新打开终端，并运行以下命令确认安装成功：

   ```sh
   python3 --version
   ```

## 第 2 部分：配置 VS Code 与 C++ 编译器

本课程使用 VS Code 编写 C++ 代码。下面介绍如何在你的电脑上配置 VS Code 和 GCC 编译器。

### Mac

#### 第一步：安装 VS Code

打开[这个链接](https://code.visualstudio.com/docs/setup/mac)，下载 Mac 版 Visual Studio Code。按照网页中 **Installation（安装）** 一节的说明操作。

在 VS Code 中进入扩展面板，搜索 **C/C++**。点击 **C/C++** 扩展，再点击 **Install（安装）**。

![VS Code 扩展面板图标](docs/vscode-extensions.png)

最后，打开命令面板（`Cmd+Shift+P`），搜索并选择 `Shell Command: Install 'code' command in PATH`。这样就可以在终端中运行 `code` 命令来启动 VS Code。

**🥳 到这里，你应该已经在 Mac 上成功安装 VS Code 了 👏**

#### 第二步：安装 C++ 编译器

1. 运行以下命令，检查是否已经安装 Homebrew：

   ```sh
   brew --version
   ```

   如果看到类似以下内容，就跳到第 3 步。如果结果看起来不正常，则继续第 2 步。

   ```sh
   brew --version
   Homebrew 4.2.21
   ```

2. 运行下面的命令：

   ```sh
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

   它会下载 Homebrew 🍺，这是 Mac 的一个软件包管理器。太好了！

3. 运行：

   ```sh
   brew install gcc
   ```

   这会在电脑上安装 GCC 编译器。

4. 记下 Homebrew 安装的 GCC 版本。原说明称，多数情况下会是 `g++-14`。

   默认情况下，Mac 上的 `g++` 命令指向内置的 `clang` 编译器。原文建议运行以下命令，让 `g++` 指向刚安装的 GCC：

   ```sh
   echo 'export PATH="$(brew --prefix)/bin:$PATH"\nalias g++="g++-16"' >> ~/.zshrc
   ```

   将其中的 `g++-14` 替换为实际安装的 GCC 版本。

5. 重新打开终端，运行以下命令，确认配置生效：

   ```sh
   g++ --version
   ```

> 译注：`g++-14` 是原稿中的版本示例，并非对当前安装版本的判断。上面的 `echo` 命令按原样保留；其中字面量 `\n` 是否变为换行取决于 shell 的 `echo` 行为。它会修改 `~/.zshrc`，本次翻译没有执行这些命令。

> [!note]
> 如果使用 VS Code 运行代码，执行最后一个命令时可能遇到问题。**请确保 VS Code 内的终端使用 `zsh`**，如图所示。
>
> ![如何在 VS Code 中切换到 zsh](docs/mac-zsh.png)
>
> 在本课程中运行 `g++` 时，需要使用这样的终端。另一种办法是把 VS Code 的默认终端设置为 `zsh`：按 `Cmd+Shift+P`，选择 `Terminal: Select Default Profile`，然后选择 `zsh`。

### Windows

#### 第一步：安装 VS Code

打开[这个链接](https://code.visualstudio.com/docs/setup/windows)，下载 Windows 版 Visual Studio Code。按照网页中 **Installation（安装）** 一节操作。

在 VS Code 中进入扩展面板，搜索 **C/C++**。点击 **C/C++** 扩展，再点击 **Install（安装）**。

![VS Code 扩展面板图标](docs/vscode-extensions.png)

**🥳 到这里，你应该已经在电脑上成功安装 VS Code 了 👏**

#### 第二步：安装 C++ 编译器

1. 按照[这个链接](https://code.visualstudio.com/docs/cpp/config-mingw)中 **Installing the MinGW-w64 toolchain（安装 MinGW-w64 工具链）** 一节操作。
2. 完成这一节的全部步骤后，运行以下命令确认安装成功：

   ```sh
   g++ --version
   ```

### Linux

以下说明适用于 Debian 系发行版，例如 Ubuntu；已在 Ubuntu 20.04 LTS 上测试。

#### 第一步：安装 VS Code

打开[这个链接](https://code.visualstudio.com/docs/setup/linux)，下载 Linux 版 Visual Studio Code。按照网页中 **Installation（安装）** 一节操作。

在 VS Code 中进入扩展面板，搜索 **C/C++**。点击 **C/C++** 扩展，再点击 **Install（安装）**。

![VS Code 扩展面板图标](docs/vscode-extensions.png)

最后，打开命令面板（`Ctrl+Shift+P`），搜索并选择 `Shell Command: Install 'code' command in PATH`。这样就可以在终端中运行 `code` 命令启动 VS Code。

**🥳 到这里，你应该已经在 Linux 机器上成功安装 VS Code 了 👏**

#### 第二步：安装 C++ 编译器

1. 在终端中更新 Ubuntu 软件包列表：

   ```sh
   sudo apt-get update
   ```

2. 安装 `g++` 编译器：

   ```sh
   sudo apt-get install g++-10
   ```

3. 默认会使用系统版本的 `g++`。如果要改用刚安装的版本，可以按下面的方式，将 Linux 配置为使用 G++ 10 或已经安装的更高版本：

   ```sh
   sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-10 10
   ```

4. 重新打开终端，确认 GCC 安装成功。原文要求 `g++` 版本为 10 或更高：

   ```sh
   g++ --version
   ```

> 译注：以上是原文保留的安装示例。后续 A6 使用 C++23 的 optional 单子操作，不能仅凭“版本不低于 10”就认定编译器及标准库支持所有后续作业。

## 第 3 部分：通过 Git 克隆课程代码！

Git 是一种流行的版本控制系统（VCS），课程使用它分发起始代码。先运行下面的命令，确认安装了 Git：

```sh
git --version
```

如果结果不正常，请从[这个页面](https://git-scm.com/downloads)下载并安装 Git！

### 下载起始代码

打开 VS Code，再打开终端（按 `Ctrl` 加反引号键，或者选择顶部菜单 **Terminal > New Terminal**），运行：

```sh
git clone https://github.com/cs106l/cs106l-assignments.git
```

它会把起始代码下载到 `cs106l-assignments` 文件夹中。

### 打开 VS Code 工作区

做这门课的作业时，建议为当前作业文件夹单独打开一个 VS Code 工作区。假如你已经有 `cs106l-assignments` 文件夹，先使用 `cd`（切换目录）进入相应目录：

```sh
cd cs106l-assignments/assignment0
```

这会将工作目录切换到 `assignment0`。随后打开以此文件夹为工作区的 VS Code：

```sh
code .
```

现在就准备好了！

> 译注：原文这里使用旧目录名 `assignment0`。当前本地实际目录为 `assignment-setup`，仓库保存位置则是上一级的 `assignments`；原命令没有针对本地路径自动改写。

### 获取作业更新

课程更新现有作业或发布新作业时，会把更新推送到这个仓库。要获取新内容，在终端进入 `cs106l-assignments` 目录，运行：

```sh
git pull origin main
```

这样就能获得最新的起始代码！

# 第 4 部分：测试环境设置！

现在编译第一个 C++ 文件，并运行自动评分器。运行 C++ 代码之前，首先需要编译。打开 VS Code 终端（按 `Ctrl` 加反引号键，或选择顶部菜单 **Terminal > New Terminal**），确认位于 `assignment0/` 目录，然后运行：

```sh
g++ -std=c++23 main.cpp -o main
```

这个命令会**编译** C++ 文件 `main.cpp`，生成名为 `main` 的可执行文件，其中包含处理器可以执行的机器码。如果编译没有报错，就可以运行：

```sh
./main
```

它会执行 `main.cpp` 中的 `main` 函数：先运行程序，然后启动自动评分器，检查安装是否正确。

> 译注：本节的 `assignment0/` 同样是旧名称，应进入当前实际存在的 `assignment-setup/` 目录。

> [!note] Windows 注意事项
> Windows 上可能需要使用下面的编译命令，才能看到输出：
>
> ```sh
> g++ -static-libstdc++ -std=c++20 main.cpp -o main
> ```
>
> 此外，可执行文件可能名为 `main.exe`，此时运行：
>
> ```sh
> ./main.exe
> ```

> [!note] Mac 注意事项
> 编译时可能因为缺少 `wchar.h` 或类似文件而报错。如果出现这种情况，原文建议使用以下命令重新安装 Xcode 命令行工具：
>
> ```sh
> sudo rm -rf /Library/Developer/CommandLineTools
> sudo xcode-select --install
> ```
>
> 完成后，应该就能正常编译。

> 译注：上面两行按原文保留。第一行会删除已有的命令行工具目录；它不是对你当前电脑的诊断结果，本次没有执行。

# 🚀 完成之后……

如果编译并运行后，自动评分器的显示如下图所示：

![自动评分器全部测试通过的终端示例](docs/autograder.png)

就表示完成了作业环境设置！太好了。现在可以继续 [作业 1](../assignment1/README.zh-CN.md)！

> 译注：原文末尾的 `assignment1/README.md` 相对路径不能从设置目录直接定位到 A1；这里的导航链接已指向本地 A1 全文译文。
