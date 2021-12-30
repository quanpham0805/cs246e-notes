# Discussion 13: constexpr if (C++17)

Recall the iterator advance that we wrote in lectures.
```cpp
template<Types Iter> Iter advance(Iter it, int n) {
    if (typeid(typename iterator_traits<Iter>::iterator_category) == typeid(random_access_iterator_tag))
        return it += n;
    else ...
}
```
- In the case that we don't have a `random_access_iterator_tag`, we will know for
sure that we will run the else block, however, the if block will still be
compiled - and our iterator doesn't have a += operator 
- To fix this, we will move the logic to compile time
- C++17 gives us a way of addressing this issue, constexpr if.
```cpp
if constexpr(cond) {
    A
} else {
    B
}
```
- This tells cpp that the condition must be evaluated at compile time
- The failing branch of the conditional will be thrown away
-  So we can instead write
```cpp
template<Types Iter> Iter advance(Iter it, int n) {
    if constexpr(typeid(typename iterator_traits<Iter>::iterator_category) == typeid (random_access_iterator_tag))
        return it += n;
    else ...
}
```
- This works now!
