[I want an even faster vector <<](./problem_27.md) | [**Home**](../README.md) | [>> Collecting Stats](./problem_29.md)

# Problem 28: I want to print the unprintable!
## **2021-11-23**

Recall [problem 7](Notes/../problem_7.md): 
- Wanted to print a `vector`
- `vector` used to hold int
- now is a template

Can do a templated `operator<<`
- Will work if `T` is printable
- won't compile otherwise

What we want now:
- print `T` if `T` is printable
- print a default msg (e.g, "unprintable") in case `T` is not printable

How might we do this?
- two overloads for the two different behaviors

```cpp
struct true_type {};
struct false_type {};

template <typename T> void doOutput(const T& x, true_type) {
    std::cout << x;
}

template <typename T> void doOutput(const T& x, false_type) {
    std::cout << "unprintable" << std::endl;
}
```

Top-level wrapper:

```cpp
template <typename T> void output (const T& x) {
    doOutput(x, typename has_output<T>::type{});
}
```
- Given `T`, should produce `true_type` if `T` supports output, false otherwise.
- How do we do this???

```cpp
template <typename T> struct has_output {
    using type = decltype(output_test<T>(0));
};
```
- Recall `decltype` only test the types of the expression, but not evaluating it (we don't want it to be evaluated anyway).
- Now we overload function `output_test` such that a version returning `false_type` is always available, but the version returning `true_type` is a better match, but is only available if `T` supports output.

```cpp
// no need any implementation since we won't even run it anyway
template <typename T> false_type output_test(...);

template <typename T, typename = decltype(cout << T())> true_type output_test(int);
```
- If `T` has output, then `typename` has type `ostream`, and otherwise it would be invalid (but still compile because SFINAE)
- Putting `(int)` will always guarantee that it is a better match.
- Ok well, what if `T` doesn't have a default ctor? That would be invalid even though `T` is printable (wrong result)?
- Hack: we don't actually need a `T` object to perform the cout anyway (because it would never need to be constructed), so what's the other way we could make a `T` object? A function that return `T`. Do we have one? Doesn't matter, we can pretend we do.

```cpp
template <typename T> T&& declval();
```
- use refs here so that we don't assume that `T` has a copy/move ctor
- Now we have

```cpp
template <typename T, typename = decltype (cout << declval<T>())> true_type output_test(int);
```

---
[I want an even faster vector <<](./problem_27.md) | [**Home**](../README.md) | [>> Collecting Stats](./problem_29.md)