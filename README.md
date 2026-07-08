# TSRI Verilog LAB Easy Practice

This repository collects several TSRI Verilog practice labs for learning basic RTL design, simulation, and verification.
Topics include ALU, FIR filter, FSM, serial data parsing, memory model, LBP image processing, and priority encoder pipeline design.

<img width="1637" height="956" alt="ic_design_flow" src="https://github.com/user-attachments/assets/82802d96-9066-4a10-bfc9-42d906f59a98" />



---

## Lab Overview

| Directory           | Topic                     | Main Files                                        | Description                                                                 |
| ------------------- | ------------------------- | ------------------------------------------------- | --------------------------------------------------------------------------- |
| `LAB3a`             | ALU Design                | `alu.v`, `alu_test.v`                             | 8-bit ALU with multiple opcode operations and automatic testbench checking. |
| `LAB3b`             | FIR Filter                | `fir.v`, `FIR_test.v`                             | 9-tap FIR filter using shift registers and fixed coefficients.              |
| `LAB4a`             | Pattern Detector          | `fsm_bspd.v`, `fsm_test.v`                        | FSM for detecting the serial bit pattern `0010`.                            |
| `LAB4b`             | Serial Data Monitor       | `SDAM.v`, `testbench.v`, `.dat` files             | Parses serial address/data and verifies outputs with pattern files.         |
| `LAB5`              | Basic ALU                 | `alu.v`, `tb.v`                                   | Another 8-bit ALU practice with basic arithmetic and logic operations.      |
| `LAB6`              | Memory Model              | `mem.v`, `tb.v`                                   | 32 x 8-bit memory with bidirectional `inout` data bus.                      |
| `LAB7`              | LBP Processing            | `LBP.v`, `LBP_optimize.v`, `testfixture.v`        | Local Binary Pattern image processing implementation and optimized version. |
| `pipeline_practice` | Priority Encoder Pipeline | `encoder.v`, `encoder_pipeline.v`, `encoder_tb.v` | 32-bit priority encoder and pipelined version for timing improvement.       |

---

## Notes

* Some labs require external `.dat` pattern files. Keep them in the same directory as the testbench.
* This repository is mainly for Verilog practice and learning.
* Practice makes perfect!
---

## Author

Henry Wu

