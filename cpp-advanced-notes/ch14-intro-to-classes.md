# Chapter 14: Introduction to Classes

**Source:** [14.x Chapter 14 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-14-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Related: [14.17 Constexpr aggregates and classes](https://www.learncpp.com/cpp-tutorial/constexpr-aggregates-and-classes/) · [13.10 Passing and returning structs](https://www.learncpp.com/cpp-tutorial/passing-and-returning-structs/)

## Summary (condensed)

**Big ideas**
- **Procedural:** functions operate on data passed in. **OOP:** program-defined types bundle data (properties) with behavior.
- **Class invariant:** a condition that must hold for the object's whole life. If violated, the object is in an **invalid state**.
- **Class** = compound type bundling data + functions. Functions in it are **member functions** (called on the **implicit object**). If a type has no data members, prefer a namespace.
- **`const` member function:** promises not to modify the object or call non-const members. Make every member function const that doesn't modify state, so it works on const objects too.

**Access control**
- **Public** = anyone. **Private** = only members of the same class. Access is per *class*, not per *object*.
- `struct` members default to public. `class` members default to private.
- A class with private members is **not an aggregate**, so no brace/aggregate initialization.
- Prefix private members with `m_`. List public members first, private last (spotlights the interface).
- **Access functions:** getters (accessors) and setters (mutators), trivial public functions to read/write a private member.
- **Interface** = public members. **Implementation** = member variables + function bodies. **Data hiding / encapsulation** = keeping them separate. (Encapsulation is also sometimes used to mean "bundling data and functions", so it's ambiguous.)

**Constructors**
- A **constructor** initializes objects. A matching one must exist for any non-aggregate object.
- **Member initializer list:** prefer it to assigning in the body. List members in the order they're declared.
- **Default constructor:** takes no arguments (or all defaults). If you declare no constructors, the compiler generates an **implicit default constructor**.
- **Delegating constructors** (constructor chaining): a constructor can hand off to another one, but it can either delegate *or* initialize, not both.
- **Copy constructor:** initializes from an existing object of the same type. The compiler gives you an **implicit memberwise copy**.
- **As-if rule:** compilers may change the program as long as observable behavior is the same. **Copy elision** is an exception: the compiler may skip a copy even if the copy constructor had side effects.
- **Converting constructor:** one that can do implicit conversion (every constructor is one by default). **`explicit`** blocks that. Make single-argument constructors `explicit` by default. Don't make copy/move constructors explicit.
- **Temporary (anonymous) object:** no name, lives for one expression.
- Member functions and constructors can be `constexpr` (as of C++14, not implicitly const).

## Quiz and answers

1. **Point2d.**
   - (a) Members `m_x{0.0}`, `m_y{0.0}`. `Point2d() = default;` plus a two-parameter constructor. `print() const` → `Point2d(0, 0)`, `Point2d(3, 4)`. A one-argument `Point2d third{4.0}` should fail to compile.
   - (b) Add `double distanceTo(const Point2d& other) const` returning `std::sqrt((m_x-other.m_x)*(m_x-other.m_x) + (m_y-other.m_y)*(m_y-other.m_y))`. Header `<cmath>`. Distance between (0,0) and (3,4) = **5**.
2. **Fraction struct → class:** private `m_numerator{0}`, `m_denominator{1}`, an `explicit` constructor with defaults, and member versions of `getFraction()`, `multiply(const Fraction&) const` and `printFraction() const`. Usage: `f1.multiply(f2).printFraction();`
3. **Why `explicit` on the Fraction constructor?** It stops nonsense like `f1.multiply(true)`. Without `explicit`, `true` would implicitly convert to a `Fraction` via the `(int, int)` constructor. With it, that's a compile error.
4. **Why keep `getFraction()` and `print()` as non-members?**
   - A non-member `getFraction()` can *create and initialize* a Fraction in one step (the member version needs construct-then-call). It also contains app-specific prompts that don't belong in the class.
   - A non-member `print()` shrinks the class interface and can't touch internals, so changing the implementation never forces an update to it.

## My additions

- **Class vs struct rule of thumb:** plain data bag with no invariants → `struct`. Anything with invariants to protect → `class` with private data.
- **Why `explicit` matters:** implicit conversions are the source of "it compiled but did something weird" bugs. Default to `explicit` unless you have a reason not to.
- **Initializer-list order gotcha:** members are initialized in *declaration* order, not the order you write in the list. The compiler warns when they differ. Keep them matching.
- **Pairing with chapter 15:** the hidden `this` pointer is how member functions know which object they're on. Chapter 15 builds directly on `const` member functions and constructors.
- **Hardware link:** classes with a private register/state and a small public interface are the standard way to wrap a peripheral driver (UART, SPI, GPIO). The invariant might be "peripheral is configured before use".

---
[← Prev](ch13-enums-and-structs.md) | [Index](README.md) | [Next →](ch15-more-on-classes.md)
