
# Continuation of structured Bindings (C++17)

Recall from last day that structured bindings do not work when there are
private fields, we need to provide our own.
```
class Posn {
    int x, y;
public:
    Posn(int x, int y): x{x}, y{y} {}
    template <size_t n> auto& get() {
        return (n == 0) ? x : y;
    }
}
```

We ask the question, does `x` and `y` need to be of the same type? No. Notice
that `n` is known at compile time, so get<0> and get<1> will be two different
functions. We deal with the return type by using the auto return type.

** the above will not type check for other reasons, we will visit it later

Continuing on:
```
namespace std {
    template<> struct tuple_size<Posn> {
        static const int value = 2;
    }
    template<> struct tuple_element<0, Posn> {
        using type = int;        
    }
    template<> struct tuple_element<1, Posn> {
        using type = int;        
    }
}
```

Now we can finally do this:
```
Posn p{1, 2};
auto& [a, b] = p;
```

Notice that this worked with Posn's private `x` and `y` fields.

## Class template argument deduction (C++17)

Recall that we can do:
```
int x = 1, y = 2;
std::swap(x, y);
```

where we did not need to saw `swap<int>(x, y)`, since the int is inferred from
the types of the arguments passed in. However, this is not possible with
template classes... until now.

Let's take a look at pair:
```
template <typename T, typename U> struct pair {
    T first;
    U second;
};
```

and when we construct it:
```
pair<int, char> p{1, 'a'};
```

this can start to get pretty tedious to type, so here's the standard
workaround. The idea is if the class can't do argument deduction, we can make
a function do it.
```
template<typename T, typename U>
pair<T, U> make_pair(T x, U y) {
    return pair<T, U>{x ,y};
}
```

and now we can do:
```
auto p = make_pair(1, 'a');
```

in this case, `make_pair` is a pass through function, so it should be doing
perfect forwarding.
```
template<typename T, typename U>
pair<T, U> make_pair(T&& x, U&& y) {
    return pair<T, U>{std::forward<T>(x), std::forward<y>(y)};
}
```

this, as you can see, is a lot of work. In C++17, we finally have template
argument deduction for classes. So we can simply say:
```
pair p{1, 'a'};
```

and pair is automatically given type `pair<int, char>`. How does this work? C++
now performs template argument deduction on the constructor of the class.

Now let  us ask, when will this not work?
```
template<typename T> class C { T x; }
C{"abc"};
```
and T is deduced as const char*. However, you may have hoped for it to be
std::string instead. You can help the compiler out by providing a 'deduction
guide':
```
template<typename T> C(const char*) -> C<string>;
class C { T x; }
```

We can also template our deduction guide:
```
template<typename T> C(T*) -> C<unique_ptr<T>>;
```

