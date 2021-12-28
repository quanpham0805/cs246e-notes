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

But we need to have more experience with template meta programming, so we will push this to the week after next week.

