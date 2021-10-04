[Linear Collections and Memory Management << ](./problem_4.md) | [**Home**](../README.md) | [>> Moves](./problem_6.md) 

# Problem 5: The Copier is broken!
## **2021-09-21**

```C++
Vector v;
v.pushback(100);
// ...
Vector w = v;  // Allowed - constructs w as a copy of v
w.itemAt(0);  // 100
v.itemAt(0); = 200;
w.itemAt(0);  // 200 - **shallow copy**, v and w share data 
```
This leads to more problems:
- When `v` and `w` run out of scope, compiler wants to pop off `w` and `v` off the stack.
- `w` is created after so the destructor will run for `w` first. It got rid of `theVector`, then `v`'s dtor runs.
- It attempts to get rid of the array that is already gone (double free), crashes.


For `Vector w = v;`
- Constructs `w` as a copy of `v`
- Invokes the **copy constructor**

```C++
struct Vector {
    Vector(const Vector &other) {...}  // Copy constructor, arg is an object of same class type
    // Compiler supplied copy-ctor, copies all fields (shallow copy)
};
```

If you want a deep copy, write your own copy constructor

```C++
struct Node {  // Vector: exercise (easy), we doing for Node
    int data;
    Node *next;
    // ...
    Node (const Node &other): 
        data{other.data}, 
        next{other.next ? new Node{*(other.next)} : nullptr} {} 
        // note: this is recursion because {} is assignment
};
```

## **2021-09-23**
However, we are not done with copy. Say, we want to assign rather than creating a new object:
```C++
Vector v;
Vector w;

w = v;  // Copy, but not a construction
        // Copy assignment operator
        // Compiler supplied: copies each field (shallow), leaks w's old data
```

### **Deep copy assignment**

```C++
struct Node {
    Node &operator=(const Node &other) {
        data = other.data;
        delete next; // avoid leaks
        next = other.next ? new Node{*(other.next)} : nullptr;
        return *this; // `this` is a pointer to an object, `*this` to deref
    }
};
```

**WRONG - dangerous**

Consider:
```C++
Node n {...};
n = n;
```
- The `delete` deleted itself, and while assigning to itself is not common, assigning 2 aliases of the same object is common.
- Must always ensure the operator = works in the case of self assignment

Another try
```C++
Node &Node::operator=(const Node &other) {
    if (this == &other) return *this;
    data = other.data;
    delete next; // avoid leaks
    next = other.next ? new Node{*(other.next)} : nullptr;
    return *this;
}
```
- This still is problematic as it only checks if the 2 things are exactly equal, but this `other` can be a sublist of `this`, but that's a discussion for encapsulation

**Alternative: copy-and-swap idiom**

```C++
#include <utility>

struct Node {
    ...
    void swap(Node &other) {
        using std::swap;
        // the next two lines, other is a completely new object and hence we get a deep copy
        swap(data, other.data);
        swap(next, other.next);
    }

    Node &operator=(const Node &other) {
        Node tmp = other; // relies on a working copy constructor
        swap(tmp); // then in this function, old me is swapped with tmp, which is new thing
        return *this; // and finally, this invokes destructor on old me, cleaning up.
    }
};
```
---
[Linear Collections and Memory Management << ](./problem_4.md) | [**Home**](../README.md) | [>> Moves](./problem_6.md) 