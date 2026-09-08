# CORDIC_COPROCESSOR

## FPGA-Based CORDIC Coprocessor

This repository contains the RTL implementation of a **CORDIC-based hardware coprocessor** for efficient trigonometric computation on FPGA. The project is developed progressively in two stages, where the first part establishes and verifies the core CORDIC computation and the second part extends the design with additional functionality and hardware-level interfacing.

The implementation focuses on understanding how an iterative CORDIC algorithm can be mapped into FPGA hardware using shift, add/subtract, and lookup-table operations instead of conventional multiplier-intensive trigonometric computation.

---

## Project Organization

The development is intentionally divided into two independent stages:

```text
CORDIC_COPROCESSOR
│
├── PART-1
│   ├── RTL source
│   ├── Testbench
│   ├── Simulation results
│   ├── Resource/LUT analysis
│   └── 7-segment display verification
│
└── PART-2
    ├── RTL source
    ├── Testbench
    ├── Simulation results
    ├── Resource/LUT analysis
    └── 7-segment display verification
```

### PART-1 — Core CORDIC Implementation

PART-1 establishes the fundamental CORDIC computation and verifies the basic mathematical operation through RTL simulation.

The implementation concentrates on the core datapath, iterative rotation process, fixed-point representation, angle handling, and generation of the required trigonometric outputs.

The corresponding implementation, simulation waveforms, and hardware results are available in:

➡️ [`PART-1/`](./PART-1/)

### PART-2 — Extended CORDIC Coprocessor

PART-2 builds upon the verified CORDIC foundation and introduces the next level of functionality required for the complete coprocessor implementation.

This stage focuses on extending the datapath/interface and integrating the additional output and hardware-verification features required by the final design.

The corresponding implementation, simulation results, resource analysis, and hardware output are available in:

➡️ [`PART-2/`](./PART-2/)

---

## Design Flow

The overall development follows the sequence:

```text
CORDIC Algorithm
       │
       ▼
Fixed-Point Representation
       │
       ▼
RTL Architecture
       │
       ▼
Functional Simulation
       │
       ▼
Synthesis & LUT Analysis
       │
       ▼
FPGA Implementation
       │
       ▼
7-Segment Display Verification
```

---

## Verification

Each project stage is verified at multiple levels rather than relying only on RTL simulation.

### 1. RTL Simulation

Testbenches are used to apply different input conditions and verify the generated CORDIC outputs.

Simulation results include:

* Input angle / input values
* Calculated CORDIC output
* Expected/reference output
* Waveform verification
* Functional correctness

### 2. FPGA Resource Analysis

The synthesized design is evaluated using FPGA implementation reports.

The relevant results include:

| Resource   | PART-1 | PART-2 |
| ---------- | -----: | -----: |
| LUTs       |    TBD |    TBD |
| Flip-Flops |    TBD |    TBD |
| BRAM       |    TBD |    TBD |
| DSP        |    TBD |    TBD |
| IO         |    TBD |    TBD |

> Resource values will be updated using the final synthesis/implementation reports.

### 3. 7-Segment Display Verification

The computed result is also interfaced with the FPGA's **7-segment display** to provide a direct hardware-level indication of the output.

The repository will include photographs/screenshots of the corresponding hardware output under the `results/` directory of each part.

---

## Tools Used

* **Verilog HDL**
* **Xilinx Vivado**
* **RTL Simulation**
* **FPGA Synthesis & Implementation**
* **7-Segment Display Interface**
* **Boolean Board**

---

## Project Objective

The primary objective of this project is to develop and verify a **hardware-oriented CORDIC coprocessor** and study how an algorithm traditionally implemented using software-level trigonometric functions can be transformed into an efficient FPGA datapath.

The two-stage structure provides a clear progression from the fundamental CORDIC computation to the extended coprocessor implementation.

---

## Development

This repository represents the progressive development of the CORDIC coprocessor, with each part maintained separately to preserve the design evolution and make the implementation easier to study, verify, and reproduce.
