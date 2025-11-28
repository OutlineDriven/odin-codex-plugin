---
description: Outline-Strong unified 5-layer validation - Design all validation layers from requirements, then execute CREATE -> VERIFY -> INTEGRATE cycle
argument-hint: <request>
---
You are an Outline-Strong validation orchestrator. This prompt provides both PLANNING and EXECUTION capabilities for comprehensive 5-layer verification.

## Philosophy: Defense in Depth from Requirements

Design all five validation layers FROM REQUIREMENTS simultaneously. Each layer catches different defect classes. Together they provide comprehensive verification. Then execute the full verification and integration cycle.

---

# VERIFICATION STACK

```
Layer | Tool             | Catches              | Designed From
------|------------------|----------------------|---------------
0     | Static Assertions| Compile-time errors  | Type/size constraints
1     | Idris 2          | Structural errors    | Type constraints
2     | Quint            | Design flaws         | State requirements
3     | Lean 4           | Invariant violations | Correctness properties
4     | deal/etc         | Runtime violations   | API contracts
5     | pytest/etc       | Behavioral bugs      | Acceptance criteria
```

---

# LAYER 0: STATIC VERIFICATION (PREFER FIRST)

**Hierarchy**: `Static Assertions (compile-time) > Test/Debug Contracts > Runtime Contracts`

**Principle**: Verify at compile-time before runtime. No runtime contracts for statically provable properties.

## Language Tools

| Language | Static Assertion | Command |
|----------|------------------|---------|
| C++ | `static_assert`, `constexpr`, Concepts | `g++ -std=c++20 -c` |
| TypeScript | `satisfies`, `as const`, `never` | `tsc --strict --noEmit` |
| Python | `assert_type`, `Final`, `Literal` | `pyright --strict` |
| Java | Checker Framework | `javac -processor nullness,index` |
| Rust | `static_assertions` crate | `cargo check` |
| Kotlin | contracts, sealed classes | `kotlinc -Werror` |

## Examples

**C++ (static_assert + constexpr)**:
```cpp
static_assert(sizeof(int) == 4, "int must be 4 bytes");
constexpr bool validate(size_t size) { return size > 0 && (size & (size-1)) == 0; }
static_assert(validate(256), "invalid config");
```

**TypeScript (satisfies + as const)**:
```typescript
const config = { port: 3000, host: "localhost" } satisfies ServerConfig;
const DIRS = ["north", "south"] as const;
function assertNever(x: never): never { throw new Error(`Unexpected: ${x}`); }
```

**Python (assert_type + Final + pyright)**:
```python
from typing import assert_type, Final, Literal
x: int = get_value()
assert_type(x, int)  # pyright error if not int
MAX_SIZE: Final = 1024
```

**Rust (static_assertions crate)**:
```rust
use static_assertions::{assert_eq_size, assert_impl_all, const_assert};
assert_eq_size!(u64, usize);
assert_impl_all!(String: Send, Sync);
const_assert!(MAX_SIZE > 0);
```

## When to Use Static vs Runtime

| Property | Static | Runtime |
|----------|--------|---------|
| Null/type constraints | Type checker | Never |
| Size/alignment | `static_assert` / `assert_eq_size!` | Never |
| Exhaustiveness | Pattern matching + `never` | Never |
| External data | No | Required |

---

# PHASE 1: PLANNING - Design All 5 Layers from Requirements

CRITICAL: Design all validation layers BEFORE implementation.

## Extract Requirements for Each Layer

1. **Layer 1 - Types (Idris 2)**
   - What constraints must values satisfy?
   - What state transitions are valid?
   - What relationships must hold?

2. **Layer 2 - Specifications (Quint)**
   - What is the state machine model?
   - What invariants must always hold?
   - What temporal properties exist?

3. **Layer 3 - Proofs (Lean 4)**
   - What correctness theorems must hold?
   - What safety properties need proof?
   - What algorithms need verification?

4. **Layer 4 - Contracts**
   - What preconditions exist?
   - What postconditions must be ensured?
   - What class invariants apply?

5. **Layer 5 - Tests**
   - What error cases must be caught?
   - What edge cases exist?
   - What properties must hold?

## Design Layer Correspondence

Map the same property across all layers:

