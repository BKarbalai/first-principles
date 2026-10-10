# Chapter 25: Virtual Functions

**Source:** [25.x Chapter 25 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-25-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Related lessons: [25.2 Virtual functions and polymorphism](https://www.learncpp.com/cpp-tutorial/virtual-functions/) · [25.11 Printing inherited classes using operator<<](https://www.learncpp.com/cpp-tutorial/printing-inherited-classes-using-operator/)

This chapter is what makes inheritance (chapter 24) actually useful: **polymorphism**, where one piece of code works with many related types and each behaves in its own way.

---

## 1. Base pointers/references to derived objects

A pointer or reference to a base class can point at a **derived** object (a Dog *is an* Animal). That lets you write one function or one array that handles every kind of derived object:

```cpp
Derived d{};
Base& ref{ d };    // OK
Base* ptr{ &d };   // OK
```

**Without `virtual`**, though, the base pointer/reference sees only the **base** part: calling `ref.getName()` runs `Base::getName()`, even though the real object is a `Derived`. That's the problem virtual functions solve.

---

## 2. Virtual functions and overrides

Mark a function **`virtual`** in the base class, and a call through a base pointer/reference is resolved to the **most-derived version** of that function that exists for the real object.

```cpp
class Base    { public: virtual std::string_view getName() const { return "Base"; } };
class Derived : public Base { public: std::string_view getName() const override { return "Derived"; } };

Derived d{};
Base& b{ d };
std::cout << b.getName();   // "Derived"
```

- A derived function that replaces a virtual base function is an **override**.
- For it to count as an override, it must have the **same signature** (name, parameter types, and `const`-ness) and the **same return type**.
- **Exception: covariant return types.** An override may return a pointer/reference to a *derived* type if the base version returns a pointer/reference to the *base* type.
- Once a function is virtual, it stays virtual in all derived classes (even without writing `virtual`).
- **Don't call virtual functions from constructors or destructors.** The derived part isn't built yet / has already been torn down, so you get the base version.

### `override` and `final`
- **`override`** (on the derived function): the compiler **errors** if it doesn't actually override something. It catches typos and signature mismatches. Use it on every override.
- **`final`**: prevents a virtual function from being overridden further, **or** prevents a class from being inherited from.

---

## 3. Virtual destructors

If you delete a derived object through a **base pointer**, and the base destructor isn't virtual, only the **base** destructor runs. The derived part isn't cleaned up (leak/UB). So:

> **Any class meant to be used polymorphically should have a `virtual` destructor.**

```cpp
virtual ~Base() = default;
```

---

## 4. Early vs late binding, and the vtable

- **Binding** = connecting a function name to its definition.
- **Early (static) binding:** resolved at compile/link time. Ordinary function calls.
- **Late (dynamic) binding:** resolved at **runtime**. Virtual calls.

How virtual calls work: each class with virtual functions gets a **virtual table (vtable)**, a lookup table of function pointers. Each object has a hidden **vptr** pointing to its class's vtable. A virtual call follows the vptr → table → function.

**Cost:** virtual calls are slightly slower (extra indirection), and every object of a class with virtual functions has **one extra hidden pointer**.

You can bypass virtual dispatch with the scope operator: `base.Base::getName()` calls the base version explicitly.

---

## 5. Pure virtual functions and abstract classes

A **pure virtual function** has no required implementation in the base, declared with `= 0`:

```cpp
virtual std::string_view getName() const = 0;
```
- A class with at least one pure virtual function is an **abstract class**: **you can't create objects of it**. It exists only to be derived from.
- A derived class **must override all pure virtual functions**, or it's abstract too.
- A pure virtual function **can still have a body** (derived classes can call it explicitly).

### Interface classes
An **interface class** has **no member variables** and **only pure virtual functions** (usually with a virtual destructor). By convention, names start with `I` (e.g. `IShape`). It defines a contract; derived classes must supply the behavior.

---

## 6. Virtual base classes (the diamond problem)

If class D inherits from both B and C, and both of those inherit from A, D normally gets **two copies** of A. Declaring the inheritance `virtual` (`class B : virtual public A`) makes D contain **only one shared** A. Used sparingly; it complicates construction rules.

---

## 7. Object slicing

If you assign or pass a **derived object to a base *object*** (by value, not pointer/reference), only the **base part is copied**. The derived members are "sliced off", and virtual calls then resolve to the base version.

```cpp
Derived d{};
Base b{ d };      // sliced! b is just a Base
```
Slicing is almost always a bug. Avoid by passing/storing by **reference or pointer**. (This is also why `std::vector<Base>` can't hold derived objects, so use `std::vector<Base*>` or `std::vector<std::unique_ptr<Base>>`.)

---

## 8. Downcasting with `dynamic_cast`

Converting a **base pointer/reference to a derived pointer/reference** is a **downcast**. `dynamic_cast` does it **safely at runtime**:

```cpp
Base* b{ getObject() };
if (auto* d{ dynamic_cast<Derived*>(b) })   // nullptr if b isn't actually a Derived
    d->derivedOnlyFunction();
```
- A failed pointer cast returns **`nullptr`**; a failed reference cast throws `std::bad_cast`.
- Needs the base to be polymorphic (have at least one virtual function) and RTTI enabled.
- `static_cast` also downcasts but does **no** runtime check (unsafe if you're wrong).
- If you need `dynamic_cast` a lot, the base interface is probably missing a virtual function.

---

## 9. Printing inherited classes with `operator<<`

`operator<<` can't be virtual (it's not a member). Standard trick: write **one** `operator<<` for the base class that calls a **virtual `print()` member**:

```cpp
class Base {
public:
    virtual void print(std::ostream& out) const { out << "Base"; }
    friend std::ostream& operator<<(std::ostream& out, const Base& b) { b.print(out); return out; }
    virtual ~Base() = default;
};
```
Each derived class overrides `print()`, and `std::cout << someBaseRef;` prints the right thing.

---

## Quiz: spot the bug (each program should print "Derived")

| # | What's wrong |
|---|---|
| 1a | `Base::getName()` isn't `virtual`, so the call goes to the base version. |
| 1b | Base's `getName()` is non-const, Derived's is `const`: different signatures, so it isn't an override. |
| 1c | `Base b{ d };` copies by value, so `d` gets **sliced**. Needs a reference or pointer. |
| 1d | `Base` is `final`, so `Derived` can't inherit from it. Compile error. |
| 1e | `Derived::getName()` is pure virtual (even with a body), making `Derived` abstract, so it can't be instantiated. |
| 1f | Output is right, but `delete b` through a `Base*` with **no virtual destructor** leaks the Derived part. |

**Coding exercises (short version of the solutions):**
- **2a:** `Shape` is abstract with a pure virtual `print(std::ostream&) const`, a friend `operator<<` that calls `print`, and a `virtual ~Shape() = default;`.
- **2b:** `Triangle` (three `Point`s) and `Circle` (centre `Point` + `int` radius) each override `print` with `override`.
- **2c:** Loop over `std::vector<Shape*>` printing `*element`. For the largest radius: `dynamic_cast<const Circle*>(element)`, only use it if non-null. Then `delete` each element.
- **2d (extra credit):** `std::vector<std::unique_ptr<Shape>>`. Can't use an initializer list (needs copies), so `push_back(std::make_unique<...>(...))`. `.get()` gives the raw pointer for `dynamic_cast`. No manual `delete` needed.

---

## My additions

- **Checklist for a polymorphic base class:** virtual destructor, `override` on every override, pure virtual for "must implement" functions, and pass/store by pointer or reference (never by value).
- **`override` is free safety.** Bugs 1a and 1b above become compile errors if `override` is written on the derived function. Make it a habit.
- **Embedded/hardware angle:** virtual calls cost an extra pointer per object plus an indirect call. On tiny microcontrollers (ESP32, STM32) with little RAM, that matters. Interface classes (a generic `ISensor` or `IDriver`) are still a clean way to swap hardware backends without changing the calling code.
- **`dynamic_cast` is a design smell if overused,** and some embedded builds disable RTTI entirely.
- **Slicing + containers:** `std::vector<Base>` slices everything you put in. That's why exercise 2c stores pointers (or smart pointers).
- **Connects to chapter 23:** a vector of base-class smart pointers is an owning container (composition) of polymorphic objects.

---
[← Prev](ch23-object-relationships.md) | [Index](README.md)
