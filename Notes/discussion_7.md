
# Unions Revisited (C++17)

Recall the use `dynamic_cast` to query the run-time type of an object in a
class hierarchy - we said that it was bad style. The reason is that existing
code may no longer work after adding additional classes to the hierarchy. So in
this case, a virtual method would be preferred over `dynamic_cast`.

'Subtype polymorphism' is about having the same approach to all possible types
in the class hierarchy, so that the client code doesn't need to care about the
actual type.

This entire argument is predicated on one assumption - that you might need to
create more classes. What if you know FOR SURE that you won't?

Example:

Suppose you are writing a c compiler and you have a class for loops. There are
only 3 types of loops: for, while and do - and that hasn't changed for decades.

The for loop is different that while and do, in that it has an update
condition. Should we force a uniform interface and give while and do a
`getUpdate` that returns empty, or should we just use dynamic cast and take
advantage of the specialization?

Our problem of storing heterogeneous data hasn't completely been solved,
imagine we would want to store a collection of students and comic books, does
it really make sense to make an artificial super class of students and comic
books?

So let's revisit unions. C++17 offers a type-safe alternative to unions:
`std::variant`.

```cpp
#include <variant>

// storing items in variant
variant<For, Do, While> loop{For{}};

// discriminating the value
if (holds_alternative<For>(loop)) {
    cout << "for loop";
} else {
    ...
}

// extracting the value
try {
    For f = get<For>(loop);
    ...
} catch(bad_variant_access& err) {
    // it was not a for
    ...
}
```

When a variant if first defined, if no initializer is given, it is set to the
first type in the list.
```cpp
variant<For, Do, While> v; // default initialized to For{}
```
What happens if the first type wasn't default constructable.
- don't default initialize the variant
- reorder the types so the first one has a default constructor
- use std::monostate as the first type - just a dummy type

Note: `variant<monostate, T>` is like a maybe type in haskell

What if we had:
```cpp
variant<int, int> v;
cout << get<int>(v);
```
this does not compile, we do not know which int to get

To get around this:
```cpp
variant<int, int> v;
cout << get<0>(v);
```
and attempting to use `get<1>(v)`, will result in a `bad_variant_access`, this
is good, cpp is keeping track of which int we stored.

And now, if we wanted to set the second int:
```cpp
variant<int, int> v{in_place_index<1>, 4};
cout << get<1>(v);
```

