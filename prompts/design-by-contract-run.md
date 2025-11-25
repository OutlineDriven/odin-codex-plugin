---
description: Execute Design-by-Contract verification
argument-hint: [LANG=<language>] [PATH=<directory>]
---

You are executing Design-by-Contract verification across multiple languages.

## Arguments

- `$LANG` - Programming language (optional, auto-detected if omitted)
- `$PATH` - Directory path containing contracts (required)

## Execution Steps

1. **DETECT**: Find contract usage in codebase
2. **VERIFY**: Check all contracts satisfied
3. **REMEDIATE**: Add missing contracts or fix violations

## Commands by Language

### Rust
```bash
# Detect
rg '#\[pre\(|#\[post\(|debug_assert!' --type rust $PATH

# Verify (debug build enables assertions)
cargo build && cargo test

# Check runtime flags
[[ "$CARGO_BUILD_TYPE" != "release" ]] || echo "Warning: debug assertions disabled"
```

### TypeScript
```bash
# Detect
rg 'z\.object|invariant\(|\.parse\(' --type ts $PATH

# Verify
npx vitest run

# Runtime check
[[ "$NODE_ENV" == "development" ]] || echo "Warning: not in dev mode"
```

### Python
```bash
# Detect
rg '@pre\(|@post\(|@invariant' --type py $PATH

# Verify
python -m pytest

# Runtime check (not -O optimized)
python -c "assert __debug__, 'assertions disabled'"
```

### Java
```bash
# Detect
rg 'checkArgument|checkState' --type java $PATH

# Verify
mvn test

# Check assertions enabled
java -ea -version 2>&1 | grep -q "enabled" || echo "Use: java -ea"
```

### Kotlin
```bash
# Detect
rg 'contract \{|require\(|check\(' --type kotlin $PATH

# Verify
./gradlew test

# Runtime check
java -ea -version
```

### C#
```bash
# Detect
rg 'Guard\.Against|Contract\.' --type cs $PATH

# Verify
dotnet test

# Check Debug configuration
dotnet build -c Debug
```

### C++
```bash
# Detect
rg 'Expects\(|Ensures\(' --type cpp $PATH

# Verify (NDEBUG not defined)
cmake -DCMAKE_BUILD_TYPE=Debug .. && make && ctest

# Check NDEBUG
grep -r "NDEBUG" $PATH && echo "Warning: assertions may be disabled"
```

### C
```bash
# Detect
rg 'assert\(' --type c $PATH

# Verify
make debug && ./test_runner

# Check NDEBUG
grep -r "NDEBUG" $PATH && echo "Warning: assertions may be disabled"
```

## Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | All contracts satisfied |
| 1 | Precondition | Fix caller to provide valid inputs |
| 2 | Postcondition | Fix function to return valid results |
| 3 | Invariant | Fix state mutation logic |
| 11 | No contracts | Add contracts to public APIs |
| 13 | Flags disabled | Enable debug/assertion flags |

## Contract Templates

### Precondition (input validation)
```
Precondition: x must be positive (got: {x})
```

### Postcondition (output guarantee)
```
Postcondition: result must exceed input (result={r}, input={x})
```

### Invariant (state consistency)
```
Invariant: balance must be non-negative (balance={b})
```

## Workflow

```
DETECT contracts
  |
  v
VERIFY (exit 1-3 on violations)
  |
  v
REMEDIATE missing/violated
  |
  v
RE-VERIFY
  |
  v
SUCCESS (exit 0)
```

Execute detection, then verification. Report violations with context.
