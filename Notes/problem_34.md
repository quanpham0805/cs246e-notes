[Generalize the Visitor Pattern! Part 2! <<](./problem_33.md) | [**Home**](../README.md) | [>> Total Control](./problem_35.md) 
# Problem 34: Policies
## **2021-11-25**

Recall - [problem 10](Notes/problem_10.md) talked about how we can make accessing array safe for `vector`.
- We offered them a choice, either using `[]` (unchecked) or using `.at()` for bound checking.
- What if the user wants to check access using `[]`?
- Maybe we could make the choice of checked/unchecked once, when the vector is created, rather than every method call.
- Here comes CRTP!

CRTP - 2 potential superclasses of vector, each with a different implementation of operator `[]`.
- These are called **policy classes**.
```cpp
template <typename T, typename V>
class Unchecked {
    T& operator[](size_t n) {
        return static_cast<V*>(this)->theVector[n];
    }
}

template <typename T, typename V>
class Checked {
    T& operator[](size_t n) {
        V* sub = static_cast<V*>(this);
        if (n >= sub->n) throw std::out_of_range("out of range");
        return sub->theVector[n];
    }
}

template <typename T, template <typename, typename> Policy>
class vector : public Policy<T, vector<T, Policy>> {
    size_t n, cap;
    T* theVector;
    friend class Policy<T, vector<T, Policy>>;
public:
    // ...
}
```
- `vector` is parameterized by an uninstantiated template `Policy`
- Also knowns as **template template parameter** 
```cpp
int main() {
    vector<int, Unchecked> v{1, 2, 3};
    vector<int, Checked> w{4, 5, 6};
    v[3]; // UB
    w[3]; // throws
}
```


---
[Generalize the Visitor Pattern! Part 2! <<](./problem_33.md) | [**Home**](../README.md) | [>> Total Control](./problem_35.md) 
