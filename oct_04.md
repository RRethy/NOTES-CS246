# Separate Compilation

* Don't want to compile each file cuz that shit is tedious
* If you compile dependant files with two linux commands, you can have linker error
* By default g++ tries to compile, link and produce an executable
  * `g++ -c main.cc` to only compile `main.cc`, but it will not create an executable
    * Produces `main.o` which contains compiled code, and info about what is defined and what is required
  * `g++ main.o vec.o` will now do the linking process. Linker will simply check that all *promises* about what each program requires is kept
    * Allows you to recompile only the files that have changed
    * Means you need to keep track of what needs to be recompiled

# make

## Makefile

* Specify project dependencies
* Compile command

```make
# The full program
myprogram: main.o vec.o
  g++ main.o vec.o -o myprogram

# Recompile vec.o when vec.cc or vec.h changes
vec.o: vec.cc vec.h
  g++ -std=c++14 -c vec.cc

# Recompile main.o when main.cc or vec.h changes
main.o: main.cc vec.h
  g++ -std=c++14 -c main.cc

.PHONY: clean

clean:
  rm *.o myprogram
```

* Make knows what needs to be recompiled by looking at linux time stamp

> We can make the Makefile a bit nicer

```make
# Throw stuff into variables
CXX = g++
CXXFLAGS = -std=c++14 -Wall
OBJECTS = main.o vec.o
EXEC = myprogram

${EXEC}: main.o vec.o
	${CXX} ${CXXFLAGS} ${OBJECTS} -o ${EXEC}

vec.o: vec.cc vec.h
# Can omit the recompile lines since make is smart

main.o: main.cc vec.h
  # Code here will be executed as a shell command, try echo "hello";pwd

.PHONY: clean

clean:
	rm ${OBJECTS} ${EXEC}
```

* make is not C/C++ specific
  * All it does is look at the time stamp of files, and if it is old, then recompile based on the commands we provide
* We can use `g++ -MMD vec.cc` to create a `.d` file which is a dependency file
  * It will generate a file with `vec.o: vec.cc vec.h`

```make
CXX = g++
CXXFLAGS = -std=c++14 -Wall -MMD
EXEC = myprogram
OBJECTS = main.o vec.o
DEPENDS = ${OBJECTS:.o=.d}

${EXEC}: ${OBJECTS}
	${CXX} ${CXXFLAGS} ${OBJECTS} -o ${EXEC}

-include ${DEPENDS}

.PHONY: clean

clean:
	rm ${OBJECTS} ${EXEC} ${DEPENDS}
```

* Use this final example for make file

> TODO: Figure out exact syntax for files in nested sub directories

# Include guards

* How to avoid multiple includes
  * Avoid conflicting definitions
* We use include guard to avoid this:

> vec.h
```cpp
#ifndef VEC_H
#define VEC_H

struct Vec {
  int x;
  int y;
}

Vec operator+(const Vec &v1, const Vec &v2);
#endif
```

* You can now `#include "vec.h"` in multiple files and don't have a compiler error
* Must be unique to the program, so use a form of the name of the file
  * **Convention**: upper case, replace dot with an underscore
* *Include guards* only in header files

# Some rules

* Never compile header files
* Never include .cc files
* Do not put `using namespace std;` in header files
  * In the industry, you won't even put this in .cc files to avoid cluttering the namespace
    * Always use fullname in real life, eg. `std::string`

# C++ Classes

> student.h
```cpp
struct Student {
  int assns, mt, final;
  float grade(); // declaration
};
```

> student.cc
```cpp
#include "student.h"

float Student::grade() {
  return assns * 0.4 + mt * 0.2 + final * 0.4;
}
```

* `::` is the scope resolution operator
  * `Student::` means that in the scope of the Student struct there is a function grade
* Use the function by using the dot operator
  * `student.grade()` will evaluate to a float
* In OOP, a class is a struct type that can contain functions
  * These functions are called **methods** or member function
* An instance of a class is called an object
* Within a method, you have access to the fields of the object on which this method was called
* Within a method, you have access to `this` which is a pointer to the object on which this method was called
  * In the above case, `this == &student`

> student.cc explicit version
```cpp
#include "student.h"

// Super verbose and not necessary
// Only used if you have conflicting names with a parameter
float Student::grade() {
  return this->assns * 0.4 + this->mt * 0.2 + this->final * 0.4;
}
```

# Initializing Objects

```cpp
// C-style initializing
// Integers below are constants
Student student{60, 50, 70};
```

* We can write special objects to construct objects
  * `constructors`

> student.h
```cpp
struct Student {
  Student(int assns, int mt, int final);
};
```

> student.cc
```cpp
Student::Student(int assns, int mt, int final) {
  this->assns = assns;
  this->mt = mt;
  this->final = final;
}
```
