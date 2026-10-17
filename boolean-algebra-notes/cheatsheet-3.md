# Cheatsheet 3: Digital Logic Gates

Source: Digital Logic Gates notes — extra version / sequel to the Boolean Algebra cheatsheets. Simple notes for complete beginners. Pages 1 and 2 are unchanged.

## 1. What Is a Logic Gate?

A gate looks at one or more inputs (each `0` or `1`) and produces one output (`0` or `1`) by a fixed rule. Everything a computer does is built from these.

`0` and `1` are voltage ranges, not abstractions. Classic 5 V TTL:

| Level | Meaning | Voltage |
|---|---|---|
| LOW | 0 / OFF | 0 V to about 0.8 V |
| HIGH | 1 / ON | about 2 V to 5 V |
| Forbidden | — | the gap between (never settle here) |

The wide ranges are why a little noise does not change a signal. Positive logic (HIGH = 1) is assumed everywhere below; negative logic flips it.

Every gate can be described three ways: its **symbol**, its **Boolean expression**, and its **truth table** (n inputs → 2ⁿ rows).

| Gate | Expression | Gate | Expression |
|---|---|---|---|
| NOT | `Y = A'` | NAND | `Y = (A·B)'` |
| Buffer | `Y = A` | NOR | `Y = (A + B)'` |
| AND | `Y = A·B` | XOR | `Y = A ⊕ B` |
| OR | `Y = A + B` | XNOR | `Y = (A ⊕ B)'` |

A bubble (small circle) on an output or input means inversion. A gate is a strict doorman: AND needs a ticket **and** an ID, OR needs a ticket **or** an ID, NOT does the opposite.

## 2. The Three Basic Gates

AND: 1 only if **all** inputs are 1. OR: 0 only if **all** inputs are 0. NOT: flips.

| A | B | AND `A·B` | OR `A + B` | NOT `A'` |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 1 |
| 0 | 1 | 0 | 1 | — |
| 1 | 0 | 0 | 1 | — |
| 1 | 1 | 1 | 1 | — |

- **NOT (inverter):** triangle with a bubble; `A'` is read "A bar" or "not A".
- **Buffer:** same triangle without the bubble, `Y = A`. Seems pointless, but it restores weak signals and lets one output drive many inputs.
- **AND:** named from English — "the lamp is ON if switch A **and** switch B are on". Series switches.
- **OR:** `1 + 1 = 1`, because "A or B" is still just true. Parallel switches.

**More than two inputs.** Gates are not limited to two. A 3-input AND is 1 only when all three are 1; a 3-input OR is 0 only when all three are 0. AND and OR are associative, so `ABCD = (AB)(CD)`.

**Order:** brackets, NOT, then AND, then OR.

## 3. NAND and NOR: The Universal Gates

NAND = NOT AND (0 only when all inputs are 1). NOR = NOT OR (1 only when all inputs are 0).

| A | B | AND | NAND `(A·B)'` | OR | NOR `(A + B)'` |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 1 | 0 | 1 |
| 0 | 1 | 0 | 1 | 1 | 0 |
| 1 | 0 | 0 | 1 | 1 | 0 |
| 1 | 1 | 1 | 0 | 1 | 0 |

Each column is its twin flipped.

**Why universal?** Every function can be written with AND, OR, NOT. If one gate type can build those three, it can build anything. NAND and NOR both can.

| Build | From NAND | From NOR |
|---|---|---|
| NOT | `(A·A)' = A'` | `(A + A)' = A'` |
| AND | `((A·B)')' = A·B` | `(A' + B')' = A·B` |
| OR | `(A'·B')' = A + B` | `((A + B)')' = A + B` |

NAND-based builds are the mirror (dual) of NOR-based builds.

**Why engineers prefer NAND.** In CMOS a NAND is simpler, smaller and faster than an AND, because AND is literally a NAND plus an inverter. A factory making one gate type is easier to run. Any sum-of-products `F = AB + CD` becomes `NAND(NAND(A,B), NAND(C,D))` in two levels: `((AB)'·(CD)')' = AB + CD` by De Morgan. Any product-of-sums maps onto two levels of NOR.

**Exam recipe ("implement using NAND only"):** (1) write as sum of products, (2) put bubbles on the AND outputs and OR inputs, (3) the bubbled OR is a NAND.

## 4. XOR and XNOR

XOR: 1 when the inputs **differ**. XNOR: 1 when the inputs are the **same** (the equivalence gate).

| A | B | XOR `A ⊕ B` | XNOR `(A ⊕ B)'` |
|---|---|---|---|
| 0 | 0 | 0 | 1 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

```text
A ⊕ B = A'B + AB'        (A ⊕ B)' = AB + A'B'
```

"Exclusive" means "one or the other, but not both".

**Useful identities:**

