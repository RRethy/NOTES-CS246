# Composition, Aggregation, Inheritance

* Composition creates a OWNS-A relationship

| Car **OWNS-A** CarParts | Catalog **HAS-A** CarParts |
|-|-|
| | A class A HAS-A class B |
| | B is not copied if A is copied (shallow copy) |
| | B is not destroyed if A is |

```cpp
// Has parts, but the parts are not super dependant on the Catalog
// ie. Catalog is not responsible for copying/destroyed Part array
class Catalog {
  Part *[10]; / ptr to objects
};
```

```
UML Diagram
===========

Catalog ◇-> parts[10] -> Part
```

* The empty diamond refers to HAS-A

# Inheritance

| Book |
|-|
| title: String |
| author: String |
| numPages: Integer |

| Text |
|-|
| title: String |
| author: String |
| numPages: Integer |
| topic: String |

| Comic |
|-|
| title: String |
| author: String |
| numPages: Integer |
| hero: String |

* How to create an array for different types of Books?
* In C, use union types, void ptrs

1. Text **IS-A** Book (with an additional topic)
2. Comic **IS-A** Book (with an additional hero)

| Book |
| - |
| title: String |
| author: String |
| numPages: Integer |

> Text Inherits from Book

| Text |
| - |
| topic: String |

> Comic Inherits from Book

| Comic |
| - |
| hero: String |

* Book would be the base/superclass/parent, while text and comic is the derived/subclass/child
* **Note**: Use an arrow in UML to show inheritance

> Book class

```cpp
// book.h
class Book {
  std::string title, author;
  int length;
 public:
  Book(const std::string &title, const std::string &author, int length);
  std::string getTitle() const;
  std::string getAuthor() const;
  int getLength() const;
  bool isHeavy() const;
};

// book.cc
Book::Book(const string &title, const string &author, int length):
  title{title}, author{author}, length{length} {}

string Book::getTitle() const { return title; }
string Book::getAuthor() const { return author; }
int Book::getLength() const { return length; }
bool Book::isHeavy() const { return length > 200; }
```

> Text class

```cpp
// text.h
class Text: public Book {
  std::string topic;
 public:
  Text(const std::string &title, const std::string &author, int length, const std::string &topic);
  std::string getTopic() const;
};

// text.cc
Text::Text(const string &title, const string &author, int length, const string &topic):
  Book{title, author, length}, topic{topic} {}

string Text::getTopic() const { return topic; }
```

> Comic class

```cpp
// comic.h
class Comic: public Book {
  std::string hero;
 public:
  Comic(const std::string &title, const std::string &author, int length, const std::string &hero);
  std::string getHero() const;
};

// comic.cc
Comic::Comic(const string &title, const string &author, int length, const string &hero):
  Book{title, author, length}, hero{hero} {}

string Comic::getHero() const { return hero; }
```

* public inheritance
* Derived classes inherit members (fields/methods) from the base class
* Any method that we can call on Book, can be called on Text/Comic
* It is because of inheritance that ifstream/istringstream objects like istream objects

# Inheriting private members

* Text inherited the private fields from Book

```cpp
int main() {
  Text t = ...;
  t.author = ....; // won't compile since author is private
}
```

* But what if we are inside the Text class

```cpp
// WON'T COMPILE
Text::Text(const string &title, const string &author, int numPages, const string &topic) :
  title{title}, author{author}, numPages{numPages}, topic{topic} {
}
```

* This will not compile
  1. These inherited fields are private && MIL can only refer to fields declared in the class
  2. Steps of Object creation (for inherited classes):

    1. space is allocated - *MIL*
    2. superclass part is constructed (this step will fail) - *MIL*
    3. subclass field initialization
    4. constructor body runs

* Step 2 fails as Book does not have a default constructor

```cpp
Text::Text(const string &title, const string &author, int numPages, const string &topic) :
  Book(title, author, numPages), topic{topic} {
}
```

> Protected access

```cpp
class Book {
  protected:
    string author;
};

class Text : public Book {
  void addAuthor(string auth) {
    author += auth;
  }
};
```

* Protected gives access to class and subclass
* Private > Protected usually to maintain invariant
* A class is responsible to maintain its invariant
* Protected breaks encapsulation (like friend)
* Better to keep fields private and provide protected methods for subclass

```cpp
class Book {
  string author;
  protected:
    void addAuthor(string auth) {
      // invariant check
      author += auth;
    }
};
```

## Method overriding

* `isHeavy`
  * Book > 200
  * Text > 300
  * Comic > 30
* `isHeavy` should be overridden in each subclass

> Book class

```cpp
// Book.h
class Book {
  std::string title, author;
  int length;
 protected:
  int getLength() const;
 public:
  Book(const std::string &title, const std::string &author, int length);
  std::string getTitle() const;
  std::string getAuthor() const;
  bool isHeavy() const;
};

// Book.cc
Book::Book(const string &title, const string &author, int length):
  title{title}, author{author}, length{length} {}

int Book::getLength() const { return length; }
string Book::getTitle() const { return title; }
string Book::getAuthor() const { return author; }
bool Book::isHeavy() const { return length > 200; }
```

> Comic class

```cpp
// comic.h
class Comic: public Book {
  std::string hero;
 public:
  Comic(const std::string &title, const std::string &author, int length, const std::string &hero);
  bool isHeavy() const;
  std::string getHero() const;
};

// comic.cc
Comic::Comic(const string &title, const string &author, int length, const string &hero):
  Book{title, author, length}, hero{hero} {}

bool Comic::isHeavy() const { return getLength() > 30; }
string Comic::getHero() const { return hero; }
```

> Text class

```cpp
// text.cc
class Text: public Book {
  std::string topic;
 public:
  Text(const std::string &title, const std::string &author, int length, const std::string &topic);
  bool isHeavy();
  std::string getTopic();
};

// text.h
Text::Text(const string &title, const string &author, int length, const string &topic):
  Book{title, author, length}, topic{topic} {}

bool Text::isHeavy() { return getLength() > 400; }
string Text::getTopic() { return topic; }
```

```cpp
Book b{..., ..., 100};
b.isHeavy(); // false

Comic c{..., ..., 40};
c.isHeavy(); // true
```

* Making use of the **IS-A** relationship

```cpp
Book b = Comic{..., ..., 40, "Batman"}; // legal
b.isHeavy(); // false
```

* Inherited fields in Comic get saved and that's what Book has, but the rest is discard (such as "batman")
* Called **object coercion/slicing**
