# CORDIC Coprocessor — PART-1
## Trigonometric and Vector Functions

A Verilog-based FPGA implementation of a **CORDIC (COordinate Rotation DIgital Computer) coprocessor** for efficient computation of trigonometric, vector, and coordinate-conversion functions.

> **Part-1 establishes the core CORDIC engine and implements nine mathematical functions using rotation and vectoring modes.**

---

## 1. Introduction

CORDIC is a hardware-efficient iterative algorithm used to perform mathematical operations using primarily **shift, addition, and subtraction operations**.

Instead of directly implementing trigonometric functions using conventional multipliers or complex mathematical operators, CORDIC decomposes a rotation into a sequence of elementary rotations with predefined angles.

In this Part-1 implementation, a reusable CORDIC engine is developed in **Verilog RTL** and implemented for FPGA hardware.

The design supports:

- Trigonometric functions
- Inverse trigonometric functions
- Vector magnitude
- Cartesian-to-polar conversion
- Polar-to-Cartesian conversion
- Vector rotation

The implementation uses **rotation mode** and **vectoring mode** to support the different mathematical operations.

---

## 2. CORDIC Principle

The basic CORDIC rotation equations are:

$$
x_{i+1}=x_i-d_i(y_i2^{-i})
$$

$$
y_{i+1}=y_i+d_i(x_i2^{-i})
$$

$$
z_{i+1}=z_i-d_i\arctan(2^{-i})
$$

where:

- $x_i$ = X component at iteration $i$
- $y_i$ = Y component at iteration $i$
- $z_i$ = residual angle at iteration $i$
- $d_i$ = rotation direction, either $+1$ or $-1$
- $2^{-i}$ = scaling factor implemented using a shift
- $\arctan(2^{-i})$ = elementary CORDIC angle

The multiplication by $2^{-i}$ is implemented using an arithmetic right shift:

$$
x_i2^{-i}=x_i>>i
$$

$$
y_i2^{-i}=y_i>>i
$$

Therefore, the main CORDIC datapath does not require a general-purpose multiplier.

---

## 3. CORDIC Micro-Rotations

Each iteration performs a small predefined rotation:

$$
\theta_i=\arctan(2^{-i})
$$

Part-1 uses **16 CORDIC iterations**.

| Iteration | $\arctan(2^{-i})$ |
|---:|---:|
| 0 | 45.000000° |
| 1 | 26.565051° |
| 2 | 14.036243° |
| 3 | 7.125016° |
| 4 | 3.576334° |
| 5 | 1.789911° |
| 6 | 0.895174° |
| 7 | 0.447614° |
| 8 | 0.223811° |
| 9 | 0.111906° |
| 10 | 0.055953° |
| 11 | 0.027976° |
| 12 | 0.013988° |
| 13 | 0.006994° |
| 14 | 0.003497° |
| 15 | 0.001749° |

The corresponding fixed-point angle constants are stored in:

```text
atan_lut.v
```

The LUT is implemented using synthesizable logic and provides the elementary angle required by each iteration.

---

# 4. CORDIC Operating Modes

Part-1 uses two fundamental CORDIC modes.

---

## 4.1 Rotation Mode

In rotation mode, an input vector is rotated through a specified angle.

The direction is selected according to the sign of the residual angle:

$$
d_i=
\begin{cases}
+1, & z_i\geq0\\
-1, & z_i<0
\end{cases}
$$

The objective is to reduce:

$$
z\rightarrow0
$$

For an initial unit vector:

$$
(x_0,y_0)=(1,0)
$$

the CORDIC output becomes:

$$
x\approx K\cos(\theta)
$$

$$
y\approx K\sin(\theta)
$$

Rotation mode is used for:

* SIN
* COS
* TAN
* Polar → Cartesian
* Vector Rotation

---

## 4.2 Vectoring Mode

In vectoring mode, the input vector is rotated toward the X-axis.

The direction is determined using the Y component:

