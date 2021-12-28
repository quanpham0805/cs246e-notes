# Discussion 8: Revisiting `void*` (C++17)
This will be a short topic.

Last time, we had `std::variant` as a type-safe alternative to unions,
- which is a constrained choice among types.

So what about `void*`?
- ie can be literally anything.
- can we make this type-safe?

Well, in C++17, we get `std::any`.

```cpp
#include <any>

std::any a {1}; // a holds an int
a = std::string {"hello"}; // a holds a string
```

Ownership semantics:
- if `a` is reassigned, it destructs what it held before.

Fetching the value:
- we must *cast* it to the correct type.
- we do this by the following:
```cpp
string str = std::any_cast<string>(a); // value semantics
string &rstr = std::any_cast<string &>(a); // reference semantics
string *pstr = std::any_cast<string>(&a); // ptr semantics

// if we try to do 
std::any_cast<int>(a); // this throws std::bad_any_cast
```

How is it implemented?
```cpp
class any {
    void *theAny; // starts out as a nullptr
    typeinfo t;

    public:
        template<typename T> any(T x):
            theAny {new T(x)}, t {typeid (T)} {}
        //etc
};
```
