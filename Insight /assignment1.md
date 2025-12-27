# assignment1

在看课程的时候，可能会学到`istringstream`跟`ostringstream`，会发现这个东西十分精妙，然后我就跃跃欲试去写了`assignment1`结果发现，我还是大意了。写的时候还是出现了很多问题，在下面一一列举。

1.`ostringstream`写入文件跟`ofstream`写入文件是不一样的（`ifstream`同理），`ostringstream`的输出目标是内存的缓冲区，不是硬盘文件，所以它没有往文件写入数据的功能;`ofstream`的输出目标才是文件，天然支持文件的写入功能。简单来说，你如果要把数据写入文件，只能用`ofstream`。

2.大家应该都会用`find`的吧，就是`string`的库的那个，这一章节只要看懂了**流**的操作，那么这一章就没什么问题了。

以下附上我的代码：

````C++
struct Course {
  std::string title;
  std::string number_of_units;
  std::string quarter;
}
````

````C++
void parse_csv(std::string filename, std::vector<Course> &courses) {
  std::ifstream iss(filename);
  std::string str;
  getline(iss, str);

  while(getline(iss, str)) {
    auto temp = split(str, ',');
    courses.push_back({temp[0], temp[1], temp[2]});
  }
}
````

````C++
void write_courses_offered(std::vector<Course> all_courses) {
  std::ofstream oss("student_output/courses_offered.csv");
  oss << "Title,Number of Units,Quarter\n";

  for(auto p : all_courses) {
    if(p.quarter.find("2023-2024") != -1) oss << p << "\n";  
  }
}
````

````C++
void write_courses_not_offered(std::vector<Course> unlisted_courses) {
  std::ofstream oss("student_output/courses_not_offered.csv");
  oss << "Title,Number of Units,Quarter\n";

  for(auto p : unlisted_courses) {
    if(p.quarter.find("2023-2024") == -1) oss << p << "\n";
  }
}
````

但是如果你这么写你是对不了的，还有一些小问题，还是要去自己找找的，推荐去看看`utils.cpp`的代码。