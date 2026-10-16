# Cheatsheet 2: Conversion, K-Maps

Source: Neso Academy Digital Electronics, videos 11-17. Notes for Boolean Algebra (Jul 17, 2026).

## 7. Short Formula to Standard Form

Short like `A + BC` is missing letters. Fill with `X+X'=1` (SOP) or `XX'=0` (POS).

SOP: missing `X` → multiply by `(X+X')`.

```text
F(A,B,C) = A + BC
A = A(B+B')(C+C') = ABC+ABC'+AB'C+AB'C' = 7,6,5,4
BC = BC(A+A') = ABC+A'BC = 7,3
F = Σm(3,4,5,6,7) = ΠM(0,1,2)
```

Shortcut: k missing → `2^k` minterms. Plain=1, bar=0, blanks take all combos.

```text
F(A,B,C,D) = AB + C'D
AB = 11__ → 1100,1101,1110,1111 = 12,13,14,15
C'D = __01 → 0001,0101,1001,1101 = 1,5,9,13
F = Σm(1,5,9,12,13,14,15)
```

POS: missing `X` → add `XX'`, use `(S+XX')=(S+X)(S+X')`.

```text
F = (A+B)(B'+C)
(A+B) lacks C → (A+B+C)(A+B+C') = M0,M1
(B'+C) lacks A → (A+B'+C)(A'+B'+C) = M2,M6
F = ΠM(0,1,2,6) = Σm(3,4,5,7)
```

## 8. K-Maps

One idea: `AB + AB' = A`. Pairs differing in one letter merge. K-map puts such pairs next to each other.

### 8.1-8.4 Two-input walkthrough

Truth table: `# A B F: 0 0 0 1 / 1 0 1 1 / 2 1 0 0 / 3 1 1 1`

Map (small grey = row no.):

| A\B | 0 | 1 |
|---|---|---|
| 0 | 1 (0) | 1 (1) |
| 1 | 0 (2) | 1 (3) |

Circles: Pair 1 = boxes 0,1. Pair 2 = boxes 1,3. Overlap OK, box 1 used twice.

Read: keep letters that never change.

- 0,1: row A=0, B changes → `A'`.
- 1,3: col B=1, A changes → `B`.

```text
F = A' + B
```

2 boxes drop 1 letter. 4 boxes drop 2, etc.

### 8.5 Gray Code

Labels must be `00, 01, 11, 10`, not `00, 01, 10, 11`.

| Order | Between neighbours | Good? |
|---|---|---|
| 00,01,10,11 | 01→10 changes two bits | No |
| 00,01,11,10 | every step one bit, incl. 10→00 wrap | Yes |

### 8.6 Three-input map

Box numbers (A rows, BC cols):

| A\BC | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 0 | 0 | 1 | 3 | 2 |
| 1 | 4 | 5 | 7 | 6 |

Majority `Σm(3,5,6,7)` filled:

| A\BC | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 0 | 0 (0) | 0 (1) | 1 (3) | 0 (2) |
| 1 | 0 (4) | 1 (5) | 1 (7) | 1 (6) |

Row 5 = A=1, BC=01 → box 5 = 1.

### 8.7 Circling rules

- Only 1s (X later). Size 1,2,4,8,16. Never 3,5,6,7.
- Rectangles only. No L, diagonal, gaps.
- Biggest circles, fewest circles, every 1 covered. Overlap + wrap allowed.

Allowed:

```text
Pair side-by-side: boxes 1,3 = 1
Square of 4: boxes 0,1,4,5 = 1
Wrap left-right: boxes 0,4 + 2,6 = 1
```

| A\BC | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 0 | 0 | 1 | 1 | 0 |
| 1 | 0 | 0 | 0 | 0 |

Pair above = boxes 1,3.

Not allowed: 3 boxes, diagonal, gap.

### 8.8 Reading a circle

Always 1 → plain. Always 0 → bar. Changes → drop.

Quad 1,3,5,7:

| Box | A | B | C |
|---|---|---|---|
| 1 | 0 | 0 | 1 |
| 3 | 0 | 1 | 1 |
| 5 | 1 | 0 | 1 |
| 7 | 1 | 1 | 1 |

A out, B out, C=1 → `C`.

| A\BC | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 0 | 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 1 | 0 |

Pair 0,4 (000,100):

| Box | A | B | C |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 4 | 1 | 0 | 0 |

→ `B'C'`.

| A\BC | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 0 | 1 | 0 | 0 | 0 |
| 1 | 1 | 0 | 0 | 0 |

### 8.9 Wrap-around

Roll map into tube: left touches right, top touches bottom. Gray makes 10–00 one bit apart.

Quad 0,2,4,6 (left col + right col):

| A\BC | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 0 | 1 | 0 | 0 | 1 |
| 1 | 1 | 0 | 0 | 1 |

A changes, B changes, C=0 → `C'`.

### 8.10 Full 3-input example

`F(A,B,C) = Σm(0,2,3,6,7)`.

| # | A | B | C | F |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 1 |
| 1 | 0 | 0 | 1 | 0 |
| 2 | 0 | 1 | 0 | 1 |
| 3 | 0 | 1 | 1 | 1 |
| 4 | 1 | 0 | 0 | 0 |
| 5 | 1 | 0 | 1 | 0 |
| 6 | 1 | 1 | 0 | 1 |
| 7 | 1 | 1 | 1 | 1 |

Filled:

| A\BC | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 0 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 1 | 1 |

