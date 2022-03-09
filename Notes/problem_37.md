[I want total control over vectors and lists <<](./problem_36.md) | [**Home**](../README.md) | [>> I want a (tiny bit) smaller vector class](./problem_38.md) 

# Problem 37: A fixed-size object allocator
## **2021-11-30**

A custom allocator can be significantly faster than the standard allocator. Why?

**Fixed size allocator:** all allocated "chunks" are the same size (ie. customized code for one class) - no need to keep track of sizes
- Beside - many traditional allocators would store the size of the block before the pointer so that the allocator knows how much space is allocated to that pointer.

**Fixed size:**
- Saves space (no hidden size field)
- Saves time (no hunting for a block of the right size)

**Approach:** create a pool of memory  - an array large enough to hold `n` `T` objects.
- When a slot in the array is given to the client, it will act as a `T` object
- When we have it, it will act as a node in a linked list
- Store an `int` in each slot, each slot stores the index of the next slot in the list

To initialize, it would store the index of the first slot in the list. To demonstrate, initially, we have
```
first_available
     +---+            +---+---+---+---+-----+----+
     | 0 |----------->| 1 | 2 | 3 | 4 | ... | -1 |
     +---+            +---+---+---+---+-----+----+

                        0   1   2   3         n-1
```
Allocation: from the front
```
          Allocated
+---+       +---+---+---+---+-----+----+
| 1 |       |///| 2 | 3 | 4 | ... | -1 |
+---+       +---+---+---+---+-----+----+

              0   1   2   3         n-1

+---+       +---+---+---+---+-----+----+
| 2 |       |///|///| 3 | 4 | ... | -1 |
+---+       +---+---+---+---+-----+----+

              0   1   2   3         n-1
```
Deallocation:
```
Free item 0
+---+       +---+---+---+---+-----+----+
| 0 |       | 2 |///| 3 | 4 | ... | -1 |
+---+       +---+---+---+---+-----+----+

              0   1   2   3         n-1

Free item 1
+---+       +---+---+---+---+-----+----+
| 2 |       | 2 | 0 | 3 | 4 | ... | -1 |
+---+       +---+---+---+---+-----+----+

              0   1   2   3         n-1
```
- Allocation/deallocation is constant time and very fast

_Implementation:_
```C++
template<typename T, int n> class fsAlloc {
private:
    // union to represent use case based on user/us having fsAlloc
    union Slot {
        int next;
        T data;
        Slot(): next{0} {}
    };

    Slot theSlots[n];
    int firstAvailable = 0;
public:
    fsAlloc() {
        for (int i = 0; i < n - 1; ++ i)
            theSlots[i].next = i + 1;
        theSlots[n - 1].next = -1;
    }

    T *allocate() noexcept {
        if (firstAvailable == -1) return nullptr;
        
        T *result = &(theSlots[firstAvailable].data);
        firstAvailable = theSlots[firstAvailable].next;
        return result;
    }

    void deallocate(void *item) noexcept {
        int index = (static_cast<char*>(item) - reinterpret_cast<char*>(theSlots)) / sizeof(Slot);
        theSlots[index].next = firstAvailable;
        firstAvailable = index;
    }
};
```

Use in a class:

```C++
class Student final {
    int assns, mt, final;     
    static fsAlloc<Student, SIZE> pool; // SIZE - how many you want
public:
    // ...
    static void* operator new(size_t size) {
        if (size != sizeof(Student)) throw std::bad_alloc;

        while (true) {
            void *p = pool.allocate();
            if (p) return p;
            std::new_handler h = std::get_new_handler();
            if (h) h();
            else throw std::bad_alloc;
        }
    }

    static void* operator delete(void *p) noexcept {
        if (p == nullptr) return;
        pool.deallocate(p);
    }
};

// static field so it needs to be declared here
fsAlloc<Student, SIZE> Student::pool;
```
_Example main:_
```C++
int main() {
    Student *s1 = new Student;  // Uses custom allocator
    Student *s2 = new Student;  // Uses custom allocator
    delete s1;
    delete s2;
}
```
- Is it fast? Yes, the example in lectures show that allocating 10000 objects would take 2 seconds less than the built-in one (7 to 5)
- Using O3 optimization, the built-in one took 6 while the custom took 2.
- Percentage-wise, this is pretty good


**Question:** Where do `s1` and `s2` reside?

**Answer:** Static memory (NOT the heap)
- Depending on how you build your allocator, you could arrange for stack/heap memory as well
- Extra, for sweaty tryhards: use policy lol

**More notes:** 
- We used a union to hold both `int *T` 
    - Advantages: Wastes less space
    - Disadvantage: if you access a dangling `T` pointer, you can corrupt the linked list
        ```C++
        Student *s = new Student;
        delete s;
        s->setAssns(...);
        ```
- Lesson: following a dangling pointer can be VERY dangerous
- We could have used a struct `[ next | T ]`
    - Advantages: `next` is before the `T` object, so you have to work hard to corrupt it (this example is not something you do by accident)
    ```C++
    reinterpret_cast<int *>(s)[-1] = ...
    ```
    - Disadvantages:
        - Using union - if one of the fields has a ctor, you have to give the union a ctor, since it's a union, the ctor should initialize only one field
        - On the other hand, if `T` doesn't have a default constructor, you will have difficulty
        ```C++
        struct Slot {
            int next;
            T data;
        };

        Slot theSlots[n];   // X - if T has no default constructor
        ```
    - Can't do the trick of  `operator new`/`placement new`
        - We are trying to write our own replacement for `operator new`!
        - Workaround:
        ```cpp
        struct SlotChars {
            char arr[sizeof(Slot)];
        }
        SlotChars theSlotChars[n];
        Slot* theSlot = reinterpret_cast<Slot*>(theSlotChars);
        ```
        which is kinda gross
        - Other way: take advantage of the fact that union only needs to initialize one field, we can just put a dummy char and the slot inside a union then initialize the char
        ```C++
        union SlotChar {
            char dummy; // As before
            slot s;
            SlotChar(): dummy{0} {}
        };
        SlotChars theSlot[n];
        ```

Also:
- Why store indices instead of addresses?
    - `int`s are smaller than ptrs on this machine (32 vs 64 bits)
    - So waste no memory as long as `T` >= size of an `int`, but we do waste memory if `T` smaller than an `int`
        - To avoid, we could use a smaller index than an `int`, ex. `short`, `char`, as long as you don't want more items than the type can hold (you prob don't want more than 2 billion items)
    - Could make the index type a parameter of the template
- Why is the class `Student` declared `final`?
    - We have written a _fixed-size allocator_
    - If the `Student` has a subclass that adds fields, the subclass will still use `Student::operator new`, which will not allocate enough space to hold the subclass objects
        - Options to resolve:
          - Have no subclasses (`final`)
          - Check size, throw if it isn't the right size (our impl has both of this and the above, though one would be enough. Note that `final` would check at compile time, while throwing detects it at runtime)
          - Check size, use global operator new/delete if not the right size (note that operator delete can take a second parameter that contains the size of the memory being deleted)
          - Derived class can have its own allocator

---
[I want total control over vectors and lists <<](./problem_36.md) | [**Home**](../README.md) | [>> I want a (tiny bit) smaller vector class](./problem_38.md) 
