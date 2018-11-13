# Big 5 revisited

```cpp
class Book {
  // imagine big 5 implemented
};

class Text : public Book {
  // big 5 not implemented
};

Text t1 = {...};
Text t2 = t1; // calls Text's copy ctor, the one we get for free

// this is what the default would do
Text::Text(const Text &other)
  : Book(other), topic{other.topic} {
}

// this is what the default would do
Text &Text::operator=(const Text &other) {
  Book::operator=(other);
  topic = other.topic;
  return *this;
}
```

## Move constructor

> Incorrect, this makes a deep copy

```cpp
Text::Text(Text &&other) : Book{other}, topic{other.topic} {
}
```

> Correct

```cpp
Text::Text(Text &&other) : Book{std::move(other)}, topic{std::move(other.topic)} {
}
```

* `std::move(T t)` allows you to get `t` (an **lvalue**) as an **rvalue**

## Move assignment operator

```cpp
Text &Text::operator=(Text &&other) {
  Book::operator=(std::move(other));
  topic = std::move(other.topic);
  return *this;
}
```

## Copy assignment operator

```cpp
Text t1{"abc", "Nomair", 400, "CS246"};
Text t2{"xyz", "Dave", 200, "CS136"};
Book *pb1 = &t1;
Book *pb2 = &t2;
*pb1 = *pb2; // object assignment through Base class ptrs
// Book::operator= is called
// t1 will become {"xyz", "Dave", 200, "CS246"}
```

* Partial Assignment Problem
  * `topic` was not copied
* We could make operator= virtual

```cpp
class Book {
  public:
    virtual Book &operator=(const Book &);
};

class Text : public Book {
  public:
    Text &operator=(const Text &) override; // does not compile, incorrect method signature

    Text &operator=(const Book &) override; // does compile, signatures match
    // We don't know that the parameter is of type Text, call Mixed Assignment Problem
};
```

* **Side Note**: method signature is name and parameters, not the return type

* **Mixed Assignment Problem**: This allows assignment to Texts from any type of Book
  * later how to avoid this will be discussed, for now, lets not make operator= virtual
* T prevent Partial assignment, lets prevent assignment through base class pointers
  * Could make Book::operator= private, but subclasses still need access
  * Could make Book::operator= protected, but we still can't assign a Book to another Book
  * The base class should have been abstract (recall Book was not abstract), this solves the above problem

| AbstractBook |
| - |
| title |
| author |
| numPages |
| - |
| + *AbstractBook* | <-- Pure Virtual
| # operator= |

* Each class inherits from AbstractBook
* Now you can assign with the exception of going through base class pointers as above

# Factory Method Pattern


> Enemy and Level are abstract classes

* Enemy
  * Turtle
  * Bullet
* Level
  * Normal
  * Castle

```cpp
Player *p = ...;
Level *l = ...;
Enemy *e = nullptr;
while (p->isNotDead()) {
  // generate enemy
  e = l->createEnemy(); // see below code snippet
  // attack player
}
```

* The type of enemy to generate next should come from a factory method which is part of the level

```cpp
class Level {
  public:
    virtual Enemy *createEnemy() = 0; // Pure Virtual
};

class Normal : public Level {
  public:
    Enemy *createEnemy() override {
      // send in more turtles
    };
};

class Castle : public Level {
  public:
    Enemy *createEnemy() override {
      // send in more bullets
    };
};
```

* **Note**: Factory Pattern is also called the Virtual Constructor Pattern
  * The factory method acts like a constructor
  * **Eg.**
    * Iterator::begin and Iterator::end
    * LinkedList::addToFront

# Template Method Pattern

* Used when a class wants subclass to override a proper subset of its methods

| Base |
| - |
| +|- methods | <-- Non virtual
| +|- *methods* | <-- virtual

* A subclass can only override a proper subset of the base class' method

```cpp
class Turtle {
  public:
    void draw() {
      drawHead();
      drawShell();
      drawFeet();
    }
  private:
    void drawHead();
    void drawFeet();
    virtual void drawShell() = 0;
};

class GreenTurtle : public Turtle {
  void drawShell() override {
  }
};
```
