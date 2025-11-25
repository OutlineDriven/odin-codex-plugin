---
description: Execute HODD-RUST validation: CREATE Rust-specific verifications from plan, VERIFY through pipeline, REMEDIATE
argument-hint: [PATH=<directory>]
---

You are executing the HODD-RUST validation pipeline. Your mission: CREATE the Rust-specific verification artifacts designed in the plan phase, VERIFY through the complete pipeline, then REMEDIATE any issues.

## Arguments

- `$PATH` - Directory containing Rust project with Cargo.toml (required)

## Philosophy: Create Verifications, Then Validate

This is the EXECUTION phase. The plan phase designed the Rust verification strategy. Now you:
1. CREATE Prusti contracts, Kani proofs, Loom tests, and property tests
2. VERIFY through the tiered pipeline
3. REMEDIATE failures through iterative fixes
4. INTEGRATE external proofs if designed

## HODD-RUST Verification Stack

```
Tier | Tool        | Creates              | Validates
-----|-------------|----------------------|------------------
0    | rustfmt     | Formatted code       | Style
0    | clippy      | Linted code          | Common issues
1    | Miri        | UB checks (local)    | Memory safety
2    | Loom        | Concurrency tests    | Thread safety
3    | Flux        | Refined types        | Constraints
4    | Prusti      | Contract annotations | Pre/post/inv
5    | Kani        | Proof harnesses      | Algorithm correctness
6    | Lean4/Quint | External proofs      | Formal properties
7    | proptest    | Property tests       | Behavioral properties
```

## Constitutional Rules (Non-Negotiable)

1. **CREATE Before Verify**: Add all annotations/tests from plan
2. **Tier Order**: Execute tiers in sequence
3. **Fail-Fast**: Stop on blocking failures
4. **Miri Advisory**: Local debugging only, not CI
5. **Complete Remediation**: Fix issues, don't skip verification

## Execution Workflow

### Phase 1: CREATE Tier 0 - Baseline Setup

```bash
cd $PATH

# Verify toolchain
rustc --version || exit 11
cargo --version || exit 11

# Format check
cargo fmt --check || exit 12

# Clippy with strict warnings
cargo clippy -- -D warnings || exit 13

# Security audit (if available)
cargo audit 2>/dev/null || echo "cargo-audit not installed"
cargo deny check 2>/dev/null || echo "cargo-deny not installed"
```

### Phase 2: CREATE Tier 4 - Prusti Contracts (from plan)

```rust
// src/account.rs
// Contracts from plan design

use prusti_contracts::*;

pub struct Account {
    balance: u64,
    status: AccountStatus,
}

impl Account {
    // Preconditions from plan PRE-1, PRE-2, PRE-3
    #[requires(amount > 0)]
    #[requires(amount as u64 <= self.balance)]
    #[requires(self.status == AccountStatus::Active)]
    // Postconditions from plan POST-1, POST-2
    #[ensures(result == amount as u64)]
    #[ensures(self.balance == old(self.balance) - amount as u64)]
    pub fn withdraw(&mut self, amount: u32) -> u64 {
        let amt = amount as u64;
        self.balance -= amt;
        amt
    }
}

// Class invariant from plan INV-1
#[invariant(self.balance <= u64::MAX)]
impl Account {
    // All methods checked against invariant
}
```

**Verify Prusti:**
```bash
if rg '#\[(requires|ensures|invariant)\]' -q -t rust; then
    echo "Prusti annotations detected"
    cargo prusti || exit 15
    echo "Prusti verification passed"
fi
```

### Phase 3: CREATE Tier 5 - Kani Proofs (from plan)

```rust
// tests/kani_proofs.rs
// Proofs from plan design

#[cfg(kani)]
mod kani_proofs {
    use super::*;

    // Proof from plan: verify_withdraw_safe
    #[kani::proof]
    #[kani::unwind(10)]
    fn verify_withdraw_preserves_invariant() {
        let initial: u64 = kani::any();
        let amount: u32 = kani::any();

        kani::assume(initial <= 1000000);
        kani::assume(amount > 0);
        kani::assume((amount as u64) <= initial);

        let mut account = Account::new(initial);
        let result = account.withdraw(amount);

        // Verified property from plan
        kani::assert(account.balance >= 0, "Balance invariant violated");
        kani::assert(result == amount as u64, "Return value incorrect");
    }

    // Proof from plan: verify_no_overflow
    #[kani::proof]
    fn verify_deposit_no_overflow() {
        let balance: u64 = kani::any();
        let amount: u64 = kani::any();

        kani::assume(balance <= u64::MAX / 2);
        kani::assume(amount <= u64::MAX / 2);

        let result = balance.checked_add(amount);
        kani::assert(result.is_some(), "Overflow possible");
    }
}
```

**Verify Kani:**
```bash
if rg '#\[kani::proof\]' -q -t rust; then
    echo "Kani proofs detected"
    cargo kani || exit 15
    echo "Kani verification passed"
fi
```

### Phase 4: CREATE Tier 2 - Loom Tests (from plan)

```rust
// tests/loom_tests.rs
// Concurrency tests from plan

#[cfg(loom)]
mod loom_tests {
    use loom::sync::{Arc, Mutex};
    use loom::thread;

    #[test]
    fn test_concurrent_access() {
        loom::model(|| {
            let account = Arc::new(Mutex::new(Account::new(1000)));

            let a1 = account.clone();
            let a2 = account.clone();

            let t1 = thread::spawn(move || {
                let mut acc = a1.lock().unwrap();
                if acc.balance >= 100 {
                    acc.withdraw(100);
                }
            });

            let t2 = thread::spawn(move || {
                let mut acc = a2.lock().unwrap();
                acc.deposit(50);
            });

            t1.join().unwrap();
            t2.join().unwrap();

            let acc = account.lock().unwrap();
            assert!(acc.balance >= 0);
        });
    }
}
```

