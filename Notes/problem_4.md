[Linear Collections and Modularity <<](./problem_3.md) | [**Home**](../README.md) | [>> The Copier is broken!](./problem_5.md)

# Problem 4: Linear Collections and Memory Management
## **2021-09-16**
**Readings:** 7.7.1, 14, 16.2 

**Arrays**
`int a[10];`
- On the stack, fixed size
```
┏━━━━━━━━━━━━━┓
┣━━━━━━━━━━━━━┫
┣━━━━━━━━━━━━━┫
┣━━━━━━━━━━━━━┫
┣━━━━━━━━━━━━━┫
┣━━━━━━━━━━━━━┫
┗━━━━━━━━━━━━━┛
---------------
```

On the heap:
- `int *p = new int[10];`
```
┏━━━━━━━━━━━━━┓
┣━━━━━━━━━━━━━┫<┅┅┅┅┅┅┅┅┅┅┅┅┓
┣━━━━━━━━━━━━━┫             ┇
┣━━━━━━━━━━━━━┫             ┇
┣━━━━━━━━━━━━━┫             ┇
┣━━━━━━━━━━━━━┫             ┇
┗━━━━━━━━━━━━━┛ // stack    ┇
```                         ┇
┏━━━━━━━━━━━━━┓ // heap     ┇
┣━━━━━━━━━━━━━┫             ┇
┣━━━━━━━━━━━━━┫             ┇
┗━━━━━━━━━━━━━┛ p ┅┅┅┅┅┅┅┅┅┅┛
---------------
```

---------------
- To delete:  `delete[] p;`
- Use `new` with `delete`, and `new [...]` with `delete[]`
- Mismatching these is undefined behaviour

**Why do we have a separate form of delete?**
> Philosophy in C++: If you are not gonna use it, you shouldn't have to pay for it.
- If you have an array of items, and u need to deallocate that array, you need to know how many items there were / how big the memory we need to get rid of. Therefore, when you declare that array, you need to store how much memory you used.
- But in special case of allocating 1 object, not an array, then why should I pay for that extra cost of saying "hey this is an object of size 1". So instead, C++ said "I know how big one object is" and so the compiler has the option if you are allocating one object, to not store that extra size information becuase its known. Therefore, the ordinary delete would not looking for sizes because it knows it was deleting one thing. Hence, having a separate form of delete for single object allows for potential optimization where you don't have to worry about checking sizes.

**Problem:** what if our array isn't big enough (when deleting)?

Note: no `realloc` for `new`/`delete` 

Use abstraction to solve the problem

- Another philosophy here is, C++ is hard, and we are going to do all the hard work so that our users (in this case, not the application users but rather the developer that uses our code) have a pleasant time coding in C++.

#### vector.h

```C++
#ifndef VECTOR_H
#define VECTOR_H

namespace CS246E {
    struct vector {
        int *theVector;
        size_t size, cap;
    };

    vector make_vector();
    size_t size(const vector &v);
    int &itemAt(const vector &v, size_t i);
    void push_back(const vector &v, int x);
    void pop_back(const vector&v);
    void dispose(vector &v);
}
#endif
```

#### vector.cc

```C++
#include "vector.h"

namespace {  // Anonymous namespace makes the function only visible to file (same as static in C)
    void increaseCap(CS246E::vector &v) {
        if (v.size == v.cap) {
            int *newVec = new int[2 * v.cap];

            for (size_t i = 0; i < v.cap; ++i) {
                newVec[i] = v.theVector[i]
            }

            delete[] v.theVector;
            v.theVector = newVec;
            v.cap *= 2;
        }
    }
}

CS246E::vector CS246E::make_vector() {
    vector v {new int[1], 0, 1};
    return v;
}

size_t CS246E::size(const vector &v) {
    return v.size;
}

int &CS246E::itemAt(const vector &v, size_t i) {
    return v.theVector[i];
}

void CS246E::push_back(vector &v, int n) {
    increaseCap(v);
    v.theVector[v.size ++] = n;
}

void CS246E::pop_back(vector &v) {
    if (v.size > 0) {
        -- v.size;
    }
}

