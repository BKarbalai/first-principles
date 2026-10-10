# Chapter 19: Dynamic Allocation

**Source:** [19.5 Void pointers](https://www.learncpp.com/cpp-tutorial/void-pointers/) · Site: [learncpp.com](https://www.learncpp.com/)
Related: [19.1 Dynamic memory allocation with new and delete](https://www.learncpp.com/cpp-tutorial/dynamic-memory-allocation-with-new-and-delete/) · [19.4 Pointers to pointers and dynamic multidimensional arrays](https://www.learncpp.com/cpp-tutorial/pointers-to-pointers/) · Next chapter: [20.1 Function pointers](https://www.learncpp.com/cpp-tutorial/function-pointers/)

> **Note:** at the time of writing, learncpp.com has **no "Chapter 19 summary and quiz"** page (the chapter is marked under construction). So the main summary below is of the chapter's last lesson, **19.5 Void pointers**. The "Background" section is my own context about the rest of the chapter, written from general C++ knowledge, not from a learncpp summary page.

---

## Background: the chapter in one page (my own context)

- **Three places memory comes from:** *static* (globals, lives the whole program), *automatic/stack* (locals, freed when the scope ends, small), and *dynamic/heap* (you request it at runtime, lives until you free it, large).
- **`new` / `delete`:** `int* p = new int{5};` allocates on the heap and returns its address; `delete p;` frees it. Arrays: `new int[n]{}` / `delete[]`.
- **Every `new` needs exactly one matching `delete`.** Forgetting = **memory leak**. Deleting twice or using after delete = **undefined behavior** (dangling pointer).
- **Why use dynamic allocation:** when the size/lifetime isn't known at compile time or the data is too big for the stack.
- Modern C++ avoids raw `new`/`delete` in favor of **smart pointers** (`std::unique_ptr`, `std::shared_ptr`) and containers (`std::vector`), which free memory automatically.
- **Pointers to pointers** exist (`int**`) and were once used for dynamic 2D arrays. Today you'd flatten to 1D or use `std::vector`.

---

## 1. What a void pointer is (19.5)

A **void pointer** (`void*`, also called a **generic pointer**) is a pointer that can point at an object of **any** data type:

```cpp
int   i{ 5 };
float f{ 2.5f };
void* ptr{};
ptr = &i;     // OK
ptr = &f;     // OK
ptr = &someStruct;   // OK
```

## 2. The catch: you can't dereference it

Since the compiler doesn't know the pointee's type, it doesn't know how many bytes to read or how to interpret them. So **`*ptr` is illegal.** You must first **cast it back** to the correct typed pointer:

```cpp
int* ip{ static_cast<int*>(ptr) };
std::cout << *ip;
```

**It's on you** to remember what the original type was. A common workaround is to pass a **type tag** alongside:

```cpp
enum class Type { tInt, tFloat, tCString };

void printValue(void* ptr, Type type)
{
    switch (type)
    {
    case Type::tInt:     std::cout << *static_cast<int*>(ptr);   break;
    case Type::tFloat:   std::cout << *static_cast<float*>(ptr); break;
    case Type::tCString: std::cout << static_cast<char*>(ptr);   break; // no *: cout prints char* as a string
    }
}
```

## 3. Rules and restrictions

- A void pointer **can be `nullptr`**.
- **`delete` on a `void*` is undefined behavior.** Cast to the right type first.
- **No pointer arithmetic** on `void*` (the compiler doesn't know the element size).
- There is **no such thing as a void reference.**

## 4. Why to avoid them

Void pointers **bypass type checking**, so the compiler can't catch mistakes. For example, `printValue(&someInt, Type::tCString)` compiles fine but is nonsense and crashes or prints garbage.

**Prefer instead:**
- **Function overloading** (one function per type), or
- **Templates** (the compiler generates the right version for each type and still type-checks).

## 5. Where you'll still see `void*`
C libraries and callback APIs that must accept "any user data": e.g. `qsort`'s comparator, thread-creation functions, and RTOS task parameters (FreeRTOS tasks take a `void* pvParameters`). If you do embedded/RTOS work, you'll meet this regularly.

---

## Quiz and answer

**What's the difference between a void pointer and a null pointer?**
A **void pointer** is about the pointer's **type**: it can point to any type but doesn't know which, so it must be cast before dereferencing. A **null pointer** is about the pointer's **value**: it points at nothing. A void pointer *can* also be null. (Type vs value.)

---

## My additions

- **Safer modern alternatives to `void*`:** templates (compile-time type), `std::variant` / `std::any` (type-safe), or a virtual base class (chapter 25).
- **Always pair a `void*` with whatever lets you recover its type** (tag, size, or a callback that knows). A bare `void*` is a bug waiting to happen.
- **Lifetime is the common thread:** chapter 12's dangling references/pointers and chapter 19's `new`/`delete` leaks are the same underlying problem: who owns this memory and when does it die?
- **Embedded angle:** heap allocation on microcontrollers is often avoided (fragmentation, unpredictable timing). Static/stack allocation and fixed-size containers like `std::array` are more common.
- **Quick check:** why can't you do `voidPtr + 1`? (Pointer arithmetic scales by the pointee's size, and `void` has no size.)

---
[← Prev](ch18-iterators-and-algorithms.md) | [Index](README.md) | [Next →](ch23-object-relationships.md)
