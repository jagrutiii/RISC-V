# Experiment: Two-Cycle Pipelined Calculator Using TL-Verilog

## Objective

To design and implement a two-cycle arithmetic calculator using TL-Verilog. The experiment focuses on distributing computations across multiple pipeline stages, controlling output validity, and improving timing performance through retiming.

---

## Theory

As clock frequencies increase, performing all arithmetic operations within a single cycle becomes increasingly difficult due to timing limitations. Pipelining helps overcome this issue by dividing the computation into multiple stages and allowing data to move progressively through the design.

In this experiment, the calculator output is generated after two cycles. A validity signal controls when outputs are accepted, while a delayed feedback path ensures proper synchronization between pipeline stages. The design also demonstrates retiming, which helps improve timing performance without changing functionality.

---

## Code

```tl-verilog
\m4_TLV_version 1d: tl-x.org
\SV


   m4_include_lib(['https://raw.githubusercontent.com/stevehoover/RISC-V_MYTH_Workshop/ecba3769fff373ef6b8f66b3347e8940c859792d/tlv_lib/calculator_shell_lib.tlv'])

\SV
   m4_makerchip_module

\TLV
   |calc

      @0
         $reset = *reset;

      @1
         // Alternate-cycle valid generation
         $toggle[0:0] = $reset ? 1'b0 : ~>>1$toggle;
         $enable = $toggle;

         // Feedback path from output delayed by 2 cycles
         $feedback[31:0] = >>2$out[31:0];

         // Random operand and operation selection
         $operand[31:0] = $rand2[3:0];
         $opcode[1:0]   = $rand1[1:0];

         // Arithmetic operations
         $add_res[31:0] = $feedback + $operand;
         $sub_res[31:0] = $feedback - $operand;
         $mul_res[31:0] = $feedback * $operand;

         $div_res[31:0] =
            ($operand == 0) ? 32'b0 :
                              ($feedback / $operand);

      @2
         // Retimed output multiplexer
         $out[31:0] =
            ($reset || !$enable) ? 32'b0 :
            ($opcode == 2'b00)  ? $add_res :
            ($opcode == 2'b01)  ? $sub_res :
            ($opcode == 2'b10)  ? $mul_res :
                                  $div_res;

   *passed = *cyc_cnt > 40;
   *failed = 1'b0;

\SV
   endmodule
```

---

## Design Flow

### Stage @0 – Reset Initialization

The first pipeline stage initializes the reset signal and prepares the calculator for operation. During reset, all internal states are cleared, ensuring predictable behavior when execution begins.

### Stage @1 – Computation and Control

The second stage performs most of the processing.

#### Toggle-Based Enable Signal

A single-bit toggle register changes state every clock cycle. This signal acts as an enable mechanism, allowing computations to proceed only during alternate cycles.

#### Feedback Mechanism

The calculator reuses previously generated outputs through a two-cycle delayed feedback path. Since the output is produced in stage `@2`, the feedback must also be delayed by two cycles to maintain proper timing alignment.

#### Arithmetic Processing

Four arithmetic results are generated:

- Addition
- Subtraction
- Multiplication
- Division

These calculations are performed in parallel and become available for selection in the next stage.

---

## Stage @2 – Output Selection

The final pipeline stage selects one arithmetic result based on the operation code.

The output multiplexer determines whether the calculator returns:

- Sum
- Difference
- Product
- Quotient

If reset is active or the enable signal is low, the output is forced to zero. This prevents invalid data from propagating through the pipeline.

---

## Retiming Implementation

The output multiplexer is intentionally placed in stage `@2` instead of the arithmetic stage.

This technique is known as **retiming**.

### Benefits of Retiming

- Reduces combinational logic delay
- Improves timing closure
- Supports higher clock frequencies
- Enhances pipeline efficiency

TL-Verilog simplifies retiming by allowing logic movement through stage annotations rather than manual register insertion.

---

## Pipeline Operation

### Cycle 1
- Operands are generated.
- Arithmetic calculations are performed.

### Cycle 2
- Result selection occurs.
- Valid output is generated.

### Cycle 3
- Delayed feedback returns to the computation stage.
- New calculations begin.

### Cycle 4
- Another valid output is produced.

This sequence continues repeatedly, creating a two-cycle computation pipeline.

---

## Features Demonstrated

- Two-cycle pipelined execution
- Alternate-cycle result generation
- Feedback synchronization using `>>2`
- Validity-controlled outputs
- Arithmetic datapath implementation
- Retiming for timing optimization
- TL-Verilog pipeline abstraction

---

## Conclusion

A two-cycle pipelined calculator was successfully implemented using TL-Verilog. The design utilizes a delayed feedback path, alternate-cycle enable logic, and retimed output selection to improve timing performance. The experiment demonstrates how TL-Verilog simplifies the implementation of multi-stage pipelines while maintaining readability and scalability.
