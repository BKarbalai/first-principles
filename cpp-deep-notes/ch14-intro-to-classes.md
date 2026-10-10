# Chapter 14: Introduction to Classes

**Source:** [14.x Chapter 14 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-14-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Related: [14.17 Constexpr aggregates and classes](https://www.learncpp.com/cpp-tutorial/constexpr-aggregates-and-classes/) · [13.10 Passing and returning structs](https://www.learncpp.com/cpp-tutorial/passing-and-returning-structs/) · Next chapter: [15.1 The hidden "this" pointer](https://www.learncpp.com/cpp-tutorial/the-hidden-this-pointer-and-member-function-chaining/)

This is the most important chapter in the series. Everything after it (inheritance, templates, operator overloading) builds on classes.

---

## 1. Procedural vs object-oriented programming

- **Procedural programming:** the focus is on **functions** ("procedures") that operate on data you pass around. Data and logic are separate.
- **Object-oriented programming (OOP):** the focus is on **program-defined types that bundle properties (data) with behaviors (functions)**. You ask the object to do something instead of passing it to a function.

---

## 2. Classes and member functions

A **class** is a compound type that bundles data *and* the functions that work on that data.

```cpp
class Date
{
    int m_year{}, m_month{}, m_day{};
public:
    void print() const { std::cout << m_year << '/' << m_month << '/' << m_day; }
};

Date d{};
d.print();     // d is the "implicit object"
```

- Functions that belong to a class are **member functions**. The object you call them on is the **implicit object**.
- Functions that aren't members are **non-member functions**.
- If a type has **no data members**, prefer a **namespace** instead of a class.

### `const` member functions
A **const member function** promises not to modify the object (or call any non-const member, which could). Put `const` after the parameter list:

```cpp
int getYear() const { return m_year; }
```
Make every member function const if it doesn't change state. That way it works on both const and non-const objects. (A non-const member function can't be called on a const object.)

---

## 3. Class invariants

A **class invariant** is a condition that must stay true for the object's entire life (e.g. "denominator is never 0", "buffer pointer is valid"). If an invariant is broken, the object is in an **invalid state** and further use may cause undefined or unexpected behavior. Good class design = making it impossible (or at least hard) for outside code to break the invariant. Access control is the tool for that.

---

## 4. Access levels (access control)

Every member has an **access level**:

- **public**: no restrictions. Accessible by anyone, including "the public" (all code outside the class: non-member functions, other classes).
- **private**: only accessible by other **members of the same class**.
- *(protected: comes with inheritance, chapter 24.)*

Rules:
- `struct` members default to **public**. `class` members default to **private**.
- Access levels are **per class, not per object** (one `Date` can read another `Date`'s private data).
- A class with private members is **no longer an aggregate**, so you can't use brace/aggregate initialization. You need constructors.
- Use `public:` / `private:` **access specifiers** to set levels. Structs should usually avoid specifiers (all public).
- **Style:** prefix member variables with `m_` to tell them apart from locals/parameters. Put **public members first**, private last, so the interface is the first thing a reader sees.

---

## 5. Interface vs implementation, data hiding

- **Interface** = how outside code uses the class = its **public members**.
- **Implementation** = the member variables and function bodies that make it work.
- **Data hiding (information hiding / data abstraction)** = keeping implementation hidden so users depend only on the interface. Benefits: you can change internals without breaking users, and invariants are protected.
- **Encapsulation** is often used as a synonym for data hiding, but it's also used to mean "bundling data and functions together" (regardless of access). So the word is ambiguous.

### Access functions (getters and setters)
- **Getter (accessor):** public member that returns a private member's value.
- **Setter (mutator):** public member that sets a private member's value.
- Make getters `const`. Return cheap types by value, expensive ones by `const&`.

---

## 6. Constructors

A **constructor** is a special member function that runs automatically when an object is created, to initialize it. A matching constructor must exist for every non-aggregate object you create.

```cpp
class Fraction
{
    int m_num{}, m_den{ 1 };
public:
    Fraction(int num, int den) : m_num{ num }, m_den{ den } {}
};
Fraction f{ 3, 4 };
```

### Member initializer lists
The `: m_num{ num }, m_den{ den }` part. **Prefer this** over assigning inside the body. It initializes members directly rather than default-initializing then assigning. **Members are initialized in declaration order**, not the order you list them (so keep the two aligned).

### Default constructors
A **default constructor** takes no arguments (or all arguments have defaults). It's used when you write `Fraction f{};`.
- If you declare **no** constructors, the compiler generates an **implicit default constructor**.
- If you declare *any* constructor, the implicit one disappears. Bring it back with `Fraction() = default;`.

### Delegating constructors (constructor chaining)
A constructor may hand off work to another constructor of the same class:
```cpp
Fraction() : Fraction{ 0, 1 } {}
```
A constructor can **delegate or initialize, but not both**.

### Temporary objects
`Fraction{ 1, 2 }` with no name creates a **temporary (anonymous/unnamed) object** that lives for one expression.

---

## 7. Copy constructor and copy elision

- A **copy constructor** initializes an object from an existing object of the same type. If you don't write one, C++ gives you an **implicit copy constructor** that copies each member (memberwise).
- **As-if rule:** the compiler may transform your program however it wants as long as the *observable behavior* is the same.
- **Copy elision** is an exception: the compiler is allowed to skip a copy even if the copy constructor had side effects (e.g. printing). When the compiler removes the call, the constructor is said to be **elided**.

---

## 8. Converting constructors and `explicit`

- A **user-defined conversion** converts a value to or from a program-defined type.
- A **converting constructor** is a constructor usable for **implicit** conversion. **By default, every constructor is one.**
- `explicit` tells the compiler not to use a constructor for implicit conversion, copy-initialization or copy-list-initialization.

```cpp
void f(Fraction);
f(5);        // OK if Fraction(int) is not explicit (silently converts 5 -> Fraction)
```
Guidelines:
- **Make single-argument constructors `explicit` by default.**
- Exception: where the implicit conversion is cheap and semantically identical (e.g. `std::string` → `std::string_view`).
- **Never** make copy/move constructors `explicit` (they don't convert anything).

---

## 9. constexpr classes
Member functions and constructors can be `constexpr` (lesson 14.17). As of C++14, `constexpr` member functions are **not implicitly const**.

---

## Quiz and answers

1. **Point2d.**
   - (a) Members `m_x{0.0}`, `m_y{0.0}`; `Point2d() = default;`; a two-parameter constructor; `print() const` → `Point2d(0, 0)`, `Point2d(3, 4)`. A one-argument `Point2d third{4.0}` should fail to compile.
   - (b) Add `double distanceTo(const Point2d& other) const` returning `std::sqrt((m_x-other.m_x)*(m_x-other.m_x) + (m_y-other.m_y)*(m_y-other.m_y))` (header `<cmath>`). Distance (0,0)→(3,4) = **5**.
2. **Fraction struct → class:** private `m_numerator{0}`, `m_denominator{1}`, an `explicit` constructor with defaults, and member versions `getFraction()`, `multiply(const Fraction&) const`, `printFraction() const`. Usage: `f1.multiply(f2).printFraction();`
3. **Why `explicit` on Fraction's constructor?** To stop nonsense like `f1.multiply(true)`. Without `explicit`, `true` would implicitly convert to a `Fraction` via the `(int, int)` constructor. With it, that's a compile error.
4. **Why keep `getFraction()` and `print()` as non-members?**
   - A non-member `getFraction()` can *create and initialize* a Fraction in one step (the member version needs construct-then-call). It also prints app-specific prompts that don't belong in the class.
   - A non-member `print()` shrinks the class interface and can't touch internals, so changing the implementation never forces an update to it.

---

## My additions

- **Class vs struct rule of thumb:** plain data bag with no invariants → `struct`. Anything with invariants to protect → `class` with private data.
- **Why `explicit` matters:** implicit conversions cause "it compiled but did something weird" bugs. Default to `explicit`.
- **Initializer-list order gotcha:** members initialize in *declaration* order, not list order. Compilers warn when they differ. Keep them matching.
- **Pairing with chapter 15:** the hidden `this` pointer is how a member function knows which object it's on. Chapter 15 builds on const member functions and constructors.
- **Prefer non-member, non-friend functions** when they can do the job through the public interface. It keeps the class small.
- **Hardware link:** a class with a private register/state and a small public interface is the standard way to wrap a peripheral driver (UART, SPI, GPIO). The invariant might be "peripheral is configured before use".

---
[← Prev](ch13-enums-and-structs.md) | [Index](README.md) | [Next →](ch15-more-on-classes.md)
