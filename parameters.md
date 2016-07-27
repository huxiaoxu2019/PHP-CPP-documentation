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

回到我们的例子中来，下面的是在PHP中调用函数的一些例子。
```
<?php
// correct call, parameters are numeric and two objects of the right type
example(12, new ExampleClass(), new OtherClass());

// also valid, first parameter is not numeric but an array, but the
// Zend engine does not check this even though it was specified
example(array(1,2,3), new ExampleClass(), new OtherClass());

// invalid, wrong number of parameters
example(12, new ExampleClass());

// invalid, wrong objects
example(12, new DateTime(), new DateTime());

// invalid, "x" and "z" are no objects
example("x", "y", "z");
```

由于PHP引擎注册了错误机制，当调用函数时传错了参数，PHP会抛出相应的错误，同时不会真正的调用到原生函数。

## Php::ByVal类的延展
Php::ByVal类可以通过两种方式来实例化。我们来看下C++头文件中的第一种构造器。

```
/**
 *  Constructor
 *  @param  name        Name of the parameter
 *  @param  type        Parameter type
 *  @param  required    Is this parameter required?
 */
ByVal(const char *name, Php::Type type, bool required = true);
```

第一参数必须是一个参数名称。这看起来有些奇怪，因为PHP语言并不像其他语言一样支持名称变量（）named variables）的概念。事实上，只有当函数调用方式有误抛出错误时，这个名称用于参数错误信息。

Php::Type参数更加有趣。支持下面的类型：

```
Php::Type::Null
Php::Type::Numeric
Php::Type::Float
Php::Type::Bool
Php::Type::Array
Php::Type::Object
Php::Type::String
Php::Type::Resource
Php::Type::Constant
Php::Type::ConstantArray
Php::Type::Callable
```

事实上，只有当你指定参数类型为Php::Type::Array或者Php::Type::Object时才起作用，因为其它类型在PHP底层引擎中不会有所区分。

最后一个参数（你还记得那个叫做'required'的参数吗？）可以用来设置参数是否可选。如果设置为true，当调用这个函数时但没有传递指定的参数，PHP引擎会注册一个错误。这个参数只对第二个开始的参数起作用，因为（当然）第一个参数不可以是可选的，后面的参数是可以的。

如果你写一个以某个对象为参数的函数的话，可以使用下面的第二种构造方法：

```
/**
 *  Constructor
 *  @param  name        Name of the parameter
 *  @param  classname   Name of the class
 *  @param  nullable    Can it be null?
 *  @param  required    Is this parameter required?
 */
ByVal(const char *name, const char *classname, bool nullable = false, bool required = true);
```

这个可选的构造器同样有'name'和'required'参数，但是前面可用的Php::Type参数被'classname'和'nullable'参数替代了。当一个函数接收一个指定类型的对象作为参数时，可以用这个构造器。举个例子，看看下面的PHP代码：

```
<?php
function example1(DateTime $time) { ... }
function example1(DateTime $time = null) { Php::Value time = params[0]; ... }
```

相应在C++中的代码如下：

```
#include <phpcpp.h>;

void example1(Php::Parameters &amp;params) { Php::Value time = params[0]; ... }
void example2(Php::Parameters &amp;params) { Php::Value time = params[0]; ... }

extern "C" {
    PHPCPP_EXPORT void *get_module() {
        static Php::Extension myExtension("my_extension", "1.0");
        myExtension.add<example1>("example1", { Php::ByVal("time", "DateTime", false); });
        myExtension.add<example2>("example2", { Php::ByVal("time", "DateTime", true); });
        return myExtension;
    }
}
```