**Verify Loom:**
```bash
if rg 'loom::' -q -t rust; then
    echo "Loom tests detected"
    RUSTFLAGS='--cfg loom' cargo test --release || exit 15
    echo "Loom tests passed"
fi
```

### Phase 5: CREATE Tier 7 - Property Tests (from plan)

```rust
// tests/property_tests.rs
// Properties from plan

use proptest::prelude::*;

proptest! {
    // Property from plan: balance_never_negative
    #[test]
    fn prop_balance_never_negative(
        initial in 0u64..1000000,
        amount in 1u32..1000
    ) {
        prop_assume!((amount as u64) <= initial);

        let mut account = Account::new(initial);
        account.withdraw(amount);

        prop_assert!(account.balance >= 0);
    }

    // Property from plan: deposit_increases_balance
    #[test]
    fn prop_deposit_increases(
        initial in 0u64..1000000,
        amount in 1u32..1000
    ) {
        let mut account = Account::new(initial);
        let old_balance = account.balance;
        account.deposit(amount);

        prop_assert!(account.balance > old_balance);
    }

    // Property from plan: withdraw_exact_amount
    #[test]
    fn prop_withdraw_returns_amount(
        initial in 100u64..1000000,
        amount in 1u32..100
    ) {
        prop_assume!((amount as u64) <= initial);

        let mut account = Account::new(initial);
        let result = account.withdraw(amount);

        prop_assert_eq!(result, amount as u64);
    }
}
```

**Verify Property Tests:**
```bash
cargo test --test property_tests || exit 13
```

### Phase 6: CREATE Tier 6 - External Proofs (from plan, optional)

```bash
# Create external proof directories if designed
mkdir -p .outline/proofs/lean
mkdir -p .outline/specs

# Lean 4 proofs (if designed in plan)
if [ -d ".outline/proofs/lean" ] && [ -f ".outline/proofs/lean/lakefile.lean" ]; then
    cd .outline/proofs/lean && lake build && cd ../../..
    test $(rg "sorry" .outline/proofs/lean/*.lean 2>/dev/null | wc -l) -eq 0 || exit 16
fi

# Quint specs (if designed in plan)
if fd -e qnt . .outline/specs/ 2>/dev/null | head -1 | grep -q .; then
    quint typecheck .outline/specs/*.qnt || exit 16
    quint verify .outline/specs/*.qnt || exit 16
fi
```

### Phase 7: Full Pipeline Execution

```bash
#!/bin/bash
set -e

echo "=== HODD-RUST VALIDATION PIPELINE ==="

echo "[Tier 0] Baseline..."
cargo fmt --check || exit 12
cargo clippy -- -D warnings || exit 13
cargo audit 2>/dev/null || echo "SKIP: cargo-audit"

echo "[Tier Tests] cargo test..."
cargo test || exit 13

echo "[Tier 4] Prusti contracts..."
if rg '#\[(requires|ensures|invariant)\]' -q -t rust; then
    cargo prusti || exit 15
fi

echo "[Tier 5] Kani proofs..."
if rg '#\[kani::proof\]' -q -t rust; then
    cargo kani || exit 15
fi

echo "[Tier 2] Loom concurrency..."
if rg 'loom::' -q -t rust; then
    RUSTFLAGS='--cfg loom' cargo test --release || exit 15
fi

echo "[Tier 1] Miri (advisory)..."
if rg 'unsafe\s*\{' -q -t rust; then
    echo "ADVISORY: Run 'cargo +nightly miri test' locally for UB detection"
fi

echo "[Tier 6] External proofs..."
if [ -d ".outline/proofs/lean" ] && [ -f ".outline/proofs/lean/lakefile.lean" ]; then
    cd .outline/proofs/lean && lake build && cd ../../..
fi

echo "=== HODD-RUST VALIDATION COMPLETE ==="
```

## Validation Gates

| Gate | Tier | Command | Pass Criteria | Blocking |
|------|------|---------|---------------|----------|
| Format | 0 | `cargo fmt --check` | Clean | Yes |
| Clippy | 0 | `cargo clippy` | No warnings | Yes |
| Tests | - | `cargo test` | All pass | Yes |
| Prusti | 4 | `cargo prusti` | Verified | Yes* |
| Kani | 5 | `cargo kani` | No violations | Yes* |
| Loom | 2 | `cargo test --cfg loom` | No races | Yes* |
| Miri | 1 | `cargo miri test` | No UB | No (local) |
| External | 6 | `lake build` | No sorry | Yes* |

*If annotations/proofs present

## Required Output

1. **Annotated Code** - Prusti contracts from plan
2. **Kani Proofs** - Proof harnesses from plan
3. **Loom Tests** - Concurrency tests if applicable
4. **Property Tests** - From plan properties
5. **External Proofs** - If designed in plan
6. **Pipeline Report** - All tiers status

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All validations pass |
| 11 | Toolchain not found |
| 12 | Format violations |
| 13 | Clippy/test failures |
| 14 | Security/dependency issues |
| 15 | Formal verification failed |
| 16 | External proofs failed |

Execute CREATE for each tier -> VERIFY through pipeline -> REMEDIATE failures.
