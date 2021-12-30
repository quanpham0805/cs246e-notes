# Discussion 12: Concepts (C++20) - Ross's Lecture!

## **2021-11-24**

Template:
- Abstract generic code
	- No other alternatives for now (if we ignore the macros), other than copy pastes
i	- Incredibly powerful (Turing complete)
- C macros

Cons:
- Clunky (remember we need to write a tons of thing just for the copy constructor like delegating the constructor, or very long name like `enable_if_blablabla`))
	- Seems hacky
	- Even simple things like overloading are weird
- Valid types are based on the implementation, not the signature

Example:
```C++
template<typename T> void f(T x) {
    // ...
	x.foo();
    // ...
}
```
- Calling `f` on an object requires that `X` has a `foo` method
- Where does this method come from? Idk, it is buried in the implementation of `f` itself

Example:
```C++
#include <list>
#include <algorithm>

int main() {
	std::list<int> X{3, 100, -5};
	std::sort(X.begin(), X.end());
}
```
- If we compile this code it will a gazillion output nonsense of compile error. This is because
	- `sort` implicitly relies on the implementation because it tried to perform nonsense substitutions.

Go back to `iterator` implementation example to show new C++20 `concepts`, making it cleaner to compile
```C++
template <typename Iter>
requires std::is_same_v<
	typename iterator_traits<Iter>::iterator_category,
	random_access_iterator_tag
>
Iter advance (Iter it, int n) {
	return it += n;
}
```
- `requires ...` is a constraint
- Now the rewrite on the type Iter are in the substitution
- We can run it based on the constraints without needing helpers
- Still hasn't addressed clunkiness, we need to write a lot like `std::is_same_v` any time we want a function on iterators.

Now, we introduce `concepts` from C++20
```C++
template <typename Iter>
concept RandomAccessIterator = std::is_same_v<
	typename iterator_traits<Iter>::iterator_category,
	random_access_iterator_tag
>;

template <typename Iter>
requires RandomAccessIterator<Iter>
Iter advances(Iter it, int n) {
	return it += n;
}
```
One more improvement: 
```C++
template <RandomAccessIterator Iter>
Iter advance(Iter it, int n) {
	return it += n;
}
```
- Doing all this, we can hide library code and much shorter and cleaner too.

Question: Why don't we just get rid of this lengthy thing (C++ has been around for so long now)? Go all the way and delete?????
- Well, look at this example:
```C++
RandomAccessIterator advance(RandomAccessIterator it, int n);
void f(const Foo&);
void f(Foo&&); // rvalue ref or forward ref???
```
- Now, if `Foo` is a class then it is a rvalue reference, and if it is a concept, it would be a forwarding reference.
- We can do something like this
```C++
RandomAccessIterator auto advance(RandomAccessIterator auto it, int n);
```
- Now we know about `concepts`, but `concepts` are weak.
Example: 
```C++
struct Mistake {
	using iterator_category = random_access_iterator_tag;
	
	void operator*();
    void operator!=();
    void operator++();
};
```
- What happens if we call advance with mistake?
	- Calling `advance` with `Mistake` would fail in the substitution step, but it would pass constraints check!
- Solution? Refer our concept to include more of what an iterator is about
```C++
template <typename It>
concept RandomAccessIterator = std::is_same_v<
	typename iterator_traits<Iter>::iterator_category,
	random_access_iterator_tag
>&& requires (Iter it, Iter other, int n) {
	*it;
	{ it != other } -> std::same_as<bool>;
	{ it ++ } -> std::same_as<Iter>;
	{ it += n } -> std::same_as<Iter>; 
}
```
- Now, trying to call advance on `Mistake` fails at the constraint check, rather than during substitution.
	- This is good because it gives you more info like "this specific line fails" rather than give you a bunch of garbage

Important lesson: `concepts` can only help you check for **syntax** but not **semantics**..

Consider a `concept` - Dateable - requires the struct has a `date` method 
```C++
template <typename T>
concept Dateable = requires(T x) {
	X.date();
}

struct Calendar {
	// ...
	date(); // return current day of calendar
}

struct User {
    // .., 
	date(); // takes someone out for a nice dinner
}
```
- These `date` methods mean different things, say, for an app like Tinder
- Hard to think of template function that should work on both
- So essentially we are only checking the syntax, but semantics mean different things that we cannot control with it

One last thing, remember we were complaining about delegating constructors

POD: Trivially constructable and standard layout. 
- Well they are removing `std::is_pod`, but it's ok because we know how to construct pod (recall [problem 27](Notes/problem_27.md))
```C++
template <typename T> 
concept POD = std::is_trivial_v<T> && std::is_standard_layout_v<T>

template <typename T> class vector {
	// ...
	struct dummy;
	vector(const vector& other): vector{other, dummy{}} {}

    // memcpy
	template <typename X = T>
	requires(POD<X>)
	vector(const vector& other, dummy) {...}
	
    // normal
	template<typename X = T>
	requires(!POD<X>)
	vector(const vector& other, dummy) {...}
}
```