---
title: C++11多线程-【2】线程的join和detach
date: 2018-10-28 19:47:34
tags: C++
---

> 本文翻译自 [C++11 Multithreading – Part 2: Joining and Detaching Threads](https://thispointer.com/c11-multithreading-part-2-joining-and-detaching-threads/)

本文介绍线程对象 std::thread 的 joining 和 detaching。

## 使用 std::thread::join() 进行线程的 joining

一旦一个线程开始之后，另一个线程可以等待此线程结束。

需要等待的线程可以调用 std::thread 的 join() 函数来实现上述功能。

```c++
std::thread th(funcPtr);
//some code
th.join();
```

下面看一个简单的例子。

假设主线程需要启动 10 个工作线程，开始这些线程之后，主线程需要等待这些线程结束。等 joining 所有的线程之后，主程序继续运行：

```c++
#include <iostream>
#include <thread>
#include <algorithm>

class WorkerThread {
public:
    void operator()() {
        std::cout << "Worker Thread " << std::this_thread::get_id() << "is Executing" << std::endl;
    }    
};

int main(void) {
    std::vector<std::thread> threadList;
    for (int i = 0; i < 10; i++) {
        threadList.push_back(std::thread(WorkerThread()));
    }
    // 等待所有的工作线程结束，即对每一个 std::thread 对象调用 join() 函数
    std::cout << "wait for all the worker thread to finish" << std::endl;
    std::for_each(threadList.begin(), threadList.end(), std::mem_fn(&std::thread::join));
    // 下面这条语句是最后打印的
    std::cout << "Exiting from Main Thread" << std::endl;
    
    return 0;
}
```

## 使用 std::thread::detach() 进行线程的 detaching

detached 线程也被称为守护/后台进程。对线程进行 detached（翻译成分离？）操作，需要使用对 std::thread 对象调用 std::detach() 。

```c++
std::thread th(funcPtr);
th.detach();
```

## 小心调用线程的 detach() 和 join()

**案例1：不要在没有相关执行线程的 std::thread 对象上调用 join() 或 detach()**

```c++
std::thread threadObj( (WorkerThread()) );
threadObj.join();
threadObj.join(); // 会导致程序中断
```

对线程对象调用 join() 函数，则当这个 join() 返回时，std::thread 对象就没有了与之相关联的线程。对这样的 std::thread 对象再次进行 join() 函数调用时，就会导致程序中断。

同理，调用 detach() 一样会使得 std::thread 对象不与其他任何的线程关联。在这种情况下，再次调用 detach() 会中断程序。

```c++
std::thread threadObj( (WorkerThread()) );
threadObj.detach();
threadObj.detach(); // 会导致程序中断
```

因此，在调用 join() 或者 detach() 之前，我们每次都应该检查线程是否是 join-able 的。

```c++
    std::thread threadObj( (WorkerThread()) );
    if(threadObj.joinable())
    {
        std::cout<<"Detaching Thread "<<std::endl;
        threadObj.detach();
    }
    if(threadObj.joinable())    
    {
        std::cout<<"Detaching Thread "<<std::endl;
        threadObj.detach();
    }
    
    std::thread threadObj2( (WorkerThread()) );
    if(threadObj2.joinable())
    {
        std::cout<<"Joining Thread "<<std::endl;
        threadObj2.join();
    }
    if(threadObj2.joinable())    
    {
        std::cout<<"Joining Thread "<<std::endl;
        threadObj2.join();
    }
```

**案例2：永远不要忘记在有关联的执行线程的 std::thread 对象上调用 join 或者 detach**

假如对一个有关联执行线程的 std::thread 对象并没有调用 join() 也没有调用 thread()，则在对象的析构的过程中会中断程序。例：

```c++
#include <iostream>
#include <thread>
#include <algorithm>

class WorkerThread {
public:
    void operator()() {
        std::cout << "Worker Thread " << std::endl;
    }
};

int main(void) {
    std::thread threadObj((WorkerThread()));
    // 程序会中断，因为既没有调用 join() 也没有调用 detach()
    return 0;
}
```

同样在异常情况下，我们也不能忘记调用 join() 或者 detach()。

为了防止这种情况，我们应该使用 ESOURCE ACQUISITION IS INITIALIZATION (RAII)。例：

```c++
#include <iostream>
#include <thread>

class ThreadRAII {
    std::thread &m_thread;
    public:
    ThreadRAII(std::thread &threadObj) : m_thread(threadObj) {}
    ~ThreadRAII() {
        // 假如 joinable 则调用 detach
        if (m_thread.joinable()) {
            m_thread.detach();
        }
    }
};

void thread_function() {
    for (int i = 0; i < 10000; i++) {
        std::cout << "thread_function executing" << std::endl;
    }
}

int main(void) {
   	std::thread threadObj(thread_function);
    
    // 注释掉这行，程序会 crash
    ThreadRAII wrapperObj(threadObj);
   	return 0;
}
```

