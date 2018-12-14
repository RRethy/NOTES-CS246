# Big Five

1. Implement the Big Five for a Stack with the following declaration, make sure it has strong guarentee:

* Assume Count words like an `int`, but since it is a class you must beware of any exceptions it might throw

```cpp
class Count;

template<typename T>
class Stack {

  Count count;
  std::vector<T> items;

  void swap();

  public:

    Stack();
    Stack(const Stack &other);
    Stack(Stack &&other);
    ~Stack();

    Stack &operator=(const Stack &other);
    Stack &operator=(Stack &&other);

};
```

2. Implement big five for a Dog which inherits from animal.
