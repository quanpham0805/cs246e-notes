[I'm leaking! << ](./problem_20.md) | [**Home**](../README.md) | [>> The copier is broken (again)](./problem_22.md)

# Problem 21 - I want a class with no objects
## **2021-10-28**

```C++
class Student {
    public:
        virtual float fees() const;
};

class RegularStudent: public Student {
    public:
        float fees() const override;    // Regular student frees
}

class CoopStudent: public Student {
    public:
        float fees() const override;    // Co-op student frees
}
```

What should `Student::fees` do?

Don't know - every `Student` should either be `RegularStudent` or `CoopStudent`.

**Solution:** explicitly give `Student::fees` no implementation.
```C++
class Student {
    public:
        virtual float fees() const = 0; // must set it = 0, Stroustrup said 0 means there is no body
};
```
`Student` is an **abstract class**  
`fees()` is a **pure virtual method**

- <details open>
  <summary>Rant</summary>
  
  - In Java, there would be a `abstract` keyword, however this is not the case in C++, because C++ is an older language, other "newer" can add anything they want.
  </details>

Abstract classes cannot be instantiated:
```C++
Student s;  // ERROR
Student *s = new Student;   // ERROR
```
Can point to instances of **concrete classes** (non-abstract classes):
```C++
Student *s = new RegularStudent;
```
Subclasses of abstract classes are abstract are abstract, unless they implement every pure virtual method in the superclass.

Abstract classes 
- used to organize concrete classes
- can contain common fields/methods and default implementation (not need to be overrided).
- Can give partial implementation.

---
[I'm leaking! << ](./problem_20.md) | [**Home**](../README.md) | [>> The copier is broken (again)](./problem_22.md)