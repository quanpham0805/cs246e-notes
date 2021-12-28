
# Discussion 1: Modules

**Goal**: get rid of `#include`

C/C++ never had a proper modules system, instead:
- header files for interface info
- implementation files separately compiled
- linking of object files

What's missing?
- explicit interface with import/export. to hide something, you would omit from
  the header file. compiler is not aware of the difference between an included
  file, so it cannot stop clients from using hidden function
- including the same file means recompiling it every time, there is nothing in
  the standard for the compiler to recompile the header and reuse it
- no correspondence between a module and the name of the file that contains it
    eg. no one forcing you to use same name for .h and .cc file(s)

### EXAMPLE 1

Example module file `mymod.cc`:
```
export module m;

export int f(int, int);

int f(int x, int y) { return x * x + y * y; }
```

in `main.cc`
```
#include <iostream>

import m;

int
main()
{
    int x = 2, y = 3;
    std::cout << f(x, y) << '\n';
}
```

How does `main.cc` know about `m`? How does it know that `f` comes from `m`?
There's a new rule now, the module must be compiled before the client.

`g++20` creates a new `gcm.cache` directory and makes a .gcm file for each
module. The .gcm file name matches the module name, not the original file name.

g++-11 -std=c++20 -fmodules-ts

### EXAMPLE 2

Can we separate interface from implementation?

In `mymod.cc`:
```cpp
export module m;
export int f(int, int);
```

In `mymod-impl.cc`:
```cpp
module m;
int g(int x) { return x * x; }
int f(int x, iny u) { return g(x)+g(y); }
```

If we attempt to call `g` somewhere else, it will fail, since it was not
exported.

### EXAMPLE 3

Can we build bigger modules out of smaller ones?

_`yourmodule.cc`_
```cpp 
export module n;
export const char* yourString();
```
_`yourmodule-impl.cc`_
``` cpp
module n;
const char* yourString() { return "world\n"; }
```

_`mymod.cc`_
```cpp
export module m;
export import n;

export const char* mysString();
```

_`mymod-impl.cc`_
```cpp
module n;
const char* myString() { return "Hello "; }
```

_`main.cc`_
```cpp
#include <iostream>
import m;

int main()
{
    std::cout << myString() << yourString();
}
```

module != header because its not the cpp preprocessor
it does not "drop" the code directly on top of your program, but rather "importing" the lib as module

Important difference: any preprocessor directives in an imported header affect the header ONLY. It do not propagate the change to the file include it anymore.
(as they would with #include)

-> pre-compiled module files have not been built for the STL lib header, so import would fail.

This would significantly improve compile time, as module are already pre-compiled

compare with namespaces:
- they serve a different purpose
- better structure
- namespace is used to prevent name-collision
- elements of module are not prefixed by the module name, unlike namespaces
- namespaces are open, anyone can add more to the namespace, with the only exception is the std:

We can also combine namespace using module. Import the two, then place them under the same namespace.:

still to be worked out: relationship between cpp and make.i
- major handle: forced order of compilation.
- Traditional cpp we can compile in any order then link them together.
- Now, modules need to be compile first, and some before some other (imagine this being a tree, we need to compile from the root)
- Also cannot discover dependencies by compiling in isolatoin, since modules are placed into gcm.cache/....
  - As a result, we cannot use the compiler to spit out the dependencies
  - Well if you already know your dependencies, you can then use cmake, but well, what's the point that time then XD
