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


















