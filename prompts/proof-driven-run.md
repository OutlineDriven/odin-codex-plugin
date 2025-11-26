---
description: Execute proof-driven validation: CREATE Lean 4 proofs from plan, VERIFY, then REMEDIATE
---

You are executing proof-driven validation using Lean 4 / Lake. Your mission: CREATE the proofs designed in the plan phase, VERIFY through compilation, then REMEDIATE any incomplete proofs.

## Philosophy: Create Proofs, Then Validate

This is the EXECUTION phase. The plan phase designed theorems and lemmas. Now you:
1. CREATE Lean 4 proof files from plan design
2. VERIFY through `lake build`
3. REMEDIATE `sorry` placeholders until zero remain

## Constitutional Rules (Non-Negotiable)

1. **CREATE First**: Generate all theorem/lemma files from plan
2. **No Sorry Allowed**: Final code must have zero `sorry`
3. **Proofs Must Build**: `lake build` must succeed
4. **Complete Coverage**: All planned theorems implemented

## Execution Workflow

### Phase 1: CREATE Proof Artifacts (from plan)

```bash
# Create proof directory structure
mkdir -p .outline/proofs/lean

# Initialize Lake project (if not exists)
cd .outline/proofs/lean
test -f lakefile.lean || lake new ProjectProofs
```

**Create Theorem Files (from plan design):**
```lean
-- .outline/proofs/lean/ProjectProofs/Basic.lean
-- From plan: Theorem designs

namespace Project

/-- From plan: Property description -/
theorem theorem_name
    (params : Types)
    (hypotheses : Conditions) :
    Conclusion := by
  sorry  -- Initial placeholder, to be completed

/-- From plan: Supporting lemma -/
lemma lemma_name
    (params : Types) :
    Statement := by
  sorry  -- Initial placeholder

end Project
```

### Phase 2: VERIFY Through Compilation

```bash
cd .outline/proofs/lean

# Verify toolchain
command -v lake >/dev/null || exit 11
command -v lean >/dev/null || exit 11

# Build proofs
lake build || exit 13

# Check for incomplete proofs
SORRY_COUNT=$(rg '\bsorry\b' --type-add 'lean:*.lean' -t lean -c 2>/dev/null | awk -F: '{sum+=$2} END {print sum+0}')
echo "Sorry count: $SORRY_COUNT"
```

### Phase 3: REMEDIATE Until Complete

**Replace each `sorry` with actual proof:**
```lean
-- Before (placeholder)
theorem withdraw_preserves_balance
    (balance : Nat) (amount : Nat)
    (h_suff : amount <= balance) :
    (balance - amount) >= 0 := by
  sorry

-- After (completed)
theorem withdraw_preserves_balance
    (balance : Nat) (amount : Nat)
    (h_suff : amount <= balance) :
    (balance - amount) >= 0 := by
  omega
```

**Iterate until zero sorry:**
```bash
while [ "$SORRY_COUNT" -gt 0 ]; do
  # Complete proofs using tactics
  # Rebuild and recount
  lake build || exit 13
  SORRY_COUNT=$(rg '\bsorry\b' -t lean -c 2>/dev/null | awk -F: '{sum+=$2} END {print sum+0}')
done
```

## Common Tactics Reference

| Tactic | Use Case |
|--------|----------|
| `simp` | Simplify with known lemmas |
| `omega` | Linear arithmetic |
| `aesop` | Automated proof search |
| `rw [h]` | Rewrite using hypothesis |
| `exact h` | Provide exact term |
| `intro h` | Introduce hypothesis |
| `cases h` | Case split |
| `induction n` | Inductive proof |
| `constructor` | Build inductive type |
| `rfl` | Definitional equality |

## Validation Gates

| Gate | Command | Pass Criteria | Blocking |
|------|---------|---------------|----------|
| Toolchain | `command -v lake` | Found | Yes |
| Build | `lake build` | Success | Yes |
| Sorry Count | `rg '\bsorry\b'` | Zero | Yes |
| Tests | `lake test` | All pass | If present |

## Required Output

1. **Created Artifacts**
   - `.outline/proofs/lean/lakefile.lean`
   - `.outline/proofs/lean/ProjectProofs/*.lean`

2. **Verification Status**
   - Build result: PASS/FAIL
   - Sorry count: 0 (required)
   - Theorems completed: [list]

3. **Proof Summary**
   - Main theorems proved
   - Supporting lemmas
   - Tactics used

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All proofs verified, zero sorry |
| 11 | lean/lake not found |
| 12 | No .lean files created |
| 13 | Build failed or proofs incomplete |
| 14 | Coverage gaps (theorems missing) |

Execute CREATE -> VERIFY -> REMEDIATE until all proofs complete.
