# Discussion 5: Class Template Argument Deduction (C++17)

Recall:
```cpp
int x = 1, y = 2;
std::swap(x,y);
```
- we don't need to say `swap<int>(x, y)` (we can, but we don't need to.)
- the `int` is inferred from the types of `x` and `y`.

```cpp
template <typename T> void swap(T &x, T &y);
```
- so for `swap(1,2)`, `T&x` and `T&y` are `ints`, so it is inferred that `T` is `int`.
- however, we could not do this for template classes.

```cpp
template <typename T, typename U> struct pair {
    T first;
    U second;
};
```

- we cannot do this in C++14: `pair p {1, 'a'};`
    - because 1 is int, and 'a' is char
    - we would need to say `pair<int, char> p {1, 'a'};`
- or as an rvalue: `f(pair<int, char> {1, 'a'});`
- the standard workaround for this are the `make_` helper functions:
```cpp
template <typename T, typename U>
pair<T, U> make_pair(T x, T y) {
    return pair<T, U> {x, y};
}
```
Now, we can do this:
```cpp
auto p = make_pair(1, 'a');
```
- since `1` is `int`, and `'a'` and `char`, it is deduced that `p` is `pair<int, char>`.
- though, to do this right, `make_pair` is a passthrough function, 
    - and anytime we have a pass-through function, we should be doing **perfect forwarding**.
- for instance:
```cpp
template <typename T, typename U> pair<T, U> make_pair(T &&x, U &&y) {
    return pair<T, U> {
        std::forward<T>(x), 
        std::forward<U>(y)
    };
}
```

However, in C++17, template argument deduction for classes is enabled, so we can now write stuff like this:
```cpp
pair p{1, 'a'}; // p is of type pair<int, char>.
// we can also write is as an rvalue:
auto g = pair{2, 'b'};
```
How does this work?
- the compiler performs *template argument deduction* on the constructors of the class.
- if there is a matching constructor, and it is enough to deduce all of the template arguments, it succeeds.
- or the remaining template parameters have defaults.

For instance:
```cpp
template <typename T, typename U> struct pair {
    T first;
    U second;
    pair(T a, U b) : first{a}, second{b} {}
};

auto q = pair{2, 'b'};
// compiler attempts to match {2, 'b'} against the constructors' arguments.
// so it matches against the first constructor, so first argument T = int, and second argument U = char
// this covers all the template parameters
// so type of q is "implied" to be pair<int, char> (we're good!)

```
- Note that the deduction also works in the case where the constructor is omitted (so we just use the default C-style initialization).

When might the deduction fail?
- it might fail if there is not a "clear relationship" between the template arguments and the constructor arguments.
- for instance, say we have something like this:
```cpp
template <typename T> class C {
    T x;
};

C{"abc"};
```
- T will be implied to be `const char *`, not `string`.
- if we wanted the type to be `string`, we could help the compiler by providing a **deduction guide**
- which works by saying that whenever we have a `C<const char *>`, we deduce that `C<string>`.
- ie 
```cpp
C(const char *) -> C<string>`.
```
- or, maybe store a `unique_ptr` when we have a raw pointer:
- ie
```cpp
template <typename T> C(T*) -> C<unique_ptr<T>>;
```
- note that the above only works if `unique_ptr`'s constructor is non-explicit.
