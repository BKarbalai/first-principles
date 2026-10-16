# Cheatsheet 2: Conversion, K-Map

Source: Neso Academy Digital Electronics, videos 11-17. Notes for Boolean Algebra (Jul 17, 2026).

## Minimal to Canonical

- SOP: missing var `X` → AND with `(X+X')`. Expand, drop dupes.
- POS: missing var `X` → OR with `XX'`. `(S+XX') = (S+X)(S+X')`.
- Shortcut: fixed bits + all combos of missing bits. k missing → `2^k` terms.

```text
A + BC → Σm(3,4,5,6,7) → ΠM(0,1,2)
AB + A'C → Σm(1,3,6,7) → ΠM(0,2,4,5)
(A+B)(B'+C) → ΠM(0,1,2,6) → Σm(3,4,5,7)
AB + C'D (4-var) → Σm(1,5,9,12,13,14,15)
```

Maxterm bits are reversed: plain = 0, `'` = 1.

## K-Map Basics

- Graphical truth table. n vars → `2^n` cells.
- Labels in Gray code: `00, 01, 11, 10`. Neighbours differ by 1 var → merge.
- Wraps around: left-right, top-bottom. Corners are neighbours.

Cell numbers:

```text
2-var (A rows, B cols):      0 1 / 2 3
3-var (A rows, BC cols):     0 1 3 2 / 4 5 7 6
4-var (AB rows, CD cols):    0 1 3 2 / 4 5 7 6 / 12 13 15 14 / 8 9 11 10
```

## Grouping (SOP)

- Group 1s. Size = power of 2: 1, 2, 4, 8, 16.
- Rectangles only. Biggest groups, fewest groups. Overlap and wrap allowed.
- Every 1 covered. X (don't-care) = use as 1 if it helps, else ignore.
- Read group: keep constants, drop changers. `2^k` cells remove k vars.

| Size | Removes | Literals left (4-var) |
|---|---|---|
| 1 | 0 | 4 |
| 2 | 1 | 3 |
| 4 | 2 | 2 |
| 8 | 3 | 1 |
| 16 | 4 | 0 (F=1) |

```text
Σm(0,1,3) = A' + B
Σm(1,2,3,5,6,7) = B + C
Σm(0,1,2,4,5,6) = B' + C'        # wrap first+last cols
Σm(1,3,5,6,7): SOP = C + AB
```

## POS from K-Map

- Group 0s, same rules. Read as sum: 0 → plain, 1 → `'`. AND the sums.
- Example: `Σm(1,3,5,6,7)` → POS `(B+C)(A+C)`. Same as `C+AB`.

## Implicants

- Implicant: any valid group of 1s.
- Prime (PI): cannot grow. Essential (EPI): covers a lonely 1, must take.
- Steps: mark all PIs → take all EPIs → cover leftovers with fewest/largest.

```text
Σm(0,1,2,5,6,7,8,9,10,14) = B'C' + CD' + A'BD
# B'C' (0,1,8,9) EPI, CD' (2,6,10,14) EPI, A'BD (5,7) covers rest
# B'D' is prime but unused.
```

Cyclic case (no EPI): multiple minimal answers OK.

```text
Σm(0,1,2,5,6,7) = A'B' + AC + BC'  or  B'C + AB + A'C'
```

## 4-Variable Map

- 4x4, wraps both ways. Pairs → 3 literals, quads → 2, octets → 1.
- Watch for: four corners, full row/col, two outer rows/cols.

```text
Σm(0,2,8,10) = B'D'                    # four corners
Σm(1,3,5,7,9,11,13,15) = D            # octet, odd minterms
Σm(1,3,4,5,6,7,9,11,12,13,14,15) = B + D
```

## Cheat Sheet

```text
Precedence: brackets, NOT, AND, OR
Absorb: A+AB=A, A+A'B=A+B, AB+AB'=A
De Morgan: break bar, flip sign
Consensus: AB+A'C+BC = AB+A'C
m = AND all vars, M = OR all vars, M_i = (m_i)'
Σm(1-rows), ΠM(0-rows), lists complement
K-map: Gray 00 01 11 10, groups 1-2-4-8-16, biggest+fewest
POS: group 0s, 0=plain 1=', AND sums
PI=cannot grow, EPI=lonely 1, take EPIs first
```

## Practice

```text
1. AB + AB' + A'B = ?              → A + B
2. A'B + AC + BC = ?               → A'B + AC (drop BC)
3. Σm(0,3,5) as ΠM?                → ΠM(1,2,4,6,7)
4. A'B + C → canonical?            → Σm(1,2,3,5,7)
5. Σm(0,1,4,5,6) via K-map?        → B' + AC'
6. Σm(0,2,8,10)+Σ? Σm(0,2,5,7,8,10,13,15)? → B'D' + BD
7. Q5 POS via 0s?                  → (A+B')(B'+C')
8. [(A+B')(C'+D)]' = ?             → A'B + CD'
```

---
[Index](README.md) | [← Prev](cheatsheet-1.md)
