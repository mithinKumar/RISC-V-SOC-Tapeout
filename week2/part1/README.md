


# **Part 1: SOC Fundamentals**

### **1. What is an SoC? (System-on-Chip Definition)**

An SoC is a design where **different electronic components** are integrated onto a **single silicon die**.

#### **Primary Drivers & Challenges**

| **Driver (Pro)**     | **Challenge (Con)**                               |
| :------------------- | :------------------------------------------------ |
| Low Area / Low Power | **Adds more complexity** to design                |
| Higher Performance   | **Magnifies heating issues** (Thermal Management) |



### **2. Typical SoC Components**

1. **Processor Subsystem**

   * CPU (Central Processing Unit)
   * GPU (Graphics Processing Unit)
   * DSP / Accelerators

2. **Memory System**

   * On-chip RAM / ROM
   * Memory Controller
   * Flash ROM / Bcd ROM

3. **Interconnects**

   * Bus (e.g., AXI / NoC)
   * DMA Controller (Direct Memory Access)

4. **Peripherals**

   * I/O Interfaces (e.g., USB, SPI, I2C, Ethernet)
   * Power Management / Clock / Timers / PLL (Phase-Locked Loops)

5. **Analog/Mixed-Signal Blocks**

   * ADC / DAC (Analog-to-Digital / Digital-to-Analog Converters)
   * RF (Radio Frequency) Transceivers / Amplifiers / Filters

---

## **2. High-Level SoC Design Flow**

The design flow is separated into two major phases: **Front-End (Logical)** and **Back-End (Physical)**.



### **I. Front-End (Logical)**

| **Stage**                            | **Description**                                     | **Key Software Tools**                       |
| :----------------------------------- | :-------------------------------------------------- | :------------------------------------------- |
| **High-Level Design / Architecture** | SystemC, TLM (Transaction-Level Modeling)           | -                                            |
| **RTL Design**                       | Designing the hardware in **Verilog/VHDL**          | -                                            |
| **Functional Verification**          | Proving the design meets specifications             | Synopsys VCS, Cadence Xcelium                |
| **Logic Synthesis**                  | Translating RTL to a gate-level netlist             | Synopsys DC (Design Compiler), Cadence Genus |
| **Formal Verification**              | Proving logical equivalence between RTL and netlist | -                                            |



### **II. Back-End (Physical)**

| **Stage**         | **Description**                                          | **Key Software Tools**                |
| :---------------- | :------------------------------------------------------- | :------------------------------------ |
| **Floorplanning** | Placing major blocks and defining power grids            | Cadence Innovus, Synopsys IC Compiler |
| **Placement**     | Placing standard cells and optimizing timing             | Integrated into P&R Tools             |
| **CTS**           | Clock Tree Synthesis (Building a low-skew clock network) | Integrated into P&R Tools             |
| **Routing**       | Connecting all pins and ports with metal layers          | Integrated into P&R Tools             |
| **Sign-off**      | Final verification (DRC, LVS, STA) before tape-out       | Siemens Calibre, Synopsys PrimeTime   |

---

## **3. IP (Intellectual Property) Blocks**

SoCs contain pre-verified IP blocks that are manufactured by different corporations.

| **Type**    | **Delivery Format**        | **Description**                                                                      |
| :---------- | :------------------------- | :----------------------------------------------------------------------------------- |
| **Soft IP** | Synthesizable **RTL** only | Technology-independent, flexible, requires synthesis and P&R by the design team.     |
| **Hard IP** | **GDSII** and **LEF**      | Technology-dependent, pre-laid-out (fixed), used for analog and critical components. |

---

## **4. Analog-Digital Co-Design on a Single Chip (Mixed-Signal Principles)**

### **The Problem: Substrate Noise Coupling**

* **Digital:** Generates high-frequency switching noise.
* **Issue:** If analog circuitry shares the same die, sensitive analog signals will be **corrupted**.
* **Phenomenon:** This is called **Substrate Noise Coupling**.


### **Solutions: Isolation and Interface**

#### **A. Isolation at the Die Level**

1. **Separate Ground Planes:**
   Running separate $\text{V}*{\text{SS}}$ and $\text{V}*{\text{DD}}$ rails for the analog and digital sections to prevent noise injection.

2. **Deep N-Well Isolation:**
   A separate **Deep N-Well** is created for analog components on the P-substrate silicon wafer to provide physical isolation from the noisy digital substrate.


#### **B. The Interface**

* **ADCs / DACs** (Analog-to-Digital and Digital-to-Analog Converters) are used for communication between the analog and digital domains.



### **Analog Simulators**

These tools are used for precise, transistor-level simulation of analog blocks:

* Cadence Spectre
* Synopsys HSPICE (High-Speed SPICE)

---



# **Part 2: VSD Baby SoC Overview**

The **VSD Baby SoC** is a simplified SoC designed for learning. It contains three major IP blocks:

1. A **Processor** (The CPU core)
2. A **Clock Generator** (The PLL)
3. An **Analog Converter** (The DAC)



### **Working Flow (High-Level)**

1. **Chip Startup via SPI:**
   First, the **PLL (Phase-Locked Loop)** activates to generate a stable clock that synchronizes the **Processor** and the **DAC** clocks.
2. **Processor Operation:**
   The processor runs instructions.

---

## **2. The Core CPU (RVMyth)**

The **RVMyth Core** is a:

* **RV32** (32-bit RISC-V)
* **6-stage pipelined** CPU core
* Implemented in **TL-Verilog**

The processor runs instructions and uses **32 registers** (The standard RISC-V register file) for cycling/operation.

---

## **3. Digital-to-Analog Converter (DAC)**

### **Role**

* The DAC receives data from the processor and converts it to **analog signals** to be sent through the I/O pins.
* The processor operates in the digital domain to communicate with analog devices like speakers, sensors, etc., so an interface is required.

### **VSD Baby SoC Specification**

* The VSD Baby SoC uses a **10-bit DAC**.

---

## **4. Clock Generation: Phase-Locked Loop (PLL)**

### **Definition**

* The PLL generates a **clock** that is **synchronized in phase with an input signal**.
* There is a **negative feedback loop** within the PLL, so it constantly tries to **lock the input and output clock signals**.



### **Why Use an On-Chip PLL instead of an Off-Chip Clock?**

Using an on-chip PLL mitigates several challenges associated with distributing and using an external clock:

* **Clock Distribution Delays:** Off-chip clocks suffer from variable path lengths across the chip.
* **Clock Jitter:** The clock edge from an off-chip source is often **not sharp** (high jitter).



### **PLL Components**

The PLL is a critical mixed-signal block composed of three main parts:

1. **PFD (Phase Frequency Detector):**

   * It is the "sensor" or analogue circuit that generates a **voltage signal proportional to the error in phase or frequency** of the input or output signal.

2. **Loop Filter:**

   * Used to generate a **clean, noise-less signal** to be fed into the VCO, filtering the error signal from the PFD.

3. **VCO (Voltage-Controlled Oscillator):**

   * Generates a **pulse** whose **frequency adjusts to the control voltage** received from the Loop Filter.


### **Challenges with the Input Clock Source**

The input clock typically comes from an external **crystal oscillator** and can contain issues that the PLL must correct for:

* **High Jitter**
* **Frequency Deviations** (PPM error)
* **Frequency Stability** (temperature dependent)
* **Aging of the oscillator**

---


