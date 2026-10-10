# Chapter 13: Program-Defined Types: Enums, Structs, Templates

**Source:** [13.x Chapter 13 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-13-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Related: [13.1 Introduction to program-defined types](https://www.learncpp.com/cpp-tutorial/introduction-to-program-defined-user-defined-types/) · [13.14 CTAD and deduction guides](https://www.learncpp.com/cpp-tutorial/class-template-argument-deduction-ctad-and-deduction-guides/) · [13.15 Alias templates](https://www.learncpp.com/cpp-tutorial/alias-templates/)

## Summary (condensed)

- **Program-defined (user-defined) types:** enums and class types (structs, classes, unions). They must be defined before use. Type definitions are **exempt from the one-definition rule**.
- **Enumeration:** a type whose values are named constants (**enumerators**). Enums are **distinct types** (unlike type aliases).
  - **Unscoped enum:** enumerators land in the *surrounding* scope and implicitly convert to integers.
  - **Scoped enum (`enum class`):** enumerators live only inside the enum's own scope and do **not** implicitly convert to integers.
- **Struct:** bundles variables into one type. Variables are **data members**.
  - Member access: `.` for objects/references, `->` for pointers to structs.
- **Aggregate:** a type holding multiple members (arrays, and structs with only data members in C++).
  - **Aggregate initialization** = brace list, initialized **memberwise in declaration order**.
  - **Designated initializers (C++20):** `{ .health = 10 }`. Must follow declaration order.
  - **Default member initializers:** `int health{ 100 };` inside the struct definition.
- **Padding:** the compiler may insert gaps between members for alignment, so `sizeof(struct)` can be bigger than the sum of its members.
- **Class templates:** a blueprint for making class types. **CTAD (C++17)** lets the compiler deduce template arguments from the initializer (`Triad t{1, 2, 3};`). Pre-C++20 you may need a **deduction guide**.

## Quiz and answers

1. **Monster struct:** `struct Monster` containing a nested `enum Type { ogre, dragon, orc, giant_spider, slime }`, a `Type type{}`, a `std::string name{}` (the struct *owns* its name) and `int health{}`. A `constexpr std::string_view getMonsterTypeString(Monster::Type)` using a `switch`, plus `printMonster(const Monster&)`. Expected output: `This Ogre is named Torg and has 145 health.` and `This Slime is named Blurp and has 23 health.`
2. **Pass by value, const address or const reference?**

| Type | How to pass | Why |
|---|---|---|
| `char`, `unsigned long`, `bool` | value | fundamental, cheap |
| `std::string` | const reference | copying allocates |
| enumerated type | value | just an integer underneath |
| `Position {x, y, z}` (3 doubles) | const reference | struct, bigger than a few bytes |
| `Player {int health; ...}` (still growing) | const reference | will get bigger, so don't edit every call site later |
| `int` where null must be allowed | **const address** (`const int*`) | can pass `nullptr` |
| `std::string_view` | value | it's already a cheap view |

3. **`Triad` class template:**
   ```cpp
   template <typename T> struct Triad { T first{}; T second{}; T third{}; };
   template <typename T> Triad(T, T, T) -> Triad<T>;   // deduction guide (C++17 only)
   template <typename T> void print(const Triad<T>& t) { std::cout << '[' << t.first << ", " << t.second << ", " << t.third << ']'; }
   ```
   Output: `[1, 2, 3][1.2, 3.4, 5.6]`

## My additions

- **Prefer `enum class`** by default. It stops accidental int conversions and name collisions. Use unscoped enums only when you want implicit integer behavior (e.g. as array indices, see 17.6).
- **Pass-by-value rule of thumb:** fundamental types, enums and `string_view` by value. Anything else non-trivial by `const&`.
- **Padding matters in hardware work.** When mapping a struct onto a hardware register block or a network/wire format, padding can silently break the layout. Order members largest-to-smallest to minimize padding, and check with `sizeof`/`static_assert`.
- **Designated initializers are great for config structs** (e.g. peripheral settings): `Config{ .baud = 115200, .parity = false }` reads like documentation.
- **Aggregate ≠ aggregation.** Aggregate = brace-initializable type (this chapter). Aggregation = has-a relationship without ownership (chapter 23).

---
[← Prev](ch12-references-and-pointers.md) | [Index](README.md) | [Next →](ch14-intro-to-classes.md)
