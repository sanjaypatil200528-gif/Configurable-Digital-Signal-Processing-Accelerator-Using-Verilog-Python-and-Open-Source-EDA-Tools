# Hardware-Accelerated FIR Filter
### Configurable Digital Signal Processing Accelerator Using Verilog, Python, and Open-Source EDA Tools

<p align="center">

  **RTL Design | Digital Signal Processing | Fixed-Point Arithmetic | Functional Verification | Hardware Synthesis**

</p>

---

## 1. Project Overview

The Hardware-Accelerated FIR Filter is a digital signal processing project that implements a configurable low-pass Finite Impulse Response (FIR) filter using synthesizable Verilog/SystemVerilog.

The project combines hardware design with software-based reference verification to investigate the trade-offs between numerical accuracy, processing latency, throughput, and hardware resource utilization.

A Python reference model generates expected filter outputs using NumPy and SciPy. The RTL implementation is simulated using Verilator, and Yosys is used to synthesize the design and estimate its hardware requirements.

The objective is to establish a reproducible workflow for designing, verifying, evaluating, and optimizing a digital signal processing accelerator using free and open-source tools.

### Primary objectives

- Design a parameterized low-pass FIR filter.
- Implement signed fixed-point arithmetic in RTL.
- Generate and quantize filter coefficients using SciPy.
- Verify hardware outputs against a Python reference model.
- Investigate latency, initiation interval, and sustained throughput.
- Estimate hardware resource requirements using Yosys.
- Explore architectural optimizations for FPGA and ASIC-oriented designs.

> **Project status:** Development in progress. Performance figures and verification results must be populated after executing the implementation and measurement flow.

---

## 2. Problem Statement

Digital signal processing algorithms are frequently implemented in software on CPUs, DSP processors, or embedded processors.

Although software implementations are flexible, certain applications benefit from dedicated hardware acceleration, particularly when they require predictable timing, sustained sample processing, or integration into a larger digital system.

An FIR filter is a useful starting point for hardware acceleration because its behavior is mathematically well-defined and its implementation exposes important digital design concepts.

These include:

- Sequential state management.
- Multiply-accumulate datapaths.
- Signed arithmetic.
- Fixed-point precision.
- Pipeline design.
- Parallelism and resource sharing.
- Hardware verification.
- Synthesis and implementation constraints.

This project explores how an FIR algorithm can be mapped into digital hardware and how implementation choices affect performance and resource consumption.

---

## 3. FIR Filter Fundamentals

A Finite Impulse Response filter calculates its output using a finite number of present and previous input samples.

For an N-tap filter:

\[
y[n] = \sum_{k=0}^{N-1} h[k]x[n-k]
\]

Where:

- \(x[n]\) is the current input sample.
- \(h[k]\) is the coefficient at tap \(k\).
- \(N\) is the number of filter taps.
- \(y[n]\) is the filtered output.
- \(n\) represents the discrete-time sample index.

For a 16-tap FIR filter:

\[
\begin{aligned}
y[n] ={}& h[0]x[n] + h[1]x[n-1] \\
        &+ h[2]x[n-2] + \cdots \\
        &+ h[15]x[n-15]
\end{aligned}
\]

Each output is computed from the current input and the previous 15 samples.

### Why use a low-pass FIR filter?

A low-pass filter preserves lower-frequency signal components while attenuating higher-frequency components.

Potential applications include:

- Audio signal processing.
- Sensor data conditioning.
- Communication receivers.
- Digital measurement systems.
- Embedded signal processing.
- FPGA-based data acquisition.

The filter's actual frequency response depends on its coefficients and sampling configuration.

---

## 4. System Architecture

```text
                  INPUT SAMPLES
                        |
                        v
              +-------------------+
              | Input Interface   |
              | in_valid / x_in   |
              +-------------------+
                        |
                        v
              +-------------------+
              | Sample Delay Line |
              | Registers         |
              +-------------------+
                        |
                        v
              +-------------------+
              | Coefficient       |
              | Multiplication    |
              +-------------------+
                        |
                        v
              +-------------------+
              | Accumulation      |
              | Datapath          |
              +-------------------+
                        |
                        v
              +-------------------+
              | Fixed-Point       |
              | Scaling & Output  |
              +-------------------+
                        |
                        v
                 FILTERED OUTPUT
                        |
                        v
              +-------------------+
              | Python Reference  |
              | Comparison        |
              +-------------------+
                        |
                        v
              +-------------------+
              | Verification and  |
              | Performance Report|
              +-------------------+
```

