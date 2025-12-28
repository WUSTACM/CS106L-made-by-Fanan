# assignment6

这个作业已经涉及到一点现代C++的知识了，他要求我们用 `optional` 这是 C++17 引入的非常有用的类型，它表示一个**可能包含值也可能不包含值**的对象。
在写本次作业前，希望大家可以先学一下 `optional` 的相关知识。

### 为什么需要 `optional`？

在没有 `optional` 之前，我们通常使用特殊值来表示"无值"状态：

- 指针：`nullptr`
- 整数：`-1`、`0` 等魔法数字
- 布尔值 + 实际值 的组合

这些方法都不够类型安全且容易出错。

例如，我们应该怎么样使用 `optional` 里面的东西呢。

## 1.创建 `optional` 对象

```
// 空 optional
std::optional<int> empty_opt;
std::optional<int> empty_opt2 = std::nullopt;  // 显式为空

// 包含值的 optional
std::optional<int> opt1(42);
std::optional<int> opt2 = 42;
auto opt3 = std::make_optional(42);  // 自动推导类型

// 原地构造
std::optional<std::string> opt4(std::in_place, "Hello", 5);  // 构造 std::string("Hello")
```

## 2.检查是否包含值

```
std::optional<int> opt = get_value();

// 方法1: has_value()
if (opt.has_value()) {
    // 有值
}

// 方法2: 转换为 bool
if (opt) {
    // 有值
}

// 方法3: value_or() - 有值时返回值，无值时返回默认值
// 没记错的话，这个应该是C++23引入的
int value = opt.value_or(-1);  // 如果opt为空，返回-1
```

## 3.各种操作

| 操作                 | 作用                                                         |
| -------------------- | ------------------------------------------------------------ |
| `std::nullopt`       | 表示空的`optional`，用于赋值或返回空值                       |
| `has_value()`        | 检查`optional`是否包含值，返回`bool`                         |
| `value()`            | 获取值，无值时抛出`std::bad_optional_access`异常（慎用）     |
| `value_or(默认值)`   | 安全获取值：有值则返回值，无值则返回传入的默认值（推荐日常使用） |
| `operator*`/`->`     | 解引用获取值（类似指针），需先确认有值，否则行为未定义       |
| `reset()`            | 清空`optional`的值，使其变为空                               |
| `std::make_optional` | 显式构造`optional`对象，支持类型推导，比直接构造更简洁（推荐） |

### (1) transform

#### 核心作用

如果当前`optional`包含值，就对这个值应用你传入的转换函数，返回一个**新的 optional**（类型由转换函数的返回值决定）；如果当前`optional`为空，直接返回空的`optional`（类型和转换后的一致）。

简单说：`transform`是 “有值就加工，没值就摆烂”。

### (2) or_else

#### 核心作用

如果当前`optional`为空，就执行你传入的备选函数（该函数返回一个同类型的`optional`），并返回这个备选结果；如果当前`optional`有值，直接返回原`optional`（不会执行备选函数）。

简单说：`or_else`是 “没值就找备胎，有值就用自己”。

### (3) transform + or_else 组合使用（实战场景）

这两个函数结合使用时，能实现 “有值就转换，无值就兜底” 的完整逻辑，替代多层 if-else：

```cpp
#include <iostream>
#include <optional>
#include <string>

// 模拟：从配置文件读取端口（可能读取失败，返回空）
std::optional<int> read_port_from_config() {
    // 模拟读取失败
    return std::nullopt;
}

// 模拟：获取默认端口
std::optional<int> get_default_port() {
    std::cout << "使用默认端口" << std::endl;
    return 8080;
}

// 转换：端口转成连接字符串
std::string port_to_conn_str(int port) {
    return "连接地址：localhost:" + std::to_string(port);
}

int main() {
    // 逻辑：读取端口 → 无值则用默认 → 有值则转成连接字符串
    auto conn_str = read_port_from_config()          // 空的optional<int>
        .or_else(get_default_port)                   // 无值，执行备选，返回optional<int>(8080)
        .transform(port_to_conn_str);                // 有值，转换为optional<string>

    std::cout << conn_str.value() << std::endl;      // 输出：连接地址：localhost:8080
    return 0;
}
```

## Part1

完成 `find_course` 函数

## Part2

完成下面对 `output` 的转换。

- 如果找到课程，

  ```bash
  Found course: <课程名称>,<学分>,<开课季度>
  ```

- 如果未找到课程，则上述代码应输出：

  ```bash
  Course not found.
  ```