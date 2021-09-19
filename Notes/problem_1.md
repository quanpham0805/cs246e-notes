[**Home**](../README.md) | [>> Separate compilation](./problem_2.md) 

# Problem 1: Program Input/Output
## **2021-09-09**

- Read 2.2, 4.3 
- running a program from the command line

`./program-name` or `path/to/programe-name`

**Note:** `.` means current directory. Linux shell is going to find the program. The system has a set of place to look for the program that we need to run, and when we try to run, system will find in those places to run.

Providing input: 2 ways

1. `./program-name arg1 arg2 ... argn`
    - args are written into the program's memory
    - [code] [n + 1] [arg1] [arg2] [...] [argn]
    - There are `n + 1` arguments (usually called `argc` stands for arguments count), the first argument would be the name of the program (`argc >= 1`) and the list of arguments (usually called `argv` stands for argument vector), in which `argv[0] = program name`
    - Why save argv? Well it might be useful XD

1. `./program-name`
    - (Then type something)
    - input comes through standard input stream (stdin, connected to the keyboard by default)
    - keyboard -> program -> stderr (never buffered) OR -> stdout (maybe buffered) -> screen by default

**Redirection**
`./my-program < infile > outfile 2> errfile`

**Note:** 0 and 1 exists respectively, but only 2 is needed since it shares the same operator

Consider (`C`):

```C
#include <stdio.h>

void echo(FILE *f) {
    for (int c = fgetc(f); c != EOF; c = fgetc(f))
        putchar(c);
    }
}
```

**c is an `int` and not `char` because other information cannot be expressed as a `char`, for example `EOF` (end of file)**

```C
int main(int argc, char *argv[]) {
    if (argc == 1) {
        echo(stdin);
    } else {
        for (int i = 1; i < argc; ++i) {
            // to work with a file, you need to open it first
            // to open a file, you need to tell the OS you want this file, 
            // and it would give you the FILE pointer that has necessary info to open that file.
            FILE *f = fopen(argv[i], "r");
            echo(f);
            // Programming language rule: after you are done, close the file.
            // Having the file open might prevent other program from accessing it.
            fclose(f);   
        }     
    }

    return 0;  // Status code given to shell (echo $?)
}
```

Observe: cmd line args / input from stdin - 2 _different_ programming techniques

To compile: `gcc -std=c99 -Wall myprogram.c -o myprogram`
`gcc` translates C source to executable binary
`-Wall` means warn all, gives warning that only appear if asked for
`-o` means what to call the output

This program is a simplification of the linux `cat` command
`cat file1 file2 ... filen` opens them and prints them one after another

`cat` - echos stdin
`cat < file`

- File used as a source for stdin
- The shell (not cat) opens the file
- Displays

_Can we write the `cat` program in C++?_

- Already valid C++
- The "C++" way
    - Command-line args are the same as in C
    - stdin/stdout: #include <iostream>

```C++
#include <iostream>

int main() {
    int x, y;
    std::cin >> x >> y;
    std::cout << x + y << std::endl;
}
```

`std::cin`, `std::cout`, `std::cerr`: streams

types: `std::istream`: (cin) or `std::ostream`: (cout, cerr)

`>>` input operator: (`cin >> x`), populates `x` as a side-effect, produces cin

`<<` output operator: (`cout << x + y`), prints `x + y` as a side-effect, produces cout

These operators return cin/cout so they can be chained:
From above: `std::cin >> x >> y;`

### **File Access**
```C++
std::ifstream f{"name-of-file"};  // ofstream for output
char c;

while (f >> c) {
    std::cout << c;
}
```
`f >> c`
- implicity converts to a bool
- true = read succeded
- false = read failed

**Input:** the quick brown fox
**Output:** thequickbrownfox

> Stream input slips whitespace (just like scanf)

To include whitespace:

```C++
std::ifstream f {"name-of-file"};
f >> std::noskipws;
char c;
...
```

**Note:** 

- No explicit calls to fopen/fclose
- Initializing `f {"name-of-file"}` opens the file
- When `f`'s scope ends, the file is closed

Try `cat` in C++

```C++
#include <iostream>

using namespace std;  // Avoids having to say std::

void echo (istream f) { // this does not work, we need to pass by ref, more discussion later
    char c;
    f >> noskipws;
    while (f >> c) {
        cout << c;
    }
}

int main(int argc, char *argv[]) {
    if (argc == 1) {
        echo(cin);
    } else {
        for (int i = 1; i < argc; ++i) {
            ifstream f{argv[i]};
            echo(f);
        }
    }
}
```

