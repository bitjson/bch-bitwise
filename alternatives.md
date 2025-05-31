# Evaluation of Alternatives

Potential alternatives to this proposal are reviewed below.

<details>

<summary><strong>Table of Contents</strong></summary>

- [Evaluation of Alternatives](#evaluation-of-alternatives)
  - [Arithmetic Shifts](#arithmetic-shifts)
    - [Alternative: Negative Rounding Toward Zero](#alternative-negative-rounding-toward-zero)
  - [Binary (Logical) Shifts](#binary-logical-shifts)
    - [Alternative: Variable-Length Results](#alternative-variable-length-results)

</details>

## Arithmetic Shifts

### Alternative: Negative Rounding Toward Zero

This proposal implements arithmetic right shift matching typical Two's Complement right shift behavior, rounding negative results toward negative infinity. This approach is typical of many common computing environments and algorithms – and may be marginally safer for financial mathematics – than rounding toward zero. See [Rationale: Behavior of Arithmetic Right Shifts on Negative Values](./rationale.md#behavior-of-arithmetic-right-shifts-on-negative-values).

## Binary (Logical) Shifts

### Alternative: Variable-Length Results

This proposal implements binary shifts within the existing byte-length of the shifted stack item, rather than an alternative which attempts to preserve all shifted bits. See [Rationale: Fixed-Length Result of Binary Shifts](./rationale.md#fixed-length-result-of-binary-shifts).
