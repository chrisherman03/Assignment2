Assignment 2
Chris Herman
Repository link: https://github.com/chrisherman03/Assignment2

What I built:
- Reg4 — four 1-bit D flip-flops (Q only) with a gated clock (EN · CLK) to store a 4‑bit value.
- RegisterFile — 4× Reg4 (R0–R3) with:
  • One write port: 2 -> 4 decode of WriteReg1:0, ANDed with WE and per‑register allow.
  • Two independent read ports: each built from cascaded 2:1 muxes (R0/R1, R2/R3 -> final select).
  • CtrlRegs[7:0] hook: LSB of each pair gates writes (AllowR0 = bit0, AllowR1 = bit2, AllowR2 = bit4, AllowR3 = bit6). For tests, these allows were set to 1.
- ALU4 — ripple of 1‑bit full adders with operand M selected by S1, S0:
  • 00 -> B, 01 -> ~B, 10 -> 0000, 11 -> 1111
  • Result: D = A + M + Cin -> implements Add, Add+Carry, Sub (A+ ~B + 1), Sub w/Borrow form (A + ~B), Transfer A, Inc, Dec, Transfer A (alt).
- RF_ALU_Top — integration: ReadData1 -> A, ReadData2 -> B, D -> RegisterFile D (writeback), shared control pins exposed.

Controls:
- CLK (tick to latch on rising edge)
- WE (global write enable)
- WriteReg1:0 (select R0–R3 for write)
- ReadReg1_1:0 (Read Port 1 select R0–R3 -> A)
- ReadReg2_1:0 (Read Port 2 select R0–R3 -> B)
- S1, S0, Cin (ALU operation select per table)
- CtrlRegs[7:0] (per‑register write allow; tests used allows=1)

Tests:
1) Seed R0 = 0011 (3) — ReadReg1 = R0, set op to Transfer/Inc/Dec as needed; WE = 1, WriteReg=R0, tick CLK until R0 reads 0011; WE = 0.
2) Seed R1 = 0101 (5) — same method on R1.
3) Add (A + B) — ReadReg1 = R0 (A = 3), ReadReg2=R1 (B = 5), S1S0Cin = 000 -> D = 1000 (8).
4) Add + Carry — same A/B, S1S0Cin = 001 -> D = 1001 (9).
5) Subtract (A−B) — ReadReg1 = R1 (A = 5), ReadReg2 = R0 (B = 3), S1S0Cin = 011 -> D = 0010 (2).
6) Subtract w/Borrow form — same A/B, S1S0Cin = 010 -> D = 0010 (2).
7) Transfer A — ReadReg1 = R0, S1S0Cin = 100 -> D = A.
8) Increment A — ReadReg1 = R0, S1S0Cin = 101 -> D = A+1.
9) Decrement A — ReadReg1 = R1, S1S0Cin = 110 -> D = A−1.
10) Writeback demo — ReadReg1 = R1 (5), ReadReg2 = R0 (3), S1S0Cin = 000 (Add), WE = 1, WriteReg = R2, tick CLK, WE = 0; then ReadReg1 = R2 & S1S0Cin = 100 (Transfer) -> D = 1000 (8) confirms stored result.