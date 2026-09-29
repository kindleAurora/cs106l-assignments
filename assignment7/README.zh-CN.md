> 本文件是用户提供的本地 [README.md](README.md) 的全文中文译文。保留原文代码、命令、输出和链接；原文中的问题另用“译注”说明。

![原作业插图](docs/art.png)

# 作业 7：独占所有权指针

截止时间：5 月 31 日，星期日，晚上 11:59。

## 概述

这次作业中，你将实现自己的 `unique_ptr`，从而接触本周课堂上介绍的 RAII 和智能指针等概念。你还将练习课程中学过的一些技能：模板、运算符重载和移动语义。

你需要处理三个文件：

- `unique_ptr.h`：包含你实现 `unique_ptr` 的全部代码。
- `main.cpp`：包含使用你的 `unique_ptr` 的一些代码。你需要在这里编写一个函数！
- `short_answer.txt`：包含几道简答题，需要你在完成作业的过程中作答。

## 运行代码

要运行代码，首先需要编译。打开终端：如果使用 VSCode，可以按 <kbd>Ctrl+`</kbd>，或者选择顶部的 **Terminal > New Terminal（终端 > 新建终端）**。接着，确保你位于 `assign7/` 目录，然后运行：

```sh
g++ -std=c++20 main.cpp -o main
```

> [!NOTE] 译注：目录名
> 原文写的是 `assign7/`，本地仓库中的实际目录是 `assignment7/`。请进入实际目录执行命令。

如果编译过程没有出现错误，现在就可以执行：

```sh
./main
```

这会实际运行 `main.cpp` 中的 `main` 函数。

> [!NOTE] 译注：入口函数的位置
> 这是原说明的说法。当前起始代码由 `main.cpp` 包含的 `autograder/utils.hpp` 提供 `main()`；不需要在 `main.cpp` 中另写一个入口函数。

按照下方说明完成作业时，建议隔一段时间就编译一次，并使用自动评分器测试，以确认自己的方向正确。

> [!NOTE] Windows 用户注意
>
> 在 Windows 上，可能需要使用下面的命令编译，才能看到输出：
>
> ```sh
> g++ -static-libstdc++ -std=c++20 main.cpp -o main
> ```
>
> 此外，生成的可执行文件可能叫 `main.exe`。这时应通过下面的命令运行：
>
> ```sh
> ./main.exe
> ```

## 第 1 部分：实现 `unique_ptr`

作业的第一部分中，你将实现我们在星期四课堂上讨论过的一种智能指针：`unique_ptr`。你要实现的是标准库 [`std::unique_ptr`](https://en.cppreference.com/w/cpp/memory/unique_ptr) 的简化版本。回顾一下：`unique_ptr` 表示一个指向动态分配内存的指针，这块内存仅由一个，也就是“唯一”的变量拥有。当该变量离开作用域时，它会通过调用 `delete`，自动清理自己拥有的已分配内存。这种行为称为 RAII，即“资源获取即初始化”（resource acquisition is initialization）。**对于本作业，你可以假设 `unique_ptr` 指向一个 `T` 类型的单独元素。你不需要在任何地方调用 `delete[]`，也不需要处理指向动态分配数组的指针。**

> [!IMPORTANT] `short_answer.txt`
> **Q1：** 与手动调用 `new` 和 `delete` 相比，使用 RAII 管理内存有哪些好处？请列出一到两点。

> [!NOTE]
> 尽管我们的 `unique_ptr` 不支持数组指针，但只要有需要，仍然可以加入这种行为。例如，C++ 标准库的 `std::unique_ptr` 使用**模板特化**，为数组指针实现不同的行为。这样的模板特化可能如下所示：
>
> ```cpp
> template <typename T>
> class unique_ptr<T[]>;
> ```
>
> 实际上，这样会得到两个版本的 `unique_ptr`：一个用于单个元素，另一个用于元素数组。每个版本支持不同的操作。例如，数组版本提供下标运算符 `operator[]` 来解引用数组中的元素，而单元素版本不提供它。

### 实现 `unique_ptr` 的功能

花一点时间浏览 `unique_ptr.h` 中提供的 `unique_ptr` 代码。我们已经给出了 `unique_ptr` 的基本接口，你需要实现这些接口。记住，`unique_ptr` 的外观和行为应当类似普通指针，支持解引用（`operator*`）和成员访问（`operator->`）等操作。其中有些方法同时提供 `const` 版本和非 `const` 版本，以确保这个类能够正确处理 `const`。

通过完成以下项目，为 `unique_ptr` 实现基本的指针接口。每个任务应该都比较直接，在 `unique_ptr.h` 中添加或修改 1—2 行即可完成：

* `unique_ptr` 的 `private` 部分。
* `unique_ptr(T* ptr)`，即构造函数。
* `unique_ptr(std::nullptr_t)`，即接收 `nullptr` 的构造函数。
* `T& operator*()`。
* `const T& operator*() const`。
* `T* operator->()`。
* `const T* operator->() const`。
* `operator bool() const`。

### 实现 RAII

到了这一步，我们的 `unique_ptr` 已经表现得像一个原始指针，但它还不会进行自动内存管理，例如在 `unique_ptr` 变量离开作用域时释放内存。而且，我们的指针还不“独占”：可以随意复制出多个指向同一块内存的副本。举例来说，假设我们的 `unique_ptr` 在离开作用域时已经能够正确清理它的数据，请看下面的代码：

```cpp
int main()
{
  unique_ptr<int> ptr1 = make_unique<int>(5);

  // ptr1 指向 5（在堆上动态分配）

  {

    unique_ptr<int> ptr2 = ptr1; // 浅复制

  } // <-- ptr2 的数据在这里被释放

  std::cout << *ptr1 << std::endl;
  return 0;
}
```

由于 `ptr1` 和 `ptr2` 指向同一块内存，`ptr2` 离开作用域时，也会把 `ptr1` 的数据一起释放！因此，`*ptr1` 会产生未定义行为。

另一方面，我们仍然应该能够**移动** `unique_ptr`。回顾一下，移动语义使我们能够取得一个对象所拥有资源的所有权，而不进行代价高昂的复制。移动独占指针是合法的，因为它保留了指针的独占性：在任何时刻，我们仍然只有一个指针拥有底层内存。我们只是改变了由谁，也就是由哪个变量，来拥有这块内存。

为了实现自动释放内存、禁止复制和移动语义这些目标，我们必须为 `unique_ptr` 类实现一些特殊成员函数。**具体来说，请实现以下特殊成员函数（SMF）：**

* `~unique_ptr()`：释放指针所指向的内存。
* `unique_ptr(const unique_ptr& other)`：复制一个独占指针。应将此函数删除。
* `unique_ptr& operator=(const unique_ptr& other)`：对独占指针进行复制赋值。应将此函数删除。
* `unique_ptr(unique_ptr&& other)`：移动一个独占指针。
* `unique_ptr& operator=(unique_ptr&& other)`：对独占指针进行移动赋值。

实现以上函数之后，你应该能够通过**第 1 部分**的全部自动评分测试。

> [!IMPORTANT] `short_answer.txt`
> **Q2：** 为 `unique_ptr` 实现移动语义时，例如实现移动构造函数 `unique_ptr(unique_ptr&& other)` 时，必须在函数退出前将参数 `other` 的底层指针设置为 `nullptr`。请用自己的话解释：如果不这样做，会出现什么问题？

## 第 2 部分：使用 `unique_ptr`

既然已经有了 `unique_ptr` 的实现，就来使用它吧！请看一下 `main.cpp`。我们已经为你提供了一个单向链表（`ListNode`）的完整实现，它利用 `unique_ptr` 确保链表中的所有节点都能正确释放。例如，下面这段代码会产生所示的输出：

```cpp
int main()
{

  auto head = cs106l::make_unique<ListNode<int>>(1);
  head->next = cs106l::make_unique<ListNode<int>>(2);
  head->next->next = cs106l::make_unique<ListNode<int>>(3);

  // head 对应的内存：
  //
  // head -> (1) -> (2) -> (3) -> nullptr
  //
  //

} // <- `head` 在这里被析构！

