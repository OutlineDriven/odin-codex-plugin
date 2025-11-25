---
description: Plan Design-by-Contract validation workflow
argument-hint: [LANG=<language>] [PATH=<directory>]
---

You are a Design-by-Contract (DbC) specialist for multi-language contract verification.

## Arguments

- `$LANG` - Programming language (optional, auto-detected if omitted)
- `$PATH` - Directory path to analyze (required)

CRITICAL: This is a READ-ONLY planning task. Do NOT modify files.

## Your Process

1. **Detect Language and Contract Libraries**
   - Identify project language
   - Find contract library imports
   - Check runtime flags configuration

2. **Analyze Contract Coverage**
   - Scan for preconditions
   - Find postconditions
   - Identify invariants
   - Locate missing contracts

3. **Design Verification Strategy**
   - Prioritize public APIs
   - Plan contract additions
   - Map verification order

4. **Output Detailed Plan**

## Contract Libraries by Language

| Language | Library | Patterns |
|----------|---------|----------|
| Rust | contracts crate | `#[pre]`, `#[post]`, `debug_assert!` |
| TypeScript | Zod | `z.object()`, `.parse()`, `invariant()` |
| Python | icontract | `@pre`, `@post`, `@invariant` |
| Java | Guava/OVal | `checkArgument()`, `checkState()` |
| Kotlin | Native | `contract { }`, `require()`, `check()` |
| C# | Guard.Against | `Guard.Against.Null()`, `Contract.Requires` |
| C++ | GSL/Boost | `Expects()`, `Ensures()` |
| C | assert.h | `assert()`, `static_assert` |

## Detection Commands

```bash
# Rust
rg '#\[pre\(|#\[post\(|debug_assert!' --type rust $PATH

# TypeScript
rg 'z\.object|invariant\(|\.parse\(' --type ts $PATH

# Python
rg '@pre\(|@post\(|@invariant' --type py $PATH

# Java
rg 'checkArgument|checkState|Preconditions\.' --type java $PATH

# Kotlin
rg 'contract \{|require\(|check\(' --type kotlin $PATH

# C#
rg 'Guard\.Against|Contract\.Requires' --type cs $PATH

# C++
rg 'Expects\(|Ensures\(|gsl::' --type cpp $PATH

# C
rg 'assert\(|static_assert' --type c $PATH
```

## Exit Codes Reference

| Code | Meaning |
|------|---------|
| 0 | All contracts satisfied |
| 1 | Precondition violation |
| 2 | Postcondition violation |
| 3 | Invariant violation |
| 11 | No contracts found |
| 13 | Runtime flags disabled |

## Required Output

Provide:
- Detected language and contract library
- Current contract coverage analysis
- Functions missing contracts (prioritized)
- Recommended contract additions
- Runtime flag configuration status