$$
d_i=
\begin{cases}
-1, & y_i\geq0\\
+1, & y_i<0
\end{cases}
$$

The objective is:

$$
y\rightarrow0
$$

while obtaining the vector magnitude and angle.

The resulting quantities are approximately:

$$
x\approx K\sqrt{x_0^2+y_0^2}
$$

and:

$$
z\approx\text{atan2}(y_0,x_0)
$$

Vectoring mode is used for:

* ATAN
* ATAN2
* Magnitude
* Cartesian → Polar

---

# 5. Functions Implemented

Part-1 supports **nine operations** selected through a 4-bit operation-select input.

| Opcode | Function | Operation                    | CORDIC Mode         |
| :----: | -------- | ---------------------------- | ------------------- |
|   `0`  | SIN      | $\sin(\theta)$               | Rotation            |
|   `1`  | COS      | $\cos(\theta)$               | Rotation            |
|   `2`  | TAN      | $\tan(\theta)$               | Rotation + Division |
|   `3`  | ATAN     | $\tan^{-1}(x)$               | Vectoring            |
|   `4`  | ATAN2    | $\text{atan2}(y,x)$          | Vectoring            |
|   `5`  | MAG      | $\sqrt{x^2+y^2}$             | Vectoring            |
|   `6`  | CART2POL | $(x,y)\rightarrow(r,\theta)$ | Vectoring            |
|   `7`  | POL2CART | $(r,\theta)\rightarrow(x,y)$ | Rotation             |
|   `8`  | ROTATE   | Vector rotation              | Rotation             |

---

# 6. Mathematical Functions

## 6.1 Sine and Cosine

Starting with a unit vector:

$$
(x_0,y_0)=(1,0)
$$

and rotating it by $\theta$:

$$
x_{raw}=K\cos(\theta)
$$

$$
y_{raw}=K\sin(\theta)
$$

After gain compensation:

$$
x_{out}\approx\cos(\theta)
$$

$$
y_{out}\approx\sin(\theta)
$$

Therefore, the same CORDIC core can generate both sine and cosine.

---

## 6.2 Tangent

Tangent is calculated from the sine and cosine components:

$$
\tan(\theta)=\frac{\sin(\theta)}{\cos(\theta)}
$$

The CORDIC engine produces:

$$
x=K\cos(\theta)
$$

$$
y=K\sin(\theta)
$$

Therefore:

$$
\frac{y}{x}=\frac{K\sin(\theta)}{K\cos(\theta)}
$$

which gives:

$$
\boxed{\tan(\theta)=\frac{y}{x}}
$$

The common CORDIC gain cancels during division.

The tangent division is implemented separately using:

```text
tan_divider.v
```

The divider uses a **restoring shift-subtract division algorithm**.

---

## 6.3 Arctangent

For ATAN, vectoring mode is used to determine the angle of a vector.

For an input:

$$
(x,y)=(1,v)
$$

the output angle approaches:

$$
\theta=\tan^{-1}(v)
$$

---

## 6.4 ATAN2

ATAN2 determines the angle of a vector while considering both X and Y signs:

$$
\theta=\text{atan2}(y,x)
$$

Unlike a single-argument arctangent, ATAN2 provides quadrant-aware angular information.

Quadrant correction is therefore applied around the CORDIC core.

---

## 6.5 Magnitude

The magnitude of a vector is:

$$
r=\sqrt{x^2+y^2}
$$

In vectoring mode, the CORDIC engine produces:

$$
x_{raw}\approx Kr
$$

Gain compensation is then applied:

$$
r\approx\frac{x_{raw}}{K}
$$

---

## 6.6 Cartesian to Polar

Cartesian coordinates:

$$
(x,y)
$$

are converted into polar coordinates:

$$
r=\sqrt{x^2+y^2}
$$

$$
\theta=\text{atan2}(y,x)
$$

Therefore:

$$
\boxed{(x,y)\rightarrow(r,\theta)}
$$

