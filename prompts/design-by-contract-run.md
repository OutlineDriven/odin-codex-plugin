---
description: Execute Design-by-Contract: CREATE contracts from plan, VERIFY enforcement, TEST violations
argument-hint: <request>
---
You are executing Design-by-Contract verification. Your mission: CREATE the contracts designed in the plan phase, VERIFY enforcement, then TEST that violations are caught.

## Philosophy: Create Contracts, Then Enforce

This is the EXECUTION phase. The plan phase designed preconditions, postconditions, and invariants. Now you:
1. CREATE contract annotations from plan design
2. VERIFY contracts are enforced at runtime
3. TEST that contract violations are properly caught

## Constitutional Rules (Non-Negotiable)

1. **CREATE All Contracts**: Implement every PRE, POST, INV from plan
2. **Enforcement Enabled**: Runtime checks must be active
3. **Violations Caught**: Tests prove contracts work
4. **Documentation**: Each contract traces to requirement

## Execution Workflow

### Phase 1: CREATE Contract Annotations (from plan)

**Python (deal):**
```python
# src/account.py
# Contracts from plan design

import deal

@deal.inv(lambda self: self.balance >= 0)  # From plan INV-1
class Account:
    def __init__(self, balance: int):
        self.balance = balance

    @deal.pre(lambda self, amount: amount > 0)  # From plan PRE-1
    @deal.pre(lambda self, amount: amount <= self.balance)  # From plan PRE-2
    @deal.ensure(lambda self, amount, result: result == amount)  # From plan POST-1
    @deal.ensure(lambda self, amount, result:
        self.balance == deal.old(self.balance) - amount)  # From plan POST-2
    def withdraw(self, amount: int) -> int:
        self.balance -= amount
        return amount
```

**Rust (contracts crate):**
```rust
// src/account.rs
// Contracts from plan design

use contracts::*;

pub struct Account {
    balance: u64,
}

impl Account {
    #[requires(amount > 0, "PRE-1: Amount must be positive")]
    #[requires(amount <= self.balance, "PRE-2: Insufficient balance")]
    #[ensures(ret == amount, "POST-1: Returns withdrawn amount")]
    #[ensures(self.balance == old(self.balance) - amount, "POST-2: Balance reduced")]
    pub fn withdraw(&mut self, amount: u64) -> u64 {
        self.balance -= amount;
        amount
    }
}

// Class invariant
#[invariant(self.balance <= u64::MAX)]
impl Account {
    // All methods checked against invariant
}
```

**TypeScript (Zod + invariant):**
```typescript
// src/account.ts
// Contracts from plan design

import { z } from 'zod';
import invariant from 'tiny-invariant';

// From plan PRE-1, PRE-2
const WithdrawInput = z.object({
  amount: z.number().positive("PRE-1: Amount must be positive"),
});

class Account {
  private _balance: number;

  constructor(balance: number) {
    this._balance = balance;
  }

  // From plan INV-1
  private checkInvariant(): void {
    invariant(this._balance >= 0, "INV-1: Balance must be non-negative");
  }

  withdraw(amount: number): number {
    // PRE: Validate input
    WithdrawInput.parse({ amount });
    invariant(amount <= this._balance, "PRE-2: Insufficient balance");

    const oldBalance = this._balance;
    this._balance -= amount;

    // POST: Verify postconditions
    invariant(this._balance === oldBalance - amount, "POST-2: Balance reduction");

    // INV: Check invariant
    this.checkInvariant();

    return amount;  // POST-1: Returns amount
  }
}
```

**Kotlin (Native):**
```kotlin
// src/Account.kt
// Contracts from plan design

class Account(private var balance: Int) {
    init {
        require(balance >= 0) { "INV-1: Balance must be non-negative" }
    }

    fun withdraw(amount: Int): Int {
        // PRE from plan
        require(amount > 0) { "PRE-1: Amount must be positive" }
        require(amount <= balance) { "PRE-2: Insufficient balance" }

        val oldBalance = balance
        balance -= amount

        // POST from plan
        check(balance == oldBalance - amount) { "POST-2: Balance reduction" }
        check(balance >= 0) { "INV-1: Invariant violated" }

        return amount  // POST-1
    }
}
```

### Phase 2: VERIFY Contract Enforcement

```bash
# Python: Ensure debug mode (assertions enabled)
python -c "assert __debug__, 'assertions disabled'" || exit 13

# Run deal linter
deal lint src/ || exit 14

# TypeScript: Ensure not in production mode
[[ "$NODE_ENV" != "production" ]] || echo "Warning: contracts may be disabled"

# Rust: Build in debug mode (assertions enabled)
cargo build  # Not --release

# Kotlin: Enable assertions
java -ea -version
```

### Phase 3: TEST Contract Violations

```python
# tests/test_contracts.py
# Tests from plan: Verify contracts catch violations

import pytest
import deal

class TestContractViolations:
    """Verify contracts are enforced"""

    def test_precondition_negative_amount(self):
        """PRE-1 violation caught"""
        account = Account(100)
        with pytest.raises(deal.PreContractError):
            account.withdraw(-50)

    def test_precondition_insufficient_balance(self):
        """PRE-2 violation caught"""
        account = Account(100)
        with pytest.raises(deal.PreContractError):
            account.withdraw(150)

    def test_invariant_enforced(self):
        """INV-1 cannot be violated"""
        # This should be impossible via public API
        account = Account(100)
        account.withdraw(100)
        assert account.balance >= 0

    def test_postcondition_return_value(self):
        """POST-1 verified"""
        account = Account(100)
        result = account.withdraw(30)
        assert result == 30
```

## Contract Libraries Reference

| Language | Library | Install |
|----------|---------|---------|
| Python | deal | `pip install deal` |
| Rust | contracts | `cargo add contracts` |
| TypeScript | Zod + tiny-invariant | `npm install zod tiny-invariant` |
| Kotlin | Native | Built-in |
| Java | Guava | `com.google.guava:guava` |
| C# | Guard.Against | `Ardalis.GuardClauses` |
| C++ | GSL | `Microsoft.GSL` |

## Validation Gates

| Gate | Command | Pass Criteria | Blocking |
|------|---------|---------------|----------|
| Contracts Created | Grep for annotations | Found | Yes |
| Enforcement Mode | Check debug/assertions | Enabled | Yes |
| Lint | `deal lint` or equivalent | No warnings | Yes |
| Violation Tests | Run contract tests | All pass | Yes |

## Required Output

1. **Created Contracts**
   - PRE-* annotations added
   - POST-* annotations added
   - INV-* annotations added
   - Traceability to plan

2. **Enforcement Status**
   - Debug mode: ENABLED/DISABLED
   - Lint results: PASS/FAIL

3. **Violation Test Results**
   - PRE violations caught: Yes/No
   - POST violations caught: Yes/No
   - INV violations caught: Yes/No

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All contracts enforced and tested |
| 1 | Precondition violation in production code |
| 2 | Postcondition violation in production code |
| 3 | Invariant violation in production code |
| 11 | Contract library not installed |
| 13 | Runtime assertions disabled |
| 14 | Contract lint failed |

Execute CREATE -> VERIFY -> TEST cycle.



$ARGUMENTS
