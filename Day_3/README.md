# Day 3 - Combinational and Sequential Optimizations

## 1. Introduction to Optimizations
An optimized design is highly efficient in terms of circuit area and power consumption. Optimization happens during synthesis when the tool analyzes the RTL logic expressions and maps them to the minimal number of technology standard cells from the `.lib` file.

### Combinational Logic Optimization
Techniques used for Combinational Logic Optimization:
* **Constant Propagation:** If any input of a logic gate is a constant value (e.g., tied to Ground or VDD), the tool simplifies the Boolean expression to eliminate unnecessary inputs or gates.
* **Boolean Logic Optimization:** The tool uses Boolean algebraic laws to simplify complex logic structures (e.g., Karnaugh maps and Quine-McCluskey logic reductions done by the compiler).

### Sequential Logic Optimization
Techniques used for Sequential Logic Optimization:
* **Sequential Constant Propagation:** If a flip-flop's input is a constant value, the register may always maintain a constant state, allowing the tool to optimize it out entirely.
* **State Optimization:** Elimination of unused, redundant, or unreachable finite state machine (FSM) states.
* **Retiming:** Shifting the positions of flip-flops across combinational logic boundaries to improve clock frequency and timing margins without changing circuit functionality.
* **Sequential Logic Cloning (Floor Plan Aware Synthesis):** Replicating a register that drives logic blocks placed very far apart on the physical floorplan to avoid timing violations caused by long wire delays.

---

## 2. Combinational Logic Optimizations (Labs)

### Lab 1: Optimizing Simple Expressions (`opt_check.v`)
The module `opt_check` represents a simple logic expression with a constant input.

**Yosys Execution Commands:**
```bash
\$ yosys
yosys> read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog opt_check.v
yosys> synth -top opt_check
yosys> opt_clean -purge
yosys> abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show -format png -prefix opt_check_mapped
```
**Synthesized Schematic Layout:**
![Gate Level Schematic for opt_check](opt check schematic.png)

### Lab 2: Optimizing Multiple Outputs (`opt_check2.v`)
**Yosys Execution Commands:**
```bash
yosys> read_verilog opt_check2.v
yosys> synth -top opt_check2
yosys> opt_clean -purge
yosys> abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
**Synthesized Schematic Layout:**
![Gate Level Schematic for opt_check2](opt_check2 schematic.png)

### Lab 3: Advanced Combinational Check (`opt_check3.v`)
**Synthesized Schematic Layout:**
![Gate Level Schematic for opt_check3](opt_check3_schematic.png)

### Lab 4: Complex Multi-Level Optimization (`opt_check4.v`)
**Synthesized Schematic Layout:**
![Gate Level Schematic for opt_check4](opt_check4_schematic.png)

### Lab 5: Sub-Module Hierarchical Optimization (`multiple_module.v`)
We analyze how Yosys performs flat versus hierarchical tracking for optimization paths across multiple boundary modules.
* **Hierarchical Logic Optimization Check:**
  ![Multiple Module Opt 1](multiple_module_opt.png)
* **Flattened Module Logic Optimization Check:**
  ![Multiple Module Opt 2](multiple_module_opt2.png)

---

## 3. Sequential Logic Optimizations (Labs)

### Lab 1: Sequential Constant Propagation 1 (`dff_const1.v`)
**Yosys Execution Commands:**
```bash
\$ yosys
yosys> read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog dff_const1.v
yosys> synth -top dff_const1
yosys> opt_clean -purge
yosys> abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
**Simulation Verification & Schematic:**
* ![GTKWave Simulation for dff_const1](dff_const1_waveform.png)
* ![Gate Level Schematic for dff_const1](dff const1.png)

### Lab 2: Sequential Constant Propagation 2 (`dff_const2.v`)
**Simulation Verification & Schematic:**
* ![GTKWave Simulation for dff_const2](dff_const2_waveform.png)
* ![Gate Level Schematic for dff_const2](dff const2.png)

---

### Technical Analysis: Structural Mechanics of Labs 1 to 5

The sequential labs demonstrate how the synthesis compiler evaluates initialization parameters, active-high/low control polarities, and steady-state constant propagation boundaries.

#### 1. Analysis of `dff_const1` (No Reset / Matching State)
* **Circuit Behavior:** In `dff_const1`, the input `D` is tied to a constant logic high value (`1'b1`). When the system transitions, the flip-flop output `Q` simply tracks this constant value.
* **Optimization Mechanism:** Because the value of `D` never toggles and there is no asynchronous override condition to change `Q` to a different state, Yosys identifies the register as a dead tracking element.
* **Synthesis Result:** Yosys optimizes away (deletes) the physical D-Flip-Flop standard cell entirely during the `opt_clean -purge` pass. The output net `Q` is hardwired directly to the power rail (VDD/Logic 1).

