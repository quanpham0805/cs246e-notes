# Discussion 4: Structure Bindings (C++17)

Recall the iterator pattern, and specifically, range-based loops:
- convenient traversal of classes that provide the right priorities; 
- e.g. vectors and lists.
- what could we do for structures?

Motivation - iterating over a map.
- map is a set of key value pairs, specifically `std::pair<key, value>`.
- eg
```cpp
map<string, int> m;
...
for (auto &p: m)
    // p is a (key, value) pair
    cout << p.first << " " << p.second << endl;
    // p.first is the key
    // p.second is the value.
```
- note that `p.first` and `p.second` are fields.

In C++17, we have **structure bindings**. For example, for the struct `Posn`:
```cpp
struct Posn {
    int x, y;
}
...
Posn p { 1, 2 };
// structure binding
auto [a,b] = p;
```
- this also works for stack arrays:
```cpp
int a[] = {1,2,3};
auto [x,y,z] = a; // x = 1, y = 2, z = 3
```
- note that the number of variables in the binding has to *match* the number of things in the struct/array.
- so, map iteration becomes:
```cpp
for (auto &[key, value]: m)
    cout << key << " " << value << endl;
```

Semantics:
- when we say `auto [a,b] = ...`, `a` and `b` are created by **value**.
    - changes will not affect the original (ie they are deep copies).
- but if we say `auto &[a,b] = p;`, then `a` and `b` are created by **reference**. 
    - then changes to the variables will also affect `p`.
    - in this case, `p` must be an lvalue.
    - note that if *any* of the fields of the struct you are trying to get are private, you cannot do this.
- we can use structure bindings on any struct whose fields are **all** public (so this does not break encapsulation).

## Structure Bindings for Encapsulated Classes

Can we provide customized structure bindings for encapsulated classes? - Yes! but... 
- it is complicated, because the elements of a struct are not necessarily the same type.
- so any structure binding we implement needs to type check (to make sure everything is well defined).
- so, we need to tell the compiler three things:
    1. The number of items in the binding;
    2. The type of each item in the binding; and
    3. What each item request should return.

So, how do we provide this information?
- we can accomplish this via **template specialization** (will be explained in future lectures).
- this is the only time we are allowed to add something to the `std` namespace: 
    - specializing a template that is already in namespace `std`.
    - in particular, in the header `<tuple>`:
        - `std::tuple_size<T>` - associates a type `T` with a number representing the number of tuple items.
        - `std::tuple_element<n,T>` - this gives the type of the nth eement of tuple `T`.
        - a member template `get<n>` - returns a reference to the nth item.

```cpp
class Posn {
        int x, y;
    public:
        Posn(int x, int y) : x{x}, y{y} {}
        template <size_t n> auto &get() {
            return (n == 0) ? x : y;
        }
};
```
- note that we can use `get<0>` and `get<1>` to access the fields of the struct.
- but note that `get<0>` and `get<1>` are *two different functions!*
    - as `n` is specified in compile-time.
    - which means that `get<0>` and `get<1>` can have two different return types!
    - this is done via using `auto`.
- we will revisit this later.

Specifying `tuple_size` and `tuple_element`:
```cpp
namespace std {
    template <> struct tuple_size<Posn> {
        // specifies that value is a compile-time global quantity
        static const int value = 2;
    };

    template <> struct tuple_element<0, Posn> {
        using type = int;
    };

    template <> struct tuple_element<1, Posn> {
        using type = int;
    };
}
```

How do we use these in code?
```cpp
Posn p {1, 2}; // x and y are private fields!
auto &[a,b] = p; // a = 1, b = 2
++a; ++b;
cout << a << ' ' << b << endl; // 2 3

auto &[c,d] = p;
cout << c << ' ' << d << endl; // 2 3
```
- so this is how we support structure bindings in a class.
