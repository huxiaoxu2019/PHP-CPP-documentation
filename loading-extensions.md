# PHP如何加载扩展
也许你知道，在类unix系统中，原生的PHP扩展需要编译成.so文件，而在Windows系统中，会编译成.dll文件，编译之后，全局php.ini文件会包含着系统中可用的扩展列表。也就是说，创建一个扩展并让PHP加载的话，需要生成一个.so或者.dll文件，然后更新PHP配置文件。

## 启动函数get_module()
在讲述如何创建你自己的扩展之前，我们先来看看PHP是如何加载扩展的。当PHP启动时，它会从配置目录下加载*.ini配置文件，并且读取配置文件中类似"extension=name.so"的每一行，相应地打开每一个库调用其中的"get_module()"方法。因此，每一个扩展（你的扩展也不例外）库都需要定义且实现”get_module()"函数。这个函数会在PHP加载扩展库后调用（and thus way before pageviews are handled），然后需要返回一个内存地址，指向着包含在扩展中有效编译的函数、类、变量和常量等信息构成的结构体。

get_module()函数返回的结构体类型已经在Zend引擎的头文件中定义过了，但它却没有很完善的文档，并且相当复杂。幸运的是，PHP-CPP会搞定这一切，因为它提供了一个可以替代这个结构体的Extension类。

```
#include <phpcpp.h>

/**
 *  tell the compiler that the get_module is a pure C function
 */
extern "C" {

    /**
     *  Function that is called by PHP right after the PHP process
     *  has started, and that returns an address of an internal PHP
     *  strucure with all the details and features of your extension
     *
     *  @return void*   a pointer to an address that is understood by PHP
     */
    PHPCPP_EXPORT void *get_module() 
    {
        // static(!) Php::Extension object that should stay in memory
        // for the entire duration of the process (that's why it's static)
        static Php::Extension myExtension("my_extension", "1.0");

        // @todo    add your own functions, classes, namespaces to the extension

        // return the extension
        return myExtension;
    }
}
```

正如你看到的上面的例子，是一个对于get_module()的简单实现。或多或少，每一个PHP扩展都会以类似的方式实现这个函数，并且它是每一个扩展启动的要点。这里有一些点需要特别注意。对于启动来说，唯一的头文件就是phpcpp.h。如果你使用PHP-CPP来开发扩展，那么你并不需要包含那些复杂的，无组织的，而且没有文档的Zend引擎中的头文件 - 所有你需要做的就是包含这一个PHP-CPP库的头文件phpcpp.h。如果你坚持不这么做的话，当然，你也可以包含那些PHP引擎中的核心头文件 - 但是真的没必要。PHP-CPP已经处理了PHP引擎中的内部细节，并且提供了一些易使用的API。





