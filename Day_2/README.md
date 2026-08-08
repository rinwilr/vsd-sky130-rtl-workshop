# Day 2 - RTL Synthesis and Coding Styles

This folder contains the practical labs and results for Day 2 of the VSD workshop.

---

## 1. Hierarchical vs Flat Synthesis
Analyzed how Yosys handles sub-modules using the `multiple_modules.v` file.

* **Hierarchical View:** Sub-module structures are preserved as clean blocks.
  ![Hierarchical Schematic](./hierarchical_schematic.png)
* **Flat View:** Module walls are deleted using the `flatten` command, showing all raw gates.
  ![Flat Schematic](./flat_schematic.png)

---

## 2. D Flip-Flop Coding Styles
Simulated an Asynchronous D Flip-Flop design (`dff_async_set.v`) to watch its behavioral timing waves.

* **Simulation Waveform:**
  ![Async DFF Waveform](./dff_asyncset_wave.png)

---

## 3. Multiplier Optimization (Special Case)
Synthesized a 2-bit multiplier design (`mul2`). Because multiplying by 2 in binary is just shifting bits left, Yosys optimized away all physical logic gates and used pure wire connections instead.

* **Netlist Text Proof:** (Shows the `assign` wire stitching code)
  ![Netlist Code](./mult2_netlist_text.png)
* **Technology Schematic Layout:** (Shows the physical bit-shift wiring block)
  ![Multiplier Wiring Schematic](./mult2_schematic.png)
