---
title: C++ 11 and 14 Lambda表达式
date: 2017-10-09 15:30:00
tags: C++
---

<!-- TOC -->

- [Lambda表达式基本介绍](#lambda表达式基本介绍)
- [Lambda表达式的使用情形](#lambda表达式的使用情形)
    - [C++标准库中使用Lambda表达式](#c标准库中使用lambda表达式)
        - [algorithm](#algorithm)
        - [thread](#thread)
        - [function](#function)
    - [Qt库信号槽中使用Lambda表达式](#qt库信号槽中使用lambda表达式)
    - [用Lambda表达式代替回调函数](#用lambda表达式代替回调函数)
- [Lambda表达式相关内容](#lambda表达式相关内容)
- [感谢](#感谢)

<!-- /TOC -->

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

充分了解lambda表达式的特点之后，我们应当了解该在哪些地方进行使用，防止频繁声明导致的代码膨胀或者运行速度降低等问题的出现。下面三种情形中，第一种是使用标准库中对Lambda表达式的支持，从而达到简化代码，清晰逻辑的目的；第二种，是我在使用Qt开发软件时，对于lambda表达式的使用观点；第三种，则是我在前两种的基础上，总结了别人的设计方法，提出的一些建议，本人经验并不丰富，谬误之处请批评指出。

### C++标准库中使用Lambda表达式

C++标准库中有 **谓词(Predicate)** 的概念：谓词表示一个可以接受迭代器解引用的值，并返回`bool`类型或者其他值作为反馈的回调对象。该回调对象，可以是一个函数，也可以是一个重载了`operator()`的类或者结构体。

在lambda表达式出现之前，我们会使用**仿函数(functor)**，来作为标准库中的算法库函数的谓词。例如某个使用自定义比较方法的快速排序：
```
#include <map>
#include <vector>
#include <iostream>
#include <algorithm>

using namespace std;

struct sort_func{
    bool operator()(const pair<int, int>& l, const pair<int, int>& r) const
    {
        if (l.first == r.first)
            return l.second < r.second;
        return l.first < r.first;
    }
};

int main()
{
    vector<pair<int, int>> num_pairs = { 
        {1, 2}, {3, 1}, {2, 2}, {4, 3}
    };
    sort(num_pairs.begin(), num_pairs.end(), sort_func());
    return 0;
}
```
而现在C++支持了lambda表达式的使用，上面的写法可以写成下面的样子:
```
#include <map>
#include <vector>
#include <iostream>
#include <algorithm>

using namespace std;

int main()
{
    vector<pair<int, int>> num_pairs = { 
        {1, 2}, {3, 1}, {2, 2}, {4, 3}
    };
    sort(num_pairs.begin(), num_pairs.end(), [](const pair<int, int>& l, const pair<int, int>& r) 
    {
        if (l.first == r.first)
            return l.second < r.second;
        return l.first < r.first;
    });
    return 0;
}
```
上面两段代码需要注意：
> 需要在支持 C++ 11 的情况下进行编译，其中用到了 **列表初始化** ，而该特性是 C++ 11 新增的。

#### algorithm

在标准库`<algorithm>`中，实现了许多常用的算法函数，他们通常是某一个元素范围上进行操作，该范围可以使用容器迭代器给出，也可以自行重载迭代器，这里要注意的是，操作的范围定义为[first, last)前闭后开区间，last为最后元素的后一个元素。

算法库中，算法函数可分为 **不修改序列操作** 和 **修改序列操作** 两大类，查找、计数、最大最小值等算法属于第一类，排序、划分、二分查找、集合交并补、堆操作等属于第二类。

我们可以看一下算法库中，对所有元素进行统一操作的`std::transform()`函数的使用：
```
#include <string>
#include <cctype>
#include <iostream>
#include <algorithm>

int main()
{
    std::string s("hello");
    std::transform(s.begin(), s.end(), s.begin(), [](char c){
        return static_cast<char>(
            std::toupper(static_cast<int>(c))
        );
    });
    std::cout << s;
 }
```
上面这段代码会将小写字母转换成大写。

在对某些对象进行操作时，我们应当首先考虑是否能够使用算法库函数来进行操作。在这里，我们需要注意，对象可以是任何的概念，包括 **内存(memory)** 。我们可以看一下算法库中`std::uninitialized_copy`的实例：
```
#include <tuple>
#include <vector>
#include <string>
#include <memory>
#include <iostream>
#include <algorithm>
 
int main()
{
    std::vector<std::string> v = {"This", "is", "an", "example"};
 
    std::string* p;
    std::size_t sz;
    std::tie(p, sz) = std::get_temporary_buffer<std::string>(v.size());
    sz = std::min(sz, v.size());
 
    std::uninitialized_copy(v.begin(), v.begin() + sz, p);
 
    for (std::string* i = p; i != p + sz; ++i) {
        std::cout << *i << ' ';
        i->~basic_string<char>();
    }
    std::return_temporary_buffer(p);
    return 0;
}
```
上面这段示例代码中，我们使用了`std::get_temporary_buffer()`函数为p指针分配了4个`std::string`使用的内存空间(如果程序没有出现内存不足等异常)，然后使用了`std::uninitialized_copy()`函数将`v`中的字符串挨个复制到了`p`指针指向的数组的对应位置上，然后`for`循环输出`p`数组的字符串，并析构它们；最后释放`p`的内存。
>注意，`std::get_temporary_buffer` 和 `std::return_temporary_buffer` 在 C++ 17 被弃用，我认为用 `new` 和 `delete` 就足够了。 另外，`std::string` 占用的空间大小是固定的，它其中封装了一个数组指针。

#### thread

线程库中可以用到lambda表达式的地方，主要在`std::thread`的构造函数上。

`std::thread`的变参构造函数：
```
template< class Function, class... Args > 
explicit thread( Function&& f, Args&&... args );
```
其中`f`必须是一个 `可调用对象(Callable)`，这也是C++的一个概念，该类型是可应用 `INVOKE操作`，例如：函数对象(FunctionObject), `std::function`, `std::bind`等。

#### function

类模板 `std::function` 是通用多态函数封装器。 `std::function` 的实例能存储、复制及调用任何可调用(Callable)目标——函数、 lambda表达式、 bind表达式或其他函数对象，还有指向成员函数指针和指向数据成员指针。

之前，我们举例符`function`的例子：
```
std::function<int(int)> sum_func = [](int x){ return x; };
```

以上，就是C++标准库常用到lambda表达式的地方，但其实还有一些被封装到了例如`<memory>`等库中，在这里并没有被列出。代码重构时希望大家能够好好利用标准库函数，写出友好而又高效的代码。

C++ 17标准对`auto`的使用又做了一些加强，其中有涉及到lambda表达式的地方，希望各位仔细查找相关资料进行学习；C++ 20标准已提出了模版lambda表达式，大家可以关注一下。目前，我在Window环境下使用C++，使用的是MSVC 14.0的编译器，对C++ 17还并不完全，可能需要尽快更换到更新的编译器下使用，而且将学习并使用Boost C++库进行开发。

### Qt库信号槽中使用Lambda表达式

Qt库是我最近比较常用的C++库，但Qt库对lambda表达式的支持并没有像C++标准库那样完善。例如，QThread类就不支持构造函数使用lambda表达式进行初始化(Qt 5.9及之前， Qt dev版还未使用过)。但Qt的 **Signals & Slots** 机制已对lambda表达式进行了支持。

十分推荐大家在Qt库的基础上使用lambda表达式这一特性：
```
connect(sender, &QObject::destroyed, [=](){ this->m_objects.remove(sender); });
```
在使用信号槽机制的时候，或者说我们在应用事件驱动机制来设计程序时，通常需要一个在某一特定上下文环境下的某一段代码进行执行，在没有lambda表达式前，我们需要写一个函数或者一个类，然后将需要的参数传进去，再将该Callable对象送到槽函数上，或者在槽函数里调用。而现在，我们只需要写一个匿名的lambda表达式，不仅可以捕捉使用到的变量，还可以简化代码。

但在使用的过程中，要确定好捕获列表是否使用正确：
```
connect(sender, &QObject::destroyed, [=](){ this->m_objects.remove(sender); });

connect(sender, &QObject::destroyed, [&](){ this->m_objects.remove(sender); });
```
上面两个`connect`区别在于捕获列表，一种是 `=` ，另外一种是 `&` ，一般情况下，使用第一种基本上不会出现什么问题，但对于不可复制类型(删除了复制构造函数和赋值符号函数)，可能就需要特殊注意一下。第二种，我们要明确connect这个模版函数会将后面的槽函数发送到一个其他作用域中进行执行，或者发送到一个事件循环作用域中执行，使用时需要注意，捕获列表中的变量是否会在析构后被调用，如果确保不会，则使用这种捕获列表的形式可以提高一些性能，且不用费劲设计一些调用函数。

使用lambda表达式，可以让我们不必要设计一些类型用来装slot函数，于是我们可以通过一段简洁的代码，来启动一个简单的FileDialog来保存文件：
```
void saveDocumentAs() 
{
    QFileDialog *dlg = new QFileDialog();
    dlg->open();
    QObject::connect(dlg, &QDialog::finished, [=](int result) {
        if (result) {
            QFile file(dlg->selectedFiles().first());
            // ... save document here ...
        }
        dlg->deleteLater();
    });
}
```
或者启动一个QNetworkManager和QEventLoop来实现一个同步http请求：
```
int CNetworkManager::sync_request(UrlType type, const std::vector<QString>& value, 
	std::function<void(const QByteArray&)> success_func)
{
	int res = 0;
	std::shared_ptr<QEventLoop> loop(new QEventLoop, [](QEventLoop* loop) {
		loop->deleteLater();
	});
	std::shared_ptr<QTimer> timer(new QTimer, [](QTimer* timer) {
		timer->deleteLater();
	});
	std::shared_ptr<QNetworkAccessManager> manager(new QNetworkAccessManager, 
		[](QNetworkAccessManager* manager) {
		manager->deleteLater();
	});
	auto url = generate_post_url(_http, httpUrl[type], in_type[type], value);
	auto body = generate_post_body(in_type[type], value);
	auto request = generate_request(url);

	connect(timer.get(), &QTimer::timeout, loop.get(), &QEventLoop::quit);
	connect(manager.get(), &QNetworkAccessManager::finished, loop.get(), &QEventLoop::quit);
	connect(manager.get(), &QNetworkAccessManager::finished, [&](QNetworkReply* reply) {
		if (reply->error() == QNetworkReply::NoError)
			success_func(reply->readAll());
		reply->deleteLater();
		qDebug() << reply->error();
	});

	auto reply = manager->post(request, body);
	timer->start(10000);
	loop->exec();

	if (timer->isActive() == false)
		res = 1;
	else
		timer->stop();
	
	reply->abort(), reply->deleteLater();
	return res;
}
```
上面这段代码，混合使用了lambda表达式以及智能指针等C++ 11新特性。

### 用Lambda表达式代替回调函数

在我们自己开发的软件中，我们可能会写出异步执行的函数，此时，我们就需要在异步执行之后，执行一个回调函数。在lambda表达式出现之后，一旦我们设计并开发出这种框架，那我们完全可以使用`std::function`来封装lambda表达式来使用，这样能够大大提高框架的灵活性。

我们需要经常学习一些新的思路和方法，来不断重构我们自己的程序。

## Lambda表达式相关内容

C++的概念中，`Callable` 是理解lambda表达式的重要概念，对象不仅仅是存储一些变量或者常量，还可以提供可调用的方法或函数，那在面向对象的思路中，函数也应当作为一个对象来理解，虽然函数的代码形式与 `class` 完全不同。实际上，C中的函数是一个指针，而C++也是继承了这一特性，`class` 类型也对应了一个指针，这个指针可以让我们获取其中的 `static` 函数，如：`MyClass::static_func()`。

提到 `Callable` 我们就需要想到除lambda表达式之外的可调用对象，如 `funcObject`, `std::function`, `std::bind` 等。灵活运用它们可以提高框架的可读性。

我最早见到lmabda表达式是在学习JavaScript时看到的，浏览器就是一个EventLoop框架，这个框架会执行JavaScript脚本，如果你使用了
```
function addEventListener(string eventFlag, function eventFunc, [bool useCapture=false])
```
并且为某一个dom对象添加了click事件监听：
```
var button = document.getElementById("button_submit");
button.attachEvent("onclick", function(){
    // ....
} , false);
```
则浏览器会在button上添加一个点击事件。

由于JavaScript的变量作用域与C++完全不同，使用js的人提出了“闭包”概念，虽然C++上可能完全没有必要，但也需要了解一下。

在python下，我们可以如下方式使用：
```
low = 3
high = 7
aList = [1, 2, 3, 4, 5, 6, 7, 8, 9]
filter(lambda x, l = low, h = high : h > x > l, aList)
```
这些脚本语言的lambda表达式形式，比起C++来讲，简洁许多。但随着新标准出现，C++的lambda表达式会变得~~越来越简单(我是不信的)~~。

本段内容限于本人经验，不能详细解释。

## 感谢

+ [Lambda 表达式 （自C++11起）](http://zh.cppreference.com/w/cpp/language/lambda)
+ [Lambda expressions (since C++11)](http://en.cppreference.com/w/cpp/language/lambda)
+ [C++11 新特性：Lambda 表达式](https://www.devbean.net/2012/05/cpp11-lambda/)
+ [Signals & Slots | Qt Core 5.9](http://doc.qt.io/qt-5/signalsandslots.html)