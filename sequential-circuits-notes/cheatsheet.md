# Cheatsheet: Sequential Circuits

Source: Sequential Circuits notes (Oct 15, 2026) — Neso Academy sequential videos: SR/D latches, clocks and triggering, SR/D/JK/T flip-flops, race-around, master-slave, conversions, state tables and diagrams, Mealy and Moore machines, sequence detectors, ASM charts, counters, registers, PLA and PAL.

## 1. Idea: Circuits That Remember

Outputs depend on current inputs **and** the stored past (the **state**). The state lives in latches or flip-flops and is fed back into the logic.

| Combinational | Sequential |
|---|---|
| Output = f(inputs) | Output = f(inputs + state) |
| No memory, no feedback | Flip-flops / latches store state |
| No clock (usually) | Clocked (edge-triggered) usually |
| Described by truth table | Described by state table / diagram |

| | Synchronous | Asynchronous |
|---|---|---|
| State changes | only at a clock edge | any time an input changes |
| Timing | global clock | none (gate delays) |
| Memory | flip-flops | latches / unclocked feedback |
| Design | easier, predictable | hazards and races; harder |
| Examples | counters, registers, CPUs | SR latch, ripple counter |

Almost all real systems are **synchronous** — the clock keeps everything in step.

## 2. Latches

Two cross-coupled gates hold each other's answer → one bit of memory.

**SR latch, NOR gates (active-high inputs):**

| S | R | Q (next) | Action |
|---|---|---|---|
| 0 | 0 | Q | hold |
| 0 | 1 | 0 | reset |
| 1 | 0 | 1 | set |
| 1 | 1 | invalid | forbidden |

**SR latch, NAND gates (active-low inputs, written S', R'):**

| S' | R' | Q (next) | Action |
|---|---|---|---|
| 1 | 1 | Q | hold |
| 0 | 1 | 1 | set |
| 1 | 0 | 0 | reset |
| 0 | 0 | invalid | forbidden |

The `11` case is forbidden because both outputs are forced to 0 (breaking `Q` vs `Q'`), and releasing both together gives an unpredictable result.

**Gated (clocked) SR latch:** an enable `EN` in front; two AND gates pass S and R only when `EN = 1`, otherwise the latch holds.

**D latch:** make the inputs opposites (`S = D`, `R = D'`) so the forbidden state can never happen.

| EN | D | Q (next) | Behaviour |
|---|---|---|---|
| 0 | X | Q | holds (latched) |
| 1 | 0 | 0 | follows D (transparent) |
| 1 | 1 | 1 | follows D (transparent) |

When enabled the latch is **transparent** — changes in D flow straight to Q.

## 3. Clocks, Triggering, Latch vs Flip-Flop

A clock is a steady square wave: the heartbeat that tells every flip-flop when to update.

- **Period** T; **frequency** f = 1/T (Hz). A 1 MHz clock has a 1 µs period.
- **Duty cycle:** fraction of the period spent high (50% for a symmetric square wave).
- **Rising (positive) edge:** 0 → 1. **Falling (negative) edge:** 1 → 0.

**Triggering** — when the element may change:

- **Level triggering:** responds the whole time the clock is at a level (latches).
- **Edge triggering:** responds only at a clock edge (flip-flops). A triangle at the clock pin marks edge triggering; a bubble in front means falling edge.

| | Latch | Flip-flop |
|---|---|---|
| Triggering | level sensitive | edge triggered |
| Output changes | any time enabled | only at the edge |
| Transparent? | yes, while enabled | no |
| Built from | gates with feedback | two latches (master-slave) or edge-detect |
| Use | simple storage | counters, registers, state machines |

Edge behaviour is what makes synchronous design work: logic has a whole clock period to settle before the next snapshot.

## 4. Flip-Flops

A flip-flop is one-bit memory that changes only on a clock edge. Each type is fully described by its **characteristic table/equation** (given Q and inputs, what is Q⁺?) and its **excitation table** (to go from Q to Q⁺, what inputs are needed?).

