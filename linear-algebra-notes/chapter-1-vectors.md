# Chapter 1: Vectors

Source: Essence of Linear Algebra (3Blue1Brown), Chapter 1. Revision notes, minimal clean markdown.

**One-line version:** a vector is something you can add and scale. Linear algebra lets you flip between the arrow picture and the list-of-numbers picture — whichever makes the problem easier.

## 1. Three Views of a Vector

| Physics | Computer science | Math |
|---|---|---|
| An arrow with a length and a direction. Location does not matter — slide it anywhere and it is the same vector. | An ordered list of numbers, any length. Number of entries = dimension. Order matters: `(price, size)` ≠ `(size, price)`. | Anything where adding two of them and scaling one by a number both make sense. Very abstract on purpose. |

CS example: a house as `(square footage, price, bedrooms)` is a 3-dimensional vector. The video used two entries `(size, price)`; a vector does not have to have two numbers.

## 2. The Linear Algebra Convention

```text
v = [4; 3]
```

- Every vector is an arrow with its **tail at the origin**.
- Coordinates are **travel instructions**: x = how far right (or left), y = how far up (or down), z (3D) = how far out of the page.
- Written as a **column**, like `[4; 3]`, so you can tell a vector from a plain point.
- Pinning the tail at the origin gives **one arrow per vector**, so each vector is exactly one point in space.

**Why root everything at the origin?** In the physics view one vector is infinitely many arrows (you can slide it anywhere). Pinning the tail to the origin picks one arrow for each vector, so the tip — the coordinates — identifies the vector completely. That is what makes it possible to translate between arrows and numbers.

## 3. Vector Addition

```text
a + b = [3; 4]
```

**Geometric (tip-to-tail):** put b's tail on a's tip; the sum runs from a's tail to b's tip.

**Numeric:** add component by component.

```text
[2; 1] + [1; 3] = [2+1; 1+3] = [3; 4]
```

Why they match: each vector is a step. Step a, then step b = one combined step.

**Order does not matter:** `a + b = b + a`. Numerically `2 + 1 = 1 + 2`, so every component agrees. Geometrically, a-then-b and b-then-a are the two routes around a parallelogram to the same far corner.

## 4. Scalar Multiplication

A **scalar** is a plain number; it scales the vector. Multiply every component by it.

```text
3·[2; -1] = [6; -3]
-0.5·[4; 6] = [-2; -3]
```

| Scalar | Effect | Example on `[4; 2]` |
|---|---|---|
| greater than 1 | stretches, same direction | `2·[4; 2] = [8; 4]` |
| between 0 and 1 | squishes, same direction | `0.5·[4; 2] = [2; 1]` |
| equals 0 | collapses to the zero vector | `0·[4; 2] = [0; 0]` |
| negative | flips direction, then stretches or squishes | `-1·[4; 2] = [-4; -2]` |

## 5. Subtraction and Division (Not in the Video, but They Follow)

- **Subtraction:** `a − b = a + (−1)·b`, component by component. `[5; 2] − [1; 4] = [4; −2]`. To draw it: draw a, flip b, go tip-to-tail.
- **Division:** dividing by n is multiplying by 1/n — not a separate operation, just a squish.
- Vector × vector is a different thing (dot and cross products, later chapters). The video's "multiplication" is only vector × number.

## 6. Sign Check: Quadrants

| Quadrant | Signs | Position |
|---|---|---|
| I | (+, +) | top right |
| II | (−, +) | top left |
| III | (−, −) | bottom left |
| IV | (+, −) | bottom right |

`3·[2; −1] = [6; −3]` is in quadrant IV; `[−6; −3]` is in quadrant III.

## 7. Quick Reference

| Operation | Rule | Picture |
|---|---|---|
| `v + w` | add matching components | tip-to-tail |
| `v − w` | subtract matching components | v plus flipped w |
| `c·v` | multiply every component by c | stretch, squish, flip |
| dimension | number of entries in the list | 2 = plane, 3 = space |

## 8. Check Your Paper Work in NumPy

```python
import numpy as np
a = np.array([2, 1]); b = np.array([1, 3])
print(a + b)                                  # [3 4]
print(b + a)                                  # [3 4] same, order does not matter
print(3 * np.array([2, -1]))                  # [ 6 -3]
print(-0.5 * np.array([4, 6]))                # [-2. -3.]
print(np.array([5, 2]) - np.array([1, 4]))    # [ 4 -2]
print(len(np.array([1200, 400000, 3])))       # 3 -> dimension
```

## Remember

- A vector is **add + scale**; everything else follows.
- Arrow and list are two views of the same object.
- Tail at the origin turns arrows into coordinates.
- Add component-wise; scale every component; a negative scalar flips.

---
[Index](README.md)
