# Day-1

This contains **3 theory sections** and **3 lab exercises**.

---

## 📘 Theory 1 — Introduction to Simulators and Testbenches (TB)

* Simulation consists of a **Simulator** and a **Testbench (TB)**.
* The simulator applies test vectors and checks the design's output.
* The **TB** generates and controls input changes (stimulus).
* The TB itself does not have primary inputs or outputs, unlike the DUT (Design Under Test).
* Simulators can be:

  1. Proprietary (commercial)
  2. Open-source (e.g., Icarus Verilog / Iverilog).
* Open-source simulators like **Iverilog** are **event-driven** — evaluation happens only when signals change.
* Inputs to the simulator:

  * the design (Verilog RTL or gate-level netlist)
  * the testbench
* Simulator output:

  * **VCD file** (Value Change Dump), viewable in **GTKWave**.

---

## 🧪 Lab 1 — GitHub Repo Cloning and Inspection

We cloned and explored the repository used for this workshop:
[https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop](https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop)

Steps:

1. Clone the repository:

   ```bash
   git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop
   ```
2. Inspect the directory contents.

Directory structure (summary):

| Directory          | Description                                                                     |
| :----------------- | :------------------------------------------------------------------------------ |
| **lib**            | Contains sky130 standard cell libraries and related technology files.           |
| **verilog_models** | Contains Verilog models of standard cells (e.g., AND, OR, etc.) for simulation. |
| **verilog_files**  | Contains RTL designs and corresponding Testbenches (TBs).                       |
| **Other files**    | May include scripts, documentation, synthesized netlists, or constraint files.  |

---

## 🧪 Lab 2 — Simulating Designs (Using Icarus Verilog - Iverilog)

This lab demonstrates compilation, execution, and waveform viewing of Verilog designs.

Steps:

1. Compile the design and its testbench:

   ```bash
   iverilog design.v tb_design.v
   ```

   → Produces an executable (default: `a.out`).

2. Run the executable:

   ```bash
   ./a.out
   ```

   → Runs the simulation and dumps results into a VCD file (e.g., `design.vcd`).

3. View the waveform in GTKWave:

   ```bash
   gtkwave design.vcd
   ```

**Typical Testbench (TB) structure**:

1. Instantiate the DUT.
2. Generate stimulus (test vectors).
3. Use `$dumpfile` / `$dumpvars` for waveform dumping.
4. Use time delays (`#10`, etc.) for timing control.

**Example outputs** (placeholders):

* Simulation terminal output → [PLACEHOLDER – img1-simulation-terminal]
* Simulated waveform in GTKWave → [PLACEHOLDER – img2-simulated-waveform]

---

## 📚 Theory 2: Introduction to Yosys (Synthesis Tool)

* **Synthesis** converts an **RTL design** into a **gate-level netlist**.
* Uses a **standard cell library** (e.g., **Sky130**) to map high-level RTL into real logic gates.
* **Inputs to Yosys**:

  * RTL Verilog source (`.v`).
  * Standard cell library (`.lib`).
* **Output from Yosys**:

  * A **gate-level netlist** in Verilog (`design_netlist.v`).

---

## 🧠 Theory 3: Synthesis & Cell Selection

### 🔁 Post-Synthesis Simulation

* After synthesis, the **netlist** is re-simulated (e.g., with `iverilog`).
* **Goal** → Confirm that **RTL sim** = **Gate-level sim** (functional equivalence).

### 📂 Standard Cell Library (`.lib`)

* Contains **modules** like AND, OR, DFF, etc.
* Each logic gate has **multiple flavors** (different sizes/power trade-offs).

### ⚡ Gate Drive Strength

* **High Drive Strength Cells**:

  * Wider PMOS/NMOS → **fast switching**.
  * Fix **setup violations** (critical paths).
  * Condition:
    [
    T_{clk} > T_{coA} + T_{comb} + T_{setup}
    ]

* **Low Drive Strength Cells**:

  * Slower → used to fix **hold violations** (path too fast).
  * Condition:
    [
    T_{hold} < T_{coA} + T_{comb}
    ]

### ⚖️ Load & Trade-offs

* Output load = **input capacitance** of connected gates.
* **Wider transistors** = lower delay, but **higher area + power**.
* **Yosys cell selection** is guided by **timing/area constraints**.

---

## 🛠️ Lab 3: Synthesis Flow with Yosys

| Step   | Command                                   | Purpose                                                    |
| ------ | ----------------------------------------- | ---------------------------------------------------------- |
| **1**  | `read_liberty -lib /path/to/sky130.lib`   | Load standard cell library (.lib).                         |
| **2**  | `read_verilog design.v`                   | Load the RTL Verilog file.                                 |
| **3**  | `synth -top design_name`                  | Run synthesis, specifying top module.                      |
| **4**  | `abc -liberty /path/to/sky130.lib`        | Technology mapping & optimization with ABC.                |
| **5**  | `show`                                    | (Optional) Generate schematic view of synthesized netlist. |
| **6a** | `write_verilog design_netlist.v`          | Export synthesized netlist.                                |
| **6b** | `write_verilog -noattrs design_netlist.v` | Export netlist without attributes (cleaner).               |

---
