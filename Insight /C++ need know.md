# auto的类型

````c++
auto multiplier = 2.4;  //double
auto name = "Fanan"; //char* (c-string)
auto bettername = string{"Fanan"}; //string
const auto& bettername1 = string{"Fanan"} //const string&
auto& refMult = multiplier; //double&

auto func = [](auto i) {
    return i * 2;
}; //lambda
````

# algorithm

## (1) std::sort

````C++
// std::sort这个大家或多或少都有接触过的吧
vector<int> num = {1, 2, 3, 4};
sort(num.begin(), num.end(), [](const int& a, const int& b) {
    return a > b;
});
````

## (2) std::nth_element

````c++
/*
给定一个容器（如vector）和一个位置nth，将容器中 “第 n 个元素” 放到完全排序后它应该在的位置；
该位置左侧的所有元素都 ≤ 它（默认升序），右侧的所有元素都 ≥ 它；
左右两侧的元素不保证有序（这是和sort的核心区别），但能保证nth位置的元素是 “正确值”。
复杂度O(n)
*/

std::vector<int> nums = {7, 2, 9, 1, 5, 4, 8, 3, 6};
auto nth_pos = nums.begin() + 2;
std::nth_element(nums.begin(), nth_pos, nums.end());
//nums将会变成 2 1 3 7 5 4 8 9 6（3在正确位置，左右分别≤/≥3）
````

## (3) std::stable_partition

````c++
/*
将容器中的元素按「自定义条件（谓词）」分成两部分：满足条件的元素在前，不满足的在后；
关键特性：两部分内部的元素会保持原有的相对顺序（“stable” 的核心含义）；
对比普通std::partition：后者仅完成分区，但不保证两部分内部的顺序，而stable_partition会保留顺序（代价是时间 / 空间复杂度略高）。
*/
std::vector<int> nums = {7, 2, 9, 4, 5, 8};
auto partition_it = std::stable_partition(nums.begin(), nums.end(), [](int a) {
    return a % 2 == 0;
});

for (auto it = nums.begin(); it != partition_it; ++it) {
    std::cout << *it << " "; // 输出：2 4 8（和原数组中偶数出现的顺序一致）
}

for (auto it = partition_it; it != nums.end(); ++it) {
    std::cout << *it << " "; // 输出：7 9 5（和原数组中奇数出现的顺序一致）
}
````

## (4) std::copy_if

````c++
/* 
从源容器中遍历元素，只将满足自定义条件（谓词） 的元素拷贝到目标容器中；
不修改原容器，仅做 “筛选 + 拷贝”，属于 “非修改型算法”；
对比std::copy：std::copy拷贝所有元素，std::copy_if只拷贝满足条件的元素，灵活性更高。
*/

std::vector<int> src = {1, 2, 3, 4, 5, 6, 7, 8};
// 目标容器（提前为空，用back_inserter自动插入),如果容器不空，他会自动在容器后面添加
std::vector<int> dest;

std::copy_if(src.begin(), src.end(), std::back_inserter(dest), [](int a) {
    return a % 2 == 0;
});

std::cout << "拷贝的偶数：";
for (int num : dest) {
    std::cout << num << " "; // 输出：2 4 6 8
}
````

## (5) std::remove_if

````C++
/*
核心作用：遍历容器，将不满足移除条件的元素向前移动（覆盖），把满足移除条件的元素挤到容器末尾；
关键特性：不实际删除元素，也不改变容器的size()，仅改变元素位置；
返回值：指向容器中 “第一个需要被删除的元素” 的迭代器（即末尾待删除区域的起始位置）；
对比std::copy_if：
	1.copy_if：拷贝满足条件的元素到新容器，不修改原容器；
	2.remove_if：原地移动元素，修改原容器内容，但需配合erase才会真正删除元素。
*/

std::vector<int> nums = {1, 2, 3, 4, 5, 6};
std::cout << nums.size() << std::endl; //输出6

auto it = std::remove_if(nums.begin(), nums.end(), [](int a) {
    return a & 1;
});
std::cout << nums.size() << std::endl; //输出6，顺序为1 3 5 2 4 6

nums.erase(it, nums.end());
std::cout << nums.size() << std::endl; //输出3

//可以连起来写
nums.erase(
	std::remove_if(nums.begin(), nums.end(), 
    	[](int a) {return a & 1;}),
    nums.end()
);
````

## (6) transform

````c++
//如果大家打过算法竞赛的话，应该有接触过一点点，比如
string s = "Fanan";
transform(s.begin(), s.end(), s.begin(), ::tolower);
//这是一个把字符串中所有字符都小写的一个写法，这就是一个典型的transform的一元操作。

template< class InputIt, class OutputIt, class UnaryOperation >
OutputIt transform( InputIt first1, InputIt last1, 
                    OutputIt d_first, UnaryOperation unary_op );
//first1,last1分别是迭代器的头跟尾，d_first是最后写入哪里，unary_op是操作。
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> src = {1, 2, 3, 4, 5};
    std::vector<int> dst(src.size());
    
    // 将每个元素乘以2
    std::transform(src.begin(), src.end(), dst.begin(),
                   [](int x) { return x * 2; });
    
    // 输出：2 4 6 8 10
    for (int x : dst) std::cout << x << " ";
    
    return 0;
}


//transform的二元操作
template< class InputIt1, class InputIt2, class OutputIt, class BinaryOperation >
OutputIt transform( InputIt1 first1, InputIt1 last1,
                    InputIt2 first2, OutputIt d_first,
                    BinaryOperation binary_op );