### Design components

| Component | Responsibility |
|---|---|
| Input interface | Accepts signed input samples |
| Delay line | Stores previous input samples |
| Coefficient storage | Holds the quantized FIR coefficients |
| Multiplication datapath | Multiplies samples by coefficients |
| Accumulator | Sums the products |
| Output scaling | Converts the accumulated result to the output format |
| Output interface | Provides the filtered sample and validity signal |
| Python reference | Computes expected fixed-point outputs |
| Synthesis flow | Estimates hardware implementation requirements |

---

## 5. Technology Stack

| Technology | Purpose |
|---|---|
| Verilog/SystemVerilog | RTL implementation |
| Verilator | RTL simulation and linting |
| Python | Test automation and analysis |
| NumPy | Numerical operations and signal generation |
| SciPy | FIR coefficient design |
| Yosys | RTL synthesis and resource estimation |
| Git | Source version control |
| GitHub | Source hosting and project documentation |

All core tools are available as free or open-source software.

Actual support for FPGA-specific resource reporting depends on the selected synthesis flow and target device.

---

## 6. Functional Specifications

The following specifications define the initial design target.

| Parameter | Initial target |
|---|---|
| Filter type | Low-pass FIR |
| Number of taps | 16 |
| Input data width | 16 bits |
| Coefficient width | 16 bits |
| Accumulator width | 40 bits |
| Coefficient format | Signed Q1.15 |
| Input format | Signed 16-bit integer |
| Output format | Signed 16-bit integer |
| Input control | `in_valid` |
| Output control | `out_valid` |
| Reset | Synchronous, active-high |
| Verification reference | Python |
| Simulation engine | Verilator |
| Synthesis tool | Yosys |

These are initial design parameters, not measured performance results.

A configurable implementation should validate parameter combinations and use consistent arithmetic widths throughout the datapath.

---

## 7. Fixed-Point Arithmetic

Hardware implementations often use fixed-point arithmetic to reduce the cost of numerical operations compared with floating-point implementations.

This project uses signed integer samples and quantized FIR coefficients.

### Coefficient quantization

A floating-point coefficient can be converted to Q1.15 representation using:

\[
h_q[k] = \operatorname{round}(h[k] \times 2^{15})
\]

Where:

- \(h[k]\) is the floating-point coefficient.
- \(h_q[k]\) is the quantized integer coefficient.
- \(2^{15}\) is the fixed-point scaling factor.

The coefficient is stored as a signed 16-bit integer.

### Multiplication

Each input sample is multiplied by its corresponding coefficient:

\[
p[k] = x[n-k]h_q[k]
\]

The multiplication result requires sufficient width to preserve the signed product.

For 16-bit signed inputs and 16-bit signed coefficients, a 32-bit signed product can represent the complete mathematical product range.

### Accumulation

The unscaled output is calculated as:

\[
A[n] = \sum_{k=0}^{N-1} x[n-k]h_q[k]
\]

The accumulator must be wide enough to prevent overflow for the supported input and coefficient ranges.

For the initial 16-tap design, a 40-bit accumulator provides additional headroom, subject to verification of the actual supported coefficient and input constraints.

### Output scaling

The accumulated integer is converted back to the output scale:

\[
y[n] = \operatorname{sat}_{16}
\left(
A[n] \mathbin{\text{arith-shift-right}} 15
\right)
\]

The output is saturated to the signed 16-bit range:

\[
-32768 \leq y[n] \leq 32767
\]

The Python reference and RTL must implement identical scaling, rounding, saturation, and overflow behavior.

---

## 8. Coefficient Design

SciPy can generate low-pass FIR coefficients using window-based filter design.

Example Python coefficient generation:

```python
import numpy as np
from scipy.signal import firwin

TAPS = 16
FRAC_BITS = 15

coeff_float = firwin(
    numtaps=TAPS,
    cutoff=0.2,
    window="hamming",
    pass_zero="lowpass",
)

coeff_quantized = np.round(
    coeff_float * (2 ** FRAC_BITS)
).astype(np.int64)

if np.any(coeff_quantized < -32768):
    raise ValueError("Coefficient underflow")

if np.any(coeff_quantized > 32767):
    raise ValueError("Coefficient overflow")

print("Floating-point coefficients:")
print(coeff_float)

print("Quantized coefficients:")
print(coeff_quantized)
```

The normalized cutoff passed to `firwin` is relative to the Nyquist frequency when `fs` is not specified.

Therefore, a cutoff of `0.2` corresponds to 20% of the Nyquist frequency.

