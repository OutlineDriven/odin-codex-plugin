---
description: Design HODD-RUST validation strategy from requirements
---

You are planning a HODD-RUST (Stronger Outline Driven Development For Rust) validation strategy FROM REQUIREMENTS.

CRITICAL: This is a READ-ONLY planning task. Design Rust-specific validations BEFORE implementation.

**Strict Enforcement**: Strictly validation-first before-and-after(-and-while) planning and execution. Design ALL validations (types, specs, proofs, contracts) BEFORE any code. No code design without validation design.

## Philosophy: Design Rust Validations First

Plan Prusti contracts, Kani proofs, and Loom verifications FROM REQUIREMENTS before any code changes. The tiered verification stack catches different defect classes.

HODD-RUST merges: Type-driven + Spec-first + Proof-driven + Design-by-contracts

## Verification Stack

```
Tier | Tool        | Catches              | When to Use
-----|-------------|----------------------|------------------
0    | rustfmt     | Style violations     | Always
0    | clippy      | Common mistakes      | Always
1    | Miri        | Undefined behavior   | Local debugging ONLY
2    | Loom        | Race conditions      | Concurrent code
3    | Flux        | Type refinement      | Numeric constraints
4    | Prusti      | Contract violations  | API boundaries
5    | Kani        | Logic errors         | Critical algorithms
6    | Lean4/Quint | Design flaws         | Complex protocols
```

## Your Process

### Phase 1: Extract Verification Requirements

1. **Identify Safety Requirements**
   - Memory safety (ownership, lifetimes)
   - Thread safety (Send, Sync, data races)
   - Panic freedom (unwrap, expect, index)
   - FFI safety (extern, #[no_mangle])

2. **Identify Correctness Requirements**
   - Algorithm correctness
   - State machine validity
   - Protocol compliance
   - Business logic invariants

### Phase 2: Design Verification Artifacts

1. **Tier 4 - Prusti Contracts**
   ```rust
   // Design from requirement: [REQ-ID]
   #[requires(amount > 0)]
   #[requires(amount <= self.balance)]
   #[ensures(self.balance == old(self.balance) - amount)]
   fn withdraw(&mut self, amount: u64) -> u64
   ```

2. **Tier 5 - Kani Proofs**
   ```rust
   // Design from requirement: [REQ-ID]
   #[cfg(kani)]
   #[kani::proof]
   #[kani::unwind(10)]
   fn verify_withdraw_safe() {
       // Bounded model checking for withdraw
   }
   ```

3. **Tier 2 - Loom Verification**
   ```rust
   // Design from requirement: [REQ-ID] - Thread safety
   #[cfg(loom)]
   fn verify_concurrent_access() {
       loom::model(|| {
           // Exhaustive concurrency verification
       });
   }
   ```

### Phase 3: Conditional Strategy

**If NO existing Rust verification artifacts:**
```bash
# Check for existing verifications
rg '#\[requires|#\[ensures|#\[invariant' -t rust $PATH  # Prusti
rg '#\[kani::proof\]' -t rust $PATH                      # Kani
rg 'loom::' -t rust $PATH                                # Loom
```

Design complete verification suite:
- Plan Prusti contracts for all public APIs
- Design Kani proofs for critical algorithms
- Specify Loom verifications for concurrent code

**If existing verification artifacts found:**
- Analyze current coverage per tier
- Identify gaps in verification
- Design additions for new requirements

### Phase 4: External Proof Integration

If requirements exceed Rust-native verification:

```
.outline/
├── proofs/
│   └── lean/          # Complex algorithm proofs
│       ├── lakefile.lean
│       └── Algorithm.lean
└── specs/
    └── protocol.qnt   # State machine specifications
```

## Exit Codes Reference

| Code | Meaning |
|------|---------|
| 0 | Plan complete, verifications designed |
| 11 | Toolchain not found |
| 12 | Unable to design from requirements |

## Required Output

### 1. Safety Analysis

```markdown
## Safety Requirements

### Memory Safety
- [ ] Ownership correctly modeled
- [ ] Lifetimes properly bounded
- [ ] No undefined behavior paths

### Thread Safety
- [ ] Send/Sync correctly derived
- [ ] No data race possibilities
- [ ] Lock ordering documented

### Panic Freedom
- Files with `.unwrap()`: [count]
- Files with `.expect()`: [count]
- Mitigation strategy: [approach]
```

### 2. Verification Design by Tier

```markdown
## Tier 4: Prusti Contracts

### Contracts to Add
| Function | Preconditions | Postconditions | Invariants |
|----------|---------------|----------------|------------|
| `func_1` | PRE-1, PRE-2 | POST-1 | INV-1 |

## Tier 5: Kani Proofs

### Proofs to Create
| Proof | Target | Property | Bounds |
|-------|--------|----------|--------|
| `verify_x` | `func_1` | Correctness | unwind(10) |

## Tier 2: Loom Verification

### Concurrency Verifications
| Verification | Target | Scenario |
|--------------|--------|----------|
| `verify_concurrent_x` | `SharedState` | Multi-thread access |
```

### 3. Critical Files

```markdown
## Critical Files for Verification

| File | Tier | Tool | Rationale |
|------|------|------|-----------|
| `src/core.rs` | 4, 5 | Prusti, Kani | Core logic |
| `src/sync.rs` | 2 | Loom | Concurrency |
| `src/api.rs` | 4 | Prusti | Public API |
```

### 4. Artifact Structure

```markdown
## Verification Artifacts

src/
├── lib.rs           # Prusti annotations inline
└── core.rs          # Prusti annotations inline

verifications/
├── kani_proofs.rs   # Tier 5: Kani proofs
└── loom_verify.rs   # Tier 2: Loom verifications

.outline/
├── proofs/lean/     # Tier 6: External proofs (if needed)
└── specs/           # Tier 6: Quint specs (if needed)
```

### 5. Validation Pipeline

```markdown
## Execution Order (for run phase)

1. `cargo fmt --check` - Tier 0
2. `cargo clippy -- -D warnings` - Tier 0
3. `cargo prusti` - Tier 4 (if annotations present)
4. `cargo kani` - Tier 5 (if proofs present)
5. `RUSTFLAGS='--cfg loom' cargo build` - Tier 2 (if loom verifications)
6. External proofs - Tier 6 (if present)
```

Design Rust verifications FROM REQUIREMENTS. Do NOT write files.
