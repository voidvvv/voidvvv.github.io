---
title: c++_template
date: 2025-06-30T18:04:57+08:00
tags:
- C++
---

c++ 中的模板，就是类似于java中的泛型，但是与java泛型又不太一样。作为强类型语言，Java中的泛型由于无法确定具体class，并且由于泛型擦除的存在，使得每个泛型在jvm中其实都是以Object的形式存储的。但是c++的模板确是在编译器确定类型，由此我们就可以在代码中对泛型进行预期之内的操作.

<!--more-->

```c++
template <typename Iter, typename Func>
void processElements(Iter begin, Iter end, Func processor)
{
    for (auto it = begin; it != end; ++it)
    {
        processor(*it); // 调用传入的函数对象处理当前元素
    } // 返回一个默认构造的对象
}
```
上面就是c++定义模板方法的方式。我们可以看到，``Func processor`` 这个泛型``Func``我们其实不知道具体类型。但是由于c++编译器会在编译器就将其类型确定，所以我们可以在用到这个方法的地方，将``Func processor``传入成为一个方法引用。

```c++
    std::vector<int> vec = {1, 2, 3, 4, 5};
    processElements(vec.begin(), vec.end(), [](int x) { std::cout << x << " "; });
    std::cout << std::endl;
```

这样，c++编译器便会在编译期将检查参数类型是否与内部使用符合。若不符合，则会在编译期报错导致编译失败.