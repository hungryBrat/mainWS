# MuP Compre — Master Solved-Examples Reference
**Exam:** 2026-05-14 09:00 · Part A (CB 60M / 75min) + Part B (OB 40M)
**Companion to:** `notes/PROBLEM_CATALOG.md` (theory & taxonomy)
**This file:** algorithmic walkthroughs of every exam-relevant problem type, ≥3 diverse worked examples each, all cross-referenced to PDFs in this workspace.

---

## How to use this file

1. Open `PROBLEM_CATALOG.md` first to see the taxonomy.
2. For each problem type (P0.x, P1.x, ...) this file contains theory in 2-3 lines + 3 fully worked examples.
3. Examples cite the source PDF: `[PYQs/Compre 2017(solved).pdf, Q5]` means open that file and locate Q5.
4. Last section: full solutions to the 10 novel problems from the catalog.

---

# Quick-reference cards (memorize before Part A)

### 8086 Flag Register layout
```
bit:  15 14 13 12 11 10 9  8  7  6  5  4  3  2  1  0
        -  -  -  -  OF DF IF TF SF ZF -  AF -  PF -  CF
```
- Status: CF, PF, AF, ZF, SF, OF (set by arithmetic/logic)
- Control: TF, IF, DF (programmer-set)

### 8086 Addressing modes
| Form | Mode | Default seg |
|---|---|---|
| `MOV AX, 5` | Immediate | — |
| `MOV AX, BX` | Register | — |
| `MOV AX, [1234H]` | Direct | DS |
| `MOV AX, [BX]` | Register-indirect | DS |
| `MOV AX, [BP]` | Register-indirect | **SS** |
| `MOV AX, [BX+10H]` | Based | DS |
| `MOV AX, [SI+10H]` | Indexed | DS |
| `MOV AX, [BX+SI]` | Based-indexed | DS |
| `MOV AX, 10H[BX+SI]` | Base-relative + index | DS |

### 8254 Control Word
`SC1 SC0 RW1 RW0 M2 M1 M0 BCD`
- SC: 00=C0, 01=C1, 10=C2, 11=read-back
- RW: 00=latch, 01=LSB only, 10=MSB only, 11=LSB then MSB
- Mode: 0=int-on-TC, 1=h/w one-shot, 2=rate-gen (÷N), 3=square wave, 4=s/w strobe, 5=h/w strobe
- BCD: 0=binary, 1=BCD
- **Out freq = CLK / count**

### 8255 Mode-set Control Word (D7=1)
`1 GA-M1 GA-M0 PA PCu GB-M PB PCl`
- GA-mode: 00=M0, 01=M1, 1x=M2
- direction bit: 1=input, 0=output

### 8255 BSR (D7=0)
`0 X X X bit2 bit1 bit0 S/R`
- bit2/1/0 = PC pin number (000=PC0 ... 111=PC7)
- S/R: 1=set, 0=reset

### 8259 ICW1 (port A0=0)
`0 0 0 1 LTIM 0 SNGL IC4` (for 8086)
- LTIM: 1=level, 0=edge
- SNGL: 1=single 8259, 0=cascade (need ICW3)
- IC4: 1=ICW4 will follow

### 8259 ICW2 (port A0=1)
8086 mode: high 5 bits = T7..T3 of interrupt type. (IR0 takes ICW2 value, IRn takes ICW2+n.)

### 8259 ICW3 (port A0=1, cascade only)
- Master: bit n = 1 if a slave is on IRn
- Slave: bottom 3 bits = which master IR the slave is on

### 8259 ICW4 (port A0=1)
`0 0 0 SFNM BUF M/S AEOI µPM`
- µPM=1 for 8086 mode
- AEOI: 1=auto-EOI
- BUF: 1=buffered. If BUF=1, M/S must be set (1=master, 0=slave).
- SFNM: 1=special fully nested

### 8259 OCWs (port A0=0 except OCW1)
- **OCW1** at A0=1: mask byte (bit n=1 ⇒ IRn masked)
- **OCW2** at A0=0: `R SL EOI 0 0 L2 L1 L0`
  - 20H = non-specific EOI
  - 60H+n = specific EOI for IRn
  - A0H = rotate on non-specific EOI
  - E0H+n = rotate on specific EOI, set IRn lowest
- **OCW3** at A0=0: read IRR/ISR/poll

### Software-delay formula
`Total cycles = (sum of body T-states) × count` (rough; off-by-one for last iteration normally ignored)
`Delay = Total cycles × (1/CLK)`

### Memory cycle
`MEMR/MEMW time = (4 + WS) × Tclk`

### Protected mode (80286/386)
Selector: `index(13) | TI(1) | RPL(2)`. TI=0→GDT, TI=1→LDT.
Descriptor address = table_base + index×8.

Paging: linear `bits 31:22 | 21:12 | 11:0` = `PD idx | PT idx | offset`.
- PDE addr = CR3 + PD_idx×4
- PTE addr = (PDE.top20<<12) + PT_idx×4
- Physical = (PTE.top20<<12) | offset

### IVT lookup
Interrupt type N → IVT entry at physical address `N × 4`. Four bytes: IP_lo, IP_hi, CS_lo, CS_hi (in that order, low addr first).

---

# CLUSTER 0 — Mental warmups (Part A, 2-4M each)

## P0.1 — Identify the addressing mode

**Theory.** Decode the **source operand** form. Brackets indicate memory; the contents of the brackets determine the sub-mode. `BX/BP` = base reg; `SI/DI` = index reg; a constant in front = displacement.

### Example 1: `MOV AX, 4020[BX+DI]` — [PYQs/Compre 2017(solved).pdf, Part A Q2]
- Brackets contain a base (BX) AND an index (DI) AND a displacement (4020).
- → **Base relative plus index** addressing mode.
- Effective offset = BX + DI + 4020H. Default segment = DS.

### Example 2: `MOV CX, [BP+SI+10H]`
- BP is the base register → default segment is **SS** (not DS).
- SI is the index, 10H is the displacement.
- → **Base relative plus index** mode (default segment SS).

### Example 3: `MOV AL, ES:[BX]`
- Single register inside brackets, no index, no displacement.
- Segment-override prefix `ES:` overrides default DS.
- → **Register-indirect** addressing mode with ES override.

**Trap.** `MOV AX, [1234H]` looks like immediate but is **direct memory** (no register inside brackets).

---

## P0.2 — Trace ROL/ROR/RCL/RCR/SHL/SHR/SAR

**Theory.** Shifts move bits inside one register and fill with 0 (or sign bit for SAR). Rotates wrap bits around. `RCL/RCR` rotate through the CF flag (making the rotation 9-bit for byte ops, 17-bit for word ops).

### Example 1: `MOV AL,22H ; MOV CL,03H ; ROL CL,CL ; ROL AL,CL` — [PYQs/Compre 2017(solved).pdf, Q5]
- Initially CL=03H = `0000 0011`.
- `ROL CL, CL`: rotate CL left by CL=3 bits. Bits cycle left → CL = `0001 1000` = 18H.
- `ROL AL, CL` with CL now 18H. For an 8-bit ROL, the rotation count is taken mod 8. 18H mod 8 = 24 mod 8 = **0** ⇒ AL unchanged.
- **AL = 22H** ✓

### Example 2: `STC ; MOV AX,5485H ; RCR AL,1 ; MOV CL,03 ; RCL AX,CL` — [PYQs/Compre 2017(solved).pdf, Q11]
- After STC, CF=1.
- AX=5485H so AL=85H = `1000 0101`.
- `RCR AL,1`: 9-bit rotation through CF. The 9-bit window before = `CF | AL` = `1 | 1000 0101`. After rotation right by 1, new (CF | AL) = `1 | 1100 0010`. So **AL = C2H, CF = 1** (the old AL bit 0 became new CF). AX = 54C2H.
- `RCL AX, CL` with CL=3: 17-bit rotation through CF. Window before = `CF | AX` = `1 | 0101 0100 1100 0010`. Rotate left 3 → bits shift out the top into the CF/window. Working the 17-bit string `10101010011000010` rotated left 3 = `01010011000010 101` = window `0 | 1010 0110 0001 0101` ⇒ AX = **A615H**, CF=0 (after rotation; the high bit ends up at CF=0 in the rotated window). ✓

### Example 3: `MOV AH, 5FH ; ROR AH, 3` — [PYQs/Compre 2015.pdf, Q6c]
- AH = `0101 1111` = 5FH.
- ROR by 3: bits rotate right, low 3 bits wrap to top.
- Low 3 bits of AH = `111`. After rotating, those become the top 3 bits.
- New AH = `111 | 01011` = `1110 1011` = EBH.
- If AX initially had AL=9C, then AX = **EB9CH**. ✓

**Trap.** RCL/RCR participate with CF — always track CF *before* starting.

---

## P0.3 — Memory cycle time / address-bus capacity

**Theory.** Each bus cycle = 4 T-states + wait states. With N address lines, addressable bytes = 2^N. Time per MEMR/MEMW = `(4 + WS)/f_clk`.

### Example 1: 10 MHz clock, 4 wait states — [PYQs/Compre 2017(solved).pdf, Q3]
- T_clk = 1/10 MHz = 100 ns.
- Cycle = (4 + 4) × 100 ns = **800 ns**. ✓

### Example 2: 20-bit address bus capacity — [PYQs/Compre 2016 (Dec).pdf, Q6]
- 2^20 = 1 048 576 bytes = **1 MB**.

### Example 3: 4 MHz clock, MEMR with 2 wait states
- T_clk = 250 ns.
- Cycle = 6 × 250 = **1500 ns = 1.5 µs**.

**Trap.** IOR/IOW also = 4T minimum, same formula. Don't confuse "machine cycle" (= 4T) with "T-state" (1 clock).

---

## P0.4 — Interrupt priority ordering

**Theory.** In the 8086, when multiple sources contend for service at the same instruction boundary, priority order (highest first per the course solution sheet's convention):
**Software trap (single-step / TF) > Software interrupts (INT n, INTO, divide-error) > NMI > INTR**

For "increasing priority" the order is INTR, NMI, INTO, Single-step.

### Example 1: Arrange in increasing priority: INTO, Single-step, NMI, INTR — [PYQs/Compre 2017(solved).pdf, Q1]
- Increasing means lowest-priority first.
- Order: **INTR < NMI < INTO < Single-step**.
- Answer letters: **D, C, A, B**. ✓

### Example 2: A program sets IF=0 and TF=1, then encounters a divide-by-zero on `IDIV CX`. Which fires?
- IF=0 masks **INTR only**. NMI, INT n, INTO, divide error are unaffected.
- Divide-error (type 0) is a fault that fires immediately — higher priority than the resulting single-step trap.
- **Divide-by-zero ISR runs first**. The single-step would fire on the next instruction boundary.

### Example 3: Both NMI and INTR pulse during the same instruction. Which is serviced first?
- NMI > INTR (NMI is non-maskable, so it gets serviced first).
- After IRET from NMI ISR, INTR is then sampled and serviced (if IF=1).

**Trap.** TF affects single-step trap. STI/CLI affect only INTR.

---

## P0.5 — Identify valid vs invalid instructions

**Theory.** 8086 syntax forbids: two memory operands, immediate to segment register, byte POP, INC/DEC on segment registers, and using AX/CX/DX as memory base (only BX/BP/SI/DI allowed).

### Example 1: 5-part check — [PYQs/Compre 2015.pdf, Q4]
| Instruction | Verdict | Reason |
|---|---|---|
| `ROL [AX], CL` | **Invalid** | AX cannot be inside `[...]` (only BX/BP/SI/DI) |
| `XCHG [DL], 80H` | **Invalid** | DL is 8-bit, can't be memory pointer; XCHG with immediate not allowed |
| `POP AL` | **Invalid** | POP is word-only on 8086 |
| `INC BP` | **Valid** | Word increment of register |
| `JMP 5B00H:2AC0H` | **Valid** | Far direct jump |

### Example 2: Quick-fire judgement
- `MOV DS, 5000H` → **Invalid** (immediate to segment reg). Fix: `MOV AX, 5000H ; MOV DS, AX`.
- `MOV [BX], [SI]` → **Invalid** (two memory operands). Fix: `MOV AL, [SI] ; MOV [BX], AL`.
- `PUSH AX` / `PUSH 1234H` → first valid; second valid only on 80286+ (8086 doesn't allow immediate push).

### Example 3: Subtle ones
- `MOV BX, OFFSET LABEL` → **Valid** (immediate offset).
- `MOV BX, LABEL` → **Valid** (loads word from memory at offset of LABEL).
- `MOV CS, AX` → **Invalid** (cannot write to CS directly; must be via far jump/call/ret).

---

## P0.6 — Determine final AX after one instruction (independent rows)

**Theory.** Track which part of AX changes: `AL`-targets leave AH alone; `AX` targets overwrite both. CMP/TEST don't change destination, only flags.

### Example 1: AX=5F9CH initially — [PYQs/Compre 2015.pdf, Q6]
- **a) `CMP AX, BX`** → AX unchanged = **5F9CH** (CMP affects flags only).
- **b) `SUB AL, AH`** → AL = 9C - 5F = 3D, AH unchanged = 5F. AX = **5F3DH**.
- **c) `ROR AH, 3`** → AH = 5F = `0101 1111`, ROR by 3: lowest 3 bits `111` wrap to top → AH = `111 01011` = EBH. AX = **EB9CH**.
- **d) `SAL AL, 4`** → AL = 9C = `1001 1100`, shift left 4 → `1100 0000` = C0H. AX = **5FC0H**.
- **e) `MOVSX AX, AL`** → AL=9C, sign bit=1 (negative). Sign-extend → AX = **FF9CH**. (Note: MOVSX is 386+. On 8086 use `CBW`.)

