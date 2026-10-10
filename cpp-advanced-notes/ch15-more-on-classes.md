# Chapter 15: More on Classes

**Source:** [15.x Chapter 15 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-15-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Lessons you studied: [15.2 Classes and header files](https://www.learncpp.com/cpp-tutorial/classes-and-header-files/) · [15.8 Friend non-member functions](https://www.learncpp.com/cpp-tutorial/friend-non-member-functions/) · [15.9 Friend classes and friend member functions](https://www.learncpp.com/cpp-tutorial/friend-classes-and-friend-member-functions/) · [15.10 Ref qualifiers](https://www.learncpp.com/cpp-tutorial/ref-qualifiers/)

---

## 1. The hidden `this` pointer

Every non-static member function has a hidden first parameter: `this`, a **const pointer to the implicit object** the function was called on. That's how `m_x` inside a member function knows *whose* `m_x` it means. The compiler rewrites `obj.func()` into something like `func(&obj)` behind the scenes.

You rarely write `this` explicitly. Two common reasons to:
1. To disambiguate a parameter from a member with the same name.
2. To **return `*this`** so calls can be chained.

```cpp
class Calc {
    int m_value{};
public:
    Calc& add(int v) { m_value += v; return *this; }
    Calc& sub(int v) { m_value -= v; return *this; }
};
Calc c;
c.add(5).sub(2).add(10);   // member function chaining
```
Return by **reference** (`Calc&`) so each call modifies the same object instead of a copy. Also, **reset the class to default state** by assigning a default-constructed object: `*this = {};`.

---

## 2. Classes and header files (15.2)

As classes get bigger, cramming member bodies into the class definition clutters the *interface*. C++ lets you split **declaration** from **definition**:

```cpp
// Date.h
class Date {
    int m_year{}, m_month{}, m_day{};
public:
    Date(int y, int m, int d);          // declaration
    void print() const;                 // declaration
    int getYear() const { return m_year; }   // trivial: stays inline
};

// Date.cpp
#include "Date.h"
Date::Date(int y, int m, int d) : m_year{y}, m_month{m}, m_day{d} {}
void Date::print() const { std::cout << m_year << '/' << m_month << '/' << m_day; }
```

- `Date::` is the **scope resolution prefix** that says "this belongs to Date".
- **Convention:** class definition → `Date.h` (with header guards). Non-trivial member functions → `Date.cpp` (and that `.cpp` must be compiled/linked into the program).
- Compiler needs the **full class definition** (not just a forward declaration) to use a class, because it needs to know member declarations and the object's size.
- **ODR details:**
  - Types are exempt from the one-definition-per-program rule, so including a class in many files is fine. Including it twice in *one* file is not (header guards fix that).
  - Member functions defined **inside** the class are implicitly `inline` → ODR-safe. Defined **outside** in a header, they are *not* inline → need an explicit `inline` or you get multiple-definition link errors. Putting them in a `.cpp` avoids the issue.
- **Why not everything in the header?** (1) Clutter. (2) Any edit forces recompiling *every* file that includes the header; editing a `.cpp` only recompiles that file.
- **Reasonable exceptions:** small one-file classes, tiny stable classes, header-only libraries, and **template classes** (the compiler must see the full definition to instantiate).
- **Default arguments** for member functions always go **in the class definition**.
- **Libraries:** `#include <string>` gives you declarations; the implementation is precompiled and linked automatically.

---

## 3. Nested types (member types)

A class can define types inside itself: enums, structs, type aliases. They're accessed through the class name, e.g. `Monster::Type`. Good when the type only makes sense with that class (like a `Fruit` class with a `Fruit::Type` enum).

---

## 4. Class templates and member functions

Member functions of a class template can be defined **inside** the class (template parameters are in scope) or **outside**, in which case you must repeat the template header:

```cpp
template <typename T>
class Box { T m_val{}; public: void set(T v); };

template <typename T>
void Box<T>::set(T v) { m_val = v; }   // outside: repeat template<...> and use Box<T>::
```
Out-of-class template member definitions generally go in the **header**, below the class, because the compiler needs the full template to instantiate it.

---

## 5. Static members

**Static member variable:** shared by **all objects** of the class, exists even with zero objects. Declared with `static`, accessed as `ClassName::member`.
- Declaring it `inline` (or `constexpr`) lets you initialize it right in the class: `static inline int s_count{ 0 };`
- Naming convention: `s_` prefix.

**Static member function:** can be called with **no object** (`ClassName::func()`).
- Has **no `this` pointer**, so it can't touch non-static members. It *can* use static ones.

```cpp
class IdGen {
    static inline int s_next{ 1 };
public:
    static int next() { return s_next++; }
};
IdGen::next();
```

---

## 6. Friend functions and friend classes (15.8, 15.9)

**Problem:** sometimes a function or class needs *private* access but doesn't belong inside the class (e.g. a function that equally uses two classes, or a Display class kept separate from a Storage class).

**Solution:** the class itself grants access with a **`friend` declaration**. A friend has full access to private and protected members, as if it were a member. **Friendship is granted by the class being accessed**, never claimed by the outsider.

```cpp
class Accumulator {
    int m_value{};
public:
    void add(int v) { m_value += v; }
    friend void print(const Accumulator& a);   // friend declaration
};
void print(const Accumulator& a) { std::cout << a.m_value; }   // non-member with private access
```

Details:
- A friend function defined **inside** the class is still a **non-member** function.
- A function can be a friend of **several classes** (needs a **forward declaration** of the later class).
- **Friend class:** `friend class Display;` gives *all* of Display's members access. This declaration doubles as a forward declaration.
- Friendship is **not reciprocal** (A friends B ≠ B friends A), **not transitive** (A→B→C ≠ A→C), and **not inherited**.
- **Friend member function:** friend just one function of another class: `friend void Display::show(const Storage&);`. This needs the full definition of `Display` first, and `Display::show` takes a `Storage&`, so you need a forward declaration of `Storage` and must define `show()` *after* `Storage`. Header/source files make this painless.
- Does friendship violate data hiding? No. The class chose to grant access, so the friend is effectively an extension of the class.
- **Prefer non-friend functions** that use the public interface. Friends depend on your private names, so renames ripple outward. If you must be a friend, still use the public interface where you can.

---

## 7. Ref qualifiers (15.10, optional lesson)

A getter that returns a reference to a member is efficient, but dangerous when called on a **temporary** (rvalue) object:

```cpp
const std::string& ref{ createEmployee("Garbo").getName() };   // temp destroyed -> dangling ref
```
- Using the returned reference **in the same expression** (e.g. printing right away) is fine.
- Saving it for later is undefined behavior.
- **Returning by value** is always safe but wastes a copy in the common lvalue case.
- **Ref qualifiers** (`&` and `&&` after the member function's parameter list) let you write **two overloads**: one for lvalue objects (return a const reference) and one for rvalue objects (return by value / a moved value), so the compiler picks the safe one automatically.

---

## Quiz: random monster generator (what each step practices)

| Step | Task | Concept |
|---|---|---|
| a | `enum class MonsterType` with a trailing `maxMonsterTypes` | scoped enum + count trick |
| b | `Monster` class with type, name, roar, hit points | private members, `m_` prefix |
| c | Make the enum a **nested unscoped** `Monster::Type` | nested types |
| d | Constructor taking all four members (`string_view` params) | member initializer list |
| e | `getTypeString()` (constexpr, `switch`) and `print()` | const member functions |
| f | `MonsterGenerator` as a **namespace** (no state) with `generate()` | namespace vs class |
| g | Add `getName(int)` and `getRoar(int)` for 0-5 | helper functions |
| h | Use `Random::get()` for type, name, roar, and 1-100 hit points | `static_cast<Monster::Type>(Random::get(0, maxMonsterTypes-1))` |

Expected output looks like: `Mort the zombie has 61 hit points and says *growl*.` A monster with 0 hit points prints `<name> the <type> is dead.`

---

## My additions

- **Declaration vs definition cheat sheet:** class body = declaration (menu), `.cpp` = definition (recipe).
- **Friend = last resort.** Order of preference: use the public interface → add a sensible access function → only then use `friend`.
- **Good friend use cases:** functions that equally use two classes (`printWeather(Temperature, Humidity)`), tightly-coupled helper classes, and operator overloading (chapter 21).
- **Static members and hardware:** a `static` counter or shared config in a driver class is a common way to track "how many instances of this peripheral are open".
- **Ref qualifier takeaway:** returning `const T&` from a getter is normal and fine on named objects. The danger is only *saving* that reference from a **temporary**.
- **Check yourself:** why can't a `static` member function read a normal data member? (No `this` pointer, so no object to read it from.)

---
[← Prev](ch14-intro-to-classes.md) | [Index](README.md) | [Next →](ch16-vectors-and-containers.md)