| Identity | Meaning |
|---|---|
| `A ⊕ 0 = A` | XOR with 0 leaves the bit alone |
| `A ⊕ 1 = A'` | XOR with 1 flips the bit (controllable inverter) |
| `A ⊕ A = 0` | signal XORed with itself is 0 |
| `A ⊕ A' = 1` | signal XORed with its opposite is 1 |
| `A ⊕ B = B ⊕ A` | commutative |
| `(A ⊕ B) ⊕ C = A ⊕ (B ⊕ C)` | associative |

**Many inputs = parity gate.** Chained XOR (`A ⊕ B ⊕ C ⊕ D`) is 1 when an **odd** number of inputs are 1, 0 when even. That is the parity bit for error detection.

**Where they are used:** adders (the sum bit is an XOR), comparators (XNOR tells equality), parity and error detection, toggle/conditional invert (`A ⊕ M`), and simple stream ciphers (`(message ⊕ key) ⊕ key = message`).

**XOR from four NAND gates:**

```text
N1: m      = (A·B)'
N2: (A·m)'
N3: (B·m)'
N4: Y = ((A·m)'·(B·m)')' = A·m + B·m = AB' + A'B = A ⊕ B
```

## 5. Bubbles and De Morgan Symbols

```text
(A·B)' = A' + B'        (A + B)' = A'·B'
```

Read them as drawings: a **NAND** is the same as an **OR with inverted inputs**, and a **NOR** is an **AND with inverted inputs**. Both drawings are legal; pick the one that reads better.

**Bubble pushing:** slide a bubble from a gate's output to its inputs (or back) and the gate flips between AND and OR. This converts circuits to NAND-only or NOR-only, and explains active-low pins such as chip-select `CS'`.

Remember: **NAND = bubbled-input OR, NOR = bubbled-input AND. Push the bubble through, change the gate.**

## 6. Reading Circuits, Truth Tables and Timing

Label intermediate wires and work left to right. Below, `X = AB` and `Z = C'`, so the final OR gives `Y = X + Z = AB + C'`.

| A | B | C | X = AB | Z = C' | Y = X + Z |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 1 | 1 |
| 0 | 0 | 1 | 0 | 0 | 0 |
| 0 | 1 | 0 | 0 | 1 | 1 |
| 0 | 1 | 1 | 0 | 0 | 0 |
| 1 | 0 | 0 | 0 | 1 | 1 |
| 1 | 0 | 1 | 0 | 0 | 0 |
| 1 | 1 | 0 | 1 | 1 | 1 |
| 1 | 1 | 1 | 1 | 0 | 1 |

A **truth table** gives the output for a fixed input. A **timing diagram** shows signals over time (what you see on an oscilloscope); at any vertical line the outputs are just the gate rules applied at that instant.

**All gates at a glance:**

| Gate | Expression | Output is 1 when… | Output is 0 when… |
|---|---|---|---|
| NOT | `A'` | A = 0 | A = 1 |
| AND | `A·B` | all inputs are 1 | any input is 0 |
| OR | `A + B` | any input is 1 | all inputs are 0 |
| NAND | `(A·B)'` | any input is 0 | all inputs are 1 |
| NOR | `(A + B)'` | all inputs are 0 | any input is 1 |
| XOR | `A ⊕ B` | inputs differ (odd number of 1s) | inputs are the same (even number of 1s) |
| XNOR | `(A ⊕ B)'` | inputs are the same | inputs differ |

## 7. Gates in the Real World

Gates are sold as ICs, a few per package. The classic family is **74xx**:

| IC | Contents | Gate |
|---|---|---|
| 7400 | 4 × 2-input | NAND |
| 7402 | 4 × 2-input | NOR |
| 7404 | 6 inverters | NOT |
| 7408 | 4 × 2-input | AND |
| 7432 | 4 × 2-input | OR |
| 7486 | 4 × 2-input | XOR |
| 7410 | 3 × 3-input | NAND |

- **Fan-in:** how many inputs a gate has. **Fan-out:** how many gate inputs one output can drive reliably.
- **Propagation delay:** the time between an input changing and the output responding. Every extra gate in a path adds delay and limits the clock speed — the reason fast adders (carry-lookahead) exist.

**TTL** (bipolar transistors) is fast but power-hungry; **CMOS** (complementary MOS) has almost no static power and is the basis of all modern processors. In CMOS each gate is a pull-up network and a pull-down network, only one ever on.

## 8. Practice Questions

1. **How many 2-input NAND gates build an XOR?** Four.
2. **Both inputs of a 2-input NOR tied together give?** A NOT: `(A + A)' = A'`.
3. **A 3-input XOR has inputs 1, 1, 1. Output?** 1 (three 1s is odd).
4. **Draw `F = AB + CD` with NAND only.** Three NANDs: `((AB)'(CD)')' = AB + CD`.
5. **A 2-input NAND with both inputs 1?** 0.
6. **What is `A ⊕ A'`?** 1 (they are always different).
7. **Rows in a 4-input gate truth table?** 2⁴ = 16.
8. **OR from NOR only?** Two NORs: `((A + B)')' = A + B`.

---
[Index](README.md) | [← Previous](cheatsheet-2.md)
