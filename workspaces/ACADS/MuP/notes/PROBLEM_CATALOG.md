# MuP Compre — Master Problem Catalog
**Exam:** 2026-05-14 09:00 · Part A (CB 60M / 75min) + Part B (OB 40M) + Part C (OB 45M/45M, recent format)
**Source coverage:** Compre 2015, 2016, 2017 (solved), 2018 (solved), 2022, 2023 (solved) + Tutorial 10

Problems are ordered **easy → hard** within each cluster, and clusters are ordered by **frequency on the comprehensive**. If your prep time is short, work top-down.

> Acronyms used: **CW** = control word, **ICW/OCW** = init/operation command word, **PA/PB/PC** = ports of 8255, **CB/OB** = closed-book/open-book Part of compre, **GDT/LDT/PD/PT** = global/local descriptor table, page directory, page table.

---

## CLUSTER 0 — One-shot mental warmups (Part A, < 5M each, very high frequency)

### P0.1 — Identify the addressing mode `[3M, easy]`
**Concept.** 8086 effective-address forms: immediate, register, direct, register-indirect, base, index, base+index, base+disp, index+disp, base+index+disp ("base relative plus index"), string.
**Recipe.**
1. Look at the source operand bracket contents.
2. No brackets, constant → immediate. No brackets, register → register.
3. `[disp]` → direct. `[BX]/[BP]/[SI]/[DI]` → register-indirect.
4. `[BX+disp]` or `[BP+disp]` → based. `[SI+disp]`/`[DI+disp]` → indexed.
5. `[BX+SI]`/`[BX+DI]`/`[BP+SI]`/`[BP+DI]` → base-indexed.
6. `disp[BX+SI]` style → **base relative plus index**.
**Worked.** `MOV AX, 4020[BX+DI]` → **base relative plus index** (2017 Q2).
**Trap.** `[BP]` defaults to SS, not DS — important if the question asks for physical address.

### P0.2 — Trace an ALU/shift snippet, give final reg value `[4M, easy]`
**Concept.** Rotate vs shift: `ROL/ROR` wrap around register; `RCL/RCR` rotate through CF; `SHL/SAL` shift in 0 from right; `SHR` shift in 0 from left; `SAR` shift in sign bit from left.
**Recipe.** Track CF and the register bit-by-bit. Always start by writing the register in binary nibbles.
**Worked (2017 Q5).** `MOV AL,22H ; MOV CL,03H ; ROL CL,CL ; ROL AL,CL`.
- CL=03H=00000011. `ROL CL,CL` rotates CL left by CL (=3 initially) → CL becomes 00011000 = 18H. (CL is now 18H = 24 decimal!)
- But wait — modulo 8 for 8-bit ROL: `ROL` count is modulo `count_size`. Count=3 → CL=00011000=18H. **Hmm but solution says AL=22H** — meaning the next ROL is `ROL AL, CL` with CL=18H. 18H mod 8 = 0 → AL unchanged → AL=22H. ✓
**Worked (2017 Q11).** `STC; MOV AX,5485H; RCR AL,1; MOV CL,03; RCL AX,CL`. Track CF carefully:
- AL=85H=10000101, CF=1. RCR AL,1 → new AL = (CF)(old bits 7..1) = 1·1000010 = 11000010 = C2H. New CF = old bit0 = 1. So AX=54C2H, CF=1.
- RCL AX,CL with CL=03H: a 17-bit rotation (16 reg bits + CF) left by 3.
  AX=0101 0100 1100 0010, CF=1. Rotate left 3 with 17-bit window:
  Bit pattern (CF·AX) = 1·0101 0100 1100 0010. Rotate left 3 → 1010 1001 1000 0101·1 → AX=A685H? But solution says **A615H**.
  Recompute carefully: original 17-bit = `1 0101 0100 1100 0010`. Shift left 3, wrapping into CF: positions 14,15,16 (from top) go around. Result reading 17 bits = `1 0100 1100 0010 101` → AX low bits should be `1010` from MSB-side carry-around. Solution AX=A615H = `1010 0110 0001 0101`. Either way, **memorize the rule**: with N-bit count, do N micro-rotations: (CF, AX) ← (AX[15], AX[14..0], CF).
**Trap.** ROL/ROR/RCL/RCR with `count > register size`: the count is taken **modulo 32** on 386+ but **modulo 8/16** on 8086 only for `count = CL`. For immediate `count > 31`, behavior is undefined.

### P0.3 — Address-bus capacity, memory-cycle time `[3M each, easy]`
**Concept.** N address lines → 2^N bytes addressable. One MEMR/MEMW = 4 T-states + wait states; T = 1/clock.
**Recipe.** Time = (4 + WS) × (1/f). For 10 MHz, T=100ns. 4+4WS = 8T = 800ns.
**Worked (2017 Q3).** 10 MHz, 4WS → **800 ns**.
**Trap.** Don't confuse Tstate with Tcycle. One bus cycle = 4 T-states minimum. IOR/IOW are also 4T.

### P0.4 — Interrupt priority ordering `[2M, easy]`
**Concept.** 8086 internal priority (high→low): **divide-by-zero / INT n / INTO** > **NMI** > **INTR** > **single-step (TF)**.
**Recipe.** Memorize the order. The catch: software interrupts (INT, INTO, divide-error) take highest service priority on the same instruction boundary. NMI > INTR (NMI cannot be masked). Single-step has the lowest precedence so it doesn't block real work.
**Worked (2017 Q1).** Increasing priority: D(INTR), C(NMI), A(INTO), B(SingleStep)? Actually solution says **D, C, A, B** for increasing — re-check: Increasing means lowest first. So lowest = INTR (maskable), then NMI, then INTO, then Single Step? That contradicts the common mnemonic. The solution sheet writes **D C A B**. Take that as authoritative for this course: INTR (lowest) < NMI < INTO < Single-Step (highest in increasing list). The instructor's convention here treats software/trap as "newest" in service order — **note this for the exam**.
**Trap.** Different textbooks give different orderings. Use the solution-sheet convention: increasing = INTR, NMI, INTO, Single-Step.

### P0.5 — Valid/invalid instruction check `[1M × 5, easy]`
**Concept.** 8086 syntax rules:
- Both operands cannot be memory. `MOV [BX], [SI]` invalid.
- Segment registers cannot take immediate (`MOV DS, 0` invalid → must go via AX).
- `POP CS` invalid; `POP AL` invalid (POP is word-wide on 8086).
- `INC/DEC` on segment registers invalid.
- `ROL [AX], CL` invalid — AX is not a valid base/index for memory addressing (only BX/BP/SI/DI).
- `XCHG [DL], 80H` — XCHG with immediate invalid; also DL is 8-bit so cannot be a memory pointer.
- Jump syntax: `JMP segment:offset` (e.g., `JMP 5B00H:2AC0H`) valid as far jump.
- `INC BP` valid.
**Worked (2016 Q4).** Invalids: `ROL [AX], CL` (AX not addressing reg); `XCHG [DL], 80H` (multiple errors); `POP AL` (no byte POP). Valid: `INC BP`, `JMP 5B00H:2AC0H`.

### P0.6 — Determine AX after one instruction (independent rows) `[1M × 5, easy]`
**Concept.** Read whether destination is **AL** (8-bit, leaves AH alone) or **AX** (16-bit, overwrites both). For sub/cmp: CMP doesn't change destination.
**Worked (2015 Q6).** AX=5F9CH start:
- `CMP AX,BX` → AX unchanged = 5F9CH (CMP only affects flags).
- `SUB AL,AH` → AL = 9C - 5F = 3D, AH unchanged = 5F. AX=5F3D**H**.
- `ROR AH, 3` → AH=5F=01011111, ROR by 3 = 11101011 = EB. AX=EB9CH.
- `SAL AL, 4` → AL=9C=1001 1100, SAL by 4 = 1100 0000 = C0. AX=5FC0H.
- `MOVSX AX, AL` → sign-extend AL=9C (msb=1, negative) into AX = FF9CH.
**Trap.** `MOVSX` is 386+; for 8086, use `CBW` (sign-extend AL → AX).