#### 2. Analysis of `dff_const2` (With Reset Overrides)
* **Circuit Behavior:** In `dff_const2`, the input `D` is still tied to an operational constant value (`1'b1`). However, the block includes an active-high asynchronous reset pin that forces the output `Q` to logic low (`1'b0`) when asserted.
* **Optimization Mechanism:** Yosys cannot evaluate this module as a pure constant. Although `Q` stays at `1` during normal clocked operation, it must preserve the physical capability to drop to `0` when the reset line goes high. Because the reset state (`0`) is the inverse of the operational steady state (`1`), the circuit must retain its memory element.
* **Synthesis Result:** Yosys cannot optimize away the sequential cell. The final netlist retains a physical hardware D-Flip-Flop instance (`sky130_fd_sc_hd__dfrtp`) to preserve the dual-state logic behavior required by the reset condition.

#### 3. Analysis of `dff_const3` (Reset Matching Constant Input)
* **Circuit Behavior:** In `dff_const3`, the input `D` is tied to a constant logic high value (`1'b1`). The block features an active-high reset that forces the register output `Q` to logic high (`1'b1`) when activated.
* **Optimization Mechanism:** Yosys checks all possible logical states of the register. Since the clocked steady-state value (`1'b1`) perfectly matches the reset override value (`1'b1`), the output net `Q` will never transition to `0` under any operational condition.
* **Synthesis Result:** Because `Q` is unconditionally stuck at `1`, the memory cell is completely redundant. Yosys purges the physical D-Flip-Flop from the gate-level netlist and shorts the output pin directly to the VDD power rail.

#### 4. Analysis of `dff_const4` (Active-Low Set Overrides)
* **Circuit Behavior:** This module utilizes an active-low asynchronous control structure. The input `D` is tied to a constant logic low (`1'b0`), while an active-low clear/set pin forces the output `Q` to logic high (`1'b1`) when pulled low (`0`).
* **Optimization Mechanism:** The default clocked state (`0`) differs from the control override state (`1`). Additionally, the control signal relies on active-low logic topology. The tool must map hardware that detects the falling edge of the control wire while sustaining an operational `0`.
* **Synthesis Result:** Yosys must preserve the register to manage the state boundary transition. It implements a specialized Sky130 technology cell containing an inverted preset port (`sky130_fd_sc_hd__dfrbp`) to respect the active-low logic requirement.

#### 5. Analysis of `dff_const5` (Active-Low Control with Matching States)
* **Circuit Behavior:** In `dff_const5`, the input `D` is tied to a constant logic high (`1'b1`). It features an active-low asynchronous control input that overrides the output `Q` and forces it to logic high (`1'b1`).
* **Optimization Mechanism:** The synthesis compiler checks the steady-state logic level (`1'b1`) against the active-low override value (`1'b1`). Even though the activation polarity of the control wire is inverted, the target output destination value remains identical across all modes.
* **Synthesis Result:** Since the register can never hold or output a logic low (`0`) state under any valid environmental stimulus, the flip-flop is optimized out. Yosys removes the cell structure and hooks the net directly to the primary power rail.

---

### Lab 3: Sequential Constant Propagation 3 (`dff_const3.v`)
**Simulation Verification & Schematic:**
* ![GTKWave Simulation for dff_const3](dff_const3_waveform.png)
* ![Gate Level Schematic for dff_const3](dff_const3.png)

### Lab 4: Sequential Constant Propagation 4 (`dff_const4.v`)
**Simulation Verification & Schematic:**
* ![GTKWave Simulation for dff_const4](dff const4_waveform.png)
* ![Gate Level Schematic for dff_const4](dff_const4.png)

### Lab 5: Sequential Constant Propagation 5 (`dff_const5.v`)
**Simulation Verification & Schematic:**
* ![GTKWave Simulation for dff_const5](dff_const5_waveform.png)
* ![Gate Level Schematic for dff_const5](dff_const5.png)

---

### Lab 6: Counter Logic Optimization (`counter_opt.v`)
Analysis of unused bits or dead-state terminal count loop trimming during synthesis iterations. When a counter's primary logic outputs are only partially utilized by the outer block, Yosys cleans up the unused tracking flip-flops to minimize silicon overhead.

**Synthesized Schematic Layout:**
![Counter Optimization Layout](counter_opt.png)
