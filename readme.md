# 2026 50.002 1D Project

Lucid V2 implementation of a 32-bit ALU datapath and supporting components, built in Alchitry Lab V2.

## Project Overview

This repository contains:

- A top-level FPGA design (`alchitry_top`) that drives IO Shield LEDs and 7-segment display.
- A 32-bit ALU (`alu`) composed of arithmetic, boolean, shift, compare, and multiply units.
- Reusable combinational building blocks (adders, muxes, bit reverse, barrel-shift stages).
- Unit testbenches for core ALU submodules.

## Source Layout

All design files are in `source/`.

### Top-level and Display Support

- `alchitry_top.luc`: board integration, reset conditioning, edge detect/counter hookup, IO routing.
- `decimal_counter.luc`: single decimal digit counter (0-9) with overflow.
- `multi_decimal_counter.luc`: cascaded multi-digit decimal counting.
- `seven_seg.luc`: BCD digit to 7-segment decoder.
- `multi_seven_seg.luc`: multiplexed multi-digit 7-segment driver.

### ALU and Datapath Modules

- `alu.luc`: top ALU module selecting result path using `alufn`.
- `adder.luc`: add/subtract path with status flags `z`, `v`, `n`.
- `rca.luc`: parameterized ripple-carry adder.
- `fa.luc`: 1-bit full adder primitive.
- `boolean.luc`: bitwise logic unit using per-bit 4:1 muxes.
- `compare.luc`: comparison logic (`==`, signed `<`, signed `<=`) from flags.
- `shifter.luc`: SHL/SHR/SRA via left-shift + bit-reverse architecture.
- `left_shifter.luc`: staged 32-bit left shifter.
- `x_bit_left_shifter.luc`: parameterized shift stage (`SHIFT` = 1/2/4/8/16).
- `bit_reverse.luc`: generic bit-order reversal.
- `multiplier.luc`: 32-bit combinational multiplier (lower 32 bits output).
- `mux2.luc`, `mux4.luc`: reusable mux primitives.

## ALU Behavior Summary

`alu` instantiates all functional units in parallel and selects `out` by `alufn[5:4]`:

- `00`: arithmetic family (`adder` output) or multiplier when `alufn[1] = 1`.
- `01`: boolean unit output.
- `10`: shifter output.
- `11`: compare output (1-bit compare padded to 32 bits).

Status flags (`z`, `v`, `n`) are driven from the adder path.

## Testbenches

The following unit tests are provided in `source/`:

- `test_adder.luc`: add/subtract correctness and flag checks.
- `test_boolean.luc`: AND/OR/XOR/pass-A logic checks.
- `test_compare.luc`: equals, signed less-than, signed less-or-equal checks.
- `test_multiplier.luc`: positive/negative/overflow-truncation multiplication cases.
- `test_shifter.luc`: SHL/SHR/SRA behavior and shift-amount edge cases.

Each testbench defines 8 directed test cases and validates outputs using `$assert`.

## How to Run

1. Open `2025-50002-1d.alp` in Alchitry Lab V2.
2. Use the simulator to run the `test_*` testbenches.
3. Build/upload `alchitry_top` for on-board validation.

## Notes

- The top-level uses reduced divider values when `$is_sim()` is true to speed up simulation.
- Multiplier output is the lower 32 bits of the product.
