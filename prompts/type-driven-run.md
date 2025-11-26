---
description: Execute type-driven development: CREATE Idris 2 types from plan, VERIFY through type checking, IMPLEMENT target code
---

You are executing type-driven development using Idris 2. Your mission: CREATE the type specifications designed in the plan phase, VERIFY through type checking, then IMPLEMENT target language code.

## Philosophy: Create Types, Then Implement

This is the EXECUTION phase. The plan phase designed dependent types and proof obligations. Now you:
1. CREATE Idris 2 type definitions from plan design
2. VERIFY through type checking and totality
3. IMPLEMENT target code that honors the type contracts

## Constitutional Rules (Non-Negotiable)

1. **CREATE Types First**: All type definitions before implementation
2. **Types Never Lie**: If it doesn't type-check, fix implementation (not types)
3. **Holes Before Bodies**: Use ?holes, let type checker guide implementation
4. **Totality Enforced**: Mark functions total, prove termination

## Execution Workflow

### Phase 1: CREATE Type Artifacts (from plan)

```bash
# Create type specification directory
mkdir -p .outline/proofs

# Initialize Idris 2 package
cd .outline/proofs
pack new myproject
```

**Create Type Definitions (from plan):**
```idris
-- .outline/proofs/src/Types.idr
module Types

-- From plan: Positive natural number
public export
data Positive : Nat -> Type where
  MkPositive : Positive (S n)

-- From plan: Non-empty list
public export
data NonEmpty : List a -> Type where
  IsNonEmpty : NonEmpty (x :: xs)

-- From plan: Bounded value
public export
data Bounded : (lo : Nat) -> (hi : Nat) -> (n : Nat) -> Type where
  MkBounded : LTE lo n -> LTE n hi -> Bounded lo hi n
```

**Create State-Indexed Types (from plan):**
```idris
-- .outline/proofs/src/StateMachine.idr
module StateMachine

-- From plan: State enumeration
public export
data State = Initial | Processing | Complete | Failed

-- From plan: State-indexed type
public export
data Workflow : State -> Type where
  MkWorkflow : Workflow Initial

-- From plan: Type-safe transitions
public export
start : Workflow Initial -> Workflow Processing
start MkWorkflow = ?start_impl  -- Hole for type-driven implementation

public export
complete : Workflow Processing -> Workflow Complete

-- Invalid transitions are TYPE ERRORS:
-- skip : Workflow Initial -> Workflow Complete  -- Won't compile
```

**Create Type-Safe Operations (from plan):**
```idris
-- .outline/proofs/src/Operations.idr
module Operations

import Types

-- From plan: Safe head (requires non-empty proof)
public export
total
head : (xs : List a) -> {auto prf : NonEmpty xs} -> a
head (x :: xs) = x

-- From plan: Safe division (requires positive divisor)
public export
total
safeDiv : (x : Nat) -> (y : Nat) -> {auto prf : Positive y} -> Nat
safeDiv x (S k) = x `div` (S k)

-- From plan: Safe index (bounded by vector length)
public export
total
index : Fin n -> Vect n a -> a
index FZ (x :: xs) = x
index (FS i) (x :: xs) = index i xs
```

### Phase 2: VERIFY Through Type Checking

```bash
# Verify Idris 2 installation
command -v idris2 >/dev/null || exit 11

# Type check all modules
idris2 --check .outline/proofs/src/Types.idr || exit 13
idris2 --check .outline/proofs/src/StateMachine.idr || exit 13
idris2 --check .outline/proofs/src/Operations.idr || exit 13

# Verify totality
idris2 --total --check .outline/proofs/src/Operations.idr || exit 13

# Check for remaining holes
HOLE_COUNT=$(rg '\?' .outline/proofs/src/*.idr -c 2>/dev/null | awk -F: '{sum+=$2} END {print sum+0}')
echo "Remaining holes: $HOLE_COUNT"
```

### Phase 3: IMPLEMENT Target Code

**Map Idris types to target language:**

| Idris Type | TypeScript | Rust | Python |
|------------|-----------|------|--------|
| `Positive n` | Runtime check | `NonZeroU32` | Assert |
| `NonEmpty xs` | `[T, ...T[]]` | `Vec1<T>` | Assert |
| `Fin n` | Bounds check | `usize` bound | Assert |
| `State index` | Discriminated union | Enum + phantom | Enum |

**TypeScript Implementation:**
```typescript
// From Idris: Types encode in TypeScript

// Positive: Runtime enforcement
function safeDiv(x: number, y: number): number {
  if (y <= 0) throw new Error('Positive violation: divisor must be positive');
  return Math.floor(x / y);
}

// NonEmpty: Type-level enforcement
function head<T>(xs: [T, ...T[]]): T {
  return xs[0];  // Guaranteed non-empty by type
}

// State machine: Discriminated unions
type WorkflowState = 'initial' | 'processing' | 'complete' | 'failed';
type Workflow<S extends WorkflowState> = { state: S };

function start(w: Workflow<'initial'>): Workflow<'processing'> {
  return { state: 'processing' };
}
// start({ state: 'complete' })  // TYPE ERROR
```

**Property Tests from Types:**
```typescript
import fc from 'fast-check';

// From Idris type: Positive -> safe division
fc.assert(
  fc.property(fc.integer(), fc.integer({ min: 1 }), (x, y) => {
    const result = safeDiv(x, y);
    return typeof result === 'number' && Number.isFinite(result);
  })
);

// From Idris type: NonEmpty -> head never throws
fc.assert(
  fc.property(fc.array(fc.integer(), { minLength: 1 }), (xs) => {
    return head(xs as [number, ...number[]]) === xs[0];
  })
);
```

## Validation Gates

| Gate | Command | Pass Criteria | Blocking |
|------|---------|---------------|----------|
| Idris 2 | `command -v idris2` | Found | Yes |
| Type Check | `idris2 --check` | No errors | Yes |
| Totality | `idris2 --total` | All total | Yes |
| Holes | `rg '\?'` | Zero | Yes |
| Target Build | `tsc` / `cargo build` | Success | Yes |
| Property Tests | Test suite | All pass | Yes |

## Required Output

1. **Created Type Artifacts**
   - `.outline/proofs/src/Types.idr`
   - `.outline/proofs/src/StateMachine.idr`
   - `.outline/proofs/src/Operations.idr`

2. **Type Check Results**
   - All modules: PASS/FAIL
   - Totality: PASS/FAIL
   - Holes remaining: 0

3. **Target Implementation**
   - Type correspondence table
   - Property tests from types

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Types verified, implementation complete |
| 11 | Idris 2 not installed |
| 12 | No .idr files created |
| 13 | Type check or totality failed |
| 14 | Holes remaining |
| 15 | Target implementation failed |

Execute CREATE -> VERIFY -> IMPLEMENT cycle.
