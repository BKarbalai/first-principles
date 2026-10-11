# NumPy Cheatsheet

Source: Codédex NumPy cheatsheet. Notes for Python NumPy and DSA (Jun 4, 2026).

## Import

```python
import numpy as np
```

## NumPy Arrays

```python
menu_prices = np.array([14.99, 5.99, 21.00])
menu_quantity = np.array([5, 8, 1])
```

## Indexing

```python
print(menu_prices[0])   # Output: 14.99
print(menu_prices[1])   # Output: 5.99
print(menu_prices[2])   # Output: 21.00

print(menu_prices[-1])  # Output: 21.00
```

## Slicing

```python
boba = np.array([3, 2, 1, 0, 1])

print(boba[0:2])  # Output: [3 2]
print(boba[2:])   # Output: [1 0 1]
print(boba[-2:])  # Output: [0 1]
```

## 2D Arrays

```python
seating_plan = np.array([
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
])

print(seating_plan[0])  # Output: [1, 2, 3]
```

## Array-Array Operations

```python
gas = np.array([20.32, 44.87, 67.03])
electric = np.array([80.10, 77.79, 67.73])

total = gas + electric  # element-wise sum
```

## Arithmetic Operations

```python
order = np.array([12.99, 9.99, 4.99])

print(order + 2.00)  # Addition
print(order - 1.50)  # Subtraction
print(order * 0.8)   # Multiplication
print(order / 3)     # Division
print(order % 2)     # Modulo
```

## NumPy Functions

```python
water = np.array([[6, 2, 3, 8, 7, 9, 7,],
                  [5, 5, 10, 4, 7, 5, 4]])

print(np.min(water))      # Output: 2
print(np.max(water))      # Output: 10
print(np.sum(water))      # Output: 82
print(np.average(water))  # Output: 5.8571
```

## Axis

```python
np.sum(water, axis=0)  # Down each column
np.min(water, axis=1)  # Across each row
```

## .shape

```python
print(water.shape)  # Output: (2, 7)
```

## .reshape()

```python
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8])
new_arr = arr.reshape(2, 4)

# [[1 2 3 4]
#  [5 6 7 8]]
```

## .arange()

```python
arr = np.arange(start=1, stop=10, step=3)
# [1 4 7]
```

## My additions

- **Vectorization is the whole point.** A NumPy operation runs one C loop over the array. The equivalent Python `for` loop runs N Python iterations. Same lesson as the timing habit: measure first, but vectorized code wins by 10-100x on large arrays.
- **Slices are views, not copies.** `b = a[0:5]` shares memory with `a`. Mutating `b` mutates `a`. Call `.copy()` when you need independence. This is the NumPy version of aliasing bugs (cf. references in C++ ch12).
- **Axis mental model:** `axis=0` collapses rows (works down each column), `axis=1` collapses columns (works across each row). The result loses the axis you summed over: `(2, 7)` summed on `axis=0` gives shape `(7,)`.
- **dtype gotcha:** `np.array([1, 2, 3])` is int. `arr / 3` still gives floats, but in-place `arr /= 3` truncates back to int silently. Declare `dtype=float` up front when division is coming.
- **`reshape(-1)` infers for you.** `arr.reshape(2, -1)` means "2 rows, however many columns fit". Fails loudly if the total size doesn't divide evenly, which is what you want.
- **Broadcasting in one line:** a shape `(3,)` array plus a shape `(3, 3)` array works by stretching the smaller one along the missing axis. Shapes must match or be 1 from the trailing edge, else you get a broadcast error.
- **Hardware/EE angle:** NumPy is the analysis workbench for everything downstream: plotting RC charge curves, FFTs of sampled signals (Fourier block), and quick golden-model checks against RTL simulation output.

---
[Index](README.md)
