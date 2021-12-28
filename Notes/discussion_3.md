# Discussion 3: The Spaceship Operator (C++20)

Inspiration: remember what is was like to compare strings in C:
```cpp
strcmp(s1, s2);
```
- `strcmp` does a lexicographic (ie "dictionary" order) comparison between the two strings.
- returns:
    - < 0 if s1 comes before s2;
    - 0 if s1 = s2; and
    - \> 0 if s1 comes after s2.

But when we compare strings in C++:
```cpp
s1 < s2; s1 <= s2; s1 == s2; ...
```
- we can compare strings using the conventional relational operators.
- so this is easier to read and write...
- ...but relational comparisons are only regarded as a "three outcome" operator (eg less than, equal, more than).
- so we would need to do something like:
```cpp
if (s1 < s2) {
    // ...
}
else if (s1 == s2) {
    // ...
}
else {
    // ...
}
```
- this works, but this is not as good as `strcmp`
- because we may have to do the comparison twice. (eg if s1 > s2).
- but `strcmp` returns a number, so we can immediately test that number to check how the strings are related.

So C++20 solves this issue by introducing a three way comparison operator, also known as the **spaceship operator**, which looks like `<=>`.

Example implementation:
```cpp
#include <compare>

string s1 = ..., s2 = ...;
std::strong-ordering s = s1 <=> s2;

if (s < 0) cout << "less";
else if (s == 0) cout << "equal";
else cout << "greater";
```
- so this just uses one comparison instead of two.
- instead of `std::strong-ordering`, we can say `auto`.
    - it figures out the type of the variable from how it is initialized.

Side note: what does *strong ordering* mean?
- if you have two objects `a` and `b` and you're comparing them, and they are equivalent, then under a strong ordering the objects would be classified as the same.
    - ie the objects are truly equal in every regard.
- in contrast, for a *weak ordering*, two objects may be *equal* even if the objects are not purely identical (eg for two students, they have the same name but different grade)

## Adding Support for <=> to a class
For instance:
```cpp
struct Posn {
    int x,y;
    std::strong-ordering operator<=>(const Posn &other) const {
        return (x <=> other.x) ? (x <=> other.x) : (y <=> other.y);
    }
};
```
So now, we can say something like this:
```cpp
Posn p1 {1, 2};
Posn p2 {1, 3};
p1 <=> p2;
```
But we can also say:
```cpp
p1 <= p2; ...
```
- the six relational operators (<, <=, >, >=, ==, !=) are automatically rewritten in terms of the spaceship operator.
- eg `p1 <= p2` gets rewritten to `(p1 <=> p2) <= 0`.

Moreover, we can also do this:
```cpp
struct Posn {
    int x,y;
    std::strong-ordering operator<=>(const Posn &other) const = default;
};  
```
- by saying `= default`, it gives the *default implementation* of the spaceship operator, which is equivalent to what we just wrote (comparison of fields in lexicographic order).
- but when might we want not this implementation?
    - eg the `Student` example discussed in Discussion Topic #2.
    - eg a comparison for the class `vector` (discussed in lectures).

Let's add a comparison to class `vector`:
```cpp
class vector {
    size_t n, cap;
    int *theVector;
};
```
- using the default spaceship operator, this does lexicographic comparison on these fields, which doesn't make sense!
- so default is not good enough.

Let's properly implement the spaceship operator:
- lexicographic comparison on the *items* in the array
```cpp
class vector {
    ...
    public:
        std::strong_ordering operator<=>(const vector &other) {
            // return the result of <=> on the first entry where theVector and other.theVector differ,
            // or, if one is the prefix of the other, the shorter one is less.
            // else equal
        }
};
```
- however, there is a problem with equality.
    - if all we care about is equality, then the spaceship operator may be doing too much.
    - for example, two vectors with different length are not equal.
    - we can verify this with O(1); no need to examine the contents.
    - to fix this, we give equality its own implementation apart from spaceship.
    - this would be field by field equality.
```cpp
class vector {
    ...
    public:
        bool operator==(const vector &other) const {
            if (n != other.n) return false;
            else return (*this <=> other) == 0;
        }
}
```
Note:
- if only operator `<=>` is defined, then all 6 relational operators will be rewritten in terms of `<=>`.
- however, if operator `==` is defined, then it will be used for `==` and `!=`.

