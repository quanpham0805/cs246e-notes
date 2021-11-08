[Insert/Remove in the middle <<](./problem_17.md) | [**Home**](../README.md) | [>> Heterogeneous Data](./problem_19.md)

# Problem 18 - Abstraction over Containers
## **2021-10-21**

Recall: `map` from Racket

```scheme
(map f (list a b c)) -> (list (f a) (f b) (f c))
```

May want to do the same with vectors:

```
+---+---+---+---+     +----+----+----+----+
| a | b | c | d | ->  |f(a)|f(b)|f(c)|f(d)|
+---+---+---+---+     +----+----+----+----+

      source                  target
```

Assume target has enough space to hold as much of source as we want to send.

```C++
template<typename T1, typename T2>

void transform(const vector<T1> &source, vector<T2> &target, T2 (*f)(T1)) {
    // reminds you of trampoline :)
    auto it = target.begin();
    for (auto &x: source) {
        *it = f(x);
        ++it;
    }
}
```
This is OK, but:
- What if we want only part of the source?
- What if we want to send the source to the middle of the target, not the beginning?

More general: use iterators

```C++
// This does not compile yet for C++14, but it would in C++20
// For now just assume it compiles and just work on how it would work out
template <typename T1, typename T2> void transform(vector<T1>::iterator start, 
    vector<T1>::iterator finish, vector <T2>::iterator target, T2 (*f)(T1)) {
    while (start != finish) {
        *target = f(*start);
        ++start;
        ++target;
    }
}
```

Ok, but:
- What if I want to transform a list, I'll write the same code again
- What if I want to transform a list to a vector, or vice versa

Solution: Make the types stand for the iterators, not the container elements. But then how do we indicate the type for `f`?

```C++
template<typename InIter, typename OutIter, typename Fn>
void transform(InIter start, InIter finish, OutIter target, Fn f) {
    while (start != finish) {
        *target = f(*start);
        ++start;
        ++target;
    }
}
```
- Works over vector iterators, list iterators, or any kinds of iterators.
- And now, it actually compiles.

InIter/OutIter can be any types that support `++`, `*`, `!=`, including ordinary raw pointers.

C++ will instantiate a template function with any type that has the operations being used by the function.

`Fn` can be any type that supports **function application**.

```C++
class Plus {
        int n;
    public:
        Plus(int n): n{n} {}
        int operator() (int m) { return n + m; } // function application operator
};

Plus p{5};

std::cout << p(7);  // 12
```
- Here is an object that is acting like a function, we call them **function object**
- Many people would call them *functor*, but Brad tends to avoid calling them like that, because that word has too many meanings in math and cs already, we don't want to overload it, it doesn't need another one

But more interesting, we can do something like

```C++
transform(v.begin(), v.end(), w.begin(), Plus{1});
```
- This is cheap and quick and dirty of getting things done
  
Now we have an arbitrary plus one function for any types


<u>Note</u>: 
- One of the advantages of writing function objects instead of functions and then having classes that are callable, is that you can do things more easily with classes than functions, in particular, unlike functions, classes maintain **states**. 
- The other way you can really maintain states in a function between multiple function calls is with a static variable, which is really tricky to use.
- Function objects **maintain** states. They are really powerful things.

OR we can do something like this in case you don't like objects:
```C++
transform(v.begin(), v.end(), w.begin(), [](int n) { return n + 1 });
                                         // ^ "lambda"
```
Lambda anatomy:
```
            param list
               |
lambda: [...](...) mutable? noexcept? { ... }
          |                              |
        capture list                    body
```
Semantics: 
```C++
void f(T1 a, T2 b) {
    [a, &b](int x) { body }
}
```
In the C++ context, lambdas are really just objects. This would be translated to:
```C++
void f (T1 a, T2 b) {
    class ??? {     // ??? - anonymous class we can't access the name, would be a random thing compiler made up
            T1 a; // any variables external to the lambda that you want to access here, list them in the capturing list
            T2 &b;
        public:
            ???(T1 a, T2 &b): a{a}, b{b} {}
            auto operator()(int x) const {
                body;
            }
    }
};

// Invocation would look like this
???{a, b}.operator() (...);
```

If the lambda is declared mutable, then `operator()` is not const.
- Capture list - provides access to selected vars in the enclosing scope.

---
[Insert/Remove in the middle <<](./problem_17.md) | [**Home**](../README.md) | [>> Heterogeneous Data](./problem_19.md)
