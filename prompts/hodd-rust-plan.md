---
description: Plan HODD-RUST validation workflow for Rust projects
argument-hint: [PATH=<directory>]
---

You are planning a HODD-RUST (Stronger Outline Driven Development For Rust) validation strategy.

## Arguments

- `$PATH` - Directory containing Rust project with Cargo.toml (required)

**Critical**: This is a READ-ONLY planning task. Do NOT modify any files.

HODD-RUST merges: Type-driven + Spec-first + Proof-driven + Design-by-contracts + Test-driven (XP)

## Your Process

1. **Detect Existing Validation Artifacts**
   - Find Prusti annotations (#[requires], #[ensures], #[invariant])
   - Find Kani proofs (#[kani::proof])
   - Find Flux refinements (#[flux::])
   - Find Loom concurrency tests (loom::)
   - Find property tests (proptest!, quickcheck)

2. **Analyze Safety Requirements**
   - Identify unsafe blocks requiring Miri/manual review
   - Find FFI boundaries (extern "C", #[no_mangle])
   - Locate concurrent code (Arc, Mutex, atomics, spawn)
   - Detect panic paths (unwrap, expect, panic!)

3. **Design Validation Strategy**
   - Match tools to code criticality
   - Identify formal verification candidates
   - Plan external tool usage (Idris2, Lean4, Quint)

4. **Identify Critical Files**
   - List 3-5 files requiring formal verification
   - Document applicable validation tools per file

## Detection Commands

```bash
# Find unsafe blocks
rg 'unsafe\s*\{' -t rust -l $PATH

# Find FFI boundaries
rg 'extern\s+"C"' -t rust -l $PATH

# Find concurrent code
rg 'Arc<|Mutex<|RwLock<|AtomicU|thread::spawn' -t rust -l $PATH

# Find Prusti annotations
rg '#\[(requires|ensures|invariant)(\(|])' -t rust -l $PATH

# Find Kani proofs
rg '#\[kani::proof\]' -t rust -l $PATH

# Find Flux refinements
rg '#\[flux::' -t rust -l $PATH

# Find Loom tests
rg 'loom::' -t rust -l $PATH

# Find property tests
rg 'proptest!|quickcheck' -t rust -l $PATH

# Find panic paths
rg '\.unwrap\(\)|\.expect\(|panic!' -t rust -l $PATH

# Count test files
fd -e rs -g '*test*' $PATH | wc -l
```

## External Tool Detection

```bash
# Idris2 models
fd -e idr -e lidr $PATH

# Lean4 proofs
fd lakefile.lean $PATH

# Quint specifications
fd -e qnt $PATH

# Verus annotations
rg 'verus!' -t rust -l $PATH
```

## Tool Stack Reference

| Layer | Tool | When to Use |
|-------|------|-------------|
| 0 | rustc/clippy | Always |
| 0 | cargo-audit/deny | CI mandatory |
| 1 | Miri | Local UB debugging ONLY |
| 2 | Loom | Critical concurrency |
| 3 | Flux | Refined types |
| 4 | Prusti | Contract verification |
| 5 | Lean4 | Formal proofs (external) |
| 6 | Kani | Bounded model checking |
| 6 | Quint | Protocol specs (external) |

## Exit Codes Reference

| Code | Meaning |
|------|---------|
| 0 | Planning complete |
| 11 | Toolchain not found |
| 12 | No Rust files found |

## Required Output

1. **Validation Artifact Inventory**
   - Existing annotations/proofs
   - Test coverage estimate

2. **Safety Analysis**
   - Unsafe block count/locations
   - FFI/concurrency summary

3. **Validation Strategy**
   - Tool selection per code region
   - Prioritized verification targets

4. **Critical Files List** (3-5 files)
   - Path with rationale
   - Applicable tools