#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> a = {1, 2, 3, 4, 5};
    std::vector<int> b = {10, 20, 30, 40, 50};
    std::vector<int> result(a.size());
    
    // 将两个vector对应元素相加
    std::transform(a.begin(), a.end(), b.begin(), result.begin(),
                   [](int x, int y) { return x + y; });
    
    // 输出：11 22 33 44 55
    for (int x : result) std::cout << x << " ";
    
    return 0;
}

//我们也可以使用back_inserter(result)来写，可以看上面的copy_if
````

# ranges

## 一、定义与核心定位

std::ranges::views（简称 views）是 C++20 标准库中 `ranges` 模块的核心组件之一，本质是对“范围（range）”的轻量级、惰性求值的适配层。其核心作用是：在不修改原始数据、不产生中间拷贝的前提下，对范围（如容器、数组、生成器等）进行过滤、转换、切片等操作，最终生成一个新的“视图”供后续使用。

视图本身不存储数据，仅保存对原始范围的引用（或迭代器对）以及操作逻辑。只有当视图被遍历（如通过循环、算法迭代）时，相关操作才会被实际执行——这种“惰性求值”特性是 views 与传统算法（如 `std::transform`、`std::filter`）的核心区别，也是其高效性的关键。

## 二、核心特性

### 1. 惰性求值（Lazy Evaluation）

视图的创建和组合操作本身不执行任何计算，仅记录操作逻辑。只有当视图被“消费”（如迭代、取值）时，才会逐元素应用所有操作。例如：将 `views::filter` 与 `views::transform` 组合后，遍历视图时会先过滤元素，再对符合条件的元素执行转换，而非先过滤生成中间容器，再转换中间容器。

### 2. 无存储、轻量级

视图不持有数据，仅保存对原始范围的引用和操作状态（如过滤条件、转换函数、切片位置等），因此创建视图的开销极低（通常为常数时间 O(1)），内存占用可忽略。

### 3. 不可修改原始数据

视图默认是“只读”的（除非显式使用可修改视图，如 `views::as_rvalue`），所有操作仅作用于遍历过程中的元素临时值，不会改变原始范围的数据。

### 4. 可组合性（Composability）

多个视图可以通过“管道运算符（|）”轻松组合，形成清晰的操作流水线。例如 `range | views::filter(...) | views::transform(...) | views::take(...)`，逻辑上等同于“先过滤、再转换、最后取前 N 个元素”，代码简洁且可读性高。

### 5. 兼容标准算法

视图本身是“范围（range）”，可直接作为标准范围算法（如 `std::ranges::for_each`、`std::ranges::count`）的输入，无需额外适配。

## 三、常见视图类型及使用示例

`AI` 告诉我可以写成 `std::views::filter` 但是我觉得还是写 `std::ranges::views::filter` 好一点，因为我写的时候报错了。

C++20 标准库提供了数十种内置视图，覆盖过滤、转换、切片、拼接、生成等常见场景，以下是最常用的几种：

### 1. 过滤视图：views::filter

功能：筛选出符合条件的元素，接收一个返回 bool 的谓词函数。

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6};
    // 筛选偶数
    auto even_nums = nums | std::ranges::views::filter([](int x) { return x % 2 == 0; });
    
    for (int x : even_nums) {
        std::cout << x << " "; // 输出：2 4 6
    }
    return 0;
}
    
```

### 2. 转换视图：views::transform

功能：对每个元素执行转换操作，接收一个转换函数。

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {1, 2, 3};
    // 将每个元素乘以 2
    auto doubled = nums | std::ranges::views::transform([](int x) { return x * 2; });
    
    for (int x : doubled) {
        std::cout << x << " "; // 输出：2 4 6
    }
    return 0;
}
    
```

### 3. 切片视图：views::take 与 views::drop

-`views::take(n)`：取前 n 个元素，若不足 n 个则取全部；

\- `views::drop(n)`：跳过前 n 个元素，取剩余元素。

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};
    // 取前 3 个元素
    auto take3 = nums | std::ranges::views::take(3);
    // 跳过前 2 个元素，取剩余
    auto drop2 = nums | std::ranges::views::drop(2);
    
    std::cout << "take(3): ";
    for (int x : take3) std::cout << x << " "; // 输出：1 2 3
    
    std::cout << "\ndrop(2): ";
    for (int x : drop2) std::cout << x << " "; // 输出：3 4 5
    return 0;
}
    
```

### 4. 拼接视图：views::concat

功能：将多个同类型的范围拼接成一个视图（惰性拼接，不产生中间拷贝）。

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> a = {1, 2};
    std::vector<int> b = {3, 4};
    // 拼接 a 和 b
    auto combined = std::ranges::views::concat(a, b);
    
    for (int x : combined) {
        std::cout << x << " "; // 输出：1 2 3 4
    }
    return 0;
}
    
```

### 5. 生成视图：views::iota

功能：生成一个无限递增的整数序列（惰性生成，只有遍历到的元素才会被计算），通常与 `views::take` 配合使用以限制长度。

```cpp
#include <ranges>
#include <iostream>

int main() {
    // 生成 1~5 的序列（从 1 开始，取 5 个元素）
    auto seq = std::ranges::views::iota(1) | std::ranges::views::take(5);
    
    for (int x : seq) {
        std::cout << x << " "; // 输出：1 2 3 4 5
    }
    return 0;
}
    
```
