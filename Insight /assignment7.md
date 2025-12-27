# assignment7

这次作业主要是看大家对于智能指针的了解，实现类似于 `unique_ptr` 的功能的复现，但是没有很全面的用到智能指针的相关内容，大家可以课后找个例子再多学一下智能指针的相关知识。

### 一、为什么需要智能指针？

首先要明确：**智能指针的核心目的是解决手动管理动态内存的痛点**。C++ 中用`new`分配的堆内存，必须用`delete`手动释放，新手很容易踩坑：

- 忘记`delete` → 内存泄漏；
- 重复`delete` → 未定义行为（程序崩溃）；
- 析构前抛出异常 → 内存泄漏；
- 多个指针指向同一块内存，析构时互相干扰（比如你作业里`unique_ptr`拷贝的问题）。

智能指针是**封装了原生指针的模板类**，遵循**RAII（资源获取即初始化）** 原则：

- 当智能指针对象**创建**时，获取内存资源；
- 当智能指针对象**超出作用域 / 析构**时，自动释放内存（无需手动`delete`）；
- 语义清晰，明确内存的 “所有权”（谁负责释放）。

### 二、C++ 核心智能指针类型

C++11 及以后标准库提供了三种核心智能指针（都在`<memory>`头文件中），其中你作业实现的`unique_ptr`是最基础、最常用的。

#### 1. unique_ptr（独占智能指针）

##### 核心特点

- **独占所有权**：同一时间，只有一个`unique_ptr`能指向同一块内存；
- **不支持拷贝**：禁用了拷贝构造 / 拷贝赋值运算符（你作业里需要删除这两个函数）；
- **支持移动**：可以通过`std::move()`转移所有权（转移后原指针变为`nullptr`）；
- **开销极小**：几乎和原生指针一样快，无额外内存开销；
- **析构时自动释放**：超出作用域时调用`delete`释放内存。

##### 基础用法示例

```cpp
#include <memory>
#include <iostream>

int main() {
    // 1. 创建unique_ptr（优先用make_unique，C++14起支持）
    std::unique_ptr<int> ptr1 = std::make_unique<int>(10);
    // 等价于：std::unique_ptr<int> ptr1(new int(10));（不推荐，易漏释放）

    // 2. 访问数据（和原生指针一样）
    std::cout << *ptr1 << std::endl; // 输出10
    std::cout << ptr1->operator*() << std::endl; // 等价写法（作业里实现的operator*）

    // 3. 移动所有权（不能直接拷贝，必须用std::move）
    std::unique_ptr<int> ptr2 = std::move(ptr1); 
    // 此时ptr1 == nullptr，ptr2指向值为10的内存

    // 4. 重置指针（手动释放/更换指向）
    ptr2.reset(); // 释放内存，ptr2变为nullptr
    ptr2.reset(new int(20)); // 指向新的内存（值为20）

    return 0;
} // ptr2超出作用域，自动释放内存（调用delete）
```

##### 适用场景

- 单个对象 / 资源的独占管理（比如你作业里的链表节点）；
- 函数返回动态分配的对象（避免返回原生指针导致内存泄漏）；
- 容器中存储独占对象（`std::vector<std::unique_ptr<T>>`）。

#### 2. shared_ptr（共享智能指针）

##### 核心特点

- **共享所有权**：多个`shared_ptr`可以指向同一块内存；

- 引用计数

  ：内部维护一个 “引用计数器”（堆上的整数）：

  - 拷贝`shared_ptr` → 计数 + 1；
  - 析构 / 重置`shared_ptr` → 计数 - 1；
  - 当计数为 0 时，自动释放内存；

- **支持拷贝 / 移动**：拷贝会增加计数，移动不改变计数；

- **开销略大**：需要维护引用计数（原子操作，线程安全）。

##### 基础用法示例

```cpp
#include <memory>
#include <iostream>

int main() {
    // 1. 创建shared_ptr（优先用make_shared，更高效）
    std::shared_ptr<int> ptr1 = std::make_shared<int>(100);
    
    // 2. 拷贝：引用计数+1（此时计数=2）
    std::shared_ptr<int> ptr2 = ptr1;
    std::cout << "引用计数：" << ptr1.use_count() << std::endl; // 输出2

    // 3. 访问数据（和unique_ptr一致）
    *ptr2 = 200;
    std::cout << *ptr1 << std::endl; // 输出200（共享同一块内存）

    // 4. 重置：计数-1（ptr1变为nullptr，计数=1）
    ptr1.reset();
    std::cout << "引用计数：" << ptr2.use_count() << std::endl; // 输出1

    return 0;
} // ptr2超出作用域，计数-1=0，释放内存
```

##### 注意：循环引用问题

`shared_ptr`的最大坑是**循环引用**：两个`shared_ptr`互相指向，导致引用计数永远不为 0，内存泄漏。比如：

```cpp
struct Node {
    std::shared_ptr<Node> next; // 互相指向，计数永远为1
};

int main() {
    auto n1 = std::make_shared<Node>();
    auto n2 = std::make_shared<Node>();
    n1->next = n2;
    n2->next = n1;
    return 0; // 析构时n1和n2的计数都是1，内存泄漏！
}
```

#### 3. weak_ptr（弱引用智能指针）

##### 核心特点

- **弱引用**：指向`shared_ptr`管理的内存，但**不拥有所有权**；
- **不增加引用计数**：不会影响内存的释放；
- **需要 lock () 才能访问**：`lock()`会返回一个`shared_ptr`（若内存已释放，返回空）；
- **核心作用**：解决`shared_ptr`的循环引用问题。

##### 解决循环引用的示例

```cpp
struct Node {
    std::weak_ptr<Node> next; // 改用weak_ptr，不增加计数
};

int main() {
    auto n1 = std::make_shared<Node>();
    auto n2 = std::make_shared<Node>();
    n1->next = n2;
    n2->next = n1;
    return 0; // 析构时计数=0，内存正常释放！
}
```





好吧其实我有点懒了，大家可以看一下一下的内容跟 `README.md` 的内容一致，只是翻译了一下。

## 本章作业任务

1. `unique_ptr`类的私有成员部分
2. 构造函数 `unique_ptr(T* ptr)`
3. 支持空指针的构造函数 `unique_ptr(std::nullptr_t)`
4. 解引用运算符 `T& operator*()`
5. 常量版本的解引用运算符 `const T& operator*() const`
6. 成员访问运算符 `T* operator->()`
7. 常量版本的成员访问运算符 `const T* operator->() const`
8. 布尔类型转换运算符 `operator bool() const`

但另一方面，我们**仍然需要支持`unique_ptr`的移动操作**。回顾移动语义的知识点：移动操作可以让我们获取一个对象的资源所有权，且不需要进行昂贵的拷贝操作。移动`unique_ptr`是合法的，因为这能保证指针的 “独占性”—— 任何时刻，始终只有一个指针指向底层内存，我们只是改变了 “谁（哪个变量）拥有这块内存”。

为了实现以下三个核心目标 ——**自动释放内存**、**禁止拷贝**、**支持移动**，你需要为`unique_ptr`类实现一组**特殊成员函数**。具体来说，需要实现以下函数：

1. `~unique_ptr()`：析构函数，释放指针指向的内存
2. `unique_ptr(const unique_ptr& other)`：拷贝构造函数，**需要将其禁用**
3. `unique_ptr& operator=(const unique_ptr& other)`：拷贝赋值运算符，**需要将其禁用**
4. `unique_ptr(unique_ptr&& other)`：移动构造函数
5. `unique_ptr& operator=(unique_ptr&& other)`：移动赋值运算符