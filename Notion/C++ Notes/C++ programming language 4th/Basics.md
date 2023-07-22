int main() {} is the minimum code for a c++ program.

#include <iostream> needed for input and output such as cout.

std::cout uses cout in namespace std

using namespace std; makes names from std visible: std::cout → cout.

+, -, *, /, % arithmetic operators.

==, !=, <, >, <=, >= comparison operators.

+=, ++, -=, --, *=, /=, %= more specific operators.

const: “i promise not to change this value”. (primarily used to specify interfaces)

constexpr: “to be evaluated at compile time”. (primarily used to specify constants)

<< (put to), >> (get from)

[] is a declaration of array of a type, * is a declaration of a pointer to a type

- prefix means contents of, & prefix means address of.

for(int i = 0; i <= 10; ++i): explicit declaration. for(auto i : {10, 11, 12, 13, 14}): range-for.

In declaration suffix & means reference to.

Empty pointer should point to nullptr, shared by all pointer types.

struct is used from User-defined types.

use . to access struct member by name and -> to access member by pointer.

Interface is defined by the public members, private members are accessible through that interface.