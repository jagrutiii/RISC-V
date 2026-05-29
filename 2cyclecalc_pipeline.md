# Experiment: Two-Cycle Pipelined Calculator Using TL-Verilog

## Objective

To design and implement a two-cycle arithmetic calculator using TL-Verilog. The experiment focuses on distributing computations across multiple pipeline stages, controlling output validity, and improving timing performance through retiming.

---

## Overview

Modern digital systems often require high operating frequencies. As the complexity of arithmetic logic increases, completing all operations within a single clock cycle becomes challenging. A practical solution is to split the computation across multiple pipeline stages.

This calculator generates results after two cycles and uses a validity mechanism to ensure that outputs are produced only when meaningful data is available.

---

## Pipeline Architecture

The design is organized into three pipeline stages:

| Stage | Function |
|---------|---------|
| @0 | Reset and initialization |
| @1 | Operand processing and arithmetic operations |
| @2 | Result selection and output generation |

Separating tasks across stages reduces combinational delay and improves overall timing behavior.

---

## Initialization Stage

### Stage @0

The first stage handles reset management.

### Purpose

- Initializes the calculator state.
- Clears previously stored values.
- Ensures predictable startup behavior.
- Prevents undefined outputs during simulation.

The reset signal acts as the starting point for all subsequent pipeline activity.

---

## Alternate-Cycle Computation Control

A single-bit toggle register is used to generate the enable signal for computation.

### Operation

The bit alternates between logic 0 and logic 1 on every clock edge:

```text
0 → 1 → 0 → 1 → 0 ...
```

This pattern creates computation windows only on alternate cycles.

### Advantages

- Demonstrates multi-cycle execution.
- Controls when outputs are allowed to propagate.
- Mimics realistic timing constraints found in complex hardware.

---

## Feedback Alignment

The calculator reuses previously generated outputs as future inputs through a delayed feedback path.

### Concept

The output is routed back after a two-stage delay to maintain synchronization with the pipeline structure.

### Why Feedback Delay Is Needed

Since the final result is produced in stage `@2`, feedback data must be delayed by the same number of stages before reuse.

This ensures:

- Correct timing alignment
- Proper data dependency handling
- Stable pipeline behavior

---

## Arithmetic Computation Stage

### Supported Operations

The calculator performs four arithmetic functions:

- Addition
- Subtraction
- Multiplication
- Division

All arithmetic results are computed in the same pipeline stage and are later selected based on the operation code.

### TL-Verilog Benefits

TL-Verilog automatically manages:

- Pipeline registers
- Stage transitions
- Signal synchronization

This significantly reduces coding effort compared to conventional Verilog implementations.

---

## Output Generation Stage

### Result Selection

The final stage chooses one arithmetic result and forwards it to the output.

The operation selector determines whether the output corresponds to:

- Sum
- Difference
- Product
- Quotient

### Output Validation

The output is allowed only when the enable signal is active.

If:

- Reset is asserted, or
- The computation cycle is invalid,

the output is forced to zero.

This prevents invalid pipeline data from reaching later stages.

---

## Retiming Strategy

A key feature of this design is the movement of output-selection logic into a later pipeline stage.

### Purpose of Retiming

Retiming helps to:

- Reduce critical path delay
- Improve timing closure
- Support higher clock frequencies
- Enhance design scalability

Instead of manually inserting registers, TL-Verilog allows retiming simply by changing stage annotations.

---

## Execution Sequence

### Cycle 1

- Operands are generated.
- Arithmetic calculations begin.

### Cycle 2

- Result selection occurs.
- Valid output is produced.

### Cycle 3

- Feedback data returns to the computation stage.
- New calculations begin.

### Cycle 4

- Next valid output is generated.

The process then continues in a repeating two-cycle pattern.

---

## Concepts Demonstrated

This experiment illustrates:

- Multi-cycle arithmetic execution
- Pipeline stage synchronization
- Feedback path implementation
- Validity-controlled output generation
- Pipeline retiming techniques
- Timing optimization
- High-frequency datapath design

---

## Conclusion

A two-cycle pipelined calculator was successfully implemented using TL-Verilog. The design demonstrates how arithmetic operations can be distributed across multiple stages while maintaining correct timing relationships. By combining feedback synchronization, valid signal control, and retiming, the calculator achieves improved timing performance and serves as an effective example of pipeline-based digital design.
