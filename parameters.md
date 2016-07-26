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

