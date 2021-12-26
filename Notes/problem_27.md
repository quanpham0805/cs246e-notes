[Generalize the Visitor pattern <<](./problem_26.md) | [**Home**](../README.md) | [>> I want to print the unprintable!](./problem_28.md)

# Problem 27: I want an even faster vector
## **2021-11-18**

In the good old days of C, you could copy an array (even an array of structs!) very quickly by calling a function `memcpy` (similar to `strcpy`, but for arbitrary memory, not just strings).

`memcpy` was probably written in assembly, and was as fast as the machine could possibly be.

Nowadays in C++, copies invoke copy constructors, which are costly function calls. I long for the good old days 😢

_Good news!_

In C++, a type is considered **POD (plain old data)** if it:
- has a trivial default constructor (equiv. to `= default`)
- is trivially copyable 
  - big 5 all have default implementations
- is standard layout
    - no virtual methods or bases
    - all members have the same visibility
    - no reference members
    - no fields in both base class & subclass, or in multiple base classes

For POD types, semantics is compatible with C, and `memcpy` is safe to use.

How can we use it? - Only safe to use if `T` is a POD type

_One option:_
```C++
template<typename T> class vector {
private:
    size_t n, cap;
    T *theVector;
public:
    // ...
    vector(const vector &other): 
        theVector{static_cast<T*>(operator new (other.n * sizeof(T)))}, n{other.n}, cap{other.cap} {
        // value is true if T is pod, false otherwise
        if (std::is_pod<T>::value) {
            memcpy(theVector, other.theVector, n * sizeof(T));
        } else {
            // as before
        }
    }
}
```
Works... But condition is evaluated at run-time, but the result is known at compile-time (compiler may or may not optimize)

_Second option (no run-time cost):_
```C++
template<typename T> class vector {
// ...
public:
    template<typename X = T>
    vector(enable_if<std::is_pod<X>::value, const T&>::type other): 
        theVector{static_cast<T*>(operator new (other.n * sizeof(T)))}, n{other.n}, cap{other.cap} {
            memcpy(theVector, other.theVector, n * sizeof(T));
        }

    template<typename X = T>
    vector(enable_if<!std::is_pod<X>::value, const T&>::type other):
        theVector{static_cast<T*>(operator new (other.n * sizeof(T)))}, n{other.n}, cap{other.cap} {
            // original implementation
        }
}
```
How does it work?
```C++
template<bool b, typename> struct enable_if;
template<typename T> struct enable_if<true, T> {
    using type = T;
};
```
- Yea this brings back memories of lambda calc
- If the boolean is false, it has no definition, does not exist!

With metaprogramming, what you don't say is as important as what you do say.

If `b` is `true`, `enable_if` defines a struct whose `type` member typedef is `T`. So if `std::is_pod<T>::value == true`, 
- then `enable_if<std::is_pod<T>::value, const T&>::type => const T&` by substitution

If `b` is `false`, the struct is declared but not defined. So `enable_if<b, T>` will not compile.

So one of the two versions of the copy constructor won't compile (the one with the `false` condition).

Then how is this a valid program?

**C++ rule:** <u>SFINAE (Substitution Failure Is Not An Error)</u>

In other words - if `t` is a type,
```C++
template<typename T> __ f(___) { ____ }
```
is a template function, and substituting `T = t` results in an invalid function, the compiler does _not_ signal an error - it just removes that instantiation from consideration during overload resolution.

On the other hand, if _no_ version of the function is in scope and substitutes validly / to handle the overload call, that is an error.

Question: why is this wrong (having no `template<typename X = T>` compared to the above)?
```C++
template<typename T> class vector {
    // ...
public:
    vector(typename enable_if<std::is_pod<T>::value, const T&>::type other);
    // ...
    vector(typename enable_if<!std::is_pod<T>::value, const T&>::type other);
};
```
That is, why do we need the extra template out front? 

Because SFINAE applies to template functions and these methods are ordinary functions (constructors), not templates.
- They depend on `T`, but `T`'s value was determined when you decided what to put in the vector
- If substituting `T = t` fails, it invalidates the entire `vector` class, not just the method

So make a separate template, with a new arg `X`, which can be defaulted to `T`, and do `is_pod<X>`, not `is_pod<T>`. 

... It compiles, but when we run it, it crashes

**Why????? Hint:** If you put debug statements into both of these constructors, they don't print. Which means they did not run.

**Ans:** We're getting the compiler-supplied copy constructor, which is doing shallow copies. Eventually leads to double free and there u go :yep:.

These templates are not enough to suppress the auto-generated copy constructor. A non-templated match is always preferred to a templated one.

What do we do about it?

Could try: disabling the copy constructor
```C++
template<typename T> class vector {
// ...
public:
    vector(const vector &other) = delete;
}
```
Not allowed, can't disable the copy constructor and then create another copy constructor.

