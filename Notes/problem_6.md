[The Copier is broken! <<](./problem_5.md) | [**Home**](../README.md) | [>> I want a constant vector](./problem_7.md)

# Problem 6: Moves
## **2021-09-23**

Consider:

```C++
Node plusOne(Node n) { // pass-by-value = copy => copy ctor (half truth)
    for (Node *p = &n; p; p = p->next) {
        ++p->data;
    }
    return n;
}

Node n {1, new Node {2, nullptr}};
Node m = plusOne(n);
```

In this case, "other" is a reference to the *temporary object* created to hold the result of plusOne.

- "Other" is a reference to this temporary
- Copy constructor deep-copies the data from this temporary

**But** the temporary is just going to be thrown out anyway, as soon as the statement `Node m = plusOne(n)` is done

It's wasteful to deep copy the temp, why not steal the data instead? - saves the cost of a copy
We need to be able to tell whether "other" is a reference to a temporary object, or a standalone object

### **Rvalue references** 
- `Node &&` is a reference to a temporary object (rvalue) of type `Node`. We need a version of the constructor that takes a `Node &&`
- In other words, we will need to be able to tell whether "other" is a reference to a temporary object (rvalue), or a standalone object (lvalue).
- We will do this by overloading the operator based on lvalue or rvalue status.

**Move Constructors** - its job is to steal other's data when constructing a new object

```C++
struct Node {
    // ...
    Node(Node &&other): data{other.data}, next{other.next} {
        other.next = nullptr; // making sure other does not have anything left
    }
};
```
Similarly:
```C++
Node m;
m = plusOne(n);  // assignment from temporary
```

Why can't we just swap memory instead of copying field-by-field?
- You can't swap because you don't have an object yet (remember this is currently constructing the object). 
- If you swap, you will be swapping garbage to `other`.
- Even when you are taking data from other object, you need to make sure it is still in a valid state because it's still an object, its ctor still runs, and if it's invalid it will crash.

**Move assignment operator**
```C++
struct Node {
    ...
    Node &operator=(Node &&other) { 
        using std::swap;            // steal other's data
        swap(data, other.data);     // you are gonna be destroyed anyway
        swap(next, other.next);     // can you take the garbage for me
        return *this;               // Easy: swap without the copy
    }
};
```
Can combine copy/move assignment:
```C++
struct Node {
    ...
    Node &operator=(Node other) { 
        swap(other); // copies if `other` is a lvalue (so it will be destroyed when out of scope)
        return *this;
    }
};
```
- Unified assignment operator (`&=operator(Node other)`)
- Pass by value:
    - Invokes copy constructor if an argument is a lvalue
    - Invokes move constructor (if there is one) if an argument is a rvalue
- We still need to implement our move ctor

**Note:** copy/swap can be expensive, hand-coded operator may do less copying

But now consider:

```C++
struct Student {
    std::string name; // string is a class
    Student(const std::string &name): name{name} {  // copies name into field (uses copy ctor)
        ...
    }
};
// ...
Student Bob{"bob"}; // we are deep copying from a temporary memory (rvalue, we want to steal)
```

What if `name` points to an rvalue?

```C++
struct Student {
    std::string name;

    Student (std::string name): name{name} {  
        // {name} may come from an rvalue, but it can be an lvalue
        // in other words, name may refer to an rvalue but name itself is an lvalue
        // went from guaranteed 1 copy to maybe 1 or 2, bad
    }
};
```
- Will copy if `name` is an lvalue, moves if `name` is an rvalue.
- The problem is that, `name{name}` will always be making a copy, because `name` pass-by-value is based on `{name}`, and not what `{name}` actually is. What this means is, even though `{name}` can be a rvalue, the variable itself is an lvalue so it will always make a copy (thats why we have 1 or 2) and if we can remove that copy, we would get what we want (0 copy if rvalue and 1 copy if lvalue).
```C++
struct Student {
    std::string name;
    Student(std::string name): name{std::move(name)} {

    }
}
```
`name{std::move(name)}` forces lvalue (`name`) to be treated as an rvalue, now strings move constructor

```C++
struct Student {
    ...
    Student(Student &&other): //move constructor
        name{std::move(other.name)} { }

    Student &operator=(Student other) { // unified assignment
        name = std::move(other.name); // invokes string's move assignment
        return *this;
    }
}
```
If you don't define move operations, copy operations will be used

If you do define them, then replace copy operations whenever the arg is a temporary (rvalue)
### **Copy/Move Elision**

```C++
vector makeAVector() {
    return vector{} //   // Basic constructor
}

vector v = makeAVector();   // move ctor? copy ctor?
```

Try in g++, just the basic constructor, not copy/move. This can be done by using `cout` in the copy/move ctors.

In some circumstances, the compiler is allowed to skip calling the copy/move constructors (but doesn't have to). `makeAVector()` writes its result directly into the space occupied by `v`, rather than copy/move it later.

For C++17, many elision opportunities become mandatory.

Ex.
```C++
vector v = vector{};    // Formally a basic construction and a copy/move construction
                        // vector{} is a basic constructor
                        // Here though, the compiler is *required* to elide the copy/move
                        // So basic constructor here only 

vector v = vector{vector{vector{}}};    // Still one basic ctor only
```

Ex.
``` C++
void doSomething(vector v) {...};   // Pass-by-value - copy/move ctor

doSomething(makeAVector());
```
Result of `makeAVector()` written directly into the param, there is no copy/move

This is allowed, even if dropping ctor calls would change the behaviour of the program (ex. if the constructors print something).

If you really need all of the constructors to run:

`g++14 -fno_elide_constructors ...`

**Note:** while possible, can slow down your program considerably

- Copying is an expensive procedure, the compiler skips these constructors as an optimization

In summary: Rule of 5 (Big 5)

- If you need to customize any one of
  1. Copy constructor
  1. Copy assignment
  1. Destructor
  1. Move constructor
  1. Move assignment

then you usually need to customize all 5.

---
[The Copier is broken! <<](./problem_5.md) | [**Home**](../README.md) | [>> I want a constant vector](./problem_7.md)
