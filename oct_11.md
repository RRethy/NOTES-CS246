# Initializing Objects (cont'd)

* `Student billy{70, 50, 75};`
  * Allocated on the stack **not** the heap
* If an appropriate constructor is present, it is called
* `Student billy = Student(80, 50, 75);`
  * Allocated on the stack **not** the heap

## Uniform Initializing Syntax

1. `int x = 5;`
2. `int x(5);`
3. `string s = "hello";`
4. `string s("string");`
5. `int x{5};`
6. `string s{"hello"};`
7. `Student billy{...};`

* You can always use curly braces (5, 6, 7) since C++11.
* They added it in C++11 to have a single initializing syntax that is can be
  used everywhere

```cpp
// pbilly is allocated on the heap NOT the stack
Student *pbilly = new Student{85, 50, 75};
// Some code
delete pbilly;
```

## Advantages of writing constructors

* Execute arbitrary code
* Default arguments
* Overload constructors
* Sanity checks
* Anything you can do with methods, you can do with a constructor

```cpp
// Note: Default values go in the declaration
struct Student {
  Student(int assns = 0, int mt = 0, int final = 0);
  int assns = 0;
  int mt = 0;
  int final = 0;
};

// This constructor has some sanity checks
Student::Student(int assns, int mt, int final) {
  this->assns = assns < 0 ? 0 : assns;
  this->mt = mt < 0 ? 0 : mt;
  this->final = final < 0 ? 0 : final;
}

Student s1{50, 50, 50};
Student s2{50, 50}; // {assns, mt}
Student s3{50}; // {assns}
Student s3; // use ALL default values (note: no curly braces)
```

* Every class comes with a default (0 param) constructor which initializes
  fields that that are objects by calling its default constructor

```cpp
struct A {
  int x;
  Student y;
  Vec *z;
};
A a; // Calls the default constructor
// x, z are uninitialized
// y is initialized
```

* **Note**: Above there is no constructor declared, so `A a;` calls the
  default constructor
* As soon as you implement any constructor, you lose the default constructor
  and C-stlye initialization

```cpp
// No default constructor
struct A {
  // Bad style, used for brevity
  A::A(int x, Vec *z) {
    this->x = x;
    this->z = z;
  }
  int x;
  Vec *z;
};

A a; // Won't compile (if you have default params it will compile)
A b(5, nullptr); // Will compile
```

* Initializing constant/reference fields

```cpp
// Option 1
// in class initialization
int z;
struct MyStruct {
  const int myConst = 10;
  int &myRef = z;
};

// Option 2
// Do not do in class initializing
// Rule: const/refs must be initialized before constructor body runs
// TODO
```

## Steps for object construction

1. allocate space
2. field initialization
3. constructor body runs

* Lets hijack step 2: Member initialization List (MIL)

```cpp
Student::Student(const int id, int assns, int mt, int final) :
  id{id}, assns{assns}, mt{mt}, final{final < 0 ? 0 : final} {
}
```

* MIL cna be used for ALL fields
* no need to use `this`
  * outside the braces the identifier is a field
  * inside the braces normal scope rules apply
* in the MIL field initialization occurs in field declaration order
* Initializing fields in the MIL *can be* more efficient than Initializing in
  the constructor body
  * It would get default value at step 2, then reassigned in constructor body.

## Initializing objects as copies of others

```cpp
Student billy{80, 50, 75};
Student bobby{billy}; // copy constructor (get for free)
```

### A class comes with...

1. default constructor
2. copy constructor
3. copy assignment operator
4. destructor
5. move constructor (C++ onwards)
6. move assignment operator (C++ onwards)

* 2-6 are called the big five **(1/4 of the midterm is on this)**

```cpp
// The free copy constructor
Student::Student(const Student &other) :
  assns{other.assns}, mt{other.mt}, final{other.final} {
}
```

* Sometimes the free copy constructor does not work the way we want it
  * So we can override it

```cpp
struct Node {
  int data;
  Node *next;
  Node(int data, Node *next);
  Node(const Node &other);
};

Node::Node(int data, Node *next) :
  data{data}, next{next} {
}

Node::Node(const Node &other) :
  data{other.data}, next{other.next} {
}
```

```cpp
Node *np = new Node{1, new Node{2, new Node{3, nullptr}}}; // heap
```

* *Stack* np -> **|** *heap* -> Node(1) -> Node(2) -> Node(3) -> nullptr

* `Node m{*np};`
  * *Stack* m(1) -> **|** *heap* -> Node(2) *same node as above*
* `Node *n1 = new Node{*np};`
  * *Stack* n1 -> **|** *heap* -> Node(1) -> Node(2) *same node as above*
* This is not a true copy of a linked list since it should share nodes
  * This is called a **Shallow copy**
* What we want is a **deep copy**

```cpp
// Deep copy
// Recursively call the copy constructor

// Incorrect: it has no base case for next
//   segmentation fault
Node::Node(const Node &other) :
  data{other.data}, next{*other.next} {
}

// Correct: has base case
Node::Node(const Node &other) :
  data{other.data}, next{other.next ? new Node{*other.next} : nullptr} {
}
```

### A copy constructor is called when...

1. an object is constructed as a copy of another
2. pass by value
3. returning by value

* Due to #2, the param of a copy constructor **must** be a reference
  * or else we will have infinite recursion
