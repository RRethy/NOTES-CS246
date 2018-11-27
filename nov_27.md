# Exception Safety

```cpp
class A {};
class B {};
class C {
  A a;
  B b;

  void f() {
    a.method1(); // strong guarantee
    b.method2(); // strong guarantee
  }
};
```

* if method1 throws, f has a strong guarantee
* if method2 throw, we need to undo what method1 did
  * often undos are not possible
  * `f()` does not have a strong guarantee
* Suppose method1/method2 only have local side effects can we rewrite f to provide a strong guarantee?
* Lets call these methods on copies of a and b

```cpp
void C::f() {
  A atemp{a};
  B btemp{b};
  atemp.method1();
  btemp.method2();
  a = atemp;
  b = btemp; // this can throw an exception and thus f() does not have a strong guarantee
}
```

* `f()` would have a strong guarantee if B::operator= is no throw

```cpp
struct CImpl { A a; B b; };
class C {
  unique_ptr<CImpl> pImpl;

  /**
   * f() now has a strong guarantee
   */
  void f() {
    unique_ptr<CImpl> temp{new CImpl(*pImpl)};
    // auto temp = make_unique<CImpl>(*pImpl);

    temp->a.method1();
    temp->b.method2();

    swap(pImpl, temp);
  }
};
```

## Exception Safety in the STL

* `std::vector` - uses RAII
* wraps around a heap array
* destructor deallocates array

```cpp
void f() {
  std::vector<MyClass> v;
} // When v is destroyed, the elements (Objects) are also destroyed
```

```cpp
void g() {
  vector<MyClass *> v;
} // When v is destroyed, the ptrs are not deleted

// g() is not exception safe
void g() {
  vector<MyClass *> v;
  // We have to explicitly delete the contents of v
  for (auto p : v) delete p; // But now we are calling delete
}
```

```cpp
void g() {
  vector<unique_ptr<MyClass>> v;
}
```

* `std::vector::emplace_back` provides a strong guarantee
* **Easy case**: size < capacity
* **Hard case**: size == capacity
  * This is exception safe:
  * Create larger array
  * copy from old to new
  * swap old and new and delete old
  * delete is no throw
  * **This is inefficient though since copying not moving**
* We should move elements to new array
  * if an exception occurs during the move, we lose our strong guarantee
  * `emplace_back` checks if the move has a no throw guarantee (`noexcept`) and uses it, otherwise copies are used

# Casting

> C-style casting

```c
Node n;
int *p = (int *) &n;
```

* Four different kinds of C++ casting:

1. `static_cast` - sensible cast since behaviour is well-defined (compile checks that it is possible)
2. `reinterpret_cast` - anything goes, relies on compiler dependant decisions on how objects appear in memory
3. `const_cast` - Used to remove `const`
4. `dynamic_cast` - see next notes

> `static_cast`

```cpp
void f(int);
void f(double);

double a = 12.2;
f(a); // calls f(double)
f(static_cast<int>(a)); // calls f(int)

Book *bp = new Text{...};
// To call getTopic, we need a Text *
Text *tp = static_cast<Text *>(bp);
// There must be an IS-A relationship between the current type and the requested type
// This is a downcast, unchecked cast
// If your assumption is not correct, behaviour is undefined
```

> `reinterpret_cast`

```cpp
Student *s = ...;
Turtle *t = reinterpret_cast<Student *>(s);
t->draw();
// Casting an Object rather than a ptr would cause slicing
```

> `const_cast`

```cpp
void g(int *p);
const int *q = ...;
g(q); // won't compile due to const
g(const_cast<int *>(q)); // will compile
// But if q is in read-only memory, then if g() modifies q, then the program will crash
```
