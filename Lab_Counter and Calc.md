# Counter and Calculator Using TL-Verilog Pipeline

## Introduction

Modern digital systems frequently combine arithmetic processing with sequential logic. In this experiment, a calculator and an incrementing counter are implemented within a TL-Verilog pipeline. The design performs arithmetic operations on dynamically generated operands while utilizing feedback paths to create sequential behavior.

---

## Architecture Overview

![Counter and Calculator Pipeline](counter_calculator_pipeline.png)

*Figure 1: Pipeline containing a counter, arithmetic datapath, operation selector, and feedback loop.*

The design consists of:

- Reset circuitry
- 32-bit counter
- Arithmetic processing unit
- Output selection multiplexer
- Feedback path

All arithmetic operations are executed in the same pipeline stage and the selected result is reused in future computations through feedback.

---

## Design Implementation

### Reset Initialization

The reset signal establishes a known starting state for the pipeline.

```tlv
@0
   $rst = *reset;
```

When reset is active, the counter and output values are cleared.

---

### Counter Generation

```tlv
$count[31:0] = $rst ? 32'd0 : >>1$count + 32'd1;
```

A 32-bit sequential counter is implemented using feedback.

The operator:

```tlv
>>1$count
```

retrieves the value of the counter from the previous clock cycle.

As a result, the counter increments continuously:

```text
0 → 1 → 2 → 3 → 4 → ...
```

The counter also acts as a source for operation selection.

---

### Operand Preparation

```tlv
$in1[31:0] = >>1$result;
$in2[31:0] = $rand1[4:1];
```

Two operands are prepared for arithmetic processing.

The first operand is obtained from the previous output value, creating a feedback loop.

The second operand is generated from a random source, ensuring varying input data throughout simulation.

---

### Operation Selection Logic

```tlv
$sel[1:0] = $count[1:0];
```

The least significant bits of the counter are used to determine the active arithmetic operation.

| Select Value | Function |
|-------------|----------|
| 00 | Addition |
| 01 | Subtraction |
| 10 | Multiplication |
| 11 | Division |

This approach automatically cycles through all operations as the counter increments.

---

### Arithmetic Processing Unit

The datapath computes four arithmetic results in parallel.

#### Addition

```tlv
$add_res = $in1 + $in2;
```

Produces the sum of the operands.

#### Subtraction

```tlv
$sub_res = $in1 - $in2;
```

Produces the difference between operands.

#### Multiplication

```tlv
$mul_res = $in1 * $in2;
```

Produces the product of the operands.

#### Division

```tlv
$div_res = ($in2 == 0) ? 32'd0 : ($in1 / $in2);
```

Produces the quotient while preventing divide-by-zero conditions.

---

### Output Selection

```tlv
$result[31:0] =
   $rst            ? 32'd0 :
   ($sel == 2'b00) ? $add_res :
   ($sel == 2'b01) ? $sub_res :
   ($sel == 2'b10) ? $mul_res :
                     $div_res;
```

The multiplexer forwards one arithmetic result to the output based on the current select signal.

Only one operation appears at the output at a given time, despite all arithmetic units being evaluated simultaneously.

---

## Feedback Mechanism

A key feature of this design is the feedback connection:

```tlv
$in1[31:0] = >>1$result;
```

The current computation depends on the previous output value.

This introduces sequential behavior and demonstrates how TL-Verilog handles pipeline feedback without manually inserting registers.

---

## Operational Flow

```text
Counter Updates
       ↓
Operation Selected
       ↓
Operands Generated
       ↓
Arithmetic Units Execute
       ↓
MUX Chooses Result
       ↓
Output Generated
       ↓
Output Fed Back
       ↓
Next Computation
```

---

## Advantages of TL-Verilog in This Design

- Simplified pipeline implementation
- Automatic stage synchronization
- Easy feedback path creation
- Reduced register management
- Clear timing abstraction
- Improved code readability

---

## Conclusion

A counter-driven arithmetic calculator was successfully implemented using TL-Verilog. The design combines sequential logic, arithmetic computation, operation selection, and feedback mechanisms within a pipelined architecture. The experiment demonstrates how TL-Verilog simplifies the construction of complex digital systems while maintaining clear timing and pipeline behavior.
