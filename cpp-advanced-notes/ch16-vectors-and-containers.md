# Chapter 16: Dynamic Arrays: std::vector

**Source:** [16.x Chapter 16 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-16-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Lessons you studied: [16.10 std::vector resizing and capacity](https://www.learncpp.com/cpp-tutorial/stdvector-resizing-and-capacity/) · [16.11 std::vector and stack behavior](https://www.learncpp.com/cpp-tutorial/stdvector-and-stack-behavior/) · Also: [16.12 std::vector\<bool\>](https://www.learncpp.com/cpp-tutorial/stdvector-bool/)

## Summary (condensed)

**Containers and arrays**
- **Container** = stores a collection of unnamed **elements**. Number of elements = **length** (also "count" or "size"). Containers are **homogeneous** (all elements same type).
- **Array** = elements stored **contiguously** in memory. Gives fast **random access** (any element in equal time).
- C++ has three array types: C-style arrays, `std::vector`, `std::array`.
- `std::vector<T>` lives in `<vector>`, a class template over the element type.
- **List constructor:** build a container from `{ ... }`. A list constructor beats other matching constructors:
  ```cpp
  std::vector v1 { 5 };  // ONE element with value 5
  std::vector v2 ( 5 );  // FIVE value-initialized elements
  ```
  Use direct initialization `( )` when your numbers aren't element values.
- `std::vector` can be `const` but **not `constexpr`**.

**Indexing and size**
- `operator[]`: zero-based, **no bounds checking** (bad index = UB). `.at()`: bounds-checked, throws `std::out_of_range`.
- Both expect `size_type` (almost always `std::size_t`, unsigned), so non-constexpr signed indices cause sign-conversion trouble.
- Length: `.size()` (unsigned), `std::size()` (C++17), `std::ssize()` (C++20, signed). Scope-qualify the type: `std::vector<int>::size_type`.

**Passing, copying, moving**
- Pass vectors by **(const) reference**. By value makes an expensive copy. Use a function template to accept any element type.
- **Copy semantics** = rules for making copies. **Move semantics** = transferring ownership of data instead of copying. Used automatically when initializing/assigning from an rvalue of a move-capable type. So returning `std::vector`/`std::string` by value is cheap.

**Traversal**
- **Traversal / iterating** = visiting each element. Watch for **off-by-one** errors.
- Prefer **range-based for**. Use `const auto&` unless you need copies (safe even if the element type changes later).
- Unscoped enums make good indices. Add a **count enumerator** (`max_items`) and `assert`/`static_assert` that the array length matches.

**Resizing and capacity** (your lessons 16.10 and 16.11)
- **Fixed-size array:** length set at creation. **Dynamic array:** can change size after creation. That's what makes `std::vector` special.
- `resize(n)` changes **length** (and capacity if needed). `capacity()` = how many elements have storage. **Valid indices depend on length, not capacity.**
- **Reallocation** (new memory, copy/move everything, free old) is expensive. Avoid it. Shrinking only reduces length. `shrink_to_fit()` *requests* capacity = length (non-binding).
- **Stack** = LIFO container with **push** and **pop**. Vector provides `push_back()`, `emplace_back()`, `pop_back()`, `back()`.
- `push_back`/`emplace_back` grow length and reallocate if capacity runs out, usually over-allocating (GCC/Clang ×2, MSVC ×1.5).
- **`resize()` vs `reserve()`:** `resize()` changes length (use with indexing). `reserve()` changes only capacity (use with stack operations).
- `emplace_back(args...)` builds the element in place (no temporary). Prefer it when constructing a new object or needing an explicit constructor. Otherwise prefer `push_back`.

**`std::vector<bool>`**
- Space-optimized specialization packing bools into bits. It's not actually a normal vector (not guaranteed contiguous, holds bits not `bool`s, not a proper container) and doesn't work with everything in the standard library. **Generally avoid.**

## Quiz and answers

1. Definitions (use CTAD where possible):
   - (a) `std::vector evens { 2, 4, 6, 8, 10, 12 };`
   - (b) `const std::vector d { 1.2, 3.4, 5.6, 7.8 };` (can't be constexpr)
   - (c) `const std::vector names { "Alex"sv, "Brad"sv, "Charles"sv, "Dave"sv };` (the `sv` suffix makes CTAD pick `string_view`)
   - (d) `std::vector v { 12 };` (list-init for element values)
   - (e) `std::vector<int> v( 12 );` (direct init for a length; type must be explicit since there's nothing to deduce from)
2. **Inventory** (health potions, torches, arrows = 1, 5, 10):
   - Unscoped `enum Type { health_potion, torch, arrow, max_items }` inside `namespace Items`, `std::vector inventory{1,5,10}`, and `assert(std::size(inventory) == Items::max_items)`.
   - Total = `You have 16 total items`.
   - Step 2: loop over the enum and print singular/plural names (`1 health potion`, `5 torches`, `10 arrows`).
3. **Min/max index function** returning `std::pair<std::size_t, std::size_t>`. Assume element 0 is both, then scan the rest. Results: `{3,8,2,5,7,8,3}` → min index 2 (value 2), max index 1 (value 8). Doubles version → min index 4 (1.2), max index 5 (8.8).
4. **Read integers until -1** with `push_back`, handle bad input with `cin.clear()` + `cin.ignore(...)`, and handle an empty vector ("The array has no elements").
5. **C++man (Hangman)**, four steps:
   1. `WordList` namespace + random word picker.
   2. `Session` class, a `draw()` function and validated `getGuess()`.
   3. Track guessed letters in a `std::vector<bool>(26)`, index by `(c % 32) - 1`.
   4. Add wrong-guess counter, win/lose conditions and output of wrong letters.

## My additions

- **`resize` vs `reserve` in one line:** `resize` = "I want N real elements"; `reserve` = "I'll add elements later, just pre-allocate the room".
- **Common bug:** `std::vector<int> v(3); v.push_back(1);` gives `{0,0,0,1}`, not `{1}`. Parentheses set the *length*.
- **`.at()` vs `[]`:** use `[]` in hot loops where indices are provably valid. Use `.at()` when the index comes from user input or computation.
- **Range-for with `const auto&`** is almost always the right default. Plain `auto` copies every element (cheap for `int`, costly for `std::string`).
- **Embedded angle:** `std::vector` uses heap allocation and reallocation, which many microcontroller projects avoid (fragmentation, unpredictable timing). That's why `std::array` (chapter 17) is the usual embedded choice, or `reserve()` once up front.
- **Why `vector<bool>` is a trap:** `for (auto& b : vecBool)` won't compile because there's no real `bool&` to bind to. Use `std::vector<char>`/`std::bitset` instead.
