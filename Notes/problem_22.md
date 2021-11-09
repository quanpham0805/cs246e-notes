[I want a class with no objects << ](./problem_21.md) | [**Home**](../README.md) | [>> I want to know what kind of Book I have](./problem_23.md)

# Problem 22 - The copier is broken (again)
## **2021-10-28**

How do copies and moves interact with inheritance?

**Copy constructor:** 
```C++
Text::Text(const Text &other): Book{other}, topic{other.topic} {}
```

**Move constructor:** 
```C++
Text::Text(const Text &other): 
    Book{std::move(other)}, 
    topic{std::move(other.topic)} {}
```

We can still call `std::move(other.topic)` even though we already moved other because `std::move(other)` only took the "`Book` part", leaving `other.topic` behind.

**Copy assignment:**
```C++
Text &Text::operator=(const Text& other) {
    Book::operator=(other);
    topic = other.topic;
}
```

**Move assignment:**
```C++
Text &Text::operator=(Text&& other) {
    Book::operator=(std::move(other));
    topic = std::move(other.topic);
}
```
- We don't do unified assignment here, because it would cause problem later, not necesarily for `Text`, but probably for `Book`, because it doesn't work that well when you have inheritance, because pass-by-value implies slicing. We want to avoid that.

But consider:
```C++
Book *b1 = new Text{...};  // Author1 writes about BASIC
Book *b2 = new Text{...};  // Author2 writes C++

// what happens here???
*b1 = *b2;
```
Now Author2 writes about BASIC and Author1 writes about C++
- Essentially only the book part gets copied over
- **Partial Assignment**
- `topic` doesn't match `title` and `author` - as a `Book` this is valid, as a `Text` it is corrupted

Possible solution: make `operator=` virtual

```C++
class Book {
        // ...
    public:
        virtual Book &operator=(const Book &other);
        // ...
};

class Text: public Book {
        // ...
    public:
        Text &operator=(const Text &other) override;
        // ...
};
```
Doesn't actually compile, `Text &operator=` must take a `Book` or it's not an override (method signatures differ)
```C++
class Text: public Book {
        ...
    public:
        Text &operator=(const Book &other) override;
        ...
};
```
- <details open>
  <summary>Rants</summary>

  - We are coming up a fundamental flaw in OOP. It exists in every strongly-typed OOP languages, not just C++.
  - If a language does not accept `Book` there, that language is flawed.
  - Public inheritance represents "is of" relationship (`Text` is a special kind of `Book`, everything you can do with a `Book`, you can do with `Text` too).
  </details>

But then we could pass a `Comic` which is also a problem. We will revisit this later.

Also note that we can return a `Text &` as opposed to a `Book &` in `class Text`. This is because we're returning a reference of a subclass, which is allowed.

**Another solution:** make all superclasses abstract

Instead of:

- `Book`
    - `Text`
    - `Comic`
  
We want 
- _`AbstractBook`_
    - `NormalBook`
    - `Text`
    - `Comic`

```C++ 
class AbstractBook {
    // ...
    protected:
        AbstractBook &operator=(AbstractBook other) { ... } // Non-virtual
};

class Text: public AbstractBook {
     public:
        Text &operator=(Text other) {
            AbstractBook::operator=(std::move(other));
            topic = std::move(other.topic);
        }
};
```

Since `operator=` is non-virtual, there is no mixed assignment!

`AbstractBook::operator=` not accessible to outsiders.

Now `*b1 = *b2` won't compile, preventing partial assignment.

Basically saying, before you assign something, understand what you're assigning and do it directly rather than through your superclass.

---
[I want a class with no objects << ](./problem_21.md) | [**Home**](../README.md) | [>> I want to know what kind of Book I have](./problem_23.md)
