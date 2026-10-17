# Cheatsheet: Combinational Circuits

Source: Combinational Circuits notes (Sep 14, 2026) — Neso Academy combinational videos: adders, subtractors, NAND/NOR realisations, CLA and BCD adders, multiplier, comparator, parity, 7-segment decoder, MUX, DEMUX, encoders and decoders.

## 1. Idea and Design Procedure

Outputs depend **only** on current inputs. No memory, no clock, no feedback.

| Combinational | Sequential |
|---|---|
| Output = f(inputs only) | Output = f(inputs + past state) |
| No memory elements | Flip-flops / latches hold state |
| No feedback | Output feeds back |
| Examples: adder, MUX, decoder | Examples: counter, register, FSM |
| Described by truth table | Described by state table / diagram |

**Design procedure (always the same):**

1. Read the spec; decide inputs and outputs.
2. Write the truth table.
3. Derive expressions (SOP or POS).
4. Simplify (algebra or K-map) — fewer gates.
5. Draw the circuit; restrict to NAND-only or NOR-only if asked.
6. Verify a few rows.

Analysis is the reverse: label every gate output, write its expression, build the truth table.

## 2. Adders

Binary addition: `0+0=0`, `0+1=1`, `1+0=1`, `1+1=10` (sum 0, carry 1).

**Half adder** (adds two bits): `Sum = A ⊕ B`, `Carry = A·B`.

| A | B | Sum | Carry |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

"Half" because it cannot take a carry-in — it adds only two bits.

**Full adder** (adds three bits A, B, Cin): `Sum = A ⊕ B ⊕ Cin`, `Cout = AB + BCin + ACin`.

| A | B | Cin | Sum | Cout |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 0 |
| 0 | 1 | 0 | 1 | 0 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 | 0 |
| 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 0 | 0 | 1 |
| 1 | 1 | 1 | 1 | 1 |

- Sum = 1 when an **odd** number of inputs are 1 (parity, `Σm(1,2,4,7)`, a checkerboard K-map).
- Cout = 1 when at least **two** inputs are 1 (majority, `Σm(3,5,6,7)`).

Built from two half adders + OR: `S1 = A⊕B`, `C1 = AB`; `Sum = S1⊕Cin`, `C2 = S1·Cin`; `Cout = C1 + C2`.

**4-bit ripple-carry adder:** four full adders chained carry-out → carry-in. Carry ripples LSB → MSB. Slow: each stage waits for the one before (about 2 gate delays per stage; a 32-bit adder ≈ 64 delays).

**Carry-lookahead adder (CLA).** Compute carries in parallel from the inputs:

```text
Generate:   Gi = Ai·Bi          (this stage makes a carry)
Propagate:  Pi = Ai ⊕ Bi        (this stage passes a carry)
Carry:      Ci+1 = Gi + Pi·Ci   Sum:  Si = Pi ⊕ Ci
```

Expanded so every carry uses only G, P and C0:

```text
C1 = G0 + P0C0
C2 = G1 + P1G0 + P1P0C0
C3 = G2 + P2G1 + P2P1G0 + P2P1P0C0
C4 = G3 + P3G2 + P3P2G1 + P3P2P1G0 + P3P2P1P0C0
```

Each is two-level AND-OR, so all carries appear together (2-3 gate delays) instead of one after another.

| | Ripple-carry | Carry-lookahead |
|---|---|---|
| Carry computation | sequential, stage by stage | all carries in parallel |
| Speed | slow (grows with bits) | fast (nearly constant) |
| Hardware | small, simple | more gates, higher fan-in |
| Used for | small adders, low cost | high-speed ALUs, 4-bit blocks |

**BCD adder.** BCD uses only 0000-1001 (0-9) per digit; 1010-1111 are invalid. A binary adder counts to 16, so correct the result when it exceeds 9: add 0110 (six) when the sum is 10-15 **or** a carry-out occurred. Why six? 16 − 10 = 6.

```text
Correction (carry-out of BCD adder) = K + Z3·Z2 + Z3·Z1
```

`K` = carry out of the first 4-bit adder; `Z3Z2Z1Z0` = its sum. `Z3Z2` flags 12-15 and `Z3Z1` flags 10, 11, 14, 15. A second adder adds 0110 when the flag is 1.

