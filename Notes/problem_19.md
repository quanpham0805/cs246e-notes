[Abstraction over containers << ](./problem_18.md) | [**Home**](../README.md) | [>> I'm leaking!](./problem_20.md)

# Problem 19 - Heterogenous Data
## **2021-10-21**

I want a mixture of types in my vector.

Can't do this with a template.

```C++
vector<template<typename T> T> v;
```
- Not allowed - templates are compile time entities, they don't exist at runtime

Ex. Fields of a struct

```C++
class MediaPlayer {
    template<typename T> T nowPlaying;   // Can't do this
};
```

Fallback to what's available to us in C:

**unions**:
```C
union Media {Song s; Movie m};
Media nowPlaying;
```
- Nice but you don't know what it might be exactly, you would need to store a member field, not so ideal

**void\***
```C
void *nowPlaying;
```
- Even worse, can point to anything

These are **not type-safe**.

Items in a heterogeneous collection will **usually** have something in common, ex. provide a common interface.

Can be viewed as different "kinds" of a more general "thing". So have a vector of "thing", or a field of type "thing".

We'll use the standard CS 246 example because it's good:

```C++
class Book {    // Superclass or Base class
        string title, author;
        int length;
    public:
        Book(string title, string author, int length):
            title{title},
            author{author},
            length{length} {}

        bool isHeavy() const { return length > 100; }
        string getTitle() const { return title; }
        // etc.
};
```
```
BOOK
+--------+
| Title  |
+--------+
| Author |
+--------+
| Length |
+--------+
```
- If you are not clear yet, we will be talking about **Inheritance**. 
## **2021-10-26**
Some books are special though
```C++
// Book would be super class / base class
// why public? later
class Text: public Book {   // Subclass or Derived class
        string topic;   // No need to mention title, etc. because it comes from book
    public:
        Text(string title, string author, int length, string topic): 
            // might be tempted to do title{title}..., but in MIL, you can only do that for **your** fields
            Book{title, author, length}, 
            topic{topic} {}

        bool isHeavy() const { return length > 200; }
        string getTopic() const { return topic; }
};
```
- This code wouldn't compile yet but we'll get back to it later
```C++
TEXT
+--------+
| Title  |
+--------+
| Author |
+--------+
| Length |
+--------+
| Topic  |
+--------+
```
```C++
class Comic: public Book {
        string hero;
    public:
        Comic(string title, string author, int length, string hero):
            Book{title, author, length},
            hero{hero} {}

        bool isHeavy() const { return length > 50; }
        string getHero() const { return hero; }
};
```
```C++
COMIC
+--------+
| Title  |
+--------+
| Author |
+--------+
| Length |
+--------+
| Hero   |
+--------+
```
Subclasses inherit all members (fields & methods) from their superclass.  

All three classes have `title`, `author`, and `length`, methods `getTitle`, `getAuthor`, `getLength`, `isHeavy`, ... except this doesn't work.

`length` is a private method in `Book`, `Text` cannot access it.

**2 options:**

1. _Use protected_
```C++
class Book {
        string title, author;
    protected:  // Accessible only to this class and its subclasses
        int length;
    public:
        ...
};
```

2. _Call public method_
```C++
bool Text::isHeavy() const { return getLength() > 500; }
```

Recommended option is 2.
- You have no control over what subclasses might do
- Protected weakens encapsulation (cannot enforce invariants on protected fields)

If you want subclasses to have priviledged access
- Keep fields private
- Provide protected `get_` and `set_` methods

## Updated object creation/destruction protocols

**Creation:**
1. Space is allocated
1. Superclass part is constructed
1. Fields constructed in declaration order
1. Constructor body runs

**Destruction:**
1. Destructor body runs
1. Fields are destructed in reverse declaration order
1. Superclass part destructed
1. Space deallocated

- Also recall that `new` and `delete` would be all step 1 to 4, while `operator new` would be step 1, then placement new `new (addr) obj` would be step 2-4, and samething for operator delete and invoking destructor.
- Roughly speaking, `new = operator new + placement new`.

Must revist everything we have learnt to see the effect of inheritance

## Type compatibility
`Text`s and `Comic`s are special kinds of `Book`s - should be usable in place of `Book`s
```C++
// this is valid
Book b = Comic{___, ___, 75, ___};

// method calls:
b.isHeavy();
``` 

This is a light `Book`, but a heavy `Comic`. What does this return? -> Returns `false`

If `b` is a `Comic`, why is it acting like a `Book`? -> Because it is a `Book` (declared as `Book`)!

Consequence of stack-allocated objects:

```C++
// Set aside enough space to hold a book
// Cannot hold it, not enough space for a comic
       +----+                +----+
       |    |                |    |
       +----+                +----+
Book b |    |  = Comic {...} |    |
       +----+                +----+
       |    |                |    |
       +----+      <---      +----+
                             |    |
                             +----+
```

Keeps only the `Book` part - `Comic` part is "chopped off" - **slicing**
- So it really is just a `Book` now
- Therefore it is `Book::isHeavy` that runs

Slicing happens even if superclass & subclass are the same size. This is a good thing because this behavior is consistent.

Similarily, if you want to collect your books:

```C++
vector<Book> library;
library.push_back(Comic {...}); 
```
only the `Book` part will be pushed - _not_ a heterogeneous collection.

Also note:
```C++
void f(Book book[]); // raw array
Comic comics[] = {...};
f(comics);  // Will compile but never do this!
```
Troubles:
- Array will be misaligned
- Will not act like an array `Books`
- Undefined behaviour!

Slicing does not happen through pointers

So if I do this instead:

```C++
Book *p = new Comic{___, ___, 75, ___};

p
+---+     +---+   
|   | --> |   |  
+---+     +---+
          |   |
          +---+
          |   |
          +---+
          |   |
          +---+
```

But `p->isHeavy();` is still false!

**Rules:** the choice of which `isHeavy` is based on the type of the pointer (static type), not the object (dynamic type).

Why? Because it's cheaper.

**C++ Design Principle (again):** If you don't use it, you shouldn't have to pay for it.

That is if you want something more expensive, you have to ask for it. To make `*p` act like a `Comic` when it is a `Comic`:

```C++
class Book {
        // ...
    public:
        // ...
        virtual bool isHeavy() const { ... }

};

class Comic {
        // ...
    public:
        // ...
        bool isHeavy() const override { ... }
};

// Assume isHeavy is virtual
p->isHeavy();   // true!
```

`override` is a contextual keyword, is only a keyword in that specific location.
- It tells C++ compiler: "Hey, this method is supposed to override something". You need to make sure everything matches too (signature, constness,...), otherwise this would not be override, it would be an overload, which might caused unexpected behaviours.
- Tells the compiler: "Make sure there is a method in the super class with exactly the same signature, that is virtual, so that I can be sure that this can override properly.
- Essentially telling the compiler to check if the override is correct.

Now we can have a truly heterogeneous collection.

```C++
vector<Book*> library;
library.push_back(new Book{...});
library.push_back(new Comic{...});

// Even better version
vector<unique_ptr<Book>> library; 

int howManyHeavy(const vector<Book*> &v) {
    int count = 0;
    for (auto &b: v) {
        if (b->isHeavy()) ++count;
    }

    return count;
}

for (auto &b: library) delete b;    // Not necessary if library is a vector of unique_ptrs
```

Correct version of `isHeavy` is always chosen, even though we don't know what's in the vector, and the items are probably not the same type.

This is called **polymorphism**.

How do virtual methods "work" and why are they more expensive? (though not _significantly_ more expensive)
- Implementation dependent, but the following is most common:

**Vtables** (only contain virtual methods)
```C++
// These are vtables
(1)
+---------+
| "Book"  |
+---------+
| isHeavy | -> Book::isHeavy // then call the body code here
+---------+

(2)
+---------+
| "Comic" |
+---------+
| isHeavy | -> Comic::isHeavy // then call the body code here
+---------+
```
So when we create two `Book`s `b1`, `b2`, and a `Comic b2`:
```C++
// actual object would look like this
Book b1;

+--------+
| vptr   | -------> (1)
+--------+
| Title  |
+--------+
| Author |
+--------+
| Length |
+--------+

Book b2;

+--------+
| vptr   | -------> (1)
+--------+
| Title  |
+--------+
| Author |
+--------+
| Length |
+--------+

Comic b2;

+--------+
| vptr   | -------> (2)
+--------+
| Title  |
+--------+
| Author |
+--------+
| Length |
+--------+
```
Non-virtual methods are just ordinary function calls.

If there is at least one virtual method in the class:
- Compiler creates a table of function pointers:
    - One per class
    - The vtable
- Each object contains a pointer to its class' vtable 
    - the `vptr`
- Calling the virtual method => follow the `vptr` to the vtable, follow the function pointer to the correct function

- `vptr` is often the "first" field
    - So that a subclass object still looks like a superclass object, and it can ignore the rest of the fields.
    - So the program knows where the `vptr` is
    - If there are no virtual methods, `vptr` does not exist
    - <details open> <summary>Rant:</summary>
  
      - Number one:
        - To make a subclass "looks like" a super class, we would chop off the latter fields, i.e, if we ignore the latter fields, it's literall just the super class.
        - That would be hard to do if the `vptr` is the last, because then we would not be able to "ignore" the last field anymore. 
        - If `vptr` is in the middle then you would have a hole in your object
      - Number two:
        - We need the `vptr` to know which object we're doing, and we currently don't know what kind of object we have
        - That's part of the reason why we need a `vptr` to tell us that. Now, if `vptr` is not always at the same place, then in order to find the `vptr` we would need to know which object we currently have?????? Chicken-and-egg problem.
      </details>

So virtual methods incur a cost in
- time (Extra pointer derefs)
- space (Each object gets a `vptr`)

Note that, if a subclass does not **override** a virtual method of the superclass, then the pointer in the `vtable` would just point to the superclass's implementation
- In the example above, if `Comic` does not override `Book`, then the `vtable` would just point to `Book::isHeavy`

---
[Abstraction over containers << ](./problem_18.md) | [**Home**](../README.md) | [>> I'm leaking!](./problem_20.md)
