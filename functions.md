# 增加原生函数
向PHP扩展中增加原生函数和（或者）类是非常有用的，且这些函数和（或者）类可以直接从PHP脚本中调用。对于函数来说非常简单。只要你按照下面的方式书写C++函数，那么你就可以直接从PHP脚本中调用它。

```
void example1();
void example2(Php::Parameters &params);
Php::Value example3();
Php::Value example4(Php::Parameters &params);
```

在上述的代码中有两个重要的PHP-CPP类，一个是Php::Value，一个是Php::Parameters。Php::Value类非常强大，它与PHP的变量作用一样：它可以容纳几乎任何值（整形、浮点型小数、字符串，当然也有索引和关联数组和对象）。Php::Parameters类以数组或向量的形式来接收传递到函数的所有变量是再适合不过了。后面我们会对这两个类进行深入的讨论。

为了能够在PHP中调用这些函数，那么你必须要将这些函数加入到extension对象中，同时赋予他们名称。将来在PHP脚本中将以这些名称来调用它们。

```
#include <phpcpp.h>
#include <iostream>

void myFunction()
{
    Php::out << "example output" << std::endl;
}

extern "C" {
    PHPCPP_EXPORT void *get_module() {
        static Php::Extension extension("my_extension", "1.0");
        extension.add<myFunction>("myFunction");
        return extension;
    }
}
```

笔者：extern "C" 说明了这段代码块中应该用存C的语法，然儿上述案例中却用了C++中的泛型特性。如果你发现编译出错，尝试将泛型语句替换成存C语法风格。

```
extension.add("myFunction", myFunction);
```

上面的代码作用不难想象。如果你部署了这个扩展，可以创建一个PHP脚本并调用myFunction方法，之后会输出"example output"。

正如我们前面所说的那样，有四种可以使用的函数形式。在第一个案例中我们展示了最简单的形式：一个不接收任何参数的函数，同时返回空。如果一个带返回值的函数应该怎么办呢？

```
#include <phpcpp.h>
#include <stdlib.h>

Php::Value myFunction()
{
    if (rand() % 2 == 0)
    {
        return "string";
    }
    else
    {
        return 123;
    }
}

extern "C" {
    PHPCPP_EXPORT void *get_module() {
        static Php::Extension extension("my_extension", "1.0");
        extension.add<myFunction>("myFunction");
        return extension;
    }
}
```

酷不酷？在PHP中，一个函数的返回值可以是一个数字，也可以是一个字符串，这完全没有问题。但这在C++中全不可行，因为一个函数必须总是返回同样类型的变量。但是，Php::Value类可以被作为一个数字，也可以作为一个字符串（数组、对象，后面会提到更多） - 现在，我们可以利用纯C++代码来写一个可以返回数字或者字符串变量的函数。你可以用简单的脚本来测试这个函数。


```
<?php
    for ($i=0; $i<10; $i++) echo(myFunction()."\n");
?>
```

上述例子中的输出如下：

```
123
123
string
123
123
string
string
string
string
```

我们注意到了一共有四种生成函数的方式。我们已经看到两种了，但是这两种都没有接收参数。让我们来以第四种方式为例，写一个既能接收参数又能返回值的函数。这面例子中的函数接收若干数字参数，并求其总和：

```
#include <phpcpp.h>

Php::Value sum_everything(Php::Parameters &parameters)
{
    int result = 0;
    for (auto &param : parameters) result += param;
    return result;
}

extern "C" {
    PHPCPP_EXPORT void *get_module() {
        static Php::Extension extension("my_extension", "1.0");
        extension.add<sum_everything>("sum_everything");
        return extension;
    }
}
```

是不是看起来很简单？事实上，Php::Parameters类基本上等同于一个由Php::Value对象填充的std::vector类 - 同时你可以进行轮询（iterate）。我们使用C++11的这种新方式来轮询，使用C++11新关键字“auto”来告诉编译器找出参数向量中存储的变量类型（当然是Php::Value类型）。