**SR flip-flop:** `Q+ = S + R'·Q` (with `S·R = 0`).

| S | R | Q+ | Action |
|---|---|---|---|
| 0 | 0 | Q | no change |
| 0 | 1 | 0 | reset |
| 1 | 0 | 1 | set |
| 1 | 1 | X | invalid |

**D flip-flop:** `Q+ = D`. Excitation: `D = Q+`. The output is the input delayed by one clock. Registers and state-machine memory are built from these.

**JK flip-flop:** `Q+ = J·Q' + K'·Q`.

| J | K | Q+ | Action |
|---|---|---|---|
| 0 | 0 | Q | no change |
| 0 | 1 | 0 | reset |
| 1 | 0 | 1 | set |
| 1 | 1 | Q' | toggle |

**T (toggle) flip-flop:** `Q+ = T ⊕ Q`. Excitation: `T = Q ⊕ Q+`.

| T | Q+ | Action |
|---|---|---|
| 0 | Q | no change |
| 1 | Q' | toggle |

With `T = 1` it halves the clock frequency — chain several and you have a counter.

| Flip-flop | Characteristic | 0→0 | 0→1 | 1→0 | 1→1 |
|---|---|---|---|---|---|
| SR | `Q+ = S + R'Q` (SR=0) | S=0, R=X | S=1, R=0 | S=0, R=1 | S=X, R=0 |
| D | `Q+ = D` | D=0 | D=1 | D=0 | D=1 |
| JK | `Q+ = JQ' + K'Q` | J=0, K=X | J=1, K=X | J=X, K=1 | J=X, K=0 |
| T | `Q+ = T ⊕ Q` | T=0 | T=1 | T=1 | T=0 |

**Race-around:** in a level-triggered JK with `J = K = 1`, the output toggles, feeds back and toggles again while the clock stays high — the final value is random. Happens when the pulse is longer than the propagation delay. Cures: edge-triggered flip-flops, a master-slave structure, or an impractically short pulse.

**Master-slave JK:** two latches in series. The **master** is enabled while `CLK = 1` and captures J, K; the **slave** is enabled while `CLK = 0` and copies the master. Q changes at the falling edge, when the master is already closed, so the new output cannot sneak back. This is effectively a falling-edge triggered flip-flop. The same trick with D latches gives a negative-edge-triggered D flip-flop.

## 5. Flip-Flop Conversion and Asynchronous Inputs

If your design needs a type you do not have, add logic in front of the one you do have.

**Procedure:** (1) write the required flip-flop's characteristic table; (2) take the available flip-flop's excitation table; (3) find the inputs needed for each Q → Q⁺; (4) simplify (K-map) as functions of the new inputs and Q; (5) draw the circuit.

| Required | Available | Inputs needed |
|---|---|---|
| D | JK | `J = D`, `K = D'` |
| D | T | `T = D ⊕ Q` |
| T | JK | `J = K = T` |
| JK | SR | `S = J·Q'`, `R = K·Q` |
| T | SR | `S = T·Q'`, `R = T·Q` |
| D | SR | `S = D`, `R = D'` |

**Preset (PRE) and Clear (CLR)** force a known power-up state. They are **asynchronous** — they act immediately, regardless of the clock.

| PRE' | CLR' | CLK | Q |
|---|---|---|---|
| 0 | 1 | X | 1 (preset) |
| 1 | 0 | X | 0 (cleared) |
| 0 | 0 | X | invalid |
| 1 | 1 | edge | normal operation |

Active-low in most chips; held at 1 in normal operation. They start counters at a known number and clear registers.

## 6. Synchronous Sequential Circuits and State Machines

A sequential circuit is fully described by three equivalent views:

