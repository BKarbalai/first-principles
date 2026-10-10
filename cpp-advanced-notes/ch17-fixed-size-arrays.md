# Chapter 17: Fixed-Size Arrays: std::array and C-Style Arrays

**Source:** [17.x Chapter 17 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-17-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Lessons you studied: [17.5 Arrays of references via std::reference_wrapper](https://www.learncpp.com/cpp-tutorial/arrays-of-references-via-stdreference_wrapper/) · [17.6 std::array and enumerations](https://www.learncpp.com/cpp-tutorial/stdarray-and-enumerations/) · [17.7 Introduction to C-style arrays](https://www.learncpp.com/cpp-tutorial/introduction-to-c-style-arrays/) · [17.8 C-style array decay](https://www.learncpp.com/cpp-tutorial/c-style-array-decay/) · [17.9 Pointer arithmetic and subscripting](https://www.learncpp.com/cpp-tutorial/pointer-arithmetic-and-subscripting/) · [17.10 C-style strings](https://www.learncpp.com/cpp-tutorial/c-style-strings/)

---

# Part A: `std::array`

## 1. What it is
- **Fixed-size (fixed-length) array:** length must be known at creation and can't change. `std::array` and C-style arrays are fixed-size. **Dynamic array:** can resize (`std::vector`).
- The length must be a **constant expression** (literal, `constexpr` variable, or unscoped enumerator, not a normal runtime variable).
- `std::array` (header `<array>`) is a template struct: `template<typename T, std::size_t N> struct array;`. `N` is a **non-type template parameter** (a value, not a type).
- It's an **aggregate** (no constructors), so it uses **aggregate initialization**.
- **Make it `constexpr` whenever possible.** If it can't be constexpr, consider `std::vector`.

```cpp
constexpr std::array<int, 5> a{ 9, 7, 5, 3, 1 };
constexpr std::array b{ 9, 7, 5, 3, 1 };      // CTAD (C++17): deduces <int, 5>
```

## 2. CTAD and `static_assert`
CTAD deduces element type and length *from the initializers*. If you accidentally give too few values, the array is silently shorter than intended. Guard with:

```cpp
static_assert(std::size(testScores) == max_students);   // compile error if mismatched
```

## 3. Length and indexing
- **Length:** `.size()` (unsigned), `std::size()` (C++17), `std::ssize()` (C++20, signed). Returned values are constexpr (except calling `.size()` on a `std::array` reference parameter before C++23).
- **Indexing:**
  - `[]`: no bounds check (UB if wrong).
  - `.at()`: runtime bounds check, throws (rarely used because it's slow).
  - **`std::get<I>(arr)`:** index is a template argument, so it's **bounds-checked at compile time**.

## 4. Passing and returning
- Pass by (const) reference. To accept any type and length, use a template:
```cpp
template <typename T, std::size_t N>
void print(const std::array<T, N>& arr);
// C++20: template <typename T, auto N>
```
- **Returning by value** copies the whole array: fine if small. For large arrays, consider an out-parameter (reference) or `std::vector`.

## 5. Brace elision
For arrays whose elements are structs/classes/arrays, extra braces are sometimes needed (`std::array<Point,2> a{{ {1,2}, {3,4} }}`). The compiler can often **elide** the outer braces when you initialize using a list of values with the element type given.

## 6. Arrays of references (17.5)
You **can't** make `std::array<int&, 3>` because:
1. references aren't objects, and
2. array elements must be assignable, but references can't be **reseated**.

**Workaround: `std::reference_wrapper<T>`** (header `<functional>`), a class that acts like a reference but *is* an object.
- `operator=` **reseats** it.
- It **implicitly converts** to `T&` (so printing/passing works).
- `.get()` returns the real `T&`. Use it to modify: `arr[1].get() = 5;` (`arr[1] = 5` is ambiguous).
- `std::ref(x)` / `std::cref(x)` are shorthand constructors (deduce the type, `cref` makes a const wrapper). C++17 CTAD also allows `std::reference_wrapper r{ x };`.
- Trap: `std::array valarr{ ref1, ref2 };` is **not** an array of references. It's `std::array<int, 2>` of copied values.

## 7. `std::array` with enumerations (17.6)
1. **Enum → name table:** if enumerators run 0, 1, 2…, store names in a parallel `constexpr std::array` and index it by the enumerator. Guard with `static_assert(std::size(names) == max_colors)`.
2. **Name → enum:** loop over the names array; the matching **index is the enumerator's value** (cast it back).
3. Plug these into `operator<<` and `operator>>` so the enum prints/reads naturally with `cout`/`cin`.
4. **Iterating an enum:** `for (auto c : Color::Type)` is illegal. Instead make a `constexpr std::array types{ black, red, blue };` and range-for over that.

---

# Part B: C-style arrays

## 8. Basics (17.7)
- Built into the language (no `#include`). The standard containers are typically built *on top of* them.
- Declared `T name[N];` with `N` a constant expression ≥ 1. `int a[5];`
- **Subscripting** works with **signed or unsigned** integers and unscoped enums (no sign-conversion issues). **No bounds checking.**
- **Initialization:** aggregate init `int p[] { 2, 3, 5, 7, 11 };`
  - No initializer → **uninitialized garbage**. Always value-initialize: `int a[5]{};`
  - Too many initializers → error. Too few → the rest are zeroed.
  - **Omit the length** when initializing every element (compiler counts for you).
  - **No `auto`/CTAD** for C-style arrays. Write the element type.
- Can be `const`/`constexpr`.
- **`sizeof(arr)`** = total bytes (e.g. 20 for five 4-byte ints). **No overhead:** the array holds elements and nothing else.
- **Length:** `std::size(arr)`, `std::ssize(arr)`. (Old trick: `sizeof(arr)/sizeof(arr[0])`, which breaks silently once decayed.)
- **Can't be assigned wholesale** (`arr = {1,2,3}` is an error). Assign elementwise or use `std::copy`. Prefer `std::vector` if you need that.

## 9. Array decay (17.8)

**Problem the C designers faced:** (1) copying a huge array into every function call is expensive and C had no references; (2) they wanted one function to accept arrays of **any** length, with no templates.

**Solution: decay.** In most expressions, a C-style array automatically converts to a **pointer to its first element** (`int[5]` → `int*`).

- Passing an array to a function really passes its **address**. It looks like pass-by-value but isn't. The function can modify the original, so use `const` if it shouldn't.
- `int[5]` and `int[7]` are different types, but both decay to `int*`, so one function accepts both.
- **What's lost:** the **length**. A decayed pointer doesn't know how many elements follow it.
- **When it does NOT decay:** under `sizeof`/`typeid`, with `&arr`, as a class member, or when passed **by reference**.
- An array is **not** a pointer. It's a sequence of elements. Decay is just an implicit conversion.
- **Parameter syntax:** `void f(const int arr[])` is treated exactly as `const int*`. Use the `[]` form to signal "this is meant to be an array". Any number inside the brackets is ignored.
- `arr[2]` is really `*(decayed_ptr + 2)`. You can also subscript a plain pointer.

**Dangers of decay:**
1. `sizeof` gives the **pointer's size** inside a function, so `sizeof(arr)/sizeof(arr[0])` silently gives the wrong length. `std::size()` instead **fails to compile** on a pointer (much safer).
2. Refactoring is fragile: code that works on a real array may break or misbehave on a decayed one.
3. You can't check the array's length in the function, so too-short arrays (or a pointer to a single int) cause UB.

**Old workarounds:** pass the length as a second parameter (nothing guarantees they match), or use a **sentinel/terminator** value (how C-strings use `'\0'`; breaks if the terminator is missing).

**Verdict:** avoid C-style arrays where practical. Use `std::string_view` (read-only text), `std::string` (modifiable text), `std::array` (non-global constexpr arrays), `std::vector` (non-constexpr arrays). Still fine for **global constexpr tables**, and for functions that deliberately take raw C-strings.

## 10. Pointer arithmetic and subscripting (17.9)
- `ptr + 1` = address of the **next object** (scaled by `sizeof` the pointee), not the next byte. Also `-`, `++`, `--`. Results are only well-defined inside the array or one-past-the-end.
- **`ptr[n]` ≡ `*(ptr + n)`.** That's also why arrays are **zero-based**: index 0 needs no arithmetic.
- Indices are **relative** to wherever the pointer currently points. After `ptr = &arr[3]`, `ptr[1]` is element 4 and `ptr[-1]` is element 2 (negative indices are legal).
- **Best practice:** use `[]` when indexing from the array start; use pointer arithmetic for relative positioning.
- **Traversal with begin/end pointers:**
```cpp
const int* begin{ arr };
const int* end{ arr + std::size(arr) };   // ONE PAST the last element (never dereference it)
for (; begin != end; ++begin) std::cout << *begin << ' ';
```
This sidesteps decay (pass `begin` and `end` instead of the array) and is the basis of **iterators**. **Range-based for loops expand to exactly this pattern.**

## 11. C-style strings (17.10)
- A C-style string is just a `char`/`const char` array ending with a **null terminator** `'\0'`. `"hello"` has type `const char[6]`.
- Prefer omitting the length so the terminator space is counted for you.
- They decay to `const char*`/`char*`. The null terminator is the **replacement for the lost length**.
- **Output:** `cout` prints characters until it hits `'\0'`. A missing terminator causes UB (prints garbage until a zero byte).
- **Input:** `cin >> buf` can **overflow the buffer** (security issue; C++20 forbids it on decayed arrays). Use `cin.getline(buf, std::size(buf))`. Best of all: use `std::string`.
- Can't be reassigned wholesale (`str = "rope"` is an error), but `str[1] = 'p'` works.
- **Length:** `std::size(str)` gives the *array* size; **`std::strlen(str)`** (`<cstring>`) gives the string length (excludes terminator), works on decayed pointers, but is slow (walks the string).
- Other `<cstring>` functions (`strcpy`, `strcat`, `strcmp`...): avoid except `strlen`.
- **Avoid non-const C-style string objects.** Use `std::string`.

---

# Part C: Multidimensional arrays
**Dimension** = number of indices needed to pick an element. An array of arrays is two-dimensional. **Flattening** reduces dimensions (often to 1D indexing `row * cols + col`). C++23's `std::mdspan` provides a multidimensional view over contiguous data.

---

## Quiz and answers

1. **What's wrong?**
   - (a) Loop uses `<=`, reading index 4 of a 4-element array (off-by-one). Use `<`.
   - (b) `int array[]` as a function parameter **decays to a pointer**, so range-based-for can't work. Use `std::array`.
   - (c) `std::array<int, length>` needs a **constant expression**, but `length` is runtime. Use `std::vector`.
2. **Roscoe's potion emporium** (3 steps): `Potion` namespace with enum + `constexpr` parallel arrays for names/costs (guarded by `static_assert`) → `Player` class (name, random gold 80-120, `std::array` inventory) → purchase loop with validated input (`q` to quit), `buy()` returning false if unaffordable, then print inventory.
3. **Card and Deck** (4 steps): `Rank`/`Suit` enums → `struct Card` with `operator<<` (e.g. `JS`), `value()` (ace = 11), `static constexpr` arrays of all ranks/suits → `Deck` (`std::array<Card, 52>`, `dealCard()` with an index and `assert`, `shuffle()` using `std::shuffle` + `Random::mt`, resets the index).
4. **Simplified Blackjack:** `Player { int score }`, `Settings` namespace (`bust = 21`, `dealerStopsAt = 17`), `playerTurn()` (hit/stand), `dealerTurn()` (draw until ≥ 17), `playBlackjack()` returns the winner.
5. **Extensions:** *Aces 1 or 11:* track an ace counter; if score > 21 and counter > 0, subtract 10 and decrement. *Ties:* return an `enum class GameResult { playerWon, dealerWon, tie }`.

---

## My additions

- **Pick the right array:**
  | Situation | Use |
  |---|---|
  | Size known at compile time, constexpr data | `constexpr std::array` |
  | Size changes / known only at runtime | `std::vector` |
  | Global read-only lookup table | C-style `constexpr` array is fine |
  | Text you don't modify | `std::string_view` |
- **Decay in one sentence:** passing a C-style array to a function hands over only a pointer, so `sizeof` inside the function gives pointer size and the length is gone.
- **Why `ptr[n]` works on pointers:** it's literally `*(ptr + n)`. Range-for over an array is a `begin`/`end` pointer walk, the foundation of iterators (chapter 18).
- **Enum-indexed parallel arrays + `static_assert`** is a clean way to build lookup tables. Very common in embedded code (state-name tables, register maps).
- **Embedded angle:** `constexpr std::array` data can live in flash with zero runtime cost. A `std::vector` allocates on the heap. On a microcontroller, prefer `std::array`.
- **Self-test:** why does `std::size(ptr)` fail to compile on a decayed array? (A pointer carries no length, so the compiler refuses rather than silently giving a wrong answer.)

---
[← Prev](ch16-vectors-and-containers.md) | [Index](README.md) | [Next →](ch18-iterators-and-algorithms.md)
