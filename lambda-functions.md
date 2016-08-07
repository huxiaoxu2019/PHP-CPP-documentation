# 匿名函数
C++和PHP都支持lambda函数，或者叫做匿名函数（在C++中通常称作lambda函数，而在PHP中通常称作匿名函数）。利用PHP-CPP库，你可以在不同语言之间传递匿名函数。在C++代码中调用PHP的匿名函数不是不可能的，同样也可以在PHP脚本中调用C++的lambda函数。

## 在C++中调用PHP匿名函数
首先，我们先举一个简单的PHP例子。在PHP中，你可以创建一个匿名函数，然后赋值给一个变量（或者传递给一个函数）。

```
<?php
// anonymous PHP function stored in the variable $f
$f = function($a, $b) {

    // return the sum of the parameters
    return $a + $b;
};

// pass the function to another function
other_function($f);

// or pass an anonymous function without assigning it to a variable
other_function(function() {

    // return the product of the parameters
    return $a * $b;
});

?>
```

对于PHP程序员来说，上面的代码不足为奇。如果想利用PHP-CPP实现如'other_function'的PHP函数，我们需要使用C++。正如在之前你看到的那些例子，C++函数会接收一个Php::Parameters的对象作为参数。这个参数其实是一个填充着Php::Value对象的std::vector。

```
#include <phpcpp.h>
/**
 *  Native function that is callable from PHP
 *
 *  This function gets one parameter that holds a callable anonynous
 *  PHP function.
 *
 *  @param  params      The parameters passed to the function
 */
void other_function(Php::Parameters &params)
{
    // make sure the function was really called with at least one parameter
    if (params.size() == 0) return nullptr;

    // this function is called from PHP user space, and it is called
    // with a anonymous function as its first parameter
    Php::Value func = params[0];

    // the Php::Value class has implemented the operator (), which allows
    // us to use the object just as if it is a real function
    Php::Value result = func(3, 4);

    // @todo do something with the result
}

/**
 *  Switch to C context, because the Zend engine expects the get_module()
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
        extension.add<other_function>("other_function");

        // return the extension details
        return extension;
    }
}
```

就是这么简单。相反也是可以的。假设我们有一个接收一个回调函数的函数。下面是一个PHP函数array_map()的简要版本的例子：

```
<?php
// function that iterates over an array, and calls a function on every
// element in that array, it returns a new array with every item
// replaced by the result of the callback
function my_array_map($array, $callback) {

    // initial result variable
    $result = array();

    // loop through the array
    foreach ($array as $index => $item) {

        // call the callback on the item
        $result[$index] = $callback($item);
    }

    // done
    return $result;
}
?>
```

假设我们使用C++中的lambda函数作为回调函数，来调用这个PHP函数。轻而易举：

```
#include <phpcpp.h>
/**
 *  Native function that is callable from PHP
 */
void run_test()
{
    // create the anonymous function
    Php::Function multiply_by_two([](Php::Parameters &params) -> Php::Value {

        // make sure the function was really called with at least one parameter
        if (params.empty()) return nullptr;

        // one parameter is passed to the function
        Php::Value param = params[0];

        // multiple the parameter by two
        return param * 2;
    });

    // the function now is callable
    Php::Value four = multiply_by_two(2);

    // a Php::Function object is a derived Php::Value, and its value can 
    // also be stored in a normal Php::Value object, it will then still 
    // be a callback function then
    Php::Value value = multiply_by_two;

    // the value object now also holds the function
    Php::Value six = value(3);

    // create an array
    Php::Value array;
    array[0] = 1;
    array[1] = 2;
    array[2] = 3;
    array[3] = 4;

    // call the user-space function
    Php::Value result = Php::call("my_array_map", array, multiply_by_two);

    // @todo do something with the result variable (which now holds
    // an array with values 2, 4, 6 and 8).
}

/**
 *  Switch to C context, because the Zend engine expects the get_module()
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
        extension.add<run_test>("run_test");

        // return the extension details
        return extension;
    }
}
```

在上面的例子中，我们将C++的lambda函数复制给一个Php::Function的对象。Php::Function类是继承于Php::Value类的。其中两者的唯一区别就是Php::Function的构造函数接收一个函数作为参数。除此之外，别无两样。事实上，可以跨过Php::Function，直接将C++函数赋值给Php::Value对象。但由于调用模糊（calling ambiguities）所以并不可行。

Php::Funtion的使用方法和Php::Value一样：你可以将其赋值给另一个Php::Value对象，也可以在调用PHP函数时，作为参数传递。正如上面的例子：我们利用C++函数'multiply_by_two'调用了用户自定义函数my_array_map()。