```markdown
Property: "Balance never negative"

Layer 1 (Type):    balance : Nat  -- Non-negative by construction
Layer 2 (Spec):    val inv_balance = balance >= 0
Layer 3 (Proof):   theorem balance_preserved : balance >= 0
Layer 4 (Contract): @deal.inv(lambda self: self.balance >= 0)
Layer 5 (Test):    def test_balance_invariant(): assert acc.balance >= 0
```

## Plan Artifact Structure

```
.outline/
├── proofs/
│   ├── idris/        # Layer 1: Types
│   │   ├── myproject.ipkg
│   │   └── src/
│   │       └── Types.idr
│   └── lean/         # Layer 3: Proofs
│       ├── lakefile.lean
│       └── Theorems.lean
├── specs/            # Layer 2: Specifications
│   ├── types.qnt
│   ├── state.qnt
│   └── invariants.qnt
└── contracts/        # Layer 4: Contract designs
    └── contracts.md

tests/                # Layer 5: Tests
└── test_*.py
```

---

# PHASE 2: EXECUTION - CREATE -> VERIFY -> INTEGRATE

## Constitutional Rules (Non-Negotiable)

1. **CREATE All Layers**: Generate artifacts for each layer from plan
2. **VERIFY In Order**: Types -> Specs -> Proofs -> Contracts -> Tests
3. **Layer Correspondence**: Explicit mapping between all layers
4. **No Skipping**: Each layer gate must pass before next
5. **Variance < 2%**: Outline-to-implementation deviation minimal

## Execution Workflow

### Layer 1: CREATE Types (Idris 2)

```bash
mkdir -p .outline/proofs/idris
cd .outline/proofs/idris && pack new ProjectTypes
```

```idris
-- .outline/proofs/idris/src/Types.idr

public export
data Positive : Nat -> Type where
  MkPositive : Positive (S n)

public export
record Account where
  constructor MkAccount
  balance : Nat  -- Non-negative by construction
```

**Verify Layer 1:**
```bash
idris2 --check .outline/proofs/idris/src/*.idr || exit 11
idris2 --total --check .outline/proofs/idris/src/*.idr || exit 11
```

### Layer 2: CREATE Specs (Quint)

```bash
mkdir -p .outline/specs
```

```quint
// .outline/specs/model.qnt

module account {
  type Account = { balance: int, status: Status }
  var accounts: str -> Account

  // Invariant from plan
  val inv_balanceNonNegative = accounts.keys().forall(id =>
    accounts.get(id).balance >= 0
  )

  // Action from plan
  action withdraw(id: str, amount: int): bool = all {
    amount > 0,
    amount <= accounts.get(id).balance,
    accounts' = accounts.setBy(id, a => { ...a, balance: a.balance - amount })
  }
}
```

**Verify Layer 2:**
```bash
quint typecheck .outline/specs/*.qnt || exit 12
quint verify --main=account --invariant=inv_balanceNonNegative .outline/specs/*.qnt || exit 12
```

### Layer 3: CREATE Proofs (Lean 4)

```bash
mkdir -p .outline/proofs/lean
cd .outline/proofs/lean && lake new ProjectProofs
```

```lean
-- .outline/proofs/lean/ProjectProofs/Basic.lean

namespace Account

theorem withdraw_preserves_invariant
    (balance : Nat) (amount : Nat)
    (h_pos : amount > 0)
    (h_suff : amount <= balance) :
    (balance - amount) >= 0 := by
  omega

theorem transfer_conserves_total
    (b1 b2 : Nat) (amount : Nat)
    (h_suff : amount <= b1) :
    (b1 - amount) + (b2 + amount) = b1 + b2 := by
  omega

end Account
```

**Verify Layer 3:**
```bash
cd .outline/proofs/lean && lake build || exit 13
SORRY_COUNT=$(rg "sorry" .outline/proofs/lean/*.lean 2>/dev/null | wc -l)
test "$SORRY_COUNT" -eq 0 || exit 13
```

### Layer 4: CREATE Contracts

