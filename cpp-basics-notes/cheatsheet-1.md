# Cheatsheet 1: C++ Fundamentals

Source: Codédex C++ cheatsheet.

## Character Output

```cpp
std::cout << "Howdy!";
```

## Program Structure

```cpp
#include <iostream>

int main() {
  std::cout << "Howdy!";
}
```

## Comments

```cpp
// Single line comments look like this

/* Multi-line comments look
something like this */
```

## Compile & Execute

```sh
g++ hello.cpp
./a.out
```

## Variables & Data Types

```cpp
std::string name = "Lady Bird";
int grade = 12;
double gpa = 3.99;
bool honor = true;
```

## Arithmetic Operators

```cpp
score = 4 + 3;  // 7
score = 4 - 3;  // 1
score = 4 * 3;  // 12
score = 4 / 3;  // 1.3333
score = 4 % 3;  // 1
```

## Chaining

```cpp
std::cout << "I'm " << age << " yrs old";
```

## User Input

```cpp
std::cout << "Enter your name: ";
std::cin >> username;
```

## Conditionals

```cpp
if (temp < 40) {
  std::cout << "Stay warm\n";
} else if (temp < 80) {
  std::cout << "Enjoy the weather\n";
} else {
  std::cout << "Time to cool down\n";
}
```

## Relational Operators

```cpp
a == b  // equal to
a != b  // not equal to
a > b   // greater than
a < b   // less than
a >= b  // greater than or equal to
a <= b  // less than or equal to
```

## Random Number

```cpp
int answer = std::rand() % 10;  // 0-9
```

## Logical Operators

```cpp
a && b  // true if both are true
a || b  // true if one is true
!a      // not true
```

## While Loop

```cpp
while (pin != 1234) {
  std::cout << "Incorrect. Enter again: ";
  std::cin >> pin;
}
```

## For Loop

```cpp
for (int i = 0; i < 10; i++) {
  std::cout << i << "\n";
}
```

---
[Index](README.md) | [Next →](cheatsheet-2.md)
