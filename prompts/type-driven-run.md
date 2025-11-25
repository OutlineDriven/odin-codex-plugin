---
description: Execute type-driven validation with Idris 2
argument-hint: [PATH=<directory>]
---

You are executing type-driven validation using Idris 2.

## Arguments

- `$PATH` - Directory path containing .idr/.ipkg files (required)

## Execution Steps

1. **CHECK**: Verify preconditions
2. **VALIDATE**: Build and type-check all packages/files
3. **GENERATE**: Run diagnostics with timing
4. **REMEDIATE**: Fix totality and coverage warnings

## Commands (Tiered)

### Basic (Precondition Check)
```bash
command -v idris2 >/dev/null || exit 11
fd -e ipkg -e idr $PATH >/dev/null || exit 12
```

### Intermediate (Validation)
```bash
fd -e ipkg $PATH -x idris2 --build {} || exit 13
fd -e idr $PATH -x idris2 --check {} || exit 13
```

### Advanced (Full Diagnostics)
```bash
fd -e ipkg $PATH -x idris2 --build {} --timing
rg -n 'maybe not total|covering' $PATH && exit 14 || exit 0
```

## Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | All validations pass |
| 11 | idris2 not found | Install Idris 2 |
| 12 | No artifacts | Check path or create .idr files |
| 13 | Type errors | Fix type mismatches, fill holes |
| 14 | Totality gaps | Add `total` annotations, exhaust patterns |

## Remediation Tips

- Add `total` annotations to enforce totality
- Use exhaustive pattern matching (no wildcards for total functions)
- Split partial functions into total helpers
- Fill metas explicitly; avoid `%hint` deferrals
- Use `%default total` at module level

## Workflow

```
CHECK (exit 11/12 on fail)
  |
  v
VALIDATE (exit 13 on type errors)
  |
  v
GENERATE (timing info)
  |
  v
REMEDIATE (exit 14 for totality gaps)
  |
  v
SUCCESS (exit 0)
```

Execute commands in order. Stop on first non-zero exit code and report.
