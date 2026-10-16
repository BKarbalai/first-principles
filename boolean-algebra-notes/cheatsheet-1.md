# Cheatsheet 1: Boolean Algebra, SOP/POS

Source: Neso Academy Digital Electronics, videos 1-10. Notes for Boolean Algebra (Jul 17, 2026).

## Ops and Precedence

- `+` = OR, juxtaposition (AB) = AND, `'` = NOT.
- Order: brackets, NOT, AND, OR.
- XOR = `A'B + AB'` (differ). XNOR = `AB + A'B'` (equal).
- NAND and NOR are universal.

## Identities

| Name | OR | AND |
|---|---|---|
| Identity | `A + 0 = A` | `A . 1 = A` |
| Null | `A + 1 = 1` | `A . 0 = 0` |
| Idempotent | `A + A = A` | `A . A = A` |
| Complement | `A + A' = 1` | `A . A' = 0` |
| Involution | `(A')' = A` | — |

## Laws

| Law | Form |
|---|---|
| Commutative | `A+B = B+A`, `AB = BA` |
| Associative | `(A+B)+C = A+(B+C)`, `(AB)C = A(BC)` |
| Distributive | `A(B+C) = AB+AC`, `A+BC = (A+B)(A+C)` |

Dual: swap `+`/`.` and `0`/`1`. True stays true. Dual != complement.

## Absorption (most used)

```text
A + AB = A
A(A + B) = A
A + A'B = A + B
A(A' + B) = AB
AB + AB' = A
(A+B)(A+B') = A
```

## De Morgan

```text
(A + B)' = A' . B'
(AB)' = A' + B'
```

Break the bar, flip the sign, complement each literal.

## Simplify — Game Plan

1. Factor common term.
2. Hunt `X + X' = 1`, `X . X' = 0`.
3. Try absorption and `A + A'B = A + B`.
4. `(A+...)(A+...)` → use `A+BC = (A+B)(A+C)` backwards.
5. Long bars → De Morgan first.
6. Check consensus (below).

```text
AB + AB' = A
A + A'B + AB' = A + B        # A+A'B = A+B, then absorb AB'
(A+B)(A+B')(A'+C) = AC       # first two give A, then A(A'+C)
[(A'+B)(A+C')]' = AB' + A'C  # De Morgan twice
```

## Redundancy (Consensus) Theorem

```text
AB + A'C + BC = AB + A'C
(A+B)(A'+C)(B+C) = (A+B)(A'+C)
```

- Find opposite pair (`A` vs `A'`), join the rest → consensus term.
- If consensus is present, delete it.

```text
AB' + BC + AC = AB' + BC     # consensus of AB', BC is AC
XY + X'Z + YZW = XY + X'Z     # YZW contains YZ, also redundant
```

## SOP and POS

- Product term: AND (`AB'C`). Sum term: OR (`A+B'+C`).
- SOP = OR of products. POS = AND of sums.
- Minterm `m_i`: AND of all vars, 1 on exactly one row. Bit 1 → plain, 0 → `'` .
- Maxterm `M_i`: OR of all vars, 0 on exactly one row. Bit 0 → plain, 1 → `'` .
- `M_i = (m_i)'`.

| i | ABC | m_i | M_i |
|---|---|---|---|
| 0 | 000 | `A'B'C'` | `A+B+C` |
| 1 | 001 | `A'B'C` | `A+B+C'` |
| 2 | 010 | `A'BC'` | `A+B'+C` |
| 3 | 011 | `A'BC` | `A+B'+C'` |
| 4 | 100 | `AB'C'` | `A'+B+C` |
| 5 | 101 | `AB'C` | `A'+B+C'` |
| 6 | 110 | `ABC'` | `A'+B'+C` |
| 7 | 111 | `ABC` | `A'+B'+C'` |

## Canonical Forms

- SOP: OR minterms where F=1. `F = Σm(1-rows)`.
- POS: AND maxterms where F=0. `F = ΠM(0-rows)`.
- Lists complement each other. `Σm(1,3,6) = ΠM(0,2,4,5,7)` for 3 vars.
- n vars: `2^n` rows, `2^(2^n)` functions.

```text
XOR: Σm(1,2) = A'B + AB' | ΠM(0,3) = (A+B)(A'+B')
Majority (3-var, 2+ ones): Σm(3,5,6,7) = A'BC+AB'C+ABC'+ABC = AB+BC+AC
```

Use SOP when fewer 1s, POS when fewer 0s.

---
[Index](README.md) | [Next →](cheatsheet-2.md)
