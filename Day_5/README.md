# Day 5: Optimization in Synthesis

## 1. If-Else Statements in Verilog
`if-else` statements are used for conditional execution within procedural blocks (`always`, `initial`, `task`, or `function`). They establish priority-encoded logic when synthesized into hardware.

### Syntax
```verilog
if (condition1) begin
    // Code block executed if condition1 is true
end else if (condition2) begin
    // Code block executed if condition2 is true
end else begin
    // Code block executed if no previous conditions match
end
```
* **Priority Encoding:** The synthesis compiler processes conditions sequentially from top to bottom. The first condition that evaluates to true determines the execution path, creating a cascading multiplexer structure in hardware.

---

## 2. Inferred Latches in Verilog
In behavioral modeling, a hardware **latch (Level-Triggered Storage)** is unintentionally inferred if a variable inside a combinational procedural block (`always @(*)`) is not assigned a value across all possible execution paths.

### Cause of Latch Inference
When a path is left unspecified (e.g., an `if` statement missing an `else` branch, or a `case` statement missing a case option/`default`), the synthesis tool is forced to retain the variable's previous state. This generates a feedback loop in hardware, yielding an unwanted latch.

### Resolution
Ensure that every variable written to inside a combinational block is assigned a value under every imaginable execution branch, or specify a default assignment at the very beginning of the procedural block.

---

## 3. Labs for If-Else and Case Statements

### Lab 1: Incomplete If Statement (`incomp_if.v`)
**RTL Source Code:**
```verilog
module incomp_if (input i0, input i1, input i2, output reg y);
always @(*) begin
    if (i0)
        y <= i1;
end
endmodule
```
**Results & Technical Analysis:**
![Incomplete If Source Graph](in_comp_if.png)
![Incomplete If Synthesis Output](incomp_synth.png)
* **Analysis:** The `if` statement lacks an `else` clause. If `i0` is false, `y` must hold its value. As a result, the synthesis engine infers a level-triggered latch (`sky130_fd_sc_hd__dlxtp`) instead of pure combinational gates.

---

### Lab 3: Nested If-Else (`incomp_if2.v`)
**RTL Source Code:**
```verilog
module incomp_if2 (input i0, input i1, input i2, input i3, output reg y);
always @(*) begin
    if (i0)
        y <= i1;
    else if (i2)
        y <= i3;
end
endmodule
```
**Results & Technical Analysis:**
![Nested If Source Graph](icomp2.png)
![Nested If Synthesis Output](incomp2synth.png)
* **Analysis:** While this nested structure builds a cascading priority line, it still lacks a final fallback `else` statement. If both `i0` and `i2` evaluate to false, `y` drops into hold-state, generating an inferred latch in the synthesis netlist.

---

### Lab 5: Complete Case Statement (`comp_case.v`)
**RTL Source Code:**
```verilog
module comp_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
always @(*) begin
    case(sel)
        2'b00 : y = i0;
        2'b01 : y = i1;
        default : y = i2;
    endcase
end
endmodule
```
**Results & Technical Analysis:**
![Complete Case Graph](compcase.png)
![Complete Case Synthesis Output](compcase_synth.png)
* **Analysis:** This design represents a clean, fully specified multiplexer. By mapping distinct select options (`2'b00`, `2'b01`) and blanketing the remaining space (`2'b10`, `2'b11`) with a explicit `default` path, latch inference is completely avoided. The output maps directly to standard combinational multiplexer cells.

---

### Lab 7: Incomplete Case Handling (`bad_case.v`)
**RTL Source Code:**
```verilog
module bad_case (
    input i0, input i1, input i2, input i3,
    input [1:0] sel,
    output reg y
);
always @(*) begin
    case(sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b1?: y = i3; // Wildcard expression usage
    endcase
end
endmodule
```
**Results & Technical Analysis:**
![Bad Case Graph](badcase.png)
![Bad Case Synthesis Output](incomp_case_synth.png)
* **Analysis:** Using wildcard or unmapped combinations without a primary `default` path creates holes in the combinational truth table. If `sel` switches to an unhandled matching space during execution, the network latches, producing a synthesis-simulation mismatch.

---

### Lab 8: Partial Assignments in Case (`partial_case_assign.v`)
**RTL Source Code:**
```verilog
module partial_case_assign (
    input i0, input i1, input i2,
    input [1:0] sel,
    output reg y, output reg x
);
always @(*) begin
    case(sel)
        2'b00: begin
            y = i0;
            x = i2;
        end
        2'b01: y = i1; // Missing assignment for 'x'
        default: begin
            x = i1;
            y = i2;
        end
    endcase
end
endmodule
```
**Results & Technical Analysis:**
![Partial Case Graph](Screenshot_2025-05-28_12-39-30)
![Partial Case Area Report](partial_co.png)
* **Analysis:** This lab illustrates partial variable tracking bugs. Although the case statement handles all `sel` states using a `default` arm, the `2'b01` execution path updates `y` but fails to provide a value for `x`. Consequently, `y` synthesizes to clean combinational gates, while `x` forces the inference of a latch.

