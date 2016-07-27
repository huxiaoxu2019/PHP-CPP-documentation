# 指定函数参数
在PHP中可以指定函数参数的类型与个数，并且可以返回一个引用类型或者值类型。在前面的例子中，我们还没有提到过这种机制。我们将这种机制设计到函数实现的环节中，通过检查函数参数'Php::Parameters'对象（它是一个装有Php::Value的std::vector），同时检查其参数的数量和类型是否正确。

所以，‘Extension.add()'方法提供了第三个可选参数。这个参数可以用来指定参数的数量，参数是用引用传递还是值传递和参数的类型：

```
#include <phpcpp.h>

void example(Php::Parameters &params)
{
}

extern "C" {
    PHPCPP_EXPORT void *get_module() {
        static Php::Extension myExtension("my_extension", "1.0");
        myExtension.add<example>("example", {
            Php::ByVal("a", Php::Type::Numeric),
            Php::ByVal("b", "ExampleClass"),
            Php::ByVal("c", "OtherClass")
        });
        return myExtension;
    }
}
```

正如上面你看到的，当我们注册'example'函数同时，通过了一些其他信息。我们告诉PHP引擎这个函数需要接收三个参数：第一个参数是整数，其它两个分别是‘ExampleClass'和'OtherClass'类的实例。最后，你的原生C++函数会接收到一个’Php::Parameters'的实例作为参数，但是当调用函数时，你可以确定Php::Parameters对象持有三个成员，其中有两个成员是上述类的相应实例。

## 你真的可以指定一个未知参数的类型么？
当我们指定参数类型为数字类型时，你可能会觉得奇怪。毕竟，在PHP中并没有官方的方式来指定一个未知参数的类型。当你用PHP写一个函数时，可以指定函数的参数是一个对象或者是数组，但是不能指定成一个字符串类型或者整数类型。

```
<?php
// example how you can enforce that a function can only be called with an object
function example1(MyClass $param)
{
    ...
}

// another example to enforce an array parameter
function example2(array $param)
{
    ...
}

// this is not (yet?) possible in PHP
function example3(int $param)
{
    ...
}
```

对于原生函数也是一样的。尽管PHP引擎和PHP-CPP库提供了可以指定函数参数类型为整数或者字符串类型，但是这种设置将来会完全废弃掉。也许，这种机制将来会有所完善（我们希望这样），但是暂时仅有指定参数类型为对象和数组才是有意义的。我们之所以在PHP-CPP库中，仍然要支持这种必要的类型指定，是因为在PHP核心引擎中是支持的。那么，我们也为将来的PHP版本做了准备。目前这些指定参数类型为整数类型的例子中是没有意义的。

