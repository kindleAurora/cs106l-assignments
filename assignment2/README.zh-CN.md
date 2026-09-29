本地 README 全文中文译文。对应[英文源文件](README.md)；独立的“译注”用于说明原文问题，不属于原题正文。

![Marriage Pact 标志](docs/marriage_pact.png)

# 作业 2：Marriage Pact（婚姻契约配对）

截止时间：4 月 24 日，星期五，晚上 11:59。

## 概述

欢迎来到作业 2！这是一份简短而轻松的练习，帮助你开始使用 STL 的容器和指针。

你需要关注以下文件：

- `main.cpp`：你的所有代码都写在这里 😀！
- `short_answer.txt`：简答题答案写在这里 📝！

要下载本作业的起始代码，请查看课程作业仓库中的[**入门说明**](../README.md#getting-started)。

> [!NOTE] 译注：原文的旧链接
> 上面的链接按原文保留，但当前仓库的根 README 没有 `Getting Started` 小节。环境设置请参阅[作业环境设置中文译文](../assignment-setup/README.zh-CN.md)。

## 运行代码

要运行代码，首先需要编译。打开终端；如果使用 VSCode，可以按 Ctrl + 反引号键，或选择顶部的 **Terminal > New Terminal（终端 > 新建终端）**。然后确保当前位于 `assignment2/` 目录，运行：

```sh
g++ -std=c++20 main.cpp -o main
```

假设代码顺利编译，没有出现编译错误，现在就可以执行：

```sh
./main
```

这会真正运行 `main.cpp` 中的 `main` 函数。

在按照下面的说明完成作业时，我们建议你不时编译，并使用自动评分器测试，确认自己正在朝正确方向前进！

> [!NOTE] Windows 用户注意
> 在 Windows 上，为了看到输出，你可能需要使用以下命令编译代码：
>
> ```sh
> g++ -static-libstdc++ -std=c++20 main.cpp -o main
> ```
>
> 此外，生成的可执行文件可能叫作 `main.exe`。这种情况下，请用以下命令运行代码：
>
> ```sh
> ./main.exe
> ```

## 第 0 部分：准备工作

欢迎参加 Marriage Pact！开始之前，我们需要知道你的姓名。请将 `main.cpp` 顶部的常量 `kYourName` 从 `"STUDENT TODO"` 改为你的全名，名与姓之间用空格分隔。

> [!NOTE] 译注：`kYourName` 的声明
> 原文将它称为“常量”，但当前起始代码将其声明为普通的 `std::string` 变量，没有使用 `const`。这里需要做的仍然是按题意修改它的初始值。

## 第 1 部分：获取所有报名者

你已经等了好几天，想知道今年 Marriage Pact 配对对象的姓名首字母，现在它们终于出现在你的收件箱里！今年活动增加了一条新规则：配对对象的姓名首字母**必须**与你自己的相同，才有资格成为你的配对对象。然而，即使你已经和朋友讨论了好几个小时，还是不知道对方可能是谁！校园里有数千名学生，你总不能靠人工逐个检查整份名单，列出所有潜在的灵魂伴侣。幸好你正在学习 CS106L，并且记得 C++ 有一种能够快速遍历一组相似信息的办法——容器！

我们提供了一个 `.txt` 文件，列出了今年报名参加 Marriage Pact 的所有学生，当然，他们都是虚构的。文件名为 `students.txt`，每行包含一位学生的名和姓。你首先要编写 `get_applicants` 函数：

> [!IMPORTANT] `get_applicants`
> 从 `.txt` 文件中解析所有姓名，并存入一个集合。名为 `filename` 的文件中，每一行都是一位报名者的姓名。实现时，你可以自由选择有序集合 `std::set` 或无序集合 `std::unordered_set`！如果选择无序集合，请修改相关的函数定义！

另外，请在 `short_answer.txt` 中回答下面的简答题：

> [!IMPORTANT] `short_answer.txt`
> **Q1：** 你可以自行选择有序集合或无序集合。请用几句话说明两者之间需要权衡的因素。另外，请给出一个有效的哈希函数示例，用于在无序集合中对学生姓名进行哈希；该示例不能是课堂上已经展示过的。

> [!NOTE]
> 本作业出现的所有姓名均属虚构。若与任何真实人物相似，无论其在世或已故，都纯属巧合。

## 第 2 部分：寻找匹配对象

你的侦探工作做得不错！现在你已经缩小了潜在灵魂伴侣的范围，该检验一下这份名单了。在参加了一整天的无伴奏合唱活动和咨询社团会议后，你回到宿舍，听室友说当晚 Main Quad 主广场要举办 Marriage Pact 配对联谊活动！找到真爱的最佳机会就在眼前——只要你能从极限飞盘训练中脱身。你迅速决定，要在联谊活动上逐一与所有姓名首字母和你相同的人交谈，于是开始编写一个函数，自动为你整理交谈顺序。

在这一部分，你将编写 `find_matches` 和 `get_match` 函数：

> [!IMPORTANT] `find_matches`
> 从上一部分生成的集合 `students` 中，找出所有姓名首字母与参数 `name` 相同的姓名，将指向这些姓名的指针放入一个新的 `std::queue` 中。
>
> - 如果你不清楚如何遍历集合，可以回顾[星期四关于迭代器与指针的课程](https://office365stanford-my.sharepoint.com/:p:/g/personal/jtrb_stanford_edu/EbOKUV784rBHrO3JIhUSAUgBvuIGn5rSU8h3xbq-Q1JFfQ?e=BlZwa7)。
> - 完成这一部分需要熟悉 `std::queue` 的操作。请查看[这里的 cppreference 文档](https://en.cppreference.com/w/cpp/container/queue)。
> - 提示：定义一个计算学生姓名首字母的辅助函数，可能会有帮助。随后就可以利用它，将 `name` 的首字母与 `students` 中每个姓名的首字母进行比较。

接下来，请实现 `get_match` 函数，找出你的“唯一真命配对对象”：

> [!IMPORTANT] `get_match`
> 从包含所有可能配对对象的队列中，选出你的“唯一真命配对对象”。具体规则由你决定；选择某种方法从队列中选出一位学生，最好比单独调用一次 `pop()` 多考虑一些，但不必特别复杂！可以考虑随机数或其他选择方式。
>
> 如果数据集中没有与你姓名首字母匹配的人，就打印 `“NO MATCHES FOUND.”`。祝你明年好运 😢

> [!NOTE] 译注：空队列时的返回值
> 原文写的是“打印”。当前 `main.cpp` 中 `get_match` 的接口返回 `std::string`，注释要求空队列时返回 `"NO MATCHES FOUND."`，辅助代码再使用这个结果。实际字符串不包含原文展示时使用的弯引号。

然后，在 `short_answer.txt` 中回答以下问题：

> [!IMPORTANT] `short_answer.txt`
> **Q2：** 注意，我们在队列中保存的是指向姓名的指针，而不是姓名本身。在这个问题中，为什么可能希望这样做？如果最初存放姓名的集合离开了作用域，之后又访问这些指针，会发生什么？

## 🚀 提交说明

提交作业时：

1. 请填写[这个链接中的反馈表](https://forms.gle/Zv27LwmtCPz88Kg46)。
2. 在 [Paperless](https://paperless.stanford.edu) 上提交作业！

需要提交的文件：

- `main.cpp`
- `short_answer.txt`

截止时间之前，你可以按需要多次重新提交。
