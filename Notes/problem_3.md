[Separate compilation <<](./problem_2.md) | [**Home**](../README.md) | [>> Linear Collections and Memory Management](./problem_4.md) 

# Problem 3: Linear Collections and Modularity
## **2021-09-14**

### Linked lists and arrays

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

### C Preprocessor

Transforms the program before the compiler sees it

`#include _____` drops the contents of a file "right here"

Including old C headers: `#include <stdio.h>` -> `#include <cstdio>`

`#define VAR VALUE`

- Preprocessor variable
- All subsequent occurences of VAR are replaced with VALUE

Ex.
```C++
#define MAX 10
int x[MAX];
```
Translates to `int x[10];`

Ex 2.
_myprogram.cc_

```C++
int main() {
    int x[MAX];
    ...
} 
```

Instead of defining the constant inside the code, we can use a command line arg

- `g++14 -DMAX=10 myprogram.cc`

```C++
#if SECURITYLEVEL == 1
    short int
#elif SECURITYLEVEL == 2
    long long int
#endif
    publicKey;
```

Choose one of the options to present to the compiler. `#else` also exists

**Special Case:**
```C++
#if 0  // industrial-strength "comment out"
...    // /* ... */ doesn't nest
#endif // Does nest 
```

Fixing the include problem: `#include guards`

_node.h_
```C++
#ifndef NODE_H  // If NODE\_H is not defined"
#define NODE_H  // Value is the empty string
... (file contents)
#endif
```

First time _node.h_ is included, symbol NODE_H not defined, so file is included.

Subsequently, symbol is defined, so file is supressed.

**ALWAYS**

- Put `#include guards` in header files

**NEVER**

- Compile.h files
- Include .cc files

Now what if someone writes:

```C++
struct Node {
   int data;
   Node *left, *right; 
};

size_t size(Node *n); // Size of tree
```

You can't use both in the same program

**Solution:** namespaces

_list.h_

```C++
namespace List {
    struct Node {
        int data;
        Node *next;
    };

    size_t size(Node *n); 
}
```

_tree.h_

```C++
namespace Tree {
    struct Node {
        int data;
        Node *left, *right;
    };

    size_t size(Node *n); 
}
```

_list.cc_

```C++
#include "list.h"

size_t List::size(Node *n) {  // List::Node *n is not necessary b/c the function is in the namespace
    ...
}
```

_tree.cc_

```C++
#include "tree.h"

size_t Tree::size(Node *n) { 
    ...
}
```

**Namespaces are open**

Anyone can add items to any namespace

_some\_other\_file.h_

```C++
namespace List {
    struct some_other_struct {...};
}
```

---
[Separate compilation <<](./problem_2.md) | [**Home**](../README.md) | [>> Linear Collections and Memory Management](./problem_4.md) 
