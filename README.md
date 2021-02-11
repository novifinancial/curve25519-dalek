
# curve25519-dalek-fiat

[![curve25519-dalek-fiat on crates.io](https://img.shields.io/crates/v/curve25519-dalek-fiat)](https://crates.io/crates/curve25519-dalek-fiat)
[![Documentation (latest release)](https://docs.rs/curve25519-dalek-fiat/badge.svg)](https://docs.rs/curve25519-dalek-fiat/)

<img
 width="33%"
 align="right"
 src="https://doc.dalek.rs/assets/dalek-logo-clear.png"/>

**A pure-Rust implementation of group operations on Ristretto and Curve25519.**

`curve25519-dalek` is a library providing group operations on the Edwards and
Montgomery forms of Curve25519, and on the prime-order Ristretto group.

`curve25519-dalek` is not intended to provide implementations of any particular
crypto protocol.  Rather, implementations of those protocols (such as
[`x25519-dalek`][x25519-dalek] and [`ed25519-dalek`][ed25519-dalek]) should use
`curve25519-dalek` as a library.

`curve25519-dalek` is intended to provide a clean and safe _mid-level_ API for use
implementing a wide range of ECC-based crypto protocols, such as key agreement,
signatures, anonymous credentials, rangeproofs, and zero-knowledge proof
systems.

In particular, `curve25519-dalek` implements Ristretto, which constructs a
prime-order group from a non-prime-order Edwards curve.  This provides the
speed and safety benefits of Edwards curve arithmetic, without the pitfalls of
cofactor-related abstraction mismatches.

# Use

To import `curve25519-dalek-fiat`, add the following to the dependencies section of
your project's `Cargo.toml`:
```toml
curve25519-dalek-fiat = "0.1.0"
```

See `CHANGELOG.md` for more details.

# Backends and Features

The `nightly` feature enables features available only when using a Rust nightly
compiler.  In particular, it is required for rendering documentation and for
the SIMD backends.

Curve arithmetic is implemented using one of the following backends:

* `fiat_u64_backend` to use a verified u64 backend supplied by the `fiat-crypto` crate
* a `u32` backend using serial formulas and `u64` products;
* a `u64` backend using serial formulas and `u128` products;
* an `avx2` backend using [parallel formulas][parallel_doc] and `avx2` instructions (sets speed records);
* an `ifma` backend using [parallel formulas][parallel_doc] and `ifma` instructions (sets speed records);

By default the `u64` backend is selected.  To select a specific backend, use:
```sh
cargo build --no-default-features --features "std u32_backend"
cargo build --no-default-features --features "std u64_backend"
# Requires nightly, RUSTFLAGS="-C target_feature=+avx2" to use avx2
cargo build --no-default-features --features "std simd_backend"
# Requires nightly, RUSTFLAGS="-C target_feature=+avx512ifma" to use ifma
cargo build --no-default-features --features "std simd_backend"
```
Crates using `curve25519-dalek` can either select a backend on behalf of their
users, or expose feature flags that control the `curve25519-dalek` backend.

The `std` feature is enabled by default, but it can be disabled for no-`std`
builds using `--no-default-features`.  Note that this requires explicitly
selecting an arithmetic backend using one of the `_backend` features.
If no backend is selected, compilation will fail.

# Safety

The `curve25519-dalek` types are designed to make illegal states
unrepresentable.  For example, any instance of an `EdwardsPoint` is
guaranteed to hold a point on the Edwards curve, and any instance of a
`RistrettoPoint` is guaranteed to hold a valid point in the Ristretto
group.

All operations are implemented using constant-time logic (no
secret-dependent branches, no secret-dependent memory accesses),
unless specifically marked as being variable-time code.
We believe that our constant-time logic is lowered to constant-time
assembly, at least on `x86_64` targets.

As an additional guard against possible future compiler optimizations,
the `subtle` crate places an optimization barrier before every
conditional move or assignment.  More details can be found in [the
documentation for the `subtle` crate][subtle_doc].

Some functionality (e.g., multiscalar multiplication or batch
inversion) requires heap allocation for temporary buffers.  All
heap-allocated buffers of potentially secret data are explicitly
zeroed before release.

However, we do not attempt to zero stack data, for two reasons.
First, it's not possible to do so correctly: we don't have control
over stack allocations, so there's no way to know how much data to
wipe.  Second, because `curve25519-dalek` provides a mid-level API,
the correct place to start zeroing stack data is likely not at the
entrypoints of `curve25519-dalek` functions, but at the entrypoints of
functions in other crates.

The implementation is memory-safe, and contains no significant
`unsafe` code.  The SIMD backend uses `unsafe` internally to call SIMD
intrinsics.  These are marked `unsafe` only because invoking them on an
inappropriate CPU would cause `SIGILL`, but the entire backend is only
compiled with appropriate `target_feature`s, so this cannot occur.

# Performance

Benchmarks are run using [`criterion.rs`][criterion]:

```sh
cargo bench --no-default-features --features "std u32_backend"
cargo bench --no-default-features --features "std u64_backend"
# Uses avx2 or ifma only if compiled for an appropriate target.
export RUSTFLAGS="-C target_cpu=native"
cargo bench --no-default-features --features "std simd_backend"
```

Performance is a secondary goal behind correctness, safety, and
clarity, but we aim to be competitive with other implementations.

# About

This is a fork of the [`curve25519-dalek`][curve25519-dalek] project, authored by
Isis Agora Lovecruft and Henry de Valence, in order to expose a formally
verified backed end supplied by the `fiat-crypto` project.

[curve25519-dalek]: https://github.com/dalek-cryptography/curve25519-dalek
[ed25519-dalek]: https://github.com/dalek-cryptography/ed25519-dalek
[x25519-dalek]: https://github.com/dalek-cryptography/x25519-dalek
[contributing]: https://github.com/dalek-cryptography/curve25519-dalek/blob/master/CONTRIBUTING.md
[docs-external]: https://doc.dalek.rs/curve25519_dalek/
[docs-internal]: https://doc-internal.dalek.rs/curve25519_dalek/
[criterion]: https://github.com/japaric/criterion.rs
[parallel_doc]: https://doc-internal.dalek.rs/curve25519_dalek/backend/vector/avx2/index.html
[subtle_doc]: https://doc.dalek.rs/subtle/
