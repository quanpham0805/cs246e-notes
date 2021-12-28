# Discussion 10: fold expression (C++17)

## 2021/11/17

Recall: variadic templates
- We used them a few times already, for example, `emplace_back`, `make_unique`
- They built the objects, then copy to the container so that we don't need to actually construct the object
- Also used these to make functions that took an arbitrary number of arguments 
- Example: variadic sum: `sum(1, 2, 3, 4, 5) = 15`. 

```C++
template <typename... Args> auto sum (Args... args) {
    return 0;
} 

// only trigger when list is not empty
// and only when we can define the first element
template <typename First, typename... Rest>
auto sum (First x, Rest... r) {
    return x + sum(r...);
}
```

- This is just like Racket, but it doesn't feel "simple", and feels more like a trick rather than something built in, not something the inventor has thought of
- Most of the things that template offer, people didn't know about it, and it's more powerful than we thought, so they are making it more simple to use in C++17


In <u>C++17</u>, we have `fold` expressions make these functions easier to write

```C++
template <typename... Args> auto sum (Args... args) {
	return (args + ... + 0);
	// or we can just do
	// return (args + ...);
	// these are called "fold" expressions
}
```

For ref: [CPP Reference fold expression](https://en.cppreference.com/w/cpp/language/fold)

Syntax: General forms of a fold expression:
- `(... op e) = ((e1 op e2) op e3) ... op en`
- `(e op ...) = e1 op (e2 op (... ( en-1 op en)))`
- `(i op ... op e) = ((i op e1) op e2) ... op en)`
- `(e op ... op e) = e1 op (e2 op (... (en op i)))`

Application: `multi_push_back`
```C++
template <typename T, typename... Args>
void multi_push_back(vector<T> &v, Args... objs) {
	(v.push_back(objs), ...); // op is the comma operator
}

```
- This will be compiled to tail recursive functions




