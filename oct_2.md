# Review

```cpp
Node *np = new Node;
delete np;

int *arr = new int[10];
delete [] arr;

// A copy of n is returned and thus will be inefficient
Node getANode() {
  Node n;
  return n;
}

// BAD
// The pointer is returned which is more efficient than above.
// However, it is a dangling pointer since it points to memory than is not
//   yours to access
// n is allocated on the stack, never return a prt/reference to data the
//   function allocated on the stack
Node *getANode() {
  Node n;
  return &n;
}

// Correctly returns a pointer (not a reference) to heap allocated memory
Node *getANode() {
  Node *np = new Node{...};
  return np;
}
```

# Operator Overloading

* Idea: we can give meaning to C++ operators for user-defined types

```cpp
struct Vec {
  int x;
  int y;
};

Vec vA = new Vec{1, 2};
Vec vB = new Vec{1, 2};

// Does not compile
Vec v1 = vA + vB;

// Make an overload for the plus operator for struct Vec
Vec operator+(const Vec &v1, const Vec &v2) {
  Vec newV{v1.x + v2.x, v1.y + v2.y};
  return newV;
}
// This DOES compile
Vec v = vA + vB;

// Another example
// Commutivity does NOT apply
Vec operator*(const int k, const Vec &v1) {
  // C++ compiler is smart as fuck
  // It will know to construct a Vec
  return {k * v1.x, k * v1.y};
}
// Works
Vec v = 3 * vA;
// Does NOT work
// Need to write a function with the following signature
// Vec operator*(const Vec &v1, const int k);
Vec v = vA * 3;
```

## Another set of examples

```cpp
Grade g = new Grade{52};
cout << g << endl;

ostream &operator<<(ostream &out, const Grade &g) {
  out << g.grade << "%";
  return out;
}
```

```cpp
Grade g = new Grade{52};
cin >> g;

istream &operator>>(istream &in, Grade &g) {
  in >> g.grade;
  if (g.grade < 0) g.grade = 0;
  if (g.grade > 100) g.grade = 100;
  return in;
}
```

# C/C++ Preprocessor

* A program that runs before the compiler gets the code
* Has the ability to change code
* All code that starts with `#` is a preprocessor directive
  * `#include <...>`
    * Look first in standard library for file inside `<>`
  * `#include "..."`
    * look in local directory
* To only run the preprocessor
  * `g++14 -E -P {file}`

```cpp
#define VAR VALUE
// Preprocessing directive to replace all instance of VAR with VALUE

#define MAX 100
int array[MAX];
```

* `#define` are used for conditional compilation
  * Supposed different clients want different security strength

```cpp
#define SECLEVEL 5 // could be something else
#if SECLEVEL == 1
short int
#elif SECLEVEL == 2
long long int;
#endif
publicKey;
```

## Example

```cpp
#define LEN 2;

int main() {
  #if LEN == 1;
  short int
  #elif
  long int
  #endif
  myNum;
}
```

**Compiles to:**

```cpp


int main() {



  long int

  myNum;
}
```

## Problems

* Requires manual changing of `LEN` in the define statement

### Solution

* We can set preprocessor variables from the command-line

`g++ -DVAR=VALUE {file}`

Eg.

`g++ -LEN=1 {file}`

## ifdef

```cpp
#define BANANA

// true if BANANA is defined
#ifdef BANANA
#endif

// true if BANANA is not defined
#ifndef BANANA
#endif
```

```cpp
#define ever ;;

for (ever) {
  // code
}

// COMPILES TO...
for (;;) {
  // code
}
```

### Tricks to comment out debug statements

```cpp
#include <iostream>
using namespace std;

// file: debug.cc

int main() {
  #ifdef DEBUG
    cout << "setting x=1" << endl;
  #endif
  int x = 1;
  while (x < 10) {
    ++x;
    #ifdef DEBUG;
      cout << "x is now " << x << endl;
    #endif
  }
  cout << x << endl;
}
```

* Show the debug logs
  * `g++ -DDEBUG debug.cc`
* Don't show debug logs
  * `g++ debug.cc`

# Seperate Compilation

* Interface (.h file)
  * Type definitions
  * function declarations
* Implementation (.cc file)
  * function implementations
* Look at files in dir `~/cs246/1189/lectures/c++/separate/example1/`

## Compilation for many files

* Can use globbing pattern `g++ *.cc`
* Better to not use globbing `g++ main.cc vec.cc`
* header files are **never** compiled, they are included and compiled due to
  the preprocessing directive

> vim:tw=78:ts=2
