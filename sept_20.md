# Testing

* Testing is not the same as debugging
* Testing does not guarantee correctness
  * Can prove presence of defects

## Writing Tests

* Write small tests
* Check various classes of input
  * numeric ranges, positive, negative, zero
  * boundaries between ranges (edge cases)
  * simultaneous boundaries (corner cases)

# C++

* Simula67 - first OO language
* Created `C with classes`
  * Became `C++`
* Including `iostream` gives us access to 3 i/o variables
  1. std::cin
    * Read from stdin
    * Use the input operator `>>`
    * `char x; std::cin >> x;`
  2. std:cout
    * Write from stdout
    * Use the output operator `<<`
    * `std::out << "Hello World";`
  3. std:cerr
    * Write to stderr
    * Same as std::cout

## cin

* If a read fails due to invalid input OR if EOF is reached, variable will be set to zero
* Ignores white space
* If a read fail, the expression cin.fail() is true
* If the read fails due to EOF, both cin.fail() and cin.eof() are true
* c++ defines an automatic conversion from a istream type to booleans
  * cout and cerr have type ostream
* cin is true if the last read succeeded
  * instead of writing `cin.fail()` we can use `!cin`
* `cin >> i` will produce `cin` which is mapped to a boolean
* If a read fails, all subsequent reads will fail unless failure is acknowledged
  * use `cin.clear(); cin.ignore();`
    * acknowledged then throw away offending input
    * `cin.ignore()` ignores a single char, if "Hello" is typing in, then only "H" is ignored

## Side note

* In c++, >> can act as both the bit shift AND input operator
  * **operator overloading**

## Compiling

```sh
$ g++ -std=f++14 hello.cc -o myprog
```

**Note**: Without `-o`, the default executable is `a.out`
