[The copier is broken (again) << ](./problem_22.md) | [**Home**](../README.md) | [>> A big unit on Object Oriented Design](./object_oriented_design.md)

# Problem 23: I want to know what kind of `Book` I have
## **2021-10-28**

**For simplicity:** Assume the old book hiearchy

- `Book`
    - `Text`
    - `Comic`

- **C-style Casting**
    - `(type) expr`
    - Forces `expr` to be treated as type `type`
    - Ex. ```C++
            int *p;
            int q = (int) p;
          ```
    - Very easy to be a source of error
    - Very difficult to search for

The `C++` casting operators - _4 operators_
- **`static_cast`:** for conversion <u>with</u> well-defined semantics
    - Ex. 
        ```C++
        void f(int a); 
        void f(double d);
        int x;
        f(static_cast<double>(x));
        ```
    - Ex. 
        ```C++
        // superclass pointer to a subclass pointer
        Book *b = new Text { ... };
        Text *t = static_cast<Text *>(b);
        ```
    - **Important**: You only do this if you're **100%** sure that pointer is pointing to a `Text`, otherwise it is not safe, undefined behaviour.
    - Telling the compiler "trust me, I know what I'm doing"
  
- **`reinterpret_cast`:** for casts <u>without</u> well-defined semantics
    - Unsafe, implementation-dependent, "weird" casts (therefore unportable)
    - Ex. 
        ```C++
        Book *b = new Book { ... };
        int *p = reinterpret_cast<int*>(b);
        ```
    - There is no expected behaviour from this.
    - Telling the compiler "I trust YOU, I think I know what you are doing"
- **`const_cast`:** for adding/removing const
    - The only C++ cast that can "cast away `const`"
    - If item is still in read-only memory, it's still read only
    - Ex. 
        ```C++
        void g(Book &b);    // Assuming we know g won't change b 
        void f(const Book &b) {
            g(const_cast<Book &>(b));
        }
        ```
    - <details open> <summary>Rant</summary>
        
        - When writing code that is having compile errors, the program is usually divided into 2 parts, correctly written `const`, and incorrectly written `const`.
        - Imagine we lay them flat into like an array, there would be a boundary between them. That boundary would be the indicator of whether we have fixed everything or not
        - Now, whenever we fix one thing, that boundary move a little bit, so now we have more correctly written code and incorrectly written code
        - But if the program is huge, this might take forever
        - People usually use `const_cast` to break this boundary, assuming they know what they are doing.
      </details>
- **`dynamic_cast`:**  `Book *pb = ...`
    - What if we _don't_ know whether `pb` points to a `Text`?
    - Then `static_cast` is not safe
        ```C++
        Text *pt = dynamic_cast<Text*>(pb);
        ```
    - If `*pb` is a `Text` or a subclass of `Text`, cast succeeds. `pt` points at the object, else `pt = nullptr` (it fails)
        ```C++
        if (pt) { 
            // ... 
            pt->getTopic() 
            // ... 
        }
        else { // Not a Text
            // ...
        }
        ```
    - Ex.
        ```C++
        void whatIsIt(Book *pb) {    
            if (dynamic_cast<Text*>(pb)) cout << "Text";
            else if (dynamic_cast<Comic*>(pb)) cout << "Comic";
            else cout << "Book";
        }
        ```
        - **Not good style**, even though you can write it like this, what happens when you (or someone else) create a new `Book` type, and they did not know about this method / you forget?
        - <details open>
            <summary> Rant </summary>

            - We can go as far as saying what we did there is the exact opposite of object oriented programming
            - OOP is about the idea of having this superclass abstraction, you can have a number of things inherit from it and you don't need to know what they are
            - All your client code works for the super class, and you get specialized behaviour through virtual methods
            - THIS IS BRUTALLLL!!!
          </details>
    - Dynamic reference casting
        ```C++
        Book *pb = ____;
        Text &t = dynamic_cast<Text &>(*pb);
        if (*pb is a Text) // ok
        else // rase std::bad_cast
        ```
    - **Note:** dynamic casting works by accessing an objects **Run-Time Type Information (RTTI)**, this is stored in the vtable for the class 
        - This means we can only `dynamic_cast` on classes/objects that have at least one virtual method
    - Dynamic reference casting offers a possible solution to take polymorphic assignment problem:
        ```C++
        Text &Text::operator=(const Book& other) { // Virtual
            Text &textother = dynamic_cast<Text &>(other);  // Throws if other is not a Text
            if (this == &textOther) return *this;
            Book::operator=(textother);
            topic = textother.topic;
            return this; 
        }
        ```
---
[The copier is broken (again) << ](./problem_22.md) | [**Home**](../README.md) | [>> A big unit on Object Oriented Design](./object_oriented_design.md)
