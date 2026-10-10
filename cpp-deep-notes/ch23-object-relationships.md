# Chapter 23: Object Relationships

**Source:** [23.x Chapter 23 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-23-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Related lessons: [23.2 Composition](https://www.learncpp.com/cpp-tutorial/composition/) · [23.3 Aggregation](https://www.learncpp.com/cpp-tutorial/aggregation/) · [23.7 std::initializer_list](https://www.learncpp.com/cpp-tutorial/stdinitializer_list/)

---

## 1. The big idea: objects built from objects

Real things are made of parts (a car has an engine, a computer has a CPU). In C++ you model this by letting one class **contain or refer to** other classes. This is called **object composition** in the broad sense. It comes in several flavors, and the main job of this chapter is telling them apart:

| Relationship | Verb | Meaning |
|---|---|---|
| **Composition** | *part-of* | whole creates/owns/destroys the part |
| **Aggregation** | *has-a* | whole uses a part it doesn't own |
| **Association** | *uses-a* | two unrelated objects work together over time |
| **Dependency** | *depends-on* | one object uses another briefly to do a task |

---

## 2. Composition (part-of)

A **composition** is a whole/part relationship where:
- The part (member) can belong to **only one** whole at a time.
- The **whole manages the part's lifetime**: the part is created when the whole is created and destroyed when the whole is destroyed.
- The part has **no knowledge** that the whole exists (one-way relationship).
- The part **cannot exist** without the whole.

Typical implementation: a **normal member variable** (or an owning pointer).

```cpp
class Fraction { int m_num{}; int m_den{}; };

class Calculator
{
    Fraction m_result{};   // composition: the Calculator owns it
};
```

Why it's good: no dangling pointers, no manual cleanup, and the whole's constructor/destructor automatically build/destroy the parts. **If a class can be a composition, make it one.**

---

## 3. Aggregation (has-a)

An **aggregation** is also whole/part, but:
- The part can belong to **multiple** wholes at once.
- The **whole does not manage the part's lifetime**.
- The part exists independently and can outlive the whole.
- One-way relationship.

Typical implementation: a **pointer or reference** to a part that lives elsewhere.

```cpp
class Teacher { std::string m_name; };

class Department
{
    const Teacher& m_teacher;   // aggregation: Department doesn't own the Teacher
public:
    Department(const Teacher& t) : m_teacher{ t } {}
};
```
If the `Department` is destroyed, the `Teacher` keeps existing (and can belong to other departments). Risk: if the `Teacher` dies first, the reference/pointer **dangles**.

---

## 4. Association (uses-a)

Two objects are **otherwise unrelated** (neither is part of the other), but one **uses** the other over a longer period.
- Both exist independently, can be used by many others, and neither manages the other's lifetime.
- The relationship can be **one-way or two-way**.
- Typical implementation: pointer, reference, or an **index/key** into some other structure.

Example: a **Doctor** and a **Patient**. The doctor treats the patient (association), a patient can see many doctors, and the doctor/patient lifetimes are independent. Two-way association (each knows the other) is possible but is trickier to keep consistent.

---

## 5. Dependency (depends-on)

A **dependency** means one object **temporarily uses** another to get something done. It's the weakest relationship: usually the other object is created, used and destroyed in one function, or passed in as a **parameter**.

```cpp
std::ostream& operator<<(std::ostream& out, const Point& p);   // output stream depends on Point only for this call
```
Example: a text editor's `save(File&)` depends on `File` only during the call. `std::cout << x` makes `cout` depend on `x`.

---

## 6. Side-by-side summary

| Property | Composition | Aggregation | Association | Dependency |
|---|---|---|---|---|
| Relationship type | whole/part | whole/part | otherwise unrelated | otherwise unrelated |
| Member can belong to multiple classes | No | Yes | Yes | Yes |
| Member existence managed by class | **Yes** | No | No | No |
| Directionality | one-way | one-way | one- or two-way | one-way |
| Relationship verb | part-of | has-a | uses-a | depends-on |
| Typical implementation | normal member (or owning pointer) | pointer / reference | pointer / reference / index | temporary local or function parameter |

---

## 7. Container classes

A **container class** holds and organizes multiple instances of another type. Two kinds:
- **Value container:** stores **copies** of its elements and owns them → a **composition**. (`std::vector<std::string>` stores strings by value.)
- **Reference container:** stores **pointers or references** to objects that live elsewhere → an **aggregation**. (e.g. `std::vector<std::reference_wrapper<Item>>`.)

Most standard containers are value containers. Container classes usually provide operations to create, insert, remove, find, sort and report size.

### `std::initializer_list` (23.7)
Header `<initializer_list>`. It's how a class lets you initialize it from a brace-enclosed list of values (`MyList l{ 1, 2, 3 };`): the constructor takes a `std::initializer_list<T>` parameter, and you loop over it or copy it into your own storage. A class can also provide an `operator=` taking an `initializer_list` so `l = { 4, 5, 6 };` works. This is the mechanism behind `std::vector v{ 1, 2, 3 };`.

---

## Quiz and answers

1. **Classify each relationship.**
   - Animal class with an animal-type enum and a name string → **Composition**. The data has no use outside the Animal.
   - Text editor with `save(File)` that writes to disk → **Dependency**. File is only used to complete one task.
   - Adventurer carrying Items that can be dropped and picked up by others → **Association**. Not whole/part, and the Adventurer doesn't manage the Items' existence.
   - Player prays at a Shrine to get a power → **Dependency**. No long-term link.
   - Computer containing a CPU that can be removed and tested alone → **Aggregation**. Has-a, but the Computer doesn't manage the CPU's existence.
   - Blacksmith using a specific anvil → **Association**. Uses-a, not whole/part.
2. **If you can design a class using more than one of these, which should you pick?** **Composition**, because the whole manages the part's lifetime and you avoid dangling pointers.

---

## My additions

- **Quick decision test:** does the part die with the whole? → composition. Can the part live without it and be shared? → aggregation. Not really "part of" at all, just used over time? → association. Only used inside one function call? → dependency.
- **Hardware analogy:** a PCB *composes* its copper traces (no board, no traces). It *aggregates* a socketed CPU or removable chip (you can pull it out and use it elsewhere). A scope probe is an *association* with whatever you test. A bench power supply used for one measurement is a *dependency*.
- **Don't confuse** "aggregation" (this chapter) with "aggregate" (a struct/array initializable with braces, chapter 13). Different things, similar names.
- **Gotcha:** an aggregation using raw pointers can leave dangling pointers if the part is destroyed before the whole. Composition avoids that because the whole controls the part's lifetime.
- **Connects to chapter 25:** a composition/aggregation of base-class pointers is how you store a mix of derived objects (e.g. `std::vector<std::unique_ptr<Shape>>`).

---
[← Prev](ch19-dynamic-allocation.md) | [Index](README.md) | [Next →](ch25-virtual-functions.md)
