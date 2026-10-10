# Chapter 25: Virtual Functions

**Source:** [25.x Chapter 25 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-25-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Related lessons: [25.2 Virtual functions and polymorphism](https://www.learncpp.com/cpp-tutorial/virtual-functions/) · [25.11 Printing inherited classes using operator<<](https://www.learncpp.com/cpp-tutorial/printing-inherited-classes-using-operator/)

## Summary (condensed)

- A **base-class pointer or reference can point at a derived object**. That lets you write one function or one array that works with any type derived from the base.
- **Without `virtual`**, a base pointer/reference only sees the base's member variables and the base's versions of functions.
- A **virtual function** resolves to the *most-derived* version (the **override**). For an override, the derived function needs the same signature and return type. Exception: **covariant return types** (an override may return a pointer/reference to a derived type if the base returns one to the base type).
- Mark intended overrides with **`override`** so the compiler checks they really are overrides.
- **`final`** stops a function from being overridden, or a class from being inherited.
- If a class will be used polymorphically, give it a **virtual destructor**, so deleting through a base pointer calls the right destructor.
- You can bypass virtual dispatch with the scope operator, e.g. `base.Base::getName()`.
- **Early binding** = direct function call, resolved at compile/link time. **Late binding** = resolved at runtime (like calling through a function pointer). Virtual functions use late binding via a **virtual table (vtable)**.
- **Cost:** virtual calls are slower, and every object with a virtual function carries one extra hidden pointer (to the vtable).
- **Pure virtual** (`= 0`) makes a class **abstract** (can't be instantiated). Derived classes must define it or they're abstract too. A pure virtual function *can* still have a body.
- **Interface class** = no member variables, all pure virtual functions. Usually named with a leading `I`.
- **Virtual base class** = a base included only once, however many times it's inherited (diamond problem fix).
- **Object slicing:** assigning a derived object to a base *object* (by value) copies only the base part.
- **Downcasting** with `dynamic_cast` converts a base pointer to a derived pointer. A failed cast returns `nullptr`.
- Easiest way to overload `operator<<` for a class hierarchy: write it once for the base class, and have it call a virtual `print()` member.

## Quiz: spot the bug (programs should print "Derived")

| # | What's wrong |
|---|---|
| 1a | `Base::getName()` isn't `virtual`, so `b.getName()` calls the base version. |
| 1b | Base's `getName()` is non-const, Derived's is `const`. Different signatures, so it isn't an override. |
| 1c | `Base b{ d };` copies by value, so `d` gets **sliced**. Needs a reference or pointer. |
| 1d | `Base` is `final`, so `Derived` can't inherit from it. Compile error. |
| 1e | `Derived::getName()` is pure virtual (even with a body), so `Derived` is abstract and can't be instantiated. |
| 1f | Output is right, but `delete b` through a `Base*` with **no virtual destructor** leaks the Derived part. |

**Coding exercises (short version of the solutions):**

- **2a:** `Shape` is abstract with a pure virtual `print(std::ostream&) const`, a friend `operator<<` that calls `print`, and a `virtual ~Shape() = default;`.
- **2b:** `Triangle` (three `Point`s) and `Circle` (centre `Point` + `int` radius) each override `print` with `override`.
- **2c:** Loop over `std::vector<Shape*>` printing `*element`. For the largest radius, `dynamic_cast<const Circle*>(element)` and only use it if it isn't null. Then `delete` each element.
- **2d (extra credit):** use `std::vector<std::unique_ptr<Shape>>`. You can't build it from an initializer list (that needs copies), so use `push_back(std::make_unique<...>(...))`. Use `.get()` to pass the raw pointer to `dynamic_cast`. No manual `delete` needed.

## My additions

- **Checklist for a polymorphic base class:** virtual destructor, `override` on every override, pure virtual for the "must implement" functions, and pass/store by pointer or reference (never by value).
- **`override` is free safety.** Bugs 1a and 1b above would have been compile errors if `override` had been written on the derived function. Make it a habit.
- **Embedded/hardware angle:** virtual calls cost an extra pointer per object plus an indirect call. On tiny microcontrollers (ESP32, STM32) with small RAM, that matters. Interface classes (like a generic `IDriver` or `ISensor`) are still a clean way to swap hardware backends without changing the calling code.
- **`dynamic_cast` is a design smell if overused.** Needing it a lot often means the base interface is missing a virtual function. It also needs RTTI, which some embedded builds turn off.
- **Slicing + containers:** `std::vector<Base>` slices everything you put in. That's why exercise 2c stores pointers (or smart pointers).

---
[← Prev](ch23-object-relationships.md) | [Index](README.md)
