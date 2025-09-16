# CHIP-2025-05 Bitwise: Re-Enable Bitwise Operations

        Title: Re-Enable Bitwise Operations
        Type: Standards
        Layer: Consensus
        Maintainer: Jason Dreyzehner
        Initial Publication Date: 2025-05-31
        Latest Revision Date: 2025-09-05
        Version: 1.1.1 (a432a19b)
        Status: Frozen for Lock-In

## Summary

This proposal re-enables bitwise operations, enabling Bitcoin Cash contracts to more efficiently implement a variety of financial and cryptographic applications.

## Deployment

Deployment of this specification is proposed for the May 2026 upgrade.

- Activation is proposed for `1763208000` MTP, (`2025-11-15T12:00:00.000Z`) on `chipnet`.
- Activation is proposed for `1778846400` MTP, (`2026-05-15T12:00:00.000Z`) on the BCH network (`mainnet`), `testnet3`, `testnet4`, and `scalenet`.

## Technical Specification

The virtual machine is modified to enable five bitwise opcodes: `OP_INVERT`, `OP_LSHIFTNUM`, `OP_RSHIFTNUM`, `OP_LSHIFTBIN`, and `OP_RSHIFTBIN`.

### Bit Inversion

#### `OP_INVERT`

```CashAssembly
<binary_data> OP_INVERT
```

The `OP_INVERT` opcode is defined at codepoint `0x83` (`131`) with the following behavior:

1. Pop the top item from the stack.<sup>1</sup>
2. Invert (bitwise "NOT") each byte of the item, then push the result to the stack.<sup>2</sup>

<small>

#### OP_INVERT Clarifications

