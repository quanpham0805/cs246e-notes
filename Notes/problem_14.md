[I want a vector of Posns <<](./problem_13.md) | [**Home**](../README.md) | [>> Memory management is hard](./problem_15.md)

# Problem 14: Less copying!
## **2021-10-07**

We need to make this work for all type `T` efficiently.

**Before:** 
```C++
// perfectly fine
void push_back(int n);
```

**Now:** 
```C++
// T can be very complicated, passing by value is kinda sus
void push_back(T x) { // (1) If T is an object, how many times is T being copied?
    increaseCap(); 
    new(theVector + (n++)) T(x);   // (2)
}
```

If the arg is an lvalue:  
- (1) is a copy constructor
- (2) is a copy constructor
- 2 copies, we want 1

If the arg is an rvalue:
- (1) is a move constructor
- (2) is a copy constructor
- 1 copy, we want 0

**fix:**
```C++
void push_back(T x) {
    increaseCap(); 
    new(theVector + (n++)) T(std::move(x));
}
```
Seems good so far:
- **lvalue:** copy + move  
- **rvalue:** move + move

But the issue is: what if `T` doesn't have a move constructor? It would make 2 copies. Can we make this so that it would work regardless of `T` having a move ctor or not?

**Better:** take `T` by reference
```C++
// lvalue
void push_back(const T &x) {    // No copy, no move
    increaseCap();
    new(theVector + (n++)) T(x);   // Copy constructor
}

// rvalue
void push_back(T &&x) { // No copy, no move
    increaseCap();
    new(theVector + (n++)) T(std::move(x)); // copy ctor will run if there is no move ctor
}
```    

- **lvalue:** 1 copy  
- **rvalue:** 1 move

If no move constructor: 1 copy

Now consider:
```C++
Vector<Posn> v;
// What could go wrong????
v.push_back(Posn {3, 4});
```

1. Constructor call to create the Posn object
1. Copy or move constructor into the vector (depending on whether Posn has a move constructor)
1. Destructor call on the temporary object

Having ctor and then dtor, they cancel out, sounds like a waste.

Could eliminate (1) and (3) if we could get vector to create the object instead of the client
- Pass constructor args to the vector and not the actual object
- How? Soon, but first...

### **A note on template functions**

Consider: `std::swap` seems to work on all types

**Implementation:**
```C++
// Without std::move, it would be good c++98, not so good c++11
template<typename T> void swap(T &a, T &b) {
    T tmp{std::move(a)}
    a = std::move(b);
    b = std::move(tmp);
}
```

```C++
int x = 1;
int y = 2;
swap(x, y)  // Equiv swap<int>(x, y);
```

Don't have to say `swap<int>`, C++ can deduce this from the types of `x` and `y`

In general, only have to say `f<T>(...)` if `T` cannot be deduced from the args

Type deduction for template args follows the same rules as type deduction for `auto`

### Back to Vector passing constructor args

- We don't know what types constructor args should have
- `T` could be any class, could have several constructors

Idea - member template function (like `swap`, it could take anything)

**2nd Problem:** how many constructor args?

**Solution:** _variadic templates_ (similar to Racket macros)

```C++
template<typename T> class vector {
    // ...
    public:
    // ...
    template<typename... Args> void emplace_back(Args... args) {
        increaseCap();
        // just pass args... as is :D
        new(theVector + (n++)) T (args...);
    }
};
```

In this case, `...` in template actually represents a variable amount of arguments
- `Args` is a _sequence_ of type vars denoting the type of the actual args of `emplace_back`  
- `args` is a _sequence_ of program vars denoting the actual args of `emplace_back`

```C++
vector<Posn> v;
v.emplace_back(3, 4);
``` 

**Problem:** args is being taken by value, can we take args by reference? (and should we take lvalue or rvalue reference?) for better efficiency.
- When we were implementing `push_back`, there was only 1 argument, and it can only be either a lvalue or a rvalue. But now we have a sequence of them, which we could have a mixture of both lvalue and rvalue.

```C++
template<typename... Args> void emplace_back(Args&&... args) {
    increaseCap();
    new(theVector + (n++)) T (args...);
}
```
**Note:** It looks like we are taking rvalue ref, but this is **NOT TRUE**

Special rules here: `Args&&` is a **universal reference** (officially: **forwarding reference**)
- Can point to an lvalue or an rvalue.
- When is a reference universal? Must have the form `T&&`, where `T` is the type arg being deduced for the current **template function call**.

Ex.
```C++
template<typename T> class c {
    public:
        template<typename U> void g(U&& x); // Universal
        template<typename U> void h(const U&& x);   // Not universal (because of const)
        void j(T&& x);  // Not universal (not being deduced, T is already known, and is a template parameter of a class, not a function)
};
```

Now recall:

```C++
class C {...};
void f(C&& x) { // rvalue reference - x points to an rvalue, but x itself is an lvalue
    g(x);   // x is passed as an lvalue to g
}
```

If you want to preserve the fact that `x` is an rvalue reference, so that a "moving" version of `g` is called (if it exists):

```C++
void f(C&& x) {
    g(std::move(x));
}
```

In the case of `args`, we don't know if the args are lvalues, rvalues, or a mix.  
Want to call `move` on `args` if and only if the args are rvalues.

```C++
template<typename... Args> void emplace_back(Args&&... args) {
    increaseCap();
    new(theVector + (n++)) T (std::forward<Args> (args)...);
}
```

`std::forward` calls `std::move` if its argument is an rvalue reference, else does nothing

Now `args` is passed to `T`'s ctor with rvalue/lvalue information preserved. This technique is called **perfect forwarding**.

---
[I want a vector of Posns <<](./problem_13.md) | [**Home**](../README.md) | [>> Memory management is hard](./problem_15.md)
