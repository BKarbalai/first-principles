# Chapter 15: More on Classes

**Source:** [15.x Chapter 15 summary and quiz](https://www.learncpp.com/cpp-tutorial/chapter-15-summary-and-quiz/) · Site: [learncpp.com](https://www.learncpp.com/)
Lessons you studied: [15.2 Classes and header files](https://www.learncpp.com/cpp-tutorial/classes-and-header-files/) · [15.8 Friend non-member functions](https://www.learncpp.com/cpp-tutorial/friend-non-member-functions/) · [15.9 Friend classes and friend member functions](https://www.learncpp.com/cpp-tutorial/friend-classes-and-friend-member-functions/) · [15.10 Ref qualifiers](https://www.learncpp.com/cpp-tutorial/ref-qualifiers/)

## Summary (condensed)

- **`this`**: inside every non-static member function, `this` is a const pointer to the current (implicit) object. Returning `*this` by reference enables **method chaining**: `obj.a().b().c();`
- **Header/source split:** put the class definition in a header named after the class. Trivial members (getters, empty-body constructors) can stay in-class. Non-trivial member functions go in a matching `.cpp`.
  - Functions defined *inside* the class are implicitly `inline` (ODR-safe). Ones defined outside in a header need an explicit `inline`.
  - Default arguments for member functions go in the class definition.
- **Nested types (member types):** a type defined inside a class, including type aliases.
- **Class templates:** member functions defined *inside* the template can use the template parameters directly. Outside, you must repeat the `template<...>` declaration, and define them in the same file, just below the class.
- **Static member variables:** one shared copy for the whole class, exist even with zero objects. Access via `ClassName::member`. Making them `inline` lets you initialize them in the class body.
- **Static member functions:** callable with no object. No `this`, can't touch non-static data members.
- **Friends:** a `friend` declaration inside a class grants a function or class full access to private/protected members. Friend **function** (member or non-member) and friend **class**.
  - Friendship is granted *by* the class being accessed. It is **not reciprocal**, **not transitive** and **not inherited**.
  - Friending a single member function needs the full definition of that function's class first, which forces a forward-declare + out-of-class-definition dance (solved by header/source files).
- **Ref qualifiers (15.10):** let you write separate member-function overloads for lvalue vs rvalue implicit objects. Solves the problem where a getter returning `const T&` becomes a dangling reference when called on a temporary.

## Quiz: random monster generator (what each step practices)

| Step | Task | Concept |
|---|---|---|
| a | `enum class MonsterType` with a trailing `maxMonsterTypes` | scoped enum + count trick |
| b | `Monster` class with type, name, roar, hit points | private members, `m_` prefix |
| c | Make the enum a **nested unscoped** `Monster::Type` | nested types |
| d | Constructor taking all four members (`string_view` params) | member initializer list |
| e | `getTypeString()` (constexpr, `switch`) and `print()` | const member functions |
| f | `MonsterGenerator` as a **namespace** (no state) with `generate()` | namespace vs class |
| g | Add `getName(int)` and `getRoar(int)` for 0-5 | helper functions |
| h | Use `Random::get()` for type, name, roar, and 1-100 hit points | `static_cast<Monster::Type>(Random::get(0, maxMonsterTypes-1))` |

Expected output looks like: `Mort the zombie has 61 hit points and says *growl*.` A monster with 0 hit points prints `<name> the <type> is dead.`

## My additions

- **Declaration vs definition cheat sheet:** class body = declaration (menu), `.cpp` = definition (recipe). `Date::print()` the `Date::` prefix tells the compiler it's a member.
- **Why not put everything in the header?** Any edit forces every file that includes it to recompile. Headers for interfaces, `.cpp` for logic.
- **Friend = last resort.** Order of preference: use the public interface → add a sensible access function → only then use `friend`. Reason: friends depend on your private names, so renaming a member breaks every friend.
- **`friend` use cases:** a function that works equally on two classes (e.g. `printWeather(Temperature, Humidity)`), or tightly-coupled helper classes (Storage/Display). Also operator overloading (chapter 21).
- **Static members and hardware:** a `static` counter or shared config in a driver class is a common way to track "how many instances of this peripheral are open".
- **Ref qualifier takeaway:** returning `const T&` from a getter is the normal choice and is fine on named objects. The danger is only saving that reference from a *temporary*.
- **Check yourself:** why can't a `static` member function read a normal data member? (No `this` pointer, so no object to read it from.)

---
[← Prev](ch14-intro-to-classes.md) | [Index](README.md) | [Next →](ch16-vectors-and-containers.md)