void CS246E::dispose(vector &v) {
    delete[] v.theVector;
}
```

#### main.cc

```C++
#include "vector.h"

using CS246E::vector;  // only allows you to use vector without CS246E::

int main() {
    vector v = CS246E::make_vector();
    push_back(v, 1);
    push_back(v, 10);
    push_back(v, 100);  // Can add as many items as we want without worrying about allocation
    itemAt(v, 0) = 2;
    dispose(v);
}
```

**Question:** why don't we have to say `CS246E::push_back`, `CS246E::itemAt`, `CS246E::dispose?`

**Answer:** Argument-Dependent Lookup (ADL) - aka Koenig lookup

- If the type of a function f's argument belongs to a `namespace n`, then C++ will search the `namespace n`, as well as the current scope, for a function matching f

This is the reason why we can say
```C++
std::cout << x
// rather than
std::operator<< (std::cout, x)
```

- **Problems** 
    - What if we forget to call `make_vector`? (uninitialized object)
    - What if we forget to call `dispose`? (memory leak)
- How can we make this more robust (easier to do correctly and harder to make it incorrect)?

### **Introduction to Classes**
First concept in OOP - functions inside structs

```C++
struct Student {
    int assns, mt, final;
    
    float grade() {
        return assns * 0.4 + mt * 0.2 + final * 0.4;
    }
};
```

- Structures that can contain functions - called **classes**
- Functions inside of structs - called **methods**
- Instances of a class - called **objects**

```C++
Student bob {90, 70, 80};
cout << bob.grade();
```

`bob` is an object, `.grade()` is a method.

What do `assns`, `mt`, `final`, mean with `grade() {...}`?

- Fields of the _current_ object, the receiver of the method call (ie. `bob`)

Formally, methods differ from functions in that methods take an implicit parameter called `this`, that is  a pointer to the receiver object.
- `bob.grade()` gets `this == &bob`

Could have written (equivalent):

```C++
struct Student {
    // ...
    float grade() {
        return this->assns * 0.4 + this->mt * 0.2 + this->final * 0.4;
    }
};
```
or 
```C++
Student::float grade() {
    return this->assns * 0.4 + this->mt * 0.2 + this->final * 0.4;
}
```

## 2021/09/21

### **Initializing objects**
```C
Student bob {90, 70, 80}
```

- C style struct initialization
- Field by field
- ok, but limited

Better initializtion method: a constructor

```C++
struct Student {
    int assns, mt, final;

    Student(int assns, int mt, int final) {
        this->assns = assns;
        this->mt = mt;
        this->final = final;
    }
};

// Now we can call
Student bob {90, 70, 80};
// Now calls the constructor with args 90, 70, 80
```

**Note:** once the constructor is defined, the C style field-by-field initialization is no longer available

**Equiv:** 
```C++ 
Student bob = Student{90, 70, 80};
```

Unified initialization using braces:
```C++
int x{5}; // this is possible and is similar to int x = 5;
```

**Heap:**
```C++
Student *p = new Student{90, 80, 70};
delete p;
```
**Advantages of constructors:**
- Default parameters
- Overloading, as long as signatures are different
- Sanity checks, making sure the initialization makes sense and if they don't, try to correct them somehow.

ex.

```C++
struct Student {
    student(int assns = 0, int mt = 0, int final = 0) {
        this->assns = assns;
        ...
    }
};

Student laura {70};  // 70, 0 , 0
Student newKid; // 0, 0, 0 
```

**Note:** Every class comes with a **default constructor** (zero argument constructor)
- The default constructor will try to initialize any fields that are objects by calling their own constructor.

ex.
```C++
Node n;  // Default constructor, Node has an int and a pointer in which both are not objects, so default constructor does nothing in this case.
```
This goes away if you write any constructor

Ex.
```C++
struct Node {
    int data;
    Node* next;

    Node (int data, Node* next = nullptr) {
        // ...
    }
};

