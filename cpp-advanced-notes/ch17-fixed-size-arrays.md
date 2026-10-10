# Chapter 17: Fixed-Size Arrays: std::array and C-Style Arrays

**Source:** [17.x Chapter 17 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-17-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Lessons you studied: [17.5 Arrays of references via std::reference_wrapper](https://www.learncpp.com/cpp-tutorial/arrays-of-references-via-stdreference_wrapper/) · [17.6 std::array and enumerations](https://www.learncpp.com/cpp-tutorial/stdarray-and-enumerations/) · [17.7 Introduction to C-style arrays](https://www.learncpp.com/cpp-tutorial/introduction-to-c-style-arrays/) · [17.8 C-style array decay](https://www.learncpp.com/cpp-tutorial/c-style-array-decay/) · [17.9 Pointer arithmetic and subscripting](https://www.learncpp.com/cpp-tutorial/pointer-arithmetic-and-subscripting/) · [17.10 C-style strings](https://www.learncpp.com/cpp-tutorial/c-style-strings/)

## Summary (condensed)

**`std::array`**
- **Fixed-size**: length known at creation, never changes (`std::array`, C-style arrays). **Dynamic**: resizable at runtime (`std::vector`).
- Length must be a **constant expression** (literal, constexpr variable, or unscoped enumerator).
- It's an **aggregate** (no constructors, aggregate initialization). Declared as `template<typename T, std::size_t N> struct array;` where `N` is a *non-type template parameter*.
- **Make it `constexpr` whenever possible.** If it can't be, consider `std::vector`.
- **CTAD** deduces element type and length from the initializers. Pair it with `static_assert(std::size(arr) == expected)` to catch missing initializers.
- **Length:** `.size()` (unsigned), `std::size()` (C++17), `std::ssize()` (C++20, signed). Returned as constexpr except when called on a `std::array` passed by reference (fixed in C++23).
- **Indexing:** `[]` (no bounds check), `.at()` (runtime check, usually avoided), `std::get<I>(arr)` (**compile-time** bounds check).
- **Passing to functions:** `template <typename T, std::size_t N>` (or `template <typename T, auto N>` in C++20).
- **Returning by value** copies the whole array: fine if small, otherwise consider an out-parameter.
- **Brace elision:** initializing a `std::array` of structs/classes/arrays without naming the element type may need an extra pair of braces.
- **Arrays of references:** not allowed (references aren't objects, can't be reseated). Use `std::reference_wrapper<T>`: `=` reseats, implicitly converts to `T&`, `.get()` gives `T&`. `std::ref()`/`std::cref()` are shorthand.

**C-style arrays**
- Built into the language, no `#include`. Declared `T name[N]`. Length must be constant expression.
- Aggregate-initialized. Prefer omitting the length when initializing all elements: `int a[] { 1, 2, 3 };`.
- Index with **signed or unsigned** integers or unscoped enums (no sign-conversion problem). No bounds check.
- Can be `const`/`constexpr`. **Can't be assigned wholesale.**
- Length: `std::size()` (C++17) or `std::ssize()` (C++20).
- **Array decay:** in most expressions a C-style array converts to a pointer to its first element, losing length info. Passed to functions by address. Doesn't decay under `sizeof`, `typeid`, `&`, as a class member, or when passed by reference.
- **Pointer arithmetic:** `ptr + 1` = next *object* (scaled by type size). `ptr[n]` ≡ `*(ptr + n)`. Use `[]` from element 0, pointer arithmetic for relative positioning.
- **C-style strings** = `char`/`const char` arrays, null-terminated, and they decay like any array.

**Multidimensional arrays**
- **Dimension** = number of indices needed. Array of arrays = 2D. **Flattening** = reducing dimensions (often to 1D). C++23's `std::mdspan` is a multidimensional view over contiguous data.

## Quiz and answers

1. **What's wrong?**
   - (a) Loop uses `<=`, which reads index 4 of a 4-element array (off-by-one). Use `<`.
   - (b) `int array[]` as a function parameter **decays to a pointer**, so range-based-for can't work. Use `std::array` (doesn't decay).
   - (c) `std::array<int, length>` needs a **constant expression**, but `length` is read at runtime. Use `std::vector`.
2. **Roscoe's potion emporium** (three steps):
   1. `Potion` namespace with unscoped `enum Type`, a `constexpr std::array types` of enumerators, and parallel `name`/`cost` arrays guarded by `static_assert`.
   2. `Player` class with name, gold (random 80-120 via `Random::get`), and an inventory `std::array<int, max_potions>`.
   3. Purchase loop with validated input (`q` to quit, reject extraneous input like `2d`), `buy()` returning false if unaffordable, then print inventory.
3. **Card and Deck** (four steps): `Rank` and `Suit` enums → move into `struct Card` → add `operator<<` (e.g. `JS`), `value()` (ace = 11) and `static constexpr` arrays `allRanks`/`allSuits` → `Deck` class (`std::array<Card, 52>`, `dealCard()` with an index tracker and `assert`, `shuffle()` using `std::shuffle` with `Random::mt`, which also resets the index).
4. **Simplified Blackjack**: `Player { int score }`, `Settings` namespace (`bust = 21`, `dealerStopsAt = 17`), `playerTurn()` (hit/stand loop), `dealerTurn()` (draw until ≥ 17), `playBlackjack()` returns who won.
5. **Extensions:**
   - **Aces as 1 or 11:** track an ace counter in `Player`. If the score goes over 21 and the counter is > 0, subtract 10 and decrement the counter (repeat as needed).
   - **Ties:** return an `enum class GameResult { playerWon, dealerWon, tie }` instead of a `bool`.

## My additions

- **Pick the right array (cheat sheet):**
  | Situation | Use |
  |---|---|
  | Size known at compile time, constexpr data | `constexpr std::array` |
  | Size changes or known only at runtime | `std::vector` |
  | Global read-only lookup table | C-style `constexpr` array is fine |
  | Text you don't modify | `std::string_view` |
- **Decay in one sentence:** passing a C-style array to a function hands over *only* a pointer, so `sizeof` inside the function gives pointer size and the length is gone.
- **Why `ptr[n]` works on pointers:** it's literally `*(ptr + n)`. Range-based for over an array is just a `begin`/`end` pointer walk, which is the basis of iterators (chapter 18).
- **Enum-indexed parallel arrays + `static_assert`** (the potion shop pattern) is a clean way to build lookup tables. It's very common in embedded code (state-name tables, register maps).
- **Embedded angle:** `constexpr std::array` data lives in flash with zero runtime cost. A `std::vector` allocates on the heap. On a microcontroller, prefer `std::array`.
- **Self-test:** why does `std::size(ptr)` fail to compile on a decayed array? (A pointer carries no length, so the compiler refuses instead of silently giving a wrong answer, unlike the old `sizeof(a)/sizeof(a[0])` trick.)

---
[← Prev](ch16-vectors-and-containers.md) | [Index](README.md) | [Next →](ch18-iterators-and-algorithms.md)
