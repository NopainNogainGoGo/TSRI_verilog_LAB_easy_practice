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
## 8. Explicitly specified bit width
In the original code, if the output logic was written as:

```verilog
ST2: {WENAB, RENAB} = {1, 1};
```

This style can easily cause problems in Verilog. The reason is that constants without an explicitly specified bit width, such as `1` or `0`, are usually treated as **32-bit integers**.

Therefore:

```verilog
{1, 1}
```

does not necessarily mean the 2-bit value `2'b11`. Instead, many tools may interpret it as a concatenation of two 32-bit values:

```verilog
{32'b...0001, 32'b...0001}
```

This creates a 64-bit value.

However, the left-hand side:

```verilog
{WENAB, RENAB}
```

is only 2 bits wide. When a 64-bit value is assigned to a 2-bit signal, Verilog keeps only the rightmost, or least significant, 2 bits.

Since the least significant 2 bits of the value are:

```verilog
2'b01
```

the actual result may become:

```verilog
{WENAB, RENAB} = 2'b01;
```

which means:

```verilog
WENAB = 0;
RENAB = 1;
```

This is different from the intended result, where both `WENAB` and `RENAB` should be set to `1`.

To avoid this issue, always specify the exact bit width when assigning concatenated or multi-bit signals. For example, use:

```verilog
2'b11
```

or:

```verilog
{1'b1, 1'b1}
```

The combinational output logic can be modified as follows:

```verilog
always @(*) begin
    case (Current_ST)
        // Use 2'b to make sure the bit width matches {WENAB, RENAB}
        ST0:     {WENAB, RENAB} = 2'b10;
        ST1:     {WENAB, RENAB} = 2'b00;
        ST2:     {WENAB, RENAB} = 2'b11;
        ST3:     {WENAB, RENAB} = 2'b10;
        default: {WENAB, RENAB} = 2'b10;
    endcase
end
```

---

## 9. cnt
reg[3:0] cnt;
<img width="865" height="586" alt="image" src="https://github.com/user-attachments/assets/d5b38044-f6bc-4689-b3e3-389cd7692d90" />
With this design, only one counter is needed to implement both `0–7` and `0–15` counting.


---

## 10. memory

## Method 1: Use a Counter as the Memory Address
In this method, a counter is used as the write address of the memory.
Every time new data comes in, the data is stored into memory at the address pointed to by `cnt`.

```verilog
memory[cnt] <= data_in;
cnt <= cnt + 1;
```

For example, if the memory needs to store 8 data values, a 3-bit counter can be used:

```verilog
reg [2:0] cnt;
reg [DATA_WIDTH-1:0] memory [0:7];
```

The counter counts from `0` to `7`. After reaching `7`, it automatically wraps around to `0`.

This method is suitable when the data needs to be stored in a memory array and accessed later by address.

## Method 2: Use Shift Registers to Store Data

In this method, data is stored by shifting registers.

Every time new data comes in, the old data is shifted to the next register, and the newest data is stored in the first register.

```verilog
data_reg[0] <= data_in;
data_reg[1] <= data_reg[0];
data_reg[2] <= data_reg[1];
data_reg[3] <= data_reg[2];
```

This method is suitable when the circuit only needs to keep the most recent several input values.

For example, in FIR filters, moving average circuits, or serial data processing, shift registers are commonly used.

In summary, if the design needs address-based storage, use a counter as the memory address.
If the design only needs to keep recent sequential data, use shift registers.

## 11. Z and X
The value `z` represents a **high-impedance** state. This means the output is electrically disconnected from the circuit.

It is not logic `0` and it is not logic `1`. Instead, the signal is floating.

The `z` value is mainly used for **tri-state buffers** and **shared buses**. When multiple devices share the same wire, only one device should drive the bus at a time. The other devices should output `z` so they do not interfere with the active driver.

For example:

```verilog id="x46k8m"
assign bus_line = (enable) ? data_out : 1'bz;
```

When `enable = 1`, `bus_line` is driven by `data_out`.

When `enable = 0`, `bus_line` becomes `z`, meaning this module is not driving the bus.

This prevents multiple circuits from driving different values onto the same wire at the same time.

---
The value `x` means **unknown**. It indicates that the simulator cannot determine whether the signal should be `0` or `1`.

This value commonly appears during simulation.

One common cause is **uninitialized registers**. For example, flip-flops may start as `x` before a reset signal assigns them a known value.

Another common cause is **logic contention**. If one circuit tries to drive a signal to `1` while another circuit tries to drive the same signal to `0`, the simulator may show the result as `x`.

For example:

```verilog id="z90aq2"
assign bus_line = enable_a ? data_a : 1'bz;
assign bus_line = enable_b ? data_b : 1'bz;
```

If both `enable_a` and `enable_b` are active at the same time, and `data_a` and `data_b` have different values, the bus may become `x` in simulation.

In real hardware, there is no actual `x` value. The voltage will eventually settle to some physical level, but it may be unstable, unpredictable, or even damaging if two outputs fight each other.







