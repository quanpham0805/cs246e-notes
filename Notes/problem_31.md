[Resolving Method Overrides at Compile-Time <<](./problem_30.md) | [**Home**](../README.md) | [>> Logging](./problem_32.md)

# Problem 31: Polymorphic Cloning
**2021-11-25**

```C++
Book *pb = ...;
Book *pb2 = // I want an exact copy of *pb;
```

Can't call constructor directly (we don't know what `*pb` is, ie. don't know which constructor to call).

**Standard Solution:** virtual `clone` method - Prototype Pattern

```C++
class Book {
    // ...
public:
    virtual Book* clone() { return new Book{*this}; }
};

class Text: public Book {
    // ...
public:
    Text* clone() override { return new Text{*this}; }
};

// Comic - similar
```
- Having different return type is fine (i.e, different signature) since `Text` and `Comic` would still be a subclass of `Book`.

Boilerplate code - can we reuse it?

Works better with an abstract base class:

```C++
class AbstractBook {
public:
    virtual AbstractBook *clone() = 0;
    virtual ~AbstractBook();
};

template<typename T> class Book_cloneable : public AbstractBook {
public:
    T* clone() override { return new T{static_cast<T&>(*this)}; } // crtp
};

class Book : public Book_cloneable<Book> {};
class Text : public Book_cloneable<Text> {};
class Comic : public Book_cloneable<Comic> {};
```

Looks good, this works better with a flat hierarchy. However, this cloning method is not generic enough
- It's tightly coupled to the `Book` hierarchy. If I want to write another class with cloning, I would need to rewrite all these again
- Reason: `Book_cloneable` is inheriting from `AbstractBook`
- Solution: use what we would learn in [problem 33](Notes/problem_33.md)

---
[Resolving Method Overrides at Compile-Time <<](./problem_30.md) | [**Home**](../README.md) | [>> Logging](./problem_32.md)
