* C++11 introduced rvalue reference
  * `Node &other` - lvalue reference
  * `Node &&other` - rvalue reference
* rvalue reference is a reference to a tmp value (one that is about to be destroyed)

# Move constructor

* Takes one parameter - rvalue reference

```
other -> Node(2) -> Node(3)
this -> Node(2) -/^
```

* `Node(2)` is copied, `Node(3)` is shared
* **Shallow copy**

```cpp
Node::Node(Node &&other) :
    data{other.data},
    next{other.next}, {
  // Old Node(2) above has the ptr removed
  other.next = nullptr;
}
```

# Move assignment operator

```cpp
Node m{1, new Node{2, nullptr}}; // notice Node(1) is stack while Node(2) is heap
m = plusOne(m); // m would otherwise get copied then destroyed
```

> Memory diagram for above code

```
other -> Node(2) -> Node(3)
m -> Node(1) -> Node(2)

AFTER move assignment operator

other -> Node(1) -\ Node(3)
m -> Node(2) -/^ Node(2)
```

> Move assignment operator

```cpp
Node &Node::operator=(Node &&other) {
  std::swap(other); // copy-swap idiom from oct_16.md
  return *this;
}
```

* If a move constructor/operator= is available, it will be used whenever the RHS is an rvalue reference
* The default move constructor/operator= go away if you write any of the big 5

# Rule of 5

* If you need to implement any of the following, then usually need to implement all 5:
  1. copy constructor
  2. copy operator
  3. destructor
  4. move constructor
  5. move operator=

# Copy/move elision

```cpp
Vec makeVec() { return {0, 0}; }
Vec v = makeVec();
```

* Intuitively either move or copy constructor would be called
  * But it doesn't
* The compiler optimizes this so Vec{0, 0} is created in the space for variable `v`
* C++ *allows* compilers to avoid calling copy/move constructors even if this would change program behaviour
  * You are able to turn off optimizations: `g++14 -fno-elide-constructors`

# Operators - functions or methods?

* `operator=` - must be implemented as a method
  * `n1 = n2` --- `n1.operator=(n2)`
* The LHS is represented by `*this`

```cpp
Vec v1 = {1, 2};
Vec v2 = {3, 4};
v1 + v2;
v1 * 5;
5 * v2;
```

```cpp
struct Vec {
  int x,y;
  Vec operator+(const Vec &);
  Vec operator*(const int);
};

// Cannot go inside the Vec struct
Vec operator*(const in, const Vec &);
```

> These are **methods**, these are **not functions**
```cpp
Vec Vec::operator*(const int k) {
  return { x * k, y * k };
}

ostream &Vec::operator<<(ostream &out) {
  out << x << "  " << y;
  return out;
}
```

* The above must be called as follows: `v1 << cout` and `v2 << (v1 << cout)`
  * Don't do this

> These are **functions**
```cpp
Vec operator*(const int k, const Vec &v) {
  return { v.x * k, v.y * k };
}

ostream operator<<(ostream &out, const Vec &v) {
  out << v.x << "  " << v.y;
  return out;
}
```

* However, the following must be implemented as methods (not too important)
  1. operator=
  2. operator[]
  3. operator->
  4. operator()
  5. operator T()

# Arrays of objects

```cpp
struct Vec {
  // free no parameter constructor goes away
  // Vec v[10] also doesn't work since no default constructor
  Vec(int x, int y) : x{x}, y{y};
};
```

* How to fix?
  1. implement a 0 parameter constructor
  2. for stack arrays, use array initialization: `Vec v3 = {Vec{0, 0}, Vec{0, 0}, Vec{0, 0}}`
  3. use array of pointers to objects

> number 3 from above

```cpp
// stack
Vec *arr[3];

// heap
Vec **arr = new Vec*[3];
array[0] = new Vec{0, 0};
for (int i = 0; i < 3; ++i) {
  delete array[i];
}
delete [] array;
```

# const methods

```cpp
struct Student {
  int assns, mt, final;
  float grade() {
    return 0.4 * assns + 0.2 * mt + 0.f * final;
  }
};

const Student billy{80, 50, 75};
billy.grade(); // won't compile since grade() doesn't promise it won't modify fields

// Must use const as follows
struct Student {
  int assns, mt, final;
  float grade() const {
    return 0.4 * assns + 0.2 * mt + 0.f * final;
  }
};
```

* a const method promises to not change field value of `*this`
* const objects can only call const methods
* **const** applies to the thing on the left, unless there is nothing on the left, then it applies to the thing on the right
  * `const int k` == `int const k`

# Invariants and Encapsulation

## Invariants

* In Node class `next` always points to heap memory or is nullptr
  * `Node n{...};`
  * `Node m{2, &n};` // should crash since n is on the stack not the heap
