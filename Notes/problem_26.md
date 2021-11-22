[Abstraction over Iterators <<](./problem_25.md) | [**Home**](../README.md) | [>> I want an even faster vector](./problem_27.md)

# Problem 26: Generalize the Visitor Pattern
## **2021-11-18**

Recall the visitor pattern:
```C++
class Book {
    // ...
public:
    virtual void accept (BookVisitor &v) { v.visit(*this); }
};

class Test: public Book {
    // ...
};

class BookVisitor {
public: 
    virtual void visit (Book& b) = 0;
    virtual void visit (Text& t) = 0;
    virtual void visit (Comic& c) = 0;
}
```
- Can we automate this process of creating `visit`? I mean we have **template metaprogramming**

Consider the following:

_visitor.h_
```C++
// right now it would work no matter how many arguments I supply, we will specialize the template for the case when the list of arguments is not empty, so this will only trigger when this is empty
template <typename...> class Visitor {
public:
    void visit();
    virtual ~Visitor() {}
};

// this template has at least 1 known first item
template <typename T, typename... Ts> class Visitor<T, Ts...> : public Visitor<Ts...> {
public:
    // here I inherit whatever visit my parent has, bring parents' visit to the scope
    using Visitor<Ts...>::visit;
    // and I add a visit method that takes a T&
    // for the code above, if we unwind the recursion, we know at each level we would get a visit, until
    // we reach the top level (which is the previous Visitor class), which we would only
    // have a visit method that takes in nothing (trivial case)
    virtual void visit(T& b) = 0;
}
```
- You might be wondering why we need to bring the parent's `visit` methods into scope. Aren’t they already present due to the public superclass declaration? Well,
  - If you overload a method that you are inheriting, so your parent gives you a method with one signature, and you give the same method with another signature, they are not considered equivalent. 
  - In terms of overload resolution, the one in your scope will take priority over anything else. 
  - So in order to make an inherited method that is an overload of a method that you have, operate at the same level of preference for overload resolution, you have to use a `using` to bring it into your scope.
  - That's true whether you are writing templates, or you are writing ordinary classes.

Anyway, now, what we can do is:

_BookVisitor.h_
```C++
class Book; class Text; class Comic; // forward declaration

// this will generate the old BookVisitor
using BookVisitor = Visitor<Book, Text, Comic>;
```

---
[Abstraction over Iterators <<](./problem_25.md) | [**Home**](../README.md) | [>> I want an even faster vector](./problem_27.md)
