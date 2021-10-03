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
