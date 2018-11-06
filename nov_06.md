# Templates, STL, Exceptions

## Last time

```cpp
template <typename T>
class Stack {
  int size;
  int capacity;
  T *contents;
  public:
    void push(T x) {}
    T top() {}
    void pop() {}
    ~Stack() {}
};
```

```cpp
Stack<int> sInts;
Stack<string> sStrings;
```

> Template List Class

```cpp
template <typename T>
class List {
  struct Node {
    T data;
    Node *next;
  };
  Node *thelist = nullptr;

  public:
    class Iterator {
      Node *cur;
      Iterator(...) {...}
      public:
        T &operator*() {...}
        Iterator &operator++() {...}
        friend class List<T>;
    }
    T ith(int i) {...}
    void addToFront(T &data) {...}
};

List<int> loi;
List<string> los;

for (List<int>::Iterator it = loi.begin(); it != loi.end(); ++it) 
  cout << *it << endl;
}

List<List<int>> loloi;
```

## STL: Standard Template Library

* `std::vector`
  * Java equivalent would be `ArrayList`
  * Dynamically resizing array of generic contents, but it avoids costly copying
  * Can shrink accordingly as well, `std::vector::resize()`

```cpp
include <vector>
vector<int> v{1, 2}; // contains vector that has an array with elements [1, 2]
v.emplace_back(3); // add to end, [1, 2, 3], uses move ctor, prefer this
v.push_back(4); // add to end, [1, 2, 3, 4], uses copy ctor
vector<int> v2(4, 5); // contains vector that has an array with elements [5, 5, 5, 5]

for (int i = 0; i < v.size(); ++i) {
  cout << v[i] << endl;
}

for (vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
  cout << *it << endl;
}

for (auto n : v) {
  cout << n << endl;
}

for (vector<int>::reverse_iterator rit = v.rbegin(); rit != v.rend(); ++rit) {
  cout << *rit << endl;
}

v.pop_back(); // remove last element, now we have essentially a stack
```

* Many STL methods take iterators as parameters
  * `v.erase(v.begin());` to remove first, like a queue
    * time: O(n)
    * now old iterator is wrong since elements are shift
    * returns an iterator pointing to the element after the element that was reversed
  * `v.erase(v.end() - 1);` to remove last
* `v[i]` gets the ith element, unchecked access
  * `i` can be out of range, no index bounds checking
* `v.at(i)` gets the ith element, checked access
  * `i` cannot be out of range, an exception will be thrown

## Exceptions

* `v.at(i);`
  * out of bounds gets discovered in the `vector::at(int)` method, that needs to get bubbled up the call stack
  * if `i` is out of bounds, a error will be thrown, you must catch the exception at the call site or the program will terminate

```cpp
// Program will terminate prematurely due to the exception
int main () {
  vector<int> v;
  v.emplace_back(2);
  v.emplace_back(4);
  v.emplace_back(6);

  cout << v.at(3) << endl;  // out of range
}
```

```cpp
#include <stdexcept>

// error is caught, program will print err msg
int main () {
  vector<int> v;
  v.emplace_back(2);
  v.emplace_back(4);
  v.emplace_back(6);

  try {
    cout << v.at(3) << endl;  // out of range
    cout << "Done with try" << endl;
  } catch (out_of_range r) { // out_of_range is a class that inherits from exception in stdexcept
    cerr << "Bad range " << r.what() << endl;
  }

  cout << "Done" << endl;
}
```

```cpp
// call chain and throwing an exception
// stack unwinding
void f () {
  cout << "Start f" << endl;
  throw (out_of_range("f")); // exception is thrown
  cout << "Finish f" << endl;
}

void g() {
  cout << "Start g" << endl;    f ();    cout << "Finish g" << endl;
}

void h() {
  cout << "Start h" << endl;    g ();    cout << "Finish h" << endl;
}

int main () {
  cout << "Start main" << endl;
  try {
    h();
  }
  catch (out_of_range) { cerr << "Range error" << endl; }
  cout << "Finish main" << endl; // this will be triggered
}
```

* This is called stack unwinding
* When an exception is thrown, the program begins to look for a catch block. This might cause stack frames to be popped.
  * If a catch block is found, code resumes there, otherwise program gets killed

```cpp
// Can have multiple catch blocks
try {
} catch (out_of_range e) {
} catch (bad_alloc e) {
}
```

### Error recovery can be done in stages

```cpp
try {
} catch (some_exception e) { // catch by value, slicing can occur to due to polymorphism
  throw some_unrelated_exception{};
  throw e; // throws whatever was caught in e
  throw; // throws the original exception, no slicing
}
```

* All c++ library exceptions inherit from `exception` class
  * but in c++ you can throw anything (`int`, `string`, `exception`, etc.)
* To catch **all** exceptions, use `...`

```cpp
try {
} catch (...) { // not very descriptive however, very hacky, don't do this please
}
```
