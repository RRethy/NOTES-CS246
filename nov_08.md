# Exceptions, Design Patterns

## Last time

* In C++, you can throw anything
* **Good practice**: using existing exception classes or create your own

```cpp
class BadInput {};
int n;
try {
  if (!(cin >> n)) throw BadInput{};
} catch (BadInput &e) {
  n = 0;
}
```

## Destructors & exceptions

* By default a program will terminate if a destructor throws an exception (std::terminal is called)
* we can change this default behaviour
  * `~class noexcept(false) {}`
* Supposed there is an exception and the stack is unwinding
  * a destructor is running
    * what if an exception occurs
  * this produces 2 simultaneous exceptions
    * `std::terminate` is called
* **Advice**: destructors shouldn't throw exceptions

## Design Patterns

* **Principle**: program to an interface, not an implementation
* Create abstract base classes to provide an interface
* Use base class ptrs and call interface methods
  * subclasses can be swapped in or out to change behaviour

### Iterator Pattern Revisited

* operator\*, operator++, operator!=

```cpp
class AbsIter {
  public:
    virtual ~AbsIter() {} // You must have an implementation since this gets called when a subclasses destructor get called
    virtual int &operator*() const = 0;
    virtual AbsIter &operator++() = 0;
    virtual bool operator!=(const AbsIter &) = 0;
};
```

```cpp
class List {
  struct Node;
  Node *head = nullptr;
  public:
    class Iterator : public AbsIter {
      Node *cur;
      Iterator() {
      }
      public:
        int &operator*() const override {
        }

        AbsIter &operator++() override {
        }

        bool operator!=(const AbsIter &other) override {
        }
    };
};
```

```cpp
class Set {
  public:
    class Iterator : public AbsIter {
      // same methods get implemented
      // We can write code that is no longer tied to any specific data structure
    };
};
```

```cpp
template <typename Fn>
void forEach(AbsIter &start, const AbsIter &end, Fn f) {
  while (start != end) {
    f(*start);
    ++start;
  }
}

void addFive(int &x) {
  x += 5;
}

List<int> l;
List::Iterator b = l.begin();
forEach(b, l.end(), addFive);
```

### Observer Pattern

* used in a publish-subscribe system
  * **Subject** - publishing/generating data
  * **Observer** - subscribe to data

| *AbsSubject* |
| - |
| + attach(AbsObserver) |
| + detach(AbsObserver) |
| + notifyObservers() |
| + *~AbsSubject* |

◇ ---↓

| *AbsObserver* |
| - |
| + *notify()* |

* We would have a Subject inherit from AbsSubject, and an Observer (which inherits from AbsObserver) **HAS-A** Subject

* Want AbsSubject to be abstract
  * no obvious Pure Virtual method
  * make the destructor Pure Virtual, but still implement it
* **Pure Virtual Method**: A Pure Virtual method must be implemented by the subclass for it to be concrete

### Decorator Pattern

* trying to update/add functionality to existing object

| *Component* |
| - |
| + *operation()* |

> ConcreteComponent inherits from Component

| ConcreteComponent |
| - |
| + operation()

> Decorator inherits from Component, but it is abstract
> Decorator **HAS-A** component

| *Decorator* |
| - |
| |

> ConcreteDecorator1 inherits from Decorator

| ConcreteDecorator1 |
| - |
| +operation() |

> ConcreteDecorator2 inherits from Decorator

| ConcreteDecorator2 |
| - |
| +operation() |
