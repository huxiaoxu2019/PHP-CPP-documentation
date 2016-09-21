# 类与对象
现在，我们终于进入到“正题”了。由于C++和PHP都是面向对象的语言，所以我们可以创建类与对象。PHP-CPP库提供了能够将这两种语言联合起来的工具，使PHP可以调用C++中的类库。

可惜的是（当然，这也是合理的），并不是每个C++类库可以直接转换成PHP类库。这需要费一些力气（当然并不麻烦）。首先，你要确定你的类库是继承于Php::Base类库的。其次，当你想将你的类库加入到扩展中时，必须确保所有期望在PHP中调用的方法被特殊处理。

```
#include <phpcpp.h>

/**
 *  Counter class that can be used for counting
 */
class Counter : public Php::Base
{
private:
    /**
     *  The initial value
     *  @var    int
     */
    int _value = 0;

public:
    /**
     *  C++ constructor and destructor
     */
    Counter() = default;
    virtual ~Counter() = default;

    /**
     *  Update methods to increment or decrement the counter
     *  Both methods return the NEW value of the counter
     *  @return int
     */
    Php::Value increment() { return ++_value; }
    Php::Value decrement() { return --_value; }

    /**
     *  Method to retrieve the current counter value
     *  @return int
     */
    Php::Value value() const { return _value; }
};

/**
 *  Switch to C context to ensure that the get_module() function
 *  is callable by C programs (which the Zend engine is)
 */
extern "C" {
    /**
     *  Startup function that is called by the Zend engine 
     *  to retrieve all information about the extension
     *  @return void*
     */
    PHPCPP_EXPORT void *get_module() {
        // create static instance of the extension object
        static Php::Extension myExtension("my_extension", "1.0");

        // description of the class so that PHP knows which methods are accessible
        Php::Class<Counter> counter("Counter");
        counter.method<&Counter::increment> ("increment");
        counter.method<&Counter::decrement> ("decrement");
        counter.method<&Counter::value>     ("value");

        // add the class to the extension
        myExtension.add(std::move(counter));

        // return the extension
        return myExtension;
    }
}
```

先来说说这里的编程习惯 - 类名称总是首字母大写，变量名称首字母小写。每个类库都有一个析构函数，并且总是有Virtual特性。这就是编程习惯，我们的习惯 - 所以你不必必须遵循。

第一个话题。上面的例子展示了一个非常简单的Counter类库，有三个方法：increment()，decrement()和value()。其中两个更新操作函数返回操作后的值，value()函数返回当前的值。

