# Chapter 16: Dynamic Arrays: std::vector

**Source:** [16.x Chapter 16 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-16-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Lessons you studied: [16.10 std::vector resizing and capacity](https://www.learncpp.com/cpp-tutorial/stdvector-resizing-and-capacity/) · [16.11 std::vector and stack behavior](https://www.learncpp.com/cpp-tutorial/stdvector-and-stack-behavior/) · Also: [16.12 std::vector\<bool\>](https://www.learncpp.com/cpp-tutorial/stdvector-bool/)

---

## 1. Containers and arrays

- A **container** stores a collection of unnamed objects called **elements**. The number of elements is its **length** (also "count" or "size").
- Containers are **homogeneous** (all elements are the same type).
- An **array** is a container whose elements are stored **contiguously** in memory. That gives **random access** (any element reachable in constant time) and good cache behavior.
- C++ has three array types: **C-style arrays**, **`std::array`**, and **`std::vector`**.

---

## 2. Creating a `std::vector`

`std::vector<T>` (header `<vector>`) is a class template; `T` is the element type.

```cpp
std::vector<int> a{};              // empty
std::vector b{ 1, 2, 3, 4 };       // CTAD deduces vector<int>, 4 elements
std::vector<int> c( 5 );           // 5 elements, all 0 (value-initialized)
```

**List constructor:** a constructor that takes an initializer list. It has priority over other matching constructors, which causes the famous gotcha:

```cpp
std::vector v1 { 5 };   // ONE element, value 5   (list constructor)
std::vector v2 ( 5 );   // FIVE elements, all 0   (length constructor)
```
Rule: **use braces when you mean "these are the element values", parentheses when you mean "this many elements".**

Other facts:
- A vector can be `const`, but **not `constexpr`** (use `std::array` for that).
- Prefer `.empty()` to check for no elements.

---

## 3. Accessing elements and getting length

| Operation | Behavior |
|---|---|
| `v[i]` | zero-based, **no bounds checking**. Bad index = undefined behavior |
| `v.at(i)` | **bounds-checked**, throws `std::out_of_range` on a bad index. Slower |

- Valid indices are `0` to `length - 1`, based on **length, not capacity**.
- Both accept **`size_type`** (an unsigned type, almost always `std::size_t`), so signed `int` indices cause sign-conversion warnings when non-constexpr.
- **Length:** `v.size()` (unsigned), `std::size(v)` (C++17), `std::ssize(v)` (C++20, **signed**, good for signed loops).
- The type name is `std::vector<int>::size_type`. You must scope-qualify it.

---

## 4. Passing, copying and moving vectors

- Pass vectors by **(const) reference**. Pass by value copies every element.
- To accept vectors of *any* element type, use a function template: `template <typename T> void print(const std::vector<T>&)`.
- **Copy semantics:** rules for making a copy of an object. Expensive for big containers.
- **Move semantics:** instead of copying, **transfer ownership** of the data to the new object. C++ uses it automatically when initializing/assigning from an **rvalue** of a move-capable type. That's why **returning a `std::vector` or `std::string` by value is cheap**.

---

## 5. Looping over a vector

- **Traversal/iteration** = visiting each element once. The classic bug is the **off-by-one error** (looping one too many or too few).
- **Prefer range-based for loops** (no index, no bounds bug).
- Use **`const auto&`** as the loop variable by default. Plain `auto` copies each element, which is costly for strings and silently wrong if you wanted to modify the originals.

```cpp
for (const auto& name : names) std::cout << name << '\n';
for (auto& x : nums) x *= 2;      // non-const ref to modify in place
```

### Arrays + enumerations
Unscoped enums make convenient indices (`scores[kenny]`). Add a **count enumerator** (`max_students`) at the end and `assert(std::size(scores) == max_students)` so a missing initializer is caught.

---

## 6. Resizing and capacity (lesson 16.10)

- **Fixed-size array:** length set at creation, can't change. **Dynamic array:** length can change after creation. `std::vector` is the dynamic one.
- **`resize(n)`**: sets the **length** to `n`. Growing keeps existing values and **value-initializes** the new ones. Shrinking drops the extras.
- **Length vs capacity:**
  - *Length* = elements currently **in use** (`size()`).
  - *Capacity* = elements it has **allocated storage for** (`capacity()`).
  - Analogy: an egg carton holding 12 (capacity) with 5 eggs in it (length).
- **Reallocation:** when capacity is exceeded, the vector (1) acquires a bigger block, (2) copies/moves every element into it, (3) frees the old block. **Expensive**, so avoid unnecessary reallocations.
- Separating length from capacity lets a vector **shrink and re-grow without reallocating**. `v.resize(3)` on a length-5 vector keeps capacity 5.
- **Shrinking only lowers length, not capacity.** To give memory back, call **`shrink_to_fit()`**, a *non-binding request* the implementation may ignore, partly honor, or fully honor.

---

## 7. Stack behavior (lesson 16.11)

A **stack** is a container where you only add/remove at one end: **last-in, first-out (LIFO)**, like a pile of plates.

| Stack op | `std::vector` function | Notes |
|---|---|---|
| push | `push_back(x)` | adds to the end |
| pop | `pop_back()` | removes the last element, returns `void` |
| peek/top | `back()` | reads the last element, doesn't remove |
| push (alt) | `emplace_back(args...)` | constructs the element in place |

- `push_back`/`emplace_back` **increase length** and reallocate if capacity is insufficient.
- When growth triggers reallocation, vectors **over-allocate** to avoid another reallocation next time. GCC/Clang **double** the capacity; MSVC multiplies by **1.5**.
- The call stack is the stack data structure in action (functions push/pop frames).

### `resize()` vs `reserve()`
- `resize()` changes **length** (and capacity if needed). With stack use it adds fake zero elements: `std::vector<int> s(3); s.push_back(1);` → `{0,0,0,1}`.
- **`reserve(n)`** changes **only capacity**. Length stays 0, so no fake elements.
- **Tip:** use `resize()` when you'll **index**; use `reserve()` when you'll **push/pop**.

### `push_back` vs `emplace_back`
- Pushing an *existing* object: they're equivalent → prefer `push_back`.
- Constructing a *new temporary* to add: `push_back({"a", 2})` builds a temporary then copies it. `emplace_back("a", 2)` forwards the args and builds it **directly inside the vector** (no copy).
- `emplace_back` also uses **explicit** constructors (can be a hazard), and before C++20 doesn't work with aggregate init.
- **Best practice:** prefer `emplace_back` when creating a new object or needing an explicit constructor; otherwise `push_back`.

### The user-input problem it solves
When you don't know how many values the user will enter, **don't guess a max or make them count**. Push each value with `push_back` and let the vector manage its own storage.

---

## 8. `std::vector<bool>` (16.12)

A **space-optimized specialization** that packs `bool`s into individual bits. But it's not really a normal vector: elements aren't actual `bool`s, aren't guaranteed contiguous, and it doesn't work with some standard library features (e.g. you can't bind `bool&` to an element). **Generally avoid** it. Use `std::bitset` or `std::vector<char>` instead.

---

## Quiz and answers

1. **Definitions (use CTAD where possible):**
   - (a) `std::vector evens { 2, 4, 6, 8, 10, 12 };`
   - (b) `const std::vector d { 1.2, 3.4, 5.6, 7.8 };` (can't be constexpr)
   - (c) `const std::vector names { "Alex"sv, "Brad"sv, "Charles"sv, "Dave"sv };` (`sv` suffix → `string_view`)
   - (d) `std::vector v { 12 };` (list-init for element values)
   - (e) `std::vector<int> v( 12 );` (direct init for a length; type explicit since nothing to deduce from)
2. **Inventory** (health potions, torches, arrows = 1, 5, 10): unscoped `enum Type { health_potion, torch, arrow, max_items }` in a namespace, `std::vector inventory{1,5,10}`, and `assert(std::size(inventory) == Items::max_items)`. Total = `You have 16 total items`.
3. **Min/max index function** returning `std::pair<std::size_t, std::size_t>`: assume element 0 is both min and max, scan the rest. `{3,8,2,5,7,8,3}` → min index 2 (value 2), max index 1 (value 8).
4. **Read integers until -1** with `push_back`, handle bad input with `cin.clear()` + `cin.ignore(...)`, handle the empty-vector case.
5. **C++man (Hangman):** (1) `WordList` namespace + random word, (2) `Session` class with `draw()` and validated `getGuess()`, (3) track guessed letters in a `std::vector<bool>(26)` indexed by `(c % 32) - 1`, (4) wrong-guess counter, win/lose conditions, wrong-letter output.

---

## My additions

- **`resize` vs `reserve` in one line:** `resize` = "I want N real elements"; `reserve` = "I'll add elements later, just pre-allocate the room".
- **`.at()` vs `[]`:** use `[]` in hot loops where indices are provably valid; `.at()` when the index comes from user input.
- **Range-for with `const auto&`** is almost always the right default.
- **Embedded angle:** `std::vector` uses the heap and may reallocate, which many microcontroller projects avoid (fragmentation, unpredictable timing). That's why `std::array` (chapter 17) is the usual embedded choice, or `reserve()` once at startup.
- **Why `vector<bool>` is a trap:** `for (auto& b : vecBool)` won't compile. Use `std::vector<char>` or `std::bitset`.

---
[← Prev](ch15-more-on-classes.md) | [Index](README.md) | [Next →](ch17-fixed-size-arrays.md)
