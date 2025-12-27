# assignment4

很明显的可以感觉到这个作业已经开始上强度了，很多东西可以用`C++`新一点的函数来写比如，这个作业里面用到的，`std::ranges::views`

## 作业要求Part1

完成 `tokenize` 这个分词函数，将 `source` 这个长字符串分割成一系列的 `Token`，代码中不能使用任何 `for/while` 循环，使用传统的 `STL`。
该 `part` 有三个小任务：1.识别所有指向空格字符的迭代器 2.在连诉的空格字符之间生成词元 3.去除空词元

## 作业要求Part2

完成 `spellcheck` 函数，识别拼写错误并给出拼写建议 ，需要使用全新的 `ranges` 库。
该 `part` 有三个小任务：1.跳过已经拼写正确的单词 2.对拼写不正确的单词，用 `levenshtein `函数来查找范围值为1的单词 3.经过处理后丢弃没有建议的拼写错误。



希望大家先看上面的要求再写lab，因为我之前直接写，没看懂要求。

好了大家先学 `ranges` 库的东西，再来写这个作业。



## Part1

````C++
Corpus tokenize(std::string& source) {
  //todo

  auto space = find_all(source.begin(), source.end(), isspace);
  Corpus tokens;

  std::transform(space.begin(), space.end() - 1, space.begin() + 1, 
    std::inserter(tokens, tokens.end()), 
    [&source](auto &&_first1, auto &&_first2) {
      return Token{source, _first1, _first2};
  }); 

  std::erase_if(tokens, [](const Token &token) { return token.content.empty(); });
  return tokens;
}
````

## Part2

````C++
std::set<Misspelling> spellcheck(const Corpus& source, const Dictionary& dictionary) {  
  //todo
  
  auto view = source 
    | std::ranges::views::filter([&dictionary](const Token &token) {
        return !dictionary.contains(token.content);
      }) 
    | std::ranges::views::transform([&dictionary](const Token &token) -> Misspelling {
        auto dview = dictionary 
          | std::ranges::views::filter([&token](const std::string &str) {
            return levenshtein(token.content, str);
          });
        std::set<std::string> sug(dview.begin(), dview.end());
        return Misspelling{token, sug};
      })
    | std::ranges::views::filter([](const Misspelling &elem) {
      return !elem.suggestions.empty();
      });
  
  return std::set<Misspelling>(view.begin(), view.end());
};
````



多嘴一句，记得编译的时候要用

````
g++ -std=c++23 main.cpp spellcheck.cpp -o main
````
