# NTU Cryptography Engineering Final Project (2025 Fall) — ML-KEM & ECDH25519 on Cortex-M4

This repository contains my final project for **Cryptography Engineering** lectured by Matthias J. Kannwischer\
It focuses on performance optimization of

- **ECDH25519**
- **ML-KEM**

**Target Platform:** ARM **Cortex-M4**\
**Goals:** Reduce cycle counts under **constant-time** constraints and a **strict NTT code-size budget**.

**Baseline reference (Course Repo):** [ce2025](https://github.com/mkannwischer/ce2025)

---


## Project Overview

### X25519 (ECDH25519)

- Optimized finite-field arithmetic modulo $p = 2^{255} - 19$, focusing on hotspots:
  - `fe25519_mul` (field multiplication)
  - `fe25519_sqr` (field squaring)
- Optimized scalar multiplication with constant-time constraints:
  - no secret-dependent branches
  - no secret-dependent table indexing
- Fixed-base acceleration via a window method with precomputed table (constant-time selection)

### ML-KEM (FIPS 203)

- Optimized polynomial arithmetic modulo `q = 3329`, degree `n = 256`, in ring $Z_q[X]/(X^{256} + 1)$
- Focus areas:
  - NTT / inverse NTT kernels (reduced memory traffic, limited layer fusion, unrolling)
  - polynomial add/sub using packed 16-bit operations (ARMv7E-M DSP instructions)
- NTT routines are implemented under a **per-routine code-size budget** (see `doc/report`).

---

## Results Summary
### ECDH25519 (cycles)

| Operation | Baseline | Optimized | Reduction |
|---|---:|---:|---:|
| Fixed-base scalar mul | 54,552,132 | 3,141,710 | 94.24% |
| Variable-base scalar mul | 51,772,998 | 12,454,872 | 75.94% |
| fe25519_mul | 9,695,045 | 1,810,245 | 81.31% |
| fe25519_sqr | 9,695,078 | 1,431,045 | 85.24% |
| fe25519_invsqrt | 46,332,245 | 39,010,210 | 15.80% |

### ML-KEM (cycles)

| Operation | Baseline | Optimized | Reduction |
|---|---:|---:|---:|
| poly_ntt | 35,380 | 13,399 | 62.13% |
| Keypair generation | 1,325,464 | 697,377 | 47.39% |
| Encapsulation | 1,481,614 | 728,243 | 50.85% |
| Decapsulation | 1,741,181 | 826,535 | 52.53% |

### ML-KEM kernel speedups (baseline / optimized)

| Kernel | Baseline | Optimized | Speedup |
|---|---:|---:|---:|
| forward NTT | 23,424 | 8,224 | 2.85× |
| inverse NTT | 32,871 | 8,681 | 3.79× |
| poly_add | 2,056 | 1,045 | 1.97× |
| poly_sub | 2,313 | 1,045 | 2.21× |