Doesn't work, won't even compile!

`cin` has type `istream` - echo takes an istream
`f` has type ifstream - is `echo(f)` as type mismatch?

_No!_ - this is actually fine, ifstream is a **subtype** of istream

Any ifstream can be treated as an istream
- Foundational concept in OOP
- details later

The error is: you can't write a function that takes an istream the way that echo does

Why not?

Compare:

```C
int x;
scanf("%d", &x);
```
vs
```C++
int x;
cin >> x;
```

C and C++ are pass-by-value languages, so scanf needs the address of `x` in order to change `x` via pointers

So why is it not `cin >> &x`?
C++ has another small pointer-like type

### **References**

```C++
int y = 10
int &z = y; 
```

- `z` is an **lvalue reference** to `y`
- lvalue (old definition): **lvalue of an expression** is the value that is has when it is on the left hand side of the assignment. When you are dealing with the left hand side you don't care about the value, you are interested in the location/address.
- lvalue (modern definition): Something that could appear on the left hand side of an assignment.
- Similar to `int *const z = &y;` but with **auto-dereferencing**

`z = 12;`, NOT `*z = 12`; `y` is now 12

`int *p = &z;  // Gives the address of y`

In all cases, `z` acts as if it were `y`, `z` is an **alias** ("another name for `y`")

`z` is a lvalue reference of `y`.

**lvalue references** must be **initialized** to something that **has an address**

✅`int &z = y` _**CORRECT**_, `y` has an address, `y` is a lvalue

❌`int &z = 4;` _**WRONG**_, 4 is not a lvalue since it does not have a location in memory, it is a number

❌ `int &z = a + b;` _**WRONG**_, `a + b` is a quantity but does not denote an area of memory

**Cannot:**
- ❌Create a pointer to a reference: `int &*x;`
  - Read right to left:
    > x is a pointer to a reference of an int
- However you can create a reference to a pointer: `int *&x = __;`
- ❌Create a reference to a reference: `int &&r = z;` (compiles but means something else) 
- ❌Create an array of references: `int &r[3] = {...};`

## **2021-09-14**

You can pass as function parameters:
```C++
void inc(int &n /* changes affect variable x */) {
    n = n + 1; // no need pointer deref
}

int x = 5;
inc(x);
cout << x; // 6
```

**Note:** cannot use `inc(5)` because references require objects with addresses (lvalue) (ie. literals do not)

`cin >> x` works because `x` is passed by reference.

The implementation would look like:

`istream& operator >> (istream &in, int &n);`

`n` is a reference to `x`.

Now consider `struct ReallyBig {...};`

```C++
int f(ReallyBig rb) {
    ...
}
```

**Note:** `struct ReallyBig` is not necessary in C++ unlike in C, i.e, we can remove `struct`.

The case above uses passes by value which copies the huge struct -> this is really slow!

Instead let's pass by reference which doesn't copy it -> this is fast!
But this can be dangerous as the function may propogate changes to the caller.
```C++
int g (ReallyBig &rb) {
    ...
}
```
So finally we can use constant references, this is fast and safe!

```C++
int h(const ReallyBig &rb) {
    ...
}
```

What this means is, we cannot use `rb` to directly mutate the argument passed to the function (caller), but we can make a copy of `rb` to mutate it. Watch out!

Prefer **pass-by-const-ref** over **pass-by-value** for anything larger than a pointer (**❌DO NOT PASS-BY-REF FOR A SINGLE CHAR/BOOL❌**), unless the function needs to make a copy anyways, or when the function needs to mutate the argument, in which case we don't need the `const` modifier.

Also:

```C++
void f(int &n) {
    ...
}

void g(const int &n) {
    ...
}
```

`f(5)` - (**❌BAD❌**) can't initialize an lvalue reference `n` to a literal value `5` which does not have a location in memoery.

`g(5)` - (**✅GOOD✅**) since `n` can never be changed, the compiler allows this (stored in some temp location so `n` has something to point to)

Back to `cat`:
```C++
void echo(istream f) {
    ...
}
```
- `f` is passed by value, `istream` is copied
- Copying streams is _not allowed_ (C++ has mechanisms to prevent copying)

Works if you pass the stream by reference:

```C++
void echo (istream &f) {
    f >> noskipws;
    char c;

    while (f >> c) {
        cout << c;
    }
}
```
To compile:
- `g++ -std=c++14 -Wall mycat.cc -o mycat`
- OR `g++14 mycat.cc -o mycat`
- `./mycat`


[**Home**](../README.md) | [>> Separate compilation](./problem_2.md) 
