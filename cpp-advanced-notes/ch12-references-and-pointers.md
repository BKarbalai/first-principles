# Chapter 12: Compound Types: References and Pointers

**Source:** [12.x Chapter 12 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-12-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Related: [12.1 Introduction to compound data types](https://learncpp.com/cpp-tutorial/introduction-to-compound-data-types) · [12.15 std::optional](https://www.learncpp.com/cpp-tutorial/stdoptional/)

## Summary (condensed)

**Value categories**
- **Compound type** = built from fundamental types or other compound types.
- **lvalue** = expression with an *identity* (a name or an addressable location). Can be modifiable or non-modifiable (const/constexpr).
- **rvalue** = anything that isn't an lvalue: literals (except string literals), values returned by value from functions/operators, temporaries.

**References**
- A reference is an **alias** for an existing object. Anything you do to the reference happens to the referent. Once bound, it **can't be reseated** (`ref = y` copies y's value into the referent).
- **Non-const lvalue reference** binds only to modifiable lvalues.
- **`const` lvalue reference** binds to modifiable lvalues, const lvalues *and* rvalues (temporaries).
- **Dangling reference** = the referent died first. Using it is undefined behavior.
- **Pass by reference** avoids copies. Prefer `const T&` unless the function must modify the argument.
- **Return by reference** avoids a copy, but the referenced object must outlive the function. Returning a reference parameter is safe. If you use the returned reference to initialize a normal variable, it's copied anyway.
- `auto` drops references and top-level `const`. Re-add them (`const auto&`) if you want them.

**Pointers**
- `&` = address-of, `*` = dereference (gives an lvalue).
- A pointer holds a **memory address**. Uninitialized = **wild pointer**. Holding the address of a dead object = **dangling pointer**.
- **`nullptr`** = null pointer literal. Rule: a pointer should hold a valid address or `nullptr`, so you only ever need to null-check.
- **Pointer to const** (`const int*`): pointee can't be changed through it, pointer can be re-pointed.
- **Const pointer** (`int* const`): can't be re-pointed, pointee can change.
- **Const pointer to const** (`const int* const`): neither can change.
- **Pass by address**: the caller passes a pointer, which is copied into the function's pointer parameter. **Return by address** works like return by reference, with the same lifetime rules.

## Quiz and answers

1. lvalue or rvalue?
   - `5` → **rvalue** (literal)
   - `x` → **lvalue** (names a variable)
   - `x + 1` → **rvalue** (temporary result)
   - `foo()` returning `int` → **rvalue**
   - `max(x, y)` returning `int&` → **lvalue** (it returns an lvalue reference)
2. Output of the `ref` program is **`7 6`**. `ref = y` is really `x = y`, so x becomes 6, then `++ref` makes it 7. `y` never changes.
3. Why prefer `const` reference params? (1) A non-const reference lets the function accidentally modify the caller's data. (2) A non-const reference only accepts modifiable lvalues. A const reference accepts modifiable lvalues, const lvalues and rvalues.
4. **Const pointer** = address fixed, value changeable. **Pointer-to-const** = value read-only through it, pointer re-pointable.
5. `sort2(int& lesser, int& greater)`: `if (lesser > greater) std::swap(lesser, greater);` (header `<algorithm>`).

## My additions

- **Read const right to left:** `const int* p` = "p is a pointer to a const int". `int* const p` = "p is a const pointer to an int". Instantly tells you which part is locked.
- **`const&` is the default for function parameters** of anything bigger than a few bytes. Cheap types (`int`, `double`, small enums) just pass by value.
- **Lifetime is the whole game.** References and pointers don't own anything. Both ways of getting UB (dangling reference, dangling pointer) come from the thing they refer to dying first. The ref qualifiers lesson (15.10) is a more subtle version of the same bug.
- **Hardware link:** a pointer to a memory-mapped register is exactly "an address you dereference." Embedded code uses `volatile uint32_t*` for this. Same mechanics as chapter 12, plus `volatile` to stop the compiler optimizing away reads/writes.
- **Quick test for yourself:** which of `const int&`, `int&`, `const int*` can bind to the literal `5` / take its address? (Only `const int&` can bind to `5`. You can't take `&5`.)