Example 8 + 7: `1000 + 0111 = 1111` (15 > 9) → `1111 + 0110 = 1 0101` → BCD 15. 9 + 9: `1001 + 1001 = 1 0010` (K = 1) → `0010 + 0110 = 1000` → BCD 18.

## 3. Subtractors

**Half subtractor** (A − B, two bits): `Difference = A ⊕ B`, `Borrow = A'·B`.

| A | B | Difference | Borrow |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 0 |

Difference is the same XOR as the half adder; only Borrow is new (1 only when A = 0, B = 1). So a half subtractor is a half adder with an inverter in front of the AND.

**Full subtractor** (A − B − Bin): `Difference = A ⊕ B ⊕ Bin`, `Bout = A'B + A'Bin + B·Bin`.

| A | B | Bin | Diff | Bout |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 1 |
| 0 | 1 | 0 | 1 | 1 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 | 0 |
| 1 | 0 | 1 | 0 | 0 |
| 1 | 1 | 0 | 0 | 0 |
| 1 | 1 | 1 | 1 | 1 |

Difference = `Σm(1,2,4,7)`, Bout = `Σm(1,2,3,7)`. Built from two half subtractors + OR.

**One circuit that adds and subtracts.** `A − B = A + B' + 1`. Put an XOR on every B input with a control line M: `M = 0` → B unchanged, C0 = 0 → adds; `M = 1` → B flipped, C0 = 1 → computes `A + B' + 1 = A − B`. This is the controllable inverter idea.

## 4. Adders and Subtractors from NAND or NOR Only

The key trick is the **4-NAND XOR**; everything else extends it. Ask: which existing wire already equals the carry (or borrow), or its inverse?

| Circuit | NAND-only gates | NOR-only gates |
|---|---|---|
| Half adder | 5 | 5 |
| Full adder | 9 | ~9 (by duality) |
| Half subtractor | 5 | 5 |
| Full subtractor | 9 | — |

- **Half adder (NAND):** `N1 = (A·B)'`; `N2 = (A·n1)'`, `N3 = (B·n1)'`, `N4 = (n2·n3)' = A⊕B`; `N5 = (n1·n1)' = AB` (inverter for the carry).
- **Half adder (NOR):** `n1 = A'`, `n2 = B'`, `n3 = (A+B)' = A'B'`, `n4 = (n1+n2)' = AB` (carry), `n5 = (n3+n4)' = A⊕B`.
- **Full adder (NAND, 9):** `N1 = (A·B)'`, `N2 = (A·n1)'`, `N3 = (B·n1)'`, `N4 = A⊕B`, `N5 = (n4·Cin)'`, `N6 = (n4·n5)'`, `N7 = (Cin·n5)'`, `N8 = Sum = (n6·n7)'`, `N9 = Cout = (n1·n5)' = AB + (A⊕B)Cin`.
- **Half subtractor (NAND, 5):** XOR for Diff; `Borrow = (n3·n3)' = A'B`.
- **Half subtractor (NOR, 5):** `n1 = (A+B)'`; `n2 = (A+n1)' = A'B` (Borrow); `n3 = (B+n1)' = AB'`; `n4 = (n2+n3)'`; `Diff = (n4+n4)' = A⊕B`.
- **Full subtractor (NAND, 9):** XOR twice for Diff; `Bout = (n3·n7)' = A'B + Bin(A⊕B)'`.

## 5. Multiplier, Comparator, Parity and 7-Segment Decoder

**2-bit binary multiplier.** Each partial product is one bit × one bit = an AND gate. Then add columns with half adders.

- `P0 = A0B0`
- `P1, c1 = HA(A1B0, A0B1)`
- `P2, P3 = HA(A1B1, c1)`

Total: four AND gates and two half adders. Largest case 3 × 3 = 9 = 1001.

**Comparator.** Per bit: `G (A>B) = A·B'`, `E (A=B) = A⊙B = AB + A'B'`, `L (A<B) = A'·B`.

2-bit (`x1 = A1⊙B1`, `x0 = A0⊙B0`):

```text
A > B: A1B1' + x1·A0B0'
A = B: x1·x0
A < B: A1'B1 + x1·A0'B0
```

