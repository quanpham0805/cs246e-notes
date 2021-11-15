
# `void*` revisited (C++17)

last time: std::variant as type-safe alternative to unions.

this time we will discuss `void*` - a pointer to literally anything. Can we
make `void*` typesafe?

in C++17, we get `std::any`.

```cpp
#include <any>

std::any a{1}; // holds an int
a = std::string{"poo"}; // holds a string
```

ownership semantics - the any is given ownership of the thing it holds, if it
is reassigned, it will properly destroy the thing it was previously holding.

how do we access the value the any is holding? you must cast it to the correct
type.

```cpp
string str = std::any_cast<string>(a);    // value
string& rstr = std::any_cast<string&>(a); // ref to a
string* pstr = std::any_cast<string>(&a); // pointer to a
```

what happens if we try to cast the any to something that it's not?

```cpp
std::any_cast<int>(a); // this will throw std::bad_any_cast
```

as to how `std::any` could be implemented:
```cpp
class any {
    void* theAny;
    typeinfo t;
    template<typename T> any(T x): theAny{new T(x)}, t{typeid(T)} {}
};
```
so when we wish to use `any_cast`, we check to see if the type requested
matches the typeinfo we hold and just use `static_cast`.

# Ownership of strings (C++17)

a string owns it's characters, thus any copy of a string is a deep copy. deep
copies means that changes to a string will not be replicated in the copies.

but, what if you don't need to change the string, what if you don't need your
own copy? you could of course, just use a reference. however, consider:
```cpp
void f(const string& s);
f("hello");
```

notice that "hello" is a `char*` that is sitting in the literal pool. but what
is really happening is that a new string is constructed as a temp object with a
new heap array containing a copy of hello and passed to `f`. we are deep
copying an existing `char*` literal. this can be expensive if the string is
long.

## ASIDE - short string optimization (SSO)

a string may look something like
```cpp
class string {
    char* theString;
    size_t size, cap;
};
```
this string takes roughly about the size of three pointers. this is fine if we
have a decent length string, but what if the string is very short, like just
one letter? then the following might be a better for short strings:
```cpp
class string {
    char theString[MAX];
    size_t size;
};
```
you break even when `MAX` is equal to the size of two pointers (16 bytes). so a
string might actually be implemented like:
```cpp
class string {
    size_t size;
    union {
        struct string {
            char* theString;
            size_t cap;
        };
        char theString[sizeof(char*)+sizeof(size_t)];
    };
};
```
end of aside.

we present `std::string_view` (C++17) - a non-owning string class. the
implementation: two pointers to the string (one to the front, one to the end).
```cpp
#include <string_view>
string s{"peepee poopoo"};
string_view sv{s};
```

the advantage of this is that when we write our `void f(string_view sv)`, this
is only just copying two pointers, no deep copying involved and no temporary
string object.

now how do we use the `string_view`? we have methods that make sense for when
we are just looking at a string, and don't intend on modifying it, methods such
as `find`, `remove_prefix(n)` and `remove_suffix(n)`. `remove_prefix` and
`remove_suffix` are very cheap, we are just changing where the pointers are
pointing at. these methods don't modify the original string, and are very cheap
to use.

a couple of caveats, the `string_view` can outlive the original string, leaving
pointers to memory that we don't own.

