---
title: Effective C++(2)--尽量用编译器取代预处理器
date: 2018-09-16 21:37:36
tags: C++
---

> 本文基本摘抄自《Effective C++》 item2

> 尽量以 const， enum，inline 替换 #define

## 1 以常量替换 #define

### 1.1 定义常量

```C++
#define ASPECT_RATIO 1.653
```

局限性：

- 不会被编译器看见，或者在编译器开始处理源码之前被编译器移走
- 运用此常量获得编译错误信息时，可能带来困惑，因为错误信息会提到 1.653 而不是 ASPECT_RATIO
- 在记号式调试器中，由于使用名称没有进入记号表导致难以追踪

解决：

> 用常量替换宏

```C++
const double AspectRatio = 1.653;
```

- 作为语言常量，AspectRatio 肯定会被编译器看到，当然就会进入到记号表内
- 对浮点常量而言，使用常量可能比使用 #define 导致较小量的码，因为预处理“盲目地将名称 ASPECT_RATIO 替换为 1.653”可能导致目标码出现多份 1.653。

下面说说两种特殊情况。

### 1.2 定义常量指针

由于常量定义式通常被放在头文件内（以便被不同的源码含入），因此有必要将指针（而不只是指针所指之物）声明为 const。

```C++
const char* const authorName = "Longhai";
```

详细见下一篇文章。**string 对象通常比他的前辈 char *-base 更合适：**

```c++
const std::string authorName = "Longhai";
```

### 1.3 定义class专属常量

为了将常量的作用域（scope）限制于 class 内，必须让它成为 class 的一个成员（member）；而为确保此常量至多只有一份实体，必须让它成为一个 static 成员：

```C++
Class GamePlayer {
private:
    static const int NumTurns = 5;	//常量声明式
    int scores[NumTurns];			//使用该常量
    ...
};
```

上面的是`NumTurns`的声明式而非定义式。

对于支持类内初始化的 C++ 编译器，这段代码可以编译通过。

但是较老的 C++ 编译器，可能不支持类内初始化，这样我们的静态常量，必须要在类外初始化。如下（在实现文件而非头文件中定义）：

```C++
const int GamePlayer::NumTurns;		//NumTurns的定义
```

由于 class 常量已在声明时获得初值，因此定义时不可以再设初值。

更老的编译器可能不支持 static 成员在其声明式上获得初值：

```C++
Class GamePlayer {
private:
    static const int NumTurns = 5;	//常量声明式
    # int scores[NumTurns];			//使用该常量
    ...
};
const int GamePlayer::NumTurns = 5;		//NumTurns的定义
```

但是当在 class 编译期间需要一个 class 常量值，例如数组声明式（编译器坚持必须在编译期间知道数组的大小）。此时可以使用`the enum hack`方法：

```C++
Class GamePlayer {
private:
    enum { NumTurns = 5 };			//令 NumTurns 成为 5 的一个记号名称
    int scores[NumTurns];			//使用该常量
    ...
};
```

- `enum hack`的行为某些方面比较像 #define 而不是 const，譬如取一个 const 的地址是合法的，但取一个 enum 的地址是不合法的，而取一个 #define 的地址通常（？为什么是通常）也不合法。
- 使用`enum hack`和 #define 一样不会导致 “不必要的内存分配”。 
- `enum hack`是模板元编程的一项基本技术，大量的代码在使用它。当你看到它时，你要认识它。 

>  除非你处理的是主要具有历史意义的编译器(即1995年之前编写的编译器)，否则你不应该使用`enum hack`。尽管如此，知道它是什么样子还是值得的，因为在那些早期简单的时代，代码中遇到它并不少见。 

## 2 实现宏

```C++
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
```

- 无论何时，**必须记住为宏中的所有实参加上小括号**

- 即使为所有实参加上小括号，也有可能有问题：

  ```C++
  int a = 5, b = 0;
  CALL_WITH_MAX(++a, b);		//a被累加2次
  CALL_WITH_MAX(++a, b+10)	//a被累加1次
  ```

> template inline 函数 -- 同时获取宏带来的效率以及一般函数的所有可预料行为和类型安全性。

```c++
/* 由于我们不知道T是什么，所以采用 pass by reference-to-const. */
template<typename T>
inline void callWithMax(const T& a, const T& b) {
    f(a > b ? a : b);
}
```

## 3 总结

- 有了 consts、enums 和 inlines，我们对预处理器（**尤其是 #define**）的需求降低了，但并非完全消除。

- `#include`仍然是必需品，而`#ifdef / #ifndef`也继续扮演控制编译的重要角色。

- 对于单纯变量，最好以 const 对象或 enums 替换 #defines
对于形式函数的宏（macros），最好改用 inline 函数替换 #defines」