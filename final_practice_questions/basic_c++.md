# Basic C++ questions

> Write the body of a function which does the following and matches the provide function signature

1. Given a string that matches this regex pattern, `^([0-9]+|)[a-z,A-Z]+$`, return a `Foo`. This pattern matches an optional number and a word.

```cpp
struct Foo {
  int num; // The number found, or -1 if no number is found
  std::string word; // The word following the number in the string
};

Foo bar(std::string s);
```

2. 

# General C++ knowledge

1. What is the difference between `std::endl` and `'\n'`
