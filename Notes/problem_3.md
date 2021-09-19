[Separate compilation <<](./problem_2.md) | [**Home**](../README.md) | [>> Linear Collections and Memory Management](./problem_4.md) 

# Problem 3: Linear Collections and Modularity
## **2021-09-14**

### **Linked lists and arrays**

**Linked List**

#### node.h

```C++
#include <cstddef>  // Provides size_t

struct Node {
    int data;
    Node *next;
};

size_t size(Node *n); // size_t is a type that can guarantee to hold any amount of memory without limit.
```

#### node.cc

```C++
#include "node.h"

size_t size(Node *n) {
    size_t count = 0;
    for (Node *cur = n; cur ; cur = cur->next) ++ count;
    return count;
}
```

#### main.cc

**Note:** Do NOT use `malloc`, `free`, and `NULL` in C++, instead use `new`, `delete`, and `nullptr`
- `malloc` and `free` is too "low-level", which means you have to calculate the memory needed by yourself and that leads to calculations error. `new` and `delete` are built into the type system, so it knows how big the memory we need.
- In C, `NULL` is not a thing, just a constant defined as 0 in standard libraries. In C++, `nullptr` is an actual type that represents null.

```C++
#include "node.h"

int main() {
    Node *n = new Node;
    n->data = 3;
    n->next = nullptr;

    Node *n2 = new Node {3, nullptr};
    Node *n3 = new Node {4, new Node {5, nullptr}};

    delete n;
    delete n2; // Memory leak!! Delete n2->next first

    while (n3) {
        Node *tmp = n3;
        n3 = n3->next;
        delete tmp;
    }
}
```

## **2021-09-16**

What would happen if we do

```C++
#include "node.h"
#include "node.h"
```

Won't compile, struct defined twice

How do we prevent this?

### **C Preprocessor**

Transforms the program before the compiler sees it

`#include _____` drops the contents of a file "right here"

Including old C headers: `#include <stdio.h>` -> `#include <cstdio>`

`#define VAR VALUE`

- Preprocessor variable
- All subsequent occurences of VAR are replaced with VALUE

**Ex 1.**
```C++
#define MAX 10
int x[MAX];
```
Translates to `int x[10];`

- This is considered as a "cheat" constant because C was the 70s tech, having a constant variable might be costly, and so instead of having the variable present at runtime, just have it replaced at compile time.
- Nowadays these are so cheap people just don't care about it. And C++ has its own built in way of expressing compile time constant, that the language actually understand, instead of that hacky way. Therefore, C define constant is obsolete.

**Ex 2.**

```C++
#define ever(;;)
// ...
for ever { /* ... */}
```

- Just a simple text substitution.

Instead of defining the constant inside the code, we can use a command line arg

```C++
    int main() {
        int x[MAX]; // MAX not defined in source code
    }
```

- Rather than having to touch the source file or trusting anybody else to touch my source file, we can do:
- `g++14 -DMAX=10 myprogram.cc`
- What this does is it makes a `#define` on the command line.

**Ex 3.**

```C++
#if SECURITYLEVEL == 1
    short int
#elif SECURITYLEVEL == 2
    long long int
#endif
    publicKey;
```
- This is called **conditional compilation**. If the condition is true, the compiler gets to see the code. Otherwise, it does not even go through at all i.e, it transformes the code before the it reaches the compiler.
- Choose one of the options to present to the compiler. `#else` also exists

**Special Case:**
```C++
#if 0  // industrial-strength "comment out"
...    // /* ... */ doesn't nest
#endif // Does nest 
```

Fixing the include problem: `#include guards`

#### node.h
```C++
#ifndef NODE_H  // If NODE\_H is not defined"
#define NODE_H  // Value is the empty string
... (file contents)
#endif
```

Fixing the double include problem: "include guard"
- First time _node.h_ is included, symbol `NODE_H` not defined, so file is included.
- Subsequently, symbol is defined, so file is supressed.

**ALWAYS**
- Put `#include guards` in header files (`.h`)

**NEVER**
- Compile `.h` files
- Include `.cc` files

Now what if someone writes:

```C++
struct Node {
   int data;
   Node *left, *right; 
};
size_t size(Node *n); // size of tree
```

You can't use both in the same program

**Solution:** namespaces

#### list.h

```C++
namespace List {
    struct Node {
        int data;
        Node *next;
    };
    size_t size(Node *n); 
}
```

#### tree.h

```C++
namespace Tree {
    struct Node {
        int data;
        Node *left, *right;
    };
    size_t size(Node *n); 
}
```

#### list.cc

```C++
#include "list.h"

// one style
size_t List::size(Node *n) {  // List::Node *n is not necessary b/c the function is in the namespace
    // ...
}
```

#### tree.cc

```C++
#include "tree.h"

// another style
namespace Tree {
    size_t size(Node *n) { 
        // ...
    }
}
```

Now we can do in our main file:

#### main.cc

```C++
#include "list.h"
#include "tree.h"

int main()
{
    List::Node *ln = new List::Node {1, nullptr};
    Tree::Node *tn = new Tree::Node {2, nullptr, nullptr};
    // ...
    delete ln; delete tn;
}
```

**Namespaces are open**
- It means, anyone can add items to any namespace

#### some\_other\_file.h

```C++
namespace List {
    int some_other_fn();
    struct some_other_struct {...};
}
```

Exception: adding members to namespace `std` is not permitted.

---
[Separate compilation <<](./problem_2.md) | [**Home**](../README.md) | [>> Linear Collections and Memory Management](./problem_4.md) 
