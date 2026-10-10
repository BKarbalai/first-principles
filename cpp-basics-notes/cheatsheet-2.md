# Cheatsheet 2: Vectors, Functions, Classes, Pointers

Source: Codédex C++ cheatsheet II.

## Vectors

```cpp
#include <vector>

std::vector<int> grades = {98, 87, 92};

std::cout << grades[0];  // Output: 98
```

## Adding & Removing Elements

```cpp
std::vector<std::string> dna = {"AUG"};

dna.push_back("UAA");   // {"AUG", "UAA"}
dna.pop_back();         // {"AUG"}
std::cout << dna.size();  // Output: 1
```

## Functions

```cpp
void voicemail() {
  std::cout << "Hi, I'm not here, bye!";
}
```

## Parameters & Return Value

```cpp
int add(int x, int y) {
  int answer = 0;
  answer = x + y;
  return answer;
}
```

## Classes

```cpp
class Student {
public:
  std::string name;
  int year;
private:
  double gpa;
  bool enrolled;
};

Student ferris;
```

## Class Methods

```cpp
class Employee {
  std::string name;
  int id;
  double salary;

  void display_info() {
    std::cout << name << id;
  }

  Employee(std::string n, int i, double s) {
    name = n;
    id = i;
    salary = s;
  }
};
```

## Pointers

```cpp
int x = 420;  // x is 420
int* p = &x;  // p holds x's address
```

## Dereferencing

```cpp
int* p = &x;
std::cout << *p << "\n";  // dereference
```

## Null Pointers

```cpp
int* p = nullptr;  // always initialize
```

## References

```cpp
int cups = 10;
int& reference = cups;
reference = 20;
std::cout << cups << "\n";  // Output: 20
```

## Pass by Reference

```cpp
void swap(int &a, int &b) {
  int temp = a;
  a = b;
  b = temp;
}
```

---
[← Prev](cheatsheet-1.md) | [Index](README.md)
