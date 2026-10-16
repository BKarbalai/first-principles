# Cheatsheet 1: Gates, Rules, SOP/POS

Source: Neso Academy Digital Electronics, videos 1-10. Notes for Boolean Algebra (Jul 17, 2026).

## 1. Start Here

Digital = only `0` (OFF) and `1` (ON). Inputs `A, B, C...`, output `Y` or `F`.

Boolean algebra = write circuit as formula, then make it shorter. Shorter = fewer gates = cheaper, faster, smaller.

| Symbol | Name | Example |
|---|---|---|
| `+` | OR | `A + B` |
| `·` / nothing | AND | `A·B` or `AB` |
| `'` | NOT | `A'` |
| `( )` | do first | `A(B + C)` |

## 2. Gates and Truth Tables

AND: 1 only if both are 1. OR: 0 only if both are 0. NOT: flips.

| A | B | AND | OR |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 |
| 1 | 1 | 1 | 1 |

| A | B | NAND | NOR | XOR | XNOR |
|---|---|---|---|---|---|
| 0 | 0 | 1 | 1 | 0 | 1 |
| 0 | 1 | 1 | 0 | 1 | 0 |
| 1 | 0 | 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 0 | 0 | 1 |

- NAND = AND then NOT. NOR = OR then NOT.
- XOR = 1 when different: `A'B + AB'`.
- XNOR = 1 when same: `AB + A'B'`.
- NAND and NOR are universal.

Order: brackets, NOT, AND, OR.

```text
A=0, B=1, C=0, Y = A + B·C'
C' = 1, B·C' = 1, A + 1 = 1
```

## 3. Rules

| Rule | Why |
|---|---|
| `A + 0 = A` | OR with 0 changes nothing |
| `A · 1 = A` | AND with 1 changes nothing |
| `A + 1 = 1` | one 1 is enough |
| `A · 0 = 0` | one 0 ruins AND |
| `A + A = A` | same thing twice |
| `A · A = A` | same |
| `A + A' = 1` | one must be 1 |
| `A · A' = 0` | never both 1 |
| `(A')' = A` | flip twice |

Rearranging:

```text
A + B = B + A, AB = BA
(A + B) + C = A + (B + C), (AB)C = A(BC)
A(B + C) = AB + AC
A + BC = (A + B)(A + C)   # new, no normal-maths equivalent
```

Shortcuts:

| Shortcut | Why |
|---|---|
| `AB + AB' = A` | B doesn't matter |
| `A + AB = A` | AB adds nothing |
| `A + A'B = A + B` | if A=0, left = B |

De Morgan — break bar, flip sign:

```text
(A + B)' = A' · B'
(A · B)' = A' + B'
(A + B + C)' = A'B'C'
```

## 4. Simplifying

Game plan: factor → look for `X+X'=1` → shortcuts → De Morgan first if long bar.

```text
AB + AB' = A(B+B') = A
A + A'B + AB' = (A+B) + AB' = A + B
A'B'C + A'BC + AB'C + ABC = C, factor C then group
(A+B)(A+B')(A'+C) = A(A'+C) = AC
(A'B + CD')' = (A+B')(C'+D)
```

Watch out: `(A+B)' = A'B'`, not `A'+B'`. `A·A'=0`, `A+A'=1`, `A+A=A`. `AB=AC` does not mean `B=C`.

## 5. Redundancy Theorem

```text
AB + A'C + BC = AB + A'C
(A+B)(A'+C)(B+C) = (A+B)(A'+C)
```

Spot it: opposite pair (`A` / `A'`), join leftovers (`B`,`C` → `BC`). If present, delete.

```text
AB + A'C + BC → AB + A'C
AB' + BC + AC → AB' + BC
```

## 6. SOP and POS

Majority example: F=1 when 2+ inputs are 1. Rows 3,5,6,7 are 1. Rows 0,1,2,4 are 0.

SOP: minterms from 1-rows. Bar the 0s.

| Row | ABC | Minterm |
|---|---|---|
| 3 | 011 | `A'BC` |
| 5 | 101 | `AB'C` |
| 6 | 110 | `ABC'` |
| 7 | 111 | `ABC` |

```text
F = A'BC + AB'C + ABC' + ABC = Σm(3,5,6,7)
```

POS: maxterms from 0-rows. Bar the 1s (reverse).

| Row | ABC | Maxterm |
|---|---|---|
| 0 | 000 | `(A+B+C)` |
| 1 | 001 | `(A+B+C')` |
| 2 | 010 | `(A+B'+C)` |
| 4 | 100 | `(A'+B+C)` |

```text
F = (A+B+C)(A+B+C')(A+B'+C)(A'+B+C) = ΠM(0,1,2,4)
```

|  | SOP | POS |
|---|---|---|
| Look at rows | F=1 | F=0 |
| Piece | minterm (AND all) | maxterm (OR all) |
| Bar when | 0 in row | 1 in row |
| Join with | OR | AND |
| Short | `Σm(...)` | `ΠM(...)` |

Shortcut: Σm + ΠM = all rows. `Σm(1,3,6) = ΠM(0,2,4,5,7)`, `F' = Σm(0,2,4,5,7)`.

Minterm/maxterm table (3 inputs):

| Row | ABC | m | M |
|---|---|---|---|
| 0 | 000 | `A'B'C'` | `(A+B+C)` |
| 1 | 001 | `A'B'C` | `(A+B+C')` |
| 2 | 010 | `A'BC'` | `(A+B'+C)` |
| 3 | 011 | `A'BC` | `(A+B'+C')` |
| 4 | 100 | `AB'C'` | `(A'+B+C)` |
| 5 | 101 | `AB'C` | `(A'+B+C')` |
| 6 | 110 | `ABC'` | `(A'+B'+C)` |
| 7 | 111 | `ABC` | `(A'+B'+C')` |

---
[Index](README.md) | [Next →](cheatsheet-2.md)
