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