Read as: "the top bits differ and decide, or the top bits are equal and the low bits decide." Chips like the 7485 cascade to make 8-bit comparators.

**Parity generator/checker.** Add one bit so the number of 1s is always even (even parity) or odd. `P = A ⊕ B ⊕ C ⊕ D` (XOR of all data bits). The checker XORs all received bits including the parity bit; `0` = OK for even parity, `1` = error. Detects any single-bit error, not which bit, and misses double-bit errors.

**BCD to 7-segment decoder.** Seven segments a-g; common cathode (drive HIGH to light) vs common anode (drive LOW; invert every output, e.g. chip 7447). Inputs 1010-1111 are don't-cares in the K-maps.

| Segment | Expression (A is MSB) |
|---|---|
| a | `A + C + BD + B'D'` |
| b | `B' + CD + C'D'` |
| c | `B + C' + D` |
| d | `A + B'D' + CD' + B'C + BC'D` |
| e | `B'D' + CD'` |
| f | `A + C'D' + BC' + BD'` |
| g | `A + BC' + B'C + CD'` |

Seven separate functions sharing the same four inputs; minimise each on its own K-map.

## 6. Multiplexers (MUX)

A MUX (data selector) sends one of 2ⁿ inputs to a single output, chosen by n select lines. `4×1` = 4 inputs, 1 output.

| MUX | Data inputs | Selects | Output |
|---|---|---|---|
| 2×1 | 2 | 1 | 1 |
| 4×1 | 4 | 2 | 1 |
| 8×1 | 8 | 3 | 1 |
| 2ⁿ×1 | 2ⁿ | n | 1 |

```text
2×1: Y = S'·I0 + S·I1
4×1: Y = S1'S0'·I0 + S1'S0·I1 + S1S0'·I2 + S1S0·I3
```

**Master pattern:** a MUX is a minterm generator on the selects, each minterm ANDed with its data input, all ORed.

**MUX trees:**

| Target | From | Count | Levels (selects) |
|---|---|---|---|
| 4×1 | 2×1 | 3 | S0 then S1 |
| 8×1 | 2×1 | 7 (N−1) | S0, S1, S2 |
| 8×1 | 4×1 | 3 (one half-used) | S1S0 then S2 |
| 16×1 | 4×1 | 5 | S1S0 then S3S2 |
| 32×1 | 8×1 | 5 | S2S1S0 then S4S3 |

**Implementing a function with a MUX:**

