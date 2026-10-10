# Chapter 19: Dynamic Allocation

**Source:** [19.5 Void pointers](https://www.learncpp.com/cpp-tutorial/void-pointers/) · Site: [learncpp.com](https://www.learncpp.com/)
Related: [19.1 Dynamic memory allocation with new and delete](https://www.learncpp.com/cpp-tutorial/dynamic-memory-allocation-with-new-and-delete/) · [19.4 Pointers to pointers and dynamic multidimensional arrays](https://www.learncpp.com/cpp-tutorial/pointers-to-pointers/) · Next chapter: [20.1 Function pointers](https://www.learncpp.com/cpp-tutorial/function-pointers/)

> **Note:** at the time of writing, learncpp.com has **no "Chapter 19 summary and quiz"** page (the chapter is marked under construction). So these notes use the chapter's last lesson, **19.5 Void pointers**, instead.

## Summary of 19.5 (condensed)

- A **void pointer** (`void*`, "generic pointer") can point at an object of **any** type:
  ```cpp
  void* ptr{};
  ptr = &someInt;     // OK
  ptr = &someFloat;   // OK
  ptr = &someStruct;  // OK
  ```
- But it doesn't know *what* it points to, so **you can't dereference it**. Cast to a typed pointer first:
  ```cpp
  int* intPtr{ static_cast<int*>(voidPtr) };
  std::cout << *intPtr;
  ```
- **You** are responsible for remembering the real type (e.g. passing a type tag alongside, like an `enum class Type { tInt, tFloat, tCString }` and a `switch` inside `printValue(void*, Type)`).
- `std::cout << static_cast<char*>(ptr)` prints a C-style string (no `*`, because `cout` treats `char*` as a string).
- **Void pointer facts:**
  - Can be set to `nullptr`.
  - **`delete` on a `void*` = undefined behavior.** Cast back to the correct type first.
  - **No pointer arithmetic** (the compiler doesn't know the element size).
  - **No void reference** exists.
- **Verdict:** avoid void pointers unless truly necessary. They bypass type checking, so `printValue(&nValue, Type::tCString)` compiles but is nonsense. Prefer **function overloading** or **templates**, which keep type checking.

## Quiz and answer

**What's the difference between a void pointer and a null pointer?**
A **void pointer** is about the pointer's **type**: it can point to any type but doesn't know which, so it must be cast before dereferencing. A **null pointer** is about the pointer's **value**: it points at nothing. A void pointer *can* be null. Type vs value.

## My additions

- **Where `void*` still shows up for real:** C APIs and callbacks (e.g. `qsort`, `pthread_create`, RTOS task parameters like FreeRTOS's `void* pvParameters`), where one function must accept "any user data". If you do embedded/RTOS work you'll meet this constantly.
- **Safer modern alternatives:** templates (compile-time type), `std::variant` / `std::any` (type-safe sum types), or a virtual base class (chapter 25).
- **Always pair a `void*` with the information needed to recover its type** (tag, size, or callback that knows it). A bare `void*` is a bug waiting to happen.
- **Chapter 19 big picture** (the lessons around this one): `new`/`delete` allocate and free heap memory. Every `new` needs exactly one matching `delete` (or use `std::unique_ptr` so you can't forget). Leaks and dangling pointers are the main risks, same lifetime theme as chapter 12.
- **Quick check:** why can't you do `voidPtr + 1`? (Pointer arithmetic scales by the pointed-to type's size, and `void` has no size.)

---
[← Prev](ch18-iterators-and-algorithms.md) | [Index](README.md) | [Next →](ch23-object-relationships.md)
