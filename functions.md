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

