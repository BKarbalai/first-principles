# Chapter 13: Program-Defined Types: Enums, Structs, Templates

**Source:** [13.x Chapter 13 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-13-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Related: [13.1 Introduction to program-defined types](https://www.learncpp.com/cpp-tutorial/introduction-to-program-defined-user-defined-types/) · [13.10 Passing and returning structs](https://www.learncpp.com/cpp-tutorial/passing-and-returning-structs/) · [13.14 CTAD and deduction guides](https://www.learncpp.com/cpp-tutorial/class-template-argument-deduction-ctad-and-deduction-guides/) · [13.15 Alias templates](https://www.learncpp.com/cpp-tutorial/alias-templates/)

---

## 1. Program-defined (user-defined) types

C++ ships with fundamental types (`int`, `double`, `bool`...), but you can create your own types. The main kinds are **enumerated types** and **class types** (structs, classes and unions).

- A program-defined type must be **defined before it's used**. The definition is called a **type definition**.
- Type definitions are **exempt from the one-definition rule** (ODR). That's why you can put one in a header and `#include` it in many `.cpp` files. (You still can't include it twice in the *same* file, hence header guards.)

---

## 2. Enumerations

An **enumeration (enum)** is a type whose possible values are a fixed set of named constants called **enumerators**.

```cpp
enum Color { red, green, blue };   // unscoped
Color c{ green };
```

Important property: an enum is a **distinct type**, meaning the compiler can tell it apart from `int` or from another enum. (A type *alias* is not distinct; it's just another name.)

### Unscoped enumerations
- Enumerator names are placed in the **same scope as the enum itself**, so `red` can be used directly. This can cause **name collisions** (two enums can't both have `red`).
- They have a named scope region too, so `Color::red` also works.
- They **implicitly convert to integers** (`red` → 0, `green` → 1, ...). Convenient (e.g. as array indices) but also a source of bugs.

### Scoped enumerations (`enum class`)
- Enumerators exist **only inside the enum's scope**: you must write `Color::red`.
- They **do not implicitly convert to integers**. You must `static_cast` explicitly.
- Prefer these by default. Fewer collisions, fewer silent conversions.

```cpp
enum class Fruit { apple, banana };
Fruit f{ Fruit::apple };
int i = static_cast<int>(f);   // explicit cast required
```

---

## 3. Structs

A **struct** bundles multiple variables into one type. Each variable inside is a **data member** (member variable).

```cpp
struct Employee
{
    int id{};
    int age{};
    double wage{};
};
```

**Member selection:**
- `.` (member selection operator) for struct objects and references: `joe.age`.
- `->` (member selection from pointer) for pointers to structs: `ptr->age` (shorthand for `(*ptr).age`).

### Aggregates and aggregate initialization
An **aggregate** is any type that can hold multiple data members. In C++ that means arrays and structs/classes with **only public data members** (no user-declared constructors, etc.).

**Aggregate initialization** uses a brace-enclosed **initializer list**. It performs **memberwise initialization**: values are assigned to members **in declaration order**.

```cpp
Employee frank{ 1, 32, 60000.0 };    // id=1, age=32, wage=60000.0
Employee joe{ 2, 28 };               // wage is value-initialized (0.0)
```

**Designated initializers (C++20)** let you name the members:

```cpp
Employee e{ .id = 3, .wage = 55000.0 };   // age defaults to 0
```
They must still follow declaration order, or it's an error.

### Default member initializers
You can give each member a default right in the definition. This is **non-static member initialization**, and the value is a **default member initializer**:

```cpp
struct Monster { int health{ 100 }; };
```

### Padding
The compiler may insert unused **padding** bytes between members for alignment/performance, so `sizeof(struct)` can be **larger than the sum** of its members' sizes.

---

## 4. Passing and returning structs
Structs can be passed to and returned from functions like any type. Pass by **const reference** to avoid copying a big struct. Returning by value is fine and common (the compiler can often elide the copy).

```cpp
void print(const Employee& e);
Employee make();                // returns by value
```

---

## 5. Class templates and CTAD

A **class template** is a blueprint the compiler uses to generate class types for particular template arguments.

```cpp
template <typename T>
struct Pair { T first{}; T second{}; };

Pair<int> p1{ 1, 2 };
Pair<double> p2{ 1.2, 3.4 };
```

**Class template argument deduction (CTAD, C++17)** lets the compiler deduce the template argument from the initializer, so you can write `Pair p{ 1, 2 };`. Before C++20, aggregate class templates often need a **deduction guide** to make this work:

```cpp
template <typename T> Pair(T, T) -> Pair<T>;
```

**Alias templates (13.15)** let you make a type alias that is itself a template, e.g. `template <typename T> using Vec = std::vector<T>;`.

---

## Quiz and answers

1. **Monster struct:** `struct Monster` with a nested `enum Type { ogre, dragon, orc, giant_spider, slime }`, a `Type type{}`, a `std::string name{}` (owns its name), and `int health{}`. A `constexpr std::string_view getMonsterTypeString(Monster::Type)` using a `switch`, plus `printMonster(const Monster&)`. Output: `This Ogre is named Torg and has 145 health.` / `This Slime is named Blurp and has 23 health.`
2. **Pass by value, const address or const reference?**

| Type | How to pass | Why |
|---|---|---|
| `char`, `unsigned long`, `bool` | value | fundamental, cheap |
| `std::string` | const reference | copying allocates |
| enumerated type | value | just an integer underneath |
| `Position {x, y, z}` (3 doubles) | const reference | struct bigger than a few bytes |
| `Player {int health; ...}` (growing) | const reference | will get bigger; avoid editing every call later |
| `int` where null must be allowed | **const address** (`const int*`) | can pass `nullptr` |
| `std::string_view` | value | already a cheap view |

3. **`Triad` class template:**
```cpp
template <typename T> struct Triad { T first{}; T second{}; T third{}; };
template <typename T> Triad(T, T, T) -> Triad<T>;          // deduction guide (C++17 only)
template <typename T> void print(const Triad<T>& t)
{ std::cout << '[' << t.first << ", " << t.second << ", " << t.third << ']'; }
```
Output: `[1, 2, 3][1.2, 3.4, 5.6]`

---

## My additions

- **Prefer `enum class`** by default. Use unscoped enums only when you *want* implicit integer behavior (e.g. as array indices, see 17.6).
- **Pass-by-value rule of thumb:** fundamentals, enums and `string_view` by value; everything else non-trivial by `const&`.
- **Padding matters in hardware work.** When a struct must match a hardware register block or a wire format, padding can silently break the layout. Order members largest-to-smallest and check with `sizeof`/`static_assert`.
- **Designated initializers are great for config structs** (e.g. peripheral settings): `Config{ .baud = 115200, .parity = false }` reads like documentation.
- **Aggregate ≠ aggregation.** Aggregate = brace-initializable type (this chapter). Aggregation = has-a relationship without ownership (chapter 23).

---
[← Prev](ch12-references-and-pointers.md) | [Index](README.md) | [Next →](ch14-intro-to-classes.md)