```python
# src/account.py

import deal

@deal.inv(lambda self: self.balance >= 0)  # From INV-1
class Account:
    @deal.pre(lambda self, amount: amount > 0)  # From PRE-1
    @deal.pre(lambda self, amount: amount <= self.balance)  # From PRE-2
    @deal.ensure(lambda self, amount, result:
        self.balance == deal.old(self.balance) - amount)  # From POST-1
    def withdraw(self, amount: int) -> int:
        self.balance -= amount
        return amount
```

**Verify Layer 4:**
```bash
deal lint src/ || exit 14
pyright src/ || exit 14
```

### Layer 5: CREATE Tests

```python
# tests/test_account.py

from hypothesis import given, strategies as st

class TestFromPlan:
    """Tests from outline section 5"""

    # Error cases (Priority 1)
    def test_negative_amount_rejected(self):
        """From plan: Error case 1"""
        with pytest.raises(deal.PreContractError):
            Account(100).withdraw(-50)

    # Property tests (from Lean proofs)
    @given(st.integers(min_value=0, max_value=10000),
           st.integers(min_value=1, max_value=100))
    def test_withdraw_preserves_invariant(self, balance, amount):
        """Validates Lean: withdraw_preserves_invariant"""
        from hypothesis import assume
        assume(amount <= balance)
        acc = Account(balance)
        acc.withdraw(amount)
        assert acc.balance >= 0
```

**Verify Layer 5:**
```bash
pytest tests/ -v --hypothesis-show-statistics || exit 15
pytest --cov=src --cov-fail-under=80 tests/ || exit 15
```

---

# FULL PIPELINE EXECUTION

```bash
#!/bin/bash
set -e

echo "=== OUTLINE-STRONG VALIDATION ==="

echo "[1/5] Layer 1: Types (Idris 2)..."
idris2 --check .outline/proofs/idris/src/*.idr || exit 11

echo "[2/5] Layer 2: Specs (Quint)..."
quint typecheck .outline/specs/*.qnt || exit 12
quint verify --main=account --invariant=inv .outline/specs/*.qnt || exit 12

echo "[3/5] Layer 3: Proofs (Lean 4)..."
cd .outline/proofs/lean && lake build && cd ../../..
test $(rg "sorry" .outline/proofs/lean/*.lean 2>/dev/null | wc -l) -eq 0 || exit 13

echo "[4/5] Layer 4: Contracts..."
deal lint src/ || exit 14
pyright src/ || exit 14

echo "[5/5] Layer 5: Tests..."
pytest tests/ -v --cov=src --cov-fail-under=80 || exit 15

echo "=== ALL 5 LAYERS VERIFIED ==="
```

---

# VALIDATION GATES

| Gate | Layer | Command | Pass Criteria | Blocking |
|------|-------|---------|---------------|----------|
| Types | 1 | `idris2 --check` | No errors | Yes |
| Specs | 2 | `quint verify` | No violations | Yes |
| Proofs | 3 | `lake build` | Zero sorry | Yes |
| Contracts | 4 | `deal lint` | No warnings | Yes |
| Tests | 5 | `pytest --cov-fail-under=80` | All pass | Yes |
| Correspondence | All | Manual review | Matrix complete | Yes |

---

# EXIT CODES

| Code | Meaning |
|------|---------|
| 0 | All layers verified |
| 11 | Layer 1 (Types) failed |
| 12 | Layer 2 (Specs) failed |
| 13 | Layer 3 (Proofs) failed |
| 14 | Layer 4 (Contracts) failed |
| 15 | Layer 5 (Tests) failed |
| 16 | Correspondence incomplete |

---

# REQUIRED OUTPUT

## Planning Phase Output

1. **Layer Designs**
   - Layer 1-5 specifications from requirements

2. **Correspondence Matrix**
   - Same property mapped across all layers

3. **Artifact Summary**
   - Location and purpose of each artifact

## Execution Phase Output

1. **Layer 1 Artifacts** - `.outline/proofs/idris/src/*.idr`
2. **Layer 2 Artifacts** - `.outline/specs/*.qnt`
3. **Layer 3 Artifacts** - `.outline/proofs/lean/*.lean`
4. **Layer 4 Artifacts** - Annotated source code
5. **Layer 5 Artifacts** - Test files
6. **Correspondence Matrix** - All layers mapped
7. **Validation Report** - All gates passed



$ARGUMENTS
