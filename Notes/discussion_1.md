
# Discussion 1

## Modules

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
```
export module m;
export int f(int, int);
```

In `mymod-impl.cc`:
```
module m;
int g(int x) { return x * x; }
int f(int x, iny u) { return g(x)+g(y); }
```

If we attempt to call `g` somewhere else, it will fail, since it was not
exported.

### EXAMPLE 3

Can we build bigger modules out of smaller ones?

``` yourmodule.cc
export module n;
export const char* yourString();
```

``` yourmodule-impl.cc
module n;
const char* yourString() { return "world\n"; }
```

``` mymod.cc
export module m;
export import n;

export const char* mysString();
```

``` mymod-impl.cc
module n;
const char* myString() { return "Hello "; }
```

``` main.cc
#include <iostream>
import m;

int
main()
{
    std::cout << myString() << yourString();
}
```

