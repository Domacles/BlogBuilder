---
title: C++ 11 and 14 Lambda表达式
date: 2017-10-09 15:30:00
tags: C++
---

**Lambda表达式** 算得上是C++ 11新增特性中最激动人心的一个。这个全新的特性听起来很深奥，但却是很多其他语言早已提供（比如 C#）。简而言之，Lambda表达式就是用于创建匿名函数的。

为什么说 lambda 表达式如此激动人心呢？举一个例子。标准 C++ 库中有一个常用算法的库，其中提供了很多算法函数，比如`sort()`和`find()`。这些函数通常需要提供一个**谓词函数(predicate function)**。所谓谓词函数，就是进行一个操作用的临时函数。比如`find()`需要一个谓词，用于查找元素满足的条件；能够满足谓词函数的元素才会被查找出来。这样的谓词函数，使用临时的匿名函数，既可以减少函数数量，又会让代码变得清晰易读。

## Lambda表达式基本介绍

首先，我们需要认识C++的Lambda表达式的形式：
```
// 完整声明格式 1：
[captures] (params) specifiers exception attr -> ret { body }

// 常用声明格式 2:
[captures] (params) -> ret { body }

// 常用声明格式 3:
[captures] (params) { body }

// 常用声明格式 4:
[captures] { body }
```
对于1,2,3,4的声明格式，我们需要了解：
1. 完整声明。
2. `const lambda`的声明：不能修改以复制捕获的对象。
3. 省略尾随返回类型：闭包的`operator()`的返回类型根据下列规则确定：
    + 若`body`仅由单个带表达式的`return`语句组成，则返回类型是被返回表达式的类型（在左值到右值、数组到指针或函数到指针隐式转换后）；否则，返回类型是 `void`。
    + 返回类型从`return`语句推导，如同对于返回类型声明为`auto`的函数。
4. 省略参数列表：函数不接收参数，如同参数列表是`()`。仅若不使用`constexpr`、`mutable`、`异常规定`、`属性`或`尾随返回`类型之一才能使用此形式。

对于声明格式的各个部分中，我们需要记住：
* `captures` - 零或更多捕获的逗号分隔列表，捕获列表能按如下方式传递（详细描述见下方）：
    + `[a,&b]` 其中，`a`以复制捕获而`b`以引用捕获，即`a`的类型的构造函数会被调用（如果有），而`b`不会。
    + `[this]` 以引用捕获当前对象（`*this`）。
    + `[&]` 以引用捕获所有被用于lambda body的非lambda体内声明的变量(使用auto&进行捕获)。
    + `[=]` 以复制捕获所有被用于lambda body的非lambda体内声明的变量(使用auto进行捕获)。
    + `[]` 不捕获
    + 若变量为静态变量，或者为线程的全局变量或者没有被用于lambda body内，则不会被捕获。
* `params` - lambda的可输入的参数列表，形式如同函数的参数: `(T arg1, T& arg2 ...)`，但请注意，不能使用默认参数， C++ 14中，参数类型可以使用`auto`。
* `specifiers` - 可含有下列指定符：
    + `mutable` ：允许`body`修改以复制捕获的参数，及调用其非 const 成员函数;
    + `constexpr` ：显式指定函数调用运算符为`constexpr`函数。此指定符不存在时，若函数调用运算符恰好满足所有`constexpr`函数要求，则它也会是`constexpr`（C++ 17）。
* `exception` - 为lambda表达式的`operator()`提供异常规定或可指定为`noexcept`。
* `attr` - 为lambda表达式的 operator() 提供**属性指定(attribute specification)**。
* `ret` - 返回类型。若存在，则由函数的`return`语句所隐含（或若函数不返回任何值则为`void`）。
* `body` - 函数体。

在这里给出一个使用 **常用声明格式 2** 的lambda表达式声明和使用的示例：
```
#include <vector>
#include <iostream>
#include <algorithm>
#include <functional>

int main()
{
    int sum = 0;
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7};
    auto sum_func = [&sum](int i) -> void { 
        std::cout << i << ' '; 
        sum += i;
    };

    std::for_each(c.begin(), c.end(), sum_func);
    std::cout << "sum of all : " << sum << std::endl;

    return 0;
}
```
在这里，我们需要注意的是，`sum_func`使用的是`auto`来自动推导类型，有时候我们或许可能写成下面的样子：
```
std::function<int(int)> sum_func = [](int x){ return x; };
```
这种形式虽然与上面示例代码差别只有在`auto`上，但这两个`sum_func`有本质的差别：示例代码中，编译器将将`sum_func`作为函数指针来处理；上面这段代码中，`sum_func`是一个`function`模版类的对象，我们只是用赋值符号`=`调用了`function`的构造函数。

## Lambda表达式的使用情形

充分了解lambda表达式的特点之后，我们应当了解该在哪些地方进行使用，防止频繁声明导致的代码膨胀或者运行速度降低等问题的出现。

### C++标准库中使用Lambda表达式

