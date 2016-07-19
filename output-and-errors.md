# 输出与错误
你可以使用C++中常规的流类库，比如一些常规的操作符 << 和特殊的功能 std::endl。然而，并不推荐使用'std::out'和'std::err'。

当PHP以Web服务模块运行的时候，使用'std::out'的输出信息会直接输出到当初启动PHP的进程的终端中。在生产环境中，这种终端不是活跃状态的，所以任何以'std::out'发送出去的信息都会被丢掉。因此，在以Web模块方式运行的扩展中，使用'std::out'是无法进行的（no-go）。但是，尽管PHP将以命令行脚本（Cli script）的方式（'std::out'可以正常使用），仍然不建议你直接使用'std::out'。使用'std::out'会被所有的已经在PHP用户脚本中定义的输出机制忽略（output handler）。

可以利用PHP-CPP库中提供的'Php::out' stream包来取而代之。这个'Php::out'变量就是众所周知的'std::ostream'类的一个实例，它会将所有的缓冲区中的输出内容指向PHP中。事实上，它和PHP中的echo函数的作用一样。

'Php::out'是'std::ostream'常规类的一个实例。这就说明了它使用了一个需要清理的内部缓冲区。当显示的写出'std::endl'或'std::flush'时，它会自动清空内部缓冲区。

```
/**
 *  Example function that shows how to generate output
 */
void example()
{
    // the C++ equivalent of the echo() function
    Php::out << "example output" << std::endl;

    // generate output without a newline, and ensure that it is flushed
    Php::out << "example output" << std::flush;

    // or call the flush() method
    Php::out << "example output";
    Php::out.flush();

    // just like all PHP functions, you can call the echo() function 
    // from C++ code as well
    Php::echo("Example output\n");
} 
```

## 错误、报警和警告
当你想注册一个PHP的错误函数时（在C++中等同于trigger_error()函数），你可以使用'Php::error'、'Php::notice'、'Php::warning'和'Php::deprecated'流。这些也是'std::ostream'类的实例。

```
/**
 *  Example function that shows how to generate output
 */
void example()
{
    // generate a PHP notice
    Php::notice << "this is a notice" << std::flush;

    // generate a PHP warning
    Php::warning << "this is a warning" << std::flush;

    // inform the user that a call to a deprecated function was made
    Php::deprecated << "this method is deprecated" << std::flush;

    // generate a fatal error
    Php::error << "fatal error" << std::flush;

    // this code will no longer be called
    Php::out << "regular output" << std::endl;
}  
```

在上面的例子中，你会看到我们使用了'std::flush'而不是'std::endl'。这么做的原因有两个：它追加了一个新行，同时冲刷了缓冲区。对于错误（errors）、提醒（notice）和警告（warnings）这些情况，我们并不需要最佳一个新行，但是为了真正的生成输出内容，需要冲刷缓冲区。

当使用'Php::error'流时有个奇怪的事情：当你刷新缓冲区时，PHP脚本会以一个致命错误（fatal error）结束，同时C++也立即结束。在底层，PHP引擎会跳转到Zend引擎内部底层。在上面的例子里面，“Php::out << "regular output" << std::endl;”这条语句永远不会被调用。

这些情况是不同寻常的，同时（对于我们来说）和正规的软件工程规则是冲突的。一个生产输出的函数本不应该抛出异常。如果一段代码看起来是正常的，那么执行起来本应该也是正常的，而不是做出一些反常行为，比如跳出当前的调用栈。因此我们不建议使用'Php::error' - 除非在比较极端的情况下使用它。

