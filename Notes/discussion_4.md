
# Continuation of discussion on spaceship

But when might you not want a lexicographic ordering?

```
class Vector {
    size_t n, cap;
    int* theVector;
}
```
By the default lexicographic ordering will compare `n`, and then 'cap' and then
the memory location of `theVector`. Does this make sense?

We want lexicographic comparison on the items in the array. You can implement
spaceship yourself.
```
class Vector {
    ...
public:
    std::strong_ordering operator<=>(cinst Vector& other) const {
        // return the result of <=> on the first entry where
        // theVector and other differ
    }
}
```
BUT we have a problem. If all you care about is equality, spaceship is doing
too much work. We know right away that two vectors with different lengths are
not equal, thus there is no need to go through each item.

To solve this you can just implement your own equality:
```
class Vector {
    ...
public:
    ...
    bool operator==(const Vector& other) const {
        if (n != other.n) return false;
        return (*this <=> other) == 0;
    }
}
```

If operator <=> is defined, all 6 relational operators will be written in terms
of <=>. However, if the operator == is defined, it will be used in place for
equality and inequality.

# Structured Bindings (C++17)

Recall iterator pattern + range-based loops. Could we do something similar for
structures??

Could we iterate over a map? We want the entire key value pair (std::pair<Key,
Value>). This is the behavior we want:
```
map<string, int> m;
for (auto &p : m) {
    cout << p.first << ' ' << p.second << endl;
}
```

C++17 gives us
```
struct Posn {
  int x, y;  
};
Posn p{1,2};
auto [a, b] = p;
```

This also works for stack arrays
```
int a[] = {1, 2, 3};
auto [x, y, z] = a;
```

Note that the number of vars in the binding must match the number of items in
the struct/array. Also the variables that are unpacked are created by value -
changes do not affect original. If you wish to modify the original, you can use
a reference.

Using this, our map iteration looks like
```
map<string, int> m;
for (auto& [key, value] : m) {
    cout << key << ' ' << value << endl;
}
```

Also, note that to do structured bindings, ALL fields must be public, the
following will not compile:
```
struct Posn {
private:
    int x;
    int y;
}
```
and
```
struct Posn {
public:
    int x;
private:
    int y;
}
```
Now we wish to ask, what if we still want private fields, can we provide
customized structured bindings?

To be able to support this, we need to tell the compiler:
- number of items in the binding
- what are the types of each item
- what each item should return

To accomplish this we need template specialization. This is actually the only
time you are allowed to add something to the std namespace, specifically, when
specializing a template.

The relevant ones are:
```
std::tuple_size<T> - number of fields in type T
std::tuple_element<n, T> - the type of the nth element of T
member template get<n> - returns ref to the nth item
```

Implementation next discussion.
