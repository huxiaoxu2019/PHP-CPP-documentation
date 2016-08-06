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











