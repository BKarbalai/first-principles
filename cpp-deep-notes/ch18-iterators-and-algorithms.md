# Chapter 18: Iterators and Algorithms

**Source:** [18.4 Timing your code](https://www.learncpp.com/cpp-tutorial/timing-your-code/) · Site: [learncpp.com](https://www.learncpp.com/)
Related: [18.1 Sorting an array using selection sort](https://www.learncpp.com/cpp-tutorial/sorting-an-array-using-selection-sort/) · [18.3 Introduction to standard library algorithms](https://www.learncpp.com/cpp-tutorial/introduction-to-standard-library-algorithms/)

> **Note:** at the time of writing, learncpp.com has **no "Chapter 18 summary and quiz"** page (the chapter is marked under construction). So the main summary below is of the chapter's last lesson, **18.4 Timing your code**. The "Background" section is my own context for the rest of the chapter (iterators and algorithms), written from general C++ knowledge, not from a learncpp summary page. Check the lessons themselves for the details.

---

## Background: what the chapter is about (my own context)

- **Iterator:** an object that points at an element of a container and can be moved to the next one. It generalizes the `begin`/`end` pointer walk from 17.9. A container's `.begin()` returns an iterator to the first element and `.end()` returns one to **one past the last element** (never dereference `end`).
- **Algorithm:** a ready-made function (in `<algorithm>`/`<numeric>`) that works on a **range** given by a pair of iterators (`begin`, `end`). Examples: `std::sort`, `std::find`, `std::count`, `std::for_each`, `std::iota`, `std::copy`.
- **Why it matters:** the algorithms are written once, work with *any* container, and are heavily optimized and tested. Prefer them to hand-written loops.
- **Selection sort (18.1):** repeatedly find the smallest remaining element and swap it into place. Simple but **O(n²)**: fine for learning, slow on large inputs.

---

## 1. Why time your code (18.4)

When you have two ways to do something, guessing which is faster is unreliable. **Measure.** C++11's `<chrono>` library can do it, but its raw interface is clumsy, so wrap it in a small **Timer class**.

## 2. The Timer class

```cpp
#include <chrono>

class Timer
{
private:
    using Clock  = std::chrono::steady_clock;
    using Second = std::chrono::duration<double, std::ratio<1>>;
    std::chrono::time_point<Clock> m_beg{ Clock::now() };

public:
    void reset() { m_beg = Clock::now(); }
    double elapsed() const
    { return std::chrono::duration_cast<Second>(Clock::now() - m_beg).count(); }
};
```

How it works:
- **`steady_clock`** is a clock that only moves forward (it never jumps back if the system time is adjusted), which is what you want for measuring intervals.
- `m_beg` records the moment the Timer was created (`Clock::now()`).
- `elapsed()` subtracts the start from the current time and converts the **duration** to a `double` number of **seconds**.
- `reset()` restarts the timer.

Usage:
```cpp
Timer t;
// ...code to measure...
std::cout << "Time elapsed: " << t.elapsed() << " seconds\n";
```

## 3. The demo: your sort vs the library's

Sorting an array of **10,000 elements** (filled in descending order with `std::iota` + reverse) gave roughly:

| Method | Time |
|---|---|
| Hand-written selection sort | ≈ 0.05 s |
| `std::sort` / `std::ranges::sort` | ≈ 0.0007 s |

So the standard library sort was about **100× faster**. Moral: **use the library's algorithms**.

## 4. Things that distort your timings

1. **Build configuration.** Time a **release** build (optimizations on), not a debug build. In debug, the same `std::sort` was dramatically slower (~33×), so debug numbers don't represent what you'd ship.
2. **Background activity.** Other programs use CPU, memory and disk. Close what you can, and note timings are only meaningful on your own machine under similar conditions.
3. **Randomness.** Random data changes how much work the algorithm does. For fair comparison, seed with a **fixed literal** temporarily so each run uses identical data (but beware if the real-world speed depends heavily on the particular sequence).
4. **User input.** Never include time spent waiting for a human.

## 5. How to measure well

- Take **at least 3 runs**. If they cluster tightly, that's your number. If not, keep measuring and look for outliers.
- **Absolute times** are specific to your machine; **relative comparisons** (variant A vs B on the same machine) are what's useful.
- **Sanity check:** after timing variant B, **re-time variant A**. If A's time is unchanged, the comparison was fair. If it changed, something about the machine changed in between, so discard and redo.

---

## Quiz

This lesson has no quiz. Self-check questions:
1. Why is a debug-build timing misleading? *(Optimizations are usually off, so you aren't measuring the code you'd ship.)*
2. Why re-run variant 1 after measuring variant 2? *(To confirm the machine's conditions didn't change between measurements.)*
3. Why `steady_clock`? *(It never goes backwards, unlike a wall clock that can be adjusted.)*
4. Why is `std::sort` so much faster than selection sort? *(Roughly O(n log n) vs O(n²).)*

---

## My additions

- **Iterators connect to chapter 17:** the `begin`/`end` pointer-walk from 17.9 *is* the iterator idea. `std::sort(arr.begin(), arr.end())` takes exactly such a pair.
- **Micro-benchmark gotcha:** the compiler can optimize away work whose result is never used. Make sure the timed code's output is used (print or accumulate it).
- **Embedded/hardware version of this lesson:** on a microcontroller you'd time with a hardware timer or cycle counter (e.g. ARM Cortex-M's DWT cycle count) instead of `<chrono>`. Same rules: repeat runs, compare variants, watch for interrupts adding noise.
- **Habit to build:** measure first, optimize second. Don't assume the "clever" version is faster.

---
[← Prev](ch17-fixed-size-arrays.md) | [Index](README.md) | [Next →](ch19-dynamic-allocation.md)
