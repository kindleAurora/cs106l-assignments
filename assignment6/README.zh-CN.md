> 本文件是用户提供的本地 [README.md](README.md) 的全文中文译文。保留原文代码、命令、输出和链接；原文中的问题另用“译注”说明。

# 作业 6：探索课程

截止时间：5 月 22 日，星期五，晚上 11:59。

## 概述

这次作业将练习你对 `std::optional` 的理解。我们会使用与作业 1 相同的 `courses.csv`。你需要编写一个函数，尝试在 `CourseDatabase` 对象中找到一个 `Course`，并将其返回。

你还将探索 `std::optional` 类提供的单子操作（monadic operations）。请查看代码，并阅读 `CourseDatabase` 类，了解它的接口。

## 运行代码

要运行代码，首先需要编译。打开终端：如果使用 VSCode，可以按 <kbd>Ctrl+`</kbd>，或者选择顶部的 **Terminal > New Terminal（终端 > 新建终端）**。接着，确保你位于 `assignment6/` 目录，然后运行：

```sh
g++ -std=c++23 main.cpp -o main
```

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
> g++ -static-libstdc++ -std=c++23 main.cpp -o main
> ```
>
> 此外，生成的可执行文件可能叫 `main.exe`。这时应通过下面的命令运行：
>
> ```sh
> ./main.exe
> ```

## 第 0 部分：包含 `<optional>`

在 `main.cpp` 的顶部包含 `<optional>`，因为这次作业将使用 `std::optional`！

## 第 1 部分：编写 `find_course` 函数

这个函数接收字符串 `course_title`，并且应当尝试在 `CourseDatabase` 对象的私有成员 `courses` 中找到对应的 `course`。返回类型应该是什么？提示：传入的 `course_title` 可能对应某个 `Course`，也可能没有对应的课程。

> [!NOTE]
> 你需要修改 `find_course` 的返回类型；它目前是 `FillMeIn`。

## 第 2 部分：修改 `main` 函数

注意，我们在 `main` 函数中这样调用 `find_course`：

```cpp
auto course = db.find_course(argv[1]);
```

