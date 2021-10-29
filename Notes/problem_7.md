[Moves << ](./problem_6.md) | [**Home**](../README.md) | [>> Tampering](./problem_8.md) 

# Problem 7: I want a constant vector
## **2021-09-23**

Say we want to print a vector:

```C++
ostream &operator <<(ostream &out, const Vector &v) {
    for (size_t i = 0; i < v.size(), ++i) {
        out << v.itemAt(i) << " ";
    }
    return out;
}
```
WON'T COMPILE!!!
- Can't call `size()` and `itemAt()` on a const object. What if these methods change fields?
- C++ is very protective of const, and we need to promise those methods will not change fields
- Since they don't, declare them as `const`

```C++
struct vector {
    // ...
    size_t size() const;         // Means these methods will not modify fields
    int &itemAt(size_t i) const; // Can be called on const objects
    // ...
};

size_t vector::size() const { return n };
int &vector:itemAt(size_t i) const { return theVector[i]; }
```

Now the loop will work.

## **2021-09-28**

BUT:

```C++
void f(const vector &v) {
    v.itemAt(0) = 4;    // Works!! v is not very const...
}
```
- `v` is a const object - cannot change `n`, `cap`, `theVector` (ptr)
- You can changed items pointed to by `theVector`

Can we fix this?

We can try the following: (does not actually work but let's try first)
```C++
struct vector {
    ...
    const int &itemAt(size_t i) const;
};

const int &itemAt(size_t i) const {
    return theVector[i];
}
```
- Now `v.itemAt(0) = 4` won't compile if `v` is `const`, but it also won't compile if `v` is not const.

To fix: **const overloading**

```C++
struct vector {
    ...
    const int &itemAt(size_t i) const;  // Will be called if the object is const
    int &itemAt(size_t i);  // Will be called if object is non-const
};

inline const int &vector::itemAt(size_t i) const {return theVector[i];}
inline int &vector::itemAt(size_t i) {return theVector[i]};
```

- Putting in `inline` suggests the compiler to replace the function call with the function body to save the cost (overhead) of having to call a function
- Merely a suggestion, compiler can choose to ignore it if it sees fit. Good idea for small functions.
- If we do it like this, we need to write the body of the function in header file. We are telling the compiler to replace the function by its body, but the compiler would not know what the body is, so it must know at the time it was compiling -> header.

So now `v.itemAt(0) = 4;` will only compile if and only if `v` is non-const

Now let's make it prettier/idiomatic:

```C++
struct vector {
    size_t size() const {return n;} // Method body inside class implcity declares the method inline
    const int &operator[](size_t i) const {return theVector[i]};
    int &operator[](size_t i) {return theVector[i];}    
};

ostream &operator<<(ostream &out, const vector &v) {
    for (size_t i = 0; i < v.size(); ++i) {
        out << v[i] << " ";
    }

    return out;
}
```

---
[Moves << ](./problem_6.md) | [**Home**](../README.md) | [>> Tampering](./problem_8.md) 
