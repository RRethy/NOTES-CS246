# Casting

* 4 kinds of casts (see last time)

> `dynamic_cast`

```cpp
vector<Book *> myBooks;
Book *bp = myBooks[0];
Comic *cp = dynamic_cast<Comic *>(bp); // If this fails, then nullptr is returned
if (cp) cout << cp->getHero();
else cout << "Not a Comic";
```

* Tentatively try the cast
  * if successful, cp is a valid Comic ptr
  * if not successful, cp becomes nullptr

## Casting with share ptrs

* used to cast a `shared_ptr` to another `shared_ptr` object

1. `static_pointer_cast`
2. `const_pointer_cast`
3. `dynamic_pointer_cast`

## Runtime-Type Information (RTTI)

```cpp
void whatIsIt(shared_ptr<Book> b) {
  if (dynamic_pointer_cast<Comic>(b)) cout << "Comic";
  else if (dynamic_pointer_cast<Text>(b)) cout << "Text";
  else cout << "Book";
}
```

* The code is tightly coupled with the class hierarchy, the above code would therefore be bad practice. Prefer `virtual` methods.

> `dynamic_cast` for references

```cpp
Comic c{};
Book &b{c};
Comic &c2 = dynamic_cast<Comic &>(b);
// if successful, c2 is a valid reference to the Comic
// if not successful, a bad_cast exception is thrown
```

> Remember from TODO: figured out which notes this example belongs to

```cpp
Text t1{}, t2{};
Book *bp1{&t1};
Book *bp2{&t2};
*bp1 = *bp2;
```

* Resolve partial assignment by making operator= virtual

```cpp
class Book {
  public:
    virtual Book &opeator=(const Book &other);
};

class Book {
  public:
    Text &opeator=(const Book &other) override {
      const Text &temp = dynamic_cast<const Text &>(other);
      Book::operator=(other);
      topic = temp.topic;
      return *this;
    }
};

Comic c{};
Book *bp3{&c};
*bp1 = *bp3; // This could throw an exception
```

# Template Functions

```cpp
template<typename T>
T main(T x, T y) {
  return x < y ? x : y;
}

int x = 5, y = 7;
int result = min(x, y); // min<int>(x, y);
// type of T is automatically inferred
```

```cpp
template<typename Func>
void foreach(AbsIter &start, AbsIter &finish, Func f) {
  while (start != finish) {
    f(*start);
    ++start;
  }
}
```

* Instead of hard coding the type `AbsIter`, we can make it a template parameter as well

```cpp
template<typename Iter, typename Func>
void foreach(Iter start, Iter finish, Func f) {
  while (start != finish) {
    f(*start);
    ++start;
  }
}

void foo(int n) {
  cout << n << endl;
}

int a[] = { 1, 2, 3, 4 };
foreach(a, a + 4, foo);
```

## std::algorithm

1. `std::for_each`
2. `std::find`
3. `std::find_if`
4. `std::find_if_not`
5. `std::copy`
6. `std::transform`

> `std::find`

```cpp
template<typename Iter, typename T>
Iter find(Iter first, Iter last, const T &val) {
  // Search for val within the range [first, last)
  // If val is not found, then return last
}
```

> `std::copy`

```cpp
template<typename InIter, typename OutIter>
OutIter copy(InIter first, InIter last, OutIter result) {
  // copy one container's range [first, last) into another starting at result
  // Require: result has enough space, no memory allocation will be performed
}

vector<int> v{ 1, 2, 3, 4, 5, 6, 7 };
vector<int> w(4); // Reserve space for 4 ints
copy(v.begin() + 5, v.begin() + 5, w.begin());
```

> `std::transform`

```cpp
template<typename InIter, typename OutIter, typename Func>
OutIter transform(InIter first, InIter last, OutIter result, Func f) {
  while (first != last) {
    *result = f(*first);
    ++first;
    ++result;
  }
  return result;
}

int add1(int n) {
  return n + 1;
}

vector<int> v{ 1, 2, 3, 4, 5, 6, 7 };
vector<int> v2(v.size());
transform(v.begin(), v.end(), v2.begin(), add1);
// Now v2 = { 2, 3, 4, 5, 6, 7, 8 };
```

# How Virtual works

> CS444 will implement this

```cpp
class C {
  int x;
  public:
    virtual void foo();
    void bar();
    ~C();
};

C c;
sizeof(c); // atleast 12 due to ptr to foo()
```

* every time a class has a virtual method, objects of that class contain a ptr "virtual pointer" (vptr)
  * Needed to implement dynamic dispatch
* `c.bar();`
  * Compiler finds the memory address for bar function at compile time
* `c.foo();`
  * For every class that has a virtual method, a single virtual table is created (vtable)
  * vptrs point to virtual tables

> virtual table, contains address of different virtual methods

| "C" |
| - |
| addresses of foo |
| addresses of destructor |

* Object C has a vptr to the table above (inside the UML for Object C)

* `p->foo();`
  * Follow object to the Object, then follow the vptr to the vtable, then find the function address
  * much more complex than a non virtual method
  * different children of Object C would have different vptrs
    * If the virtual method is no overridden in a child, then the address to `foo()` would be the same as the address in the parent