// 输出：
// Constructing node with value '1'
// Constructing node with value '2'
// Constructing node with value '3'
// Destructing node with value '1'
// Destructing node with value '2'
// Destructing node with value '3'
```

注意，我们没有调用过任何一次 `delete`！`unique_ptr` 的 RAII 行为保证了链表中的所有内存都会被递归释放。当 `head` 离开作用域时，它会调用节点 `(1)` 的析构函数，接着调用节点 `(2)` 的析构函数，然后调用节点 `(3)` 的析构函数。

> [!IMPORTANT] `short_answer.txt`
> **Q3：** 这种通过 RAII 进行递归释放的方式对短链表很有效，但在长链表上可能产生问题。为什么？提示：递归函数的调用栈所能达到的“深度”有什么限制？

**你的任务是实现 `create_list` 函数，将 `std::vector<T>` 转换为 `unique_ptr<ListNode<T>>`。** 链表中应保留向量里元素的原有顺序；如果向量为空，应返回 `nullptr`。实现方式有很多，其中一种是倒着构建链表：从尾部开始，逐步向头部推进。**注意，必须使用 `cs106l` 命名空间下的 `cs106l::unique_ptr`，不能使用 `std::unique_ptr`！** 实现时请遵循以下算法：

1. 初始化 `cs106l::unique_ptr<ListNode<T>> head = nullptr`。
2. **倒序**遍历 `std::vector`。对向量中的每个元素：
   - 2a. 创建一个新的 `cs106l::unique_ptr<ListNode<T>> node`，其值为当前向量元素。
   - 2b. 将 `node->next` 设置为 `head`。
   - 2c. 将 `head` 设置为 `node`。
3. 最后，返回 `head`。

> [!IMPORTANT] `short_answer.txt`
> **Q4：** 实现上面的第 2b 和 2c 步时，你可能会遇到编译器不允许赋值的问题，例如将 `node->next` 赋给 `head` 时，它会提示没有复制赋值运算符。这正是我们希望的行为，因为正如前面讨论的，`unique_ptr` 不能被复制！
>
> 为了得到我们需要的行为，必须让编译器将 `head` **移动赋值**给 `node->next`，而不是进行复制赋值。回顾移动语义的课程，可以通过 `node->next = std::move(head)` 来做到这一点。
>
> 在这里，`std::move` 的作用是什么？为什么在这个地方使用 `std::move` 和移动语义是安全的？

> [!NOTE] 译注：Q4 中赋值方向的表述
> 原文 Q4 第一段说“将 `node->next` 赋给 `head`”，方向与它后面的代码和第 2b 步相反；译文保留了原文表述。后面的示例 `node->next = std::move(head)` 实际表示将 `head` 移动赋值给 `node->next`。

> [!NOTE]
> 倒序遍历向量时，要小心把 `size_t` 用作下标。`size_t` 只能表示非负整数；当检查 `for` 循环边界时尝试让它降到零以下，可能出现意料之外的行为。
>
> 要解决这个问题，可以尝试改用 `int`。

> [!NOTE] 译注：倒序下标的范围
> 原文建议改用 `int`，适用于元素数量能用 `int` 表示的情况；从很大的容器尺寸转换为 `int` 可能超出范围。倒序迭代器等写法也能避免无符号下标减到零以下的问题。

实现了 `create_list` 后，我们就可以创建链表并打印它了。如果愿意多探索一点，可以看看 `map_list()` 和 `linked_list_example()`：它们会一起调用你的 `create_list`，然后逐行打印链表元素。到了这一步，你应该能够通过**第 2 部分**的全部测试。

## 🚀 提交说明

如果通过了所有测试，就可以提交了！提交作业的步骤如下：

1. 请填写[此链接中的反馈表](https://forms.gle/uHr3J8Vm3gECkZpm9)。
2. 在 [Paperless](https://paperless.stanford.edu) 上提交作业！

需要交付的文件为：

- `unique_ptr.h`
- `main.cpp`
- `short_answer.txt`

截止时间之前，你可以不限次数地重新提交。