如果你想在PHP调用类方法，那么必须满足如下的任意一种形式。这些特性同样也是[可导出文本函数](http://www.php-cpp.com/documentation/functions)的特性，除了那些有常亮版本和那些有非常量函数的。

```
// signatures of supported regular methods
void        YourClass::example1();
void        YourClass::example2(Php::Parameters &params);
Php::Value  YourClass::example3();
Php::Value  YourClass::example4(Php::Parameters &params);
void        YourClass::example5() const;
void        YourClass::example6(Php::Parameters &params) const;
Php::Value  YourClass::example7() const;
Php::Value  YourClass::example8(Php::Parameters &params) const;
```

方法和[常规函数](http://www.php-cpp.com/documentation/functions)的工作方式是一样的，唯一不同的是在方法中可以（当然）访问当前实例的成员变量。

为了能够在PHP中使用C++中的类库，必须在get_module()函数中将其加入到扩展对象中。这时Php::Class模板类库就派上用场了。其中模板参数是你的实现类。这样，当在PHP中调用“new”操作符时，Php::Class类库才能识别出需要实例化哪一个类。

其中Php::Class的构造函数接收一个字符串作为参数，值为PHP中的类名称。正如上面的例子，我们可以利用Php::method这个方法来将方法注册到该类中，并得以在PHP中调用。你也许会发现，在上面的例子中我们使用C++11 std::move()方法来将该类加入到扩展中。相对于直接拷贝，这么做更有效率。

## 方法参数
类的成员方法和函数很相似，指定类成员方法参数的方式可以参照指定函数参数的方式，比如使用Php::ByVal和Php::ByRef类库。

```
#include <phpcpp.h>

/**
 *  Counter class that can be used for counting
 */
class Counter : public Php::Base
{
private:
    /**
     *  The internal value
     *  @var    int
     */
    int _value = 0;

public:
    /**
     *  C++ constructor and destructor
     */
    Counter() {}
    virtual ~Counter() {}

    /**
     *  Increment operation
     *  This method gets one optional parameter holding the change
     *  @param  int     Optional increment value
     *  @return int     New value
     */
    Php::Value increment(Php::Parameters &params) 
    { 
        return _value += params.empty() ? 1 : (int)params[0];
    }

    /**
     *  Decrement operation
     *  This method gets one optional parameter holding the change
     *  @param  int     Optional decrement value
     *  @return int     New value
     */
    Php::Value decrement(Php::Parameters &params) 
    { 
        return _value -= params.empty() ? 1 : (int)params[0]; 
    }

    /**
     *  Method to retrieve the current value
     *  @return int
     */
    Php::Value value() const 
    { 
        return _value; 
    }
};

/**
 *  Switch to C context to ensure that the get_module() function
 *  is callable by C programs (which the Zend engine is)
 */
extern "C" {
    /**
     *  Startup function that is called by the Zend engine 
     *  to retrieve all information about the extension
     *  @return void*
     */
    PHPCPP_EXPORT void *get_module() {
        // create static instance of the extension object
        static Php::Extension myExtension("my_extension", "1.0");

        // description of the class so that PHP knows which methods are accessible
        Php::Class<Counter> counter("Counter");

        // register the increment method, and specify its parameters
        counter.method<&Counter::increment>("increment", { 
            Php::ByVal("change", Php::Type::Numeric, false) 
        });

        // register the decrement, and specify its parameters
        counter.method<&Counter::decrement>("decrement", { 
            Php::ByVal("change", Php::Type::Numeric, false) 
        });

        // register the value method
        counter.method<&Counter::value>("value", {});

        // add the class to the extension
        myExtension.add(std::move(counter));

        // return the extension
        return myExtension;
    }
}
```

正如上面的例子，我们修改了第一个例子。其中increment和decrement接收一个可选的参数'change'，类型为数字类型，代表当前counter属性的变化。注意这个参数是可选的 - 所以在该方法的内部实现中，我们需要检查该值是否大于0，以避免参数严重的段错误（segementation faults）。

```
<?php
$counter = new Counter();
$counter->increment(5);
$counter->increment();
$counter->decrement(3);
echo($counter->value()."\n");
?>
```

上面PHP脚本例子中，使用了原生的counter类库。输出即为（正如你期望的那样）3。

在这个例子中，我们并没有展示如何使用Php::ByRef类库，但是它的用法和functions很类似，所以我们认为这个例子并不重要（当然，我们也不是使用引用传递参数的忠实粉丝）。

## 静态方法

同样，也支持静态方法。静态方法即一个没有访问'this'权限的方法。在C++中，类的静态方法和常规函数是一致的，常规函数也没有访问'this'的权限。其中，C++方法和C++函数的唯一区别在于编译时：编译器允许静态方法访问私有变量（private data）。然而，静态方法和常规函数的签名是一致的。

```
#include <phpcpp.h>

/**
 *  Regular function
 *
 *  Because a regular function does not have a 'this' pointer,
 *  it has the same signature as static methods
 *
 *  @param  params      Parameters passed to the function
 */
void regularFunction(Php::Parameters &params)
{
    // @todo add implementation
}

/**
 *  A very simple class that will <b>not</b> be exported to PHP
 */
class PrivateClass
{
public:
    /**
     *  C++ constructor and destructor
     */
    PrivateClass() = default;
    virtual ~PrivateClass() = default;

    /** 
     *  Static method
     *
     *  A static method also has no 'this' pointer and has
     *  therefore a signature identical to regular functions
     *
     *  @param  params      Parameters passed to the method
     */
    static void staticMethod(Php::Parameters &params)
    {
        // @todo add implementation
    }
};

/**
 *  A very simple class that will be exported to PHP
 */
class PublicClass : public Php::Base
{
public:
    /**
     *  C++ constructor and destructor
     */
    PublicClass() = default;
    virtual ~PublicClass() = default;

    /** 
     *  Another static method
     *
     *  This static has exactly the same signature as the
     *  regular function and static method that were mentioned
     *  before
     *
     *  @param  params      Parameters passed to the method
     */
    static void staticMethod(Php::Parameters &params)
    {
        // @todo add implementation
    }
};

/**
 *  Switch to C context to ensure that the get_module() function
 *  is callable by C programs (which the Zend engine is)
 */
extern "C" {
    /**
     *  Startup function that is called by the Zend engine 
     *  to retrieve all information about the extension
     *  @return void*
     */
    PHPCPP_EXPORT void *get_module() {
        // create static instance of the extension object
        static Php::Extension myExtension("my_extension", "1.0");

        // description of the class so that PHP knows which methods are accessible
        Php::Class<PublicClass> myClass("MyClass");

        // register the PublicClass::staticMethod to be a
        // static method callable from PHP
        myClass.method<&PublicClass::staticMethod>("static1");

        // regular functions have the same signatures as 
        // static methods. So nothing forbids you to register
        // a normal function as static method too
        myClass.method<regularFunction>("static2");

        // and even static methods from completely different
        // classes have the same function signature and can
        // thus be registered
        myClass.method<&PrivateClass::staticMethod>("static3");

        // add the class to the extension
        myExtension.add(std::move(myClass));

        // In fact, because a static method has the same signature
        // as a regular function, you can also register static
        // C++ methods as regular global PHP functions
        myExtension.add("myFunction", &PrivateClass::staticMethod);

        // return the extension
        return myExtension;
    }
}
```

也许你会问，到底有多实用？我们建议您保持您的代码干净、简单并且可维护，同时仅注册已经添加到C++类中的PHP静态方法。但是，C++却不限于此。让我们用一个例子来说明如何调用静态方法。

```
<?php
// this will call PublicClass::staticMethod()
MyClass::static1();

// this will call PrivateClass::staticMethod()
MyClass::static2();

// this will call regularFunction()
MyClass::static3();

// this will also call PrivateClass::staticMethod()
myFunction();
?>
```
