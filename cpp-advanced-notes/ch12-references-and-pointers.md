# Chapter 12: Compound Types: References and Pointers

**Source:** [12.x Chapter 12 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-12-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Related: [12.1 Introduction to compound data types](https://learncpp.com/cpp-tutorial/introduction-to-compound-data-types) · [12.15 std::optional](https://www.learncpp.com/cpp-tutorial/stdoptional/)

---

## 1. Compound types and value categories

A **compound (composite) data type** is a type built out of fundamental types or other compound types. References, pointers, arrays, structs and classes are all compound types.

Before references make sense, you need to know that every **expression** has a **value category** that says whether it resolves to a value, a function, or an object.

- **lvalue** = an expression that evaluates to an object or function that has an **identity**. "Identity" means it has a name or an addressable memory location.
  - **Modifiable lvalue:** you can change it (`int x{5};` → `x`).
  - **Non-modifiable lvalue:** you can't (`const int y{5};` → `y`).
- **rvalue** = anything that is not an lvalue. That includes literals (except string literals), results of arithmetic (`x + 1`), and values returned **by value** from a function. Think "a temporary value with no home".

```cpp
int x{ 5 };
x;        // lvalue (has a name and address)
5;        // rvalue (literal)
x + 1;    // rvalue (temporary result)
```

Why it matters: references only bind to certain categories, and that's where most beginner compile errors come from.

---

## 2. Lvalue references

A **reference** is an **alias** (another name) for an existing object. Anything you do through the reference is done to the original object.

```cpp
int x{ 5 };
int& ref{ x };   // ref is an alias for x
ref = 10;        // x is now 10
```

Key terms and rules:
- When a reference is initialized with an object, it is **bound** to it. The object is the **referent**.
- **References must be initialized.** There is no "empty" reference.
- A reference can **never be reseated** (pointed at a different object). This is the trap in the quiz: `ref = y;` does *not* make `ref` refer to `y`, it copies `y`'s value into whatever `ref` already aliases.
- A **non-const lvalue reference** (`int&`) can only bind to **modifiable lvalues**. It can't bind to a const object (that would let you modify it) or an rvalue (nothing to modify).

```cpp
int x{ 4 }, y{ 6 };
int& ref{ x };
ref = y;        // same as x = y; ref still aliases x
```

### Dangling references
If the referent is destroyed before the reference, the reference is **dangling**: it refers to an object that no longer exists. Using it is **undefined behavior**. This is the single most important danger of references.

---

## 3. References to const

Adding `const` makes a reference that treats its referent as read-only:

```cpp
const int& cref{ x };   // can read x, can't modify through cref
```

A **const lvalue reference** is far more flexible than a non-const one. It can bind to:
1. modifiable lvalues,
2. non-modifiable (const) lvalues, **and**
3. **rvalues** (like literals and temporaries).

When it binds to a temporary, the temporary's lifetime is **extended** to match the reference. A **temporary (anonymous / unnamed) object** is one created for use within a single expression and then destroyed.

```cpp
const int& r{ 5 };   // OK: 5 becomes a temporary whose life is extended
int& bad{ 5 };       // ERROR: non-const ref can't bind to an rvalue
```

---

## 4. Pass by reference

When a function parameter is a reference, it binds to the argument when called. **No copy is made.**

```cpp
void print(const std::string& s);   // no copy of the string
void sort2(int& a, int& b);         // can modify the caller's variables
```

Two reasons to prefer **`const&`** over plain `&` when you don't need to modify:
1. A non-const reference lets the function accidentally change the caller's data.
2. A non-const reference only accepts modifiable lvalues. A const reference accepts lvalues (const or not) *and* rvalues.

Rule of thumb: cheap types (`int`, `double`, `char`, enums) → pass by value; expensive-to-copy types (`std::string`, vectors, big structs) → pass by `const&`.

---

## 5. Pointers

An **address-of operator** `&` gives you the memory address of its operand. The **dereference operator** `*` takes an address and gives you the object there (as an lvalue).

A **pointer** is an object whose value is a **memory address** (usually of another object).

```cpp
int x{ 5 };
int* ptr{ &x };     // ptr holds x's address
std::cout << *ptr;  // dereference: prints 5
*ptr = 7;           // x is now 7
```

Pointers differ from references: they **can be reseated** (re-pointed), they **can be null**, and they need explicit `&`/`*`.

### Uninitialized and dangling pointers
- A pointer that hasn't been initialized is a **wild pointer** (holds a garbage address). Never dereference it.
- A **dangling pointer** holds the address of an object that no longer exists (same lifetime problem as dangling references).

### Null pointers
A pointer can hold a special **null value** meaning "points at nothing". The literal for it is **`nullptr`**.

```cpp
int* p{ nullptr };
if (p) { /* only runs if p is non-null */ }
```

**Best practice:** a pointer should either hold the address of a valid object, or be `nullptr`. That way, "non-null" reliably means "safe to use" and you only ever need to check for null.

---

## 6. Const and pointers (three flavors)

Read the declaration right-to-left.

| Declaration | Name | Can change pointee through it? | Can re-point it? |
|---|---|---|---|
| `const int* p` | pointer to const | No | Yes |
| `int* const p` | const pointer | Yes | No |
| `const int* const p` | const pointer to const | No | No |

---

## 7. Pass by address

Instead of passing an object, the caller passes its **address**. The pointer is copied into the function's pointer parameter, and the function dereferences it to reach the original.

```cpp
void increment(int* p) { if (p) ++(*p); }
int x{ 1 };
increment(&x);   // x becomes 2
```

Use it when "no object" is a valid input (pass `nullptr`), or when the pointer itself matters. Check for null before dereferencing.

---

## 8. Return by reference / return by address

**Return by reference** returns a reference to an existing object, avoiding a copy.
- **Major caveat:** the referenced object must **outlive** the function. Returning a reference to a local variable gives a dangling reference.
- It's safe to return a reference parameter (the caller's object already exists).
- If you use a returned reference to initialize a *non-reference* variable, the value is copied anyway.

**Return by address** is the same idea with a pointer, and it carries the same lifetime rules.

---

## 9. `auto` and references

Type deduction with `auto` **drops references and top-level const**. Re-add them explicitly if you want them:

```cpp
const std::string& getName();
auto a = getName();          // std::string (a copy!)
const auto& b = getName();   // const std::string& (no copy)
```

---

## 10. std::optional (lesson 12.15, bonus)

`std::optional<T>` is an alternative to magic return values or null pointers when a function might not have a result. It either holds a `T` or holds nothing. Check with `.has_value()` or `if (opt)`, read with `*opt` or `.value()`.

---

## Quiz and answers

1. **lvalue or rvalue?**
   - `5` → **rvalue** (literal)
   - `x` → **lvalue** (names a variable)
   - `x + 1` → **rvalue** (temporary result)
   - `foo()` returning `int` → **rvalue**
   - `max(x, y)` returning `int&` → **lvalue** (returns an lvalue reference)
2. Output is **`7 6`**. `ref = y` is really `x = y`, so x = 6, then `++ref` makes x = 7. `y` never changes.
3. **Why prefer const-ref params?** (1) Prevents accidental modification of the argument. (2) A const ref accepts modifiable lvalues, const lvalues *and* rvalues; a non-const ref only accepts modifiable lvalues.
4. **Const pointer vs pointer-to-const:** const pointer = address fixed, pointee changeable. Pointer-to-const = pointee read-only through it, pointer re-pointable.
5. `sort2(int& lesser, int& greater)`: `if (lesser > greater) std::swap(lesser, greater);` (header `<algorithm>`).

---

## My additions

- **Read const right-to-left:** `const int* p` = "p is a pointer to const int". `int* const p` = "p is a const pointer to int".
- **`const&` is the default for function parameters** of anything bigger than a few bytes.
- **Lifetime is the whole game.** References and pointers don't own anything. Both kinds of dangling come from the target dying first. Chapter 15's ref qualifiers lesson is a subtler version of the same bug.
- **Hardware link:** a pointer to a memory-mapped register is literally "an address you dereference". Embedded code uses `volatile uint32_t*` for this: same mechanics, plus `volatile` so the compiler doesn't optimize away reads/writes.
- **Self-test:** which of `const int&`, `int&` can bind to the literal `5`? (Only `const int&`.) Can you write `&5`? (No, it's an rvalue, so it has no address.)

---
[Index](README.md) | [Next →](ch13-enums-and-structs.md)
