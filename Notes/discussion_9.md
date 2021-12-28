# Discussion 9: Ownership of Strings (C++17)

We know that a string owns its characters.
- hence, any copy of a string (ie copy construction, pass by value etc) is a deep copy.
- deep copies mean that changes to a string will not be replicated in the copies.
- but if we don't need to change the string?

But what if we don't need our own copy? We have several options: 

1. Call `s.data()`;
    - use the underlying char array
2. Use a reference to the string `s`.

i.e. why are we here? Isn't this a solved problem?
- just pass the string by reference!

But consider the following:
```cpp
void f(const string &s);
f("hello");
```
- here, `"hello"` is a `char*`, and it's sitting in the literal pool right now.
- so, a new string must be constructed as a temporary, with a new heap array, containing a copy of `"hello"`, so that we have something to refer to.
- thus, it's a deep copy of an existing `char *` literal, and this is expensive if the string is long.

Aside: the **short string optimization.**

## Short String Optimization (SSO)
Usually, strings are stored like this:
```cpp
class string {
    char *theString;
    size_t size, cap;
};
```
But, if the string is short, it could be stored like this:
```cpp
class string {
    size_t size;
    char theString[MAX];
};
```
So, the general implementation of string is as follows:
```cpp
class String {
    size_t size;
    union {
        struct {
            char *theString;
            size_t cap;
        };
        char theString[sizeof(char *) + sizeof(size_t)];
    };
};
```

(back to main topic)

Or, we could take a `char *` in `f`:
```cpp
void f(const char *)
```
- but this isn't C.
- now we don't have access to string methods, and so we have to deal with null terminators, etc.
- gross.

To solve this, C++17 introduces `std::string_view`.
- essentially, this is a non-owning string class.

The implementation is basically using two pointers.
```cpp
#include <string_view>

using namespace std;

string s {"Hello world"};
string_view sv {s};
```
The advantage of using `string_view`:
```cpp
void f(string_view sv) {
    ...
}
```
- this passes two pointers
- so there is no deep copying.
```cpp
f(s)
```
- this also passes two pointers.
```cpp
f("hello world")
```
- this passes two pointers
- so there is no temporary string object.

Within `f`, we have the `string_view`. What can we do with it?
- we do have method calls on `string_view` objects:
    - `find` - find a substring within the view; 
    - `remove_prefix(n)` - advance the first pointer by n
    - `remove_suffix(n)` - back up the second pointer by n
    - `substr` - produces a view f the range `[m, n)`.

Note:
- `string::substr` produces a new string (so this is expensive); but
- `string_view:substr` produces a new view (so this is cheap.)