Node n {3};  // GOOD
Node n;  // BAD - no default constructor, won't compile
```
The ctor now can accept either 1 or 2 arguments, but not 0 because it's not the default.

The first two are equivalent:
```C++
Node n{};
Node n;
Node n(); // but watch out for this, this is prototype for function
```

### **Object creation protocol**

When an object is created, there are 4 steps:

1. Space is allocated, we need space to hold it.
2. (later)
3. Fields are constructeed in declaration order (field constructors are called for fields that are objects)
4. Constructor body runs

Field initialization should happen in step 3, but constructor body happens in step 4

Consequence: object fields are intialized twice (step 4 is considered assignment step):

```C++
#include <string>

struct Student {
    int assns, mt, final;
    std::string name;

    Student (std::string name, int assns, int mt, int final) {
        this->name = name;  // etc.
    }
};

Student mike {"Mike", 90, 70, 60};
// name default-initialized in step 3 ("")
// then reassigned in step 4 ("Mike")
```

**To fix:** the Member Initialization List (MIL)

```C++
struct Student {
    Student (string name, int assns, int mt, int final): 
        name{name}, assns{assns}, mt{mt}, final{final}  // Step 3
    {  // Step 4

    }
}
```
where the inside is the params, and the outside is the class fields.

Changing the order in the MIL will not change the order in which the fields are initialized, they will be in declaration order.

MIL _must_ be used for fields that are 

- Constants
- References
- Objects

In general, it should be used as much as possible.

Careful: single argument constructors
```C++
struct Node {
    Node(int data, Node* next = nullptr): data{data}, next{next} {}
}
```

- Single argument constructors create implicit conversions

```C++
Node n {4};     // OK
Node n = 4;     // OK - implicity converted from int to Node

void f(Node n);
f(4); // Will compile - maybe trouble
```

However you can add an `explicit` keyword to disable the implicit conversion

```C++
explicit struct Node {
    Node(int data, Node* next = nullptr): ... {}
}

Node n {4};  // OK
Node n = 4;  // BAD, won't compile

f(4) // BAD, won't compile
f(Node {4}) // OK
```

### **Object Destructor**

A method called the **destructor** (dtor) runs automatically

- Built-in dtor: calls dtor on all fields that are objects
- Object destruction protocol:
    1. Dtor body runs
    2. Fields destructed (dtors called on fields that are objs) in reverse declaration order
    3. (Later)
    4. Space deallocated

```C++
struct Node {
    int data;
    Node* next;
};
```

In this case the built-in destructor does nothing because neither field is an object

If we have:
```C++
Node* n = new Node {3, new Node {4, new Node {5, nullptr}}}
delete n;  // only deletes the first node (memory leak!)
```

We can fix this by writing our own destructor:

```C++
struct Node {
    ...

    ~Node() {
        delete next;
    }
};

delete n;  // Now frees the whole list
```
- This is recursion, it does consume stack space, base case is when `next = nullptr`, in which `delete nullptr` is safe.
- If you run out of stack space, you are probably using wrong data structure.
- We can avoid this with encapsulation.

Also:
```C++
{
    Node n {1, new Node {2, new Node {3, nullptr}}};
}  // Scope of n ends; whole list is freed
```

Objects:

- Proper constructions and destructions of objects
- A constructor always runs when they are created
- A destructor always runs when they are destroyed

#### vector.h

```C++
#ifndef VECTOR_H
#define VECTOR_H

namespace CS246E {
    struct vector {
        size_t n, cap;
        int *theVector;

        vector();
        size_t size();
        int &itemAt(size_t i);
        void push_back();
        void pop_back();
        ~vector();
    };
}
#endif
```

#### vector.cc

```C++
#include "vector.h"

namespace {
    void increaseCap(vector &v) {
        ...
    }
}

const size_t startSize = 1;

CS246E::vector::vector(): 
    n{0}, cap{startSize}, theVector{new int[cap]} {
}

size_t CS246E::vector::size() {
    return n;
}

// Etc.

CS246E::vector::~vector() {
    delete[] theVector;
}
```

#### main.cc

```C++
int main() {
    vector v;   // Constructor is already called - no make_vector
    v.push_back(1);
    v.push_back(10);
    v.push_back(100);
    v.itemAt(0) = 2; 
}   // No dispose - destructor cleans v up
```

---
[Linear Collections and Modularity <<](./problem_3.md) | [**Home**](../README.md) | [>> The Copier is broken!](./problem_5.md)