Solution that actually works: **overloading**
```C++
template<typename T> class vector {
    // ...
    // a dummy structure just give me a type so I can overload on
    struct dummy{};
public:
    // create a copy ctor
    // just gonna have it call my actual constructor I wanna use (the 2 ctors below)
    // btw this is constructor delegation, if both ctor has body, then the other ctor's body would run first, and this current body will run after
    vector(const vector &other): vector{other, dummy{}} {}

    // Now we can do our SFINAE
    template<typename X = T> 
    vector(typename enable_if<std::is_pod<X>::value, const T&>::type other, dummy) { ... }

    template<typename X = T> 
    vector(typename enable_if<!std::is_pod<X>::value, const T&>::type other, dummy) { ... }
};
```
- Overload the constructor with an unused `dummy` arg
- Have the copy constructor delegate to the overloaded constructor
- Copy constructor is inline, so no function call overhead
- Most importantly, this works!

Can write some "helper" definitions to make `is_pod` and `enable_if` easier to use (C++ 14)
```C++
// convention: if the name has _v, it would extract the value field
template<typename T> constexpr bool is_pod_v = std::is_pod<T>::value
// convention: if the name has _t, it would extract the type field
template<bool b, typename T> using enable_if_t = typename enable_if<b, T>::type
```
- Standardized conventions since C++ 14

Now slightly cleaner
```C++
template <typename T> class vector {
    // ...
    struct dummy{};
public:
    // ...
    template<typename X = T> vector(enable_if_t<is_pod_v<X>, const T&> other, dummy);

    template<typename X = T> vector(enable_if_t<!is_pod_v<X>, const T&> other, dummy);
};
```

## Move / Forward implementation

<u>Aside:</u> We now have enough machinery to implement `std::move` and `std::forward`.
- _`std::move`_ - treat a lvalue as a rvalue (cast).
- First attempt
```C++
template<typename T> T &&move(T & x) {
    return static_cast<T &&>(x);
}
```

Doesn't quite work, `T&&` is a universal reference, not necessarily an rvalue reference. If `x` was a lvalue reference, `T&&` is a lvalue reference.
- Need to make sure `T` is not an lvalue reference
    - If `T` is an lvalue reference, get rid of the reference
    - Basically we get rid of all the ref to get the bare type with `std::remove_reference`.

```C++
template<typename T> inline typename std::remove_reference<T>::type&& move(T&& x) {
    return static_cast<typename std::remove_reference<T>::type &&>(x);
    // turns T&, T&& into T
}
```

**Exercise:** write `remove_reference`

**Q:** can we save typing and use `auto`? Ex.

```C++
template<typename T> auto move(T &&x) { ... }
```

**A:** No! By-value auto throws away reference and outer const
- Recall that `auto x = y` gives x the same type as the **value** of y (i.e, the type of `y` as if it was copied)

```C++
int z;
int &y = z;
auto x = y; // x is an int

const int &w = z;
auto v = w; // int
```

By reference, `auto &&` is a universal reference, so the code from the question section works (it does compile), but not as we expected.

But still... is there a way to do type deduction without losing ref and const?

Need a type definition rule that doesn't discard references.

The answer is yes, by using `decltype`. It produces the type its argument was declared to have.

**Note**: The argument of `decltype` is never evaluated, only type-checked.

```C++
decltype(var)  // returns the declared type of the variable
decltype(expr) // returns lvalue or rvalue, depending on whether the expr is an lvalue or rvalue

int z;
int &y = z;
decltype(y) x = z;  // x is an int&, auto would only give you int
x = 4;  // Affects z

/* Path/Example 1 */
auto z;
x = 4;  // Does not affect z

/* Path/Example 2 */
decltype(z) s = z;  // s is an int
s = 5; // Does not affect z

/* Path/Example 3 */
decltype((z)) r = z;    // r is an int&, since (z) is a ref, since () is an expression
r = t;  // Does affect z

decltype(auto) - perform type deduction, like auto, but use the decltype rules
```

Can we do type deduction using the `decltype` rules instead of the auto rules? 

Yes - use `decltype(auto)`

```C++
template<typename T> decltype(auto) move(T &&x) {
    return static_cast<std::remove_reference_t<T>&&>(x);
}
```

- Let's try implementing `std::forward`
_`std::forward`_
```C++
template<typename T> T&& forward(T &&x) {
    return static_cast<T&&>(x);
}
```
- Doesn't seem right - casting `x` to its own type.

**Reasoning:**
- If `x` is an lvalue, `T&&` is an lvalue reference
- If `x` is an rvalue, `T&&` is an rvalue reference

Doesn't work, `forward` is called on expressions that are lvalues, that may point at rvalues.

```C++
template<typename U> void f(U&& y) {
    ... forward(y) ...  // y is an lvalue
}
```

`forward(y)` is will _always_ yield an lvalue reference.

In order to work, `forward` must know what type (including l/rvalue) was deduced for `y`, ie. needs to know `U`.

So in principle, `forward<U>(y)` would work.

**2 Problems:**
- Supplying `T` means `T&&` is no longer universal
- Want to prevent the user from omitting `<T>`

**Instead:** separate lvalue/rvalue cases

```C++
template<typename T> 
inline constexpr T&& forward(std::removed_reference_t<T>&x) noexcept {
    return static_cast<T&&>(x);
}

template<typename T> 
inline constexpr T&& forward(std::removed_reference_t<T>&&x) noexcept {
    return static_cast<T&&>(x);
}
```

---
[Generalize the Visitor pattern <<](./problem_26.md) | [**Home**](../README.md) | [>> I want to print the unprintable!](./problem_28.md)