现在，你需要使用[单子操作](https://en.cppreference.com/w/cpp/utility/optional)，正确填充 `output` 字符串。下面逐步说明如何完成。

你要实现下面这种行为，**但不能使用 `if` 语句等任何条件判断**：

```cpp
if (course.has_value()) {
    std::cout << "Found course: " << course->title << ","
            << course->number_of_units << "," << course->quarter << "\n";
} else {
    std::cout << "Course not found.\n";
}
```

简单来说，如果找到了课程，那么 `main` 底部的这一行：

```cpp
std::cout << output << std::end;
```

应当产生下面的输出：

```bash
Found course: <title>,<number_of_units>,<quarter>
```

如果没有找到课程，那么这一行：

```cpp
std::cout << output << std::end;
```

应当产生下面的输出：

```bash
Course not found.
```

> [!NOTE] 译注：原文输出语句的笔误
> 上面两处 `std::end` 按原文保留。这里应为用于换行和刷新输出的 `std::endl`；起始文件 `main.cpp` 中已经正确写成了 `std::endl`。

### 单子操作

有三种单子操作：[`and_then`](https://en.cppreference.com/w/cpp/utility/optional/and_then)、[`transform`](https://en.cppreference.com/w/cpp/utility/optional/transform) 和 [`or_else`](https://en.cppreference.com/w/cpp/utility/optional/or_else)。请阅读课程幻灯片中对它们的介绍，也查看一下[标准库文档](https://en.cppreference.com/w/cpp/utility/optional)。你只需要使用其中两种单子操作。

你的代码最终应该类似这样：

```cpp
std::string output = course
    ./* 第一种单子操作 */ (/* ... */)
    ./* 第二种单子操作 */ (/* ... */)
    .value();                                  // 或使用 `.value_or(...)`，见下文
```

**先想清楚 `output` 的类型，再从结果往回推**，可能会有帮助。注意下方提示对各个单子操作作用的描述。

> [!NOTE]
> 回顾每个单子操作的职责。官方 C++ 库文档在这方面解释得不太清楚，所以这里提供了一份简短参考。假设 `T` 和 `U` 是任意类型。
>
> ```cpp
> /**
>  * 简而言之：
>  * 如果存在值，就调用一个函数来产生新的 optional；否则返回空值。
>  *
>  * 传给 `and_then` 的函数接收一个不是 optional 的 T 类型实例，返回 std::optional<U>。
>  * 如果 optional 有值，and_then 就把函数应用于该值，并返回结果。
>  * 如果 optional 没有值（即它为 std::nullopt），就返回 std::nullopt。
>  */
> template <typename U>
> std::optional<U> std::optional<T>::and_then(std::function<std::optional<U>(T)> func);
>
> /**
>  * 简而言之：
>  * 如果存在值，就把函数应用于保存的值，并将结果包装在 optional 中；否则返回空值。
>  *
>  * 传给 `transform` 的函数接收一个不是 optional 的 T 类型实例，返回一个不是 optional 的 U 类型实例。
>  * 如果 optional 有值，transform 就把函数应用于该值，并返回包装在 std::optional<U> 中的结果。
>  * 如果 optional 没有值（即它为 std::nullopt），就返回 std::nullopt。
>  */
> template <typename U>
> std::optional<U> std::optional<T>::transform(std::function<U(T)> func);
>
> /**
>  * 简而言之：
>  * 如果存在值，就返回 optional 本身；否则调用一个函数来产生新的 optional。
>  *
>  * 与 and_then 相反。
>  * 传给 or_else 的函数不接收参数，返回 std::optional<U>。
>  * 如果 optional 有值，or_else 就返回它。
>  * 如果 optional 没有值（即它为 std::nullopt），or_else 就调用函数并返回其结果。
>  */
> template <typename U>
> std::optional<U> std::optional<T>::or_else(std::function<std::optional<U>(T)> func);
> ```
>
> 例如，给定一个 `std::optional<T> opt` 对象，可以像这样调用单子操作：
>
> ```cpp
> opt
>   .and_then([](T value) -> std::optional<U> { return /* ... */; })
>   .transform([](T value) -> U { return /* ... */; });
>   .or_else([]() -> std::optional<U> { return /* ... */; })
> ```
>
> 注意，lambda 函数中的 `->` 写法是一种显式指定函数返回类型的方式。
>
> 每个方法都会返回一个 `std::optional`，因此可以将它们链式连接起来。如果能够确定链式调用最后一定有值，可以调用 [`.value()`](https://en.cppreference.com/w/cpp/utility/optional/value) 取出该值。否则，可以调用 [`.value_or(fallback)`](https://en.cppreference.com/w/cpp/utility/optional/value_or)：存在值就取得结果，不存在值就取得备用值 `fallback`。

> [!NOTE] 译注：参考伪代码中的类型和语法问题
> 上述签名和示例按原文保留，用于说明概念，并不是可以直接复制编译的完整代码。
>
> - `or_else` 的回调不接收参数，且须返回与当前对象相同类型的 `std::optional<T>`。原文签名却给回调写了一个 `T` 参数，并允许任意 `U`，与实际接口不符。
> - 示例中 `and_then` 的结果是 `optional<U>`，后续 `transform` 的回调参数应与这个 `U` 匹配，不能在 `T`、`U` 不同时继续按 `T` 接收。
> - 示例在 `.transform(...)` 后写了分号，会提前结束链式表达式；下一行单独以 `.or_else` 开头不能编译。应去掉中间分号，并在完整表达式末尾结束语句。
> - 标准库的实际接口使用可调用对象模板，并非这里用来讲解的这些 `std::function` 签名。`std::nullopt` 是表示空状态的标记；一个空的 optional 对象本身仍具有 `std::optional<T>` 类型。

## 🚀 提交说明

如果通过了所有测试，就可以提交了！提交作业的步骤如下：

1. 请填写[此链接中的反馈表](https://forms.gle/aGuFqLyhB18mNoPKA)。
2. 在 [Paperless](https://paperless.stanford.edu) 上提交作业！

需要交付的文件为：

- `main.cpp`

截止时间之前，你可以不限次数地重新提交。
