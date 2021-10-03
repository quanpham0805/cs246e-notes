# CS 246 Enriched (Object Oriented Programming) Notes

_Course information at the bottom of the page._

These notes cover CS 246E in Fall 2021. The textbook used is [Stroustrup, Bjarne. The C++ Programming Language, 4th edition, Addison Wesley](http://www.stroustrup.com/4th.html),
reading sections are referenced to this book. 

## Table of Contents

1. [F2021 updated] [Program Input / Output](Notes/problem_1.md)
1. [F2021 updated] [Separate compilation](Notes/problem_2.md)
1. [F2021 updated] [Linear Collections and Modularity](Notes/problem_3.md)
1. [F2021 updated] [Linear Collections and Memory Management](Notes/problem_4.md)
1. [The Copier is broken!](Notes/problem_5.md)
1. [Moves](Notes/problem_6.md)
1. [I want a constant vector](Notes/problem_7.md)
1. [Tampering](Notes/problem_8.md)
1. [Efficient Iteration](Notes/problem_9.md) 
1. [Staying in bounds](Notes/problem_10.md)
1. [I want a vector of chars](Notes/problem_11.md)
1. [Better Initialization](Notes/problem_12.md)
1. [I want a vector of Posns](Notes/problem_13.md)
1. [Less Copying!](Notes/problem_14.md)
1. [Memory management is hard!](Notes/problem_15.md)
1. [Is vector exception safe?](Notes/problem_16.md)
1. [Insert/remove in the middle](Notes/problem_17.md)
1. [Abstraction over containers](Notes/problem_18.md)
1. [Heterogeneous Data](Notes/problem_19.md)
1. [I'm leaking!](Notes/problem_20.md)
1. [I want a class with no objects](Notes/problem_21.md)
1. [The copier is broken](Notes/problem_22.md)
1. [I want to know what kind of Book I have](Notes/problem_23.md)
    1. [A Big Unit on Object Oriented Design](Notes/object_oriented_design.md)
1. [Shared Ownership](Notes/problem_24.md)
1. [Abstraction over Iterators](Notes/problem_25.md)
1. [I want an ever faster vector](Notes/problem_26.md)
1. [Collecting Stats](Notes/problem_27.md)
1. [Resolving Method Overrides at Compile Time](Notes/problem_28.md)
1. [Polymorphic Cloning](Notes/problem_29.md)
1. [Logging](Notes/problem_30.md)
1. [Total Control](Notes/problem_31.md)
1. [I want total control over vectors and lists](Notes/problem_32.md)
1. [A fixed-size allocator](Notes/problem_33.md)
1. [I want a (tiny bit) smaller vector class](Notes/problem_34.md)

## Other
1. [Valgrind + GDB](Notes/tutorial_1.md)
1. [Recursive Descent](Notes/tutorial_7.md)

## Discussions
1. [Modules](Notes/discussion_1.md)
1. [More modules](Notes/discussion_2.md)
1. [Tiny optimizations](Notes/discussion_3.md)

## Index
Work in progress (feel free to contribute)!

### A
- Abstract Class
    - [21](Notes/problem_21.md)
- Adapter Pattern
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Anonymous Namespace
    - [4](Notes/problem_4.md)
- Argument-Dependent Lookup (ADL)
    - [4](Notes/problem_4.md)

### B
- Basic Guarantee
    - [15](Notes/problem_15.md)

### C
- Class
    - [8](Notes/problem_8.md)
- Cohesion
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Const Cast
    - [22](Notes/problem_22.md)
- Const Overloading
    - [7](Notes/problem_7.md)
- Concrete Class
    - [21](Notes/problem_21.md)
- Contravariance Problem
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Copy and Swap Idiom
    - [5](Notes/problem_5.md)
- Copy Constructor
    - [5](Notes/problem_5.md)
- Copy/Move Elision
    - [6](Notes/problem_6.md)
- Coupling
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- The Curiously Recurring Template Pattern (CRTP)
    - [27](Notes/problem_27.md)

### D
- Decorator Pattern
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Destructor
    - [4](Notes/problem_4.md)
- Dependency Inversion Principle
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Dynamic Cast
    - [22](Notes/problem_22.md)

### E
- Exception Safety
    - [15](Notes/problem_15.md)

### F
- Factory Method Pattern
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Friend
    - [9](Notes/problem_9.md)
- Forwarding Reference
    - [14](Notes/problem_14.md)

### G

### H

### I
- Initializer List
    - [12](Notes/problem_12.md)
- Inline
    - [7](Notes/problem_7.md)
- Interface Segregation Principle
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Iterator
    - [9](Notes/problem_9.md)

### J

### K

### L
- Liskov Substitution Principle
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)

### M
- Makefile
    - [2](Notes/problem_2.md)
- Member Initialization List
    - [4](Notes/problem_4.md)
- Move Assignment
    - [6](Notes/problem_6.md)
- Move Constructor
    - [6](Notes/problem_6.md)

### N
- Namespaces
    - [3](Notes/problem_3.md)
- Non-Virtual Interface (NVI) Idiom
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Nothrow Guarantee
    - [15](Notes/problem_15.md)
- Nullptr
    - [3](Notes/problem_3.md)

### O
- Observer Pattern
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Open/Closed Principle
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)

