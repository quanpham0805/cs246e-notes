[A fixed-size object allocator <<](./problem_37.md) | [**Home**](../README.md)

# Problem 38: I want a (tiny bit) smaller vector class
## **2021-12-02** 

Last class (but I'm watching this lec 3 weeks after end of school term :monkaS:)

**Currently:** `vector`/`vector_base` have an allocator field. Standard allocator is stateless (has no fields)

What is its size? 0? No - C++ does not allow 0 size types (messes things up, ex. pointer arithmetic).

Every type has at least size 1.

Practically - compiler adds a dummy `char` to the allocator.

So having an allocator field makes the `vector` larger by a byte. Probably more, due to alignment (e.g, 8 bytes)

Having an allocator field in the vector_base, may add another byte (or more due to alignment).

To save this space, C++ provides the **empty base optimization (EBO)**.

Under EBO, an empty base class does not have to occupy space in an object.

So we can eliminate the space cost of an allocator by making it a base class. At the same time, make `vector_base` a base class of a `vector` (also done in STL)

**Note:** From C++17 and onwards, this EBO applies to empty classes, so we don't incur any of these costs at all anymore (so this problem might cease to exist in the future and get replaced with something more interesting???)

```C++
template <typename T, typename Alloc = allocator<T>>
struct vector_base : private Alloc { // struct has default public inheritance
    size_t n, cap;
    T *v;
    
    // Since Alloc is a template arg, we don't know what it looks like
    // so if we want to use inherited members, we need using to make
    // them visible
    using Alloc::allocate;
    using Alloc::deallocate;
    // etc.

    vector_base(size_t n): n{0}, cap{n}, v{allocate(n)} {}
    ~vector_base() { deallocate(v); }
};

template <typename T, typename Alloc = allocator<T>>
class vector: vector_base<T, Alloc> {   // private inheritance - no is-a relation
    using vector_base<T, Alloc>::n;
    using vector_base<T, Alloc>::cap;
    using vector_base<T, Alloc>::v;
    
    using Alloc::allocate;   // or say this->allocate 
    using Alloc::deallocate; // this->deallocate
public:
    // ... Use n , cap, v instead of vb.n, vb.cap, vb.v
};
```

`uninitialized_copy`, etc. - need to call construct/destroy
- Simplest - let them take an allocator as a parameter

```C++
template <typename T, typename Alloc> {
    void uninitialized_fill(T *start, T *finish, const T& x, Alloc a) {
        // ...
        a.construct(...)
        // ...
        a.destroy(...)
        // ...
    }
}
```
How can vector pass an allocator to these functions?
```C++
// Cast yourself to base class reference
uninitialized_fill(v, v + n, x, static_cast<Alloc&>(*this));
```

**There is an issue though:**
- `vector` doesn't know that `Alloc` is its grandparent
    - `vector` (privately) inherits from `vector_base` and `vector_base` (privately) inherits from `Alloc`
    -  `Alloc` is a parent of `vector_base`, but the only one who knows that is `vector_base`, because the inheritance is private 
    -  `vector` knows that `vector_base` is its parent, but only `vector` knows that because private inheritance 
    -  Now, since `vector` is not `vector_base` because private inheritance, `vector` does not know that `Alloc` is `vector_base`'s parent and as a result it does not know `Alloc` is its grandparent
- So we are in a situation where the inheritance relationship is known to the child only but not the grandchildren. 

**Solution:**
- This is one of the only use cases of **protected inheritance**
- Kinda similar to private inheritance (fields become `protected` though), but the relationship is known to your children and yourself, so the children knows who their ancestors are 
  ``` cpp
  template <typename T, typename Alloc = allocator<T>>
  struct vector_base : protected Alloc {...};
  ```
- Now, is-a relationship is knowns to `vector_base` and its subclasses

Remaining details: exercise

## Conclusions: How should you view what you have learnt?

This was not a course on C++

This was not a course on subtype-based polymorphism

These were means to an end, but not the end

This was a course on **abstraction**. C++ and subtype polymorphism helped us accomplish that, but so did templates, exception safety, metaprogramming, etc.

Programming is full of chaos - higher abstractions are often slower

C++ provides mechanisms for creating high-level abstractions that perform well, because you have such precise control over implementation

This ability to operate on several levels of abstractions is not common among programming languages (snake language)

When you write libraries, you get to consider all these hard problems so that your clients can use your abstractions with as much ease as with any other languages

`vector`, `string`, `unique_ptr`, `auto` - make managing memory and navigating types as easy as in any garbage-collected, newer language

So, in conclusion: C++ is easy because it is hard!

---
[A fixed-size object allocator <<](./problem_37.md) | [**Home**](../README.md)
