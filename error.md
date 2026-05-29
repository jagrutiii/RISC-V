# Experiment: Pipeline Error Aggregation using TL-Verilog

## Identifier Naming Rules in TL-Verilog

In TL-Verilog, the type of an identifier is determined by its **symbol prefix**, **letter case**, and **delimiter style**.

### Identifier Structure

Example:

```tlv
$pipe_signal
```

- `$` → Symbol prefix
- `pipe` and `signal` → Tokens
- `_` → Delimiter

### Delimitation Styles

The first token must begin with two alphabetic characters.

#### Pipe Signals

```tlv
$lower_case
```

Uses lowercase letters and underscores.

#### State Signals

```tlv
$CamelCase
```

Uses Pascal/Camel case notation.

#### Keyword Signals

```tlv
$UPPER_CASE
```

Uses uppercase letters and underscores.

### Numeric Identifier Rules

Numbers can appear only after alphabetic characters.

Valid:

```tlv
$base64_value
```

Invalid:

```tlv
$bad_name_5
```

### Numeric Identifiers

```tlv
>>1
```

Represents a signal delayed by one pipeline stage.

---

# TL-Verilog Code

```tlv
\m4_TLV_version 1d: tl-x.org

\SV
   m4_makerchip_module

\TLV
   |comp

      @1
         $bad_input  = 1'b0;
         $illegal_op = 1'b1;

         $err1 = ($bad_input || $illegal_op) ? 1 : 0;

      @3
         $overflow = 1'b0;

         $err2 = ($err1 || $overflow) ? 1 : 0;

      @6
         $divby0 = 1'b1;

         $err3 = ($err2 || $divby0) ? 1 : 0;

   *passed = *cyc_cnt > 40;
   *failed = 1'b0;

\SV
   endmodule
```

---

# Objective

The objective of this experiment is to demonstrate error detection and aggregation in a pipelined TL-Verilog design. Multiple error conditions generated at different pipeline stages are combined into a single error indication signal using logical OR operations.

---

# Error Conditions

The design uses four error conditions:

## 1. Bad Input Error (`$bad_input`)

This signal indicates that invalid or unexpected input data has been supplied to the system.

```tlv
$bad_input = 1'b0;
```

A value of `0` indicates that no bad input error has occurred.

---

## 2. Illegal Operation Error (`$illegal_op`)

This signal indicates that an unsupported or invalid operation has been requested.

```tlv
$illegal_op = 1'b1;
```

A value of `1` indicates that an illegal operation has been detected.

---

## 3. Overflow Error (`$overflow`)

Overflow occurs when the result of an arithmetic operation exceeds the range representable by the available hardware bits.

```tlv
$overflow = 1'b0;
```

A value of `0` indicates that no overflow has occurred.

---

## 4. Divide-by-Zero Error (`$divby0`)

This signal becomes active when an attempt is made to divide a number by zero.

```tlv
$divby0 = 1'b1;
```

A value of `1` indicates that a divide-by-zero condition has occurred.

---

# Error Aggregation

The error conditions are combined progressively through the pipeline.

## Stage @1

```tlv
$err1 = ($bad_input || $illegal_op) ? 1 : 0;
```

This OR operation combines:

- `$bad_input`
- `$illegal_op`

Since:

```text
0 || 1 = 1
```

Therefore:

```text
err1 = 1
```

---

## Stage @3

```tlv
$err2 = ($err1 || $overflow) ? 1 : 0;
```

This combines:

- Previous error signal `$err1`
- Overflow error `$overflow`

Since:

```text
1 || 0 = 1
```

Therefore:

```text
err2 = 1
```

---

## Stage @6

```tlv
$err3 = ($err2 || $divby0) ? 1 : 0;
```

This combines:

- Previous aggregated error `$err2`
- Divide-by-zero error `$divby0`

Since:

```text
1 || 1 = 1
```

Therefore:

```text
err3 = 1
```

---

# Pipeline Flow

```text
@1                         @3                         @6

bad_input -----\
                \
                 OR --> err1 ----\
illegal_op ----/                  \
                                   OR --> err2 ----\
overflow -------------------------/                 \
                                                     OR --> err3
divby0 ---------------------------------------------/
```

---

# Result

The four independent error conditions (`bad_input`, `illegal_op`, `overflow`, and `divby0`) are successfully aggregated into a single final error signal (`err3`). This approach simplifies error monitoring in pipelined designs by providing one consolidated error indication while preserving the ability to detect faults originating from multiple pipeline stages.
