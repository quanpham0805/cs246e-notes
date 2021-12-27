[Logging <<](./problem_32.md) | [**Home**](../README.md) | [>> Policies](./problem_34.md) 
# Problem 33: Generalize the Visitor Pattern! Part 2!
## **2021-11-25**

Recall - we generalized visitor to automatically create the `Visitor` superclass.

Now - use CRTP to add `accept` methods to the `Book` hierarchy.

```cpp
class AbstractBook {
public: 
    // ...
    virtual void accept (BookVisitor& bv) = 0;
    virtual ~AbstractBook() = default;
};

template <typename C> class BookAcceptor : public AbstractBook {
public:
    void accept(BookVisitor& bv) override {
        bv.visit(*static_cast<C*>(this));
    }
};

class Text : public BookAcceptor<Text> {/* ... */};
// etc
```
- This acceptor is depending on the `Book` hierarchy (unnecessary coupling, still lots of boilerplate code)
- `BookAcceptor` inherits from `AbstractBook` and accept a `BookVisitor`
- If I want to make another acceptor for another visitor, I have to write all this again

Let's abstract these:
```cpp
template <typename B, typename C, typename V> 
class Acceptor : public B { // mixin class
public:
    void accept(V& v) override { v.visit(*static_cast<C*>(this)); }
};

class Text : public Acceptor<AbstractBook, Text, BookVisitor> {
    // ...
};
```
- Now I don't need to write all those boilerplate code again, I can put this in a library and it would work on its own.

So far, these things work because they don't have any field, but what if the abstract superclass has fields?
```cpp
class AbstractBook {
    string title, author;
    int length;
public:
    AbstractBook(string title, string author, int length);
    virtual void accept(BookVisitor& bv) = 0;
    virtual ~AbstractBook() = default;
};

template <typename B, typename C, typename V>
class Acceptor : public B {
    // same as above
};

class Text : public Acceptor<AbstractBook, Text, BookVisitor> {
    string topic;
    // ...
};
```
- `AbstractBook` has fields, and so it has a non-trivial constructor, which must be called to initialize the `Text` object.
- `Text` cannot call `AbstractBook`'s ctor because it is not the direct parent of `Text`. 
- `Text` can only call `Acceptor`'s ctor
- `Acceptor` cannot call `AbstractBook`'s ctor because it doesn't know what its parent is!
    - Well, parent is `B`, but `B` could be anything.
    - The intermediate class in this case is too generic so it cannot call the parent's ctor.
- Good news! There is a one liner solution to this!
```cpp
template <typename B, typename C, typename V>
class Acceptor : public B {
public:
    using B::B; // here is this one line
    void accept(V& v) override { v.visit(*static_cast<C*>(this)); }
};
```
- The `using` line doesn't _just_ bring back the ctor into scope like what we used before (that wouldn't make sense anyway).
- It brings all of the `B`'s ctors into scope within `Acceptor`, rename them into `Acceptor`, and simply passing through `B`'s actual ctors.
- Now I can do
```cpp
Text::Text(string title, string author, int length, string topic) :
    Acceptor{title, author, length}, topic{topic} {}
```

Exercise: apply this to the cloning problem.


---
[Logging <<](./problem_32.md) | [**Home**](../README.md) | [>> Policies](./problem_34.md) 
