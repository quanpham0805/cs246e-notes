[Input/Output <<](./problem_1.md) | [**Home**](../README.md) | [>> Linear Collections and Modularity](./problem_3.md) 
# Problem 2: Separate compilation
## **2021-09-14**
Put echo in its own module

#### echo.h
```C++
void echo (istream &f);
```

#### echo.cc
```C++
#include "echo.h"

void echo(istream &f) {
    // ...
}
```

#### main.cc
```C++
#include <iostream>
#include <fstream>
#include "echo.h"

int main(...) {
    echo(cin);
    //...
    echo(f);
}

```

**Compiling separately:** 
- `g++14 echo.cc` (fails)
    - linking error: no main
- `g++14 main.cc` (fails)
    - linking error: no echo

Correct:

`g++14 -c echo.cc` -> creates `echo.o`

`g++14 -c main.cc` -> creates `main.o` (these are object files, binary code but imcomplete program)

`-c` indicates **only compile, don't link**

`g++14 echo.o main.o -o mycat` (linker)

Advantage:
- Only have to recompile the parts you change, then relink (no so expensive)
    - Ex. Change echo.cc -> recomplie echo.cc -> relink
- However if you change echo.h, you must recompile `echo.cc` and `main.cc` and relink
    - This is because both files include `echo.h`
- What if we don't remember what we changed or what depends on what?
    - Linux tool: `make`
    - Create a **Makefile**

```mak
mycat: main.o echo.o
    g++ main.o echo.o -o mycat

main.o: main.cc echo.h
    g++ -std=c++14 -Wall -c main.cc

echo.o: echo.cc echo.h
    g++ -std=c++14 -Wall -c echo.cc

.PHONY: clean

clean:
    rm mycat main.o echo.o
```

**Note:** cannot use `g++`aliases, requires **TABS** not **SPACE**

- **targets:** `mycat`, `main.o`, `echo.o`
- **dependencies:** everything right of colon
- **recipes:** tabbed information
- `make clean`: removes all binary.
- `.PHONY: clean`: `clean` is not a file name.

**How to use:**
- From the command line `cd` to the right directory then type `make`. This build the whole program. Done.
- If we make changes and call `make`, it will only recompile the necessary files. If no change was made, nothing happens.
- `make clean`: removes all binary.

How make works: list a dir in long form `ls -l`

Example: `-rw-r----- 1 j2smith j2smith 25 Sep 9 15:27 echo.cc`
- First dash (`-`) is type. The next 3 groups of 3 characters would represent owner/group/other permissions.
- The number after is `links`, should not worry about it for now.
- Next 3 are owner, group, and size of file.
- Next is last modified date and time.
- Last is file name.
- Based on last modified time.
- Starting at the leaves of the dependency graph
    - if the dependency is newer than the target, rebuild the target ... and so on, up to the root target

Example: 
- echo.cc newer than echo.o -rebuild echo.o
- echo.o newer than mycat - rebuild mycat

Shortcuts - use variables:

```bash
CXX = g++                       (name of compiler)
CXXFLAGS = -std=c++14 -Wall     (compiler options)
EXEC = myprogram
OBJECTS = main.o echo.o

${EXEC}: ${OBJECTS}
    ${CXX} ${OBJECTS} -o ${EXEC}

main.o: main.cc echo.h

echo.o: echo.cc echo.h

.PHONY: clean

clean: 
    rm ${OBJECTS} ${EXEC}
```

- Omitted recipes - make "guesses" the right one

Writing the dependencies still hard (but compiler can help).

- `g++14 -c -MMD echo.cc`: generates `echo.o, echo.d`. The `d` stands for dependencies.
- `cat echo.d` -> `echo.o echo.cc echo.h` 

```C
CXX = g++
CXXFLAGS = -std=c++14 -Wall -MMD
EXEC = mycat
OBJECTS = main.o echo.o
DEPENDS = ${OBJECTS:.o=.d}

${EXEC}: ${OBJECTS}
    ${CXX} ${OBJECTS} -o ${EXEC}

-include ${DEPENDS}

.PHONY: clean

clean:
    rm ${EXEC} ${OBJECTS} ${DEPENDS}

```
Now, all we need to keep track of is `OBJECTS` and `EXEC`.

Always use makefiles. Create MakeFile before we start coding.

---
[Input/Output <<](./problem_1.md) | [**Home**](../README.md) | [>> Linear Collections and Modularity](./problem_3.md) 