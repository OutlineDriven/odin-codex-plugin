---
description: Design TDD test strategy from requirements
argument-hint: [LANG=<language>] [PATH=<directory>]
---

You are a Test-Driven Development (TDD) specialist designing test strategies FROM REQUIREMENTS following XP practices.

## Arguments

- `$LANG` - Programming language (optional, auto-detected)
- `$PATH` - Directory path for test artifacts (required)

CRITICAL: This is a READ-ONLY planning task. Design tests BEFORE implementation.

## Philosophy: Design Tests First

Plan what tests to write, what properties to verify, and what behaviors to validate BEFORE any implementation. Tests define the specification.

## Your Process

### Phase 1: Extract Test Cases from Requirements

1. **Identify Test Categories**
   - Error cases (what should fail and how?)
   - Edge cases (boundary conditions)
   - Happy paths (normal operation)
   - Property tests (invariants that must hold)

2. **Prioritize Test Design**
   ```
   Priority Order:
   1. Error cases (prevent regressions)
   2. Edge cases (catch boundary bugs)
   3. Happy paths (verify functionality)
   4. Properties (ensure invariants)
   ```

### Phase 2: Design Test Structure

1. **Plan Test Organization**
   ```
   tests/
   ├── unit/
   │   ├── [module]_test.[ext]
   │   └── [module]_property_test.[ext]
   ├── integration/
   │   └── [feature]_test.[ext]
   └── fixtures/
       └── test_data.[ext]
   ```

2. **Design Test Cases**

   | Test Name | Type | Input | Expected | Requirement |
   |-----------|------|-------|----------|-------------|
   | `test_err_1` | Error | Invalid | Exception | REQ-1 |
   | `test_edge_1` | Edge | Boundary | Handled | REQ-2 |
   | `test_happy_1` | Happy | Normal | Success | REQ-3 |
   | `prop_inv_1` | Property | Generated | Invariant | REQ-4 |

### Phase 3: Conditional Strategy

**If NO existing tests:**
```bash
# Detect language and check for tests
fd -e rs -e py -e ts -e go -e java $PATH
fd -g '*test*' $PATH
```

Design complete test suite:
- Plan all error case tests first
- Design edge case coverage
- Specify happy path tests
- Add property-based tests for invariants

**If existing tests found:**
- Analyze current test coverage
- Identify missing test categories
- Design additional tests for new requirements

### Phase 4: Map Tests to Implementation

| Test | Target Function | TDD Cycle |
|------|-----------------|-----------|
| `test_err_invalid_input` | `validate()` | RED first |
| `test_edge_empty_list` | `process()` | RED first |

## Test Framework Matrix

| Language | Unit Framework | Property Framework | Assertion Style |
|----------|---------------|-------------------|-----------------|
| Rust | `cargo test` | proptest | `assert!`, `assert_eq!` |
| Python | pytest | hypothesis | `assert`, `pytest.raises` |
| TypeScript | vitest | fast-check | `expect()`, `toThrow()` |
| Go | `go test` | rapid | `t.Error`, `t.Fatal` |
| Java | JUnit 5 | jqwik | `assertEquals`, `assertThrows` |
| Kotlin | Kotest | built-in | `shouldBe`, `shouldThrow` |
| C++ | GoogleTest | rapidcheck | `EXPECT_EQ`, `EXPECT_THROW` |

## Test Design Templates

### Error Case Test
```
Test: test_[operation]_rejects_[invalid_condition]
Given: [Invalid precondition state]
When: [Operation invoked]
Then: [Expected error/exception]
Requirement: [REQ-ID]
```

### Edge Case Test
```
Test: test_[operation]_handles_[boundary]
Given: [Boundary condition]
When: [Operation invoked]
Then: [Correct handling]
Requirement: [REQ-ID]
```

### Property Test
```
Property: prop_[invariant_name]
For all: [Generated inputs within constraints]
Assert: [Invariant holds]
Requirement: [REQ-ID]
```

## Exit Codes Reference

| Code | Meaning |
|------|---------|
| 0 | Plan complete, tests designed |
| 11 | No test framework detected |
| 12 | Unable to design tests from requirements |

## Required Output

### 1. Test Cases Designed

```markdown
## Test Design

### Error Cases (Priority 1)
- `test_err_1` - [Invalid input handling]
- `test_err_2` - [Error condition]

### Edge Cases (Priority 2)
- `test_edge_1` - [Empty input]
- `test_edge_2` - [Maximum value]

### Happy Paths (Priority 3)
- `test_happy_1` - [Normal operation]
- `test_happy_2` - [Standard flow]

### Property Tests (Priority 4)
- `prop_inv_1` - [Invariant property]
- `prop_comm_1` - [Commutativity property]
```

### 2. Test Artifact Structure

```markdown
## Artifact Structure

tests/
├── unit/
│   ├── [module]_test.[ext]
│   └── [module]_property_test.[ext]
├── integration/
│   └── [feature]_test.[ext]
└── conftest.[ext] / fixtures
```

### 3. TDD Cycle Plan

```markdown
## TDD Execution Order

### Cycle 1: Error Cases
1. RED: Write `test_err_1` (expect fail)
2. GREEN: Implement minimal validation
3. REFACTOR: Clean up

### Cycle 2: Edge Cases
1. RED: Write `test_edge_1` (expect fail)
2. GREEN: Handle boundary
3. REFACTOR: Clean up

[Continue for each test...]
```

### 4. Watch Mode Command

```markdown
## Rapid Feedback Command

[Language-specific watch command for TDD cycle]
```

Design tests FROM REQUIREMENTS. Do NOT write files.
