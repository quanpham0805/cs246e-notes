# Discussion Topic #12: Ross's Lecture! - Concepts (C++20)

Recall templates:
1. Allows us to abstract generic code;
2. It is also Turing complete (so incredibly powerful).

But,
1. They are clunky!
2. Even simple things like overloading are a chore.
3. Valid types are based on the implementation, not the signature.

For example,
```cpp
template <typename T> void f(T x) {
    ...
    x.foo();
    ...
}
```
Calling `f` on an object requires that `X` has a `foo` method.
- but this is buried in the implementation of `f` itself!
```cpp
#include <list>
#include <algorithm>

int main() {
    std::list<int> x {3, 100, -5};
    std::sort(x.begin(), x.end());
}
```
But this has a long list of errors!

The interface implicitly relies on the implementation.

How can we provide the compiler with more information with what is actually going on?
- let's go back to the `iterator` example to show new C++20 concepts, making it clearer to write.

```cpp
template <typename Iter> requires std::is_same_v<typename iterator_traits<Iter>::iterator_category, random_access_iterator_tag>
Iter advance(Iter it, int n) return it += n;
```
`requires ...` is a constraint.

Now, the requirements of the type `Iter` are in the signature.

We can now overload based on the constraints without needing helpers.

But we still haven't addressed the clunkiness - we need `requires std::is_same_v` any time we want a function on iterators.

Introducing **concepts** - which are collections of constraints.

```cpp
template <typename Iter> template RandomAccessIterator = std::is_same_v<typename iterator_traits<Iter>::iterator_category>, random_access_iterator_tag>;

template <typename Iter> requires RandomAccessIterator<Iter> Iter advance(Iter it, int n) return it += n;
```
One more improvement:
```cpp
template <RandomAccessIterator Iter> Iter advance(Iter it, int n) {
    return it += n;
}
```

Recall:
```cpp
RandomAccessIterator advance(RandomAccessIterator it, int n);

void f(const Foo&);
void f(Foo&&);
// if Foo is a class, then its an lvalue ref
// if Foo is a concept, then its a forwarding ref
```
Instead, we would need to write
```cpp
RandomAccessIterator auto advance(RandomAccessIterator auto it, int n);
```
Now, we know about concepts, but our concepts are weak.

For example:
```cpp
struct Mistake {
    using iterator_category = random_access_iterator_tag;
    void operator*();
    void operator!=();
    void operator++();
};
```
What happens if we call `advance` on `Mistake`?

Calling `advance` with `Mistake` would fail in the substitution step, but it would pass the constraints check. This is a problem!

Solution: refine our concept to include more of what an iterator is about.
```cpp
template <typename Iter> concept RandomAccessIterator = std::is_same_v<typename iterator_traits<Iter>::iterator_category, random_access_iterator_tag> &&requires(Iter it, Iter other, int n) {
    *it;
}
// it != other => std::same_as<bool>;
// it++ => same_as<Iter>;
// it += n => same_as<Iter>;
```
Now, trying to call `advance` on `Mistake` fails at the constraint check, rather than during substitution.

Important lesson: concepts can only check or syntax, but they should model semantics.

Consider a concept `Dateable`, which just requires that the object has a `date` method.
```cpp
template <typename T>
concept Dateable = requires(T x) {
    x.date();
}
struct Calendar {
    ...
    date(); // returns current day of calendar
};

struct User {
    ...
    date(); // takes someone out for a nice dinner
};
```
- but these `date()` methods mean very different things, say for an app like *Tinder*?
- hard to think of template functions that should work on both.

Recall: POD refers to trivial data which has a standard layout.
```cpp
template <typename T>
concept POD = std::is_trivial_v<T> &&std::is_standard_layout_v<T>;

template <typename T> class vector {
    ...
    struct dummy;
    vector(const vector &other): vector{other, dummy {}} {}

    template<typename X=T> requires POD<X> vector(const vector &other, dummy) {
        // memcpy version
    }

    template <typename X=T> requires (!POD<X>) vector(const vector &other, dummy) {
        // non-memcpy version
    }

}

