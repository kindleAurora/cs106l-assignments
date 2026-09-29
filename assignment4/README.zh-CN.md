> [!INFO] 本地 README 全文中文译文
> 本文逐段翻译同目录的[英文原文](README.md)。代码、命令、标识符和程序输出保留原样；原文中的技术疑点另列在文末“译注”中。日期按原文保留。

![黑色背景的页首图片，以代码字体显示 ispell 命令](docs/header.png)

# 作业 4：Ispell

截止时间：5 月 8 日（星期五）晚上 11:59。

## 概述

我们已经花了一些时间讨论 STL 的核心组成部分——容器、迭代器、函数对象和算法——以及支撑这一切的关键要素——模板。现在，把这些知识结合起来吧！在这次作业中，你将编写 [Ispell](https://en.wikipedia.org/wiki/Ispell) 的核心逻辑。这是一种老式 Unix 风格的拼写检查器，能够执行简单的拼写检查。为此，你需要编写一些使用 `<algorithm>` 头文件和新的 C++ ranges 库的代码。

你的全部代码都将写在 `spellcheck.cpp` 中。完成后，你会得到一个这样的拼写检查器：

![拼写检查程序在终端中的运行示例](docs/spellcheck.png)

> [!IMPORTANT]
> 这份作业说明看起来可能很长，但实际上你需要编写的代码并不多！我们加入了许多额外细节，希望能让实现过程更加清楚。如果有任何让你困惑的地方，请告诉我们（可以在 Ed、课堂或答疑时间联系课程团队）！我们也将在星期二（05/05）的课堂上讲解 `tokenize`，帮助大家开始这次作业！

要下载本作业的起始代码，请查看课程作业仓库中的[**入门说明（Getting Started）**](../README.md#getting-started)。

## 运行代码

要运行代码，首先需要编译。打开终端（如果使用 VSCode，可以按 Ctrl 加反引号键，或从顶部菜单选择 **Terminal > New Terminal**，即“终端 > 新建终端”）。然后确认你位于 `assignment4/` 目录中，运行：

```sh
g++ -std=c++20 main.cpp spellcheck.cpp -o main
```

假设代码编译成功，没有任何编译错误，现在就可以执行：

```sh
./main
```

这条命令会实际运行 `main.cpp` 中的 `main` 函数。

在按照下面的说明完成作业时，我们建议你不时编译代码，并使用自动评分程序测试，确认自己的实现方向正确！

> [!NOTE]
>
> ### Windows 注意事项
>
> 在 Windows 上，你可能需要使用下面的命令编译，才能看到输出：
>
> ```sh
> g++ -static-libstdc++ -std=c++20 main.cpp spellcheck.cpp -o main
> ```
>
> 此外，生成的可执行文件可能名为 `main.exe`。在这种情况下，使用下面的命令运行：
>
> ```sh
> ./main.exe
> ```

## 构建 Ispell

经典 Unix 程序 Ispell 的工作方式如下。首先，把包含所有常见英文单词的词典载入内存。如果在词典中找不到某个单词，就认为它拼写有误。程序使用 [Damerau-Levenshtein 距离](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance)算法，为每个拼错的单词寻找修改建议。这个距离大致表示，把一个单词变成另一个单词需要进行多少次编辑：添加、删除或替换一个字母，或者交换两个相邻字母。如果某个词典单词与拼错单词之间的 Damerau-Levenshtein 距离恰好是 1，就把它加入建议列表。这里的想法是：人们拼错单词时，通常只差一个小改动，例如比较 `"mispelled"` 与 `"misspelled"`。

在这次作业中，我们已经提供了构建这个拼写检查器所需的全部基础设施，包括 Damerau-Levenshtein 函数的实现。你的任务是实现单词拼写检查算法的核心。具体来说，你将编写一个算法，把输入字符串拆分为 token 集合（`tokenize`）；再编写另一个算法，接收已经分词的输入字符串和词典，识别真正拼错的单词（`spellcheck`）。为了增加一点挑战，也为了与上周的课程内容联系起来，有一条限制：你的代码不能使用任何 `for` 或 `while` 循环。你必须完全使用 STL 来实现这些任务：`tokenize` 使用传统 STL 算法，`spellcheck` 使用全新的 ranges 库。在这个过程中，你会接触如何使用算法和 lambda 函数，操作 C++ 中的现代数据结构。

听起来可能要做很多事，但不用担心！这份说明会详细带你完成每个算法。

### `tokenize`

```cpp
struct Token { std::string content; size_t src_offset; };
using Corpus = std::set<Token>;
Corpus tokenize(std::string& input);
```

`tokenize` 方法接收一个输入字符串，并将其拆分为一组 `Token` 对象。看看我们在 `spellcheck.h` 中定义的 `Token` 结构体。一个 `Token` 表示较大文件中的一小段内容：从概念上看，它就是一篇较长文本中的一个单词；在代码中，它是文件中从索引 `src_offset` 开始的一段 `std::string`。我们的目标是把输入文件拆分为一组 `Token`，称为一个 `Corpus`（`Corpus` 只是 `std::set<Token>` 的类型别名）。

这个问题的一个关键约束是：token 的两边由空白字符和／或输入文件的边界界定。例如，短字符串 `"history will absolve me"` 包含四个 token：

* `{ content: "history", src_offset: 0 }`
* `{ content: "will", src_offset: 8 }`
* `{ content: "absolve", src_offset: 13 }`
* `{ content: "me", src_offset: 21 }`

为了实现 `tokenize`，我们将使用 `std::transform` 等传统 STL 方法，不使用任何 `for` 或 `while` 循环。整体思路如下：

1. 找出所有指向空白字符的迭代器。
2. 根据相邻空白字符之间的内容生成 token。
3. 删除空 token。

下面是可以按步骤操作的实现指南：

1. **第一步：找出所有指向空白字符的迭代器**

    如果能够获得字符串中所有指向空白字符的迭代器，我们就可以大致把 token 看作两个空白字符之间的字符。我们需要的操作很像反复调用 `find_if`，收集全部指向空白字符的迭代器。幸运的是，我们已经提供了完成这项工作的函数 `find_all`。

    > [!INFO] 📄 [`find_all`](./utils.cpp)
    >
    > ```cpp
    > template <typename Iterator, typename UnaryPred>
    > std::vector<Iterator> find_all(Iterator begin, Iterator end, UnaryPred pred);
    > ```
    >
    > 返回一个 vector，包含 `begin` 与 `end` 之间、所指元素满足一元谓词 `pred` 的全部迭代器。**这个 vector 也包含边界迭代器 `begin` 和 `end`**。换句话说，如果 `it` 是返回 vector 中的一个迭代器，那么 `pred(*it)`、`it == begin` 或 `it == end` 中至少有一项成立。vector 中的迭代器保证按顺序排列。

    对字符串 `source` 调用 `find_all`，并传入一个判断字符是否为空白字符的一元谓词，就能得到由所有空白字符迭代器组成的 vector。好在 C++ 已经提供了这样的函数，名为 `isspace`。

    > [!INFO] 📄 [`std::isspace`](https://en.cppreference.com/w/c/string/byte)
    >
    > 注意：把这个函数作为谓词传入时，必须写成 `std::isspace`[^1]。
    >
    > ```
    > int std::isspace(int ch);
    > ```

2. **第二步：根据相邻空白字符之间的内容生成 token**

    现在，我们已经得到了所有指向空白字符的迭代器，可以把一个 token 看作两个相邻的空白字符迭代器之间的字符区间。下面这幅图有助于理解：

    ```
    "history will absolve me"
     ▲      ▲    ▲       ▲  ▲
     ├──────┼────┼───────┼──┤
     │  t1  │ t2 │   t3  │t4│
    ```

    箭头表示 `find_all` 返回的迭代器。可以看到，token 就是相邻两支箭头之间的字符。不必担心某个迭代器是否确实指向空白字符，也不必担心如何“修剪”token：`Token` 有一个接收一对迭代器的构造函数，会自动清理边缘的空白字符。

    > [!INFO] 📄 [`Token`](./spellcheck.cpp)
    >
    > ```cpp
    > template <typename It>
    > Token(std::string& source, It begin, It end);
    > ```
    >
    > 给定字符串 `source`，以及界定 `source` 中某个 token 范围的一对迭代器 `begin` 和 `end`，构造一个 `token`。构造函数会自动清理 token 边缘多余的空白字符和标点符号。

    我们需要以某种方式，为每一对相邻迭代器调用这个构造函数。为此，将使用 [`std::transform` 的重载 (3)](https://en.cppreference.com/w/cpp/algorithm/transform)。

    > [!INFO] 📄 [`std::transform`](https://en.cppreference.com/w/cpp/algorithm/transform)
    >
    > ```cpp
    > template <class InputIt1, class InputIt2, class OutputIt, class BinaryOp>
    > OutputIt std::transform(InputIt1 first1, InputIt1 last1, InputIt2 first2,
    >                         OutputIt d_first, BinaryOp binary_op);
    > ```
    >
    > 给定两个大小相同的区间，分别从 `first1` 和 `first2` 开始，第一个区间的尾后迭代器为 `last1`。函数对两个区间中的每一对迭代器应用二元函数 `binary_op`，例如 `binary_op(first1, first2)`、`binary_op(first1 + 1, first2 + 1)` 等，并把结果存入从 `d_first` 开始的同样大小的输出区间。

    对于 `binary_op`，可以提供一个 lambda 函数，接收两个 `std::string::iterator`：`it1` 和 `it2`。正如课堂上讨论的，你也可以把 lambda 的参数类型写成 `auto`。它使用前面介绍的构造方式 `Token { source, it1, it2 }` 来构造 `Token`。注意，必须把 `source` 传给这个构造函数，因此需要在所创建的 lambda 中捕获它！**必须按引用捕获 `source`，否则代码无法正常运行！**

    > [!WARNING] ‼️⚠️📢🚨 警告 🚨📢⚠️‼️
    > 这里再次强调上一点，因为过去有同学在这里遇到过问题。为了让 `Token` 构造函数正常工作，**必须在 lambda 函数中按引用捕获 `source`**。如果忘记了如何操作，请回顾课程幻灯片中的 lambda 捕获语法。

    对于输出区间 `d_first`，先创建一个 `std::set<Token>`，存放找到的 token。假设这个集合叫作 `tokens`，那么就可以创建一个 [`std::inserter(tokens, tokens.end())`](https://en.cppreference.com/w/cpp/iterator/inserter)，用于存放产生的 token。

    > [!INFO] 📄 [`std::inserter`](https://en.cppreference.com/w/cpp/iterator/inserter)
    >
    > ```cpp
    > template <class Container>
    > std::insert_iterator<Container> inserter(Container& c, typename Container::iterator i);
    > ```
    >
    > 这是一个输出迭代器：写入它的任何值，都会被插入容器 `c` 中的 `i` 位置，其中 `i` 的类型是该容器的迭代器类型。返回值是一个 [`std::insert_iterator<Container>`](https://en.cppreference.com/w/cpp/iterator/insert_iterator)，可以作为输出区间传给其他 STL 算法，例如 `std::transform`。
    >
    > 注意，`std::inserter` 返回的迭代器与我们见过的其他迭代器类型略有不同，但它仍然是输出迭代器！其他算法可以对它解引用并写入值，而它在内部把元素插入底层容器。

    对于输入区间 `first1`、`last1` 和 `first2`，需要稍微巧妙地选择迭代器。应当让 `binary_op(first1, first2)` 构造容器中的第一个 token，让 `binary_op(first1 + 1, first2 + 1)` 构造第二个 token，以此类推。如何调整这些参数，才能把 `binary_op` 应用于相邻的空白字符迭代器对？记住，`tokens.begin()` 是容器中的第一个迭代器，`tokens.begin() + 1` 是第二个迭代器，以此类推。**提示：没有任何规定禁止 `first1` 所确定的区间与 `first2` 所确定的区间相互重叠！**

3. **第三步：删除空 token**

    到目前为止，生成的部分 token 会是空的。例如，字符串里出现多个连续空白字符时会怎样？需要删除这些 token。幸运的是，[`std::erase_if` 函数](https://en.cppreference.com/w/cpp/container/set/erase_if)可以从 `std::set` 中删除满足指定条件的元素。

    > [!INFO] 📄 [`std::erase_if`](https://en.cppreference.com/w/cpp/container/set/erase_if)
    >
    > ```cpp
    > template <class Key, class Compare, class Alloc, class Pred>
    > std::set<Key, Compare, Alloc>::size_type erase_if (std::set<Key, Compare, Alloc>& c, Pred pred);
    > ```

    对于 `pred`，可以传入一个检查 token 是否为空的 lambda。例如，检查 `token.content.empty()`。

    最后，返回 `tokens`；它包含输入字符串中的全部有效 token。

完成这一步后，你的拼写检查程序应该开始报告 token 数量。编译代码后，可以运行：

```sh
./main "hello wrld"
```

来检查字符串 `"hello wrld"` 的拼写。它应该输出：

```
Loading dictionary... loaded 464811 words.
Tokenizing input... got 2 tokens.
```

你的 `tokenize` 方法以非常快的速度，完成了包含约 50 万个词的英文词典以及输入字符串 `"hello wrld"` 的分词。不过，此时它还没有真正检查拼写：`"wrld"` 会被报告为拼写正确。要修复这个问题，需要实现 `spellcheck` 函数。

### `spellcheck`

```cpp
struct Misspelling { Token token; std::set<std::string> suggestions; };
using Dictionary = std::unordered_set<std::string>;
std::set<Misspelling> spellcheck(const Corpus& source, const Dictionary& dictionary);
```

`spellcheck` 方法接收已经分词的 `Corpus`，也就是 `tokenize` 方法的输出，以及一个 `Dictionary`。后者只是一个 `std::unordered_set<std::string>`，表示全部有效的英文单词。函数返回一个由 `Misspelling` 结构体组成的集合。每个 `Misspelling` 都记录一个拼错的 `token`，以及一组建议单词；可以用这些词替换 `token`，使其拼写正确。

为了识别拼写错误，我们将执行下面的算法。这一次，将练习使用 `std::ranges::views` 命名空间中的新 ranges/views 库：

1. 跳过已经拼写正确的单词。
2. 对其他单词，使用 Damerau-Levenshtein 在词典中查找只差一次编辑的词。
3. 丢弃没有任何修改建议的拼写错误。

下面是这个算法的分步实现指南：

1. **第一步：跳过已经拼写正确的单词。**

    如果一个单词出现在 `dictionary` 中，就知道它拼写正确。例如，`dictionary.contains("world")` 返回 `true`，而 `dictionary.contains("wrld")` 返回 `false`。第一步是跳过 `source` 中已经拼写正确的单词。为此，可以使用 `std::ranges::views::filter` 视图。

    > [!INFO] 📄 [`std::ranges::views::filter`](https://en.cppreference.com/w/cpp/ranges/filter_view)
    >
    > ```cpp
    > template <ranges::viewable_range R, class Pred>
    > constexpr ranges::view auto filter(R&& r, Pred&& pred);
    >
    > template <class Pred>
    > constexpr /* 区间适配器闭包 */ filter(Pred&& pred);
    > ```
    >
    > `filter(r, pred)` 生成一个视图，对底层区间 `r` 进行适配：遍历这个视图时，只会包含满足 `pred` 的元素。`filter(pred)` 创建一个*区间适配器*，可以通过 `operator|` 把它连接到某个区间上，如下所示。

    构建 `std::ranges::views` 管道时，我们通过一系列步骤把区间连接起来。每一步都对上一步的结果进行*适配*，通过 lambda 以惰性方式执行某种操作，例如筛选或转换元素。查看上面的 `std::ranges::views::filter` 定义，可以看到有两种使用方式：

    ```cpp
    auto view = std::ranges::views::filter(source, /* 一个用作谓词的 lambda 函数 */);

    /* ……等价于…… */

    auto view = source | std::ranges::views::filter(/* 一个用作谓词的 lambda 函数 */);
    ```

    第二种写法可以说更简洁，因为它允许我们通过 `operator|`，把管道中的多个步骤连接起来，而不必为每一步创建单独的变量。注意，完整写出 `std::ranges::views::filter` 有些繁琐，因此人们经常通过创建*命名空间别名*来缩短写法：

    ```cpp
    namespace rv = std::ranges::views;
    auto view = source | rv::filter(/* 一个用作谓词的 lambda 函数 */);
    ```

    自动评分程序接受这两种写法：使用命名空间别名的 `rv::filter`，或完整的 `std::ranges::views::filter`。

    你在这一步的任务，是把 `/* 一个用作谓词的 lambda 函数 */` 替换成一个接收 `Token` 的 lambda；当这个 token 的内容拼写**不正确**时，返回 `true`。我们只关心拼错的单词。为此，需要在 lambda 内部使用 `dictionary`，因此必须捕获它。应该按引用捕获，还是按值捕获呢？

2. **第二步：使用 Damerau-Levenshtein，在词典中查找只差一次编辑的词**

    此时，`view` 表示 `source` 中所有*拼写不正确*的 token 所组成的视图。接下来，使用 `std::ranges::views::transform` 视图，把这些拼错的 token 分别转换为对应的 `Misspelling` 对象，并在转换过程中生成修改建议。

    > [!INFO] 📄 [`std::ranges::views::transform`](https://en.cppreference.com/w/cpp/ranges/transform_view)
    >
    > ```cpp
    > template <ranges::viewable_range R, class F>
    > constexpr ranges::view auto transform(R&& r, F&& func);
    >
    > template <class F>
    > constexpr /* 区间适配器闭包 */ transform(F&& func);
    > ```
    >
    > `transform(r, func)` 生成一个视图，对底层区间 `r` 进行适配：遍历这个视图时，会对 `r` 中的每个元素 `e` 应用 `func(e)`，将其转换为一个新元素。`transform(pred)` 创建一个*区间适配器*，可以通过 `operator|` 把它连接到某个区间上。

    如果把这一步与上一步结合起来，代码大致如下：

    ```cpp
    namespace rv = std::ranges::views;
    auto view = source
        | rv::filter(/* 一个用作谓词的 lambda 函数 */)
        | rv::transform(/* 一个把 Token 转为 Misspelling 的 lambda 函数 */);
    ```

    注意：这只是其中一种做法。如果选择使用 `transform(r, func)` 重载，或不使用 `namespace rv` 别名，你的解法可能看起来不同。

    在 `/* 一个把 Token 转为 Misspelling 的 lambda 函数 */` 处应该填写什么？应将其替换为一个接收 `Token` 对象的 lambda，生成一个 `Misspelling` 对象，其中包含对 `token` 的全部替代拼写建议。为了找出建议，我们会搜索 `dictionary`，寻找与 `token.content` 之间的 Damerau-Levenshtein 距离恰好为 `1` 的所有单词。可以使用已经提供的 `levenshtein` 函数来计算该距离。

    > [!INFO] 📄 [`levenshtein`](./spellcheck.h)
    >
    > ```cpp
    > size_t levenshtein(const std::string& a, const std::string& b);
    > ```
    >
    > 返回 `a` 与 `b` 之间的 Damerau-Levenshtein 距离。粗略地说，这表示把 `a` 变成 `b` 所需的修改次数。实际上，这个函数实现了经过高度优化的 Damerau-Levenshtein 距离算法：如果在计算过程中的任何时刻判断出距离将大于 `1`，就会提前退出。

    注意，对*每一个*拼错的单词，都应遍历 `dictionary` 并查找建议。**这意味着，需要在 `/* 一个把 Token 转为 Misspelling 的 lambda 函数 */` 内部，再嵌套一次 `std::ranges::views::filter` 调用。**为了构造存放建议的 `std::set`，需要使用 [`std::set` 构造函数的重载 (4)](https://en.cppreference.com/w/cpp/container/set/set)，将内部的建议词视图实化为一个集合，从而触发惰性求值。

    > [!INFO] 📄 [`std::set`](https://en.cppreference.com/w/cpp/ranges/transform_view)
    >
    > ```cpp
    > template <class InputIt>
    > set(InputIt first, InputIt last, const Compare& comp = Compare(), const Allocator& alloc = Allocator());
    > ```
    >
    > 从两个迭代器 `first` 和 `last` 之间的元素区间创建一个 `set`。

    例如，下面的代码可以把一个视图实化为集合：

    ```cpp
    auto view = dictionary | rv::filter(/* 一个用作谓词的 lambda 函数 */);
    std::set<std::string> suggestions(view.begin(), view.end());
    ```

    最后，要根据 `token` 和建议集合 `suggestions` 创建 `Misspelling` 对象，可以使用统一初始化：

    ```cpp
    Misspelling { token, suggestions }
    ```

    它应该作为上面 `/* 一个把 Token 转为 Misspelling 的 lambda 函数 */` 的返回值。

3. **第三步：丢弃没有任何修改建议的拼写错误。**

    此时，`view` 包含全部拼错的单词及其修改建议：它是一个由 `Misspelling` 对象组成的集合视图。不过，某些 `Misspelling` 对象可能没有任何建议。例如，乱码单词 `"adskadnfknfs"` 肯定拼写不正确，但英文词典中没有哪个词与它只差一次编辑。在返回结果前，我们希望从视图中删除这些没有建议的拼写错误。

    再次对 `view` 应用 `std::ranges::views::filter` 即可。到这里，你应该已经掌握完成这一步所需的全部信息！过滤掉没有建议的 `Misspelling` 后，需要把 `view` 实化为 `std::set<Misspelling>` 并返回。方法与上面第二步中处理 `suggestions` 的过程类似！

    > [!WARNING] [`std::ranges::to`](https://en.cppreference.com/w/cpp/ranges/to)
    >
    > 你可能记得，课堂上我们使用 `std::ranges::to`，把一个 `char` 视图实化为 `std::string`：
    >
    > ```cpp
    > auto v = s | rv::filter(isalpha)
    >            | /* 其他步骤 */
    >            | std::ranges::to<std::string>();
    > ```
    >
    > 你可能想在这里类似地使用 `std::ranges::to<std::set<Misspelling>>()`。这个想法很好！但 `std::ranges::to` 是 C++23 才引入的方法，具体能否编译取决于你的编译器版本。为保险起见，也为了确保我们使用自动评分程序时能成功编译，请使用接收迭代器的 `std::set<Misspelling>` 构造函数。**总的来说，本作业只允许使用截至 C++20 的 C++ 特性。**

如果到这里的所有内容都实现正确，你应该已经拥有一个能够完整工作的拼写检查器！为了测试，可以重新编译并运行：

```sh
./main "This string is mispelled"
```

应该看到类似下面的输出：

![拼写检查程序在终端中的运行示例](docs/mispelled.png)

也可以检查已经提供的某个示例文件：

```sh
./main --stdin < "examples/(marquez).txt"
```

> [!NOTE] PowerShell 用户
> 如果使用 Microsoft PowerShell（Windows），检查示例文件的语法略有不同：
>
> ```sh
> Get-Content "examples/(marquez).txt" | ./main --stdin
> ```

> [!NOTE]
> 我们鼓励你多尝试这个拼写检查程序，看看能发现哪些有趣的行为。下面是可以尝试的完整选项列表：
>
> ```
> ./main [--dict dict_path] [--stdin] [--unstyled] [--profile] text
>
> --dict dict_path  设置词典位置，默认为 words.txt
> --stdin           从标准输入读取，可以用它通过管道或重定向读入文件
> --unstyled        不给输出添加任何颜色！
> --profile         分析代码性能，打印分词和拼写检查耗时
> text              不使用标准输入时，要检查拼写的文本
> ```
>
> 如果想增加一点挑战，可以尝试使用 `--profile` 选项运行代码。虽然我们的拼写检查算法采用了简单的暴力搜索方式，需要遍历整个约 50 万词的词典，但它运行起来仍然相当快！欢迎研究如何在保持输出正确的前提下，改进算法性能！这完全是可选内容，不过我们很想看看你能想出什么办法。

## 🚀 提交说明

为了全面测试拼写检查器，请重新编译并运行自动评分程序：

```sh
./main
```

如果通过全部测试，就可以提交了！提交作业的步骤如下：

1. 请填写[此链接中的反馈表](https://forms.gle/AMq7kvVKprKmBafKA)。
2. 在 [Paperless](https://paperless.stanford.edu) 上提交作业！

需要交付的文件为：

* `spellcheck.cpp`

截止时间前可以任意多次重新提交。

[^1]: 使用 `std::isspace` 时，实际上存在不止一个版本的函数。

    ```cpp
    int isspace(int ch);                          // 定义于头文件 <cctype> 和 <ctype.h>

    template <class CharT>
    bool isspace(CharT ch, const locale& loc);    // 定义于头文件 <locale>
    ```

    严格来说，第一个版本既作为 [`namespace std` 的一部分](https://en.cppreference.com/w/cpp/header/cctype)存在，也作为[从 C 继承而来的独立函数](https://en.cppreference.com/w/c/string/byte)存在，后者不属于任何特定的命名空间。第二个版本属于 `std`，定义在 `<locale>` 头文件中。单独写 `isspace` 指的是 C 版本，而 `std::isspace` 同时指代上面的两个函数，因此编译器难以推导 `UnaryPred` 类型参数。

    有时会看到 `::isspace` 这样的写法：它只是告诉 C++ 去*全局命名空间*中查找 `isspace`，而不是在 `std` 中查找，效果相同。

---

## 译注：原文与起始代码中的细节

以下说明独立于原文，不更改作业要求或给出完整解法。

1. **`isspace` 的原文表述存在矛盾。** 正文说必须传入 `std::isspace`，脚注又指出该名字可能包含重载而使模板参数推导失败。可以使用签名明确的包装 lambda 来调用它；调用 `<cctype>` 的字符分类函数时，参数必须是 `EOF` 或能表示为 `unsigned char` 的值，不能直接传入负的普通 `char` 值。脚注所说的全局函数也不应作为所有头文件环境下都存在的保证。
2. **`std::transform` 操作的是解引用后的元素。** 原文将调用简写为 `binary_op(first1, first2)`；通常准确的表达是 `binary_op(*first1, *first2)`。本作业中，输入 vector 的元素恰好也是迭代器，所以 lambda 接收到的仍是指向原字符串的迭代器。
3. **`tokens.begin() + 1` 是原文的变量使用错误。** 前文的 `tokens` 是 `std::set<Token>`，它的迭代器不支持 `+ 1`。该段应理解为操作 `find_all` 返回的迭代器 vector；vector 的迭代器支持随机访问运算。
4. **现有距离函数与文中名称并不完全一致。** 当前 `utils.cpp` 中的 `levenshtein` 实现包含插入、删除、替换及提前退出，但没有相邻字符交换的计算分支。应按提供的函数完成作业，无需自行补写或替换算法。另外，提前退出的返回值不保证是完整精确距离；本题关注它是否恰好等于 `1`。
5. **原文有两处链接指向不准确。** `Token` 构造函数的实际定义在 [`spellcheck.h`](spellcheck.h)，原文链接指向 `spellcheck.cpp`。`std::set` 说明框中的原链接指向 `transform_view` 页面；集合构造函数的正确参考链接是 [`std::set` 构造函数](https://en.cppreference.com/w/cpp/container/set/set)。原链接在正文中保留，便于对照。
6. **`transform(pred)` 是原文中的参数命名笔误。** 在该段给出的接口中，转换函数名为 `func`，因此此处应理解为 `transform(func)`。`/* 区间适配器闭包 */` 等内容仍是文档里的说明占位符，不是可以原样编译的完整声明。
7. **token 数量与输出文本以本地程序为准。** 原文示例保留了 `464811` 及旧版输出措辞；当前 `main.cpp` 会报告 `unique words`，具体数量取决于词典与分词结果。
