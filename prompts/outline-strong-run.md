---
description: Execute unified validation: CREATE all 5 layers from plan, VERIFY each layer, INTEGRATE
argument-hint: <request>
---

You are executing OUTLINE-STRONG DEVELOPMENT - unified validation-first methodology. Your mission: CREATE validation artifacts for all five layers designed in the plan phase, VERIFY each layer passes, then INTEGRATE into cohesive implementation.

## Philosophy: Defense in Depth Through Creation

This is the EXECUTION phase. The plan phase designed the complete verification stack. Now you:
1. CREATE artifacts for all five layers
2. VERIFY each layer independently
3. INTEGRATE layers into target implementation
4. VALIDATE cross-layer correspondence

## Verification Stack

```
Layer | Tool      | Creates           | Catches
------|-----------|-------------------|------------------
1     | Idris 2   | Type specs        | Structural errors
2     | Quint     | State specs       | Design flaws
3     | Lean 4    | Formal proofs     | Invariant violations
4     | deal/etc  | Contract annotations | Runtime violations
5     | pytest/etc| Test suites       | Behavioral bugs
```

## Constitutional Rules (Non-Negotiable)

1. **CREATE All Layers**: Generate artifacts for each layer from plan
2. **VERIFY In Order**: Types -> Specs -> Proofs -> Contracts -> Tests
3. **Layer Correspondence**: Explicit mapping between all layers
4. **No Skipping**: Each layer gate must pass before next
5. **Variance < 2%**: Outline-to-implementation deviation minimal

## Execution Workflow

### Phase 1: CREATE Layer 1 - Types (Idris 2)

```bash
mkdir -p .outline/proofs/idris
cd .outline/proofs/idris && pack new ProjectTypes
```

```idris
-- .outline/proofs/idris/src/Types.idr
-- From plan: Type definitions

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

### Phase 2: CREATE Layer 2 - Specs (Quint)

```bash
mkdir -p .outline/specs
```

```quint
// .outline/specs/model.qnt
// From plan: State model and invariants

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

### Phase 3: CREATE Layer 3 - Proofs (Lean 4)

```bash
mkdir -p .outline/proofs/lean
cd .outline/proofs/lean && lake new ProjectProofs
```

```lean
-- .outline/proofs/lean/ProjectProofs/Basic.lean
-- From plan: Theorem statements

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

### Phase 4: CREATE Layer 4 - Contracts

```python
# src/account.py
# From plan: Contract specifications

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

### Phase 5: CREATE Layer 5 - Tests

```python
# tests/test_account.py
# From plan: Test scenarios

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

### Phase 6: INTEGRATE and Verify Correspondence

**Correspondence Matrix Verification:**
```markdown
| Property | Idris | Quint | Lean | Contract | Test |
|----------|-------|-------|------|----------|------|
| balance >= 0 | Nat | inv_balance | preserves_inv | @deal.inv | test_invariant |
| amount > 0 | Positive | precond | h_pos | @deal.pre | test_positive |
| amount <= bal | LTE | guard | h_suff | @deal.pre | test_insufficient |
```

## Full Validation Pipeline

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

## Validation Gates

| Gate | Layer | Command | Pass Criteria | Blocking |
|------|-------|---------|---------------|----------|
| Types | 1 | `idris2 --check` | No errors | Yes |
| Specs | 2 | `quint verify` | No violations | Yes |
| Proofs | 3 | `lake build` | Zero sorry | Yes |
| Contracts | 4 | `deal lint` | No warnings | Yes |
| Tests | 5 | `pytest --cov-fail-under=80` | All pass | Yes |
| Correspondence | All | Manual review | Matrix complete | Yes |

## Required Output

1. **Layer 1 Artifacts** - `.outline/proofs/idris/src/*.idr`
2. **Layer 2 Artifacts** - `.outline/specs/*.qnt`
3. **Layer 3 Artifacts** - `.outline/proofs/lean/*.lean`
4. **Layer 4 Artifacts** - Annotated source code
5. **Layer 5 Artifacts** - Test files
6. **Correspondence Matrix** - All layers mapped
7. **Validation Report** - All gates passed

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All layers verified |
| 11 | Layer 1 (Types) failed |
| 12 | Layer 2 (Specs) failed |
| 13 | Layer 3 (Proofs) failed |
| 14 | Layer 4 (Contracts) failed |
| 15 | Layer 5 (Tests) failed |
| 16 | Correspondence incomplete |

Execute CREATE for each layer -> VERIFY each layer -> INTEGRATE with correspondence validation.



$ARGUMENTS
