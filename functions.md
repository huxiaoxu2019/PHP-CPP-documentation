# 增加原生函数
向PHP扩展中增加原生函数和（或者）类是非常有用的，且这些函数和（或者）类可以直接从PHP脚本中调用。对于函数来说非常简单。只要你按照下面的方式书写C++函数，那么你就可以直接从PHP脚本中调用它。

```
void example1();
void example2(Php::Parameters &params);
Php::Value example3();
Php::Value example4(Php::Parameters &params);
```

在上述的代码中有两个重要的PHP-CPP类，一个是Php::Value，一个是Php::Parameters。Php::Value类非常强大，它与PHP的变量作用一样：它可以容纳几乎任何值（整形、浮点型小数、字符串，当然也有索引和关联数组和对象）。Php::Parameters类以数组或向量的形式来接收传递到函数的所有变量是再适合不过了。后面我们会对这两个类进行深入的讨论。