### Coefficient consistency

The coefficient values used in hardware must exactly match the coefficients used by the Python reference model.

The recommended workflow is:

1. Generate floating-point coefficients.
2. Quantize the coefficients.
3. Export the quantized coefficients.
4. Load or include the same values in the RTL.
5. Use those exact integers in the Python reference.
6. Compare the RTL and Python outputs.

Coefficient quantization can change the filter's frequency response. The resulting response should be evaluated before claiming particular passband or stopband performance.

---

## 9. Python Reference Verification

The Python reference model acts as a numerical specification for the RTL.

It calculates the expected fixed-point output for a sequence of input samples.

### Verification strategy

The initial test plan includes:

| Test | Purpose |
|---|---|
| Impulse input | Checks the filter's impulse response |
| Constant input | Checks steady-state behavior |
| Zero input | Checks the zero-input response |
| Positive input | Checks signed arithmetic |
| Negative input | Checks negative sample handling |
| Alternating samples | Exercises rapid sample changes |
| Sinusoidal input | Evaluates filtering behavior |
| Random input | Exercises a broad range of input combinations |
| Invalid input cycles | Checks sample-hold and validity behavior |
| Reset | Checks initialization and output validity |

### Comparison criteria

The verification script should compare every valid RTL output against the expected Python output.

For bit-exact verification:

\[
e[n] = y_{\text{RTL}}[n] - y_{\text{Python}}[n]
\]

The required result is:

\[
e[n] = 0
\]

for every compared output sample.

If the implementation intentionally uses a different rounding or saturation convention, the reference model must be updated to match that convention before comparison.

### Verification metrics

The test report should include:

- Number of input samples.
- Number of output samples.
- Number of mismatches.
- Maximum absolute error.
- Reset test status.
- Input-valid behavior.
- Overall pass/fail result.

A passing test suite provides evidence for the tested cases, not a formal proof of correctness for all possible inputs.

---

## 10. RTL Simulation with Verilator

Verilator is used to lint and simulate the RTL implementation.

The simulation flow should verify functional behavior before resource estimates are interpreted.

### Planned simulation stages

1. Parse and lint the RTL.
2. Resolve width, signedness, and structural warnings.
3. Compile the design and testbench.
4. Apply reset.
5. Drive input samples.
6. Capture outputs when `out_valid` is asserted.
7. Compare captured outputs with the Python reference.
8. Report mismatches.
9. Save test results.

### Required simulation properties

- No unexpected unknown or uninitialized values.
- Correct output-valid behavior.
- Correct sample ordering.
- Correct fixed-point scaling.
- Correct saturation behavior.
- No unintended state changes when input validity is low.
- Consistent reset behavior.

The exact simulation command depends on the chosen testbench structure and build configuration.

---

## 11. Synthesis with Yosys

Yosys is used to transform the RTL into a synthesized representation and report estimated resource usage.

### Initial synthesis objectives

- Check the RTL hierarchy.
- Process sequential and combinational logic.
- Perform logic optimization.
- Inspect the synthesized cell statistics.
- Compare alternative datapath architectures.

### Resource metrics

| Metric | Purpose |
|---|---|
| Flip-flops | Estimates sequential storage |
| Multipliers | Indicates multiplication hardware requirements |
| Adders | Indicates arithmetic logic requirements |
| Logic cells | Estimates combinational implementation cost |
| DSP blocks | Relevant to supported FPGA-specific flows |
| Area | Requires an appropriate target library or device flow |
| Timing | Requires suitable timing constraints and analysis |

Generic synthesis statistics are not equivalent to final FPGA utilization or ASIC physical area.

Actual physical metrics depend on the target technology, synthesis settings, clock constraints, and implementation flow.

---

## 12. Latency, Throughput, and Performance

Performance analysis is a major objective of this project.

### Latency

Latency is the time between accepting an input sample and producing its corresponding output.

\[
L = t_{\text{output}} - t_{\text{input}}
\]

In clocked hardware, it can be measured in cycles:

\[
L_{\text{time}} = \frac{L_{\text{cycles}}}{f_{\text{clk}}}
\]

For example, at 100 MHz, one clock cycle is 10 ns.

The actual filter latency must be measured from the implemented design.

### Initiation interval

The initiation interval (II) is the minimum number of clock cycles between consecutive accepted input samples under sustained operation.

An implementation with an initiation interval of one can accept a new sample every clock cycle.

### Throughput

The sustained throughput is:

\[
T = \frac{f_{\text{clk}}}{II}
\]

