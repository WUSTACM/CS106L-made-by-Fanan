# assignment5

这个作业就是正常的涉及到面向对象中的构造函数，析构函数，以及函数的重载等。

## Part1

编写一个`operator<<`运算符，用于将`User`对象输出到`std::ostream`，多一嘴他应该是一个友元函数。输出样例如下：

````
User(name=Alice, friends=[Bob, Charlie])
````

## Part2

1. 为`User`类实现析构函数。需实现`~User()`特殊成员函数。
2. 使`User`类可拷贝构造。需实现`User(const User& user)`特殊成员函数。
3. 使`User`类可拷贝赋值。需实现`User& operator=(const User& user)`特殊成员函数。
4. 禁止`User`类可移动构造。需删除`User(User&& user)`特殊成员函数。
5. 禁止`User`类可移动赋值。需删除`User& operator=(User&& user)`特殊成员函数。

## Part3

编写一个`operator+=`运算符用于表示将一个用户添加到另一个用户的好友列表中。该操作应是对称的，例如，将Charlie添加到Alice的好友列表时，Alice也应被添加到Charlie的好友列表中。示例代码如下：

```cpp
User alice("Alice");
User charlie("Charlie");

alice += charlie;
std::cout << alice << std::endl;
std::cout << charlie << std::endl;

// 预期输出：
// User(name=Alice, friends=[Charlie])
// User(name=Charlie, friends=[Alice])
```