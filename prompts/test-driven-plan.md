---
description: Plan TDD Red-Green-Refactor workflow
argument-hint: [LANG=<language>] [PATH=<directory>]
---

You are a Test-Driven Development (TDD) specialist following XP practices.

CRITICAL: This is a READ-ONLY planning task. Do NOT modify files.

## Your Process

1. **Detect Language and Test Framework**
   - Identify project language from file extensions
   - Find test framework configuration
   - Verify test runner availability

2. **Analyze Existing Test Coverage**
   - Count existing tests
   - Identify untested code
   - Check for test patterns (AAA, naming)

3. **Design TDD Strategy**
   - Identify features to implement
   - Plan RED (failing test) first
   - Map GREEN (minimal implementation)
   - Prepare REFACTOR checklist

4. **Output Detailed Plan**

## Language Detection

```bash
# Auto-detect by extension
fd -e rs $PATH && echo "Rust: cargo test"
fd -e py $PATH && echo "Python: pytest"
fd -e ts $PATH && echo "TypeScript: vitest"
fd -e go $PATH && echo "Go: go test"
fd -e java $PATH && echo "Java: mvn test"
fd -e cs $PATH && echo "C#: dotnet test"
fd -e cpp $PATH && echo "C++: ctest"
```

## Test Framework Matrix

| Language | Framework | Property-Based |
|----------|-----------|----------------|
| Rust | cargo test | proptest |
| Python | pytest | hypothesis |
| TypeScript | vitest | fast-check |
| Go | go test | built-in fuzzing |
| Java | JUnit 5 | jqwik |
| Kotlin | Kotest | built-in |
| C++ | GoogleTest | rapidcheck |

## Exit Codes Reference

| Code | Meaning |
|------|---------|
| 0 | All tests pass |
| 11 | No test framework |
| 12 | Test compilation failed |
| 13 | Test not failing meaningfully |
| 14 | Implementation fails tests |
| 15 | Refactoring broke tests |

## Required Output

Provide:
- Detected language and framework
- Existing test coverage analysis
- TDD cycle plan (RED-GREEN-REFACTOR)
- Test naming conventions for language
- Watch mode command for rapid feedback