The magnitude is obtained from the vectoring-mode X output and the angle is obtained from the CORDIC residual angle.

---

## 6.7 Polar to Cartesian

For polar coordinates:

$$
(r,\theta)
$$

the corresponding Cartesian coordinates are:

$$
x=r\cos(\theta)
$$

$$
y=r\sin(\theta)
$$

The CORDIC rotation engine performs this conversion using iterative micro-rotations.

Therefore:

$$
\boxed{(r,\theta)\rightarrow(x,y)}
$$

---

## 6.8 Vector Rotation

A vector:

$$
\mathbf{v}=
\begin{bmatrix}
x\\
y
\end{bmatrix}
$$

rotated through an angle $\theta$ gives:

$$
x'=x\cos(\theta)-y\sin(\theta)
$$

$$
y'=x\sin(\theta)+y\cos(\theta)
$$

The CORDIC engine performs this transformation iteratively using shift-and-add/subtract operations.

---

# 7. Fixed-Point Representation

Part-1 uses fixed-point arithmetic to make the design suitable for FPGA implementation.

---

## 7.1 External X/Y Format — Q1.15

The external X/Y data uses signed 16-bit **Q1.15** representation.

```text
16-bit Q1.15

┌─────┬───────────────────────────────┐
│ Sign│          Fraction             │
│  1  │             15                │
└─────┴───────────────────────────────┘
```

The numerical value is:

$$
Value=\frac{Integer}{2^{15}}
$$

The scaling factor is:

$$
2^{15}=32768
$$

Examples:

| Decimal | Integer Representation |
| ------: | ---------------------: |
|     0.5 |                  16384 |
|    0.25 |                   8192 |
|    0.75 |                  24576 |
|    -0.5 |                 -16384 |

Q1.15 provides high fractional precision for normalized signals and trigonometric outputs.

---

# 8. Binary Angle Measurement (BAM)

Angles are represented using a **32-bit Binary Angle Measurement (BAM)** format.

A complete revolution corresponds to:

$$
360^\circ=2^{32}
$$

Therefore:

$$
BAM(\theta)
=
\frac{\theta}{360^\circ}\times2^{32}
$$

Important angle mappings are:

| Angle | 32-bit BAM   |
| ----: | ------------ |
|    0° | `0x00000000` |
|   90° | `0x40000000` |
|  180° | `0x80000000` |
|  270° | `0xC0000000` |
|  360° | `0x00000000` |

The 32-bit representation naturally wraps around after one complete revolution.

---

# 14. RTL Modules

The Part-1 implementation is divided into modular Verilog blocks.

| File            | Description                                                              |
| --------------- | ------------------------------------------------------------------------ |
| `cordic_top.v`  | Top-level control, function selection, preprocessing and output handling |
| `cordic_core.v` | Main iterative CORDIC datapath                                           |
| `atan_lut.v`    | LUT containing CORDIC arctangent constants                               |
| `gain_scale.v`  | CORDIC gain compensation                                                 |
| `sat18to16.v`   | 18-bit to 16-bit saturation and output conversion                        |
| `tan_divider.v` | Restoring divider used for TAN                                           |

---

# 15. CORDIC Core Operation

The core performs one micro-rotation per clock cycle.

For each iteration:

```text
1. Read current X, Y and Z
2. Determine rotation direction
3. Perform arithmetic shifts
4. Perform X update
5. Perform Y update
6. Update residual angle Z
7. Increment iteration counter
```

The basic hardware operations are:

```text
x_shift = y >>> i
y_shift = x >>> i

x_next = x -/+ x_shift
y_next = y +/- y_shift
z_next = z -/+ atan_lut[i]
```

The direction depends on whether the core is operating in rotation or vectoring mode.

---

# 16. Sequential Architecture

The CORDIC engine uses an iterative finite-state-machine architecture.

