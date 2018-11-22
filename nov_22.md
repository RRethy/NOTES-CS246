# Exception Safety

* Want our program to recover from exceptions
  * Program to not leak memory
  * Want to stay in a consistent state (no dangling ptrs, no broken invariants)
  * C++ guarentee: during stack unwinding all stack allocated objects are destroyed, destructors run and memory is reclaimed

> Old f method

```cpp
void f() {
  MyClass *p = new MyClass();
  MyClass m;
  g(); // g() might throw an exception
  delete p;
}
```

> Refactor f method

```cpp
void f() {
  MyClass *p = new MyClass();
  try {
  MyClass m;
  } catch (...) {
    delete p;
    throw;
  }
  delete p;
}
```

* But this code is super complex, tedious, and very error prone
* Want a way to guarantee that some code runs irrespective of whether an exception occurs or not

| Language | Mechanism |
| - | - |
| Java | finally |
| Scheme | dynamic-wind |
| C++ | nothing |

* in C++, we need to rely on the guarantee for stack objects
  * Maximize stack usage

## C++ idiom: RAII: Resource Acquisition Is Initialization
  * Every resource should be wrapped within a stack allocated object whose destructor released the resource

```cpp
int main() {
  ifstream f{"file.txt"}; // automatically opens the file
}
```

* The file resource is released (closed) when the stack object goes out of scope

### RAII for heap memory

* Wrap the heap object within a stack object whose destructor deallocates the heap object
* STL provides a template class to do this
  * `std::unique_ptr<T>`
    * Constructor takes a T\* as an argument
    * destructor deletes the ptr
    * overloads operator\*, operator->

```cpp
void f() {
  std::unique_ptr<MyClass> p{new MyClass};
  // auto p = std::make_unique<MyClass>();
  MyClass m;
  g();
  // Don't need to delete p since it is stack allocated and will clean the heap allocated MyClass up
}
```

* TODO: `c++/unique_ptr/example1.cc`

## Copying vs Moving `unique_ptr`

```cpp
class c{...};
auto p = make_unique<c>();
std::unique_ptr<c>;
unique_ptr<c> q = p; // won't compile
```

* `unique_ptr` does not have a copy constructor
* `unique_ptr` cannot be copied as then you would have multiple stack objects pointing tot he same heap object
  * double `delete` will get called
* Sample implementation of `unique_ptr`

> TODO: `c++/unique_ptr/basicimpl.cc`

* Note that `void f() = delete;` disables method `f()`

## Ownership

* Who owns the heap resource
* If code does not own the heap resource, it can be provided the "raw" pointer
  * Use `get()` to access the raw ptr
* If there are multiple owners, use `std::shared_ptr`

```cpp
void g() {
  std::shared_ptr<MyClass> p1 = make_shared<MyClass>();
  if () {
    auto p2 = p1; // copy constructor
  } // p2 goes out of scope (won't deleted MyClass object as p1 still points to it)
} // p1 out of scope, destructor will delete MyClass ptrs
```

* `shared_ptr` uses reference counting
  * destructor only deletes heap memory when ref count reaches 0

## 3 Levels of Exception Safety

1. Basic Guarantee: if an exception occurs the program is in a valid but unspecified state (no memory leaks, no dangling ptrs, etc.)
2. Strong Guarantee: if an exception occurs during f, it is as if f was never called
3. No-throw Guarentee: f does not throw exceptions, achieves its task