### P0.7 — Will this code generate the overflow flag? `[3M, easy]`
**Concept.** OF is set when sign of result differs from both operand signs in a way that's impossible mathematically: positive+positive=negative or negative+negative=positive.
**Worked (2016 Q2).** AL=70H (positive, +112), BL=60H (positive, +96). Sum=D0H (negative, −48 in two's-comp). Two positives → negative ⇒ **OF=1**.
**Recipe.** Convert hex to signed (msb=sign), add, check sign-rule violation.

### P0.8 — Set/reset specific flag bits `[4M, medium]`
**Concept.** Direct flag-register manipulation uses `PUSHF / POP AX / <bit-ops> / PUSH AX / POPF`. STC/CLC/CMC for CF; STI/CLI for IF; STD/CLD for DF. No direct instruction for TF, OF, SF, ZF, AF, PF.
**Recipe.** PUSHF → POP AX → AND/OR with bit mask → PUSH AX → POPF.
**Worked (2017 Q7) — set all conditional, reset all status:** Conditional flags = OF/DF/IF/TF (control); Status = SF/ZF/AF/PF/CF. Solution given:
```
MOV  AX, 0FFFFH    ; all 1s
AND  AX, 0700H     ; keep only bits 8,9,10 (TF,IF,DF) — wait, that's reversed
PUSH AX
POPF
```
Inspect: 0700H = 0000 0111 0000 0000 — bits 8 (TF), 9 (IF), 10 (DF). That sets TF/IF/DF and clears everything else. Treat 0700H mask as the canonical "conditional-only-set" mask in this course.
**Worked (2017 Q8) — set only TF, leave rest:**
```
PUSHF
POP  AX
OR   AX, 0100H     ; TF is bit 8
PUSH AX
POPF
```

### P0.9 — ISR return address from stack snapshot `[4M, medium]`
**Concept.** On interrupt, 8086 pushes **FLAGS, CS, IP** (in that order). Stack-top after pushes contains IP-low, IP-high, CS-low, CS-high, FL-low, FL-high. The "return 20-bit address" = CS × 16 + IP.
**Recipe.**
1. Read top 6 bytes from the snapshot, in stack order: IP_lo, IP_hi, CS_lo, CS_hi, FL_lo, FL_hi.
2. Concatenate: IP = IP_hi·IP_lo, CS = CS_hi·CS_lo.
3. Physical = (CS << 4) + IP.
**Worked (2017 Q10).** Stack (top down): 00, 65, 00, 14, 00, 15, 08, 16. IP_lo=00, IP_hi=65 → IP=6500H. CS_lo=00, CS_hi=14 → CS=1400H. Physical = 1400×16 + 6500 = 14000+6500 = **1A500H**. ✓

### P0.10 — Sign-extend an N-bit value using only logical/shift `[4M, medium]`
**Concept.** Sign extension preserves the sign bit by replicating it. `CBW/CWD/MOVSX` do it natively. Without them, use shift trick: shift left to push sign-bit to MSB, then arithmetic-shift right back.
**Recipe.** To sign-extend the low N bits of a 16-bit register: `SHL reg, (16-N)` then `SAR reg, (16-N)`. SAR brings the sign bit back while replicating.
**Worked (2017 Q6) — 12-bit data in AX, AH upper nibble=0:** AX = 0000 XXXX YYYYYYYY. To sign-extend, push sign bit of bit-11 to bit-15, then SAR back:
```
SHL AX, 04         ; bit 11 → bit 15
SAR AX, 04         ; sign-extends bits 15..12
```
Solution writes `ROL AX,04 / SAR AX,04` — but ROL doesn't preserve sign correctly in general; SHL is the canonical answer. (Solution shows that on 80286 ROL was also accepted because AH upper nibble was zero, so ROL ≈ SHL.)
**Trap.** "Only logical instructions" usually excludes SAR; if the question forbids arithmetic shifts, you must reconstruct: copy AX, mask out bits 0-11, test bit 11, branch to set high nibble = 0FFF or 0000.

---

## CLUSTER 1 — 8086 ALP fragments and tracing (Part A and Part C, very high)

### P1.1 — DEBUG `u`/`d` output fill-in (decode addresses & opcodes) `[8M, medium]`
**Concept.** `debugx`/`debug` displays code/data in `[segment]:[offset] [machine code] [mnemonic] [operands]` form. The offset advances by the instruction length each line. The data dump starts at the offset you typed after `d`.
**Recipe.**
1. Locate the start `.startup` IP — first line in the `u` output.
2. For each fill-in: count bytes from the previous line's offset = previous offset + (number of byte-pairs in previous machine-code column).
3. For data segment lookups: addresses inside the variable references (e.g., `[XXXX]`) correspond to **offsets in the data segment**. Use `.data` section to compute offsets:
   - First declared variable → offset 0.
   - Add bytes per declaration to find the next.
4. For data dump (`d 0119`): byte at offset 0119 in DS, the dump shows 16 bytes per line.
**Worked (2023 Part C Q1).** data1 (3 bytes "[,]") at offset 0x0100, data2 (4 words = 8 bytes) at 0x0103, data3 (2 bytes) at 0x010B, array (9 words = 18 bytes) at 0x010D... actually the offsets in the answer indicate `.data` starts at offset 0x0100 in this DOSBOX assembler. Walk through each `MOV` operand:
- `MOV BX, data2` → BX is loaded with the **offset** of data2. From answer: BX, [011C]? No, BX = offset 0103. Solution shows BX, [011C]. The opcode `8B1E1C01` means MOV BX, [011C]. So data2 is at offset 011CH in this run. Recompute layout: data1 (3) at 0100, data2 (8) at 0103? But machine code shows 011CH for data2 — meaning `.data` segment starts at a different alignment. Use the disassembly itself as ground truth.
- Use that disassembly to back-fill: data2 at 011CH ⇒ data1 at 011C - 3 = 0119H. Then array (after data2 (8) and data3 (2)) at 011C + 8 + 2 = 0126H.
**Trap.** Hex offsets in machine code are stored **little-endian**: `BA1619` → operand `1916H` = offset of data3.

### P1.2 — Square-wave / pulse via 8253/54 `[6M, easy-medium]`
**Concept.** OUT freq = CLK / count. Square wave → Mode 3. Continuous pulses → Mode 2 (rate generator).
**Recipe.**
1. count = CLK / output-freq.
2. If count > 65535, **cascade counters**: factor count into products ≤ 65535 each.
3. Build CW for each counter: `SC1 SC0 RW1 RW0 M2 M1 M0 BCD`. RW=11 for LSB-then-MSB; M=011 for mode 3.
4. Load CW to CREG, then load count LSB then MSB to chosen counter port.
**Worked (2015 Q3).** 1 kHz on OUT1, CLK1=1 MHz, base=0BH, counter1 port = 07H.
- Count = 1e6/1e3 = 1000 = 03E8H.
- CW for counter 1, mode 3, binary, 16-bit: `01 11 011 0` = 76H.
```
MOV AL, 76H ; OUT 0BH, AL    ; CW
MOV AL, E8H ; OUT 07H, AL    ; LSB
MOV AL, 03H ; OUT 07H, AL    ; MSB
```
**Worked (Tut-10 Q1).** Generate 278×10⁻⁶ Hz sq-wave, CLK=2.5 MHz. Count = 9×10⁹ — way beyond 65535. Factor as 6×10⁴ × 5×10⁴ × 3. Cascade Counter 0 (rate-gen mode 2, count=EA60H), Counter 1 (mode 2, count=C350H), Counter 2 (square-wave mode 3, count=3). CW0=36H, CW1=76H, CW2=96H. Wire OUT0→CLK1, OUT1→CLK2.

### P1.3 — Multi-counter cascade (Part B big timer problem) `[18M, hard]`
**Concept.** When required period exceeds 2^16/CLK, chain Mode-2 rate generators feeding subsequent counters' CLK input, then a final mode-3 or mode-0 stage for the desired output.
**Worked (2023 Part B Q2).** Interrupt every 85896 s, CLK=4 MHz; Counter 0 already loaded with FFFFH (do not touch).
- Frequency needed = 1/85896 ≈ 1.164e-5 Hz. Total count = 4e6 / 1.164e-5 = 3.4358e11.
- Factor: 3.4358e11 = 65535 × 65536 × 80 → C0=FFFF (untouched, given), C1=FFFFH, C2=50H (=80 dec).
- All counters in mode 2 (rate generator), wire OUT0→CLK1, OUT1→CLK2. INTR=OUT2.
- Control words (binary, RW=11=LSB+MSB except C2 = byte): CW0=34H or 36H (mode 2 or 3), CW1=74H/76H, CW2=94H/95H (byte load).
**Trap.** When loading a 2-byte counter, always write LSB then MSB to the **same** counter port. If a counter only needs 1 byte (count<256), CW must specify RW=01 (LSB only).
**Worked (2017 Part B Q4) — 10-second timer:** CLK=2 MHz, want a 10-s low strobe.
- C0 (mode 3, rate gen): 2 MHz → 100 Hz, count = 20000 = 4E20H, CW=36H.
- C1 (mode 3): 100 Hz → 1 Hz, count = 100 = 64H, CW=76H or 56H (LSB only).
- C2 (mode 5 — h/w-triggered strobe): 1 Hz × 10 = count 10 = 0AH, CW=9AH.
- OUT0→CLK1, OUT1→CLK2, ANSW press → GATE2 (latches & enables strobe).

### P1.4 — Special freq cases (Mode 3 with count=0) `[4M, medium]`
**Concept.** When count is loaded as 0 in mode 3, 8254 interprets it as 2^16 = 65536.
**Worked (2017 Q4).** CLK=2 MHz, count=0000H → 65536. Output freq = 2e6 / 65536 = **30.5175 Hz**.

### P1.5 — Software-delay loop calculation `[4-8M, medium-hard]`
**Concept.** Total cycles = (initialization MOV) + (count × loop-body cycles) + last-iteration JNZ-not-taken difference. JNZ taken = 16 T-states; JNZ not-taken = 4 T-states (on 8086).
**Recipe.**
1. Sum T-states of one loop iteration body.
2. Total delay in seconds = T-states × cycle-time. Set equal to required delay.
3. Solve for count.
**Worked (2017 Q9).** 63 ms @ 5 MHz, body=DEC(2)+NOP(3)+JNZ(16) = 21 T per iteration. T_cycle = 200 ns. Iterations = 63e-3 / (21 × 200e-9) = 15000 = 3A98H.
```
        MOV CX, 3A98H
DELAY:  DEC CX
        NOP
        JNZ DELAY
```
**Worked (2023 Part B Q1).** 4 MHz, body= AND(3) + OR(3) + NOP(3) + LOOP(17). Total per iter except last = 26 T. (LOOP taken=17, not taken=5 — the 26 figure used by the solution assumes 9+17.) Required count = 200e-3 / (26 × 250e-9) = 30769.69 → load 7831H or 7832H.
**Trap.** Don't forget `LOOP` is 17 T (taken) vs `JNZ`=16T. The course expects you to use the T-state table given in the question; do not look up from textbook unless asked.

### P1.6 — Block ALP (genetic-mutation / max-ones / parity / search) `[10-14M, hard]`
**Concept.** These are open-ended ALP problems where you write a small routine. Marking favors: data setup, correct loop with INC pointer + DEC count + JNZ, correct bitwise logic.
**Skeleton template.**
```
.model small / tiny
.data
SRC  db  ...
DST  db  N dup(?)
.code
.startup
    LEA  SI, SRC
    LEA  DI, DST
    MOV  CX, N
X1: MOV  AL, [SI]
    ; <transform AL>
    MOV  [DI], AL
    INC  SI
    INC  DI
    LOOP X1
.exit
END
```
**Worked (2018 Q4) — mutation: XOR bits at positions given by LSB and MSB nibbles of LOC byte.** Solution treats LOC nibbles as positions (0-F) for two XOR-flip locations:
```
LEA  DX, LOC
LEA  DI, MUTATED
LEA  SI, POPULATION
MOV  CH, 30           ; population size
X1: MOV  CL, byte ptr [DX]
    AND  CL, 0FH       ; low nibble = LSB-side position
    INC  CL            ; positions 0..15
    MOV  AX, 00
    STC                ; bit-mask = 1 shifted
    RCL  AX, CL        ; mask for low-side bit
    MOV  BX, word ptr [SI]
    XOR  BX, AX        ; flip
    MOV  CL, byte ptr [DX]
    AND  CL, 0F0H
    SHR  CL, 04        ; high nibble = MSB-side position
    MOV  AX, 00
    INC  CL
    STC
    RCL  AX, CL
    XOR  BX, AX
    MOV  word ptr [DI], BX
    INC  DI ; INC DI
    INC  SI ; INC SI
    INC  DX
    DEC  CH
    JNZ  X1
.exit
END
```
**Worked (2023 Part C Q2) — count-ones / max-element-by-bitcount:**
```
COUNT_ONES PROC          ; counts ones in AL, returns count in AH
    PUSH AX
    PUSH BX
    PUSH CX
    MOV  AH, 0
    MOV  CL, 08H         ; 8 bits
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
```
**Recipe for marking.** Always: PUSHA/POPA in procs, EOI-aware ISRs end with IRET, LEA for offsets, BYTE/WORD PTR for sized memory access.

### P1.7 — Set trap flag preserving others / specific bit operations `[4M, medium]`
Covered in P0.8 — same technique. Trap-flag specifically is bit 8 of FLAGS.

### P1.8 — Translate machine ↔ assembly `[3-6M, medium]`
**Concept.** 8086 instruction format: `[prefix] [opcode] [MOD-REG-R/M] [disp] [imm]`. Opcode tables in Brey Appendix C.
**Recipe.** For exam, you'll typically be given short machine codes. Decompose:
- First byte determines opcode group.
- For MOV-immediate-to-reg: opcode `1011W reg` (B0-BF). Bytes after = immediate.
- For most ALU: 8-bit/16-bit modifier, MOD/REG/R/M follows.
**Worked (2015 Q5iii).** `8AF3H` = `8A F3` = `MOV r8, r/m8` with MOD-REG-R/M = F3 = `11 110 011` → MOD=11 (reg-reg), REG=110 (DH), R/M=011 (BL). Result: `MOV DH, BL`.
`MOV [SI], AH` → `8824H`: opcode 88 = MOV r/m8, r8; modregrm=24H=`00 100 100` → MOD=00, REG=100 (AH), R/M=100 (SI direct). Bytes: 88 24.

---

## CLUSTER 2 — Memory Interfacing (Part B big question, every year)

### P2.1 — Calculate address-line count, map start/end `[5M, easy]`
**Concept.** A chip of size 2^k bytes consumes the low k address lines (A0..Ak−1) internally. The remaining high lines select the chip via decoder. Start address = base; End = base + size − 1.
**Recipe.**
1. log2(chip size in bytes) = local-address lines.
2. End_addr = Start_addr + size − 1.
**Worked (2023 Part A Q3).** 8 × 2764 ROMs (8 KB each = A0..A12 internal). Start = 80000H. ROM1 = 80000–81FFFH? 2764=8K=2^13 → end=80000+1FFF=81FFFH. With two banks (odd/even), ROM1+ROM2 form 16 KB at 80000–83FFF; ROM3+ROM4 at 84000–87FFF; ROM5+ROM6 at 88000–8BFFF; ROM7+ROM8 at 8C000–8FFFF. Then 4 × 6164 RAMs (8K each). Two banks: RAM1+RAM2 at 98000–9BFFF; RAM3+RAM4 at 9C000–9FFFF.
**Trap.** When chips are 8-bit but the bus is 16-bit (8086), pair chips into **odd/even banks**. Each pair holds 2× single-chip bytes.

### P2.2 — Odd/even bank wiring (BHE'/A0 enables) `[4M, medium]`
**Concept.** 8086 16-bit bus: A0=0 enables even bank (D0-D7); BHE'=0 enables odd bank (D8-D15). For 16-bit transfer both are active.
**Recipe.**
1. Connect even-bank chips' CS' to (decoder-out OR A0). When A0=0 → CS' active (low).
2. Connect odd-bank chips' CS' to (decoder-out OR BHE'). When BHE'=0 → CS' active.
3. For 8086, A0 of chips' internal addressing skips bit 0 — i.e., chip's A0 line ties to processor's **A1**.
**Worked (2023 Part A Q3 wiring).** Even-bank ROM1 → 2-input OR(LS138-out, A0); Odd-bank ROM2 → OR(LS138-out, BHE'). LS138 takes A16,A15,A14 as ABC (after pairing eats A13 internally and A0 is bank-select).
**Trap.** Single bank (8088 or 8-bit bus, or 80386 with byte enables): no BHE'/A0 needed — connect chip CS' directly to decoder.

### P2.3 — Generate decoder enables with NOT/AND/NAND/OR `[3M, easy-medium]`
**Concept.** LS138 (3:8 decoder) has 3 select inputs A,B,C and 3 enables G1 (active-high), G2A' G2B' (active-low). Output Yn' goes low when selected and enabled.
**Recipe.**
1. Identify the high address bits (above your decoder selects) that must equal a fixed pattern for any of your chips to be selected.
2. Build a "system enable" using those bits → feed to G2A'/G2B' (active-low) or G1 (active-high) as required.
**Worked (2017 Part B Q1) for 80386, 7 MB at 000000–6FFFFFH:** A23 A22 A21 select via LS138 (8 zones of 1 MB chunks within the 24-bit space spanning 8 MB), but actual mapping uses A23A22A21 + sub-grouping with NAND/NOT. Solution uses LS138 to break the 24-bit address into 8 zones of 2 MB each, then NAND gates split 2-MB zone into 1-MB chunks.

### P2.4 — Full Part-B memory design `[15-20M, hardest]`
**Recipe — universal template.**
1. **List requirements**: total size, ROM/RAM split, starting addresses, chip sizes.
2. **Compute chip counts**: required / single-chip = N. If 16-bit bus, ensure N is even (split into banks). If chip < bank width, multiple chips per byte position aren't applicable here (8-bit chips on 16-bit bus = pairs).
3. **Address-line accounting**: write a binary table with columns A_max..A0. For each chip, write the chip's address range showing high bits as fixed (decoded by LS138 or gates) and low bits as 1s/0s.
4. **Decoder placement**: choose 3 contiguous bits for LS138 A,B,C selects. Choose higher bits for the enable.
5. **Bank logic** (for 16-bit 8086 only): OR(LS138_out, A0) for even, OR(LS138_out, BHE') for odd. (NAND-only style: NAND(LS138_out, A0') for even.)
6. **Draw labelled diagram**: 8086/8088/80286/80386 + decoder + bank-select gates + chips with addr lines A1..Ak, CS' wired to gate output, D0-D7 to even bank, D8-D15 to odd bank, MEMR'→OE', MEMW'→WE'.
7. **Memory map table**: Chip name, size, start (hex), end (hex), bank.

**Common variants seen:**
- 2015 Q1: 64K ROM @ F0000 + 64K RAM1 @ D0000 + 64K RAM2 @ E0000 with 32K chips. (Two 32K chips per memory region, paired as one bank — for 8086 this means need 2 pairs per 64K region.)
- 2017 Q1: 80386, 24-bit bus, 7MB system from 000000 with 512K/1MB chips. Pure NAND/NOT gating.
- 2018 Q2: 32K ROM + 64K RAM from 00000 with 16K ROM and 32K RAM chips, NAND only.
- 2022 Q1: 80286, 1024K (256K ROM + RAM rest) from F00000 onwards with 27256 (32K) and 61512 (64K) chips.
- 2023 Part A Q3: 8086, 8 × 2764 + 4 × 6164 from 80000 with LS138 + 2-input OR.

**Key per-variant differences.**
- 80286/8086 with 8-bit chips → odd/even bank pairing required.
- 80386 → 32-bit data bus, four banks (BE0'-BE3'); but exam problems usually treat 80386 in 16-bit subset mode to simplify (or single-banked).
- "Use only NAND" → build OR using NAND: A OR B = NAND(NOT A, NOT B) = NAND(NAND(A,A), NAND(B,B)). NOT A = NAND(A,A).

### P2.5 — Identify chip-count when only one chip type is available `[3M, easy]`
**Recipe.** chips = total_size / chip_size; multiply by 2 if 8086 and bus is 16-bit.

---

## CLUSTER 3 — Protected Mode (80286/386) — Part A (heavy in recent years)

### P3.1 — GDT vs LDT selection, descriptor lookup `[1-2M each, medium]`
**Concept.** Segment selector (16 bits) = `Index (13b) | TI (1b) | RPL (2b)`.
- TI=0 → GDT, TI=1 → LDT.
- Index × 8 + table base → descriptor address.
- RPL = requested privilege level (0=highest).
**Recipe.**
1. Convert selector to binary.
2. Bits 15..3 = index. Bit 2 = TI. Bits 1..0 = RPL.
3. Descriptor base = GDTR/LDTR + (index × 8).
**Worked (2023 Q1).** DS = 0026H = `0000 0000 0010 0110`. Index = 0026/8 = 4 (since lowest 3 bits=110: 1 = TI, RPL=10). TI=1 → **LDT**. Index = 4 → **4th entry** in LDT. RPL = 10 → privilege 2. Application can access segments with **DPL ≥ max(CPL, RPL)**.

### P3.2 — Compute linear address from descriptor + offset `[2-4M, medium]`
**Concept.** 8-byte descriptor layout (little-endian, 80286):
- bytes 0-1: limit (low 16)
- bytes 2-4: base (low 24)
- byte 5: access rights byte (DPL, type, P, S, E, W/R)
- byte 6: limit (high 4) + flags (G, D/B, 0, AVL) on 80386
- byte 7: base (high 8)
**Recipe.**
1. Read bytes 0-7 of descriptor (the question gives LSB-first hex array).
2. Extract base32 = byte7·byte4·byte3·byte2.
3. Linear = base + offset.
**Worked (2023 Q1).** Descriptor = `00 FB 27 BD 00 00 80 1F` (read as bytes 0..7 from LSB first). Base = byte7=00, byte4=00, byte3=BD, byte2=27 → 00·00·BD·27? Wait. Conventionally, base32 = (byte7)(byte6.bits 31-24 of base = byte7 only on 80386; on 80286 base is 24-bit = byte4·byte3·byte2). For 80386: base = byte7 byte4 byte3 byte2 = `00 80 BD 27`? The solution shows base = 00 80 00 00 and linear = 0080 0010 00? Hmm, solution gives linear address = **0080 0010 0H** for this question. Let me re-derive: take the descriptor bytes 0..7 as given on question (LSB first), the question writes `00 FB 27 BD 00 00 80 1F` → byte indexes 0-7. Base on 80386 = bytes 2,3,4,7 = 27, BD, 00, 1F? Linear addr from solution = 0080001H so apparently base = 008000?? Use the **rule** directly: base32 = byte[7]·byte[4]·byte[3]·byte[2]. Plug in: 1F·00·BD·27 → 1F00BD27. Then linear = base + offset = 1F00BD27 + 00000010 = 1F00BD37. Doesn't match. The given solution **0080 0010 0** likely uses a different descriptor (maybe different selector resolution). For exam: trust the **rule** and the cheat sheet's exact mapping; don't try to memorize byte-order.

> **Best practice in exam**: redo byte indexing from the descriptor figure given on the cheat-sheet attached to the paper — don't memorize indices.

### P3.3 — Paging: linear → physical `[5-7M, medium-hard]`
**Concept.** With CR3 (PDBR) pointing to Page Directory base. Linear address split:
- bits 31-22: PD index (10 bits)
- bits 21-12: PT index (10 bits)
- bits 11-0: offset (12 bits)
**Recipe.**
1. Linear in binary. Slice into 10/10/12.
2. PDE address = CR3 + PD_idx × 4. Read 4-byte PDE; top 20 bits = PT base.
3. PTE address = PT_base + PT_idx × 4. Read 4-byte PTE; top 20 bits = page frame base.
4. Physical = (frame_base << 12) | offset12.
**Worked (2023 Q2).** Linear=01 80 00 65 H = 0000 0001 1000 0000 0000 0000 0110 0101. PD idx = 0000000110 = 6 (6th entry). PT idx = 0000000000 = 0? Actually = 0000000000 = 0. Wait 01 80 00 65 in binary: split top 10 / mid 10 / low 12. The solution says PD entry = 6, PT entry = 9. Let me redo: linear hex `01800065`. Binary = `0000 0001 1000 0000 0000 0000 0110 0101`. Top 10 (b31..b22): `0000 0001 10` = 6. Mid 10 (b21..b12): `00 0000 0000` = 0 → but solution says 9. Hmm. The question may have linear = `0180D065H` — re-check: probably a transcription issue. Regardless, **trust the recipe**: extract bits 31-22 for PD, 21-12 for PT, 11-0 for offset, then look up tables.
**Trap.** PDE/PTE in 80386 are **little-endian 32-bit**, lower 12 bits hold flags (P, RW, US, ...), top 20 bits hold table/frame base.

### P3.4 — Identify privilege/type/access from rights byte `[1-2M, easy-medium]`
**Concept.** Access-rights byte layout:
- bit 7 (P): present
- bits 6-5 (DPL): privilege level
- bit 4 (S): 0=system, 1=code/data
- bit 3 (E): 1=code, 0=data
- bit 2: for data, ED=expand direction (1=expands down/stack-like). For code, C=conforming.
- bit 1: for data, W=writable. For code, R=readable.
- bit 0 (A): accessed.
**Recipe.** Convert byte to binary, decode field-by-field.
**Worked (2023 Q1).** Access byte = 1F? Actually access byte = byte5 of descriptor. For data segment with type bits showing data-expand-up-writable. Solution: "Data segment, expand-up, writable" — bits read as 10010011.

### P3.5 — Compute privilege accessibility `[1M, easy]`
**Rule.** A descriptor with DPL D can be accessed by a process with CPL C only if C ≤ D for data, or via conforming/gate rules for code.
Application with CPL 2: can read any data segment with DPL 2 or 3. Specifically `DS=0026H` with RPL=2 allows access to data DPL ∈ {2, 3} (i.e., **levels 2,3**).

---

## CLUSTER 4 — 8255 PPI (Part A fill-ins + Part B system design, every year)

### P4.1 — Build a Mode-0 Control Word `[2-3M, easy]`
**Concept.** CW layout (D7=1 → mode-set, =0 → BSR):
- D7=1 fixed
- D6-D5: Group-A mode (00=M0, 01=M1, 1x=M2)
- D4: PA dir (1=in, 0=out)
- D3: PCu (PC4-PC7) dir
- D2: Group-B mode (0=M0, 1=M1)
- D1: PB dir
- D0: PCl (PC0-PC3) dir
**Recipe.** Decide each port direction → fill bits.
**Worked (2023 Q4).** PA=in (sensors), PB=out (LEDs), PC=in (sensors). Both groups mode 0. CW = `1 00 1 1 0 0 1` = 99H. ✓
**Worked (2022 Q1 BSR).** BSR set/reset PC5: CW format `0 X X X bit3 bit2 bit1 S/R`. Bit-number=PC5 → bits3-1=101. S/R=0 to reset (turn LED off if active-high). CW = `0 000 101 0` = 0AH.
**Trap.** "Switch off LED" depends on LED polarity — if LED's cathode is on PCx with anode to Vcc, then PCx=1 = OFF. Read circuit carefully.

### P4.2 — Decode 8255 port addresses from base `[1-3M, easy]`
**Concept.** 8255 occupies 4 consecutive port addresses: A1A0 selects PA(00), PB(01), PC(10), CREG(11).
**Recipe.** base = PA address, base+1 = PB, base+2 = PC, base+3 = CREG. But if there's an address-line skip (e.g., A2/A1 used instead of A1/A0 because 8255's A1/A0 are wired to bus's A3/A2), increment by 4 instead of 1.
**Worked (2023 Q4).** Base 8255 at E7H ⇒ PA=E7H, PB=EFH, PC=F7H, CREG=FFH. The spacing of 8 suggests A1A0 of 8255 are wired to A4A3 of 8086 — verify by binary: E7=11100111, EF=11101111, F7=11110111, FF=11111111. Only bits A3,A4 change — confirms wiring.
**Worked (2018 Q3 — parking lot).** Base 87H ⇒ PA=87H, PB=8FH, PC=97H, CREG=9FH (A3,A4 are select pins).

### P4.3 — Data-bus pin assignment when CS' decoding is independent `[1M, easy]`
**Recipe.** 8255 is 8-bit device. If 8086 is on a 16-bit bus, you choose one bank: **even** (D0-D7) if base address has A0=0; **odd** (D8-D15) if A0=1.
**Worked (2022 Q1).** Base B7H, A0=1 → 8255 to **D8-D15** (odd bank).

### P4.4 — Part-B 8255 system design (drone / parking / odometer-display) `[18-30M, hardest]`
**Skeleton:**
1. Read each I/O requirement (sensor → port direction, indicator → port direction, mode).
2. Compute CW for desired direction matrix.
3. Decode CS' from given base address using NAND/AND/OR + LS138.
4. ALP: initialize CW, then loop: `IN AL, port` / process / `OUT port, AL`.
**Worked (2023 Q4 — drone modes).** PA=in, PC=in, PB=out (LEDs on PB3,PB4,PB5). CW=99H.
```
PORTA EQU E7H ; PORTB EQU EFH ; PORTC EQU F7H ; CREG EQU FFH
MOV AL, 99H ; OUT CREG, AL
CALL status_check          ; returns mode in BL: 0=bad, 1=land, 2=flight
CMP BL, 0   ; JE bad_weather_mode
CMP BL, 1   ; JE landing_mode
CMP BL, 2   ; JE normal_flight_mode

bad_weather_mode:    MOV AL, 08H ; OUT PORTB, AL   ; PB3 high (red)
landing_mode:        MOV AL, 10H ; OUT PORTB, AL   ; PB4 high (yellow)
normal_flight_mode:  MOV AL, 20H ; OUT PORTB, AL   ; PB5 high (green)
```
**Worked (2018 Q3 — Smart parking lot).** Two 8255 ports for UID scanners (Port A) and exit-scanner (Port B), Port C for display strobe. ISR fires on entry/exit. Cost = C_sec × (T_OUT - T_IN). Solution stores in_id/out_id arrays, time arrays, uses REPNE CMPSB to find matching ID on exit, multiplies stay-duration by C_sec via MUL.

### P4.5 — 8255 Mode 1/2 (handshake) recognition `[4M, medium, rare]`
**Concept.**
- Mode 1: strobed I/O. PC pins act as handshake signals (STB', IBF, INTR for input; OBF', ACK', INTR for output).
- Mode 2: bidirectional on PA only (PC for handshake).
**Cue from question.** "When data is ready, sensor pulses STROBE..." → Mode 1.

---

## CLUSTER 5 — 8259 PIC (Part A: ICW/OCW write-out; Part B: system integration)

### P5.1 — ICW write-out for single 8259 `[4-6M, medium]`
**ICW formats (8086 mode):**
- **ICW1**: `A7-A5=000 1 LTIM ADI SNGL IC4` — for 8086: bits 7-5=0, bit 4=1 (mandatory), bit3=LTIM(1=level,0=edge), bit2=ADI(don't care for 8086), bit1=SNGL(1=single, 0=cascade), bit0=IC4(1=ICW4 needed).
- **ICW2**: T7-T3 (5 MSB of interrupt type number, low 3 bits = 0). For type 40H, ICW2 = 40H.
- **ICW3** (cascade only):
  - Master: bit n = 1 if IRn has a slave.
  - Slave: low 3 bits = slave ID (matches master's IRn position).
- **ICW4**: `0 0 0 SFNM BUF M/S AEOI µPM`. For 8086: µPM=1. AEOI=0 normal. BUF=0 non-buffered. SFNM=0 default.
**Worked (2018 Q1 — odometer).** IR1 used. Single 8259. Edge-triggered. Non-buffered. No SFNM. AEOI=0.
- ICW1: `xxx1 LTIM ADI SNGL IC4` = `0001 0 0 1 1` = 13H. (LTIM=0 edge, SNGL=1 single, IC4=1.)
- ICW2: Mask IR0 etc; IR0=90H → T7-T3 of 90H = 10010 → ICW2 = `10010 000` = 90H.
- ICW3: not needed (SNGL=1).
- ICW4: `0000 0001` = 01H (8086 mode).
**Worked (2017 Part B Q3 — master + 2 slaves).** Master+Slave1+Slave2. Master IR6→Slave2, IR7→Slave1, AEOI, buffered.
- ICW1 (each) = 11H (cascade, edge, IC4 needed) → answer table shows 11H.
- ICW2 master = 28H (type 28 base). Slave1 = F0H. Slave2 = F8H.
- ICW3 master = C0H = `1100 0000` (slaves on IR6,IR7). Slave1 = 07H (ID=7, on master IR7). Slave2 = 06H.
- ICW4 = 0FH = `0000 1111` (AEOI=1, BUF=1, M/S=1 for master / 0 for slave, µPM=1) → master OF, slave OB.

### P5.2 — OCW write-out `[4-8M, medium]`
**OCW formats:**
- **OCW1** (written to A0=1 odd port): mask byte, bit n = 1 → IRn masked.
- **OCW2** (written to A0=0): `R SL EOI 0 0 L2 L1 L0`. Common values:
  - `001` → non-specific EOI = 20H
  - `011 L2L1L0` → specific EOI for IRn
  - `101` → rotate-on-non-specific-EOI = A0H
  - `100` → automatic rotation set
  - `111 L2L1L0` → rotate on specific EOI, set IRn as lowest
- **OCW3** (A0=0, bit3=1, bit4=0): poll, read IRR/ISR, special mask.
**Worked (2023 Part B Q4).** Mask IR0 & IR5 → OCW1 = `00100001` = 21H. Rotate on specific EOI, set IR2 as lowest → OCW2 = `11100010` = E2H.
**Worked (2017 Q3 master).** OCW1 = 3F (mask all except IR6 & IR7) = `0011 1111` = 3FH.

### P5.3 — Address decoding for 8259 `[4M, medium]`
**Concept.** 8259 has only A0 as device-internal address (selects ICW1/OCW2/OCW3 vs ICW2/OCW1). Base+0 takes ICW1/OCW2/OCW3; base+1 takes ICW2/ICW3/ICW4/OCW1.
**Recipe.** From given base (e.g., 0740H), use M/IO'=0 for I/O-mapped, then decode A1..A15 with LS138/NAND. Skip A0 entirely (let A1 of 8086 = A0 of 8259 because of even/odd alignment).
**Worked (2018 Q1).** Base 740H. 0740=0000 0111 0100 0000. Use LS138 with A3,A4,A5 as ABC selects (selects between port chunks of 8 within block); A6-A15 enable through NAND.

### P5.4 — Write ISR for an interrupt application `[6-10M, medium-hard]`
**Skeleton:**
```
ISR_NAME PROC FAR
    PUSHA                  ; or PUSHF and individual PUSH
    ; <ISR logic>
    MOV  AL, 20H           ; OCW2 = non-specific EOI
    MOV  DX, base_8259
    OUT  DX, AL
    POPA
    IRET
ISR_NAME ENDP
```
**Worked (2018 Q1 — odometer ISR).**
```
ODOMETER PROC FAR
    LEA  BX, TICK
    LEA  CX, MTR
    LEA  AX, KM
    INC  byte ptr [BX]      ; tick++
    CMP  byte ptr [BX], 32H ; reached 50?
    JAE  DIST
    JMP  IEXT
DIST:
    MOV  byte ptr [BX], 0
    MOV  DL, 64H            ; 100m
    ADD  word ptr [CX], DL  ; MTR += 100
    CMP  word ptr [CX], 3E8H ; MTR==1000?
    JE   MTRRST
    JMP  IEXT
MTRRST:
    MOV  word ptr [CX], 0
    INC  word ptr [AX]      ; KM++
IEXT:
    MOV  AL, 20H            ; non-specific EOI
    MOV  DX, 0740H
    OUT  DX, AL
    IRET
ODOMETER ENDP
```
**Worked (2022 Q5 — NMI vs IR6/IR7).** NMI ISR sets bit 0 of memory at 80H without affecting others; IR7 ISR sets bit 1; both must preserve registers.
```
NMI_ISR:
    MOV DX, 80H
    MOV AL, 00H           ; read-modify-write through I/O port; per question, set bit 0
    OUT DX, AL            ; (the question stipulates set bit 0 — adjust if read-modify-write)
    IRET
```
Actually the canonical pattern uses IN/OR/OUT for read-modify-write:
```
    IN  AL, DX
    OR  AL, 01H           ; set bit 0
    OUT DX, AL
```
For IR7: same, but `OR AL, 02H`.

### P5.5 — Full hardware schematic for 8259 `[4-6M, medium]`
**Recipe.**
1. 8086 (or 8088) on left. Pin-out INTR ↔ 8259 INT. INTA' ↔ 8259 INTA'.
2. 8259's A0 ↔ 8086's A1 (because 8086 is 16-bit on even/odd alignment).
3. CS' from decoder using high address bits and M/IO'.
4. IR0-IR7 to your interrupt sources.
5. For cascade: master's CAS0-CAS2 → slaves; slaves' CS'/SP-EN'=0 for slave, +5V for master.

---

## CLUSTER 6 — Machine Cycles & Timing (Part B 6-10M)

### P6.1 — Identify machine cycles per instruction `[6-10M, medium]`
**Concept.** Every instruction generates a sequence of bus cycles: MEMR fetches the instruction byte(s), then MEMR/IOR for operand fetch, then MEMW/IOW for write-back.
**Recipe.**
1. Number of opcode bytes → that many MEMR for **instruction fetch** (at even addresses, one MEMR per word).
2. If instruction reads memory operand → add 1 MEMR/IOR for **operand fetch**.
3. If instruction writes memory or I/O → add 1 MEMW/IOW for **writeback**.
4. List in order: fetches first, then operand, then write.
**Worked (2023 Part B Q3).**
- `XLAT` (1 byte, reads memory at BX+AL): 1 MEMR fetch, 1 MEMR operand. (Solution: 2 lines, MEMR/INST, MEMR/OPERAND.)
- `INC WORD PTR [BX][SI][0123H]` (4 bytes — opcode FF, modregrm, disp lo, disp hi): 1 MEMR fetch (word at even addr, but op is 4 bytes → 2 word reads = **2 MEMR** for fetch on 8086), 1 MEMR operand, 1 MEMW writeback. Solution: 4 lines: 2 MEMR fetch, 1 MEMR operand, 1 MEMW writeback.
- `IN AL, 78H` (2 bytes — E4 78): 1 MEMR fetch (one word), 1 IOR operand. Solution: 2 lines: MEMR/INST + IOR/OPERAND.

### P6.2 — Software delay loop
See P1.5.

---

## CLUSTER 7 — ADC/DAC (Part A 5M)

### P7.1 — ADC selection by resolution `[2.5M, easy]`
**Concept.** Resolution = Vref / 2^n where n = bits. ADC0808 is 8-bit; AD570 is 8-bit (but with bipolar Vref like ±5V).
**Recipe.** Required resolution = sensor mV/unit. Select the ADC whose resolution ≤ requirement (so the LSB is fine enough).
**Worked (2022 Q3).** Sensor 20 mV/°C. ADC0808 with Vref=5V → res = 5/256 = 19.5 mV ≈ 20 mV ✓. AD570 with ±5V to −15V swing has wider range and coarser res. **Pick ADC0808**.

### P7.2 — Conversion time `[2.5M, easy]`
**Concept.** ADC0808 needs ~64 clock cycles for 8-bit successive approximation. At 1 MHz clock → ~64µs. Sometimes quoted as 100µs.
**Worked (2022 Q3).** 1 MHz, ADC0808 → **64 µs** (or 100 µs depending on textbook). Solution shows "1 µs" which is suspect — accept whichever the cheat sheet quotes.

---

## CLUSTER 8 — FAT File System Parsing (Part B, 2015/2016/2017 style)

### P8.1 — Root directory entry parse `[10-12M, medium]`
**Concept.** 32-byte FAT directory entry layout:
- bytes 0-7: filename (8 ASCII, space-padded)
- bytes 8-10: extension (3 ASCII)
- byte 11: attribute (R/H/S/V/D/A bits)
- bytes 12-21: reserved/created times (10 bytes)
- bytes 22-23: time of last write (HMS packed)
- bytes 24-25: date of last write (YMD packed)
- bytes 26-27: starting cluster (little-endian)
- bytes 28-31: file size (4 bytes, little-endian)
**Time packing (16-bit).** `HHHHH MMMMMM SSSSS` (hours×2048 + min×32 + sec/2).
**Date packing (16-bit).** `YYYYYYY MMMM DDDDD` where Y = year − 1980.
**Attribute byte.** bit0=RO, bit1=H, bit2=S, bit3=V, bit4=D, bit5=A.
**Worked (2017 Q2).** Entry = `42 49 54 53 44 41 54 41 - 58 4C 53 23 7B BD 4D 18 - A4 3A 4A 3A 00 00 0F 20 - CE 28 0F 00 04 00 00 04`.
- Name = "BITSDATA" + ext "XLS" → BITSDATA.XLS.
- Attribute = 23H = 0010 0011 = bits 0,1,5 = R + H + A → **Hidden, Read-only, Archive**.
- Created time (bytes 14-15): `4D 18` → 184DH = 0001 1000 0100 1101 → H=3, M=2, S=0×2 = 0. (Time 03:02:00 — but solution says 04:00:30!) Re-check: time = bytes 22-23 (write time), bytes 14-15 are creation time. Solution uses write time `0F 20` (bytes 22-23) → 200FH = 0010 0000 0000 1111 → H=00100=4, M=000000=0, S=01111×2=30. **04:00:30** ✓.
- Write date (bytes 24-25) = `CE 28` → 28CEH = 0010 1000 1100 1110 → Y=20 → 2000, M=0110=6, D=01110=14 → **14 June 2000**. (Solution says "2000 or 2010" — 0010100 in Y gives 20 ⇒ 2000.)
- Starting cluster (bytes 26-27) = `0F 00` → 000FH = **15**.
- Size (bytes 28-31) = `04 00 00 04` → 04000004H = **67108868**, but solution says **269168**. Recompute: 04 00 00 04 little-endian = 04000004H = 67108868. But solution says 269168. So size bytes are `04 00 00 04`? In dec 04000004H=67108868. Solution differs — likely solution mis-parsed. **Use the rule**: little-endian, byte28 LSB → size = byte31·byte30·byte29·byte28.

### P8.2 — FAT-12 cluster chain `[3M, medium]`
**Concept.** FAT-12 = each FAT entry is 12 bits (1.5 bytes). Chain: starting cluster from dir entry → look up FAT[start] → next cluster → FAT[next] → ... until 0xFFF (EOF).
**Recipe.** Walk the chain to find the next-cluster sequence, then convert to byte stream packing every two 12-bit entries into three bytes (little-endian, low nibbles first).
**Trap.** FAT-12 packing: for clusters N (low) and N+1 (high):
- byte0 = N[7:0]
- byte1 = N[11:8] | (N+1)[3:0]<<4
- byte2 = (N+1)[11:4]

### P8.3 — Boot sector parse `[6M, medium]`
**Concept.** Boot sector (DOS BPB) starts with jump (3 bytes), OEM (8 bytes), then BIOS Parameter Block:
- bytes 11-12: bytes per sector
- byte 13: sectors per cluster
- bytes 14-15: reserved sectors
- byte 16: number of FATs
- bytes 17-18: root dir entries
- bytes 19-20: total sectors
**Worked (2015 Q4(B)).** EB 3C 90 / 2A 60 59 60 48 49 43 0A / 00 40 04 01 00 / 02 / 10 01 / 00 AF F0 09 ... → bytes per sector = `00 40` little-endian = 4000H = 16384? Or =`40 00`=64? Conventionally bytes-per-sec = bytes11+byte12 LE = 4000H = 16384 — too large for floppy. Usually 0200H=512. Solution likely interprets as 0040H. Trust the recipe; pull the correct bytes per question.

---

## CLUSTER 9 — DMA, Bus interfaces, miscellaneous (Part A 1-3M, low frequency)

### P9.1 — How does PCI distinguish MEMR vs MEMW? `[3M, easy]`
**Answer.** PCI uses **C/BE'[3:0]** command lines during the address phase. Memory Read = 0110, Memory Write = 0111. (MEMR/MEMW pins from ISA are encoded into the multiplexed AD/CBE protocol.)

### P9.2 — 8237 DMA mode/HRQ/HLDA handshake `[low freq]`
**Concept.** DMA acquires bus by raising HRQ to CPU; CPU asserts HLDA when ready. Modes: block, demand, single, cascade.

---

## CLUSTER 10 — Hardware/Pin/Mode (Part A 1-2M, very low)

### P10.1 — 8086 min vs max mode `[concept]`
- Min mode: CPU drives bus directly. Pin 24-31 = WR', M/IO', INTA' etc.
- Max mode: needs 8288 bus controller. Pins 24-31 redefined to S0'-S2', QS0-QS1, RQ/GT0-1, LOCK'.

### P10.2 — BIU vs EU split, instruction queue, segmentation
- BIU fetches via 6-byte queue; EU executes.
- Physical = (Segment << 4) + Offset.

---

# Part-A Quick Hit-List (closed-book mental loadout)

| Topic | Likely M | Time budget |
|---|---|---|
| Identify addressing mode | 3 | 1 min |
| Interrupt priority order | 2 | 1 min |
| MEMR cycle time | 3 | 1 min |
| 8254 freq / mode count | 4-8 | 4 min |
| Flag manipulation | 4 | 3 min |
| Sign extension | 4 | 3 min |
| Set TF only | 4 | 3 min |
| Software delay | 4 | 5 min |
| ISR return address | 4 | 3 min |
| RCR/RCL trace | 4 | 4 min |
| 8255 BSR | 5 | 3 min |
| ADC selection | 5 | 3 min |
| Protected-mode walk | 10-15 | 12 min |
| 8259 ICW/OCW | 8-12 | 10 min |

**Recommended order on exam day:** Protected-mode walk (uses cheat-sheet, slow) FIRST while fresh. Then 8259 ICW/OCW. Then small-M questions in order.

---

# Open-Book Sheet (Part B) — Essential Templates

Print these and add to your handwritten notes:

**8254 CW** — `SC1 SC0 RW1 RW0 M2 M1 M0 BCD`. Modes: 0=int-on-TC, 1=h/w one-shot, 2=rate-gen, 3=sq-wave, 4=s/w-strobe, 5=h/w-strobe. RW=11 for LSB+MSB.

**8255 CW (M/S)** — `1 GA-M1 GA-M0 PA PCu GB-M PB PCl`. Direction bit: 1=input.

**8255 BSR** — `0 X X X bit3 bit2 bit1 S/R`.

**8259 ICW1** — `0 0 0 1 LTIM 0 SNGL IC4`. For 8086, cascade, edge, ICW4 needed: `0001 0 0 0 1` = 11H. Single mode → SNGL=1 = 13H.

**8259 ICW2** — top 5 bits = T7-T3 of vector base. Bottom 3 = 0.

**8259 ICW3** — Master: bit n = 1 if slave on IRn. Slave: bottom 3 bits = slave ID.

**8259 ICW4** — `0 0 0 SFNM BUF M/S AEOI 1`. 01H = 8086 mode, all defaults.

**8259 OCW2** — `R SL EOI 0 0 L2 L1 L0`.
- 20H = non-specific EOI
- 60+n = specific EOI for IRn
- A0H = rotate on non-specific EOI
- E0+n = rotate on specific EOI, set IRn lowest

**Memory interfacing template:** binary table + LS138 + bank gates (OR-with-A0 for even, OR-with-BHE' for odd).

**Protected mode (cheat sheet)**: Selector format, descriptor byte layout, paging split 10/10/12. Always refer to the cheat sheet given on the exam.

---

# 10 Novel Comprehensive Problems

> These mirror the difficulty/marks of recent Part-B / Part-A questions but use scenarios not in your PYQ corpus. They test the **same underlying concepts** in fresh combinations.

### N1. **Solar-tracker (8086 + 8255 + ADC0808 + 8259) — Part B [22M]**
A solar panel tracks the sun using one 8255 (base 0080H) and one ADC0808 (8 channels) interfaced with 8086.
- Port A: reads ADC output. Port B: 4 LEDs (PB0..PB3) showing tracker state {idle, east, west, fault}. Port C upper: 4 control signals to ADC (ALE, START, OE, EOC=PC4 input).
- ADC reads two light sensors on CH0 (east) and CH1 (west). The tracker rotates east if CH0 - CH1 > 32 LSB, west if CH1 - CH0 > 32 LSB, else idle.
- EOC raises an interrupt on IR3 of an 8259 (base 0040H). Edge-triggered, single mode, IR3 vector 50H, default priorities, normal EOI.

Tasks:
- (a) 8255 control word and ALP to initialize.
- (b) 8259 ICW/OCW values.
- (c) ISR that reads ADC, compares, and pulses the relevant LED.
- (d) Hardware schematic showing CS' decoding for 8255 and 8259 + ADC pins.

**Concept being tested:** integrated multi-peripheral system + bit-banged ADC handshake + interrupt-driven state machine. Tests P4.4, P5.1, P5.4 simultaneously.

### N2. **GPS-pulse counter (8254 + 8259) — Part A [10M]**
A 1-pulse-per-second GPS module fires on IR2 of an 8259. The 8086 must count "missed seconds" — i.e., gaps where the next pulse arrives > 1.5 s after the previous. A free-running 8254 Counter 0 (mode 2) at 1 MHz feeds Counter 1 (mode 0, count = FFFFH) whose current value the ISR latches via OCW3-equivalent latch-counter command.
- (a) Compute Counter 1 input frequency from Counter 0 count (assume Counter 0 = 03E8H).
- (b) What count value in Counter 1 corresponds to 1.5 s?
- (c) Write the ISR to latch and compare.

**Concept:** Cascade analysis + 8254 latch-counter command + ISR. Tests P1.3, P5.4.

### N3. **8086 stack snapshot puzzle — Part A [4M]**
An ISR was entered via `INT 18H`. Inside the ISR, the programmer did `PUSH AX` (AX=1234H) then `PUSH BX` (BX=5678H). The stack pointer afterwards is at offset 1FF0H. Stack-top bytes (top-down): 78 56 34 12 00 04 00 02 00 09. Determine:
- (a) Return CS:IP for the IRET.
- (b) Saved FLAGS value.
- (c) Physical return address.

**Concept:** Stack growth direction + interrupt pushes order. Tests P0.9.

### N4. **Mode-1 strobed input ALP — Part C [8M]**
8255 is configured in Mode 1 for Port A input. Strobe is on PC4, IBF on PC5, INTR on PC3. Port B is in Mode 0 output. CW=0xB0 has been written. Write ALP that:
- Polls IBF on PC5 (i.e., reads Port C, tests bit 5).
- When IBF=1, reads Port A into AL, mirrors it to Port B.
- Uses no interrupts; pure polling loop.

**Concept:** 8255 Mode 1 vs Mode 0, status-read via PC pins. Tests P4.1, P4.5.

### N5. **Mixed memory map (NAND-only) — Part B [16M]**
80286 system at FE00000H upward needs 1 MB ROM at top, 512 KB SRAM below it, 256 KB EEPROM below that. Available: 256 KB chips (×8), only 2-input NAND gates and inverters. No LS138 available.
- (a) Memory map with start/end addresses.
- (b) Decoder logic using only NAND.
- (c) Show even/odd bank wiring on 80286.

**Concept:** All-NAND decoding (build OR, AND from NAND), bank wiring. Tests P2.1-P2.4, NAND identities.

### N6. **Protected-mode descriptor forgery — Part A [10M]**
Given GDTR=00100000H. A descriptor at GDT[8] reads `FF FF 00 00 50 F2 40 00` (LSB-first).
- (a) What is the base, limit, granularity, type?
- (b) An app with CPL=3 attempts to access this segment with `MOV DS, AX` where AX=0043H. Will the descriptor load succeed? Why?
- (c) If granularity bit G=1, what's the effective limit?

**Concept:** Descriptor decoding, RPL/DPL/CPL rules, granularity. Tests P3.1-P3.4.

### N7. **Cascaded delay + ISR coordination — Part B [14M]**
Build a system where every 1 minute, the 8086 ISR (entered via 8259's IR6) blinks a LED on 8255's PC0 by toggling its state. Use a single 8254 with all three counters cascaded from a 4-MHz master clock.
- (a) Counter modes + counts.
- (b) Wiring CLK/OUT chain.
- (c) ISR pseudocode using BSR mode of 8255 to toggle PC0.

**Concept:** 8254 cascade + 8255 BSR + 8259 EOI. Tests P1.3, P4.1, P5.4.

### N8. **ALP — Run-length encoding of a byte array — Part C [10M]**
Source array SRC of 100 bytes. Encode into DST in pairs (count, value), e.g., `AA AA AA BB` → `03 AA 01 BB`. Counts are byte-wide (1–255). Write the routine, handling boundary cases (single-byte runs).

**Concept:** Loop-with-state, pointer arithmetic, BYTE PTR, conditional flush. Tests P1.6.

### N9. **DOSBOX `d` output reverse-engineer — Part C [8M]**
You ran `d 0200` and got:
```
0863:0200  42 49 54 53 50 49 4C 41-4E 49 00 00 41 42 43 44   BITSPILANI..ABCD
```
Source contains:
```
data1 db 'BITS', 'PILANI', 0
data2 dw 0DDCCH, 0EEFFH
```
Where does `data2` start (offset)? Why doesn't the dump match it? Write the corrected `db` directive that would produce the observed bytes.

**Concept:** Endianness, `db` vs `dw`, ASCII to hex. Tests P1.1.

### N10. **Hybrid hardware/software — Part B [22M]**
Design an 8086-based "smart lock" that opens after the correct 4-digit code is keyed in on a 4×4 keypad. Keypad is interfaced via 8255 (PA = column drive, PB = row read, both Mode 0). A latch on PC0 controls the lock motor. After 3 wrong attempts, an alarm on PC1 sounds for 30 seconds (use 8254 to time the 30 s).
- (a) 8255 CW and base address (you choose).
- (b) 8254 cascade + count for 30 s @ 2 MHz.
- (c) ALP for scan-and-compare, with retry counter.
- (d) Hardware schematic.

**Concept:** Keypad row/column scanning, BSR toggling, 8254 one-shot vs rate-gen, retry-state machine. Tests P1.6, P4.1-P4.4, P1.3.

---

# 48-hour study plan (from 2026-05-12)

**Day 1 (May 12)** — Read clusters 2 (memory), 3 (protected mode), 4 (8255), 5 (8259). Solve 2017 Part B Q1, 2018 Part B Q1, 2023 Part A Q3 fresh on paper.

**Day 2 (May 13)** — Read clusters 1 (ALP), 6 (machine cycles), 7 (ADC), 8 (FAT). Solve 2023 Part A Q1+Q2 (protected mode), 2017 Part B Q3 (8259 cascade), 2017 Part B Q4 (8254). Practice 5 novel problems from above.

**Day 3 (May 14, morning)** — Skim cluster 0 cheat-list. Re-derive 8259 ICW1-4 layout from memory. Walk through one full 8086 ALP template (mutation/parking/odometer style). Walk into exam knowing: addressing modes, ICW/OCW table layouts, 8254 mode 2/3 distinction, memory-map binary table, bank-select rule (A0 even, BHE' odd).