For a hypothetical 100 MHz implementation with an initiation interval of one:

\[
T = 100 \text{ million samples/second}
\]

This is an illustrative calculation, not a measured result or a guarantee that the design meets 100 MHz timing.

### Performance report

Record actual measurements in the following table:

| Metric | Result |
|---|---|
| Target clock frequency | To be specified |
| Achieved clock frequency | Not measured |
| Latency in cycles | Not measured |
| Latency in nanoseconds | Not measured |
| Initiation interval | Not measured |
| Sustained throughput | Not measured |
| Synthesized resource usage | Not measured |
| Verification status | Pending |

---

## 13. Hardware Optimization Roadmap

After the baseline implementation has been verified, several architectural variants can be investigated.

### A. Fully parallel FIR

A straightforward implementation computes all tap products in parallel.

**Advantages**

- High potential throughput.
- Simple sample scheduling.
- Suitable for deeply pipelined implementations.

**Trade-offs**

- Potentially high multiplier usage.
- Greater combinational logic.
- More challenging timing closure without adequate pipelining.

### B. Symmetric FIR

For symmetric coefficients:

\[
h[k] = h[N-1-k]
\]

The equation can be rearranged:

\[
y[n] =
\sum_{k=0}^{N/2-1}
h[k]\left(x[n-k]+x[n-(N-1-k)]\right)
\]

For an even-length symmetric FIR, this can reduce the number of multiplications by approximately half.

The pre-adders require sufficient width, and the coefficients must actually satisfy the symmetry condition.

### C. Resource-shared FIR

A resource-shared architecture reuses a smaller number of multipliers over multiple clock cycles.

**Advantages**

- Lower multiplier usage.
- Potentially lower area.
- Useful for area-constrained designs.

**Trade-offs**

- Increased latency.
- More complicated scheduling and control.
- Potentially lower throughput.

### D. Pipelined FIR

Registers can be inserted into the datapath to divide long combinational paths into shorter stages.

**Potential benefits**

- Higher achievable clock frequency.
- Improved timing closure.
- Better suitability for high-throughput implementations.

Pipelining increases latency unless carefully balanced with the existing architecture.

### E. Coefficient configurability

The design can be extended to support:

- Coefficient initialization files.
- Different tap counts.
- Different coefficient sets.
- Parameterized data widths.
- Alternative fixed-point formats.

Each configuration must be tested for arithmetic correctness and synthesis compatibility.

---

## 14. Repository Structure

The intended repository structure is:

```text
hardware-accelerated-fir-filter/
│
├── rtl/
│   └── fir_filter.sv
│
├── python/
│   ├── reference.py
│   └── generate_coefficients.py
│
├── tests/
│   ├── test_fir.py
│   └── testbench.sv
│
├── scripts/
│   └── run_all.sh
│
├── results/
│   ├── verification.txt
│   └── synthesis.txt
│
├── requirements.txt
├── README.md
├── LICENSE
└── .gitignore
```

The repository should contain the actual implementation, testbench, coefficient-generation code, and reproducible measurement scripts.

Results should be generated by executing the tools rather than entering estimated numbers manually.

---

## 15. Reproducibility

The project is intended to support reproducible development and testing.

A reproducible workflow should document:

- Operating system.
- Verilator version.
- Yosys version.
- Python version.
- NumPy version.
- SciPy version.
- Filter parameters.
- Coefficient values.
- Simulation inputs.
- Synthesis settings.
- Timing constraints.
- Verification results.

Randomized tests should use a recorded random seed.

Any published performance measurement should identify the target technology and relevant synthesis or timing-analysis settings.

---

## 16. Limitations

The initial version focuses on the digital filter datapath and software-based functional verification.

The following are not assumed to be completed:

- FPGA board deployment.
- ASIC physical design.
- Post-route timing analysis.
- Power estimation from a physical implementation.
- Formal verification.
- Measured maximum operating frequency.
- Verified PPA comparisons across architectures.

These capabilities can be added as the project develops.

---

## 17. Future Work

Potential extensions include:

- Automated coefficient generation and RTL integration.
- More extensive randomized verification.
- Bit-accurate fixed-point analysis.
- Symmetric FIR optimization.
- Multiplier-sharing architectures.
- Pipeline optimization.
- FPGA-specific synthesis and utilization analysis.
- Switching-activity-based power estimation.
- Formal property checking.
- Continuous integration for automated regression tests.
- Comparative PPA reports for multiple architectures.

---

## 18. Engineering Skills Demonstrated

This project is designed to dem