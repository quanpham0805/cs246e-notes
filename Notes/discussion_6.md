
# Range Abstraction (C++20)

Consider the method `vector<T>::erase`, which takes an iterator to the item,
removes the item, and shuffles the following items forward. Finally, it returns
an iterator to the point of erasure. This has O(n) for shuffling.

BUT, what if i wanted to erase several consecutive items? Calling erase in a
loop is inefficient - we are shuffling mutliple times for no reason.

For this reason, most methods comes with a range version:
```
iterator vector<T>::erase(iterator first, iterator last);
```
this allows us to pay the linear shuffling cost only once.

Maybe theres potential for abstraction here.
Consider composing multiple methods on some input, for example, filter +
transform (take all odd numbers and square them)

Note: many functions in <algorithm> have an `_if` version that are conditional
on a predicate.

Using cpp we already know:
```
auto odd = [](int n) { return n%2 != 0; }
auto square = [](int n) { return n*n; }

vector<int> v{ ... };
vector<int> w, x;
copy_if(v.begin(), v.end(), w.begin(), odd);
transform(w.begin(), w.end(), x.begin(), square);
```

A couple problems with this approach:
- calls don't compose well: you need 2 function calls, and you can't chain them
- you need to create seperate objects to hold intermediate results

Notice that for both `copy_if` and `transform`, they need both where to start
and where to stop. This suggests that instead of returning iterators, we should
be returning ranges.

Let's also clarify by what we mean by 'range', let's say that a range is
anything that provides a `begin` and `end` method, just like vector is:
```
transform(copy_if(v , odd), square);
```

We still haven't solved the intermediate storage problem? but do we need it?

A range only needs to look like it's sitting ontop of a container, we can
instead have the range fetch items on demand (lazy evaluation).

These range objects are called views, they do on demand fetching an no
intermediate storage. Theses exist in C++20:
```
#include <ranges>

vector v{1,2,3,4,5,6,7,8,9,10};

auto x = std::ranges::views::transform(
    std::ranges::views::filter(v, odd), square
);
```

it gets better, filter and transform are calle drange adaptors, they don't hold
data, they attach themselves to data and operate on it. They take a second
form, you can just supply the function, not the range.
    ie filter(pred), transform(f)
you get back a callable object that is waiting for a range (curried)
    ie filter(pred)(R), transform(f)(R)

C++20 also defines operator| such that R|A is mapped to A(R), for example:
    B(A(R)) = B(R|A) = R|A|B

Thus, we can write:
```
auto x = v | filter(odd) | transform(square)
```

