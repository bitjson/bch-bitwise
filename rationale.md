# Rationale

This section documents design decisions made in this specification.

<details>

<summary><strong>Table of Contents</strong></summary>

- [Rationale](#rationale)
  - [Behavior of Arithmetic Right Shifts on Negative Values](#behavior-of-arithmetic-right-shifts-on-negative-values)
  - [Fixed-Length Result of Binary Shifts](#fixed-length-result-of-binary-shifts)
  - [Disallowance of Negative Shifts](#disallowance-of-negative-shifts)
  - [Equivalent Range Limit Across Shift Operations](#equivalent-range-limit-across-shift-operations)
  - [Calculation of Operation Costs](#calculation-of-operation-costs)

</details>

## Behavior of Arithmetic Right Shifts on Negative Values

This proposal implements arithmetic right shift matching typical Two's Complement right shift behavior, rounding negative results toward negative infinity. In addition to being the expected behavior in many common computing systems and algorithms, this approach may be marginally safer for financial mathematics: the shifted result for negative inputs is either more negative or equal to the true mathematical quotient, ensuring that fractional results produced by later computations do not result in small increases to positive values<sup>1</sup>. Finally, note that the common alternative behavior – rounding toward zero – is available via the `OP_DIV` operation.

1. E.g. payouts from a contract being rounded up might produce a deficit in the contract's treasury.

## Fixed-Length Result of Binary Shifts

This proposal implements binary shifts within the existing byte-length of the shifted stack item. This approach maximally-differentiates binary shifts from arithmetic shifts<sup>1</sup>, enabling improved efficiency in a wider variety of bit-manipulation and bit-inspection use cases.

1. In addition to enforcing numeric inputs and ensuring valid numeric outputs, arithmetic shifts may produce stack items of different byte lengths than their shifted inputs.

## Disallowance of Negative Shifts

This proposal specifically disallows negative shifts for `OP_LSHIFTNUM`, `OP_RSHIFTNUM`, `OP_LSHIFTBIN`, and `OP_RSHIFTBIN`: bit counts below `0` trigger an error (transaction rejection). This disallowance maintains consistency with both the [historical implementation](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/blob/c4319e678f693d5fbc49bd357ded1c8f951476e9/script.cpp#L595-605) and commonly-expected shift behaviors in most computing environments<sup>1</sup>, reducing the likelihood of simple contract errors and simplifying development. Additionally, because shift direction cannot be altered by the operation's input, some kinds of contract vulnerabilities are eliminated or made less exploitable.

Alternatively, this proposal could instead provide bidirectional abstractions, e.g. an "`OP_BINSHIFT`" and "`OP_NUMSHIFT`" which map negative counts to left shifts and positive counts to right shifts (or vice versa). Such an alternative could save two codepoints for future upgrades, and could also presumably allow for additional branch-free constructions – offering minor contract length optimizations<sup>2</sup> in relatively unusual cases<sup>3</sup>.

However, the bidirectional alternative (if not activated in addition to unidirectional opcodes) would disallow the implicit directionality guarantees and ensuing contract development and safety advantages of this proposal's unidirectional opcodes. Additionally, because `OP_0` through `OP_16` allow single-byte pushes of many common VM numbers for shift operations – and only `OP_1` has a symmetric negative opcode (`OP_1NEGATE`) – bidirectional shift opcodes would necessarily reduce the relative efficiency (by 1 byte) of one shift direction in common cases. The resulting asymmetry between each shift direction could also complicate compilers and resulting contracts, as optimal contracts (in terms of compiled length) may be biased toward the protocol-favored shift direction.

1. C, C++, JavaScript, Java, Python, Rust, x86, MIPS, ARM, RISC-V, WASM, etc. (some error, others mask or differently interpret the negative shift). A notable exception is Haskell's `Data.Bits`, in which `shift` interprets negative bit counts as right shifts. However, `shiftL` and `shiftR` are simultaneously supported.
2. 3+ bytes: eliminating `OP_IF [...] OP_ELSE [...] OP_ENDIF` and any additional stack manipulation operations required of the conditional-based alternative.
3. E.g. within a loop, a duplication and logical shift inspecting bits in some value from left-to-right-to-left, vice versa – rather than a typical, unidirectional inspection.

## Predictable Handling of Excessive Shift Operations

This proposal requires only that shift operations specify non-negative bit counts. All shifts greater than or equal to `0` are valid, with clear, consistent behavior: binary shifts continue to "clear" (zero-fill) bits up to and beyond the item's length, excessive right arithmetic shifts correctly result in `0` or `-1` (see [Behavior of Arithmetic Right Shifts on Negative Values](#behavior-of-arithmetic-right-shifts-on-negative-values)), and left shifts fail only for results which would be rejected due to stack item length limits (matching `OP_MUL`; implementations may optionally include a fast-failure path).

This proposal's behavior aligns with the existing numeric operations, maximizes the predictability of shift operations (by eliminating potentially-unexpected failures due to excessive shifts by otherwise-valid VM numbers), minimizes room for implementation-specific errors (e.g. the [`OP_RSHIFT` crash](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/c4319e678f693d5fbc49bd357ded1c8f951476e9) and [`CVE-2010-5137` `OP_LSHIFT` crash](https://nvd.nist.gov/vuln/detail/CVE-2010-5137)), and reduces overall protocol complexity.

## Calculation of Operation Costs

As both a simplifying and defensive measure, the Bitcoin Cash virtual machine (VM) currently limits all operations prior to execution via the limitation of pushed bytes. In effect, **every operation is limited as if it had linear time complexity**, reducing the risk of practically-exploitable performance issues in VM implementations. See [Limits CHIP Rationale: Limitation of Pushed Bytes](https://github.com/bitjson/bch-vm-limits/blob/master/rationale.md#limitation-of-pushed-bytes).

Given this comprehensive limitation approach, and because no new superlinear-time operations are added, no specialized cost accounting is required. The [Operation Cost](http://github.com/bitjson/bch-vm-limits#operation-cost-limit) of each proposed operation – `OP_INVERT`, `OP_LSHIFTNUM`, `OP_RSHIFTNUM`, `OP_LSHIFTBIN`, and `OP_RSHIFTBIN` – is the [Base Instruction Cost](https://github.com/bitjson/bch-vm-limits?#base-instruction-cost) plus the length of the pushed result (i.e. the potential cost of the next operation).
