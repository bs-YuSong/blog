---
title: Swift4.0.3学习疑惑
tags:
categories:
---

1. Tuple和Struct有很多相似的地方，Apple推荐短期存在用tuple，长期存在用struct，事实上struct的功能更强大，甚至可以实现方法，而tuple更灵活，创建一个tuple更简单，但是除此之外，还有其他的原因吗？

2. Swift中数据类型的强转，其实就是通过当前数据初始化一个新类型的数据

3. 只有Optional类型可以被赋值为nil

4. Error handling怎么使用

5. tuple中的数据类型如果完全相同，且实现了Camparable协议，则tuple也可以比较

6. String是值类型，数据类型南峰子的博客里好像提到过，印象中有：值类型、引用类型、计算类型

7. 统计一下Swift中的基础协议，例如equatable， comparable， collection, Substring, StringProtocol, Sequence, MutableCollection, Hashable, RawRepresentable等等

8. substring没有自己的存储空间，string有自己的存储空间

9. Swift中的string相等，要符合语义上的相等，如果语义不同，哪怕看起来是一样的，也不符合相等的条件。

10. Struct 和 Class还有一个重要的区别就是，Struct应该是值类型，而Class是引用类型

11. 函数做参数时，函数的默认值，arguments label等等特性都不再，所有的参数函数都默认使用参数名缺省模式，默认值失效，但是可变数量的参数依然有效

12. operator methods 这个的具体用法

13. Swift的闭包逃逸和OC的循环引用再确认一遍概念

14. 什么事First Class，这样区分有什么好处

15. rawValue和associated value的区别

16. 