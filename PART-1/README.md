# PART-1 — Core CORDIC Implementation

## Overview

PART-1 establishes the **core hardware implementation of the CORDIC algorithm** as the foundation of the coprocessor.

The design translates the iterative CORDIC rotation process into RTL hardware using fixed-point arithmetic, shift operations, additions/subtractions, and stored angle constants. The objective of this stage is to obtain the required trigonometric computation directly from FPGA logic and verify its functional behavior through simulation.

The detailed mathematical explanation of the CORDIC algorithm is provided in the project poster; this README focuses on the actual FPGA implementation and verification.

---

## PART-1 Focus

The first stage concentrates on:

* CORDIC iterative datapath
* Fixed-point input/output representation
* Angle processing
* Shift-and-add/subtract based computation
* CORDIC lookup-table constants
* RTL functional verification
* FPGA synthesis and resource analysis
* 7-segment display output

---

## Implementation Flow

```text
Input
  │
  ▼
Angle / Data Processing
  │
  ▼
CORDIC Iterations
  │
  ├── Shift
  ├── Add/Subtract
  └── LUT-based Angle Constants
  │
  ▼
Trigonometric Output
  │
  ├── Simulation
  │
  └── FPGA → 7-Segment Display
```

---

## Simulation Results

The RTL testbench was used to verify the CORDIC computation for different input conditions.

### Simulation Output

Add the final simulation screenshot here:

```text
[ INSERT PART-1 SIMULATION OUTPUT IMAGE ]
```

Example location:

```text
results/simulation/part1_simulation.png
```

---

## FPGA Resource Utilization

Add the synthesis/implementation results obtained from Vivado.

| Resource  | Utilization |
| --------- | ----------: |
| LUT       |         TBD |
| Flip-Flop |         TBD |
| BRAM      |         TBD |
| DSP       |         TBD |
| IO        |         TBD |

### LUT / Resource Report

```text
[ INSERT PART-1 RESOURCE UTILIZATION SCREENSHOT ]
```

---

## 7-Segment Display Output

The computed result was mapped to the FPGA 7-segment display for hardware-level verification.

```text
[ INSERT PART-1 7-SEGMENT DISPLAY IMAGE ]
```

Example:

```text
results/seven-segment/part1_output.jpg
```

---

## PART-1 Status

| Verification Stage     | Status      |
| ---------------------- | ----------- |
| RTL Compilation        | ✅ Completed |
| Functional Simulation  | ✅ Completed |
| Synthesis              | ✅ Completed |
| Resource Analysis      | ✅ Completed |
| 7-Segment Verification | ✅ Completed |

---

## Next Stage

PART-1 provides the verified CORDIC foundation used for the extended implementation in **[PART-2](../PART-2/)**.
