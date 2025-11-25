---
description: Execute spec-first verification: CREATE Quint specs from plan, VERIFY, then IMPLEMENT
argument-hint: [PATH=<directory>] [MODULE=<module-name>]
---

You are executing specification-first verification using Quint. Your mission: CREATE the specifications designed in the plan phase, VERIFY through Quint, then IMPLEMENT target code.

## Arguments

- `$PATH` - Directory path for specification artifacts (required)
- `$MODULE` - Main module name for verification (required for verify command)

## Philosophy: Create Specifications, Then Validate

This is the EXECUTION phase. The plan phase designed state machines and invariants. Now you:
1. CREATE Quint specification files from plan design
2. VERIFY through `quint verify`
3. IMPLEMENT target code that honors the specification

## Constitutional Rules (Non-Negotiable)

1. **CREATE First**: Generate all .qnt files from plan
2. **Invariants Must Hold**: All invariants verified
3. **Actions Must Type**: All actions type-check
4. **Implementation Follows Spec**: Target code mirrors specification

## Execution Workflow

### Phase 1: CREATE Specification Artifacts (from plan)

```bash
# Create spec directory
mkdir -p .outline/specs
```

**Create Type Definitions (from plan):**
```quint
// .outline/specs/types.qnt
// From plan: Type designs

module types {
  // Domain types from plan
  type EntityId = str
  type Amount = int
  type Status = Pending | Active | Complete

  // Compound types from plan
  type Entity = {
    id: EntityId,
    value: Amount,
    status: Status
  }
}
```

**Create State Module (from plan):**
```quint
// .outline/specs/state.qnt
module state {
  import types.*

  // State variables from plan
  var entities: EntityId -> Entity
  var totalValue: Amount

  // Initial state from plan
  val init = all {
    entities' = Map(),
    totalValue' = 0
  }
}
```

**Create Actions (from plan):**
```quint
// .outline/specs/actions.qnt
module actions {
  import state.*

  // Action from plan: ACT-1
  action create(id: EntityId, value: Amount): bool = all {
    value > 0,
    not(entities.keys().contains(id)),
    entities' = entities.put(id, { id: id, value: value, status: Pending }),
    totalValue' = totalValue + value
  }

  // Action from plan: ACT-2
  action activate(id: EntityId): bool = all {
    entities.keys().contains(id),
    entities.get(id).status == Pending,
    entities' = entities.setBy(id, e => { ...e, status: Active }),
    totalValue' = totalValue
  }
}
```

**Create Invariants (from plan):**
```quint
// .outline/specs/invariants.qnt
module invariants {
  import state.*

  // Invariant from plan: INV-1
  val inv_valueNonNegative = entities.keys().forall(id =>
    entities.get(id).value >= 0
  )

  // Invariant from plan: INV-2
  val inv_totalConsistent = totalValue ==
    entities.keys().fold(0, (acc, id) => acc + entities.get(id).value)
}
```

**Create Main Module:**
```quint
// .outline/specs/main.qnt
module main {
  import types.*
  import state.*
  import actions.*
  import invariants.*

  // Compose all modules
  val step = any {
    nondet id = oneOf(Set("a", "b", "c"))
    nondet amount = oneOf(1.to(100))
    create(id, amount),
    nondet id = oneOf(entities.keys())
    activate(id)
  }
}
```

### Phase 2: VERIFY Specifications

```bash
# Verify Quint installation
command -v quint >/dev/null || exit 11

# Type check all specs
quint typecheck .outline/specs/*.qnt || exit 12

# Verify invariants
quint verify --main=main --invariant=inv_valueNonNegative .outline/specs/main.qnt || exit 13
quint verify --main=main --invariant=inv_totalConsistent .outline/specs/main.qnt || exit 13

# Run tests if defined
quint test .outline/specs/*.qnt || exit 14
```

### Phase 3: IMPLEMENT Target Code

**Generate implementation stubs from verified spec:**

```python
# From Quint spec: actions.create
def create(id: str, value: int) -> bool:
    """
    From spec invariants:
    - PRE: value > 0
    - PRE: id not in entities
    - POST: entities[id].value == value
    - POST: total_value == old(total_value) + value
    """
    if value <= 0:
        raise ValueError("INV: value must be positive")
    if id in entities:
        raise ValueError("PRE: id already exists")

    entities[id] = Entity(id=id, value=value, status=Status.PENDING)
    global total_value
    total_value += value
    return True
```

## Validation Gates

| Gate | Command | Pass Criteria | Blocking |
|------|---------|---------------|----------|
| Quint | `command -v quint` | Found | Yes |
| Typecheck | `quint typecheck` | No errors | Yes |
| Invariants | `quint verify` | All hold | Yes |
| Tests | `quint test` | All pass | If present |

## Required Output

1. **Created Artifacts**
   - `.outline/specs/types.qnt`
   - `.outline/specs/state.qnt`
   - `.outline/specs/actions.qnt`
   - `.outline/specs/invariants.qnt`
   - `.outline/specs/main.qnt`

2. **Verification Status**
   - Typecheck: PASS/FAIL
   - Each invariant: PASS/FAIL
   - Test results (if any)

3. **Implementation Mapping**
   - Spec action -> Target function
   - Spec invariant -> Runtime assertion

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Specification verified, ready for implementation |
| 11 | Quint not installed |
| 12 | Syntax/type errors in specification |
| 13 | Invariant violation detected |
| 14 | Specification tests failed |
| 15 | Implementation incomplete |

Execute CREATE -> VERIFY -> IMPLEMENT cycle.
