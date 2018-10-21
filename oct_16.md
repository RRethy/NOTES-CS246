# Last time

* Copy constructor: used to create copies of objects
* Note on single parameter constructors

```cpp
Node::Node(int data) : data{data}, next{nullptr} {
}

Node n{4};
Node n = 4; // Calls the syntax above

void foo(Node n) {
}

foo(4); // Legal

// Explains why this is legal:
string s = "Hello";
string s{"Hello"}; // Same as above
```

* One parameter constructors create implicit/automatic conversions
* Disallow the single parameter conversion, use the `explicite` keyword:

```cpp
struct Node {
  explicit Node(int data);
};

Node n = 4; // Illegal now

void foo(Node n) {
}
foo(4); // Illegal
foo(Node{4}); // Legal
```

# Destructor

* A method called the destructor runs whenever objects are destroyed
* When is it called
  * stack allocated -> when it goes out of scope
  * heap allocated -> destroyed when you call `delete` on a pointer to that object
* Steps for object destruction
  1. destructor body runs
  2. fields that are objects are destroyed in reverse declaration order
  3. deallocate space
* The destructor body of the "free" destructor is empty
  * This might not be sufficient

```cpp
// ptr np -> Node(0) -> Node(1) -> Node(2)
// stack     heap       heap       heap

delete np;
```

* Node(0) is deallocated
* Node(1) is not deallocated
* Node(2) is not deallocated
* Node(1) and Node(2) are both leaked
* We need to recursively deallocate the nested nodes

```cpp
struct Node {
  ~Node(); // no parameters and cannot be overloaded
  Node n;
  int data;
};

Node::~Node() {
  delete n; // recursively calls the destructor
}
```

**Note:** You are able to call `delete` on `nullptr`
**Note:** The destructor above will crash if `n` is allocated on the stack

# Copy assignment operator

```cpp
Student billy{80, 50, 75};
Student bobby = billy; // copy constructor
Student jane;
jane = billy; // copy assignment operator
jane.operator=(billy); // Same as above
```

```cpp
// Return type is explained at bottom of code
// Example of a question on the midterm
// 5/5 on exam for this question
Node &Node::operator=(const Node &newN) {
  data = newN.data;

  // see #1 below for explanation
  if (this == &newN) return *this;

  delete n;

  // relying on the copy constructor to make sure a true copy is made
  // See #2 below for when this fails
  n = newN.n ? new Node(*newN.n) ? nullptr;

  return *this;
}

Node n1{1};
Node n2{2};
Node n3{3};

n1 = n2;
n1.operator=(n2);

n1 = n2 = n3; // we need to return a Node from the operator=() method
```

1. Self assignment could be problematic:

```cpp
Node n{1, new Node{2, nullptr}};
n = n;
```

* The code above will delete `this->n` and then try accessing it through `newN.n` which will be a dangling pointer since you no longer own that memory location
  * Use self-assignment check to avoid dangling pointer

2. If call to `new Node()` fails (heap has no more memory), then `n` is not assigned, but it was already deleted and thus has become a dangling pointer.
  * Can be fixed with the following implementation of the copy assignment operator

```cpp
// 5/5 on exam for this question, better than above
// Important on final
Node &Node::operator=(const Node &newN) {
  if (this == &newN) return *this;
  Node *tmp = n;

  // If this line fails, no lines after this will be run
  n = newN.next ? new Node(*newN.next) : nullptr;

  data = newN.data;
  delete tmp; // delaying the deleting to see if the above line fails
  return *this;
}
```

## Alternate method (need to know both this and the above method)

> Copy and Swap Idiom

> node.h
```cpp
struct Node {
  void swap(Node &other);
};
```

> node.cc
```cpp
#include <utility>

void Node::swap(Node &other) {
  std::swap(data, other.data);
  std::swap(next, other.next);
}

Node &Node::operator=(const Node &other) {
  Node tmp{other}; // deep copy. Memory allocated on the stack
  swap(tmp);
  return *this;
}
```

## Complicated example

```cpp
// pass-by-value
Node plusOne(Node n) {
  for (Node *p {&n}; p ; p = p->next) {
    ++p->data;
  }
  // return by value
  return n;
}

Node n{1, new Node{2, nullptr}};
Node n2{plusOne(n)}; // 6 calls to copy constructor
```