```text
             ┌─────────┐
             │  IDLE   │
             └────┬────┘
                  │
               start
                  │
                  ▼
             ┌─────────┐
             │  LOAD   │
             └────┬────┘
                  │
                  ▼
             ┌─────────┐
             │ ITERATE │
             │         │
             │ 0 → 15  │
             └────┬────┘
                  │
             16 iterations
                  │
                  ▼
             ┌─────────┐
             │  DONE   │
             └────┬────┘
                  │
                  ▼
             ┌─────────┐
             │  IDLE   │
             └─────────┘
```

With 16 CORDIC iterations, the core performs the computation over multiple clock cycles rather than implementing all iterations as separate combinational stages.

This approach reuses the same arithmetic hardware and therefore provides a compact FPGA implementation.

---

# 17. Interface

The top-level CORDIC interface contains control, input and output signals.

| Signal         | Width | Description                   |
| -------------- | ----: | ----------------------------- |
| `clk`          |     1 | System clock                  |
| `rst`          |     1 | Reset                         |
| `start`        |     1 | Starts a CORDIC operation     |
| `op_sel`       |     4 | Function selection            |
| `x_in`         |    16 | Q1.15 X/input data            |
| `y_in`         |    16 | Q1.15 Y/input data            |
| `angle_in`     |    32 | BAM angle input               |
| `result_x`     |    16 | X output                      |
| `result_y`     |    16 | Y output                      |
| `result_angle` |    32 | BAM angle output              |
| `busy`         |     1 | Indicates active computation  |
| `done`         |     1 | Indicates completed operation |

---

# 18. Function Selection

The 4-bit `op_sel` signal selects the mathematical operation.

```text
op_sel

0000 → SIN
0001 → COS
0010 → TAN
0011 → ATAN
0100 → ATAN2
0101 → MAG
0110 → CART2POL
0111 → POL2CART
1000 → ROTATE
```

The remaining opcode combinations are reserved.

---

# 19. Function Data Flow

```text
                         op_sel
                           │
                           ▼
                 ┌──────────────────┐
                 │ Function Select   │
                 └────────┬─────────┘
                          │
          ┌───────────────┴────────────────┐
          │                                │
          ▼                                ▼
    ROTATION MODE                    VECTORING MODE
          │                                │
    ┌─────┼─────┐                    ┌────┼─────┐
    │     │     │                    │    │     │
   SIN   COS   TAN                  ATAN ATAN2  MAG
    │     │     │                    │    │     │
    └─────┼─────┘                    └────┼─────┘
          │                                │
      POL2CART                         CART2POL
          │
        ROTATE
```

---

# 22. Simulation Results

### Simulation Waveform

![PART-1 Simulation Output](results/simulation/part1_simulation.png)

---

# 23. Simulation Output Table

The final measured simulation values can be documented below.

| Function | Input | Expected | Simulated | Error |
|---|---|---:|---:|---:|
| SIN | 30° | 0.5000 | 0.49997 | 0.00003 |
| COS | 60° | 0.5000 | 0.49997 | 0.00003 |
| TAN | 45° | 1.0000 | 0.99951 | 0.00049 |
| ATAN | 1 | 45° | 45.0026° | 0.0026° |
| ATAN2 | (1,1) | 45° | 44.9974° | 0.0026° |
| MAG | (3,4) | 5.0000 | 5.0000* | ≈0 |
| CART2POL | (3,4) | (5,53.13°) | ≈(5.0000,53.13°)* | Very small |
| POL2CART | (5,53.13°) | (3,4) | ≈(3.0000,4.0000)* | Very small |
| ROTATE | (1,0), 45° | (0.7071,0.7071) | ≈(0.7071,0.7071) | Very small |

---

# 24. LUT Implementation

The CORDIC angle constants are stored in:

```text
atan_lut.v
```

The LUT contains the elementary angles:

$$
\arctan(2^{-i})
$$

for:

$$
i=0,1,2,\ldots,15
$$

This avoids calculating the inverse tangent during runtime.

The LUT therefore converts a mathematical constant-generation problem into a simple hardware lookup operation.