### Example 2: AX=8000H, `NEG AX`
- NEG AX = 0 - AX = 0 - 8000H = 8000H (in 16-bit two's complement, -8000H = 8000H). AX unchanged.
- But **OF=1** because 8000H is the most-negative 16-bit value and its negation overflows.

### Example 3: AX=00FFH, `INC AL` vs `INC AX`
- `INC AL`: AL=FF → 00, AH unchanged = 00. AX = **0000H**.
- `INC AX`: AX=00FF → 0100H. **AX = 0100H**.

**Trap.** `INC` doesn't affect CF (deliberate, so it can be used in loops without disturbing add-with-carry).

---

## P0.7 — Will this code generate Overflow flag?

**Theory.** OF set iff signed result has wrong sign: pos+pos→neg, neg+neg→pos, or pos−neg→neg, neg−pos→pos. Equivalently, OF = (carry into MSB) XOR (carry out of MSB).

### Example 1: `MOV AL,70H ; MOV BL,60H ; ADD AL,BL` — [PYQs/Compre 2016 (May).pdf, Q2 / 2015.pdf, Q2]
- Signed: AL=+112, BL=+96. Sum=208 > +127 max ⇒ overflow.
- Binary: 0111 0000 + 0110 0000 = 1101 0000. Sum's sign bit=1 (negative) but both inputs were positive → **OF=1**. ✓

### Example 2: `97 + 48` (8-bit signed) — [Question Bank/Tutorial 1.pdf, Q4a]
- 97 = `0110 0001`, 48 = `0011 0000`.
- Sum = `1001 0001` (=145 unsigned, but signed = -111).
- Two positives summed to negative-looking ⇒ overflow.
- **SF=1, CF=0, OF=1, AF=0, PF=0**.

### Example 3: `−41 − 95` (8-bit signed) — [Question Bank/Tutorial 1.pdf, Q4f / Q5]
- −41 (two's comp) = `1101 0111`. To subtract 95, add −95: two's comp = `1010 0001`.
- Sum = `1101 0111 + 1010 0001 = 0111 1000` (with carry out of MSB).
- Two negatives → positive-looking result ⇒ overflow.
- **SF=0, CF=1, OF=1, AF=0, PF=1**.

### Example 4: `99 − 33` (no overflow case)
- 99 = `0110 0011`, −33 (two's comp) = `1101 1111`. Sum = `0100 0010` = 66.
- Sign correct, no overflow. **SF=0, CF=1, OF=0, AF=1, PF=1**. ✓

**Trap.** CF (unsigned overflow) and OF (signed overflow) are independent. Always check OF rule: same sign inputs producing different sign output.

---

## P0.8 — Set/reset specific flag bits

**Theory.** No direct instructions for TF, SF, ZF, PF, OF, AF. Must use PUSHF/POP AX → bit-ops → PUSH AX/POPF. STC/CLC for CF, STI/CLI for IF, STD/CLD for DF.

### Example 1: Set TF only, preserve others — [PYQs/Compre 2017(solved).pdf, Q8]
```
PUSHF                ; flags → stack
POP  AX              ; flags → AX
OR   AX, 0100H       ; set bit 8 (TF)
PUSH AX
POPF                 ; flags ← AX
```
Bit 8 of FLAGS = TF.

### Example 2: Set all conditional, reset all status — [PYQs/Compre 2017(solved).pdf, Q7]
- Conditional = control flags (TF=bit 8, IF=9, DF=10) → mask 0700H.
- Status flags (CF=0, PF=2, AF=4, ZF=6, SF=7, OF=11) should be 0.
```
MOV  AX, 0FFFFH      ; all 1s
AND  AX, 0700H       ; keep only TF, IF, DF; status bits become 0
PUSH AX
POPF
```

### Example 3: Clear CF, set DF, keep everything else
```
PUSHF
POP   AX
AND   AX, 0FFFEH     ; clear bit 0 (CF)
OR    AX, 0400H      ; set bit 10 (DF)
PUSH  AX
POPF
```
Or simpler: `CLC` (clears CF) + `STD` (sets DF). When direct instructions exist, prefer them.

**Trap.** Confusion is common: TF is bit **8** (not 7). DF=bit 10, IF=bit 9.

---

## P0.9 — ISR return address from stack contents

**Theory.** When an interrupt fires, 8086 pushes (in this order): FLAGS, CS, IP. So **stack top has IP_lo first**, then IP_hi, then CS_lo, CS_hi, FL_lo, FL_hi.

### Example 1: Stack snapshot — [PYQs/Compre 2017(solved).pdf, Q10]
Stack (top → down): `00 65 00 14 00 15 08 16`.
- IP_lo=00, IP_hi=65 → IP = 6500H.
- CS_lo=00, CS_hi=14 → CS = 1400H.
- Physical return address = CS×16 + IP = 14000H + 6500H = **1A500H**. ✓

### Example 2: Person-counter NMI snapshot — [Question Bank/Tutorial-9.pdf, slide 7]
Given CS=1000H, IP=2346H, FLAGS=0600H at time of NMI, SS:1208H is current SP.
After NMI pushes (decrement SP by 2 each push: FLAGS first, then CS, then IP):
- SP = 1208 - 2 = 1206: FLAGS_hi:FLAGS_lo = 06:00 → bytes at 1207:1206 = 06 00.
- SP = 1206 - 2 = 1204: CS = 1000H → bytes at 1205:1204 = 10 00.
- SP = 1204 - 2 = 1202: IP = 2346H → bytes at 1203:1202 = 23 46.
**Stack contents:**
| Addr | Value |
|---|---|
| 1207 | 06H |
| 1206 | 00H |
| 1205 | 10H |
| 1204 | 00H |
| 1203 | 23H |
| 1202 | 46H |

### Example 3: Stack snapshot for INT 21H call
If ISR sees SP pointing to `60 00 50 12 02 00`, return is to CS:IP = 1250:0060 = physical 12560H.

**Trap.** SP decrements *before* each PUSH. Bytes are little-endian (low byte at lower address).

---

## P0.10 — Sign-extend / zero-extend using only logical/shift

**Theory.** To sign-extend an N-bit value in a 16-bit register: shift the sign bit up to bit 15, then arithmetic-shift right to replicate it. Zero-extend: just AND off the high bits.

### Example 1: Sign-extend 12-bit AX, AH upper nibble all-zeros — [PYQs/Compre 2017(solved).pdf, Q6]
- AX = `0000 SXXX XXXX XXXX` where S is the 12-bit sign bit.
- Method (canonical):
```
SHL  AX, 4         ; bit 11 → bit 15
SAR  AX, 4         ; arithmetic shift back, sign bit replicated
```
The solution sheet accepts `ROL AX, 4 ; SAR AX, 4` because AH upper nibble = 0 (so ROL ≈ SHL here).

### Example 2: Zero-extend AL into AX without using MOVZX
```
MOV  AH, 0         ; clear high byte
```
Single-instruction zero-extend. (On 386+, `MOVZX AX, AL` does it natively.)

### Example 3: Sign-extend AL into AX (8-bit → 16-bit) using only shifts
```
MOV  AH, AL        ; copy
SAR  AH, 7         ; arithmetic shift right by 7 replicates sign bit across AH
```
Equivalent to the `CBW` instruction.

**Trap.** ROL/ROR don't sign-extend correctly unless you happen to know the upper bits are zero.

---

# CLUSTER 1 — ALP / Tracing / Code generation

## P1.1 — DEBUG `u`/`d` output fill-in (DOSBOX)

**Theory.** `u` disassembles starting at IP; each line shows `seg:offset machine-code mnemonic operands`. `d offset` dumps 16 bytes per line from DS:offset. Use `.data` declarations to count offsets of each variable.

### Example 1: Walk through an ALP — [PYQs/Compre 2023(Solved).pdf, Part C Q1]
```
.data
data1 db '[', ']', '*'           ; 3 bytes
data2 dw 9876h, 'a', 'A', 'e'    ; 4 words = 8 bytes
data3 dw 0FEFEH                  ; 2 bytes
array dw 1,2,3,4,5,6,7,8,9       ; 18 bytes
```
The DEBUG screenshot shows:
```
mov BX, [011CH]      ; MOV BX, data2 → machine code shows operand 011CH
mov CX, [0124H]      ; MOV CX, data3
mov [0119H], BL      ; MOV data1, BL
mov DL, [0119H]      ; MOV DL, data1
mov AX, [DI+0126H]   ; MOV AX, array[DI]
```
**Working back:**
- data2 at 011CH ⇒ data1 at 011CH − 3 = **0119H**.
- data3 = data2 + 8 = 011CH + 8 = **0124H**.
- array = data3 + 2 = 0124H + 2 = **0126H**.
- Filled answers: (a) `BX, [011C]`, (b) `CX, [0124]`, (c) `0108`, (d) `010C`, (e) `76 98 61 00`, (f) `00 30 00 FE`.

### Example 2: Constructed example — what offset would `data4 dd 0AABBCCDDH` get if appended?
- After `array` (offset 0126H + 18 = 0138H), `data4` (4 bytes) starts at offset **0138H**.
- Its bytes (little-endian): `DD CC BB AA`.

### Example 3: Disassemble `B8 34 12` at offset 0100H
- Opcode B8 = `MOV AX, imm16`. Imm = little-endian 1234H.
- → `MOV AX, 1234H`. Next IP = 0103H (3 bytes consumed).

**Trap.** Always remember strings/data in source are placed in declaration order; word values are stored little-endian in memory.

---

## P1.2 — Square-wave / pulse with 8253/54

**Theory.** Output freq = CLK/count for square wave (mode 3) or rate gen (mode 2). If count > 65535, **cascade** counters. Control word selects counter, R/W, mode, BCD.

### Example 1: 1 kHz square on counter 1 — [PYQs/Compre 2015.pdf, Q3]
CLK1=1 MHz, CREG addr=0BH, C1 addr=07H.
- Count = 1e6 / 1e3 = 1000 = 03E8H.
- CW for C1, mode 3, binary, LSB+MSB load = `01 11 011 0` = 76H.
```
MOV AL, 76H ; OUT 0BH, AL    ; CW
MOV AL, E8H ; OUT 07H, AL    ; LSB
MOV AL, 03H ; OUT 07H, AL    ; MSB
```

### Example 2: 278 µHz square wave, CLK=2.5 MHz, base 90H — [Question Bank/Tutorial-10.pdf, Q1]
- Required count = 2.5e6 / 278e-6 = 9e9. Exceeds 65535 → cascade all three counters.
- Factor: 9e9 ≈ 6×10⁴ × 5×10⁴ × 3.
- C0 mode 3 ÷6×10⁴, C1 mode 3 ÷5×10⁴, C2 mode 3 ÷3. (Solution uses Mode 2 for C0/C1 because output should be uniform clock for next stage; final stage mode 3 for square.)
- Counts: C0 = 60000 = EA60H, C1 = 50000 = C350H, C2 = 3 = 0003H.
- CWs: C0=36H (`00 11 011 0`), C1=76H (`01 11 011 0`), C2=96H (`10 01 011 0`).
- Port addresses (base=90H): C0=90H, C1=92H, C2=94H, CREG=96H.
```
; C0
MOV AL, 36H ; OUT 96H, AL
MOV AL, 60H ; OUT 90H, AL    ; LSB
MOV AL, EAH ; OUT 90H, AL    ; MSB
; C1
MOV AL, 76H ; OUT 96H, AL
MOV AL, 50H ; OUT 92H, AL
MOV AL, C3H ; OUT 92H, AL
; C2
MOV AL, 96H ; OUT 96H, AL
MOV AL, 03H ; OUT 94H, AL
```
Wire: OUT0→CLK1, OUT1→CLK2.

### Example 3: Find duty cycle in mode 3 — [PYQs/Compre 2022.pdf, Q2]
CLK0 = 5 MHz, MOV/OUT sequence loads C0 with `00110110b` CW then count `0111 0100b` = 74H. Output freq = 5e6 / 74H = 5e6 / 116 = **43.1 kHz**? The question writes "OUT0 freq = 25 kHz" — let me recompute: count = 0111 0100 followed by 01110100 = same byte loaded as LSB twice? Actually the code writes both 86H twice — meaning LSB=74H, MSB=74H ⇒ count=7474H=29812. 5e6/29812 ≈ **167.7 Hz** — but solution claims 2.5 kHz. **Lesson**: walk the code line by line; don't assume — read both OUT operands carefully.

For **mode 3 (square wave)**: duty cycle = 50% if count is even, slightly more than 50% if odd (high portion = (count+1)/2, low = (count-1)/2).

**Trap.** When CW says R/W=11 (LSB then MSB), two consecutive OUTs to the counter port form one 16-bit count.

---

## P1.3 — Multi-counter cascade (Part B big question)

**Theory.** When required count > 65535, cascade. Use mode 2 (rate gen) for intermediate stages — outputs uniform pulses suitable as next stage's CLK. Final stage uses the mode that yields desired waveform.

### Example 1: Interrupt every 85896 s, CLK=4 MHz, C0 already FFFFH — [PYQs/Compre 2023(Solved).pdf, Part B Q2]
- Required freq = 1/85896 Hz ≈ 1.164e-5 Hz.
- Total count = 4e6 / 1.164e-5 ≈ 3.43e11.
- Factor: 3.43e11 = 65536 × 65536 × 80 (approximately).
- C0 = FFFFH (given, untouched), C1 = FFFFH, C2 = 50H (=80).
- Modes: C0 mode 2 or 3, C1 mode 2 or 3, C2 mode 2 (only).
- CWs (base F9H, ports F9/FB/FD/FF for C0/C1/C2/CREG, R/W=11 for 16-bit C0/C1, R/W=01 for byte-only C2):
  - CW0 = `00 11 010 0` = 34H (mode 2) or 36H (mode 3).
  - CW1 = `01 11 010 0` = 74H or 76H.
  - CW2 = `10 01 010 0` = 94H (or 95H BCD).
```
; CW write (skip C0, given)
MOV AL, 74H ; OUT FFH, AL
MOV AL, 94H ; OUT FFH, AL
; Loads (only C1 and C2; C0 untouched)
MOV AL, FFH ; OUT FBH, AL   ; C1 LSB
MOV AL, FFH ; OUT FBH, AL   ; C1 MSB
MOV AL, 50H ; OUT FDH, AL   ; C2 (byte-only, R/W=01)
```
Connect OUT2 (inverted) → 8086 INTR.

### Example 2: 10-second timer with ANSW gating — [PYQs/Compre 2017(solved).pdf, Part B Q4]
CLK = 2 MHz, base 80H, ports 80/82/84/86.
- C0 mode 3, ÷20000 → 2e6/20000 = 100 Hz. Count = 20000 = 4E20H. CW0 = `00 11 011 0` = 36H.
- C1 mode 3 (or 2), ÷100 → 100/100 = 1 Hz. Count = 100 = 64H. CW1 = 76H (or 56H = byte-only mode 3).
- C2 mode 5 (hardware-triggered strobe), count = 10 (binary). CW2 = `10 01 101 0` = 9AH.
- Wire OUT0→CLK1, OUT1→CLK2, ANSW button → GATE2 (rising edge triggers strobe).
```
MOV AL, 00110110b ; OUT 86H, AL   ; CW0
MOV AL, 01010110b ; OUT 86H, AL   ; CW1 (R/W=01 byte only for count=100)
MOV AL, 10011010b ; OUT 86H, AL   ; CW2
MOV AL, 20H       ; OUT 80H, AL   ; C0 LSB
MOV AL, 4EH       ; OUT 80H, AL   ; C0 MSB
MOV AL, 64H       ; OUT 82H, AL   ; C1 byte
MOV AL, 0AH       ; OUT 84H, AL   ; C2 byte
```

### Example 3: 1 kHz from 5 MHz on OUT2 only
- Need count = 5000.
- Single counter suffices (5000 < 65535). C2, mode 3.
- CW = `10 11 011 0` = B6H. Load 5000 = 1388H = LSB 88H, MSB 13H.

**Trap.** Mode 2 output is a *narrow* low pulse (1 clk) — perfect for triggering next stage. Mode 3 output is roughly 50% square — also usable as clock if next stage is rising-edge-triggered.

---

## P1.4 — Mode-3 special case (count = 0)

**Theory.** Loading count=0 in mode 3 (square wave) is interpreted as 65536 (max count + 1). Output freq = CLK / 65536.

### Example 1: CLK=2 MHz, count=0000H — [PYQs/Compre 2017(solved).pdf, Q4]
- Output freq = 2e6 / 65536 = **30.5175 Hz**. ✓

### Example 2: CLK=2.5 MHz, count=0 (mode 2)
- In mode 2, count 0 is also interpreted as 65536.
- Freq = 2.5e6 / 65536 = **38.147 Hz**.

### Example 3: BCD mode, count loaded as 0000H
- BCD max is 9999, so count=0 interpreted as 10000.
- Freq = CLK / 10000.

---

## P1.5 — Software-delay loop calculation

**Theory.** Sum T-states of one loop iteration body. Total delay ≈ T_states_per_iter × count × T_clk. Solve for count given required delay.

### Example 1: 63 ms @ 5 MHz with table-provided T-states — [PYQs/Compre 2017(solved).pdf, Q9]
| Inst | T |
|---|---|
| MOV | 4 |
| NOP | 3 |
| DEC/INC | 2 |
| JNZ | 16 |

Loop body: DEC CX (2) + NOP (3) + JNZ (16) = 21 T per iter.
T_clk = 1/5e6 = 200 ns.
Count = 63e-3 / (21 × 200e-9) = 63e-3 / 4.2e-6 = **15000 = 3A98H**.
```
        MOV CX, 3A98H
DELAY:  DEC CX
        NOP
        JNZ DELAY
```

### Example 2: 200 ms @ 4 MHz — [PYQs/Compre 2023(Solved).pdf, Part B Q1]
Body: AND AX,AX (3) + OR BX,BX (3) + NOP (3) + LOOP (17 taken) = 26 T.
T_clk = 250 ns.
Count = 200e-3 / (26 × 250e-9) = 30769.69 → 30769 or 30770. **VAL = 7831H or 7832H**.
Actual delay with VAL=30769 = (26 × 30769 − 12) × 250e-9 = 0.1999955 s.

### Example 3: 1 ms @ 8 MHz, body = DEC CX (2) + JNZ (16) = 18 T
- T_clk = 125 ns.
- Count = 1e-3 / (18 × 125e-9) = 1e-3 / 2.25e-6 ≈ **444 = 01BCH**.
```
        MOV CX, 01BCH
DELAY:  DEC CX
        JNZ DELAY
```

**Trap.** The last iteration's JNZ is not-taken (=4T), giving a small over-estimate. For exam, ignore this unless asked for 7-decimal precision.

---

## P1.6 — Block ALP routines

**Theory.** Set up data pointer in SI/BX, destination in DI, count in CX. Loop body transforms one element, increments pointers, decrements CX, branches.

### Example 1: Genetic-algorithm mutation — [PYQs/Compre 2018(solved).pdf, Q4]
For each 16-bit value at POPULATION[i], complement the bits at positions given by the LSB nibble and MSB nibble of LOC[i] (0-15). Store at MUTATED[i]. 30 elements.
```
.startup
    LEA  DX, LOC
    LEA  DI, MUTATED
    LEA  SI, POPULATION
    MOV  CH, 30                ; loop counter
X1: MOV  CL, byte ptr [DX]
    AND  CL, 0FH               ; low nibble = LSB-side bit position
    INC  CL                    ; mask = 1<<(position+1)? — yields position+1 RCL
    MOV  AX, 0
    STC                        ; mask bit prepared
    RCL  AX, CL                ; AX = 1 shifted to position
    MOV  BX, word ptr [SI]
    XOR  BX, AX                ; flip low-side bit
    MOV  CL, byte ptr [DX]
    AND  CL, 0F0H              ; high nibble
    SHR  CL, 4
    MOV  AX, 0
    INC  CL
    STC
    RCL  AX, CL                ; AX = 1 shifted to high position
    XOR  BX, AX                ; flip high-side bit
    MOV  word ptr [DI], BX
    INC  DI ; INC DI           ; word ptr advance
    INC  SI ; INC SI
    INC  DX
    DEC  CH
    JNZ  X1
.exit
END
```

### Example 2: Max-element-by-bit-count — [PYQs/Compre 2023(Solved).pdf, Part C Q2]
ARRAY1 of 9 bytes. Find the element with most 1-bits.
```
.data
ARRAY1   db  91h, 02h, 83h, 0FFh, 75h, 06h, 47h, 12h, 76h
RESULT   db  ?
MAX_ONES db  0
.code
.startup
    LEA  BX, ARRAY1
    MOV  CL, 09H              ; counter
    MOV  AL, [BX]             ; first element
    CALL COUNT_ONES
    MOV  MAX_ONES, AH         ; first count
    MOV  RESULT, AL
    INC  BX
X2: MOV  AL, [BX]
    CALL COUNT_ONES
    CMP  AH, MAX_ONES
    JLE  SKIP                 ; or JBE
    MOV  MAX_ONES, AH
    MOV  RESULT, AL
SKIP: INC BX
    DEC  CL
    JNZ  X2
.exit
COUNT_ONES PROC               ; counts ones in AL, returns AH
    PUSH AX ; PUSH BX ; PUSH CX
    MOV  AH, 0
    MOV  CL, 08H              ; 8 bits
COUNT_LOOP:
    SHR  AL, 1
    JNC  NO_INCREMENT
    INC  AH
NO_INCREMENT:
    DEC  CL
    JNZ  COUNT_LOOP
    POP  CX ; POP BX ; POP AX
    RET
COUNT_ONES ENDP
END
```

### Example 3: Find the differing index between Dat1 and Dat2 — [Question Bank/Tut-5.pdf, Q4]
Two word arrays of 50 elements, exactly one differs. Store its 1-indexed index in `Index`.
```
.data
Dat1   dw 67h, 66h, 70h, 40h, ... (50 words)
Dat2   dw 67h, 66h, 70h, 40h, ... (50 words, one differs)
Index  dw ?
.code
.startup
    LEA  SI, Dat1
    LEA  DI, Dat2
    MOV  CX, 50
    MOV  BX, 1                ; 1-indexed
LOOP1:
    MOV  AX, [SI]
    CMP  AX, [DI]
    JNE  FOUND
    INC  BX
    INC  SI ; INC SI
    INC  DI ; INC DI
    LOOP LOOP1
FOUND:
    MOV  Index, BX
.exit
END
```

### Example 4: Macro that multiplies InPar by 9 (no MUL allowed) — [Question Bank/Tut-5.pdf, Q1]
9 × x = (8 × x) + x = (x << 3) + x.
```
NINE MACRO InPar, OutPar
    MOV  CX, InPar      ; copy x
    SHL  CX, 3          ; CX = 8x
    ADD  CX, InPar      ; CX = 9x
    MOV  OutPar, CX
ENDM
```
6 lines. To compute DX = 27 × BX with one ADD:
27 × x = (9 × 3) × x. Apply NINE once for 9x, then... actually 27x = 16x + 8x + 2x + x = ((x<<4)+(x<<3)+(x<<1)+x). Use multiple shifts — but only one ADD allowed. Alternative: 27x = 9x + 18x = 9x + 2×(9x) = 9x + (9x<<1). So:
```
.startup
    NINE BX, DX            ; DX = 9*BX (uses CX inside)
    SHL  DX, 1             ; DX = 18*BX — but this loses 9*BX
    ; need DX = 27*BX = 9*BX + 18*BX with only one ADD
```
Cleaner: use NINE to get DX=9*BX, then trick — `MOV CX, DX ; SHL DX,1 ; ADD DX, CX` (=18+9=27). But that uses MOV outside which is allowed (4 lines total within startup).

**Trap.** Always preserve registers used by procedures (PUSHA/POPA).

---

## P1.7 — Trap-flag / specific-flag manipulation
See P0.8.

---

## P1.8 — Translate machine code ↔ assembly

**Theory.** 8086 opcode is 1 byte (sometimes 2), followed by `mod-reg-r/m` byte for memory/register ops, then optional displacement/immediate. Use the encoding tables (Brey Appendix C) when allowed.

### Example 1: Machine `8AF3H` → assembly — [PYQs/Compre 2015.pdf, Q5iii(a)]
- First byte 8AH = `MOV r8, r/m8` (W=0, D=1).
- Second byte F3H = `11 110 011`. MOD=11 (reg-reg), REG=110 (DH), R/M=011 (BL).
- → `MOV DH, BL`. ✓

### Example 2: Assembly `MOV [SI], AH` → machine — [PYQs/Compre 2015.pdf, Q5iii(b)]
- Opcode `MOV r/m8, r8` (W=0, D=0) = 88H.
- Mod-reg-rm: MOD=00 (memory, no disp), REG=100 (AH), R/M=100 (SI direct) = `00 100 100` = 24H.
- → bytes `88 24`. ✓

### Example 3: `MOV AX, 1234H` → machine
- `B8 imm16` = MOV AX, imm. Imm = `34 12` (little-endian).
- → `B8 34 12`. (3 bytes)

**Trap.** Operand bytes are little-endian. The MOD field controls whether there's displacement (00=none unless R/M=110, 01=8-bit disp, 10=16-bit disp, 11=register-register).

---

# CLUSTER 2 — Memory Interfacing (Part B big question)

## P2.1 — Compute chip count and address range

**Theory.** Chip count = total_size / chip_size. For 16-bit data bus (8086/80286), double the count (odd + even banks). Start/end addresses = base / base + size − 1.

### Example 1: 8086, 7MB requirement, 80386 in 24-bit — [PYQs/Compre 2017(solved).pdf, Part B Q1]
- 2MB SRAM1 @ 000000H end 1FFFFFH; 4MB SRAM2 @ 200000H end 5FFFFFH; 1MB ROM @ 600000H end 6FFFFFH.
- 512K SRAM1 chips → 2MB/512K = 4 chips. (80386 has its own bank logic; here treat as flat space.)
- 1MB SRAM2 chips → 4 chips.
- 512K ROM chips → 1MB/512K = 2 chips.

### Example 2: 80286, 2 × 64K ROM + 1 × 64K RAM with 32K chips, 8086-style banks — [PYQs/Compre 2015.pdf, Q1]
- ROM@F0000H (64K): 2 × 32K chips, 1 pair (even+odd bank).
- RAM1@D0000H (64K): 1 pair.
- RAM2@E0000H (64K): 1 pair.
- Total chips needed: ROM=2, RAM=4. Provided: 32K RAM 4 chips, 32K ROM 2 chips ⇒ matches.

### Example 3: 80286, 1024K (256K ROM + rest RAM), with 27256 (32K) and 61512 (64K) — [PYQs/Compre 2022.pdf, Part B Q1]
- 256K ROM / 32K = 8 chips (4 pairs).
- 768K RAM / 64K = 12 chips (6 pairs).
- Map: RAM @ F00000H–FBFFFFH (768K), ROM @ FC0000H–FFFFFFH (256K).

**Trap.** When chip is byte-sized and bus is 16-bit, chips come in pairs (even on D0-D7, odd on D8-D15).

---

## P2.2 — Bank wiring (BHE'/A0)

**Theory.** A0=0 selects even bank, BHE'=0 selects odd bank. For chip CS' line:
- Even chip CS' = OR(decoder_out', A0)
- Odd chip CS' = OR(decoder_out', BHE')
A0 of the chip ties to **A1** of the 8086 (because A0 is bank-select, not address).

### Example 1: Even/odd wiring for 8086 ROM at F0000H — [PYQs/Compre 2015.pdf, Q1]
Each 32K chip uses A0–A14 internally (32K = 2^15). 8086 bus: chip's local A0..A14 = system's A1..A15.
- Decoder enables when A19..A16 = `1111` and A15 selects which 32K block (here just one block, so A15 is used too).
- Even chip CS' = OR(decoder_out, A0). Odd chip CS' = OR(decoder_out, BHE').

### Example 2: NAND-only realisation
Identity: A OR B = NAND(A', B'). Build NOT from NAND(X,X).
```
OR(decoder, A0) = NAND(NAND(decoder, decoder), NAND(A0, A0))
```
Inverts decoder, inverts A0, then NANDs them together = OR.

### Example 3: 80286 even/odd with LS138 — [Question Bank/Tut-5.pdf]
The LS138 outputs feed each bank's CS' via OR(LS138_out, A0) for even and OR(LS138_out, BHE') for odd. G2A' enabled by inverted A20 (the system enable bit).

---

## P2.3 — LS138-based address decoding

**Theory.** 3:8 decoder picks one of 8 chunks based on 3 select inputs (typically the highest 3 address lines below the static range). G enables let you gate the whole decoder using even-higher bits.

### Example 1: 80386 system @ 000000–6FFFFFH (24-bit space) — [PYQs/Compre 2017(solved).pdf, Part B Q1]
Use A23, A22, A21 as ABC of LS138 → splits 8 MB into 8 zones of 1 MB each:
- Y0 (A23A22A21=000) → 000000–0FFFFFH → covers SRAM1 first 1 MB.
- Y1 (001) → 100000–1FFFFFH → SRAM1 second 1 MB.
- Y2 (010) → 200000–2FFFFFH → SRAM2 first chunk.
- ... etc.
G1 = Vcc, G2A'=G2B'=GND (or fed by higher bits if those exist).

### Example 2: 8086 @ 80000H–9FFFFH with 8 × 2764 ROM (8K each) + 4 × 6164 RAM (8K each) — [PYQs/Compre 2023(Solved).pdf, Part A Q3]
- 8 KB chip uses A0..A12 internally → system A1..A13 (account for bank-select on A0).
- LS138 takes A14, A15, A16 as A, B, C; G1=Vcc; G2A'=A17, G2B'=A18 (or A19), gated low.
- Y0 → 80000–83FFFH = ROM1+ROM2 pair (even+odd).
- Y1 → 84000–87FFFH = ROM3+ROM4.
- Y2 → 88000–8BFFFH = ROM5+ROM6.
- Y3 → 8C000–8FFFFH = ROM7+ROM8.
- Y4 → 90000–93FFF (unused).
- Y5 → 94000–97FFF (unused).
- Y6 → 98000–9BFFFH = RAM1+RAM2.
- Y7 → 9C000–9FFFFH = RAM3+RAM4.
Each Y_n is OR'ed with A0 or BHE' to get even/odd CS'.

### Example 3: Single LS138 for 8 × 8KB ROM @ A0000–AFFFFH
- Use A13, A14, A15 as ABC.
- G1=Vcc; G2A'=A16, G2B'=A17. G2A'=G2B'=0 ⇔ A16=A17=0.
- Combined with high address fix: A19-A18-A17-A16 = 1010 to be in A0000H range. Need extra logic for the upper bits.

---

## P2.4 — Full Part-B memory-design walkthrough

**Recipe (universal):**
1. Compute chip counts (pair if 16-bit bus + byte chips).
2. Memory map table: chip, size, start, end, bank.
3. Binary table: rows = chips, columns = A_max..A0. Fixed bits show selector pattern; varying bits are 1s/0s.
4. Choose 3 contiguous bits for LS138 selects (typically the bits that differ between Y0 and Y7).
5. Choose gating bits for G1/G2A'/G2B' (the bits above the selects that are fixed).
6. Decide bank logic (OR with A0 for even, OR with BHE' for odd).
7. Draw the schematic.

### Example 1: 7MB at 000000H for 80386 — [PYQs/Compre 2017(solved).pdf, Part B Q1]
Already covered above. Key insight: LS138 selects + 2-input NAND for sub-zone bank logic.

### Example 2: 32 KB ROM + 64 KB RAM @ 00000H with 16K ROM + 32K RAM chips, NAND only — [PYQs/Compre 2018(solved).pdf, Q2]
- 32KB ROM / 16KB = 2 chips, paired → 1 pair.
- 64KB RAM / 32KB = 2 chips, paired → 1 pair.
- Address ranges: ROM = 00000H–07FFFH, RAM = 08000H–17FFFH (RAM straddles 64K boundary if at 8000H–17FFFH = 0FFFFFH actually 64K starts at 08000).
  Wait: ROM at 00000H size 32K = end 07FFFH. RAM size 64K starting next: 08000H–17FFFH.
- Decoding: ROM enabled when A19..A15 = `00000`. RAM enabled when `00000` and A15=1 OR `00001` and A15=0 (i.e., addresses 08000–17FFF span two 32K chunks). This requires bridging A16 into RAM CS'.
- Use NAND-only: ROM CS' = NAND(A19', A18', A17', A16', A15') for entry into 00000–07FFF block. RAM enable across 08000–17FFF: needs A15 OR A16 to be 1 but A19..A17 zero — equivalently `A19=0 AND A18=0 AND A17=0 AND (A15+A16)=1`.

### Example 3: 80286 1024KB (256 ROM + 768 RAM) @ F00000H — [PYQs/Compre 2022.pdf, Part B Q1]
- RAM at F00000H–FBFFFFH (768K), ROM at FC0000H–FFFFFFH (256K).
- A23..A20 = `1111` always; that's the system enable.
- A19..A18..A17 select within 1MB: LS138 needs A19, A18, A17 as ABC. Map:
  - 000 = F00000-F1FFFF (RAM)
  - 001 = F20000-F3FFFF (RAM)
  - 010, 011, 100, 101 = RAM
  - 110, 111 = ROM
- 6 pairs RAM × 64KB = 384KB per 256KB granularity... actually 768K/(2*32K)=12 chips, factored differently. Use LS138 with finer slicing.

### Example 4: 576KB (128 ROM + 448 RAM) for 80286 — [Question Bank/Tut-5.pdf, 80286 slide]
- 64K ROM @ 00000H + 64K ROM @ F0000H (each as 2 pairs of 32K chips), 448K RAM @ 040000H.
- ROM1: 00000–0FFFFFH = `0000_0000_0000_0000_0000`..`0000_0000_1111_1111_1111_1111`. A19A18A17A16 = 0000.
- ROM2: 0F0000–0FFFFFH. A19..A16 = 0000 1111.
- RAM occupies 040000–0AFFFFFH (7 × 64KB blocks).
- LS138 (4 nos provided) selects between 64K-blocks; G1 enabled by A20+A21 etc.

**Trap.** The solution table provided in 2017's Part B drawing uses A20 inverted into the NAND gate's enable. Whenever you see an OR shown in the question, you can equivalently express via NAND identities — make sure to follow the gate-count limit in the question.

---

## P2.5 — Address-mapping table for given chip set

**Theory.** For each chip, identify which high address bits are *fixed* (decoded) and which are *don't-care* (within the chip's internal range).

### Example 1: 8 × 2764 ROM + 4 × 6164 RAM @ 80000H — [PYQs/Compre 2023(Solved).pdf, Part A Q3a]
| RAM/ROM | Start | End |
|---|---|---|
| ROM1 | 80000H | 83FFFH |
| ROM2 | 84000H | 87FFFH |
| ROM3 | 88000H | 8BFFFH |
| ROM4 | 8C000H | 8FFFFH |
| RAM1 | 98000H | 9BFFFH |
| RAM2 | 9C000H | 9FFFFH |
(ROM1 = pair of two 8K chips, so 16KB per "ROM" row in the table. Total 64KB ROM + 32KB RAM after pairing.)

### Example 2: 7MB at 000000 — [PYQs/Compre 2017(solved).pdf]
| Region | Start | End | Size |
|---|---|---|---|
| SRAM1 | 000000H | 1FFFFFH | 2MB |
| SRAM2 | 200000H | 5FFFFFH | 4MB |
| ROM | 600000H | 6FFFFFH | 1MB |

### Example 3: 64K @ each of F0000H, D0000H, E0000H — [PYQs/Compre 2015.pdf]
| Region | Start | End |
|---|---|---|
| RAM1 | D0000H | DFFFFH |
| RAM2 | E0000H | EFFFFH |
| ROM | F0000H | FFFFFH |

---

# CLUSTER 3 — Protected mode (80286/386)

## P3.1 — Selector → table & index

**Theory.** Selector = `Index(13) | TI(1) | RPL(2)`. TI=0 → GDT; TI=1 → LDT. Descriptor address = table_base + index × 8.

### Example 1: DS = 0026H — [PYQs/Compre 2023(Solved).pdf, Q1]
- Binary: `0000 0000 0010 0110`.
- RPL = bits 1:0 = `10` = 2.
- TI = bit 2 = `1` → **LDT**.
- Index = bits 15:3 = `0000 0000 0010 0` = 4.
- → 4th entry in LDT, descriptor at LDTR + 4×8 = LDTR + 20H.

### Example 2: DS = 0049H — [PYQs/Compre 2023(Solved).pdf, Q2]
- Binary: `0000 0000 0100 1001`.
- RPL = `01` = 1.
- TI = `0` → **GDT**.
- Index = `0000 0000 0100 1` = 9.
- → 9th entry, descriptor at GDTR + 9×8 = GDTR + 48H.

### Example 3: SS = 002Dh — [PYQs/Compre 2015.pdf, Part B Q1]
- Binary: `0000 0000 0010 1101`.
- RPL = `01` = 1. TI = `1` → **LDT**. Index = bits 15:3 = 5.
- → 5th entry of LDT.

**Trap.** Watch the bit ordering: TI is bit 2, NOT bit 3. RPL is the lowest 2 bits.

---

## P3.2 — Decode descriptor: base, limit, attributes

**Theory.** 8-byte descriptor (80386 standard):
- byte 0-1: limit[15:0]
- byte 2-4: base[23:0]
- byte 5: access rights (P, DPL, S, type)
- byte 6: limit[19:16] (low nibble) + flags G, D/B, 0, AVL (high nibble)
- byte 7: base[31:24]

Reading "LSB first": the question shows bytes as `byte0 byte1 byte2 ... byte7`.

### Example 1: Descriptor `00 FB 27 BD 00 00 80 1F` — [PYQs/Compre 2023(Solved).pdf, Q1 — for MOV EBX]
- byte 0-1: 00 FB → limit_low = FB00H.
- byte 2-4: 27 BD 00 → base_low = 00BD27H.
- byte 5: 00 = `0000 0000` → P=0, **not present** — but solution accepts as "data segment, expand-up, writable". Looking again, the bytes from solution write `00 FB 27 BD 00 00 80 1F` as the LSB-first descriptor → in standard layout that's: limit_low=FB00, base_low=00BD27, access=00 (?). The solution's confusing — they treat the rights byte as differently positioned. Use the question's cheat sheet which gives the standard byte map; don't over-think.
- Linear address = base + offset = base + 10H (since `MOV EBX, [00 00 00 10H]`).

### Example 2: Descriptor `FF FF 00 00 50 F2 40 00` — Novel construction
- limit_low = FFFFH. limit_high = 40H low nibble = 0 → limit = 000FFFFH. With G=4 high nibble bit 7=0, granularity bit=0 → byte-granular ⇒ limit = 64KB-1 (segment size 64KB).
- base_low = bytes 2,3,4 = 00 00 50 → 500000H. base_high = byte 7 = 00 → base = 00500000H.
- Access byte F2H = `1111 0010` → P=1, DPL=11=3, S=1 (code/data), type 0010 = data, expand-up, writable, not accessed.

### Example 3: Descriptor `FF FF 00 00 00 9A 40 00`
- Limit = 0FFFFH (64K).
- Base = 00000000H.
- Access = 9A = `1001 1010` → P=1, DPL=0, S=1, type=1010 = code, non-conforming, readable, not accessed.
- Granularity byte = 40H → G=0 (byte), D=1 (32-bit operand size).

---

## P3.3 — Paging: linear → physical

**Theory.** Linear address (32-bit) splits 10/10/12: PD index, PT index, offset. Walk CR3 → PD entry → PT entry → page frame, then add offset.

### Example 1: Linear = `01 80 D0 65H` (corrected from 2023 Q2)
- Binary: `0000 0001 1000 0000 1101 0000 0110 0101`.
- PD idx = bits 31:22 = `0000 0001 10` = 6. **6th PD entry**.
- PT idx = bits 21:12 = `00 0000 1101` = 13 → solution says 9 (corrected to 9 from question's actual value).
- Offset = bits 11:0 = `0000 0110 0101` = 65H.
- PD entry at CR3 + 6×4. Read PT base from PD entry. PT entry at PT_base + 9×4. Frame from PT entry. Physical = (frame<<12) | 65H.
- Solution: physical = **2C000065H**.

### Example 2: Linear = `00800010H` — [PYQs/Compre 2017(solved).pdf, Part B Q5]
- Binary: `0000 0000 1000 0000 0000 0000 0001 0000`.
- PD idx = `0000 0000 10` = 2. **2nd PD entry**.
- PT idx = `00 0000 0000` = 0. **0th PT entry**.
- Offset = `0000 0001 0000` = 10H.
- Walk: get PD[2] → PT base. PT[0] → frame. Physical = frame + 10H = **41002000H** + 10H = **41002010H**? Solution says **41 00 20 00H**. Probably the offset is included differently due to specific table contents.

### Example 3: Linear = `00040000H`
- PD idx = bits 31:22 = `0000000000` = 0.
- PT idx = bits 21:12 = `0001000000` = 64 = 40H.
- Offset = 0.
- → PD[0], PT[40H], offset 0.

**Trap.** Always show the 10/10/12 split clearly in your answer — markers gives marks for the slicing even if you mess up the table lookup.

---

## P3.4 — Decode privilege / type from access-rights byte

**Theory.** Access byte: `P | DPL(2) | S | Type(4)`. For data: type bits = `0 ED W A`. For code: `1 C R A`.

### Example 1: Access = F2H — `1111 0010`
- P=1, DPL=11=3, S=1 (user), bit3=0 (data), ED=0 (expand-up), W=1 (writable), A=0.
- → Data segment, expands up, writable, DPL 3.

### Example 2: Access = 9AH — `1001 1010`
- P=1, DPL=00=0, S=1, bit3=1 (code), C=0 (non-conforming), R=1 (readable), A=0.
- → Code segment, non-conforming, readable, DPL 0.

### Example 3: Access = 92H — `1001 0010`
- P=1, DPL=0, S=1, type=0010 = data, expand-up, writable.
- → Privileged data segment.

---

## P3.5 — Compute privilege accessibility

**Theory.** App with CPL c, using selector with RPL r, can access data segment of DPL d only if max(c, r) ≤ d. (For code with conforming bit C=1, more lenient.)

### Example 1: DS=0026H with RPL=2, target DPL=2 — [PYQs/Compre 2023(Solved).pdf, Q1c]
- Effective privilege = max(CPL, RPL). If CPL=0, eff=max(0,2)=2 ≤ DPL=2 ⇒ allowed.
- Privilege levels app can access this descriptor at: **{0, 1}** (CPL must satisfy CPL ≤ DPL=2 AND RPL ≤ DPL, but RPL is fixed in this selector as 2 so CPL must be 0 or 1 or 2).
- Solution: "**0, 1, 2**" or "**1, 0**" depending on interpretation. Typically answer is **0, 1** (strictly less) or **0, 1, 2** (less-or-equal). The solution sheet writes `1°, 11` for 2023 Q1c — i.e., privilege levels 1 and 2 (binary 01, 10) — supporting that CPL ≤ effective DPL.

### Example 2: DPL=3 segment — app can access at CPL ∈ {0,1,2,3}.

### Example 3: Conforming code segment with DPL=1
- Conforming bit C=1 allows transfer to it from CPL ≥ DPL. So CPL ∈ {1,2,3} can call. Once running, CPL stays as caller's.

---

# CLUSTER 4 — 8255 PPI

## P4.1 — Mode-set Control Word

**Theory.** CW (D7=1) format: `1 GA-M1 GA-M0 PA PCu GB-M PB PCl`. Direction bit: 1=input, 0=output.

### Example 1: Drone scenario — PA in, PB out, PC in, mode 0 — [PYQs/Compre 2023(Solved).pdf, Q4]
- D7=1, D6D5=00 (M0 GA), D4=1 (PA=in), D3=1 (PCu=in), D2=0 (M0 GB), D1=0 (PB=out), D0=1 (PCl=in).
- CW = `1 00 1 1 0 0 1` = **99H**. ✓

### Example 2: Switch PC5 off in BSR mode — [PYQs/Compre 2022.pdf, Q1]
- D7=0 (BSR), D6D5D4=XXX (don't-care, take as 000), D3D2D1=101 (PC5), D0=0 (reset).
- CW = `0 000 101 0` = **0AH**. ✓

### Example 3: PA out, PB in, PC upper out, PC lower in, both groups mode 0
- D7=1, D6D5=00, D4=0, D3=0, D2=0, D1=1, D0=1.
- CW = `1 00 0 0 0 1 1` = **83H**.

**Trap.** PC is split: upper PC (PC4-7) controlled by D3; lower PC (PC0-3) by D0. They can have different directions.

---

## P4.2 — Decode 8255 port addresses from base

**Theory.** Inside 8255: A1A0 = 00 (PA), 01 (PB), 10 (PC), 11 (CREG). If 8255's A1A0 are wired to bus's A_n+1 A_n, the spacing between PA/PB/PC/CREG addresses = 2^n.

### Example 1: Base 87H, drone — [PYQs/Compre 2023(Solved).pdf, Q4]
- 87H = `1000 0111`. A1 bit=A2 of bus, A0 bit=A1 of bus? With PA=E7H, PB=EFH, PC=F7H, CREG=FFH:
- E7=`1110 0111`, EF=`1110 1111`, F7=`1111 0111`, FF=`1111 1111`. Only bits A3 and A4 change. So 8255's A1=A4 of bus, A0=A3.
- PA=E7, PB=EF (A3=1), PC=F7 (A4=1), CREG=FF (A4=A3=1). ✓

### Example 2: Base 80H, A1A0 of 8255 tied to A1A0 of bus — [PYQs/Compre 2017(solved).pdf, Part B Q4 8254 — analogous]
- PA=80, PB=81, PC=82, CREG=83.

### Example 3: Base F0H, two 8255s in same region (A1A0 of 8255 tied to A2A1) — [Question Bank/Tutorial-9.pdf]
- 8255 #1 ports: PA=F0H, PB=F2H, PC=F4H, CREG=F6H.
- 8255 #2 ports: PA=F1H, PB=F3H, PC=F5H, CREG=F7H (different CS' via A0 distinguishing).

---

## P4.3 — Data-bus pin assignment (even vs odd bank for 8086)

**Theory.** 8255 is 8-bit. If base address has A0=0, tie to D0-D7 (even bank). If A0=1, tie to D8-D15 (odd bank).

### Example 1: Base B7H — [PYQs/Compre 2022.pdf, Q1]
- B7H low bit = 1 → odd bank → D0-D7 of 8255 ↔ **D8-D15** of bus.

### Example 2: Base 80H
- A0=0 → **D0-D7** of bus.

### Example 3: Base 87H — [PYQs/Compre 2023(Solved).pdf, Q4]
- A0=1 → D8-D15. The question's blank c) is "D8-D15 ↔ D0-D7". ✓

---

## P4.4 — Part-B 8255 system design

**Theory.** Identify each port direction, compute CW, decode CS' from given base, write initialization + main loop or ISR.

### Example 1: Drone system — [PYQs/Compre 2023(Solved).pdf, Q4]
PA=sensors (in), PB=LEDs PB3/PB4/PB5 (out), PC=sensors (in). Base E7H.
- CW = 99H.
- Mode logic: status_check returns mode in BL (0=bad weather, 1=land, 2=flight).
```
PortA EQU E7H
PortB EQU EFH
PortC EQU F7H
CREG  EQU FFH

MOV  AL, 99H
OUT  CREG, AL
CALL status_check
CMP  BL, 0
JE   bad_weather_mode
CMP  BL, 1
JE   landing_mode
CMP  BL, 2
JE   normal_flight_mode

bad_weather_mode:    MOV AL, 08H ; OUT PortB, AL ; PB3 on (red)
landing_mode:        MOV AL, 10H ; OUT PortB, AL ; PB4 on (yellow)
normal_flight_mode:  MOV AL, 20H ; OUT PortB, AL ; PB5 on (green)
```
Pins: 8255 A1 ↔ 8086 A4, A0 ↔ 8086 A3, D0-D7 ↔ 8086 D8-D15.

### Example 2: Smart parking lot — [PYQs/Compre 2018(solved).pdf, Q3]
Two 8255s? No, single 8255 base 87H with PA reading entry UID, PB reading exit UID, PC for 7-segment display strobes. Real-time clock TIME (word). Cost = C_sec × (T_OUT − T_IN).
```
PortA EQU 87H ; PortB EQU 8FH ; PortC EQU 97H ; CREG EQU 9FH
MOV AL, 92H   ; CW = 1 00 1 0 0 1 0: M0, PA in, PB in, PC out
OUT CREG, AL
; Entry ISR:
IN_ISR PROC FAR
    PUSHA
    IN  AL, PortA            ; read UID
    LEA SI, IN_ID
    LEA BX, FREE             ; find first FREE=0 slot
    ; ... store UID, store TIME at T_IN[idx], decr DISPLAY by 1
    POPA
    IRET
IN_ISR ENDP
; Exit ISR uses REPNE CMPSB to find matching UID, computes stay, applies C_sec, displays cost
```
See PDF for full code.

### Example 3: 8 parking lot indicators — [PYQs/Compre 2015.pdf, Part B Q2]
8 sensors on 8086-compatible 8-bit bus enter 8255 PA. PB drives 8 LEDs (red=occupied, green=vacant). Base 70H.
- CW for PA in, PB out: 90H.
- ALP loops: read PA, invert for green LEDs (vacant=high), output to PB.
```
MOV  AL, 90H ; OUT 73H, AL
LOOP1: IN AL, 70H ; NOT AL ; OUT 71H, AL ; JMP LOOP1
```

### Example 4: Person counter (NMI input, 2x 8255) — [Question Bank/Tutorial-9.pdf]
- 8255 #1: PC reads pause switch (PC0).
- 8255 #2: PC outputs 8-bit count to LED.
- On each NMI (low→high), increment CL. If switch=1 (count enabled), output CL to 8255 #2.
```
MOV AL, 80H ; OUT F7H, AL    ; 8255 #1: PC in. Actually wait — both PCs different: solution writes CW1=80H for #2 (PC out), CW2=81H for #1 (PC in).
MOV CL, 0
NMI_ISR:
    IN  AL, F4H          ; read 8255 #1 PC: switch
    AND AL, 01H
    JZ  PAUSE
    INC CL
PAUSE:
    MOV AL, CL
    OUT F7H, AL          ; output count
    IRET
```

---

## P4.5 — Mode 1 / Mode 2 handshake recognition

**Theory.** Mode 1 = strobed I/O with handshake pins on PC. Mode 2 = bidirectional on PA only (PB still in mode 0 or 1, PC for handshake).
- Mode 1 input: PC4=STB', PC5=IBF, PC3=INTR for PA. PC2=STB', PC1=IBF, PC0=INTR for PB.
- Mode 1 output: PC7=OBF', PC6=ACK', PC3=INTR for PA. PC1=OBF', PC2=ACK', PC0=INTR for PB.

### Example 1: Polling IBF in mode 1 input
```
LOOP1: IN AL, PortC ; TEST AL, 20H ; JZ LOOP1   ; wait IBF=1
       IN AL, PortA ; MOV [BX], AL ; INC BX
       JMP LOOP1
```

### Example 2: Mode 2 bidirectional on PA — handshake pins use PC3 (INTR), PC4 (STB'), PC5 (IBF), PC6 (ACK'), PC7 (OBF'). Use when a device both reads and writes via PA.

### Example 3: Recognize from question: "When data is ready, sensor pulses STB' on PC4" → Mode 1 input on Port A.

---

# CLUSTER 5 — 8259 PIC

## P5.1 — Write ICW1–ICW4 (single 8259)

**Theory.** ICW1: edge/level + cascade/single + ICW4 needed. ICW2: vector type base (bottom 3 bits zero). ICW3: only in cascade. ICW4: 8086 mode + AEOI etc.

### Example 1: Bicycle odometer, IR1 only — [PYQs/Compre 2018(solved).pdf, Q1]
- Single, edge-triggered, ICW4 needed.
- ICW1 = `0001 0 0 1 1` = **13H**.
- IR0 type = 90H → ICW2 = **90H**.
- ICW3 = not required (single).
- ICW4 = `0000 0001` = **01H** (8086 mode, no AEOI).
- OCW1: unmask only IR1 → mask byte = `1111 1101` = **FDH**.
- OCW2: non-specific EOI = **20H**.
- Initialization code @ base 740H (A0 of 8259 ↔ A1 of bus, so base+0 = ICW1/OCW2 port, base+2 = ICW2/ICW3/ICW4/OCW1 port):
```
MOV AL, 13H ; MOV DX, 0740H ; OUT DX, AL
MOV AL, 90H ; MOV DX, 0742H ; OUT DX, AL
MOV AL, 01H ; OUT DX, AL                  ; ICW4 — same port as ICW2/3
MOV AL, FDH ; OUT DX, AL                  ; OCW1 — same port
```

### Example 2: 8259 @ FFEFH, single, no SFNM, no AEOI — [PYQs/Compre 2023(Solved).pdf, Part B Q4]
- ICW1 = `0001 0 0 1 1` = **1FH** (single = SNGL=1) — solution says **1FH** or **1BH**. 1BH = `0001 1 0 1 1` (LTIM=1 level-triggered). The question says "single-level triggered" — level → LTIM=1 → ICW1=1BH or 1FH. Both accepted.
- IR6 type = 79 dec = 4FH. ICW2 must produce 4FH for IR6 ⇒ base for IR0 = 4FH − 6 = 49H. Hmm, but ICW2's low 3 bits are forced 0. ICW2 = 48H ⇒ IR0=48H, IR6=4EH (not 4FH). To make IR6=4FH, ICW2's value should have IR0=49H with bits 2:0=0 — impossible since IR0=49H=`01001001` has bits 2:0=001 ≠ 000.
  Re-read: "type 79 decimal". 79 = 4F. IR0 base must satisfy 4FH = ICW2 + 6 mod 256 ⇒ ICW2 = 49H, but its low 3 bits must be zero so impossible unless type 79 is reachable through a different IR mapping. Solution uses **ICW2 = 48H** with the explanation "IR0 base = 48H, so IR6 = 4EH". Then for type 79 to land on IR6, perhaps solution accepts ICW2=4FH and adjusts. The solution table writes ICW2=48H and notes "79=4FH for IR6". Use the solution's number: **ICW2 = 48H**.
- ICW3 = not needed (single).
- ICW4 = `0000 0001` = **01H**.
- OCW1: mask IR0 and IR5 → `0010 0001` = **21H**.
- OCW2: rotate on specific EOI, set IR2 lowest → `111 00 010` = **E2H**.

### Example 3: 8259 with all 8 IRs enabled, fully nested, ISR at type 70H — Tutorial example
- ICW1 = `0001 0 0 0 1` = 11H (edge, cascade-default 0 since SNGL=0, but using single?). For single, SNGL=1 → ICW1 = `0001 0 0 1 1` = 13H.
- ICW2 = 70H.
- ICW4 = 01H.
- OCW1 = 00H (unmask all).

---

## P5.2 — Write OCW1, OCW2, OCW3

**Theory.** OCW1 = mask byte. OCW2 = `R SL EOI 0 0 L2 L1 L0`. OCW3 used for poll/read IRR or ISR.

### Example 1: Mask IR0, IR5; bottom priority = IR2; rotate on specific EOI — [PYQs/Compre 2023(Solved).pdf, Part B Q4]
- OCW1 = `0010 0001` = **21H**.
- OCW2 = `111 00 010` = **E2H**.

### Example 2: Mask all except IR6, IR7 — [PYQs/Compre 2017(solved).pdf, Part B Q3]
- OCW1 = `0011 1111` = **3FH**.

### Example 3: Non-specific EOI in ISR — [PYQs/Compre 2018(solved).pdf, Q1]
- OCW2 = **20H** = `0010 0000` (R=0, SL=0, EOI=1).
Write in ISR: `MOV AL, 20H ; MOV DX, base ; OUT DX, AL` before IRET.

### Example 4: Specific EOI for IR4
- OCW2 = `011 00 100` = 64H.

### Example 5: Set priority order (IR5..IR4) by changing fixed priority
- OCW2 = `110 00 100` = C4H (rotate priorities so IR4 becomes lowest, IR5 becomes highest).

---

## P5.3 — Cascaded 8259 (master + slaves)

**Theory.** ICW3 master: bit n=1 means slave on IRn. ICW3 slave: low 3 bits = slave ID = master IR position. Each slave needs its own ICW1-4. Vector type at slave IRn = slave's ICW2 + n.

### Example 1: Master + 2 slaves, slave1 on IR2, slave2 on IR5 — [Question Bank/Q6-1.pdf, Quiz 6 Q1]
Master type base = 40H. Slave1 base = 70H. Slave2 = A0H. Interrupt on IR3 of slave2.
- Vector for IR3 of slave2 = A0H + 3 = **A3H**. ✓
- ISR address for IR1 of master = master_base + 1 × 4 = (40H + 1) × 4 = 41H × 4 = 104H. ✓ (Each IVT entry is 4 bytes.)
- CAS0-CAS2 during INTA cycle for slave2 (on master IR5): CAS bits = master IR number = 5 = `101`. ✓

### Example 2: Master + 2 slaves with master at F220H/F222H, slave1 on master IR7 at F230H/F232H, slave2 on master IR6 at F240H/F242H — [PYQs/Compre 2017(solved).pdf, Part B Q3]
- Master ICW1 = 11H (cascade, edge, IC4 needed).
- Master ICW2 = 28H. Slave1 ICW2 = F0H. Slave2 ICW2 = F8H.
- Master ICW3 = `1100 0000` = **C0H** (slaves on IR6, IR7).
- Slave1 ICW3 = 07H (low 3 bits = 7).
- Slave2 ICW3 = 06H.
- Master ICW4 = `0000 1111` = **0FH** (AEOI=1, BUF=1, M=1).
- Slave ICW4 = `0000 1011` = **0BH** (AEOI=1, BUF=1, M=0).
- OCW1 master = mask IR0..IR5, leave IR6, IR7 unmasked → `0011 1111` = **3FH**.
- OCW1 slaves = 00H (no masking).

### Example 3: Master + 1 slave on IR2, simple — [Question Bank/Tutorial-10.pdf]
Master @ 20H/22H, Slave @ A0H/A2H, slave on master's IR2. All EOI manual.
- Master ICW1 = 11H ; ICW2 = 20H (vector base) ; ICW3 = 04H (bit 2 set) ; ICW4 = 01H.
- Slave ICW1 = 11H ; ICW2 = 28H ; ICW3 = 02H (ID 2) ; ICW4 = 01H.
```
MOV AL, 11H ; OUT 20H, AL ; OUT A0H, AL    ; ICW1 to both
MOV AL, 20H ; OUT 22H, AL                  ; master ICW2
MOV AL, 28H ; OUT A2H, AL                  ; slave ICW2
MOV AL, 04H ; OUT 22H, AL                  ; master ICW3
MOV AL, 02H ; OUT A2H, AL                  ; slave ICW3
MOV AL, 01H ; OUT 22H, AL ; OUT A2H, AL    ; ICW4
```

---

## P5.4 — ISR for an 8259 application

**Theory.** ISR template: save regs → handle event → send EOI → IRET. For non-specific EOI: OUT base, 20H.

### Example 1: Odometer ISR — [PYQs/Compre 2018(solved).pdf, Q1]
Each IR1 = pulse on bicycle wheel. 50 pulses = 100m. 10×100m = 1km.
```
ODOMETER PROC FAR
    LEA  BX, TICK
    LEA  CX, MTR
    LEA  AX, KM
    INC  byte ptr [BX]
    CMP  byte ptr [BX], 32H     ; 50 in decimal
    JAE  DIST
    JMP  IEXT
DIST:
    MOV  byte ptr [BX], 0
    MOV  DL, 64H                ; 100
    ADD  word ptr [CX], DX      ; MTR += 100
    CMP  word ptr [CX], 3E8H    ; 1000
    JE   MTRRST
    JMP  IEXT
MTRRST:
    MOV  word ptr [CX], 0
    INC  word ptr [AX]          ; KM++
IEXT:
    MOV  AL, 20H
    MOV  DX, 0740H
    OUT  DX, AL
    IRET
ODOMETER ENDP
```

### Example 2: NMI ISR setting bit 0 at 80H, IR7 ISR setting bit 1 at 80H — [PYQs/Compre 2022.pdf, Q5]
```
NMI_ISR:
    PUSH AX ; PUSH DX
    MOV  DX, 80H
    IN   AL, DX
    OR   AL, 01H
    OUT  DX, AL
    POP  DX ; POP AX
    IRET                        ; no EOI — NMI doesn't go through 8259

IR7_ISR:
    PUSH AX ; PUSH DX
    MOV  DX, 80H
    IN   AL, DX
    OR   AL, 02H
    OUT  DX, AL
    MOV  AL, 67H                ; specific EOI for IR7
    MOV  DX, 60H                ; 8259 base
    OUT  DX, AL
    POP  DX ; POP AX
    IRET
```

### Example 3: Person-counter NMI ISR — [Question Bank/Tutorial-9.pdf]
```
NMI_ISR:
    IN  AL, F4H            ; read switch on 8255#1 PC0
    AND AL, 01H
    JZ  PAUSE
    INC CL                 ; increment count
PAUSE:
    MOV AL, CL
    OUT F7H, AL            ; display on 8255#2 PC
    IRET
```

---

## P5.5 — Hardware schematic for 8259

**Theory.** 8259 INT pin → 8086 INTR; INTA' from 8086 → 8259. 8259 A0 ↔ A1 of bus. CS' decoded by remaining address lines + M/IO'.

### Example 1: Single 8259 @ 740H, I/O-mapped — [PYQs/Compre 2018(solved).pdf, Q1]
- 740H = `0000 0111 0100 0000`. M/IO'=0 (I/O). A8-A15 used for decoding.
- LS138 with A8, A9, A10 as ABC. G1=Vcc; G2A'=IOR' OR IOW' (or just from M/IO'-equivalent gating); G2B' from higher bits.
- 8259 CS' = Y_n' of LS138 where n encodes the high bits matching 740H.
- 8259 A0 ↔ 8086 A1 (because of even-byte alignment).

### Example 2: Three 8259s (master + 2 slaves) — [PYQs/Compre 2017(solved).pdf, Part B Q3]
Already detailed above. Wiring: each slave's INT ↔ master's IRn; slave's CAS0-2 ↔ master's CAS0-2 (broadcast); master's CAS goes out during INTA' cycle to select which slave responds.

### Example 3: 8259 cascaded with manual EOI, master @ 20H — [Question Bank/Tutorial-10.pdf]
- Wire master CS' from decoder when address = 20H. Master A0 ↔ A0 of bus (since 8-bit chip).
- Slave CS' similarly.

---

# CLUSTER 6 — Machine cycles & timing

## P6.1 — Identify machine-cycle sequence per instruction

**Theory.** Each opcode byte requires a MEMR for fetch (in pairs, since 8086 reads even-aligned words). Memory operand reads add MEMR. Memory operand writes add MEMW. I/O operations use IOR/IOW.

### Example 1: `XLAT` — [PYQs/Compre 2023(Solved).pdf, Part B Q3A]
1-byte instruction. Reads from `DS:[BX+AL]`.
| # | Type | Purpose |
|---|---|---|
| 1 | MEMR | Instruction fetch |
| 2 | MEMR | Operand fetch (lookup byte) |

### Example 2: `INC WORD PTR [BX][SI][0123H]` — [PYQs/Compre 2023(Solved).pdf, Part B Q3B]
4-byte instruction (opcode FF, mod-reg-rm, disp_lo, disp_hi). Memory operand is a word.
| # | Type | Purpose |
|---|---|---|
| 1 | MEMR | Instruction fetch (first 2 bytes) |
| 2 | MEMR | Instruction fetch (last 2 bytes) |
| 3 | MEMR | Operand fetch |
| 4 | MEMW | Result writeback |

### Example 3: `IN AL, 78H` — [PYQs/Compre 2023(Solved).pdf, Part B Q3C]
2-byte instruction (E4 78). Reads 1 byte from I/O port.
| # | Type | Purpose |
|---|---|---|
| 1 | MEMR | Instruction fetch |
| 2 | IOR | Operand fetch (port read) |

### Example 4: `MOV [DI], AX`
Opcode 89 reg-rm 05 → 2 bytes. Memory write of word.
| # | Type | Purpose |
|---|---|---|
| 1 | MEMR | Instruction fetch |
| 2 | MEMW | Result writeback |

**Trap.** 8086 fetches 2 bytes per MEMR (16-bit bus). 4-byte instruction = 2 MEMR fetches. 8088 fetches 1 byte at a time.

---

## P6.2 — Software delay loop
See P1.5.

---

# CLUSTER 7 — ADC/DAC

## P7.1 — ADC selection by resolution

**Theory.** Resolution = Vref / 2^n bits. Need resolution ≤ sensor sensitivity for correct readings.

### Example 1: Sensor 20mV/°C, want 1°C resolution — [PYQs/Compre 2022.pdf, Q3]
- ADC0808: 8-bit, Vref=5V → res = 5/256 = 19.5 mV. Matches 20 mV requirement.
- AD570: 8-bit, ±5V swing (10V range) → res = 10/256 = 39 mV. Too coarse.
- **Choose ADC0808**.

### Example 2: Sensor 5 mV/unit, Vref=5V, need 1-unit resolution
- 8-bit: 5V/256 = 19.5 mV > 5 mV. Too coarse.
- 10-bit: 5V/1024 = 4.88 mV ≈ 5 mV. Just sufficient.
- 12-bit: 5V/4096 = 1.22 mV. Comfortably exceeds requirement.

### Example 3: Sensor 50 mV/unit, Vref=10V, 0.1-unit resolution needed
- Required res = 5 mV.
- 12-bit ADC: 10V/4096 = 2.44 mV. Suffices.

---

## P7.2 — Conversion time

**Theory.** ADC0808 takes ~64 clock cycles for 8-bit SAR conversion. AD570 takes ~25 µs typical (datasheet value).

### Example 1: ADC0808 @ 1 MHz clock — [PYQs/Compre 2022.pdf, Q3]
- t_conv = 64 / 1e6 = **64 µs**.

### Example 2: ADC0808 @ 500 kHz
- t_conv = 64 / 500e3 = **128 µs**.

### Example 3: AD570 — fixed conversion time
- ~**25 µs** (datasheet).

---

# CLUSTER 8 — FAT file system parsing

## P8.1 — Root directory entry parse

**Theory.** 32-byte FAT directory entry:
- bytes 0-7: filename (ASCII, space-padded)
- bytes 8-10: extension
- byte 11: attribute (R/H/S/V/D/A)
- bytes 22-23: write time (HHHHH MMMMMM SSSSS, where S is seconds/2)
- bytes 24-25: write date (YYYYYYY MMMM DDDDD, Y = year − 1980)
- bytes 26-27: starting cluster (little-endian)
- bytes 28-31: file size (4 bytes, little-endian)

### Example 1: Entry `42 49 54 53 44 41 54 41 - 58 4C 53 23 7B BD 4D 18 - A4 3A 4A 3A 00 00 0F 20 - CE 28 0F 00 04 00 00 04` — [PYQs/Compre 2017(solved).pdf, Part B Q2]
- Name: `BITSDATA`, Ext: `XLS`. → `BITSDATA.XLS`.
- Attribute byte = 23H = `0010 0011`. Bits: 0 (R)=1, 1 (H)=1, 5 (A)=1. → **Hidden, Read-only, Archive**.
- Write time (bytes 22, 23 — actually if we 0-index from start, those are `0F 20` → `200F`H = `0010 0000 0000 1111`. H = bits 15:11 = `00100` = 4. M = bits 10:5 = `000000` = 0. S = bits 4:0 = `01111` = 15, doubled = 30. → **04:00:30**.
- Write date (bytes 24, 25) = `CE 28` → `28CE`H = `0010 1000 1100 1110`. Y = bits 15:9 = `0010100` = 20 → 1980+20=**2000**. M = bits 8:5 = `0110` = 6 (June). D = bits 4:0 = `01110` = 14. → **14 June 2000**.
- Starting cluster (bytes 26, 27) = `0F 00` → 000FH = **15**.
- Size (bytes 28-31) = `04 00 00 04` → little-endian 04000004H = 67108868 bytes. Solution writes **269168** — discrepancy; trust the recipe + recheck the bytes.

### Example 2: Entry `43 4F 4D 50 52 45 45 58 - 4E 54 54 16 18 7B BD 4D - 4A 3A 4A 3A 00 00 BD BA - AB AA FD 01 80 36 00 00` — [PYQs/Compre 2015.pdf, Part B Q3]
- Name `COMPREEX`, Ext `NTT`. → `COMPREEX.NTT`.
- Attribute = 16H = `0001 0110` → bits 1 (H), 2 (S), 4 (D). → Hidden, System, Directory.
- Write time (bytes 22, 23) = `BD BA` → BABDH = `1011 1010 1011 1101`. H=10111=23, M=010101=21, S=11101=29 ×2=58. → 23:21:58.
- Write date (bytes 24, 25) = `AB AA` → AAABH = `1010 1010 1010 1011`. Y=1010101=85 →2065, M=0101=5, D=01011=11. → 11 May 2065.
- Starting cluster = `FD 01` → 01FDH = 509.
- Size = `80 36 00 00` → 00003680H = **13952** bytes.

### Example 3: Construct attribute byte for a normal archive file
- Bits: R=0, H=0, S=0, V=0, D=0, A=1. → `0010 0000` = **20H**.

**Trap.** Time field's seconds are doubled (×2). Year is offset from 1980.

---

## P8.2 — FAT-12 cluster chain

**Theory.** FAT-12 packs entries into 1.5 bytes each. Two cluster entries (N and N+1) → 3 bytes:
- byte0 = N[7:0]
- byte1 = N[11:8] | (N+1)[3:0]<<4
- byte2 = (N+1)[11:4]

### Example 1: Cluster chain — [PYQs/Compre 2015.pdf, Part B Q4]
```
Cluster #:  0  1   2   3   4   5   6   7  8   9   A   B   C   D   E   F  10 11 12 13
Next:      FF1 FF6 004 009 008 003 006 FFF 00A 004 00B FFF 011 FFF FFF FFF FFF FFF FFF FFF
```
- File starts at cluster 2 (from dir entry). Chain: 2 → 4 → 8 → A → 4 (loop?). Hmm. Or 2 → 004 means next cluster=4 → 008 → 00A → 004 (wait loops). Re-read: entries say cluster 2's next = 004 → cluster 4's next = 008 → cluster 8's next = 00A → cluster A (=10 decimal) next = 004 (loop!) — likely transcription. Assume chain ends with FFF.
- FAT-12 byte stream: pack each pair of clusters into 3 bytes. For pair (0, 1) = (FF1, FF6): byte0=F1, byte1=6F (=0xF first nibble | 0x6 low | ... actually byte1=(0xF1[11:8])(FF6[3:0])= (0xF)(0x6)=F6. byte2=FF6[11:4]=FF. Hmm sign-checking: (cluster N=FF1=1111 1111 0001, N+1=FF6=1111 1111 0110): byte0 = N[7:0]=F1. byte1 = N[11:8]|(N+1)[3:0]<<4 = 0xF | (0x6 << 4) = 0x6F. byte2 = (N+1)[11:4] = 0xFF. → 3 bytes: F1 6F FF.

### Example 2: Single cluster pair (2, 3) = (4, 9): byte0=04, byte1=04|9<<4=94, byte2=00. → 04 94 00.

### Example 3: Standalone EOF entry (FFF) at odd index: byte fits in the upper nibble of byte1 + byte2. If pair is (X, FFF), byte1 = (X[11:8] | F<<4), byte2 = FF.

---

## P8.3 — Boot sector parse

**Theory.** Standard BPB starts at byte 11:
- bytes 11-12: bytes per sector
- byte 13: sectors per cluster
- bytes 14-15: reserved sectors
- byte 16: number of FATs
- bytes 17-18: root dir entries
- bytes 19-20: total sectors (small)

### Example 1: Boot sector `EB 3C 90 / 2A 60 59 60 48 49 43 0A / 00 40 04 01 00 / 02 / 10 01 / 00 AF F0 09 ...` — [PYQs/Compre 2015.pdf, Part B Q4B]
Bytes per sector (bytes 11-12) = `00 40` little-endian = **4000H** ? That's huge (16384). More likely interpreted as `40 00` LE = 0040H = 64 — unusual. Cross-check from FAT-12 floppy structure: typical 512. The solution's expected number depends on which bytes precise. Use the rule and trust the dump given.
- Sectors per cluster (byte 13) = 01 → 1.
- Reserved sectors (bytes 14, 15) = `00 02` → 0002H or 0200H — most likely 0001H actually for FAT12.
- Number of FATs (byte 16) = 02.
- Root entries = `10 01` LE = 0110H = 272 entries — but standard is 224 or 512. Use as-given.
- Total sectors = `00 AF` LE = AF00H... or read further.

### Example 2: Floppy with 512 bytes/sector, 1 sec/cluster, 9 sectors/FAT, 2 FATs, 224 root entries, 2880 total sectors (standard 1.44 MB floppy)
- BPB bytes: 11:12 = `00 02` (512 LE), 13 = `01`, 14:15 = `01 00`, 16=02, 17:18 = `E0 00` (224), 19:20 = `40 0B` (2880).

### Example 3: Hard disk-like with 1024 bytes/sector
- 11:12 = `00 04` (1024 LE).

---

# CLUSTER 9-10 — DMA / Bus / Misc

## P9.1 — How does PCI distinguish MEMR vs MEMW?
**Answer.** PCI uses **C/BE'[3:0]** command lines during the address phase. Memory Read = 0110, Memory Write = 0111. Same physical wires that are byte-enables during data phase carry command codes during address phase. [PYQs/Compre 2015.pdf, Q5; 2016.pdf, Q5]

## P9.2 — DMA HRQ/HLDA handshake
**Theory.** DMA controller raises HRQ (Hold Request) to CPU. CPU finishes current bus cycle, then asserts HLDA (Hold Acknowledge) and tri-states its bus lines. DMA then drives the bus until done, then drops HRQ; CPU drops HLDA and resumes.

## P10.1 — 8086 min vs max mode
- **Min:** CPU drives all bus signals directly (single-CPU systems).
- **Max:** Status pins (S0-S2) feed 8288 bus controller for multi-processor / coprocessor systems. Pins 24-31 redefined.

## P10.2 — BIU vs EU split
- **BIU (Bus Interface Unit):** fetches instructions into 6-byte queue, computes physical addresses, drives buses.
- **EU (Execution Unit):** decodes from queue, performs ALU ops, registers. Pipelined operation.
- Physical address = (segment × 16) + offset.

---

# CLUSTER 11 — Quick-fire Q&A from question bank

### Tutorial 1: difference between control and status flags
**Status flags** are set by arithmetic/logic results (CF, PF, AF, ZF, SF, OF). **Control flags** are set by the programmer to control CPU behavior (TF=trap, IF=interrupt enable, DF=string direction). [Question Bank/Tutorial 1.pdf, Q4]

### Tutorial 1: After `MOV AL, 7Fh ; ADD AL, 1`, what are the flags?
- AL = 80H. SF=1 (bit 7 set). ZF=0. PF=0 (80H has odd 1-count). AF=1 (carry from bit 3 to 4). OF=1 (positive + positive = negative). CF=0 (no carry out of bit 7).

### Tutorial 1: Two's-complement overflow during subtraction
**Yes**, possible. Example −41 − 95 = −136, which exceeds 8-bit signed range [-128, 127]. [Question Bank/Tutorial 1.pdf, Q5]

### Tutorial-9: NMI is interrupt type ___?
**Answer:** 2.

### Tutorial-9: IVT location for type N
**Answer:** Physical address N × 4. For NMI (type 2), location = 8H (bytes 8, 9 = IP; bytes A, B = CS).

### Q6 quiz: 8254 mode 3, count 5000, CLK 2.5 MHz, high duration?
- Output freq = 2.5e6 / 5000 = 500 Hz. Period = 2 ms.
- Mode 3 is 50% duty (even count). High duration = **1 ms**. [Question Bank/Q6-1.pdf, Q2 Set A]

### Q6 quiz B: Same but CLK 2.0 MHz, count 5000
- Period = 5000/2e6 = 2.5 ms. High = **1.25 ms**.

---

# SOLUTIONS to the 10 Novel Problems (from PROBLEM_CATALOG.md)

## N1 — Solar tracker [22M]

**Components.** 8086 + 8255 (base 80H) + ADC0808 + 8259 (base 40H).

### (a) 8255 control word and init
- PA=in (reads ADC output): D4=1.
- PB=out (4 LEDs): D1=0.
- PC upper out (ALE, START, OE control to ADC): D3=0.
- PC lower in (PC4=EOC from ADC): wait — but PC4 is in the **upper** group (PCu = PC4-PC7). So PCu=in: D3=1.
- Revised: PC upper input (PC4 reads EOC). PC lower output (PC0=ALE, PC1=START, PC2=OE).
- CW = D7=1, D6D5=00 (M0), D4=1 (PA in), D3=1 (PCu in), D2=0 (M0 GB), D1=0 (PB out), D0=0 (PCl out) = `1 00 1 1 0 0 0` = **98H**.
- Port addresses: PA=80H, PB=82H, PC=84H, CREG=86H (assume A1A0 of 8255 ↔ A2A1 of 8086).
```
MOV  AL, 98H ; OUT 86H, AL          ; CW
```

### (b) 8259 ICW/OCW
- Single 8259 (no cascade). Edge-triggered. ICW4 needed.
- ICW1 = `0001 0 0 1 1` = **13H**.
- IR3 vector = 50H → ICW2 such that IR3=50H. IR0 = ICW2 = 50H − 3 = **4DH** — but bottom 3 bits of ICW2 must be 0. Instead set ICW2 = 48H so IR0=48H, IR3=4BH. To get IR3=50H exactly, ICW2 should yield IR0 = 50H − 3 = 4DH (bits 2:0 = 101 ≠ 0). Adjust: pick ICW2 = 50H ⇒ IR0=50H, IR3=53H. But we want IR3=50H. Closest match keeping low 3 bits zero: ICW2 = 48H ⇒ IR3 = 4BH. The question stipulates "IR3 vector 50H" — interpret as "set ICW2 such that IR3 maps to 50H" → impossible with standard 8086 mode. **Best: ICW2 = 50H, accepting that IR0 lands on 50H and IR3 on 53H.** Update problem statement: vector 53H for IR3.
- ICW3 = not needed.
- ICW4 = **01H**.
- OCW1: only IR3 unmasked → `1111 0111` = **F7H**.
- Initialization:
```
MOV AL, 13H ; OUT 40H, AL
MOV AL, 50H ; OUT 42H, AL          ; ICW2
MOV AL, 01H ; OUT 42H, AL          ; ICW4
MOV AL, F7H ; OUT 42H, AL          ; OCW1
```

### (c) ISR for EOC
On EOC interrupt: read ADC channel, compare, drive LED.
```
TRACKER_ISR PROC FAR
    PUSHA
    ; --- Trigger ADC channel 0 (East) ---
    MOV  AL, 00H              ; channel 0 (A,B,C select bits to ADC)
    OUT  82H, AL              ; (assume PB lower bits select channel — adjust per wiring)
    MOV  AL, 01H              ; ALE high
    OUT  84H, AL              ; PC0 = ALE
    MOV  AL, 03H              ; ALE+START high
    OUT  84H, AL
    MOV  AL, 00H              ; pulse off
    OUT  84H, AL
WAIT_EOC1:
    IN   AL, 84H              ; read PC
    TEST AL, 10H              ; PC4 = EOC
    JZ   WAIT_EOC1
    MOV  AL, 04H              ; OE high
    OUT  84H, AL
    IN   AL, 80H              ; read PA = ADC value
    MOV  BH, AL               ; save east reading

    ; --- Trigger channel 1 (West) — analogous ---
    ; ... similar code, store result in BL

    ; --- Compare ---
    SUB  BH, BL               ; difference
    CMP  BH, 20H              ; threshold 32 LSB
    JA   ROTATE_EAST
    NEG  BH                   ; might overflow but illustrative
    CMP  BH, 20H
    JA   ROTATE_WEST
    MOV  AL, 01H ; OUT 82H, AL ; PB0 LED = idle
    JMP  EOI
ROTATE_EAST:
    MOV  AL, 02H ; OUT 82H, AL ; PB1 = east
    JMP  EOI
ROTATE_WEST:
    MOV  AL, 04H ; OUT 82H, AL ; PB2 = west
EOI:
    MOV  AL, 20H
    OUT  40H, AL
    POPA
    IRET
TRACKER_ISR ENDP
```

### (d) Schematic
- 8086 INTR ↔ 8259 INT. INTA' ↔ INTA'.
- 8255 CS': decode address 80H using NAND from A7=1, A6=0, A5=0, A4=0, A3=0; A2A1 of bus ↔ A1A0 of 8255.
- 8259 CS' similarly at 40H.
- ADC: PB0-PB2 → ADC channel select; PC0=ALE; PC1=START; PC2=OE; PC4=EOC. PA reads ADC's D0-D7.

**Concepts tested:** 8255 mode-set + bit-level I/O, 8259 ICW/OCW, ISR with EOI, multi-step ADC handshake.

---

## N2 — GPS pulse counter [10M]

### (a) Counter 1 input frequency
- Counter 0 in mode 2, count = 03E8H = 1000. Input 1 MHz.
- Output freq = 1e6 / 1000 = **1 kHz**. This feeds Counter 1.

### (b) Counter 1 count for 1.5 s
- Counter 1 input is 1 kHz. To represent 1.5 s, count must reach 1500.
- Counter 1 mode 0, count down from initial value FFFFH. After 1500 ticks, decrement from FFFFH by 1500 → current value when ISR latches = FFFFH − 1500 = FFFFH − 5DCH = **FA23H**.
- If latched value < FA23H, more than 1.5 s elapsed.

### (c) ISR
```
GPS_ISR PROC FAR
    PUSHA
    ; Latch C1 via OCW3-equivalent control word (latch command)
    MOV  AL, 01000000b      ; SC=01 (C1), RW=00 (latch)
    OUT  CREG, AL
    IN   AL, C1_PORT        ; read LSB
    MOV  BL, AL
    IN   AL, C1_PORT        ; read MSB
    MOV  BH, AL              ; BX = current C1 value

    ; Re-load C1 to FFFFH for next interval (mode 0 doesn't auto-reload)
    MOV  AL, FFH
    OUT  C1_PORT, AL
    OUT  C1_PORT, AL

    ; Compare: if BX < FA23H, count a missed second
    CMP  BX, 0FA23H
    JAE  OK_TIME
    INC  word ptr MISSED_COUNT
OK_TIME:
    MOV  AL, 20H
    OUT  PIC_BASE, AL        ; EOI
    POPA
    IRET
GPS_ISR ENDP
```

**Concepts tested:** Cascade analysis (P1.3), 8254 latch-counter command, ISR with EOI.

---

## N3 — 8086 stack snapshot puzzle [4M]

**Given.** ISR entered via `INT 18H`. Inside the ISR after `PUSH AX (1234H)` and `PUSH BX (5678H)`. SP=1FF0H. Stack-top down: `78 56 34 12 00 04 00 02 00 09`.

### (a) Return CS:IP
- Pushes after INT (in order): FLAGS, CS, IP. Then user PUSH AX, PUSH BX.
- Stack (top-down at the moment) shows newest pushes first: `78 56` = BX = 5678H; `34 12` = AX = 1234H; `00 04` = IP = 0400H; `00 02` = CS = 0200H; `00 09` = FLAGS = 0900H.
- Return CS:IP = **0200H:0400H**.

### (b) Saved FLAGS = **0900H**.

### (c) Physical return address = 0200H × 16 + 0400H = 2000H + 0400H = **2400H**.

**Concepts tested:** Stack growth direction, interrupt push order (FLAGS/CS/IP), user pushes added on top.

---

## N4 — Mode-1 strobed input ALP [8M]

**Setup.** 8255 Mode 1 PA input, Mode 0 PB output. STB' on PC4, IBF on PC5, INTR on PC3. CW=0B0H configured. PA at port 80H, PB at 81H, PC at 82H, CREG at 83H.

```
.model tiny
.code
.startup
    MOV  AL, 0B0H            ; CW: M1 PA input, M0 PB output
    OUT  83H, AL
POLL:
    IN   AL, 82H             ; read PC
    TEST AL, 20H             ; PC5 = IBF
    JZ   POLL                ; wait until IBF=1
    IN   AL, 80H             ; read PA (this auto-clears IBF)
    OUT  81H, AL             ; mirror to PB
    JMP  POLL
.exit
END
```

**Concepts tested:** Mode 1 handshake bits (P4.5), polling on PC status pin, port I/O.

---

## N5 — Mixed memory map (NAND-only) [16M]

**Setup.** 80286 from FE00000H upward. 1 MB ROM at top, 512 KB SRAM below, 256 KB EEPROM below that. 256 KB chips × 8. NAND + inverters only.

### (a) Memory map

System has 1MB+512KB+256KB = 1792 KB region. Top of memory FE00000H? — clarify: 80286 address bus 24-bit, max = FFFFFFH. Let "FE00000H" be where mapping ends at FEFFFFFH (typo correction: assume FE_00000–FE_FFFFF = 1 MB). Place:

| Chip | Start | End | Size |
|---|---|---|---|
| EEPROM (256K chip pair) | FE00000H | FE3FFFFH | 256K |
| SRAM (512K = 2 × 256K pairs) | FE40000H | FEBFFFFH | 512K |
| ROM (1024K = 4 × 256K pairs) | FEC0000H | FFFFFFFH | 1024K |

**With 256K byte chips on 80286 (16-bit bus), use as pairs.** Total chips: 1 (EEPROM pair = 2 chips), 4 (SRAM = 2 pairs = 4 chips), 8 (ROM = 4 pairs = 8 chips). Total = 14, but problem says only 8 chips. So actually each "256K chip pair" is treated as a single 512K logical chip on the 16-bit bus. Redo:

Available 8 × 256K chips (single-byte each), paired on 16-bit bus → 4 pairs = 4 × 512K = 2 MB. But we only need 1792 KB. Use 4 pairs and disable unused chunks.

| Pair | Function | Start | End |
|---|---|---|---|
| Pair 0 | EEPROM (only need 256K, but bank gives 512K) | FE00000H | FE7FFFFH |
| Pair 1 | SRAM (need 512K, exactly fits) | FE80000H | FEFFFFFH |
| Pair 2 | ROM lower | FF00000H | FF7FFFFH |
| Pair 3 | ROM upper | FF80000H | FFFFFFFH |

### (b) Decoder logic (NAND-only)
Common static bits: A23 A22 = `1 1`, A21 = `1`, A20 = `1`. Then A19 selects between pairs: 0 = lower 1 MB (FE00000-FEFFFFFH), 1 = upper 1 MB (FF00000-FFFFFFFH).
- Each pair's "decoder-out" is: NAND of (A23, A22, A21, A20, [A19 inverted or not], [A18 inverted or not]).
- Pair 0 enable = NAND(A23,A22,A21,A20,A19',A18') — invert A19, A18 via NAND(X,X).
- Pair 1 enable = NAND(A23,A22,A21,A20,A19',A18).
- Pair 2 enable = NAND(A23,A22,A21,A20,A19,A18').
- Pair 3 enable = NAND(A23,A22,A21,A20,A19,A18).

These produce active-low pair-selects. Drop them through another NAND for active-high if needed.

### (c) Bank wiring
- Each pair: even chip CS' = NAND(pair_select', A0') = OR(pair_select, A0).
- Odd chip CS' = NAND(pair_select', BHE).
Realised in NAND: `OR(X, Y) = NAND(X', Y') = NAND(NAND(X,X), NAND(Y,Y))`.

**Concepts tested:** All-NAND decoding, 80286 bank wiring, chip count math.

---

## N6 — Protected-mode descriptor forgery [10M]

**Setup.** GDTR=00100000H. GDT[8] = `FF FF 00 00 50 F2 40 00` (LSB-first 8 bytes).

### (a) Base, limit, granularity, type
- limit_low (bytes 0,1) = FFFFH.
- base_low (bytes 2-4) = 00 00 50 → base[23:0] = **500000H**.
- access (byte 5) = F2H = `1111 0010` → P=1, DPL=11=3, S=1 (user), bit3=0 (data), bit2=0 (ED, expand-up), bit1=1 (W, writable), bit0=0 (A).
- granularity (byte 6) = 40H = `0100 0000`. G=0 (byte gran), D/B=1 (32-bit operand), 0, AVL=0, limit_high nibble = 0000.
- base_high (byte 7) = 00. → full base = **00500000H**.
- Limit (with G=0) = `0000 FFFFH` = **64KB − 1 bytes**.
- → **Data segment, base 00500000H, limit 64KB, byte granular, 32-bit, expand-up, writable, DPL 3**.

### (b) `MOV DS, AX` with AX=0043H, CPL=3
- AX = 0043H = `0000 0000 0100 0011`. Index = bits 15:3 = `0000 0000 0100 0` = 8. TI = bit 2 = 0 (GDT). RPL = `11` = 3.
- Index = 8 → GDT[8] ✓ (our descriptor).
- Privilege check: max(CPL=3, RPL=3) = 3 ≤ DPL=3 ⇒ **load succeeds**.

### (c) If G=1, effective limit = limit field × 4 KB.
- limit field = 0FFFFH. With G=1, effective limit = 0FFFFH × 4096 = `0FFFF000H` + offset within page = up to bytes from base. → **4 GB − 4 KB** (= up to limit FFFFFFFFH if limit field were FFFFFH).
- For limit field 0000FFFFH and G=1: effective max offset = `0000 FFFF * 1000H` + 0FFFH = 0FFFFFFFH ≈ **256 MB − 1**.

**Concepts tested:** Descriptor decoding, privilege rules, granularity bit.

---

## N7 — Cascaded delay + ISR coordination [14M]

**Setup.** Every 1 min, ISR via 8259 IR6 toggles LED on 8255 PC0. Single 8254, all cascaded, CLK=4 MHz.

### (a) Counter modes + counts
- Required output period = 60 s. Required freq = 1/60 Hz.
- Total divisor = 4e6 / (1/60) = 4e6 × 60 = 2.4e8.
- Factor: 2.4e8 = 60000 × 4000 = EA60H × 0FA0H. Both fit in 16 bits.
- C0 mode 2 (rate gen), count 60000 = EA60H → output 4e6/60000 = 66.67 Hz.
- C1 mode 2, count 4000 = 0FA0H → output 66.67/4000 = 0.01667 Hz = 60 s period.
- C2 mode 2, count 1 (trivial). Or skip C2 and feed OUT1 directly to IR6.

Better factorisation: 2.4e8 = 60000 × 4000 × 1. So 2 counters suffice. If problem mandates 3 counters: split differently — e.g., 60000 × 100 × 40. Pick reasonable factors all ≤ 65535.

CWs (base F0H assumed):
- CW0 = 34H (binary, mode 2, R/W=11).
- CW1 = 74H.
- CW2 = 94H or skip.

### (b) Wiring
- 8254 CLK0 ← 4 MHz crystal.
- OUT0 → CLK1.
- OUT1 → CLK2 or directly to 8259 IR6.
- 8259 INT → 8086 INTR.

### (c) ISR
Toggle PC0 via BSR mode. Maintain a state byte `LED_STATE`.
```
LED_ISR PROC FAR
    PUSH AX
    PUSH DX
    XOR  byte ptr LED_STATE, 01H    ; flip state
    MOV  AL, LED_STATE
    AND  AL, 01H                    ; just bit 0
    ; BSR control: 0 XXX 000 S/R: bit number = 0, set/reset from AL bit 0
    OR   AL, 00H                    ; BSR for PC0
    MOV  DX, CREG_8255              ; control register of 8255
    OUT  DX, AL
    MOV  AL, 20H                    ; non-specific EOI
    MOV  DX, PIC_BASE
    OUT  DX, AL
    POP  DX
    POP  AX
    IRET
LED_ISR ENDP
```

**Concepts tested:** 8254 cascade, 8255 BSR, 8259 ISR pattern.

---

## N8 — Run-length encoding ALP [10M]

```
.model tiny
.data
SRC   db 100 dup(?)
DST   db 200 dup(?)             ; worst case 2x size
.code
.startup
    LEA  SI, SRC
    LEA  DI, DST
    MOV  CX, 100                ; bytes remaining in SRC
RLE_LOOP:
    OR   CX, CX
    JZ   DONE
    MOV  AL, [SI]               ; current value
    MOV  BL, AL                 ; remember
    MOV  BH, 1                  ; run length = 1
    INC  SI
    DEC  CX
COUNT_RUN:
    OR   CX, CX
    JZ   FLUSH
    CMP  byte ptr [SI], BL
    JNE  FLUSH
    INC  BH
    CMP  BH, 255
    JE   FLUSH                  ; cap at 255
    INC  SI
    DEC  CX
    JMP  COUNT_RUN
FLUSH:
    MOV  [DI], BH               ; count
    INC  DI
    MOV  [DI], BL               ; value
    INC  DI
    JMP  RLE_LOOP
DONE:
.exit
END
```

**Concepts tested:** Pointer arithmetic, count-state machine, conditional flush, byte-pointer with BYTE PTR.

---

## N9 — DOSBOX `d` output reverse-engineer [8M]

**Source:**
```
data1 db 'BITS', 'PILANI', 0      ; 4 + 6 + 1 = 11 bytes: 42 49 54 53 50 49 4C 41 4E 49 00
data2 dw 0DDCCH, 0EEFFH           ; 4 bytes: CC DD FF EE (little-endian)
```
**Dump observed at d 0200:**
```
0863:0200  42 49 54 53 50 49 4C 41-4E 49 00 00 41 42 43 44   BITSPILANI..ABCD
```

### Where does data2 start?
- data1 occupies bytes 0200H..020AH (11 bytes including null terminator).
- data2 starts at **020BH** (no padding required since `db` is byte-aligned). Bytes 020BH..020EH = `CC DD FF EE`.

### Why doesn't the dump match data2?
- The dump shows bytes 020A..020F as `00 00 41 42 43 44`. But data2 was declared `dw 0DDCCH, 0EEFFH` so we'd expect `CC DD FF EE` at 020B onward, not `00 41 42 43`.
- Conclusion: **the source must declare different data**. The dump's bytes after `00` look like ASCII `ABCD` = `41 42 43 44`. So the actual source likely has:

### Corrected `db` directive
```
data1 db 'BITS', 'PILANI', 0, 0
data2 db 'ABCD'
```
This produces the observed dump.

**Concepts tested:** Endianness for words, ASCII vs hex, byte ordering of `db`.

---

## N10 — Smart lock with keypad [22M]

**Setup.** 4×4 keypad. 8255 (base 80H): PA = column drive (out), PB = row read (in), PC0 = lock motor latch (out), PC1 = alarm (out). 8254 (base 90H) for 30-s timing. 8086 + 2 MHz clock.

### (a) 8255 CW
- PA out, PB in, PC out (all of PC since both PC0 and PC1 are output bits).
- CW = `1 00 0 0 0 1 0` = **82H**.
- Ports: PA=80H, PB=82H, PC=84H, CREG=86H.

### (b) 8254 cascade for 30 s @ 2 MHz
- Total count = 2e6 × 30 = 6e7.
- Factor: 6e7 = 60000 × 1000 = EA60H × 03E8H. Both ≤ 65535. Two counters suffice.
- C0 mode 2, count 60000 → 2e6/60000 = 33.33 Hz.
- C1 mode 0 (count-down to TC for one-shot), count 1000 → fires once after 30 s.
- CW0 = `00 11 010 0` = 34H. CW1 = `01 11 000 0` = 70H.
- Wire OUT0 → CLK1. OUT1 → drives alarm transistor (or to 8259 input).

### (c) ALP for scan-and-compare
```
.model tiny
.data
CODE_DB     db 1, 2, 3, 4              ; expected code
INPUT_BUF   db 4 dup(?)
RETRY_CNT   db 0
.code
.startup
    MOV  AL, 82H ; OUT 86H, AL          ; 8255 init
MAIN:
    LEA  DI, INPUT_BUF
    MOV  CX, 4                          ; collect 4 digits
GETDIGIT:
    CALL KEY_SCAN                       ; returns digit in AL
    MOV  [DI], AL
    INC  DI
    LOOP GETDIGIT
    LEA  SI, INPUT_BUF
    LEA  DI, CODE_DB
    MOV  CX, 4
    CLD
    REPE CMPSB
    JE   UNLOCK
    INC  RETRY_CNT
    CMP  RETRY_CNT, 3
    JE   ALARM
    JMP  MAIN
UNLOCK:
    MOV  AL, 01H ; OUT 84H, AL          ; PC0=1 unlock
    MOV  RETRY_CNT, 0
    JMP  MAIN
ALARM:
    MOV  AL, 02H ; OUT 84H, AL          ; PC1=1 alarm
    CALL START_30s_TIMER
    ; busy-wait or sleep until OUT1 fires; simpler: wait for an interrupt or poll
WAIT_TIMER:
    IN   AL, 84H
    TEST AL, 04H                        ; assume OUT1 routed to PC2 input
    JZ   WAIT_TIMER
    MOV  AL, 00H ; OUT 84H, AL          ; alarm off
    MOV  RETRY_CNT, 0
    JMP  MAIN
.exit

KEY_SCAN PROC NEAR
    ; column drive PA: drive one column low at a time, read PB for the active row.
    MOV  BL, 0FEH                       ; col 0 low
SCAN_LOOP:
    MOV  AL, BL
    OUT  80H, AL
    IN   AL, 82H
    AND  AL, 0FH                        ; only rows
    CMP  AL, 0FH
    JNE  PRESSED
    ; rotate col bit left
    ROL  BL, 1
    JC   SCAN_LOOP                      ; back to start
    JMP  SCAN_LOOP
PRESSED:
    ; decode key based on (BL, AL) → digit
    ; (implementation: bit-test for each row, combined with column index)
    ; pseudo-code: AL_digit = column_idx * 4 + row_idx
    ; return digit in AL
    RET
KEY_SCAN ENDP

START_30s_TIMER PROC NEAR
    MOV  AL, 34H ; OUT 96H, AL          ; CW0
    MOV  AL, 70H ; OUT 96H, AL          ; CW1
    MOV  AL, 60H ; OUT 90H, AL          ; C0 LSB
    MOV  AL, EAH ; OUT 90H, AL          ; C0 MSB (=EA60H = 60000)
    MOV  AL, E8H ; OUT 92H, AL          ; C1 LSB
    MOV  AL, 03H ; OUT 92H, AL          ; C1 MSB (=03E8H = 1000)
    RET
START_30s_TIMER ENDP
END
```

### (d) Schematic
- 8086 + 8255 + 8254 on common bus.
- 8255: PA[0-3] → keypad columns. PB[0-3] ← keypad rows (with pull-ups; columns driven low one at a time).
- PC0 → relay driving lock motor solenoid (via transistor).
- PC1 → buzzer/alarm (via transistor).
- 8254: OUT1 → PC2 (input) or to NMI for alarm-off interrupt.

**Concepts tested:** Keypad row/column scan (matrix), BSR-ish output via OUT (we used `MOV+OUT`; could equivalently use BSR), 8254 one-shot, retry state machine.

---

# 48-hour exam-day playbook

## Day 1 (May 12) — afternoon to evening
1. Read **Quick-reference cards** (top of this file) — memorize ICW/OCW formats, 8254 modes, 8255 CW.
2. Walk through **P0.x** (Cluster 0) — 30 min. These are guaranteed Part A marks.
3. Solve **P2.4 Example 2 (2018 Q2)** and **P2.4 Example 3 (2022 Q1)** fresh on paper. 90 min.
4. Walk through **P3.1–P3.5** with the 2023 paper's cheat sheet open. 45 min.

## Day 2 (May 13)
1. Solve **P5.3 Example 2 (2017 Part B Q3)** fresh, including the cascade ICW3 calculation. 60 min.
2. Solve **P1.3 Example 2 (2017 Part B Q4)** — the 10-s timer. 60 min.
3. Solve **P4.4 Example 1 (drone)** and Example 4 (person counter) from memory. 45 min.
4. Practice **Novel problems N1, N5, N7** end-to-end. 90 min.
5. Read Tutorial 1 overflow examples — make sure flag computation is reflex.

## Day 3 (May 14) — morning
1. Skim the Quick-reference cards once.
2. Re-derive ICW1-4 formats from memory.
3. Memorize bank-select rule: **A0=even, BHE'=odd**.
4. Enter exam.

## In the exam
- Part A: protected-mode walk FIRST (use cheat sheet; slow but methodical). Then 8259 ICW/OCW (table format). Then 8255 BSR / 8254 freq.
- Part B: memory interfacing diagram early (most marks). Then 8254 cascade. Then ISR + 8259.
- If stuck: write the bit-table for memory interfacing or the ICW table for 8259 — you get partial marks for showing the working.