1. If the stack is empty (no `binary_data`), error. Note that implementations may choose to operate on the top stack item in-place, but operation cost must be incremented as if the result were pushed again.
2. The [Operation Cost](http://github.com/bitjson/bch-vm-limits#operation-cost-limit) of `OP_INVERT` is the [Base Instruction Cost](https://github.com/bitjson/bch-vm-limits?#base-instruction-cost) plus the length of the pushed result (see [Rationale: Calculation of Operation Costs](./rationale.md#calculation-of-operation-costs)). Note that the length of the result is always equal to the length of the input.

</small>

### Arithmetic Shifts

#### `OP_LSHIFTNUM`

```CashAssembly
<valid_number> <bit_count> OP_LSHIFTNUM
```

The `OP_LSHIFTNUM` opcode is defined at codepoint `0x8d` (`141`) with the following behavior:

1. Pop the top item from the stack as a bit count (VM number).<sup>1</sup>
2. Pop the next item from the stack as the value (VM number) to shift.<sup>2</sup>
3. Perform an arithmetic left shift of the value by the bit count (`result = value * (2 ^ bit_count)`), then push the result to the stack.<sup>3</sup>

<small>

#### OP_LSHIFTNUM Clarifications

1. If the stack is empty (no `bit_count`), error. If the popped item is not a VM Number, error. If the bit count is negative, error. See [Rationale: Disallowance of Negative Shifts](./rationale.md#disallowance-of-negative-shifts) and [Rationale: Predictable Handling of Excessive Shift Operations](rationale.md#predictable-handling-of-excessive-shift-operations). Note that Maximum Stack Item Length may be increased by future upgrades. See [Notice of Possible Future Expansion](#notice-of-possible-future-expansion).
2. If the stack is empty (no `valid_number`), error. If the popped item is not a valid VM Number, error.
3. If the byte length of the result would exceed the [Maximum Stack Item Length](https://github.com/bitjson/bch-vm-limits#increased-stack-element-length-limit), error (implementations may optionally fast-fail such shifts). The [Operation Cost](http://github.com/bitjson/bch-vm-limits#operation-cost-limit) of `OP_LSHIFTNUM` is the [Base Instruction Cost](https://github.com/bitjson/bch-vm-limits?#base-instruction-cost) plus the length of the pushed result (see [Rationale: Calculation of Operation Costs](./rationale.md#calculation-of-operation-costs)). Note that the length of the output VM number may differ from the length of the input VM number.

</small>

#### `OP_RSHIFTNUM`

```CashAssembly
<valid_number> <bit_count> OP_RSHIFTNUM
```

The `OP_RSHIFTNUM` opcode is defined at codepoint `0x8e` (`142`) with the following behavior:

1. Pop the top item from the stack as a bit count (VM number).<sup>1</sup>
2. Pop the next item from the stack as the value (VM number) to shift.<sup>2</sup>
3. Perform an arithmetic right shift of the value by the bit count (`result = value / (2 ^ bit_count)`) rounding towards negative infinity<sup>3</sup>, then push the resulting VM number to the stack.<sup>4</sup>

<small>

#### OP_RSHIFTNUM Clarifications

1. If the stack is empty (no `bit_count`), error. If the popped item is not a VM Number, error. If the bit count is negative, error. See [Rationale: Disallowance of Negative Shifts](./rationale.md#disallowance-of-negative-shifts) and [Rationale: Predictable Handling of Excessive Shift Operations](rationale.md#predictable-handling-of-excessive-shift-operations). Note that Maximum Stack Item Length may be increased by future upgrades. See [Notice of Possible Future Expansion](#notice-of-possible-future-expansion).
2. If the stack is empty (no `valid_number`), error. If the popped item is not a valid VM Number, error.
3. The common behavior in Two's Complement right shifts (and the behavior standardized in C++20). See [Rationale: Behavior of Arithmetic Right Shifts on Negative Values](./rationale.md#behavior-of-arithmetic-right-shifts-on-negative-values). Implementations may optionally include a faster return path for excessive shifts (`0` for excessive shifts on positive values or `-1` for excessive shifts on negative values).
4. The [Operation Cost](http://github.com/bitjson/bch-vm-limits#operation-cost-limit) of `OP_RSHIFTNUM` is the [Base Instruction Cost](https://github.com/bitjson/bch-vm-limits?#base-instruction-cost) plus the length of the pushed result (see [Rationale: Calculation of Operation Costs](./rationale.md#calculation-of-operation-costs)). Note that the length of the output VM number may differ from the length of the input VM number.

</small>

### Binary (Logical) Shifts

#### `OP_LSHIFTBIN`

```CashAssembly
<binary_data> <bit_count> OP_LSHIFTBIN
```

The `OP_LSHIFTBIN` opcode is defined at codepoint `0x98` (`152`) with the following behavior:

1. Pop the top item from the stack as a bit count (VM number).<sup>1</sup>
2. Pop the next item from the stack as the binary data to shift.<sup>2</sup>
3. Perform a fixed-length, logical left shift of the data by the bit count, shifting-in `0` bits from the right and dropping shifted-out bits on the left<sup>3</sup>, then push the result to the stack.<sup>4</sup>

<small>

#### OP_LSHIFTBIN Clarifications

1. If the stack is empty (no `bit_count`), error. If the popped item is not a VM Number, error. If the bit count is negative, error. See [Rationale: Disallowance of Negative Shifts](./rationale.md#disallowance-of-negative-shifts) and [Rationale: Predictable Handling of Excessive Shift Operations](rationale.md#predictable-handling-of-excessive-shift-operations). Note that Maximum Stack Item Length may be increased by future upgrades. See [Notice of Possible Future Expansion](#notice-of-possible-future-expansion).
2. If the stack is empty (no `binary_data`), error. Note that any stack item is a valid input; zero-byte inputs produce a zero-byte output.
3. Implementations may optionally include a faster return path for excessive shifts (returning an appropriately-sized, zero-filled stack item).
4. The [Operation Cost](http://github.com/bitjson/bch-vm-limits#operation-cost-limit) of `OP_LSHIFTBIN` is the [Base Instruction Cost](https://github.com/bitjson/bch-vm-limits?#base-instruction-cost) plus the length of the pushed result (see [Rationale: Calculation of Operation Costs](./rationale.md#calculation-of-operation-costs)). Note that the length of the result is always equal to the length of the input. See [Rationale: Fixed-Length Result of Binary Shifts](./rationale.md#fixed-length-result-of-binary-shifts).

</small>

#### `OP_RSHIFTBIN`

```CashAssembly
<binary_data> <bit_count> OP_RSHIFTBIN
```

The `OP_RSHIFTBIN` opcode is defined at codepoint `0x99` (`153`) with the following behavior:

1. Pop the top item from the stack as a bit count (VM number).<sup>1</sup>
2. Pop the next item from the stack as the binary data to shift.<sup>2</sup>
3. Perform a fixed-length, logical right shift of the data by the bit count, shifting-in `0` bits from the left and dropping shifted-out bits on the right<sup>3</sup>, then push the result to the stack.<sup>4</sup>

<small>

#### OP_RSHIFTBIN Clarifications

1. If the stack is empty (no `bit_count`), error. If the popped item is not a VM Number, error. If the bit count is negative, error. See [Rationale: Disallowance of Negative Shifts](./rationale.md#disallowance-of-negative-shifts) and [Rationale: Predictable Handling of Excessive Shift Operations](rationale.md#predictable-handling-of-excessive-shift-operations). Note that Maximum Stack Item Length may be increased by future upgrades. See [Notice of Possible Future Expansion](#notice-of-possible-future-expansion).
2. If the stack is empty (no `binary_data`), error. Note that any stack item is a valid input; zero-byte inputs produce a zero-byte output.
3. Implementations may optionally include a faster return path for excessive shifts (returning an appropriately-sized, zero-filled stack item).
4. The [Operation Cost](http://github.com/bitjson/bch-vm-limits#operation-cost-limit) of `OP_RSHIFTBIN` is the [Base Instruction Cost](https://github.com/bitjson/bch-vm-limits?#base-instruction-cost) plus the length of the pushed result (see [Rationale: Calculation of Operation Costs](./rationale.md#calculation-of-operation-costs)). Note that the length of the result is always equal to the length of the input. See [Rationale: Fixed-Length Result of Binary Shifts](./rationale.md#fixed-length-result-of-binary-shifts).

</small>

### Notice of Possible Future Expansion

While unusual, it is possible to design pre-signed transactions, contract systems, and protocols which rely on the rejection of otherwise-valid transactions made invalid only by specifically exceeding one or more current VM limits. This proposal interprets such failure-reliant constructions as intentional – the constructions are designed to fail unless/until a possible future network upgrade in which such limits are increased, e.g. upgrade-activation futures contracts. Contract authors are advised that future upgrades may raise VM limits by increasing [Maximum Stack Item Length](https://github.com/bitjson/bch-vm-limits#increased-stack-element-length-limit) (A.K.A. `MAX_SCRIPT_ELEMENT_SIZE`), maximum bytecode length (A.K.A. `MAX_SCRIPT_SIZE`), or otherwise. See [Limits CHIP Rationale: Inclusion of "Notice of Possible Future Expansion"](https://github.com/bitjson/bch-vm-limits/blob/master/rationale.md#inclusion-of-notice-of-possible-future-expansion).

## Rationale

- [Appendix: Rationale &rarr;](rationale.md#rationale)
  - [Behavior of Arithmetic Right Shifts on Negative Values](rationale.md#behavior-of-arithmetic-right-shifts-on-negative-values)
  - [Fixed-Length Result of Binary Shifts](rationale.md#fixed-length-result-of-binary-shifts)
  - [Disallowance of Negative Shifts](rationale.md#disallowance-of-negative-shifts)
  - [Predictable Handling of Excessive Shift Operations](rationale.md#predictable-handling-of-excessive-shift-operations)
  - [Calculation of Operation Costs](rationale.md#calculation-of-operation-costs)

## Evaluations of Alternatives

- [Appendix: Evaluations of Alternatives &rarr;](alternatives.md#evaluation-of-alternatives)
  - [Arithmetic Shifts](./alternatives.md#arithmetic-shifts)
    - [Alternative: Negative Rounding Toward Zero](./alternatives.md#alternative-negative-rounding-toward-zero)
  - [Binary (Logical) Shifts](./alternatives.md#binary-logical-shifts)
    - [Alternative: Variable-Length Results](./alternatives.md#alternative-variable-length-results)

## Risk Assessment

- [Appendix: Risk Assessment &rarr;](risk-assessment.md#risk-assessment)

## Test Vectors

This proposal includes [a suite of functional tests and benchmarks](./vmb_tests/) to verify the performance of all operations within virtual machine implementations.

## Implementations

Please see the following implementations for examples and additional test vectors:

- C++:
  - [Bitcoin Cash Node (BCHN)](https://bitcoincashnode.org/) – A professional, miner-friendly node that solves practical problems for Bitcoin Cash. [Merge Request !1962](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/merge_requests/1962/).
- **JavaScript/TypeScript**:
  - [Libauth](https://github.com/bitauth/libauth) – An ultra-lightweight, zero-dependency JavaScript library for Bitcoin Cash. [Branch `next`](https://github.com/bitauth/libauth/tree/next).
  - [Bitauth IDE](https://github.com/bitauth/bitauth-ide) – An online IDE for bitcoin (cash) contracts. [Branch `next`](https://github.com/bitauth/bitauth-ide/tree/next).

## Feedback & Reviews

- [Bitwise CHIP Issues](https://github.com/bitjson/bch-bitwise/issues)
- [`CHIP 2025-05 Bitwise: Re-Enable Bitwise Operations` - Bitcoin Cash Research](https://bitcoincashresearch.org/t/chip-2025-05-bitwise-re-enable-bitwise-operations/1580)

## Acknowledgements

Thank you to the following contributors for reviewing and contributing improvements to this proposal, providing feedback, and promoting consensus among stakeholders:
[Calin Culianu](https://github.com/cculianu)

## Changelog

This section summarizes the evolution of this proposal.

- **v1.1.1 – 2025-09-05** ([`a432a19b`](https://github.com/bitjson/bch-bitwise/commit/a432a19bcbc597391d0f0a29ae62a902b575d1dd) – [diff vs. `master`](https://github.com/bitjson/bch-bitwise/compare/a432a19bcbc597391d0f0a29ae62a902b575d1dd...master))
  - Update VMB tests and benchmarks
- **v1.1.0 – 2025-06-09** ([`1c954588`](https://github.com/bitjson/bch-bitwise/commit/1c954588ab971f5848844f44485fafe6d31bef56))
  - More predictable handling of excessive shifts ([#2](https://github.com/bitjson/bch-bitwise/pull/2))
- **v1.0.0 – 2025-05-31** ([`4fab9dfe`](https://github.com/bitjson/bch-bitwise/commit/4fab9dfe83f28b03211a38c850f2ca034eed700c))
  - Initial publication

## Copyright

This document is placed in the public domain.
