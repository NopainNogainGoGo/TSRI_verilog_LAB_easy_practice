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

# Design Notes

## 1. Design Compiler Setup
In Design Compiler, `target_library` specifies the standard cell library used during synthesis.

```tcl
set target_library "slow.db"
```

Usually, the **slow corner** must pass timing first because cell delay is larger in the slow corner, making setup timing violations more likely.

Common corner concepts:

```text
slow    → stricter for setup timing
fast    → often used for hold timing checks
typical → reference condition
```
---

## 2. Timing Report Notes

By default, `report_timing` usually reports only the worst timing path.

To view more paths, use:

```tcl
report_timing -max_paths 10
```

To separately check setup and hold timing:

```tcl
report_timing -delay_type max
report_timing -delay_type min
```

Where:

```text
-delay_type max → setup timing
-delay_type min → hold timing
```

Before tape-out, you should not only check one worst path. You should examine several critical paths and confirm whether the issues are concentrated in certain circuits, such as multipliers, adder trees, memory interfaces, or long combinational paths.

---

## 3. Reset Notes

If the reset is an active-low asynchronous reset, common names are:

```verilog
rst_n
```

Common coding style:

```verilog
always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        q <= 1'b0;
    else
        q <= d;
end
```

## 4. Array Reset

### Issue

In Verilog, you cannot directly assign a value to an entire memory array at once.

For example, the following style is illegal or not synthesizable:

```verilog
val = 8'b0;
```

If `val` is an array, for example:

```verilog
reg [7:0] val [0:5];
```

Then each element must be reset separately.

### Correct Style

Use a `for loop` to initialize each element:

```verilog
integer i;

always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        for (i = 0; i < 6; i = i + 1) begin
            val[i] <= 8'b0;
        end
    end
    else begin
        // normal operation
    end
end
```

Key point:

```text
A memory array cannot be reset all at once.
Use an index or a for loop to reset each element.
```

---

## 5. Signals Assigned Inside an `always` Block Should Be Declared as `reg`

### Issue

In Verilog, module outputs are `wire` by default.

If a signal is assigned inside an `always` block, it must be declared as `reg`.

Incorrect style:

```verilog
output [9:0] out_n;
```

If `out_n` is assigned inside an `always` block, this will cause problems.

### Correct Style

```verilog
module SMC (
    input  [1:0] mode,
    output reg [9:0] out_n
);

always @(*) begin
    case (mode)
        2'b00: out_n = val[3] + val[4] + val[5];
        2'b01: out_n = val[0] + val[1] + val[2];
        default: out_n = 10'd0;
    endcase
end

endmodule
```

Key point:

```text
assign statements usually drive wire signals.
Signals assigned inside always blocks should be declared as reg.
reg does not always mean a real hardware register.
Whether it becomes a register depends on the always block style.
```

For example:

```verilog
always @(*)
```

This usually represents combinational logic.

```verilog
always @(posedge clk)
```

This usually represents sequential logic or registers.

---

## 6. Multi-driven Signals

### Issue

The same signal should not be assigned in multiple `always` blocks.

For example:

```verilog
always @(posedge clk) begin
    val[i] <= data_in;
end

always @(*) begin
    val[i] = sorted_value;
end
```

This causes `val[i]` to be driven by two different circuits. The synthesis tool will report an error.

This type of issue is called:

```text
multi-driven signal
multiple drivers
```

### Solution

The related logic should be combined into one block, or different temporary signals should be used.

---

## 7. Reset Signals to Avoid Unknown Values

### Issue

In RTL simulation, if a register is not reset, its initial value may be unknown:

```text
X
```

This may cause later calculation results to also become `X`, making waveforms difficult to debug.

For example:

```verilog
reg [7:0] count;

always @(posedge clk) begin
    count <= count + 1'b1;
end
```

If `count` has no initial value, it may be `X` at the beginning of simulation.

### Correct Style

```verilog
always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        count <= 8'd0;
    else
        count <= count + 1'b1;
end
```

Key point:

```text
Important registers should be reset.
FSM states must be reset.
Counters should usually be reset.
Output registers should usually be reset.
```

This prevents many unknown values in simulation and makes the circuit startup behavior more stable.

---



