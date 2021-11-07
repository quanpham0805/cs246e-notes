[Memory management is hard <<](./problem_15.md) | [**Home**](../README.md) | [>> Insert/Remove in the middle](./problem_17.md)

# Problem 16: Is vector exception safe?
## **2021-10-19**

Consider:

```C++
template<typename T> class vector {
    size_t n, cap;
    T *theVector;
public:
    vector(size_t n, const T &x): 
        n{n}, cap{n}, 
        theVector{static_cast<T*>(operator new(n * sizeof(T)))} {
            for (size_t i = 0; i < n; ++i)
                // copy constructor for T, could throw - then what?
                new(theVector + i) T(x);
        }
};
```
- `malloc` vs `operator new`: `malloc` returns a `nullptr` if it fails, `operator new` fails it would throw an exception, but then it means we wouldn't have done anything, nothing has been allocated (strong guarantee), no problem.
- Other place we can get an exception is in the copy ctor of `T`.
- Partially constructed vector - destructor will not run
    - Broken invariant, does not contain `n` valid objects

**Fix:**
```C++
template<typename T> class vector {
    size_t n, cap;
    T *theVector;
public:
    vector(size_t n, const T &x): 
        n{n}, cap{n}, 
        theVector{static_cast<T*>(operator new(n * sizeof(T)))} {
            size_t progress = 0;
            try {
                for (size_t i = 0; i < n; ++i) {
                    new(theVector + i) T(x);
                    ++progress;
                }
            } catch(...) {  // ... supresses all type checking (accept whatever)
                for (size_t i = 0; i < progress; ++i) theVector[i].~T();
                operator delete(theVector);
                throw;  // rethrow the exception, we don't need to refer to the exception to throw it
            }
        }
};
``` 
- `...` in the `catch` block is literally `...` and not Brad wants us to fill in the blank. In languages like Java, there is a base class for all errors, but this is not the case for C++ as it can throw literally anything. One way we can *possibly* catch all is to catch `std::exception`, but again not everything follows this.
- Be careful about throwing `std::string`, because it actually throws `char*` and not `string`.


Abstract the filling part into its own function (since the function is quite big now):
```C++
template<typename T> void uninitialized_fill(T *start, T *finish, const T &x) {
    T *p;
    try {
        for (p = start; p != finish; ++p) {
            new(static_cast<void *>(p)) T(x);
        }
    } catch(...) {
        for (T *q = start; q != p; ++q) q->~T();
        throw;
    }
}
```

This is an all or nothing function (strong guarantee).

```C++
template<typename T> class vector {
        size_t n, cap;
        T *theVector;
    public:
        vector(size_t n, const T &x): n{n}, cap{n}, theVector{static_cast<T*>(operator new(n * sizeof(T)))} {
            size_t progress = 0;
            try {
                uninitialized_fill(theVector, theVector + n; x);
            } catch (...) {
                operator delete(theVector);
                throw;
            }
        }
};
```
- Vector is now responsible for 2 things: Creation and destruction of the array, and managing the content of the array.

Can clean this up using RAII on the array:

```C++
// Just to manage the array, nothing to do with the content, just the array itself
template<typename T> struct vector_base {
    size_t n, cap;
    T *v;
    vector_base(size_t n): 
        n{n}, cap{n == 0 ? 1 : n}, v{static_cast<T*>(operator new(cap * sizeof(T)))} {}
    ~vector_base() { operator delete(v); }
};

template<typename T> class vector {
        vector_base<T> vb;  // Cleaned up implicitly when vector is destroyed
    public:
        vector(size_t n, const T &x): vb{n} {
            uninitialized_fill(vb.v, vb.v + vb.n, x); // strong-guarantee
        }
        ~vector() {
            // or clear(), destruct all items
            destroy_elements(); 
        }
};
```  
- We have made the filling ctor exception safe.

### **Copy Constructor:**
```C++
template<typename T> vector<T>::vector(const vector &other): vb{other.size()} {
    uninitialized_copy(other.begin(), other.end(), vb.v);   // Similar to uninitialized_fill, details, exercise
}
```

Assignment: copy & swap is exception-safe because swap is no-throw

Pushback:
```C++
void push_back(const T& x) {
    increaseCap();
    new(vb.v + vb.n) T{x}; // If this throws, have the same vector
    ++ vb.n; // Don't increment n before you know the construction succeded
}
``` 

What about `increaseCap`? 

```C++
void increaseCap() {
    if (vb.n == vb.cap) {
        vector_base vb2 {2 * vb.cap};   // RAII, anything goes wrong we're good
        uninitialized_copy(vb.v, vb.v + vb.n, vb2.v);   // Strong guarantee
        destroy_elements(); // clear();
        std::swap(vb, vb2); // might be no throw
    }
}
```

**Note:** only `try` blocks are in `uninitialized_copy` and `uninitialized_fill`.

But we have an efficiency issue - copying from old array to the new one knowing that the old array is going to be destroyed. Moving would be better.
- But moving destroys the old array, so if an exception is thrown during moving, our vector is destroyed
- Therefore we can only move if we are sure that the move operation is nothrow.

```C++
void increaseCap() {
    if (vb.n == vb.cap) {
        vector_base vb2 {2 * vb.cap};
        uninitialized_copy(vb.v, vb.v + vb.n, vb2.v);   
        destroy_elements();
        std::swap(vb, vb2); 
    }
}

template<typename T>
void uninitialized_copy_or_move(T *start, T *finish, T *target) {
    T *p;
    try {
        for (p = start; p != finish; ++p, ++target) {
            new(static_cast<void*>(target)) T{std::move_if_noexcept(*p)};
        }
    } catch(...) {
        while (p != start) (-- p)->~T();
    }
}
```

`std::move_if_noexcept(x)` produces `std::move(x)` if `x` has a non-throwing move constructor, produces `x` otherwise.

But how should the compiler know if `T`'s move constructor is non-throwing? You have to tell it:

```C++
class C {
    public:
        C(C &&other) noexcept;
        ...
}
```

In general: moves and swaps should be non-throwing. Declare them so - will allow more optimized code to run.

Any function you are sure will never throw or propogate an exception, you should declare `noexcept`.

**Q:** Is `std:swap` `noexcept`?

```C++
template<typename T> void swap(T &a, T &b) {
    T c (std::move(a))
    a = std::move(b);
    b = std::move(c);
}
```

**Answer:** Only if T has a `noexcept` move constructor and a `noexcept` move assignment. How do we specify this?

```C++
template<typname T> void swap(T &a, T &b) 
    noexcept(std::is_nothrow_move_constructible<T>::value &&
             std::is_nothrow_move_assignable<T>::value) {
    ...
}
``` 
**Note:** `noexcept` = `noexcept(true)`

---
[Memory management is hard <<](./problem_15.md) | [**Home**](../README.md) | [>> Insert/Remove in the middle](./problem_17.md)