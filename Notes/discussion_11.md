# Discussion 11: std::variant revisited

## 2021/11/17

First, we need to spend some time on template meta programming

If you think about it, C++ templates is too powerful that it forms a functional language that operates at the level of program syntax (and surprisingly, it is turing complete)

We saw our first example with iterator advances

We can have a cleaner example:
```C++
template <int n> struct Fact {
	static const int result = n * Fact(n - 1)::result;
}

template <> struct Fact <0> {
	static const int result = 1;
}

int main() {
	int x = Fact<5>:: result; // 120
}
```

You already paid for everything during compile time, as `static` force all `int` to be evaluated at compile time, so `x` is computed at compile time

Now, how might we go about implementing `std::variant`

```C++
template <typename... Types> union variant_base {};

// then we use the same trick for variadic sum

template <typename First, typename... Rest> 
union variant_base {
	First first;
	variant_base<Rest...> rest;
};
```
- This is a gigantic nesting of variant, the actual type is only either `First`, or the `First` of `rest`, or ...

But we still need:
- A discriminator
- fetching operations

First, discriminator is easier:
```C++
template <typename... Types> class variant {
	variant_base<Types...> u;
	size_t index;
}

```
For fetching, we need `get<N>`, a way to get the nth item of the variant. To do this, we need to find the type of the nth item.
```cpp
template<typename N, typename... Types>
constexpr variant_alternative_t <N, variant<Types...>>& get(variant<Types...>& v);
```

Notice something interesting: when we call get, we say:
```cpp
get<N>(v);
```
But `get` is templated over `N` and `variant<types...>`. in this case `N` is not
deducible, but the variant type is, so we can actually leave out the variant
type - partial type deduction (only rule is non-deducible templates come
first).

Now to write `variant_alternative_t<N, variant<Types...>>`:
```cpp
template<size_t N, typename variant> struct variant_alternative;

// base case
template<typename First, typename... Rest>
struct variant_alternative<0, variant<First, Rest...>> {
    using type = First;
};

// recursion
template<size_t N, typename First, typename... Rest>
struct variant_alternative<N, variant<First, Rest...>> : variant_alternative<N - 1, variant<Rest...>> { };

template<size_t N, typename variant>
using variant_alternative_t = typename variant_alternative<N, variant>::type
```
and now to write `get`
```cpp
template<size_t N, typename... Types> constexpr variant_alternative_t<N, variant<Types...>>&
_get(var<Types...>& u) { }

// base case
template<typename... Types> constexpr variant_alternative_t<0, variant<Types...>>&
_get(variant_base<Types...>& u)
{
    return u.first;
}

// recursion
template<size_t N, typename First, typename... Rest> constexpr variant_alternative_t<N, variant<First, Rest...>>&
_get(variant_base<First, Rest...>& u)
{
    return _get<N - 1>(u.rest);
}

template<size_t N, typename... Types> constexpr variant_alternative_t<N, variant<Types...>>& get(variant<Types...>& v) {
    if (v.index() != N) throw ...;
    return _get<N>(v.u);
}
```
