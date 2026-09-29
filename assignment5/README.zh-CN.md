> 本文件是用户提供的本地 [README.md](README.md) 的全文中文译文。保留原文代码、命令、输出和链接；原文中的问题另用“译注”说明。

![Treebook 的标志：一家虚构的、以斯坦福为基地的社交媒体创业公司](docs/logo.jpeg)

# 作业 5：Treebook

截止时间：5 月 15 日，星期五，晚上 11:59。

## 概述

斯坦福最新的社交媒体创业公司叫 Treebook，而你是创始团队的一员！为了让产品起步，并与一款来自哈佛、名字不便透露且在法律上与本公司完全无关的应用竞争，你被分配了实现用户资料的任务。

这次作业中，你将实现一个类的部分功能，使它支持运算符重载，同时修改一些特殊成员函数的行为。

你需要处理两个文件：

* `user.h`：包含 `User` 类的声明，你将为它补充特殊成员函数和运算符。
* `user.cpp`：包含 `User` 类的定义。

要下载本作业的起始代码，请参阅课程作业仓库中[开始使用（Getting Started）](../README.md#getting-started)的说明。

> [!NOTE] 译注：原文的旧链接
> 当前仓库根目录的 README 已不包含 `Getting Started` 小节。本地代码已经下载；环境准备请阅读[环境设置全文译文](../assignment-setup/README.zh-CN.md)。

## 运行代码

要运行代码，首先需要编译。打开终端：如果使用 VSCode，可以按 <kbd>Ctrl+`</kbd>，或者选择顶部的 **Terminal > New Terminal（终端 > 新建终端）**。接着，确保你位于 `assign5/` 目录，然后运行：

```sh
g++ -std=c++20 main.cpp user.cpp -o main
```

> [!NOTE] 译注：目录名
> 原文写的是 `assign5/`，本地仓库中的实际目录是 `assignment5/`。请进入实际目录执行命令。

如果编译过程没有出现错误，现在就可以执行：

```sh
./main
```

这会实际运行 `main.cpp` 中的 `main` 函数。

按照下方说明完成作业时，建议隔一段时间就编译一次，并使用自动评分器测试，以确认自己的方向正确。

> [!NOTE] Windows 用户注意
>
> 在 Windows 上，可能需要使用下面的命令编译，才能看到输出：
>
> ```sh
> g++ -static-libstdc++ -std=c++20 main.cpp user.cpp -o main
> ```
>
> 此外，生成的可执行文件可能叫 `main.exe`。这时应通过下面的命令运行：
>
> ```sh
> ./main.exe
> ```

## 第 1 部分：查看用户资料

看一下头文件 `user.h`。你的同事已经开始编写 `User` 类，用于保存加入社交平台的每位用户的姓名和好友列表。为了让这个类非常高效，他们选择使用一个原始指针来表示 `std::string` 数组形式的好友列表，这有点像 `std::vector` 在内部存储元素的方式。幸好，他们已经写好了创建新 `User`，以及向现有 `User` 的好友列表添加好友的逻辑（`add_friend`）；不过，在使用 `User` 对象时，他们开始遇到一些奇怪的问题。

首先，没有方便的方法把每个 `User` 对象的信息打印到控制台，这给 Treebook 的调试工作带来了困难。为了帮助同事，请编写 `operator<<`，将一个 `User` 打印到 `std::ostream`。**这个运算符应在 `user.h` 中声明为友元函数，并在 `user.cpp` 中实现。** 例如，一个名叫 `"Alice"`、好友为 `"Bob"` 和 `"Charlie"` 的用户，打印到控制台时应得到：

```text
User(name=Alice, friends=[Bob, Charlie])
```

注意：`operator<<` 不应打印任何换行符。

> [!IMPORTANT]
> 实现 `operator<<` 时，需要访问并遍历 `User` 类的私有字段 `_friends`，才能打印用户的好友。通常，非成员函数无法访问类中的私有字段；这里可以把 `operator<<` 在 **`User` 类内部声明为友元函数**，从而允许它访问。更多信息请参阅星期二的课程幻灯片。

## 第 2 部分：不太“友好”的行为

借助你写的 `operator<<`，同事们已经在开发社交应用方面取得了不错的进展。不过，他们还是不明白：尝试在内存中复制 `User` 对象时，为什么会出现一些看似古怪的问题。刚学过 CS106L 的你怀疑，这可能与 `User` 类的特殊成员函数有关，或者说，与它缺少这些函数有关。为了解决问题，我们将为 `User` 类实现自己的特殊成员函数（SMF），并删除另外一些编译器生成版本无法满足需要的函数。

具体来说，你需要：

1. 为 `User` 类实现析构函数，即实现 `~User()` 这个特殊成员函数。
2. 让 `User` 类可以进行复制构造，即实现 `User(const User& user)` 这个特殊成员函数。
3. 让 `User` 类可以进行复制赋值，即实现 `User& operator=(const User& user)` 这个特殊成员函数。
4. 禁止 `User` 类进行移动构造，即删除 `User(User&& user)` 这个特殊成员函数。
5. 禁止 `User` 类进行移动赋值，即删除 `User& operator=(User&& user)` 这个特殊成员函数。

完成这些任务时，你需要修改 `user.h` **和** `user.cpp` 两个文件。

> [!IMPORTANT]
> 实现上面的第 2、3 项时，需要复制 `_friends` 数组的内容。回顾星期四有关特殊成员函数的课程：可以先为新数组分配内存，必要时也可以在成员初始化列表中完成，然后通过 `for` 循环复制各个元素。
>
> 还要确保设置正在修改的那个实例的 `_size`、`_capacity` 和 `_name`！

## 第 3 部分：不断结交好友

修改特殊成员函数之后，你已经让 Treebook 在斯坦福推广开来，它的名声也开始传到其他大学！不过，你和同事发现，按照目前的类实现方式，`User` 类的一些常见用法不够方便，甚至根本无法实现。你认为，或许可以通过实现一些自定义运算符解决这些问题。

你将为 `User` 类重载两个运算符。**请把这两个运算符都实现为成员函数**，也就是在 `user.h` 中的 `User` 类内部声明，并在 `user.cpp` 中提供实现。

### `operator+=`

`+=` 运算符表示将一个用户添加到另一个用户的好友列表。这种关系应当是对称的：例如，把 Charlie 加入 Alice 的好友列表，也应当让 Alice 出现在 Charlie 的好友列表中。来看下面的代码：

```cpp
User alice("Alice");
User charlie("Charlie");

alice += charlie;
std::cout << alice << std::endl;
std::cout << charlie << std::endl;

// 预期输出：
// User(name=Alice, friends=[Charlie])
// User(name=Charlie, friends=[Alice])
```

这个运算符的函数签名应为 `User& operator+=(User& rhs)`。注意，它和复制赋值运算符一样，返回对自身的引用。

### `operator<`

回顾一下：要把用户存储在 `std::set` 中，需要 `<` 运算符，因为 `std::set` 的实现依赖比较运算符。请实现 `operator<`，按照姓名的字母顺序比较用户。例如：

```cpp
User alice("Alice");
User charlie("Charlie");

if (alice < charlie)
  std::cout << "Alice is less than Charlie";
else
  std::cout << "Charlie is less than Alice";

// 预期输出：
// Alice is less than Charlie
```

这个运算符的函数签名应为 `bool operator<(const User& rhs) const`。

> [!NOTE] 译注：`std::set` 的比较方式
> 这里说的是题目使用默认比较方式的情形。`std::set` 也可以指定自定义比较器，因此并非所有用法都必须定义 `operator<`。

## 🚀 提交说明

如果通过了所有测试，就可以提交了！提交作业的步骤如下：

1. 请填写[此链接中的反馈表](https://forms.gle/tfLJSKnuUbUx9Xdi6)。
2. 在 [Paperless](https://paperless.stanford.edu) 上提交作业！

需要交付的文件为：

- `user.h`
- `user.cpp`

截止时间之前，你可以不限次数地重新提交。
