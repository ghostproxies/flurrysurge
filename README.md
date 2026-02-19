# PulseField™

[![PulseField™](https://repository-images.githubusercontent.com/1141499400/0d3ae64a-3add-467b-9471-bc1a8ddea02b)](https://github.com/ghostproxies/pulsefield)

## Table of Contents

- [Introduction](README.md?tab=readme-ov-file#introduction)
- [Author](README.md?tab=readme-ov-file#author)
- [License](README.md?tab=readme-ov-file#license)
- [Period](README.md?tab=readme-ov-file#period)
- [Parallelism](README.md?tab=readme-ov-file#parallelism)
- [Jumps](README.md?tab=readme-ov-file#jumps)
- [Reference](README.md?tab=readme-ov-file#reference)
- [Empirical](README.md?tab=readme-ov-file#empirical)
- [Enterprise](README.md?tab=readme-ov-file#enterprise)

## Introduction

PulseField™ is the segmentation algorithm for parallel PRNG sequences.

It enables [independent parallel sequences](README.md?tab=readme-ov-file#parallelism) for efficient PRNGs such as [BlastCircuit™](https://github.com/ghostproxies/blastcircuit).

Furthermore, it has a [minimum period](README.md?tab=readme-ov-file#period), [decent mixing](README.md?tab=readme-ov-file#empirical), reversibility, [simple implementation](README.md?tab=readme-ov-file#reference) and no vendor-specific SIMD.

## Author

PulseField™ was created by [William Stafford Parsons](https://github.com/williamstaffordparsons) as a product of [GhostProxies](https://ghostproxies.com).

## License

PulseField™ is licensed with the [BSD-3-Clause](LICENSE) license.

## Period

PulseField™ uses a Weyl sequence to generate deterministic sequences that each have a period of at least 2ⁿ (ⁿ is defined as `BITS_LENGTH` in the [reference](README.md?tab=readme-ov-file#reference)).

## Parallelism

PulseField™ can generate up to 2ⁿ (ⁿ is defined as `BITS_LENGTH` in the [reference](README.md?tab=readme-ov-file#reference)) parallel instances that each avoid full state collisions (for at least 2ⁿ output results) among the set of parallel instances.

[pulsefield.c](pulsefield.c) demonstrates independent parallel sequences when `BITS_LENGTH` is `8` and the `stdint.h` header defines an 8-bit, unsigned integral type for `uint8_t`.

Small overlapping sequences of `a` escape quickly due to the [seed requirements](README.md?tab=readme-ov-file#reference) aligning to ensure occasional XOR rotation overlaps are XORed with the [Weyl sequence](README.md?tab=readme-ov-file#period) at non-overlapping points.

## Jumps

To prevent full state collisions among parallel sequences, PulseField™ uses efficient [seed conditions](README.md?tab=readme-ov-file#reference) instead of jump functions.

Jumping an arbitrary number of steps is redundant in the remaining practical use cases that require full sequence reproducibility.

## Reference

The following pseudocode block demonstrates the PulseField™ algorithm structure with operations that are simple to port to a majority of programming languages.

``` c
a = ROTATE_LEFT(a, SHIFT) ^ b;
b += INCREMENT;
```

Both `a` and `b` must have a matching bits length of `BITS_LENGTH` (or `ⁿ`) as a power of 2 that's a supported data type length greater than or equal to `8`.

Each instance within a set of parallel PRNG instances that use PulseField™ must seed `a` with a number that's unique among the set of parallel instances and must seed `b` with a number that's consistent among the set of parallel instances.

For example, the following PulseField™ seed values are valid for a set of 3 parallel PRNG instances.

```
a: 0
b: 0

a: 1
b: 0

a: 2
b: 0
```

`SHIFT` must be a number greater than `0` and less than `BITS_LENGTH`.

`INCREMENT` must be an odd number greater than `0` and less than `2ⁿ`.

Neither `a` nor `b` must be assigned a value outside of the aforementioned PulseField™ algorithm.

`a` must either be mixed into additional state (excluding `b`) or returned.

`b` must be a Weyl sequence that wraps around `BITS_LENGTH` bits (as demonstrated in [pulsefield.c](pulsefield.c)).

## Empirical

Although PulseField™ isn't intended to replace non-cryptographic PRNGs, it adds decent mixing that can pass some empirical tests in specific scenarios.

For example, the following variant of PulseField™ (implemented in C as a standalone, non-cryptographic PRNG for demonstrative purposes) passed TestU01 1.2.3 SmallCrush when each state variable was seeded with `0` and the `stdint.h` header defined a 64-bit, unsigned integral type for `uint64_t`.

``` c
#include <stdint.h>

struct pulsefield_state {
  uint64_t a;
  uint64_t b;
};

uint32_t pulsefield(struct pulsefield_state *s) {
  s->a = ((s->a << 29) | (s->a >> 35)) ^ s->b;
  s->b += 1111111111111111;
  return s->a;
}
```

## Enterprise

GhostProxies provides the following enterprise solutions for RNG system efficiency.

- [Nightmare℠](https://ghostproxies.com/nightmare) is the RNG system efficiency enhancement service.
- [Tome™](https://ghostproxies.com/tome) is a multi-level SLA for GhostProxies products.
- [Ectoplasm™](https://ghostproxies.com/ectoplasm) is the entropy schema that generates bits efficiently with the highest level of unpredictability.