- **Method 1:** as many selects as variables; tie each data input to 1 if that minterm is in the function, else 0. Easy but wasteful.
- **Method 2 (smaller, 2ⁿ⁻¹×1):** use n−1 variables as selects and feed the last variable (C, C', 0 or 1) to the data inputs. For each select combination, look at the two rows differing only in C: both 0 → input 0; both 1 → 1; output follows C → C; output opposite → C'.

Example `F(A,B,C) = Σm(1,3,5,6)` with a 4×1 (A on S1, B on S0): I0 = I1 = I2 = C, I3 = C'.

**Full adder with two 4×1 MUXes (A, B as selects):**

| AB | Sum input | Cout input |
|---|---|---|
| 00 | Cin | 0 |
| 01 | Cin' | Cin |
| 10 | Cin' | Cin |
| 11 | Cin | 1 |

**Reverse problem** — given the circuit, write `Σ(select minterm × data input)`.

## 7. Demultiplexers (DEMUX)

One input → one of 2ⁿ outputs, chosen by n selects. Mirror image of a MUX (one-to-many).

```text
1×2: Y0 = S'·D,      Y1 = S·D
1×4: Y0 = S1'S0'·D,  Y1 = S1'S0·D,  Y2 = S1S0'·D,  Y3 = S1S0·D
```

| | MUX | DEMUX |
|---|---|---|
| Direction | many → one | one → many |
| I/O | 2ⁿ data in, 1 out | 1 data in, 2ⁿ out |
| Other name | data selector | data distributor |
| Special use | implements functions | decoder when D = 1 |

**As a decoder:** set D = 1 and the outputs become the select minterms.

**Function with a DEMUX:** a 1×8 DEMUX (D = 1) with A, B, Bin on the selects gives a full subtractor: `Difference = Y1+Y2+Y4+Y7`, `Borrow = Y1+Y2+Y3+Y7`.

## 8. Decoders and Encoders

**Decoder:** n-bit input → activates exactly one of 2ⁿ outputs (its own number). 2-to-4:

| EN | A | B | Y0 | Y1 | Y2 | Y3 |
|---|---|---|---|---|---|---|
| 0 | X | X | 0 | 0 | 0 | 0 |
| 1 | 0 | 0 | 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 0 | 1 | 0 | 0 |
| 1 | 1 | 0 | 0 | 0 | 1 | 0 |
| 1 | 1 | 1 | 0 | 0 | 0 | 1 |

`Y0 = A'B'·EN`, `Y1 = A'B·EN`, `Y2 = AB'·EN`, `Y3 = AB·EN`. Four AND gates + two inverters. Two 3-to-8 decoders + enable bit make a 4-to-16.

**Decoder as function generator:** OR the minterm outputs. Full adder: 3-to-8 decoder, `Sum = OR(Y1,Y2,Y4,Y7)`, `Cout = OR(Y3,Y5,Y6,Y7)`. Active-low outputs → use NAND instead of OR.

**Encoder:** reverse — 2ⁿ inputs (exactly one active) → n-bit code.

- 8-to-3: `Y2 = D4+D5+D6+D7`, `Y1 = D2+D3+D6+D7`, `Y0 = D1+D3+D5+D7` (D0 is unused: code 000).
- Decimal-to-BCD (10→4): `A = D8+D9`, `B = D4+D5+D6+D7`, `C = D2+D3+D6+D7`, `D = D1+D3+D5+D7+D9`.
- Plain encoders fail if two inputs are active (garbage) and cannot tell "line 0" from "nothing".

**Priority encoder** — rank the inputs; output the highest-priority active one; add a valid bit V. 4-to-2 (D3 highest):

| D3 | D2 | D1 | D0 | Y1 | Y0 | V |
|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | X | X | 0 |
| 0 | 0 | 0 | 1 | 0 | 0 | 1 |
| 0 | 0 | 1 | X | 0 | 1 | 1 |
| 0 | 1 | X | X | 1 | 0 | 1 |
| 1 | X | X | X | 1 | 1 | 1 |

`Y1 = D3 + D2`, `Y0 = D3 + D2'·D1`, `V = D3 + D2 + D1 + D0`. Used in interrupt controllers.

| | Decoder | Encoder |
|---|---|---|
| Direction | code → one-hot line | one-hot line → code |
| I/O | n in, 2ⁿ out | 2ⁿ in, n out |
| Gates | AND (minterms) | OR |
| Uses | memory addressing, function generation, displays | keypads, interrupt controllers |

## 9. Summary of the Circuits

| Circuit | I/O | Key equations |
|---|---|---|
| Half adder | 2→2 | `S = A⊕B`, `C = AB` |
| Full adder | 3→2 | `S = A⊕B⊕Cin`, `Cout = AB + BCin + ACin` |
| CLA | per bit | `G = AB`, `P = A⊕B`, `Ci+1 = Gi + PiCi` |
| Half subtractor | 2→2 | `D = A⊕B`, `Bo = A'B` |
| Full subtractor | 3→2 | `D = A⊕B⊕Bin`, `Bo = A'B + A'Bin + B·Bin` |
| BCD adder | 2 digits → digit + carry | add 0110 when `K + Z3Z2 + Z3Z1 = 1` |
| 2-bit multiplier | 4→4 | 4 ANDs + 2 half adders |
| Comparator | A,B → G,E,L | `G = AB'`, `E = A⊙B`, `L = A'B` |
| Parity generator | n→1 | `P = XOR of all bits` |
| MUX 2ⁿ×1 | 2ⁿ+n → 1 | `Y = Σ(select minterm × input)` |
| DEMUX 1×2ⁿ | 1+n → 2ⁿ | `Yi = (minterm i)·D` |
| Decoder n→2ⁿ | n → 2ⁿ | `Yi = minterm i` |
| Encoder 2ⁿ→n | 2ⁿ → n | `output bit = OR of lines with that bit set` |
| Priority encoder | 4→2+V | `Y1 = D3+D2`, `Y0 = D3+D2'D1`, `V = ΣD` |

---
[Index](README.md)
