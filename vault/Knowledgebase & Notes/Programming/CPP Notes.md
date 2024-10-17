---
tags: cpp, programming
---

```c++
std::reverse(BEGIN, END); // reverses container.
std::reverse_copy(BEGIN, END, DEST_START); // copies container into another in reverse order
std::iter_swap(ONEPTR, ANOTHERPTR); // swaps the iterator of 2 elements. better than swapping values.
std::back_inserter(ptr to begin iterator) // inserts element into expandable container
std::to_string(someting) // converts something (usually a number) to a string.
```
```c++
int i = 0;
for (int j = 0; j < i; i++){
std::find(xxx);
}
```

# User Defined literals
https://en.cppreference.com/w/cpp/language/user_literal
You can append fun stuff with stuff like _m behind literals. This applies to string literals, integers, floats etc.
## String literal overload use
Example of overloading user-defined string literal, from ChatGPT:
```cpp
#include <iostream>
#include <string>

class MyString {
public:
    // Constructor to initialize from std::string
    MyString(const std::string& str) : value(str) {}

    // Overload the string literal operator
    friend MyString operator"" _mystr(const char* str, size_t) {
        return MyString(str);
    }

    void display() const {
        std::cout << "MyString: " << value << std::endl;
    }

private:
    std::string value;
};

int main() {
    MyString myStr = "Hello, World!"_mystr;  // Using the string literal operator
    myStr.display();
    return 0;
}
```
This example allows users to append `_mystr` behind a string literal, so that `"Hello, World!"_mystr` evaluates into a `MyString` type, not a `const char*` type.
Why? You can then add on error checking to string literals before passing it and letting others use it, or appending functions to the `MyString` class to e.g. display the string literal differently, or add more shit like getlength.
## Other literal overload use
```cpp
#include <iostream>

class Meter {
public:
    Meter(double value) : value(value) {}

    double getValue() const { return value; }

    friend Meter operator"" _m(long double val) {
        return Meter(static_cast<double>(val));
    }

private:
    double value;
};

int main() {
    Meter distance = 5.0_m;  // Using the string literal operator for meters
    std::cout << "Distance: " << distance.getValue() << " meters" << std::endl;
    return 0;
}
```
Extremely simple literal overload to be able to append `_m` to `long double`s, and returns basically the same thing, but still wrapped in a `Meter` class and accessible by `getValue`.
This helps to differentiate and do sanity checking that you are indeed working with meters and not inches or millimeters or something. You can abuse this by forcing the compiler to complain if you try to use the `Meter` class with `Inch` class or something, to enforce the correct use of unit types.

This can help to differntiate radians and degrees too. Also, having all your values tagged with _pi or _rad makes it easy to tell wtf you're working with in code.

## Downsides
Lots of work needed for each and every class to make it cross compatible with everything else, and this shit sin't very backwards compatible with the original string/double/int types either. Basically, almost everything needs to be touched and modified... Eh.