# 调用PHP函数
首先，让我们先来明确一件事情。那就是，相对于运行PHP代码来说，运行原生代码是更快的。所以，一旦当你的C++函数或方法调用后，参数将转换成原生的变量，并且你将飞快地运行你自己的算法。同时，从那以后你将不愿意去调用其它PHP函数。

当然，如果你想调用PHP函数，无论这个函数已经定义在PHP引擎中，还是定义在扩展中，或者甚至是PHP应用层定义的函数，都是可以的。

```
#include <phpcpp.h>
#include <iostream>

/**
 *  Native function that is callable from PHP
 *
 *  This function gets two parameters: an associative array and a callback.
 *  It does not do anything meaningful, it is just a demonstration function.
 *
 *  @param  params      The parameters passed to the function
 */
void example_function(Php::Parameters &params)
{
    // first parameter is an array
    Php::Value array = params[0];

    // call the PHP array_keys() function to get the parameter keys
    std::vector<std::string> keys = Php::array_keys(array);

    // loop through the keys
    for (auto &key : keys) 
    {
        // output key
        Php::out << "key: " << key << std::endl;
    }

    // call a function from user space
    Php::Value data = Php::call("some_function", "some_parameter");

    // create an object (this will also call __construct())
    Php::Object time("DateTime", "now");

    // call a method on the datetime object
    Php::out << time.call("format", "Y-m-d H:i:s") << std::endl;

    // second parameter is a callback function
    Php::Value callback = params[1];

    // call the callback function
    callback("some","parameter");

    // in PHP it is possible to create an array with two parameters, the first
    // parameter being an object, and the second parameter should be the name
    // of the method, we can do that in PHP-CPP too
    Php::Array time_format({time, "format"});

    // call the method that is stored in the array
    Php::out << time_format("Y-m-d H:i:s") << std::endl;
}

/**
 *  Switch to C context, because the Zend engine expects get get_module()
 *  to have a C style function signature
 */
extern "C" {
    /**
     *  Startup function that is automatically called by the Zend engine
     *  when PHP starts, and that should return the extension details
     *  @return void*
     */
    PHPCPP_EXPORT void *get_module() 
    {
        // the extension object
        static Php::Extension extension("my_extension", "1.0");

        // add the example function so that it can be called from PHP scripts
        extension.add("example_function", example_function);

        // return the extension details
        return extension;
    }
}
```

从上面的例子中，你可以看出相对于PHP而言，利用C++在调用函数上会有所不同。第一个不同点就是调用Php::array_keys()函数。在PHP-CPP内部有着所有PHP重要函数的长列表，而且你可以直接从扩展中直接调用这些函数。Php::array_keys就是其中一个。

许多内置函数，比如Php::array_keys()，会返回Php::Value类的实例。然而，在这个例子中，你可以看出我们直接利用std::vector接收返回值。这是可以的，因为在PHP::Value类内部有着隐身转换运算符（implicit casting operator），她会自动地将这个对象转换成std::vector类型。

当然，并不是所有的PHP函数可以像内置函数一样来调用。用户自定义函数，或者其它可选扩展中的函数，PHP-CPP类库并不会自动将其转换。这些函数可以通过Php::call()方式来调用。你必须提供调用的函数名称，以及一个可选的参数列表。

这个Php::Object类（来源于Php::Value类）可以用来创建对象，也可以用来隐似的调用对象。为了调用对象的某一方法，你可以利用Php::Value::call()方法，正如例子中，它可以用来调用PHP函数DateTime::format()。

在PHP脚本中，你可以创建一个填充两个成员的数组：一个对象和一个方法名称的字符串。这个数组可以当做一个常规函数来用。在C++中同样可以，正如我们例子中所展示的“time_format”变量。







