
# if / switch initialization (c++17)

Inspiration:
```
for (int i = 0; i < n; ++i)
```
The scope of `i` ends when the loop is done. would be nice to have something
similar for if and switch.

Something like this is already legal in c++14
```
if (int errorcode = doSomething(()) {
    // handle error
    // errorcode is in scope
} else {
    // normal behavior
    // errorcode is in scope
}
```
However, this is only helpful for error checking, where success means a return
value of zero, and error means non-zero.

But in c++17:
```
if (int result = doSomething(); result >= 0) {
    // do ur stuff
    // result is in scope
} else {
    // do ur other stuff
    // result is in scope
]
```

This also works with switch
```
switch (Token t = getToken(); t.tokenType) {
    // rest of switch stuff
}
```

# Spaceship operator (c++20)

Comparing strings in c: `strcmp(s1, s2)` - compares lexicographically, returns
a number depending on the result (<0, 0 or >0).

Comparing strings in cpp: `s1 < s2`, `s1 == s2` etc, since we can overload
operators. Much easier to read/write. However, it's a lot messier if we want to
have three outcomes:
```
if (s1 < s2) {
    // ...
} else if (s1 == s2) {
    // ...
} else {
    // ...
}
```
Notice that this uses two comparisons, unlike strcmp.

c++20 introduces the spaceship operator `<=>`.
```
#include <compare>

std::string s1 = ...;
std::string s2 = ...;

std::strong_ordering s = s1 <=> s2;

if (s < 0) {
    // ...
} else if (s == 0) {
    // ...
} else {
    // ...
}
```
Now there is only one comparison being computed.

What is a strong ordering? If two objects compare to be equal, the must be
indistinguishable.

## Adding spaceship to your class

```
struct Posn {
    int x, y;

    std::strong_ordering operator<=>(const Posn& other) const {
        return (x <=> other.x) ? (x <=> other.x) : (y <=> other.y);
    }
};
```
But there's more, if you implement spaceship, cpp will automatically define the
6 relational operators in terms of spaceship, so you get them all for free.

ie. p1 <= p2   --->   (p1 <=> p2) <= 0

There's even more, we can in fact do:
```
struct Posn {
    int x, y;

    std::strong_ordering operator<=>(const Posn& other) const = default;
};
```
Spaceship does not come by default, but we can ask for it. The default behavior
of spaceship is lexicographic on the fields.

But when might you not want a lexicographic ordering?

wait till next week.

