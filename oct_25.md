# Last time

```cpp
List lst;
lst.addToFront(1);
for (List::Iterator it = list.begin(); it != list.end(); ++i) {
  cout << *it << endl;
}
for (auto it = list.begin(); it != list.end(); ++i) {
  cout << *it << endl;
}
```

```cpp
int y = 10;
// similar to var in kotlin or js
auto x = y; // compiler will know y is type int, it will make x the same type as y
```

* C++ has builtin support for the iterator design pattern (so does Java)
  * **Range-based for loops**

```cpp
for (auto n : lst) {
  cout << n << endl; // n is a copy of whatever operator* returns
}
for (auto &n : lst) {
  cout << n << endl; // n is a reference
  ++n; // valid now
}
```

* To use a range-based for loop for a class, MyClass
  1. MyClass must implement `begin()` and `end()` that must return some Iterator (name of iterator class doesn't matter) object
  2. That Iterator class must implement `operator*`, `operator++`, `operator!=`

# friend keyword

* Constructor for List::Iterator is public, List::begin && List::end call it
* Want to make List::Iterator constructor private, but then begin && end cannot access it
* Iterator can declare List to be a friend

```cpp
class List {
  class Node;
  Node *theList = nullptr;

 public:
  class Iterator {
    Node *p;
    explicit Iterator(Node *p);
   public:
    int &operator*() const;
    Iterator &operator++();
    bool operator==(const Iterator &other) const;
    bool operator!=(const Iterator &other) const;
    friend class List;
  };

  Iterator begin();
  Iterator end();

  void addToFront(int n);
  int ith(int i);
  ~List();
};
```

* `friend` breaks encapsulation
  * Thus, try to use `friend` as rare as possible
* Keep fields private, use setters/getters for them

```cpp
class Vec {
  int x, y;
  public:
    int getX() const { return x; }
    int getY() const { return y; }
    void setX(int x) { this->x = x; }
    void setY(int y) { this->y = y; }

    friend ostream &operator<<(ostream &, const Vec &);
};

ostream &operator<<(ostream &out, const Vec &v) {
  out << v.x << v.y; // won't compile
  return out;
}
```

* Single point of entry into the variable
  * Useful for invariance
* This comes default in a bunch of languages, see `Kotlin`

# mutable

```cpp
struct Student {
  int assns, mt, final;
  mutable int numCalls = 0;
  float grade() const {
    ++numCalls; // legal modification since numCalls is mutable
    return 0.4 * assns + 0.2 * mt + 0.4 * final;
  };
};
```

# static

## We can make fields static

* A static field belongs to the class & not each object of the class

```cpp
struct Student {
  int assns, mt, final;
  static int numObjects; // in-class initialisation is not allowed
  Student() : ..... { ++numObjects; } {
  }
  float grade() const {
    ++numCalls; // legal modification since numCalls is mutable
    return 0.4 * assns + 0.2 * mt + 0.4 * final;
  };
};
```

* static fields must be defined external to the file defining the class

> Student.cc

```cpp
int Student::numObjects = 0; // now memory is allocated
```

## We can make static member functions

```cpp
struct Student {
  int assns, mt, final;
  static int numObjects; // in-class initialisation is not allowed
  Student() : ..... { ++numObjects; } {
  }
  float grade() const {
    ++numCalls; // legal modification since numCalls is mutable
    return 0.4 * assns + 0.2 * mt + 0.4 * final;
  };
  // Static functions not static methods since it doesn't belong to an instance of a class
  // Does not have a this ptr
  static int objCreated() {
    return numObjects;
  }
};
```

* Call be called as follows `int num = Student::objCreated()`
  * Don't need an instance of the class Student

# System Modelling

* A good design requires:
  1. determining the major abstractions (classes)
  2. relationship between classes
* **UML**: Unified Modelling Language
  * Standard

## Modelling a class

1. Name of class
2. Fields
3. Methods

> - is for private, + is for public

| name of class | Vec |
| ------- | ------------ |
| fields | - x: Integer |
| fields | - y: Integer |
| --- | ----- |
| methods | + Vec(Integer, Integer) |
| methods | + getX(): Integer |

* Relationship 1: Composition

```cpp
class Vec {
  int x, y;
  public:
    Vec(int x, int y);
};

class Basic {
  Vec v1, v2;
  public:
    Basis() : v1{0, 0}, v2{1, 1} {
    }
};

Basic b;
```

* Embedding an object (Vec) within another (Basic) is composition. We say: `Basic owns-a Vec`
  * If A owns-a B, then B does not exist outside A
  * copying A, copies B (deep copies)
  * destroying A, destroys B
* For the List class, List owns-a Node

### Showing the owns-a relationship

> Draw each class as a table, similar to above. Shaded diamond means owns-a

`Basic ◆->v1,v2-> Vec`
