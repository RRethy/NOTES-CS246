# Review

* You can also do operator overloading
  * `cin >> x` where x can be `int` or `bool` or something else
    * `operator>>(cin, x);` is the same as before. Operator is implemented as a functon

# Structs

```cpp
struct Node {
  int data;
  struct Node *next;
};

//or

struct Node {
  int data;
  Node *next;
};

Node n = { 2, NULL };

//or

Node n = { 2, nullptr }; // c++11 and onwards
```

**Note**: The struct keyword above is optional in this line `struct Node *node;`

# Constants

```cpp
const int MAX = 100;
const Node n1{ 2, nullptr }; // Cannot change the fields of n1
int x = 10;

const int *p = &x; // p is a pointer to an constant int
p = &y; // VALID
*p = 10; // INVALID

int * const p1 = &x; // p is a constant pointer to an int;
p = &y; // INVALID
*p = 10; // VALID
```

```cpp
// pass-by-value
void inc(int n) {
  n = n + 1;
}

int x = 5;
inc(x);
cout << x; // prints 5
```

```cpp
// pass-by-pointer
void inc(int *n) {
  *n = *n + 1;
}

int x = 5;
inc(&x):
cout << x; // prints 6
```

# pass-by-reference

* Lvalue Reference
* Two types of references

```cpp
int y = 10;
int &z = y; // z is an lvalue reference to y

z = 15; // note not *z = 15;
// Changes the value of y

int *p = &z; // p now points to y
sizeof(z); // gives the size of y
```

* An lvalue reference is a constant ptr with automatic dereferencing
* z will continue to "point" to y forever
* `z` has no concept of itself, it is all changed at compile time
  * Thus, lvalue reference is just syntactic sugar

### Things you cannot do

* Cannot leave it uninitialize
  * Since it is a const ptr under the hood
* Must initialize with an `lvalue`
  * `lvalue`: Something that can appear on the LHS of an assignment. Something that has a memory address.
    * `int &x = 5; // illegal since 5 cannot have a memory address, rvalue`
    * `int &x = y + z; // illegal since "y + z" cannot have a memory address, rvalue`
    * `int &x = y; // legal since "y" has a memory location`
* Cannot create a pointer to a reference
* Cannot create a reference to a reference
* Cannot create an array of references

```cpp
void inc(int &n) {
  n = n + 1;
}

int x = 5;
inc(x);
cout << x; // prints 6
```

* The signature of `cin`:

```cpp
// @param in: Cannot make a copy of a stream so we must pass-by-reference
//            Want changes to stream to be visible when function is done
// @return: cin >> x evaluates to cin, so it must return {@param in}
std::istream &operator>>(std::istream &in, int &data) {
  // Method body
}
```

* Use case for big structs

```cpp
struct RellyBig {  };

void f(ReallyBig rb) {
  // Copy of ReallyBig is created, very expensive
  // Cannot modify the original copy
}

// C++ style
void g(ReallyBig &rb) {
  // pass-by-reference to avoid copying the whole struct
  // Can modify the original copy
}

// C style
void h(ReallyBig *rb) {
  // pass-by-pointer to avoid copying the whole struct
  // Can modify the original copy
}

// C++ style
void l(const ReallyBig &rb) {
  // pass-by-reference to avoid copying the whole struct
  // Cannot modify the original copy
}

g(5); // ILLEGAL
int y = 5;
g(y + y); // ILLEGAL
g(y); // LEGAL

// BUT IN C++11
// Since the pass-by-reference param is const
l(5); // LEGAL
int y = 5;
l(y + y); // LEGAL
l(y); // LEGAL
```

**Note**: Always prefer passing by reference to const

# Dynamic Memory

```cpp
// C style (not valid in this class
int *p = malloc(5);
free(p);

// C++ style
struct Node {
  int data;
  Node *next;
}
Node *np = new Node;
// np is a variable on the stack that contains the address to memory in the heap
// When np goes out of scope (popped off the stack), the heap memory will continue to exists
// Use this to "free" the memory in the heap allocated for np
delete np; // np must be created from a call "new" call
```

## Arrays

```cpp
int *arr = new int[10]; // Allocated from the heap, same as before
delete [] arr; // Syntax for deallocating arrays
delete arr; // WILL COMPILE but will cause memory leak
```
