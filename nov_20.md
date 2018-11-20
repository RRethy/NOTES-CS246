# Visitor Design Pattern and include vs forward declaration

* Files: `se/visitor`
  * Has a cyclic include and won't compile
  * An include creates a compilation dependency
  * Often, a forward declaration of a class is all we need (rather than the include)
    * `class XYZ; // you are telling the compiler to trust that class XYZ is getting compiled`
  * **Whenever possible**, prefer forward declaration to include:

1. reduces compilation dependencies and reduce circular dependencies
2. reduce compile time
3. reduces frequency of compilation for a file

> File: a.h

```cpp
class A {
};
```

> File: b.h

```cpp
#include "a.h"

class B : public A {
};
```

> File: c.h

```cpp
#include "a.h"

class C {
  A a;
};
```

> File: d.h

```cpp
class A;

class D {
  A *ptrA;
};
```

> File: d.cc

```cpp
#include "a.h"

void D::foo() {
  ptrA->someMethod();
}
```

> File: e.h

```cpp
class A;

class E {
  A f(A x);
};
```

> File: e.cc

```cpp
A E::f(A x) {
}
```

# Reducing Compilation Dependencies

> File: window.h

```cpp
#include <xlib/x11.h>

class XWindow {
  Display *d;
  Window w;

  public:
    void draw();
};
```

> File: client.cc

```cpp
#include "window.h"

myXWindow->draw() {
}
```

* `client.cc` has to recompile even if private members in window.h change

## Solution: Ptr to Implementation (pImpl idiom)

* Take the private implementation out of window.h

> File: XWindowImpl.h

```cpp
struct XWindowImpl {
  Display *d;
  WIndow w;
};
```

> File: window.h

```cpp
struct XWindowImpl;

class XWindow {
  XWindowImpl *pImpl;
  public:
    void draw();
};
```

> File: window.cc

```cpp
#include "xwindowimpl.h"
#include "window.h"

XWindow::XWindow() :
  pImpl{new XWindowImpl()} {
  pImpl->d = ...;
}
```

* If private implementation changes, window.h and therefor client.cc are not affected
* In UML:

| XWwindow |
| - |

◆
|
↓

| XWindowImpl |
| - |

### Generalization: Bridge Pattern


| XWwindow |
| - |

◆
|
↓

| *XWindowImpl* |
| - |

* *XWindowImpl* will have bunch of implementations

### Terms

* **Coupling**: degree to which modules/classes interact with each other
  * **low coupling**: interaction through a public interface - **preferred**
  * **high coupling**: interaction through a public implementation

* **Cohesion**: how related are things within a module
  * **low cohesion**: loosely connected, such as `<utility>` which has loads of unrelated things
  * **high cohesion**: module/class achieves exactly a single task - **preferred**

### Decoupling the interface

```cpp
class ChessBoard {
  void foo() {
    cout << "Your move";
  }
};
```

* `ChessBoard` is now coupled with `stdout`
* Slightly better would be to use an `ostream` variable
* `ChessBoard` class has 2 responsibilities: game state, communication
* Violates single responsibility principle
* It would be better to have a separate communication class

# MVC (Model-View-Controller)

* Multiple opinions on implementation
1. `Controller` tells the `Model` something has changed, the `Model` then notifies the `View` which then queries from the `Model`
2. `Controller` tells the `Model` something has changed, the `Model` then updates the `Controller` and the `Controller` notifies the `View`

# Exception Safety

```cpp
void f() {
  MyClass *p = new MyClass();
  MyClass c;
  g(); // if g() throws an exception, the next line won't get hit and p will leak
  delete p;
}
```