- **State equations:** next value per flip-flop, e.g. `A(t+1) = A·x + B·x`.
- **State table:** rows are (present state, input); columns give next state and output.
- **State diagram:** circles are states, arrows labelled input (and output for Mealy).

**Mealy vs Moore:**

| | Moore | Mealy |
|---|---|---|
| Output depends on | present state only | present state + input |
| Output written | inside the state | on the transition |
| Typical states | more | fewer |
| Output timing | changes at clock edges (clean) | can change with the input (may glitch) |
| Reaction | one clock later | immediate |
| Easier to | design and debug | make compact |

**Analysis procedure** (given a circuit): (1) write each flip-flop's input equations; (2) write the output equations; (3) apply the characteristic equation (`D: Q+=D`, `JK: Q+ = JQ'+K'Q`, `T: Q+ = T⊕Q`); (4) build the state table; (5) draw the state diagram and describe it.

**Design procedure:** (1) specification → state diagram, pick Mealy or Moore; (2) state table; (3) state reduction; (4) state assignment; (5) pick flip-flop type and use its excitation table; (6) simplify flip-flop and output equations with K-maps (unused codes are don't-cares); (7) draw the circuit.

**Sequence detector example — detect 101 (overlapping).** States: S0 = nothing, S1 = "1", S2 = "10". From S2 an input 1 completes 101; with overlap that 1 starts the next match, so go to S1.

Mealy state table:

| Present | x = 0: next / z | x = 1: next / z |
|---|---|---|
| S0 | S0 / 0 | S1 / 0 |
| S1 | S2 / 0 | S1 / 0 |
| S2 | S0 / 0 | S1 / 1 |

Assign S0 = 00, S1 = 01, S2 = 10 (11 is a don't-care). With D flip-flops:

```text
D1 = Q0·x'      D0 = x      z = Q1·x
```

Two D flip-flops, two AND gates, one inverter. On stream `1 0 1 0 1 1 0 1` the output is `0 0 1 0 1 0 0 1` (fires on the overlapping match too).

Moore version needs a fourth state S3 holding `Z = 1`: `D1 = Q0·x' + Q1·Q0'·x`, `D0 = x`, `Z = Q1·Q0`. Costs one more state and one clock of delay, but no output glitches.

**State reduction:** two states are equivalent if they have the same output and, for every input, go to equivalent next states. Merge equivalents (fewer flip-flops). Larger tables use an implication table.

**State assignment:** N states need at least ⌈log₂N⌉ flip-flops.

| Method | 4 states | Flip-flops | Comment |
|---|---|---|---|
| Binary | 00, 01, 10, 11 | 2 | fewest flip-flops, logic may be larger |
| Gray | 00, 01, 11, 10 | 2 | neighbours differ by one bit, fewer glitches |
| One-hot | 0001, 0010, 0100, 1000 | 4 | simple and fast; popular in FPGAs |

Unused codes are treated as don't-cares.

**ASM charts** describe the same behaviour as a flowchart, with three symbols: a **state box** (state name + Moore outputs), a **decision box** (tests an input, exits 0/1), and a **conditional output box** (Mealy-type output, reached only through the decisions). One ASM block = one clock period.

## 7. Counters

A counter steps through a sequence of states, one per clock.

- n flip-flops → up to 2ⁿ states.
- **Modulus (MOD-N):** number of states in one cycle (3-bit binary = MOD-8; decade = MOD-10).
- Direction: up, down, or up/down. Clocking: asynchronous (ripple) or synchronous.

| | Asynchronous (ripple) | Synchronous |
|---|---|---|
| Clock | only the first flip-flop gets it | all flip-flops share the clock |
| Speed | slow (delays add up) | fast (all change together) |
| Hardware | simple, no extra gates | needs extra gates |
| Glitches | brief false intermediate states | clean |
| Design | easy | state table + excitation tables |

**3-bit asynchronous up counter:** chain T flip-flops with `T = 1` (toggle every clock); each is clocked by the previous output. Q0 = clock ÷ 2, Q1 = ÷ 4, Q2 = ÷ 8, so `Q2Q1Q0` counts up. A 4-bit version is MOD-16; the nth output divides by 2ⁿ (a 32.768 kHz crystal → 1 Hz with 15 flip-flops).

*Ripple drawback:* the last bit changes about n·t_pd after the edge, so the counter briefly shows false values (011 → 100 passes through 010, 000). The clock period must exceed n·t_pd.

**Down counter:** clock each stage from the previous `Q'` instead of Q (falling-edge triggered). **Up/down:** select with a MUX: `CLKi+1 = M·Qi + M'·Qi'` (M = 1 up, M = 0 down).

**MOD-N by clear:** watch for the first unwanted count and NAND its 1-bits into the active-low clear inputs.

| Counter | Flip-flops | Sequence | Detect (NAND of) | Clear at |
|---|---|---|---|---|
| MOD-6 | 3 | 000-101 | Q2, Q1 | 110 (6) |
| MOD-10 (decade) | 4 | 0000-1001 | Q3, Q1 | 1010 (10) |
| MOD-12 | 4 | 0000-1011 | Q3, Q2 | 1100 (12) |
| MOD-13 (count to 12) | 4 | 0000-1100 | Q3, Q2, Q0 | 1101 (13) |

The cleared state exists briefly (a glitch); synchronous design avoids it. Cascading MOD-A with MOD-B gives MOD-(A×B).

**Synchronous up counter (T flip-flops):** `T = Q ⊕ Q+`; a bit toggles when **all lower bits are 1** (like a carry):

```text
T0 = 1      T1 = Q0      T2 = Q1·Q0      (4-bit: T3 = Q2·Q1·Q0)
```

With JK, set `J = K = T`. For up/down: `T1 = Up·Q0 + Down·Q0'`, `T2 = Up·Q1Q0 + Down·Q1'Q0'`. Any sequence works the same way, unused states as don't-cares.

**Ring counter:** a shift register with the last output fed back to the first, so a single 1 circulates. n flip-flops → MOD-n, outputs one-hot. No decoding needed (great as a sequencer), but inefficient (n states instead of 2ⁿ) and must be initialised.

**Johnson (twisted-ring) counter:** feed back the **complement** of the last output. Starting from 0000 it fills with 1s then 0s: `0000 → 1000 → 1100 → 1110 → 1111 → 0111 → 0011 → 0001 → 0000`. n flip-flops → 2n states (MOD-2n), and every state decodes with a 2-input AND.

| Counter | Flip-flops for N states | States | Decoding |
|---|---|---|---|
| Binary | log₂ N | 2ⁿ | n-input gates |
| Ring | N | n | none (one-hot) |
| Johnson | N/2 | 2n | 2-input gates |

## 8. Registers and Shift Registers

A register is n flip-flops with a common clock, holding n bits. A **load (enable)** input decides whether new data is accepted or the old value feeds back.

| Type | Input | Output | Load time | Typical use |
|---|---|---|---|---|
| SISO | serial | serial | n clocks | delay lines |
| SIPO | serial | parallel | n clocks | serial→parallel (receivers) |
| PISO | parallel | serial | 1 load + n shift | parallel→serial (transmitters) |
| PIPO | parallel | parallel | 1 clock | plain storage |

- **SISO:** each flip-flop copies the one before it; data enters left and leaves right. A 4-bit register delays the stream by 4 clocks.
- **SIPO:** same hardware; read all outputs at once after n clocks to get the parallel word (UART, SPI receivers).
- **PISO:** a 2×1 MUX on each D input, `Di = Load·Pi + Load'·Qi−1`; Load = 1 captures all bits, Load = 0 shifts them out.
- **PIPO:** each flip-flop has its own input and output, no shifting; fastest, the CPU's working registers.
- **Bidirectional:** `Di = M·Qi−1 + M'·Qi+1` (M = 1 shift right, M = 0 shift left). Shifting left doubles a number, right halves it.
- **Universal shift register (e.g. 74194):** a 4×1 MUX per flip-flop, two mode lines.

| S1 | S0 | Operation |
|---|---|---|
| 0 | 0 | hold |
| 0 | 1 | shift right |
| 1 | 0 | shift left |
| 1 | 1 | parallel load |

## 9. Programmable Logic: PLA and PAL

Every function is a sum of products, so a fixed **AND plane** (product terms) plus an **OR plane** (sums) can implement it. Crossings are programmable (fuses, transistors, memory cells).

| Device | AND plane | OR plane | Comment |
|---|---|---|---|
| PROM | fixed (all minterms) | programmable | simple, but doubles with every input |
| PLA | programmable | programmable | most flexible; terms shared between outputs |
| PAL | programmable | fixed | cheaper, faster; each output has a fixed number of terms |

**PLA example** — `F1 = AB + A'C`, `F2 = AB + BC`; product terms `P1 = AB`, `P2 = A'C`, `P3 = BC`, with P1 shared:

| Product | A B C | F1 | F2 |
|---|---|---|---|
| P1 = AB | 11- | 1 | 1 |
| P2 = A'C | 0-1 | 1 | 0 |
| P3 = BC | -11 | 0 | 1 |

In input columns, 1 = true, 0 = complemented, dash = unused. A PLA with n inputs, k product terms and m outputs has 2n×k AND-plane points and k×m OR-plane points. PAL cannot share terms between outputs. Later: GAL (erasable PAL), CPLD (PAL blocks + wiring), FPGA (lookup tables + flip-flops).

## 10. Practice Problems

1. **JK, J = K = 1, Q = 0, after 3 pulses?** Toggles 1, 0, 1 → Q = 1.
2. **T flip-flop with T = 1 clocked at 8 kHz?** Q = 4 kHz.
3. **3-stage ripple counter at 16 kHz — frequency of Q2?** 16/2³ = 2 kHz.
4. **MOD-12 counter: flip-flops and reset point?** 4 flip-flops; reset at 12 = 1100 (NAND of Q3, Q2).
5. **4-bit ring from 1000 after 3 pulses? 4-bit Johnson from 0000?** Ring 0001; Johnson 1110.
6. **Convert a D flip-flop into a T flip-flop?** `D = T ⊕ Q` — one XOR gate.
7. **Forbidden NOR SR combo?** S = R = 1: both outputs 0, unpredictable on release.
8. **When does race-around occur?** Level-triggered JK, J = K = 1, clock pulse longer than t_pd.
9. **4-bit SIPO receives 1,0,0,1 (first bit first) after 4 clocks?** Q0 Q1 Q2 Q3 = 1 0 0 1.
10. **States for an overlapping 1101 Mealy detector?** Four (matched 0, 1, 11, 110).
11. **Flip-flops for a 10-state Johnson counter?** 5 (2n states).
12. **8-bit register 0000 0011 shifted left twice?** 0000 1100 = 12.

## 11. Summary

| Circuit | Key fact |
|---|---|
| SR latch (NOR) | S = 1 sets, R = 1 resets, 00 holds, 11 invalid |
| D latch / flip-flop | Q follows D (while enabled / at the edge) |
| Master-slave JK | no race-around; output changes at the clock fall |
| Mealy / Moore | output on arrows (fewer states) / output in states |
| Ripple counter | n stages: f/2ⁿ; delay n·t_pd |
| Synchronous counter | `T = Q ⊕ Q+`; a bit toggles when all lower bits are 1 |
| Ring / Johnson | n states / 2n states; Johnson feeds back Q' |
| SISO / SIPO / PISO / PIPO | serial or parallel in, serial or parallel out |
| PLA / PAL | AND + OR planes; PLA both programmable, PAL only AND |

---
[Index](README.md)