### P
- Partial Assignment
    - [22](Notes/problem_22.md)
- Polymorphism
    - [19](Notes/problem_19.md)
- Pure Virtual Function
    - [21](Notes/problem_21.md)

### Q

### R
- Reference
    - [1](Notes/problem_1.md)
- Reinterpret Cast
    - [23](Notes/problem_23.md)
- Resource Acquisition is Initialization (RAII)
    - [15](Notes/problem_15.md) 
- Round Bracket Initialization
    - [12](Notes/problem_12.md)
- Run-Time Type Information (RTTI)
    - [22](Notes/problem_22.md)
- Rvalue Reference
    - [6](Notes/problem_6.md)

### S
- Separate Compilation
    - [2](Notes/problem_2.md)
- SFINAE (Substitution Failure Is Not An Error)
    - [26](Notes/problem_26.md)
- Single Responsiblity Principle
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- SOLID Principles of OO Design
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Static Cast
    - [23](Notes/problem_23.md)
- Strong Guarantee
    - [15](Notes/problem_15.md)
- Superclass
    - [19](Notes/problem_19.md)

### T
- Template
    - [11](Notes/problem_11.md)
- Template Metaprogramming
    - [25](Notes/problem_25.md)

### U
- Unique Pointer
    - [15](Notes/problem_15.md)
- Universal Reference
    - [14](Notes/problem_14.md)
- UML
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)

### V
- Virtual
    - [19](Notes/problem_19.md)
- Virtual Constructor Pattern
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Visitor Pattern
    - [A Big Unit on OO Design](Notes/object_oriented_design.md)
- Vtable
    - [19](Notes/problem_19.md)

### W

### X

### Y

### Z


## Course Information (Fall 2021 Pandemic version)
_Brad Lushman_  
_Online, M3 1006_  
_bmlushma@uwaterloo.ca_  
_[https://www.student.cs.uwaterloo.ca/~cs246e](https://www.student.cs.uwaterloo.ca/~cs246e)_  

Must use Linux:

**Windows:** 

putty.exe 

- connect to linux.student.cs.uwaterloo.ca
- enable X11 forwarding
- win scp

**Mac/Linux:** 

- terminal, ssh userid@linux.student.cs.uwaterloo.ca

Also Install xwindows server, eg. Xming, XQuartz

## Goals:

- Meet the CS 246 objectives, more breadth, more depth
- A course on abstraction
- Demand-driven, problem-oriented presentation, introduce C++ concepts as needed
- Linux tools on the side/tutorials
- C++17, C++20 features during discussion sessions.

## Todo:
- [ ] Introduced new `problem_2` and as a result, we need to move all the old files + 1. Need to update those as we go (header and footer for each file).