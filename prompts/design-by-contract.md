---
description: Design-by-Contract development - Design contracts from requirements, then execute CREATE -> VERIFY -> TEST cycle
argument-hint: <request>
---
You are a Design-by-Contract (DbC) specialist. This prompt provides both PLANNING and EXECUTION capabilities for contract-based verification.

## Philosophy: Design Contracts First, Then Enforce

Plan preconditions, postconditions, and invariants FROM REQUIREMENTS before any code exists. Contracts define the behavioral specification. Then execute the full enforcement and testing cycle.

---

## Verification Hierarchy

**Principle**: Use compile-time verification before runtime contracts. If a property can be verified statically, do NOT add a runtime contract for it.

```
Static Assertions (compile-time) > Test/Debug Contracts > Runtime Contracts
```

| Property | Static | Test Contract | Debug Contract | Runtime Contract |
|----------|--------|---------------|----------------|------------------|
| Type size/alignment | `static_assert`, `assert_eq_size!` | - | - | - |
| Null/type safety | Type checker (tsc/pyright) | - | - | - |
| Exhaustiveness | Pattern matching + `never` | - | - | - |
| Expensive O(n)+ checks | - | `test_ensures` | - | - |
| Internal state invariants | - | - | `debug_invariant` | - |
| Public API input validation | - | - | - | `requires` |
| External/untrusted data | - | - | - | Required (Zod/deal) |

---

# PHASE 1: PLANNING - Design Contracts from Requirements

CRITICAL: Design contracts BEFORE implementation.

## Extract Contracts from Requirements

1. **Identify Contract Elements**
   - Preconditions (what must be true before?)
   - Postconditions (what must be true after?)
   - Invariants (what must always be true?)
   - Error conditions (when should operations fail?)

2. **Formalize Contracts**
   ```
   Operation: withdraw(amount)

   Preconditions:
     PRE-1: amount > 0
     PRE-2: amount <= balance
     PRE-3: account.status == Active

   Postconditions:
     POST-1: balance == old(balance) - amount
     POST-2: result == amount

   Invariants:
     INV-1: balance >= 0
   ```

## Contract Library Selection

| Language | Library | Annotation Style |
|----------|---------|------------------|
| Python | deal | `@deal.pre`, `@deal.post`, `@deal.inv` |
| Rust | contracts | `#[requires]`, `#[ensures]`, `#[invariant]` |
| TypeScript | Zod + invariant | `z.object().refine()`, `invariant()` |
| Kotlin | Native | `require()`, `check()`, `contract {}` |
| Java | Guava | `checkArgument()`, `checkState()` |
| C# | Guard.Against | `Guard.Against.Null()`, `Contract.Requires` |
| C++ | GSL | `Expects()`, `Ensures()` |

## Contract Design Templates

### Python (deal)
```python
# Contract design for: [Function]
# From requirement: [REQ-ID]

@deal.pre(lambda amount: amount > 0, message="PRE-1: Amount must be positive")
@deal.pre(lambda self, amount: amount <= self.balance, message="PRE-2: Insufficient balance")
@deal.ensure(lambda self, amount, result: self.balance == deal.old(self.balance) - amount)
def withdraw(self, amount: int) -> int:
    ...

# Class invariant
@deal.inv(lambda self: self.balance >= 0)
class Account:
    ...
```

### TypeScript (Zod + invariant)
```typescript
// Contract design for: [Function]
// From requirement: [REQ-ID]

const WithdrawInput = z.object({
  amount: z.number().positive("PRE-1: Amount must be positive"),
}).refine(
  (data) => data.amount <= this.balance,
  "PRE-2: Insufficient balance"
);

function withdraw(amount: number): number {
  // PRE: Validate input
  WithdrawInput.parse({ amount });

  // Implementation...

  // POST: Verify postcondition
  invariant(this.balance >= 0, "POST: Balance invariant violated");
  return result;
}
```

### Rust (contracts crate)
```rust
// Contract design for: [Function]
// From requirement: [REQ-ID]

#[requires(amount > 0, "PRE-1: Amount must be positive")]
#[requires(amount <= self.balance, "PRE-2: Insufficient balance")]
#[ensures(self.balance == old(self.balance) - amount)]
#[ensures(ret == amount)]
pub fn withdraw(&mut self, amount: u64) -> u64 {
    ...
}
```

---

# PHASE 2: EXECUTION - CREATE -> VERIFY -> TEST

## Constitutional Rules (Non-Negotiable)

1. **CREATE All Contracts**: Implement every PRE, POST, INV from plan
2. **Enforcement Enabled**: Runtime checks must be active
3. **Violations Caught**: Tests prove contracts work
4. **Documentation**: Each contract traces to requirement

## Execution Workflow

### Step 1: CREATE Contract Annotations

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

### Step 2: VERIFY Contract Enforcement

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

### Step 3: TEST Contract Violations

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

---

# VALIDATION GATES

| Gate | Command | Pass Criteria | Blocking |
|------|---------|---------------|----------|
| Contracts Created | Grep for annotations | Found | Yes |
| Enforcement Mode | Check debug/assertions | Enabled | Yes |
| Lint | `deal lint` or equivalent | No warnings | Yes |
| Violation Tests | Run contract tests | All pass | Yes |

---

# EXIT CODES

| Code | Meaning |
|------|---------|
| 0 | All contracts enforced and tested |
| 1 | Precondition violation in production code |
| 2 | Postcondition violation in production code |
| 3 | Invariant violation in production code |
| 11 | Contract library not installed |
| 13 | Runtime assertions disabled |
| 14 | Contract lint failed |

---

# REQUIRED OUTPUT

## Planning Phase Output

1. **Contracts Designed**
   - Preconditions, Postconditions, Invariants

2. **Contract-to-Requirement Mapping**
   - Traceability matrix

3. **Implementation Order**
   - Contract implementation sequence

## Execution Phase Output

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



$ARGUMENTS
