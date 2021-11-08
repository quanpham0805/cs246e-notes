[Is vector exception safe? << ](./problem_16.md) | [**Home**](../README.md) | [>> Abstraction over containers?](./problem_18.md) 

# Problem 17: Insert/Remove in the Middle
## **2021-10-21**

A method like `vector<T>::insert(size_t i, const T &x)` is easy to write.  
But for the same `list<T>` requires an upfront traversal.

Using iterators can be good for both.

```C++
template<typename T> class vector {
    ...
    public:
        iterator insert(iterator posn, const T&x) {
            increaseCap();
            ptrdiff_t offset = posn - begin(); // ptrdiff_t incase result is negative (in general)
            iterator newPosn = begin() + offset;
            new(static_cast<void*>(end()) T(std::move_if_noexcept(*(end() - 1)));
            ++vb.n;
            for (iterator it = end() - 1; it != Posn; --it) {
                *it = std::move_if_noexcept(*(it - 1));
            }
            *newPosn = x;
            return newPosn;
        }
    }
};
```

Exception safe? Assuming `T`'s copy/move operations are exception safe (at least basic guarantee), insert offers the basic guarantee.
- May get a partially shuffled vector, but it will be a valid vector.

**Note:** if you have other iterators pointing at the vector

```
+---+---+---+---+---+  
| 1 | 2 | 3 | 4 |...|  
+---+---+---+---+---+  
  ^       ^   ^    
 it1      h  it2  

and you insert at h  

+---+---+---+---+---+---+  
| 1 | 2 | 5 | 3 | 4 |...|  
+---+---+---+---+---+---+  
  ^       ^   ^    
 it1      h  it2  
```

it2 will now point at a different item.

**Convention:** after a call to insert or erase, all iterators pointing after the point of insertion/erasure are considered invalid and should not be used.
- Also, if a reallocation happens, _all_ iterators pointing into the vector become invalid.

Exercises: 
- **erase** - remove the item pointer to by an iterator, return an iterator to the point of erasure
- **emplace** - like insert but takes constructor args

BUT - that means there is a problem with `push_back`. If `increaseCap` successfully reallocates and placement new (constructor) throws, there vector is the same but the iterators were invalidated! (and we claimed that this method offer a strong guarantee, not so strong now isn't it???)

This is really tricky. A general rule of thumb (in programming only, not during exams) is to deal with the hard stuffs first.
- General technique for exception safety: Do the unsafe things first.
  - If it fails, you have not done anything yet
  - If it succeeds, you are home free :D

So let's write the new item before we transfer the old ones.
```C++
template<typename T> class vector {
    // ...
public:
    void push_back(const T& x) {
        // we will increase size by hand rather than relying on our existing method
        if (vb.n == vb.cap) {
            vector_base<T> vb2{2 * cap};
            new (vb2.v + vb.n) T(x); // if this fails, then vb2 is clean up automatically, which is good
            vb2.n = vb.n + 1;
            try {
                uninitialized_copy_or_move(vb.v, vb.v + vb.n, vb2.v);
                clear();
                std::swap (vb, vb2);
            } catch (...) {
                // if uninit... fails, we still have the new vb2 array, everything else was clean up, and so the only thing left we need to clean up now is the T(x) item we just put in
                (vb2.v + vb2.n - 1)->~T();
                throw;
            }
        } else {
            new (vb.v + vb.n) T(x);
            ++ vb.n; // we don't want to do this postfix, that would be too soon because new (addr) T(x) might fail, and we don't want to increase n if it fails 
        }
    }
}
```
- This offers strong guarantee
- Exercise: go through this again and convince yourself that this is strong guarantee

---
[Is vector exception safe? << ](./problem_16.md) | [**Home**](../README.md) | [>> Abstraction over containers?](./problem_18.md)
