# More I/O, stream

## Strings in C++

* C used null terminated char arrays
  * Manual memory management
* C++ provides a **<string>** header
  * Automatic resizing as needed
  * `string s = "hello";`
    * `string s{"Hello"}:`
    * `"hello"` is a C style string since it is just an array of char
  * Compare with ==, ~=, <, >, <=, >=
  * Length: `str.length();`
  * Characters: `str[0];` (Same as C with char[])
  * `cin` will stop after reading seeing a space
  * `getline(cin, s)` will get the entire line until a `\n` is encountered

## I/O manipulators (iostream, iomanip)

* Print `x` in hexadecimal
  * Remember the code is evaluated left-to-right
  * `cout << hex << x;`
  * Change how cout works
    * All future `cout` will print in hex
  * `cout << dec;`
    * Change `cout` back to decimal
  * `cout << showpoint << setprecision(3);`
    * Print to three decimal points

## Reading/writing files

* The *stream abstraction* can work on other sources of data
* Header `<fstream>`
  * `ifstream` - read from files
  * `ofstream` - write to files
* Anything we can do with cin(type istream) we can do with myfile(type ifstream)
  * Same for cout & variable of type ofstream
* When is this file close?
  * myfile is stack allocated and thus destroyed when it goes out of scope
  * When it is destroyed, it closes the file

## Reading/writing to strings

* Header `<sstream>`
  * istringstream -- read from strings
  * ostringstream -- write to strings
* `buildStrings.cc`
  * Using ostringstream and writing to a string

```cpp
#include <iostream>
#include <string>
#include <sstream>
using namespace std;

int main () {
  ostringstream ss;
  int lo {1}, hi {100};
  ss << "Enter a # between " << lo << " and " << hi;
  string s {ss.str()};
  cout << s << endl;
}
```

* `getNum.cc`
  * Insist user inputs a number

```cpp
#include <iostream>
#include <string>
#include <sstream>
using namespace std;

int main () {
  int n;
  while (true) {
    cout << "Enter a number:" << endl;
    string s;
    cin >> s;
    // if (cin == EOF) return;
    istringstream ss{s};
    if (ss >> n) break;
    cout << "I said, ";
  }
  cout << "You entered " << n << endl;
}
```

* `readIntsSS.cc`

```cpp
#include <iostream>
#include <sstream>
using namespace std;

int main () {
  string s;
  while (cin >> s) {
    istringstream ss{s};
    int n;
    if (ss >> n) cout << n << endl;
  }
}
```

## Default Arguments

* New in C++

```cpp
void printFile(string file = "myfile.txt") {
  ifstream f {file};
}

printFile();
printFile("hello.txt"):
```

* Parameters with default arguments **must** appear last
  * `void test(int num = 5, string str) {}`
    * **Illegal**
  * `void test(int num = 5, string str = "foo") {}`
    * **Legal**
    * `test(10, "bar");`
    * `test(10);`
    * `test();`
    * ^ Legal
    * This is illegal: `test("bar");` and `test(,"bar");`
  * Functions can have the same name in C++
    * Function overloading
    * Looks at function signature (looks at parameters types and/or number of parameters)
    * Differing on return type is not enough
    * default args act as if the function with that param only was implemented so it conflicts if you try to overload it

```cpp
int negate(int i) { return -i; }
bool negate(bool b) { return !b; }
```