---

## 4. For Loops in Verilog
A behavioral `for` loop is used inside procedural loops (`always` or `initial`) to duplicate block computations. It does **not** create hardware sequentially over time; instead, the synthesis tool unrolls the statements to form concurrent parallel hardware pathways.

### Key Rule
To be synthesizable, the loop limits and loop counter range must be static constants evaluated completely at compile time.

---

## 5. Generate Blocks in Verilog
A `generate` block uses structural loops (`for`, `if-else`, or `case`) to programmatically duplicate hardware objects (like module instantiations, primitive gates, or continuous assignments) during design compilation.

### Key Difference: For Loops vs. Generate Blocks
* **Behavioral `for` loop:** Placed *inside* an execution block (`always`). It handles multiple procedural evaluations of variables.
* **Structural `generate for` block:** Placed *outside* procedural blocks. It acts as an automated hardware duplication script using structural index ranges (`genvar`).

---

## 6. What is a Ripple Carry Adder (RCA)?
A **Ripple Carry Adder (RCA)** computes the binary sum of two $N$-bit numbers by cascading $N$ Full Adder structural instances in a chain. 

The carry output signal ($C_{out}$) generated by each individual full adder stage hooks directly into the input carry pin ($C_{in}$) of the subsequent higher-significant bit stage. The computation "ripples" sequentially across the hardware from the Least Significant Bit (LSB) up to the Most Significant Bit (MSB).

![RCA Architecture Diagram](rca_org.png)

---

## 7. Labs on Loops and Generate Blocks

### Lab 9: 4-to-1 MUX Using For Loop (`mux_generate.v`)
**RTL Source Code:**
```verilog
module mux_generate (
    input i0, input i1, input i2, input i3,
    input [1:0] sel,
    output reg y
);
wire [3:0] i_int;
assign i_int = {i3, i2, i1, i0};
integer k;
always @(*) begin
    y = 1'b0; // Default anchor assignment to prevent latching
    for (k = 0; k < 4; k = k + 1) begin
        if (k == sel)
            y = i_int[k];
    end
end
endmodule
```
**Results & Technical Analysis:**
![Mux Loop Unrolling Graph](mux_generate.png)
* **Analysis:** The synthesis engine fully unrolls the loop counter range from $0$ to $3$. The underlying hardware parses the sequential `if` branches into a fast parallel combinational decoder tree structure, resulting in a glitch-free multiplexer.

---

### Lab 10: 8-to-1 Demux Using Case (`demux_case.v`)
**RTL Source Code:**
```verilog
module demux_case (
    output o0, output o1, output o2, output o3,
    output o4, output o5, output o6, output o7,
    input [2:0] sel,
    input i
);
reg [7:0] y_int;
assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int;
always @(*) begin
    y_int = 8'b0;
    case(sel)
        3'b000 : y_int[0] = i;
        3'b001 : y_int[1] = i;
        3'b010 : y_int[2] = i;
        3'b011 : y_int[3] = i;
        3'b100 : y_int[4] = i;
        3'b101 : y_int[5] = i;
        3'b110 : y_int[6] = i;
        3'b111 : y_int[7] = i;
    endcase
end
endmodule
```
**Results & Technical Analysis:**
![Demux Case Logic Tree](demux-case.png)
* **Analysis:** Explicitly listing all 8 combinations of the 3-bit selection vector maps the hardware directly to clear, independent gating cells, routing the primary input `i` to the selected output port with zero storage overhead.

---

### Lab 11: 8-to-1 Demux Using For Loop (`demux_generate.v`)
**RTL Source Code:**
```verilog
module demux_generate (
    output o0, output o1, output o2, output o3,
    output o4, output o5, output o6, output o7,
    input [2:0] sel,
    input i
);
reg [7:0] y_int;
assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int;
integer k;
always @(*) begin
    y_int = 8'b0;
    for (k = 0; k < 8; k = k + 1) begin
        if (k == sel)
            y_int[k] = i;
    end
end
endmodule
```
**Results & Technical Analysis:**
![Demux Loop Unrolling Tree](demux-generate.png)
* **Analysis:** The synthesis engine rolls out the 8 individual loop conditions into a parallel comparator fabric. This produces an identical gate network to the explicit case statement, proving that looping constructs optimize down to clean combinational decoders.

---

### Lab 12: 8-bit Ripple Carry Adder with Generate Block (`rca.v`)
