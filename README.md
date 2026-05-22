# NES-E

**Nintendo Entertainment System Emulator** — a cycle-accurate 6502 CPU implementation in Verilog, targeting the [Terasic DE0-Nano SoC](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English&CategoryNo=167&No=941) FPGA development board.

The name is a nod to Nessie.

## Goal

A full NES emulator running on FPGA, with the initial target of getting Donkey Kong playable. This required implementing the NES's three main chips:

| Component | Status |
|-----------|--------|
| 6502 CPU | **Complete** — passed the 6502 functional test suite |
| PPU (Picture Processing Unit) | Not started |
| APU (Audio Processing Unit) | Not started |

The project was abandoned after the PPU proved difficult to approach, and after discovering [MiSTer](https://github.com/MiSTer-devel/Main_MiSTer/wiki) — an open-source FPGA retro gaming platform that already had a mature NES core.

## 6502 Implementation

The CPU implements the full standard 6502 instruction set across all addressing modes:

- Implied, Immediate
- Zero Page, Zero Page Indexed (X/Y)
- Absolute, Absolute Indexed (X/Y)
- Indexed Indirect `(ind,X)`, Indirect Indirect `(ind),Y`
- Relative (branches)

Flag updates are delayed by one clock cycle to match 6502 behaviour. The ALU is a separate module (`ALU.v`) handling ADD, AND, EOR, OR, ASL, LSR, ROL, ROR, and subtraction via inversion. NMI is detected via a falling-edge detector. IRQ and NMI interrupts are handled at instruction boundaries.

CPU glitches and undocumented opcodes are not implemented. Undocumented opcodes are treated as NOPs.

## Architecture

```
de0_nano_soc_baseline.v   (top-level, board pin definitions)
│
├── src/cpu/
│   ├── CPU_6502.v        6502 CPU — fetch, decode, execute
│   ├── ALU.v             Arithmetic and logic unit
│   └── instructions.v    Instruction tasks (`include`d by CPU_6502.v)
│
├── src/memory/
│   └── memory_64k.v      64KB addressable memory
│
├── src/clock/
│   ├── master_clock.v    Master clock controller
│   ├── clk_stepper.v     Single-step clock for debugging
│   ├── clk_count_match.v Triggers at a specific clock cycle count
│   ├── clk_count_stop.v  Halts at a specific clock cycle count
│   ├── clk_int_matcher.v Matches on interrupt timing
│   └── clk_pc_matcher.v  Triggers when PC reaches a specific address
│
├── src/io/
│   ├── uart_rx.v         UART receiver
│   └── uart_tx.v         UART transmitter
│
├── src/debug/
│   ├── monitor_cmnd_ctrl.v     Debug monitor command controller
│   └── monitor_command_fifo.v  Command FIFO buffer
│
├── src/util/
│   ├── falling_edge_detector.v  Used for NMI detection
│   └── increment_64.v           64-bit counter increment
│
└── src/ip/
    └── cpu_pll.v         Quartus PLL megafunction (clock generation)
```

The clock debug modules (`clk_stepper`, `clk_count_match`, `clk_pc_matcher`) allow stepping through execution cycle by cycle and halting at specific instructions or addresses — useful for validating CPU behaviour against the test suite.

The UART and monitor interface allow inspecting CPU registers and memory over serial from a host PC.

## Test Benches

Verilog testbenches in `test_benches/` cover the ALU, CPU, clock utilities, and the falling edge detector.

## Tools

- **Quartus Prime** — synthesis and place-and-route
- **ModelSim** — simulation
- **Hardware:** Terasic DE0-Nano SoC (Intel Cyclone V FPGA)

## License

MIT