---

# 25. FPGA Resource Utilization

The synthesized design should be evaluated using the FPGA implementation report.

![PART-1 Resource Utilization](results/resource-utilization/part1_utilization.png)

---

# 26. 7-Segment Display Verification

The computed result is mapped to the FPGA's 7-segment display for direct hardware-level verification.

The display provides a simple physical indication that the synthesized RTL is operating correctly on the FPGA

![PART-1 7-Segment Output](results/seven-segment/part1_7segment.jpg)

---

# 27. Recommended Part-1 Directory

```text
PART-1/
│
├── README.md
│
├── rtl/
│   ├── cordic_top.v
│   ├── cordic_core.v
│   ├── atan_lut.v
│   ├── gain_scale.v
│   ├── sat18to16.v
│   └── tan_divider.v
│
├── testbench/
│   └── cordic_tb.v
│
├── constraints/
│   └── <constraints-file>
│
├── images/
│   ├── cordic_architecture.svg
│   ├── cordic_micro_rotation.svg
│   ├── rotation_mode.svg
│   ├── vectoring_mode.svg
│   ├── fixed_point_q115.svg
│   ├── bam_angle.svg
│   ├── quadrant_reduction.svg
│   ├── cordic_gain.svg
│   └── function_map.svg
│
└── results/
    ├── simulation/
    │   └── part1_simulation.png
    │
    ├── resource-utilization/
    │   └── part1_utilization.png
    │
    └── seven-segment/
        └── part1_7segment.jpg
```

---

### Rotation Mode

![Rotation Mode](images/rotation_mode.svg)

### Vectoring Mode

![Vectoring Mode](images/vectoring_mode.svg)

### Fixed-Point Representation

![Q1.15 Fixed-Point Format](images/fixed_point_q115.svg)

### BAM Angle Representation

![32-bit BAM Angle](images/bam_angle.svg)

---

# 30. Design Summary

| Parameter            | Implementation           |
| --------------------- | ------------------------ |
| Algorithm             | CORDIC                   |
| HDL                   | Verilog                  |
| Architecture          | Sequential / Iterative   |
| CORDIC Modes          | Rotation + Vectoring     |
| Iterations            | 16                       |
| External X/Y          | 16-bit Q1.15             |
| Internal X/Y          | 18-bit signed            |
| Angle Representation  | 32-bit BAM               |
| Angle LUT             | 16 entries               |
| CORDIC Gain           | ≈ 1.6467602579           |
| Gain Compensation     | Shift + Add/Subtract     |
| TAN Format            | Q4.11                    |
| TAN Divider           | Restoring Shift-Subtract |
| Saturation            | 18-bit → 16-bit          |
| Functions             | 9                        |
| Hardware Output       | 7-Segment Display        |

---

# 31. Applications

CORDIC-based hardware accelerators are useful in applications where mathematical operations must be performed efficiently using FPGA resources.

Potential applications include:

* Signal processing
* Digital communications
* Robotics
* Navigation systems
* Motor control
* Computer graphics
* Coordinate transformations
* Radar and sonar processing
* Biomedical signal processing
* Digital instrumentation

---

# 32. Conclusion

PART-1 demonstrates the implementation of a reusable **FPGA-based CORDIC coprocessor** capable of performing nine mathematical functions using a common iterative hardware engine.

The architecture combines:

$$
\boxed{
\text{Shift}
+
\text{Add/Subtract}
+
\text{LUT}
+
\text{Control Logic}
}
$$

to perform computationally intensive mathematical operations in FPGA hardware.

The use of fixed-point arithmetic, BAM angle representation, quadrant handling, gain compensation, and an iterative datapath provides a practical balance between numerical accuracy and hardware resource usage.

The verified CORDIC engine developed in Part-1 forms the foundation for the additional functionality introduced in **PART-2**.

---

### PART-2

**Extended CORDIC Coprocessor**

[Go to PART-2 →](../PART-2/)
