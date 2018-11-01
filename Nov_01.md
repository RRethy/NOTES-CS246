# Dynamic Dispatch

* Method Overriding

```cpp
bool Book::isHeavy() { return numPages > 200; }
bool Text::isHeavy() { return numPages > 300; }
bool Comic::isHeavy() { return numPages > 30; }

Book b = Comic{..., ..., 40, ...};
b.isHeavy(); // calls Book::isHeavy() due to object slicing

Comic c{..., ..., 40, ...};
Comic *cptr{&c};
cptr->isHeavy(); // calls Comic::isHeavy()

Book *bptr{&c};
bptr->isHeavy(); // calls Book::isHeavy(), no slicing has occured
// Bool:isHeavy() is called since the compiler looks at the declared type of the ptr
```

* To change this behaviour, we want to use Dynamic Dispatch using `virtual`

```cpp
// Book class
class Book {
  std::string title, author;
  int length;
 protected:
  int getLength() const;
 public:
  Book(const std::string &title, const std::string &author, int length);
  std::string getTitle() const;
  std::string getAuthor() const;
  virtual bool isHeavy() const;
};
```

```cpp
// Comic class
class Comic: public Book {
  std::string hero;
 public:
  Comic(const std::string &title, const std::string &author, int length, const std::string &hero);
  bool isHeavy() const override;
  std::string getHero() const;
};
```

> Now the following code has different behaviour due to `virtual` and `override` in the above classes

```cpp
Comic c{..., ..., 40, ...};
Comic *cp{&c};
Book *bp{&c};
Book &br{c};
cp->isHeavy(); // calls Comic::isHeavy()
bp->isHeavy(); // calls Comic::isHeavy()
br.isHeavy(); // calls Comic::isHeavy()
```

* A virtual method is dispatched dynamically
  * The decision on which method to call is based on the runtime type of the object
  * dynamic dispatch has a cost since it is determined at runtime
  * CS744

```cpp
// The objects will be sliced, but the correct isHeavy will be called
Book *collection[20];
for (int i = 0; i < 20; ++i) cout << collection[i]->isHeavy();
```

# Polymorphism

> The ability to accommodate multiple types within the same abstraction

* collection is a polymorphic array

```cpp
// Must use references to avoid slicing to occur on the params
// ostream is polymorphic
ostream &operator<<(ostream &, Student &);
```

# Destructors

* When an object of a derived class is destroyed:

1. subclass destructor body runs
2. subclass fields are destroyed(reverse declaration order, opposite to construction)
3. superclass destructor is called
4. memory is deallocated

```cpp
// Run with valgrind
// leaks memory

class X {
  int *x;
 public:
  X(int n): x{new int [n]} {}
  ~X() { delete [] x; }
};

class Y: public X {
  int *y;
 public:
  Y(int n, int m): X{n}, y{new int [m]} {}
  ~Y() { delete [] y; }
};

int main () {
  X x{5};
  Y y{5, 10};

  X *xp = new Y{5, 10};

  // calls class X destructor but not class Y destructor
  // destructor is statically dispatched
  delete xp;
}
```

```cpp
// Run with valgrind
// no memory leak

class X {
  int *x;
 public:
  X(int n): x{new int [n]} {}
  virtual ~X() { delete [] x; }
};

class Y: public X {
  int *y;
 public:
  Y(int n, int m): X{n}, y{new int [m]} {}
  ~Y() { delete [] y; } // doens't need virtual keyword since it was declared in class X
};

int main () {
  X x{5};
  Y y{5, 10};

  X *xp = new Y{5, 10};

  // calls class Y destructor since X destructor is virtal
  // destructor is dynamically dispatched
  delete xp;
}
```

* If you don't want your class to be subclassed, declare the class `final`
  * This is default in `kotlin` but not `java`

```cpp
class Y final : public X {
};
```

* Note that final can be a field name, it is one of the keywords that only counts as a keyword if it is in a specific location, this is to maintain backwards compatibility

```cpp
class Student {
  public:
    // fees has no default impl
    virtual int fees();
};

class Coop : public Student {
  public:
    int fees() override {
    }
};

class Regular : public Student {
  public:
    int fees() override {
    }
};
```

* We want Student::fees to not have an implementation
  * Make it Pure Virtual

```cpp
class Student {
  public:
    // Pure Virtual method
    virtual int fees() = 0;
};
```

* `virtual` methods MAY be overridden by subclasses
* `Pure Virtual` methods MUST be overridden by subclasses to be considered concrete
* Student is not a **concrete class** since it has a Pure Virtual method
  * `abstract` class
* A class with even a single Pure Virtual method is abstract and cannot be instantiated

```cpp
Student s: // will not compile
```

* Abstract classes are used to organize types

  1. shared fields/methods
  2. polymorphism

* A concrete class declares no new Pure Virtual methods and overrides **all** inherited Pure Virtual methods

## UML

* Both `virtual` and `Pure Virtual` use italics
* `Abstract` class name use italics
* `static` uses bold

# Templates

```cpp
// A non generic Stack class
class Stack {
  int count;
  int capacity;
  int *contents;

  public:
    int pop();
    void push(int);
    int top();
    ~Stack();
};
```

* To make this Stack class generic, we can use C++ templates
  * Template class is parameterized on a type
    * The type would be `int` in the above class

```cpp
// A non generic Stack class
template <typename T>
class Stack {
  int count;
  int capacity;
  T *contents;

  public:
    T pop();
    void push(T);
    T top();
    ~Stack();
};

Stack<int> sInts;
Stack<string> sStrings;
```
