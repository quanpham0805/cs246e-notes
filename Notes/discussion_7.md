# Discussion 7: Unions Revisited (C++17)

Recall the use of dynamic_cast to query the run_time type of an object in a class hierarchy.
- and we said it was usually bad style.
- Why?
    - what if we want to add more classes to the hierarchy?
    - we need to rewrite a bunch of stuff.
- instead, we should write virtual methods (or visitors)

Subtype polymorphism is about having a uniform approach to all possible types in the class.
- so that we never need to rewrite anything.
- but, all of this is predicated on one assumption: that we might need to make more classes.
    - but what if we know for sure that we won't need to?
    - maybe an `A` object can *only* be either a `B` or a `C` object,
        - and maybe this will not change?
        - or if it does, it will have enough of an impact that widespread rewriting is truly unavoidable?

Here's an example:
- suppose we are writing a C compiler.
- and we have a class for loops.
    - there are only 3 kinds of loops: for, while and do.
    - and this has not changed for decades.
    - in this case, it is really so bad to have specialized code for `for`, `while`, `do`, and to start by asking what kind of loop we have?
    - also, doing everything by virtual methods forces you to impose a uniform interface on classes that might not be similar.
    - this makes sense if the possible subclasses are theoretically unbounded, but not here.
    - here, it makes sense to give each class its own specialized interface.

So maybe unions aren't so bad?
- well, yes, they are`
- since unions are not type-safe, and invite abuse.

But, in C++17, we are offered a type-safe alternative to unions: `std::variant`.
- we get this by `#include <variant>`.

How might we use this?
```cpp
variant<For, Do, While> loop;
// or
using Loop = variant<For, Do, While>;
Loop loop;
```

Storing items in the variant:
```cpp
variant<For, While, Do> loop {For {}};
```

Discriminating the value:
```cpp
if (holds_alternative<For>(loop)) {
    cout << "for loop" << endl;
}
else {...}
```

Extracting the value:
```cpp
try {
    For f = get<For>(loop);
    // use f...
}
catch (bad_variant_access&) {
    // handle error
}
```
- note that storing one type in a variant and attempting to fetch it as another type will throw!

When a variant is frst defined, if no initializer is given, it is set to the first type in the list.
```cpp
variant<For, Do, While> v; // set to For {};
```
- this type is default initialized.

But what if the first type doesn't have a default constructor?
- then, the program will not compile.

So what do we do about this?
1. We could reorder the types so that the first one has a default constructor;
2. Use `std::monostate` as the first type.
    - this is a dummy type that can be used as a default.
    - note: `variant<monostate, T>` gives us a way to mimic `Maybe T` (like in Haskell).
    - we also have `std::optional<T>`, which does the exact same thing.

