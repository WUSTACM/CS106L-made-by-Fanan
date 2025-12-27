# assignment2

它还是跟流有关的作业，如果课程已经完全听懂了，那么这章并不难。
这一章需要注意的是，如果你想用`istringstream`的话，必须要头文件`sstream`。

直接上奋吧

## Part1

````c++
std::set<std::string> get_applicants(std::string filename) {
  std::set<std::string> applicant;
  std::ifstream iss(filename);

  std::string str;
  while(getline(iss, str)) {
    applicant.insert(str);
  }

  return applicant;
}
````

## Part2

````c++
//这一段要求是，找出students中，名字首字母之和=kYourName的名字首字母之和。
std::queue<const std::string*> find_matches(std::string name, std::set<std::string>& students) {
  std::istringstream mch(name);
  std::string mtch = "";

  std::queue<const std::string*> ans;

  std::string temp;
  while(mch >> temp) {
    mtch += temp[0];
  }

  for(const auto &stu : students) {
    mch.str(stu);
    std::string s = "";
    while(mch >> temp) {
      s += temp[0];
    }

    if(s == mtch) {
      ans.push(&stu);
    }
  }

  return ans;
}
````

还有一个很神奇的写法，用到了很现代的东西，可以先做了解

````
std::string get_initials(const std::string &text) {
  auto combined_initials = text
                           | std::views::split(' ')
                           | std::views::filter([](auto&& word) { return !word.empty(); })
                           | std::views::transform([](auto&& word) { return *word.begin(); });
  return std::string(combined_initials.begin(), combined_initials.end());
}

std::queue<const std::string*> find_matches(std::string name, std::set<std::string>& students) {
  std::queue<const std::string*> matches;
  std::string given_initials = get_initials(name);
  for (const auto &stu : students) {
    if (get_initials(stu) == given_initials) {
      matches.push(&stu);
    }
  }
  return matches;
}
````

## Part3

````c++
std::string get_match(std::queue<const std::string*>& matches) {
  std::istringstream mch(kYourName);
  std::string mtch = "";

  std::string temp;
  while(mch >> temp) {
    mtch += temp[0];
  }

  while(!matches.empty()){
    std::string stu = *matches.front();
    mch.str(stu);
    std::string s = "";
    while(mch >> temp) {
      s += temp[0];
    }

    if(s == mtch) {
      return stu;
    }
  }
  return "NO MATCHES FOUND.";
}
````

