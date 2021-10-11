[I want a constant vector <<](./problem_7.md) | [**Home**](../README.md) | [>> Efficient Iteration](./problem_9.md)

# Problem 8: Tampering
## **2021-09-28**

```C++
vector v;
v.cap = 100;    // Sets cap without allocating memory
v.push_back(1);
v.push_back(10);
...  // Undefined behaviour - will likely crash
```
**Interfering with ADTs (Abstract Data Types)**

1. Forgery 
    - Creating an object without a constructor function
        - Not possible once we wrote constructors (resolved)
1. Tampering
    - Accessing the internals without using provided interface functions

What's the big deal? **Invariants**

- Statement that will always be true about an abstraction

ADT's provide and rely on invariants

- **Stacks**
    - Provide the invariant that the last item pushed is the first item popped
- **Vectors**
    - Rely on the invariant that elements `O`, ..., `cap-1` denote valid locations

Cannot gaurantee invariants if the user can interfere, makes the program hard to reason behind

**Fix:** _encapsulation_, seal objects into "black boxes"

```C++
struct vector {
    private:    // Fields are only accessible within the vector class
        size_t n, cap;      
        int *theVector;
    public:     // Visible to all
        vector();
        size_t size() const;
        void push_back(int n);
        // ...
};
```

If no access specifer is given: default is public for `struct`

In a previous lecture:

#### vector.cc
```C++
#include "vector.h"
namespace {
    void increaseCap(vector &v) {...}   // Doesn't work anymore! Doesn't have access to v's internals
}
```
Try again:

#### vector.h
```C++
struct vector {
    private:
        size_t ...
        ...
    public:
        vector();
        ...

    private:
        void increaseCap(); // Now a private method
};
```
NOW

#### vector.cc
```C++
namespace CS246E {
    vector::vector() {...}  
    // etc. as before
    void vector::increaseCap() {...}  // Don't need anonymous namespaces anymore!
};
```

- **Structs** (`struct`)
    - Default public access
    - Would be better if we had default private
- **Classes** (`class`)
    - Exactly like struct, except private default access

_vector.cc_
```C++
class vector {
        size_t n, cap;
        int *theVector;
    public:
        vector();
        ...

    private:
        void increaseCap(); // Now a private method
};
```
- We prefer using `class` to `struct`: it sounds cooler, and it has only 5 chars

A similar problem with linked lists:

```C++
Node n {3, nullptr};    // Stack allocated
Node m {4, &n}; // m's dtor will try to delete &n (undefined, pointer pointed to a memory allocated on stack)
```

There was an **invariant** that - `next` is `nullptr` or was allocated by `new`, we don't want the user to use it wrong.

How can we enforce this?
- Encapsulate `Node` inside a "wrapper" class.

```C++
class list {
    struct Node {    // Private nested class - not available outside
        int data;
        Node *next; // ... methods
    };

    Node *theList;
    
    public:
        list(): theList{nullptr} { }
        // we can deconstruct this iteratively: run a loop inside the list class, this is actually possible now
        ~list(): { delete theList; }
        size_t size() const;

        void push_front(int n) {
            theList = new Node{n, theList};
        }

        void pop_font() {
            if (theList) {
                Node *tmp = theList;
                theList = theList->next;
                tmp->next = nullptr;
                delete tmp;
            }
        }

        const int &operator[](size_t i) const {
            Node *cur = theList;
            for (size_t j = 0; j < i && cur; ++j, cur=cur->next);
            return curr->data;
        }

        int &operator[](size_t i) {
            Node *cur = theList;
            for (size_t j = 0; j < i && cur; ++j, cur=cur->next);
            return curr->data;
        }   
};
```
Client cannot manipulate the list directly
- No access to next pointers
- Invariant is maintained

---
[I want a constant vector <<](./problem_7.md) | [**Home**](../README.md) | [>> Efficient Iteration](./problem_9.md)