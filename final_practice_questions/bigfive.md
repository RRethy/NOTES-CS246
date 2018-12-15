# Big Five

1. Implement the Big Five for a Stack with the following declaration, make sure it has strong guarentee:

* Assume Count works like an `int`, but since it is a class you must beware of any exceptions it might throw

```cpp
class Count;

template<typename T>
class Stack {

  Count count;
  std::vector<T> items;

  void swap(Stack &other);

  public:

    Stack();
    Stack(const Stack &other);
    Stack(Stack &&other);
    ~Stack();

    Stack &operator=(const Stack &other);
    Stack &operator=(Stack &&other);

};
```

> Soln

```cpp
void Stack::swap(Stack &other) {
  std::swap(count, other.count);
  std::swap(items, other.items);
}

Stack::Stack() {
}

Stack::Stack(const Stack &other) : items{other.items}, count{other.count} {
}

Stack::Stack(Stack &&other) : items{std::move(other.items)}, count{std::move(other.count)} {
}

Stack::~Stack() {
}

Stack Stack::&operator=(const Stack &other) {
  Stack tmp{other};
  swap(tmp);
  return *this;
}

Stack Stack::&operator=(Stack &&other) {
  swap(std::move(other));
  return *this;
}

```

2. Implement big five for a Dog which inherits from animal.

```cpp
class Animal {
  // Assume big five
};

class Dog final : public Animal {
  
  Count age;

  public:
    Dog(Count age, Something else) : Animal(else), age{age} {
    }
    ~Dog() override {
    }
    Dog(const Dog &other) : Animal(other), age{other.age} {
    }
    Dog(Dog &&other) : Animal{std::move(other)}, age{std::move(other.age)} {
    }
    Dog &operator=(const Dog &other) {
      Animal::operator=(other);
      age = other.age;
      return *this;
    }
    Dog &operator=(const Dog &&other) {
      Animal::operator=(std::move(other));
      age = std::move(other.age);
      return *this;
    }
};
```
