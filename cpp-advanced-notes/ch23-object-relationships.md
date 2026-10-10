# Chapter 23: Object Relationships

**Source:** [23.x Chapter 23 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-23-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Related lessons: [23.2 Composition](https://www.learncpp.com/cpp-tutorial/composition/) · [23.3 Aggregation](https://www.learncpp.com/cpp-tutorial/aggregation/) · [23.7 std::initializer_list](https://www.learncpp.com/cpp-tutorial/stdinitializer_list/)

## Summary (condensed)

**Object composition** = building complex objects out of simpler ones. It has two subtypes: *composition* and *aggregation*.

| Property \ Type | Composition | Aggregation | Association | Dependency |
|---|---|---|---|---|
| Relationship | whole/part | whole/part | otherwise unrelated | otherwise unrelated |
| Member can belong to several classes? | No | Yes | Yes | Yes |
| Class manages member's lifetime? | Yes | No | No | No |
| Direction | one-way | one-way | one-way or two-way | one-way |
| Verb | **part-of** | **has-a** | **uses-a** | **depends-on** |
| Typical implementation | normal member variable (or owning pointer) | pointer / reference | pointer / reference / index or key | temporary local or function parameter |

- **Composition**: the part exists *because of* the whole. Whole creates and destroys it. The part doesn't know the whole exists. If a class can be a composition, make it one.
- **Aggregation**: the whole *has* the part but doesn't own it. The part can outlive the whole and be shared.
- **Association**: looser. Two otherwise unrelated objects where one *uses* the other over time (doctor and patient).
- **Dependency**: one class uses another to do a task, usually temporarily (created, used, destroyed, or passed in as an argument).
- **Container class**: holds multiple objects of another type. *Value container* = composition (stores copies). *Reference container* = aggregation (stores pointers or references to objects living elsewhere).
- **`std::initializer_list`** (header `<initializer_list>`) lets constructors, assignment operators and other functions accept a brace-enclosed list.

## Quiz (my wording) and answers

1. Classify each relationship.
   - Animal class holding an animal-type enum and a name string → **Composition**. The data has no use outside the Animal.
   - Text editor with `save(File)` that writes to disk → **Dependency**. File is only used to complete one task.
   - Adventurer carrying Items that can be dropped and picked up by others → **Association**. Not whole/part, and the Adventurer doesn't manage the Items' existence.
   - Player prays at a Shrine to get a power → **Dependency**. No long-term link.
   - Computer containing a CPU that can be removed and tested alone → **Aggregation**. Has-a, but doesn't manage the CPU's existence.
   - Blacksmith using a specific anvil → **Association**. Uses-a, not whole/part.
2. If you can design a class using one of the four, you should use **composition**.

## My additions

- **Quick decision test:** does the part die with the whole? → composition. Can the part live without it and be shared? → aggregation. Not really "part of" at all, just used over time? → association. Only used inside one function call? → dependency.
- **Hardware analogy:** a PCB *composes* its copper traces (no board, no traces). It *aggregates* a socketed CPU or removable chip (you can pull it out and use it elsewhere). A scope probe is an *association* with whatever you test. A bench power supply used for one measurement is a *dependency*.
- **Don't confuse** "aggregation" (this chapter) with "aggregate" (a struct or array initializable with braces). Different things, similar names.
- **Gotcha:** an aggregation using raw pointers can leave dangling pointers if the part is destroyed before the whole. Composition avoids this because the whole controls the part's lifetime.
