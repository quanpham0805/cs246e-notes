# Discussion 6: Range Abstraction (C++20)

Consider the method `vector<T>::erase`:
- takes an iterator to the item;
- removes the item;
- shuffles the following items forward; and
- returns an iterator to the point of erasure.

The cost of `erase` is `O(n)` (for shuffling) - this is fine.

But, what if we need to erase consecutive items?
- we could call `erase` repeatedly;
    - but this is `O(n*m)`, where m is the number of items we are taking out - this is not good!
- a faster method is to shuffle the items up `k` positions in one step each, where `k` is the number of items being erased.
- for this reason, most methods come with a *range* version, which looks something like this:
```cpp
iterator vector<T>::erase(iterator first, iterator next)
```
- which erase items in `[first, last)`, and only pays the linear cost once.
- this already exists in C++14.

We saw this in lectures;
- `transform` took two iterators, specifying a range `[start, finish)` on the input
- maybe there's something to this idea - maybe a new kind of abstraction!

Consider: **composing multiple methods on some input**.
- eg `filter` + `transform`
- say, take all the odd numbers and square them (and discard the even numbers).
- note: many functions in `<algorithm>` have an `_if` version
    - eg `replace_if`, that takes a predicate as an additional parameter, and are conditional on said predicate.
    - however, `transform` is not one such function.

Let's pick out the odd numbers and square them.
```cpp
vector<int> v {...}; // using init list constructor
vector<int> w,x;

auto odd = [](int i){return i % 2 == 1;});
auto square = [](int i){return i * i;});

// pick out odd numbers (ie discard even numbers)
copy_if(v.begin(), v.end(), w.begin(), odd); 
// square all remaining numbers (ie all the odd numbers)
transform(w.begin(), w.end(), x.begin(), square);
```
There are two problems with this solution:
1. The calls don't compose well; we need two separate function calls, and there's no way to chain them.
2. We needed to create a vector for intermediate storage (ie vector `w`).

`transform` and `copy_if` return an iterator to the beginning of the output range.
- what if instead we got a *pair* of iterators to the beginning and end of the output array?
- and what if instead, `copy_if` and `transform` took a range, rather than a pair of iterators?
- if this were the case, we could write something like
```cpp
transform(copy_if(v, odd), square);
```
- a *range* is anything with `begin()` and `end()` producing iterator types.

So now, the functions are composable.
- but what about intermediate storage?
- who says we need any?
- a range only needs to *look* like it's sitting on top of a container.
- so, instead, have the range fetch items *on demand*.

We can write pseudocode for `filter` and `transform`:

`filter`:
- on fetch:
    - iterate through the range until we find an item that satisfies the predicate;
    - then return it.

`transform`:
- on fetch:
    - fetch an item `x` from the range below;
    - then return `f(x)`.

These range objects are called **views**.
- they do on-demand fetching, with no intermediate storage.
- these exist in the C++20 standard library.
```cpp
#include <ranges>

vector v {1,2,3,4,5,6,7,8,9,10};
// compiler figures out that vector is of ints, so we don't need to explicitly specify vector's type.
auto x = std::ranges::views::transform(
    std::ranges::views::filter(v, odd),
    square
);
```

But it gets better!
- `filter` and `transform` are called **range adaptors**.
- they take a second form:
```cpp
filter(pred), transform(f)
```
- ie just supply the function, not the range.
- these become callable objects, parameterized by a *range*.
- so, we can say something like
```cpp
filter(pred)(R);
transform(f)(R);
```
Even better, C++ defined the following operator `|` such that `R|A` := `A(R)`.
- so, `B(A(R)) = B(R|A) = R|A|B`, and so we can write something like
```cpp
auto x = v | filter(odd) | transform(square);
```
- this looks just like a Linux pipeline!

