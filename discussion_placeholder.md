
```
        _ _                        _               _ _
     __| (_)___  ___ _   _ ___ ___(_) ___  _ __   / / |
    / _` | / __|/ __| | | / __/ __| |/ _ \| '_ \  | | |
   | (_| | \__ \ (__| |_| \__ \__ \ | (_) | | | | | | |
    \__,_|_|___/\___|\__,_|___/___/_|\___/|_| |_| |_|_|
 
 ```

# Implementing std::variant

```cpp
template<tynemae.. Types> union variant_base {};

template<typename First, typename... Rest>
union variant_base {
    First first;
    variant_base<Rest...> rest;
};
```

we also need a way to tell what is being stored, as well as a way to fetch data
```cpp
template<typename... Types> class variant {
    variant_base<Types...> u;
    size_t index;
}
```

we also need `get<N>`, a way to get the nth item of the variant. to do this
we need to find the type of the nth item.
```cpp
template<typename N, typename... Types>
constexpr variant_alternative_t<N, variant<Types...>>&
get(variant<Types...>& v);
```

Notice something interesting: when we call get, we say:
```cpp
get<N>(v);
```
but get is templated over `N` and `variant<types...>`. in this case `N` is not
deducable, but the variant type is, so we can actually leave out the variant
type - partial type deduction (only rule is non deducable templates come
first).

now to write `variant_alternative_t<N, variant<Types...>>`:
```cpp
template<size_t N, typename variant> struct variant_alternative;

template<size_t N, typename First, typename... Rest>
struct variant_alternative<N, variant<First, Rest...>>
    : variant_alternative<N-1, variant<Rest...>> { }

template<typename First, typename... Rest>
struct variant_alternative<0, variant<First, Rest...>> {
    using type = First;
};

template<size_t N, typename variant>
using variant_alternative_t = typename variant_alternative<N, variant>::type
```

and now to write get
```cpp
template<size_t N, typename... Types> constexpr variant_alternative_t<N, variant<Types...>>&
get(variant<Types...>& v)
{
    if (v.index() != N) throw ...;
    return _get<N>(v.u);
}

template<size_t N, typename... Types> constexpr variant_alternative_t<N, variant<Types...>>&
_get(var<Types...>& u) { }

template<size_t N, typename First, typename... Rest> constexpr variant_alternative_t<N, variant<First, Rest...>>&
_get(variant_base<First, Rest...>& u)
{
    return _get<N-1>(u.rest);
}
template<typename... Types> constexpr variant_alternative_t<0, variant<Types...>>&
_get(variant_base<Types...>& u)
{
    return u.first;
}
```

# constexpr if (C++17)

recall the iterator advance that we wrote in lectures.
```cpp
template<Types Iter> Iter
advance(Iter it, int n)
{
    if (typeid (typename iterator_traits<Iter>::iterator_category) == typeid (random_access_iterator_tag))
        return it += n;
    else ...
}
```
in the case that we don't have a `random_access_iterator_tag`, we will know for
sure that we will run the else block, however, the if block will still be
compiled - and our iterator doesn't have a += operator. to fix this, we moved
the logic to compile time.

c++17 gives us a way of addressing this issue, constexpr if.
```cpp
if constexpr(cond) {
    A
} else {
    B
}
```
this tells cpp that the condition must be evaluated at compile time. the
failing branch of the conditional will be thrown away.

so we can instead write
```cpp
template<Types Iter> Iter
advance(Iter it, int n)
{
    if constexpr(typeid (typename iterator_traits<Iter>::iterator_category) == typeid (random_access_iterator_tag))
        return it += n;
    else ...
}
```
this works now!