Circles: Quad 2,3,6,7 (blue). Pair 0,2 (red, overlap allowed).

- Quad 010,011,110,111: A out, C out, B=1 → `B`.
- Pair 000,010: B out, A=0, C=0 → `A'C'`.

```text
F = B + A'C'
Check: B covers 2,3,6,7. A'C' covers 0,2. = 0,2,3,6,7 ✓
```

### 8.11 Four-input map

Box numbers (AB rows, CD cols):

| AB\CD | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 00 | 0 | 1 | 3 | 2 |
| 01 | 4 | 5 | 7 | 6 |
| 11 | 12 | 13 | 15 | 14 |
| 10 | 8 | 9 | 11 | 10 |

Each box has 4 neighbours incl. wrap. Box 0 touches 1,2,4,8.

Shapes:

- Pair `A'BC'`: boxes 4,5.
- Pair wrap top-bottom `B'C'D'`: boxes 0,8.
- Quad row `A'B`: row 01 all 1.
- Quad square `BD`: boxes 5,7,13,15.
- Quad corners `B'D'`: boxes 0,2,8,10.
- Octet two rows `A'`: rows 00+01 all 1.

Pair removes 1, quad 2, octet 3.

### 8.12 Full 4-input example

`F(A,B,C,D) = Σm(0,1,2,3,6,7,8,9,10,11)`.

Filled:

| AB\CD | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 00 | 1 | 1 | 1 | 1 |
| 01 | 0 | 0 | 1 | 1 |
| 11 | 0 | 0 | 0 | 0 |
| 10 | 1 | 1 | 1 | 1 |

Circles: Octet top+bottom rows (wraps). Quad 2,3,6,7.

- Octet: AB 00→10, A changes, B=0, C/D out → `B'`.
- Quad 0010,0011,0110,0111: A=0, C=1 → `A'C`.

```text
F = B' + A'C
```

### 8.13 POS from K-map

Circle 0s, flip convention: always 0 → plain, always 1 → bar. AND sums.

`F = Σm(1,3,5,6,7)`:

SOP (1s): C (1,3,5,7) + AB (6,7) → `F = C + AB`.

| A\BC | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 0 | 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 1 | 1 |

POS (0s): boxes 0,4 → B=0,C=0 → `(B+C)`. Boxes 0,2 → A=0,C=0 → `(A+C)`.

```text
F = (B+C)(A+C)
Check: = AB+C ✓
```

### 8.14 Don't cares (X)

X = never happens / don't care. Use as 1 if helps, else 0. Never must-cover.

`F = Σm(1,3,6) + d(5,7)`:

| A\BC | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 0 | 0 | 1 | 1 | 0 |
| 1 | 0 | X | X | 1 |

C uses X5,X7 (quad). AB uses X7 (pair 6,7).

```text
F = C + AB
```

### 8.15 Implicants

- Implicant: any valid circle.
- Prime: cannot grow.
- Essential: only one covering some 1 → must use.
- Non-essential: covered elsewhere, use only if needed.

`F(A,B,C) = Σm(0,1,3,7)`:

| A\BC | 00 | 01 | 11 | 10 |
|---|---|---|---|---|
| 0 | 1 | 1 | 1 | 0 |
| 1 | 0 | 0 | 1 | 0 |

- `A'B'` (0,1) essential, only covers 0.
- `BC` (3,7) essential, only covers 7.
- `A'C` (1,3) prime, not needed.

```text
F = A'B' + BC
```

### 8.16 Checklist

1. Grid, Gray `00,01,11,10`.
2. 1s in boxes (X for don't care).
3. Biggest groups: 16,8,4,2,1. Wrap + corners.
4. Every 1 covered, fewest circles.
5. Read: 1=plain, 0=bar, change=drop.
6. OR terms. POS: circle 0s, flip, AND.

## 9. Cheat Sheet + Practice

| Topic | Remember |
|---|---|
| Order | Brackets, NOT, AND, OR |
| 0/1 | `A+0=A, A+1=1, A·1=A, A·0=0, A+A=A, A+A'=1, A·A'=0` |
| Shortcuts | `AB+AB'=A, A+AB=A, A+A'B=A+B` |
| De Morgan | `(A+B)'=A'B', (AB)'=A'+B'` |
| Redundancy | `AB+A'C+BC=AB+A'C` |
| Minterm | AND all, bar 0s. `Σm(1-rows)` |
| Maxterm | OR all, bar 1s. `ΠM(0-rows)` |
| Other list | missing numbers = other list |
| K labels | Gray `00,01,11,10`, wraps |
| K circles | 1,2,4,8,16, rectangles, biggest+fewest |
| Read | 1=plain, 0=bar, change=drop |
| POS | circle 0s, 0=plain 1=bar, AND |
| Prime | cannot grow, essential=covers lonely 1 |

Practice:

```text
1. Y = AB + AB' + A'B → A + B
2. Y = A'B + AC + BC → A'B + AC
3. Σm(0,3,5) → ΠM(1,2,4,6,7)
4. A'B + C → Σm(1,2,3,5,7)
5. Σm(0,1,4,5,6) → B' + AC'
6. Σm(0,2,5,7,8,10,13,15) → B'D' + BD
7. Q5 POS via 0s (2,3,7) → (A+B')(B'+C')
8. [(A+B')(C'+D)]' → A'B + CD'
```

---
[Index](README.md) | [← Prev](cheatsheet-1.md)
