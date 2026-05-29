# Experiment: Calculator with Single-Value Memory

## Objective

To design and implement a pipelined calculator with a single-value memory using TL-Verilog. The calculator performs arithmetic operations, stores results in memory, and supports memory recall through a synchronized feedback path.

---

## Introduction

Many calculators provide memory functionality that allows users to save intermediate results and retrieve them later. This capability eliminates the need to repeatedly perform the same calculations.

In this experiment, a memory register is added to the calculator datapath. The stored value can be recalled when required, while normal arithmetic operations continue to update the memory. The design demonstrates how storage elements can be integrated into a pipelined architecture.

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

         // Arithmetic unit
         $alu_add[31:0] = $val1 + $val2;
         $alu_sub[31:0] = $val1 - $val2;
         $alu_mul[31:0] = $val1 * $val2;

         $alu_div[31:0] =
            ($val2 != 0) ?
            ($val1 / $val2) :
            32'b0;

      @1
         // Recall operation detection
         $recall_mode = ($op == 3'b100);

         // ALU result selection
         $alu_result[31:0] =
            ($op == 3'b000) ? $alu_add :
            ($op == 3'b001) ? $alu_sub :
            ($op == 3'b010) ? $alu_mul :
            ($op == 3'b011) ? $alu_div :
                              32'b0;

      @2
         // Memory register
         $store_reg[31:0] =
            $reset ? 32'b0 :
            $recall_mode ? >>1$store_reg :
                           >>1$alu_result;

         // Final output
         $out[31:0] =
            $recall_mode ? >>2$store_reg :
                           $alu_result;

   *passed = *cyc_cnt > 80;
   *failed = 1'b0;

\SV
   endmodule
```

---

## Pipeline Structure

| Stage | Purpose |
|---------|---------|
| @0 | Arithmetic computation |
| @1 | Operation selection |
| @2 | Memory storage and recall |

---

## Stage @0 – Arithmetic Unit

The first pipeline stage performs all arithmetic calculations. Addition, subtraction, multiplication, and division are evaluated in parallel, ensuring that the required result is available for selection in the following stage.

A divide-by-zero check is included to prevent invalid arithmetic operations. If the divisor is zero, the division result is forced to zero.

---

## Stage @1 – Operation Control

This stage determines the action requested by the operation code.

A dedicated signal called `$recall_mode` identifies whether the calculator should perform a memory recall operation. During normal operation, one of the arithmetic results is selected and forwarded through the pipeline.

The selected result is stored in `$alu_result`, which acts as a common datapath for all arithmetic operations.

---

## Stage @2 – Memory Storage and Recall

The final stage introduces a single-value memory register.

### Memory Write

During arithmetic operations, the latest calculator result is stored in the memory register. This allows the value to be reused later without recalculation.

### Memory Hold

When a recall request occurs, the memory register retains its current value instead of being overwritten.

### Memory Recall

If recall mode is active, the stored value is retrieved through a delayed feedback path and sent to the output. The delay ensures proper synchronization between pipeline stages.

---

## Sequential Storage Behavior

Unlike a purely combinational calculator, this design contains memory. The stored value persists across clock cycles and can be accessed later when needed.

This introduces the concept of state retention, which is fundamental in processor register files, accumulators, and other sequential digital systems.

---

## Features Demonstrated

- Pipelined arithmetic execution
- Single-value memory implementation
- Memory store and recall operations
- Sequential feedback paths
- State retention
- Divide-by-zero protection
- Pipeline synchronization

---

## Conclusion

A calculator with single-value memory was successfully implemented using TL-Verilog. The design extends a standard arithmetic calculator by introducing storage and recall functionality while maintaining proper pipeline timing. The experiment demonstrates how memory elements can be integrated into a pipelined datapath to support sequential computation and result reuse.
