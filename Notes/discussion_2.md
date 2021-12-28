# Discussion 2: If/Switch Initializations
This comes from C++17.

Inspiration: in a for loop, we have the following:
```cpp
for (int i = 0; i < n; ++i) {
    ...
}
```
- we have the ability to create a variable which is only defined within the scope of the `for` loop (ie i).
- this would be nice to have in other contexts.
- we already do - sort of.
    - the following is legal in C++14 and before:
```cpp
if (int errorcode = doSomething()) {
    // handle the error
    // errorcode in scope
}
else {
    // normal outcome
    // note that errorcode is still in scope
}
```
- we can declare a variable as a condition in an `if` statement
- the variable is within the scope of the if/else statement.
- so this is good for checking non-zero, non-null (eg error checks)
- however, this is not good for "general" conditions.

But in C++17, we can do this:
```cpp
if (int result = doSomething(); result >= 0) {
    // result >= 0 case
    // result is in scope
}
else {
    // result < 0 case
    // result is also in scope
}
// result is out of scope
```
This also works for `switch` statements:
```cpp
switch (Token t = getToken(); t.tokenType()) {
    case ...:
    ...
}
// t is out of scope.
```
- note that `t.tokenType()` has to be an enum type / int type (rules of switch statements)

