#  TODO

# Visitor design pattern

* TODO: get these notes from elsewhere

## NVI Idiom (Non-virtual interface)

1. all public methods are non virtual
2. all virtual methods should be private/protected

* **Exception**: the destructor which is public/protected

```cpp
// not using NVI
class DigitalMedia {
  public:
    virtual void play() = 0;
    virtual ~DigitalMedia();
};

// using NVI
class DigitalMedia {
  public:
    virtual void play() {
      // we could update analytics here or whatever we want
      doPlay();
    }
    virtual ~DigitalMedia();

  private:
    virtual void doPlay() = 0;
};
```

# std::map

* generalization of an array
* template class parametrized on two types: the key and the value

```cpp
#include <map>

map<string, int> m;
m["abc"] = 1;
m["def"] = 2;
std::cout << m["abc"] << std::endl;
m.erase("abc");
std::cout << m["xyz"] << endl; // key not in map, it gets inserted with a default value (0)

// to search for a key in a map
if (m.count("pqr")) { // 0 if not found, 1 if found
}

for (auto &p : m) {
  // p is of type std::pair
  std::cout << p.first << p.second << std::endl;
}
```

* map is a binary tree, the key needs to support the `<` operator

# Visitor Design Pattern

* used to do **double dispatch**
* **virtual**: choose the method to execute based on runtime type of the object
* what if the choice of the method to run depends on 2 objects

* Assume we have an `Enemy` class with subclasses `Turtle` and `Bullet`
* Assume we have an `Weapon` class with subclasses `Stick` and `Rock`

```cpp
virtual void Enemy::strike(Stick &) = 0;
virtual void Enemy::strike(Rock &) = 0;

while (player->notDead()) {
  e = l->createEnemy();
  w = player->chooseWeapon();
  e->strike(*w);
}
```

* VDP uses a combination of overriding and overloading

```cpp
class Enemy {
  public:
    virtual void strike(Weapon &) = 0;
};

class Turtle : public Enemy {
  public:
    void strike(Weapon &w) override {
      w.useOn(*this);
    }
};

class Bullet : public Enemy {
  public:
    void strike(Weapon &w) override {
      w.useOn(*this);
    }
};

class Weapon {
  public:
    virtual void useOn(Turtle &) = 0;
    virtual void useOn(Bullet &) = 0;
};

class Stick : public Weapon {
  public:
    void useOn(Turtle &) override {
    }

    void useOn(Bullet &) override {
    }
};

class Rock : public Weapon {
  public:
    void useOn(Turtle &) override {
    }

    void useOn(Bullet &) override {
    }
};
```

## Another VDP

1. separation of concerns
2. adding functionality without cluttering classes with new virtual methods

* requires class hierarchy to be setup to accept visitors

```cpp
class Book {
  public:
    virtual void accept(BookVisitor &v) {
      v.visit(*this);
    }
};

class Text : public Book {
  public:
    void accept(BookVisitor &v) override {
      v.visit(*this);
    }
};

class Comic : public Book {
  public:
    void accept(BookVisitor &v) override {
      v.visit(*this);
    }
};

class BookVisitor {
  public:
    virtual void visit(Book &) = 0;
    virtual void visit(Comic &) = 0;
    virtual void visit(Text &) = 0;
};
```

### Catalog Books

* count Books based on author
* count Text based on topic
* count Comic based on hero

```cpp
class Catalog : public BookVisitor {
  std::map<string, int> catalog;

  public:
    void visit(Book &b) {
      ++cat[b.getAuthor()];
    }
    void visit(Text &b) {
      ++cat[b.getTopic()];
    }
    void visit(Comic &b) {
      ++cat[b.getHero()];
    }
};
```
