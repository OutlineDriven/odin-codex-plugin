---
description: Execute TDD: CREATE tests from plan, RED state, GREEN implementation, REFACTOR
---

You are executing Test-Driven Development using the Red-Green-Refactor cycle. Your mission: CREATE the tests designed in the plan phase, achieve RED (failing), then GREEN (passing), then REFACTOR.

## Philosophy: Create Tests First, Then Implement

This is the EXECUTION phase. The plan phase designed test cases. Now you:
1. CREATE test files from plan design (expect RED)
2. Achieve GREEN through minimal implementation
3. REFACTOR while keeping tests green

## Constitutional Rules (Non-Negotiable)

1. **CREATE Tests First**: Write all tests before implementation
2. **RED Before GREEN**: Tests must fail initially (proves test works)
3. **Minimal GREEN**: Only enough code to pass tests
4. **REFACTOR Safely**: Tests stay green during cleanup

## Execution Workflow

### Phase 1: CREATE Test Files (from plan)

**Priority Order from Plan:**
1. Error cases (Priority 1)
2. Edge cases (Priority 2)
3. Happy paths (Priority 3)
4. Property tests (Priority 4)

**Python Example:**
```python
# tests/test_account.py
# From plan: Test designs

import pytest
from hypothesis import given, strategies as st

class TestAccountFromPlan:
    """Tests designed in plan phase"""

    # Error Cases (Priority 1)
    def test_withdraw_rejects_negative_amount(self):
        """From plan: ERR-1"""
        account = Account(balance=100)
        with pytest.raises(ValueError, match="amount must be positive"):
            account.withdraw(-50)

    def test_withdraw_rejects_insufficient_balance(self):
        """From plan: ERR-2"""
        account = Account(balance=100)
        with pytest.raises(ValueError, match="insufficient balance"):
            account.withdraw(150)

    # Edge Cases (Priority 2)
    def test_withdraw_handles_exact_balance(self):
        """From plan: EDGE-1"""
        account = Account(balance=100)
        result = account.withdraw(100)
        assert result == 100
        assert account.balance == 0

    def test_withdraw_handles_minimum_amount(self):
        """From plan: EDGE-2"""
        account = Account(balance=100)
        result = account.withdraw(1)
        assert result == 1

    # Happy Paths (Priority 3)
    def test_withdraw_returns_amount(self):
        """From plan: HAPPY-1"""
        account = Account(balance=100)
        result = account.withdraw(30)
        assert result == 30

    # Property Tests (Priority 4)
    @given(st.integers(min_value=0, max_value=10000),
           st.integers(min_value=1, max_value=100))
    def test_balance_never_negative(self, initial, amount):
        """From plan: PROP-1"""
        from hypothesis import assume
        assume(amount <= initial)
        account = Account(balance=initial)
        account.withdraw(amount)
        assert account.balance >= 0
```

### Phase 2: Achieve RED State

```bash
# Run tests - expect failures
pytest tests/ -v

# Verify tests actually fail (RED state confirmed)
pytest tests/ && echo "ERROR: Tests should fail!" && exit 13
echo "RED state achieved - tests fail as expected"
```

### Phase 3: Achieve GREEN State

**Implement minimal code to pass tests:**
```python
# src/account.py
# Minimal implementation to pass tests

class Account:
    def __init__(self, balance: int):
        self.balance = balance

    def withdraw(self, amount: int) -> int:
        # From ERR-1
        if amount <= 0:
            raise ValueError("amount must be positive")
        # From ERR-2
        if amount > self.balance:
            raise ValueError("insufficient balance")

        self.balance -= amount
        return amount
```

```bash
# Run tests - expect all pass
pytest tests/ -v || exit 14
echo "GREEN state achieved - all tests pass"
```

### Phase 4: REFACTOR

```bash
# Clean up code while keeping tests green
# After each refactor:
pytest tests/ || exit 15
echo "REFACTOR complete - tests still green"
```

## Test Framework Commands

| Language | Run Tests | Watch Mode | Coverage |
|----------|-----------|------------|----------|
| Python | `pytest` | `pytest-watch` | `pytest --cov` |
| Rust | `cargo test` | `cargo watch -x test` | `cargo tarpaulin` |
| TypeScript | `vitest run` | `vitest --watch` | `vitest --coverage` |
| Go | `go test ./...` | `gotestsum --watch` | `go test -cover` |
| Java | `mvn test` | - | `mvn jacoco:report` |
| Kotlin | `./gradlew test` | `./gradlew -t test` | `./gradlew jacocoTestReport` |

## Validation Gates

| Gate | Command | Pass Criteria | Blocking |
|------|---------|---------------|----------|
| Framework | Detect test framework | Found | Yes |
| RED | Initial test run | Tests fail | Yes |
| GREEN | Post-implementation | Tests pass | Yes |
| REFACTOR | Post-cleanup | Tests still pass | Yes |
| Coverage | Coverage report | >= 80% | No |

## Required Output

1. **Created Test Files**
   - `tests/test_[module].py` or equivalent
   - Error case tests
   - Edge case tests
   - Happy path tests
   - Property tests

2. **TDD Cycle Report**
   - RED: [X] tests failing
   - GREEN: All tests passing
   - REFACTOR: Code cleaned up

3. **Coverage Summary**
   - Lines covered
   - Branch coverage
   - Uncovered areas

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | TDD cycle complete, all tests pass |
| 11 | No test framework detected |
| 12 | Test compilation failed |
| 13 | Tests not failing (RED state not achieved) |
| 14 | Tests fail after implementation (GREEN not achieved) |
| 15 | Tests fail after refactor (regression) |

Execute CREATE -> RED -> GREEN -> REFACTOR cycle.
