# Encapsulation

* treat object as black boxes
* hide the implementation
* provide and interface (select number of methods)

```cpp
// by default for struct, things are pubic
struct Vec {
  Vec(int x, int y);
  private:
    int x, y;
  public:
    Vec operator+(const Vec &other) {
      return {x + other.x, y+other.y};
    }
};

Vec v{1, 2}; // legal
Vec v1 = v + v; // legal
std::cout << v.x << v.y << std::endl; // illegal
```

* **Advice**: at a minimum, keep all fields private (use setters/getters)
* In a `class`, default visibility is `private` not `public`, prefer `class` over `struct`

```cpp
class Vec {
  int x, y;
  public:
    Vec(int x, int y);
    Vec operator+(const Vec &);
};
```

# Node Invariant

* Prevent client code from creating Nodes, and accessing the next ptr
  * Create a wrapper `List` class which will have sole access to creating nodes or updating the next ptr

> list.h
```cpp
class List {

  struct Node;
  Node *head = nullptr;

  public:
    void addToFront(int);
    int ith(int); // returns the ith data value at the ith node
    ~List();
};
```

> list.cc
```cpp
struct List::Node {
  int data;
  Node *next;
  ~Node() {
    delete next;
  }
}

List::~List() {
  delete head;
}

void List::addToFront(int n) {
  head = new Node{n, head};
}

/**
 * The ith node must exist
 */
int List::ith(int i) {
  Node *cur = head;
  for (int j = 0; j < i && cur; ++j, cur = cur->next);
  return cur->data;
}
```

* Suppose we want to print the list, what is the Big O time complexity
  * If done outside the List class, it will be O(n^2)
  * If done inside the List class, it can be O(n)
    * Avoid this since you will need a custom method for each thing, thus we can create some abstract list function and iterators

## Design Patterns

### Iterator Design Pattern

* For O(n) traversal, we will need to track where we are inside the List
  * **Challenge**: Without using a public Node ptr
  * **Solution**: Create another class that keeps track of where we are, but does so privately
    * This Iterator class will act as an abstraction of a ptr inside the List

```cpp
// arr is an array
for (int *p = arr; p != arr + arraySize; ++p) {
}
```

```cpp
class List {

  struct Node;
  Node *head = nullptr;

  public:
    class Iterator {

      Node *cur;

      public:

        Iterator(Node *cur) : cur{cur} {
        }

        int &operator*() const {
          return cur->data;
        }

        // unary prefix
        Iterator &operator++() {
          cur = cur->next;
          return *this;
        }

        // unary postfix
        //Iterator &operator++(int) {
          // ...
        //}

        bool operator!=(const Iterator &other) {
          return cur != other.cur;
        }

    };
    void addToFront(int);
    int ith(int); // returns the ith data value at the ith node
    ~List();

    Iterator begin() {
      return Iterator{head};
    }

    Iterator end() {
      return Iterator{nullptr};
    }
};
```

> client code to interact with the above code snippet

```cpp
list lst;
list.addtofront(2);
list.addtofront(3);
// order o(n) traversal
for (auto it = list.begin(); it != list.end(); ++it) {
  cout << *it << endl;
  *it = *it + 1; // changes the node data
}
```

> Nicer way of writing the above

```cpp
list lst
list.addtofront(2);
list.addtofront(3);
// order o(n) traversal
for (auto x : lst) {
}
for (auto &x : lst) {
}
```
