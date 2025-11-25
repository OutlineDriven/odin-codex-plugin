---
description: Execute TDD Red-Green-Refactor cycle
argument-hint: [LANG=<language>] [PATH=<directory>]
---

You are executing Test-Driven Development using the Red-Green-Refactor cycle.

## Arguments

- `$LANG` - Programming language (optional, auto-detected if omitted)
- `$PATH` - Directory path containing tests (required)

## Execution Steps

1. **RED**: Write a failing test
2. **GREEN**: Implement minimal code to pass
3. **REFACTOR**: Clean up while keeping tests green

## Commands by Language

### Rust
```bash
# Basic
cargo test

# With coverage
cargo test && cargo tarpaulin --out Lcov

# Watch mode
cargo watch -x test
```

### Python
```bash
# Basic
pytest

# With coverage
pytest --cov=. --cov-report=term

# Watch mode
pytest-watch
```

### TypeScript
```bash
# Basic
vitest run

# With coverage
vitest --coverage

# Watch mode
vitest --watch
```

### Go
```bash
# Basic
go test ./...

# With coverage
go test -v -cover ./...

# Watch mode
gotestsum --watch
```

### Java
```bash
# Basic
mvn test

# Specific test
mvn test -Dtest=TestClass#method

# With mutation testing
mvn verify pitest:mutationCoverage
```

### C#
```bash
# Basic
dotnet test

# With coverage
dotnet test --collect:"XPlat Code Coverage"

# Watch mode
dotnet watch test
```

### C++
```bash
# Basic
ctest

# Verbose
ctest --verbose --output-on-failure

# Repeat for flaky detection
ctest --repeat until-fail:10 --parallel 4
```

## Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Tests pass | Proceed to next feature |
| 11 | No framework | Install test framework |
| 12 | Compile error | Fix syntax |
| 13 | Wrong failure | Refine test expectations |
| 14 | Test fails | Fix implementation |
| 15 | Regression | Revert, refactor carefully |

## Best Practices

### Test Naming
- Rust: `test_<feature>_<condition>_<expected>`
- Python: `test_<feature>_when_<condition>_then_<expected>`
- TypeScript: `'should <expected> when <condition>'`
- Go: `Test<Feature><Condition>`

### AAA Pattern
```
ARRANGE: Set up test data
ACT: Execute the function
ASSERT: Verify the result
```

### One Assertion Per Test
Keep tests focused on single behaviors.

## Workflow

```
RED (write failing test)
  |
  v
GREEN (minimal implementation)
  |
  v
REFACTOR (clean code, tests stay green)
  |
  v
REPEAT
```

Execute the cycle. Report test results and coverage metrics.
