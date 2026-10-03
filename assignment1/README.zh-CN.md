本地 README 全文中文译文。对应[英文源文件](README.md)；独立的“译注”用于说明原文问题，不属于原题正文。

# 作业 1：EnrollmentNavigator（选课导航）

## 配置 C++ 环境

请找到并按照[作业环境设置](../assignment-setup/README.zh-CN.md)中的说明，为本作业配置 C++ 编译器和自动评分器。

## 概述

又到了学季里的这个时候：该使用 EnrollmentNavigator 了 🤗，好耶！每个斯坦福学生在大学生活的某个时刻都会意识到，自己终究得毕业。因此，选课成了一项需要策略的活动：既要尽可能多地积累通向毕业的经验值（XP），又要保证每晚能睡上超过 4 个小时！

在这份希望不会花费你太久的作业中，我们将使用 ExploreCourses API 的示例数据，判断 ExploreCourses 上哪些计算机科学课程开课、哪些不开课！我们会运用流，同时练习 C++ 中的初始化与引用。开始吧 ʕ•́ᴥ•̀ʔっ

你只需要关注两个文件：

- `main.cpp`：你的所有代码都写在这里 😀！
- `utils.cpp`：包含一些辅助函数。你会使用这个文件中定义的函数，但除此之外，不需要修改它。

## 运行代码

要运行代码，首先需要编译。打开终端；如果使用 VSCode，可以按 Ctrl + 反引号键，或选择顶部的 **Terminal > New Terminal（终端 > 新建终端）**。然后确保当前位于 `assignment1/` 目录，运行：

```sh
g++ -std=c++20 main.cpp -o main
```

假设代码顺利编译，没有出现编译错误，现在就可以执行：

```sh
./main
```

这会真正运行 `main.cpp` 中的 `main` 函数。程序会先执行你的代码，再运行自动评分器，检查代码是否正确。

在按照下面的说明完成作业时，我们建议你不时编译，并使用自动评分器测试，确认自己正在朝正确方向前进！

> [!NOTE] ✏️ Windows 用户注意
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

## 第 0 部分：阅读代码并补全 `Course` 结构体

1. 在本作业中，我们会在 C++ 中使用 `Course` 结构体来表示从 ExploreCourses 获取的记录。查看 `main.cpp` 中尚未完成的 `Course` 结构体定义，补全各字段的定义。最终，我们将使用流来生成 `Course` 对象——还记得流处理的是什么类型吗？
2. 查看 `main.cpp` 中的 `main` 函数，尤其注意 `courses` 是如何传递给 `parse_csv`、`write_courses_offered` 和 `write_courses_not_offered` 的。想一想这些函数在做什么。你需要修改函数定义中的某些内容吗？提前透露一下：需要。

## 第 1 部分：`parse_csv`

查看 `courses.csv`。这是一个 CSV 文件，包含三列：Title（课程标题）、Number of Units（学分数）和 Quarter（季度）。实现 `parse_csv`，使它为 CSV 文件中的每一行创建一个 `Course` 结构体，保存这一行的标题、学分数和季度。

你需要考虑几个问题：

1. 打算怎样读取 `courses.csv`？嘿嘿，也许可以用流 😏？
2. 怎样逐行取得文件中的内容？

### 提示

1. 看看我们在 `utils.cpp` 中提供的 `split` 函数，它可能派得上用场！
   - 欢迎查看 `split` 的实现，并向我们提出任何相关问题。它使用了 `stringstream`，所以你应该能够理解它的工作原理。
2. 每一**行**就是一条记录！*这点很重要，所以我们再说一遍 :>)*
3. 在 CSV 文件中，尤其是这里的 `courses.csv`，第一行通常用于定义列名，也就是表头。这一行实际上并不对应一个 `Course`，所以你需要设法跳过它！

## 第 2 部分：`write_courses_offered`

现在，你已经填好了 `courses` 向量，其中包含 `courses.csv` 的所有记录，每条记录都整齐地保存在一个 `Course` 结构体中！你只想了解开课的课程，对吧？**如果一门课程的 Quarter 字段不是字符串 `“null”`，就认为它开课。** 在这个函数中，将 quarter 字段不是 `“null”` 的所有课程写入 `“student_output/courses_offered.csv”`。

> [!IMPORTANT]
> 写入 CSV 文件时，请遵循以下格式：
>
> ```text
> <Title>,<Number of Units>,<Quarter>
> ```
>
> 注意，逗号之间**不要添加空格**！如果不遵守这个格式，自动评分器可不会满意！
>
> 另外，**务必将列名表头写为输出文件的第一行**。它就是你在上一步读取 `courses.csv` 时需要跳过的那一行！

> [!NOTE] ✏️ 译注：引号与空格
> 上文忠实保留了原文展示字符串和路径时使用的弯引号。实际数据值是 `null`，文件路径是 `student_output/courses_offered.csv`，不包含这些引号。“不要添加空格”指不要在 CSV 分隔逗号旁额外加空格，不是删除课程标题本身含有的空格。

调用 `write_courses_offered` 后，我们希望所有开课的课程，也就是所有写入输出文件的课程，都从 `all_courses` 向量中移除。**这意味着该函数运行完毕后，`all_courses` 应当只包含不开课的课程！**

一种做法是使用另一个向量等方式记录开课的课程，再从 `all_courses` 中删除它们。与 Python 和许多其他语言一样，遍历一个数据结构的同时删除其中的元素，通常不是个好主意。因此，你可能会希望在将所有开课课程写入文件*之后*，再进行删除。

## 第 3 部分：`write_courses_not_offered`

你还想了解哪些课程不开课……在 `write_courses_not_offered` 函数中，将 `unlisted_courses` 中的课程写入 `“student_output/courses_not_offered.csv”`。记住，你已经在上一步删除了开课课程，因此 `unlisted_courses` 自然只包含不开课的课程——真幸运！所以这一步应该与第 2 部分十分相似，只是更短，也稍微简单一点。

## 🚀 提交说明

编译并运行之后，如果自动评分器看起来像这样：

![终端窗口：自动评分器已运行，所有测试均通过](docs/autograder.png)

那么，你就完成这份作业了！好耶！

提交作业时：

1. 请填写[这个链接中的反馈表](https://forms.gle/UeD6zjmUpFbhGgw98)。
2. 在 [Paperless](https://paperless.stanford.edu) 上提交作业！

需要提交的文件：

- `main.cpp`
