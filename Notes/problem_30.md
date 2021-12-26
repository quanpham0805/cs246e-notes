[Collecting Stats <<](./problem_29.md) | [**Home**](../README.md) | [>> Polymorphic Cloning](./problem_31.md)

# Problem 30 - Resolving Method Overrides at Compile-Time
## **2021-11-23**

**Recall:** Template Method Pattern

```C++
class Turtle {
    public:
        void draw() {
            drawHead();
            drawShell();
            drawFeet();
        }
    private:
        void drawHead();
        virtual void drawShell() = 0;   // vtable lookup
        void drawFeet();
};

class RedTurtle: public Turtle {
    void drawShell() override;
};
```
- Is there a way to avoid vtable lookup?

**Consider:**

```C++
template<typename T> class Turtle {
    public:
        void draw() {
            drawHead();
            static_cast<T*>(this)->drawShell();
            drawFeet();
        }
    private:
        void drawHead();
        void drawFeet();
};

class RedTurtle: public Turtle<RedTurtle> {
    friend class Turtle;
    void drawShell();
};

class GreenTurtle: public Turtle<GreenTurtle> {
    friend class Turtle;
    void drawShell();
};
```

No virtual method methods, no vtable lookup
- Drawback: no relationship between `RedTurtle` & `GreenTurtle`
    - Can't store a mix of them in a container

Can give `Turtle` a parent:

```C++
template<typename T> class Turtle: public Enemy { ... };
```

Then can store `RedTurtles` and `GreenTurtles`
- But then can't access the `draw` method
- Could give Enemy a virtual `draw` method

```C++
class Enemy {
    public:
        virtual void draw() = 0;
};
```

But then there will be a vtable lookup
- On the other hand, if `Turtle::draw` calls several would-be virtual helpers, could trade away several vtable lookups for one
- <details close> <summary>Note to self: Recall the AGE engine collision:</summary>

  ```cpp
  template <typename...> class Collidable {
  public:
  	virtual void collideWithBorder(Entity*, Border*) {}
      virtual ~Collidable() = default;
  };

  template <typename T> class Collidable<T> : public virtual Collidable<> {
  public:
  	virtual void collideWithEntity(Entity* currentEntity, T* other) = 0;
  };

  template <typename T, typename... Ts> class Collidable<T, Ts...> : public Collidable<Ts...>, public virtual Collidable<T> {
  public:
  	using Collidable<T>::collideWithEntity;
  };
  ```
  - What I did was making the client collision class extend 

  ```cpp
  MyEntityCollidable<{{entities that myEntity wants to collide with}}>
  ```
  - And the subclass of Entity:
  ```cpp
  void MyEntity::collideWithOtherEntity(Entity *other) {
  	Collidable<>* otherCollidablePtr = other->getCollidableForMutation();
  	if (!otherCollidablePtr)
  		return;
  	
  	if (auto* newCollidable = dynamic_cast<Collidable<MyEntity>*>(otherCollidablePtr)) {
  		newCollidable->collideWithEntity(other, this);
  	} else {
  		throw std::runtime_error(BAD_COLLISION);
  	}
  }
  ```
  - which means I'm relying on the client to correctly write this code and this is prone to error. 
  - One way to solve this is probably using templated method. 
  - Another way is probably use what we learnt in this problem, having the client adding the current class to the list of `typename` and do something with it.
</details>

---
[Collecting Stats <<](./problem_29.md) | [**Home**](../README.md) | [>> Polymorphic Cloning](./problem_31.md)