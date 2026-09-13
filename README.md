# 12-Bit Microprocessor

A CPU with a custom instruction set, in SystemVerilog on a Terasic DE0-CV.

**Stack:** SystemVerilog · Cyclone V (DE0-CV) · Quartus Prime 22.1

## Highlights

- **Custom ISA.** Fixed 12-bit instruction, four 3-bit fields: `OPCODE | RA | RB | RD`. 8 opcodes, 8 registers.
- **No spare bits for an immediate**, so `LDI` reinterprets the two source-register *address* fields as a 6-bit constant.
- **Built for debugging on real hardware** — display mux + slow clock let you single-step and watch every internal value. 
- Single-cycle datapath: fetch, decode, execute, write-back between two clock edges.
- 64-word instruction ROM initialized from `instructions.mif`.

## Instruction set

| Opcode | Name | Effect |
|---|---|---|
| `000` | HALT | Stop fetching |
| `001` | LDI | `RD = {RA, RB}` (6-bit immediate) |
| `010` | ADD | `RD = RA + RB` |
| `011` | ADI | `RD = RA + RB_field` |
| `100` | MUL | `RD = RA * RB`, low 6 bits |
| `101` | CMPJ | If `RA >= RB`, jump forward by `RD` |
| `110` | JMP | Jump to `{RA, RB}` |
| `111` | NOP | No operation |

## Debug features

| Control | Function |
|---|---|
| `SW0` | Reset. Clears PC and register file. |
| `SW1` | Slow-clock mode. Hold `KEY0` to step 1 instruction/second. |
| `SW4:SW2` | Selects `HEX3:HEX0` source — register file, instruction word, PC, opcode, or ALU output |
| `SW7:SW5` | Register select when displaying register file |
| `LED[5:0]` | Live program counter |

## Files

| File | Purpose |
|---|---|
| `microprocessor.sv` | Top level. PC, fetch, ALU, control, debug mux. |
| `ourRegister.sv` | 8 × 6-bit register file. Two read ports + a third for the debug display. |
| `counter.sv` | Parameterized modulo counter. Takes modulus, derives width via `$clog2`. |
| `ourDff.sv` / `ourHex.sv` | D flip-flop with enable; 7-segment decoder. |
| `instructions.mif` | The program. Edit and recompile to run something else. |

## Build

1. Open `microprocessor.qpf` in Quartus Prime 22.1
2. Compile, program `.sof` to a DE0-CV
3. `SW1` up, hold `KEY0` to step through the sample program

## Limitations

- `CMPJ` only jumps forward (`RD` is unsigned). Backward branches need `JMP`.
