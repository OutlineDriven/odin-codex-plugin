---
description: Execute HODD-RUST validation pipeline for Rust projects
argument-hint: [PATH=<directory>]
---

You are executing the HODD-RUST (Stronger Outline Driven Development For Rust) validation pipeline.

## Arguments

- `$PATH` - Directory containing Rust project with Cargo.toml (required)

HODD-RUST merges: Type-driven + Spec-first + Proof-driven + Design-by-contracts + Test-driven (XP)

## Execution Steps

1. **CHECK**: Verify Rust toolchain
2. **BASIC**: rustfmt, clippy, cargo-audit, cargo-deny
3. **TEST**: cargo test with coverage
4. **CONTRACT**: Prusti verification (if annotations)
5. **SPEC**: Kani/Flux verification (if proofs)
6. **CONCURRENCY**: Loom tests (if present)
7. **RUNTIME**: Miri (advisory: local debugging only)
8. **EXTERNAL**: Idris2/Lean4/Quint (if specs exist)

## Commands (Tiered)

### Basic (Precondition Check)
```bash
cd $PATH

# Verify toolchain
command -v rustc >/dev/null || exit 11
command -v cargo >/dev/null || exit 11
test -f Cargo.toml || exit 12

# Format and lint
cargo fmt --check || exit 12
cargo clippy -- -D warnings || exit 13
```

### Intermediate (Security & Tests)
```bash
cd $PATH

# Security
command -v cargo-audit && cargo audit || exit 14
command -v cargo-deny && cargo deny check || exit 14

# Tests
cargo test || exit 13

# Coverage
command -v cargo-tarpaulin && cargo tarpaulin --out Html
```

### Advanced (Formal Verification)
```bash
cd $PATH

# Prusti contracts
rg '#\[(requires|ensures|invariant)(\(|])' -q -t rust && {
  command -v cargo-prusti && cargo prusti || exit 15
}

# Kani proofs
rg '#\[kani::proof\]' -q -t rust && {
  command -v cargo-kani && cargo kani || exit 15
}

# Flux refinements
rg '#\[flux::' -q -t rust && {
  command -v flux && flux check || exit 15
}

# Loom tests
rg 'loom::' -q -t rust && {
  cargo test --features loom || exit 15
}

# Miri (advisory: local debugging only, NOT for CI)
# cargo +nightly miri test || exit 15
```

### External Tools (Optional)
```bash
# Idris2
fd -e idr proofs/ 2>/dev/null | head -1 | grep -q . && {
  idris2 --check proofs/*.idr || exit 16
}

# Lean4
fd lakefile.lean proofs/ 2>/dev/null | head -1 | grep -q . && {
  (cd proofs && lake build) || exit 16
  rg 'sorry' proofs/*.lean && exit 16
}

# Quint
fd -e qnt specs/ 2>/dev/null | head -1 | grep -q . && {
  quint typecheck specs/*.qnt && quint verify specs/*.qnt || exit 16
}

# Verus
rg 'verus!' -q -t rust && {
  verus src/*.rs || exit 16
}
```

## Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | All pass | Deploy |
| 11 | Toolchain missing | Install rustup |
| 12 | Format/structure | Run cargo fmt |
| 13 | Clippy/tests fail | Fix code |
| 14 | Security issues | Review audit |
| 15 | Verification fail | Fix proofs |
| 16 | External fail | Fix specs |

## Miri Usage Notes

Miri is for LOCAL DEBUGGING ONLY:
- Requires nightly Rust
- Much slower than normal tests
- NOT recommended for CI/CD

Manual execution:
```bash
rustup +nightly component add miri
cargo +nightly miri test
```

## Workflow

```
CHECK
  |
  v
BASIC (fmt->clippy->audit->deny)
  |
  v
TEST (cargo test)
  |
  +--[#[requires]]-----> PRUSTI
  +--[#[kani::proof]]--> KANI
  +--[#[flux::]]-------> FLUX
  +--[loom::]----------> LOOM
  +--[unsafe, local]---> MIRI (advisory)
  |
  v
EXTERNAL (optional)
  |
  +--[*.idr]-----------> IDRIS2
  +--[lakefile.lean]---> LEAN4
  +--[*.qnt]-----------> QUINT
  +--[verus!]----------> VERUS
  |
  v
COMPLETE (exit 0)
```

## Required Output

1. **Validation Summary**
   - Each tier: PASS/FAIL/SKIP
   - Error details with file:line

2. **Coverage Report** (if available)

3. **Formal Verification Status**
   - Prusti/Kani/Flux/Loom results

4. **Recommendations**
   - Unverified critical code
   - Missing annotations
