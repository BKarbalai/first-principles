# Chapter 18: Iterators and Algorithms

**Source:** [18.4 Timing your code](https://www.learncpp.com/cpp-tutorial/timing-your-code/) · Site: [learncpp.com](https://www.learncpp.com/)
Related: [18.1 Sorting an array using selection sort](https://www.learncpp.com/cpp-tutorial/sorting-an-array-using-selection-sort/) · [18.3 Introduction to standard library algorithms](https://www.learncpp.com/cpp-tutorial/introduction-to-standard-library-algorithms/)

> **Note:** at the time of writing, learncpp.com has **no "Chapter 18 summary and quiz"** page (the chapter is marked under construction). So these notes use the chapter's last lesson, **18.4 Timing your code**, instead.

## Summary of 18.4 (condensed)

**Why time code?** To decide which of two approaches is faster instead of guessing. C++11's `<chrono>` does it, but it's awkward, so wrap it in a small `Timer` class.

**The Timer class (idea):**
- Aliases: `Clock = std::chrono::steady_clock`, `Second = std::chrono::duration<double, std::ratio<1>>`.
- Stores a start `time_point` initialized to `Clock::now()`.
- `reset()` restarts it. `elapsed()` returns seconds as a `double` via `duration_cast<Second>(Clock::now() - m_beg).count()`.
- Usage: create `Timer t;` where you want to start, then print `t.elapsed()` when done.

**Demo result:** sorting 10,000 elements (filled with `std::iota` in reverse):
- Hand-written selection sort ≈ **0.05 s**
- `std::sort` / `std::ranges::sort` ≈ **0.0007 s**
- So the standard library sort was roughly **100× faster**. Moral: prefer library algorithms.

**Things that distort timing:**
1. **Build type:** use a *release* build, not debug. (The same `std::sort` was ~33× slower in debug.)
2. **Background load:** close other apps, since CPU/disk/memory activity adds noise.
3. **Randomness:** random data changes the amount of work. Temporarily seed with a fixed literal for repeatable runs (but beware if performance depends heavily on the particular sequence).
4. **User input:** never time the program waiting for a human.

**How to measure well:**
- Take **at least 3 runs**. If they cluster, that's your number. Otherwise keep measuring and identify outliers.
- Absolute times only mean something on *your* machine. **Relative** comparisons on one machine are useful (variant A vs B).
- **Sanity check:** after measuring variant B, re-measure variant A. If A is unchanged, the comparison is fair. If not, something on the machine changed, so discard and redo.

## Quiz

This lesson has no quiz. Self-check questions:
1. Why is a debug-build timing misleading? *(Optimizations are usually off, so you aren't measuring the code you'd ship.)*
2. Why re-run variant 1 after measuring variant 2? *(To confirm the machine's conditions didn't change between measurements.)*
3. Why is `steady_clock` the right clock for timing? *(It never jumps backwards, unlike a wall clock.)*

## My additions

- **Algorithms vs hand-rolled loops:** `std::sort` is O(n log n) while selection sort is O(n²), which is where the 100× came from at n = 10,000. The gap grows as n grows.
- **Iterators connect to chapter 17:** the `begin`/`end` pointer-walk from 17.9 *is* the iterator idea. `std::sort(arr.begin(), arr.end())` takes exactly such a pair.
- **Micro-benchmark gotcha:** the compiler can optimize away work whose result you never use, so make sure your timed code's output is used (e.g. print or accumulate it).
- **Embedded/hardware version of this lesson:** on a microcontroller you'd time with a hardware timer or cycle counter (e.g. the ARM Cortex-M DWT cycle count) instead of `<chrono>`. The same rules apply: repeat runs, compare variants, and watch for interrupts adding noise.
- **Habit to build:** measure first, optimize second. Don't assume the "clever" version is faster.

---
[← Prev](ch17-fixed-size-arrays.md) | [Index](README.md) | [Next →](ch19-dynamic-allocation.md)
