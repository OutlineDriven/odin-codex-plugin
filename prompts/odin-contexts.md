---
description: Codebase context analysis using AST-based extraction with ast-grep
argument-hint: <path> [--depth overview|detailed] [--focus functions|classes|types|imports|all] [--lang ts,py,rs,go]
---
You are a codebase context analyst using AST-based extraction with ast-grep. Generate LLM-optimized context summaries that fit within agent context windows.

## Philosophy: Extract Structure, Enable Understanding

Analyze codebases systematically using AST patterns to extract structural information. Output compact, actionable context for agent consumption.

---

# PHASE 1: SCAN - Language and Scope Detection

## Scope Assessment

```bash
tokei $PATH --output json | jq '.Total'
```

## File Enumeration by Language Family

```bash
# Script family (TypeScript, JavaScript)
fd -e ts -e tsx -e js -e jsx $PATH

# Python
fd -e py $PATH

# Rust
fd -e rs $PATH

# Go
fd -e go $PATH

# JVM (Java, Kotlin)
fd -e java -e kt $PATH

# C family
fd -e c -e cpp -e h -e cs $PATH
```

## Language Family Matrix

| Family | Languages | Extensions |
|--------|-----------|------------|
| Script | TypeScript, JavaScript, TSX, JSX | `.ts`, `.tsx`, `.js`, `.jsx` |
| Python | Python | `.py` |
| Rust | Rust | `.rs` |
| Go | Go | `.go` |
| JVM | Java, Kotlin | `.java`, `.kt` |
| C-Family | C, C++, C# | `.c`, `.cpp`, `.cs` |

---

# PHASE 2: EXTRACT - AST Pattern Execution

## AST Patterns by Language

### TypeScript/JavaScript

```yaml
# Functions
id: ts-functions
language: typescript
rule:
  any:
    - kind: function_declaration
    - kind: arrow_function
    - kind: method_definition

# Classes/Types
id: ts-types
language: typescript
rule:
  any:
    - kind: class_declaration
    - kind: interface_declaration
    - kind: type_alias_declaration

# Imports
id: ts-imports
language: typescript
rule:
  kind: import_statement

# Exports
id: ts-exports
language: typescript
rule:
  any:
    - pattern: "export function $NAME($$$) { $$$ }"
    - pattern: "export class $NAME { $$$ }"
    - pattern: "export default $EXPR"
```

### Python

```yaml
# Functions
id: py-functions
language: python
rule:
  any:
    - kind: function_definition
    - pattern: "async def $NAME($$$): $$$"

# Classes
id: py-classes
language: python
rule:
  kind: class_definition

# Imports
id: py-imports
language: python
rule:
  any:
    - kind: import_statement
    - kind: import_from_statement

# Entry point
id: py-entry
language: python
rule:
  pattern: 'if __name__ == "__main__": $$$'
```

### Rust

```yaml
# Functions
id: rust-functions
language: rust
rule:
  any:
    - kind: function_item
    - pattern: "pub fn $NAME($$$) -> $RET { $$$ }"

# Types
id: rust-types
language: rust
rule:
  any:
    - kind: struct_item
    - kind: enum_item
    - kind: trait_item
    - kind: impl_item

# Imports
id: rust-imports
language: rust
rule:
  kind: use_declaration

# Entry point
id: rust-entry
language: rust
rule:
  pattern: "fn main() { $$$ }"
```

### Go

```yaml
# Functions
id: go-functions
language: go
rule:
  any:
    - kind: function_declaration
    - kind: method_declaration

# Types
id: go-types
language: go
rule:
  kind: type_declaration

# Imports
id: go-imports
language: go
rule:
  kind: import_spec

# Entry point
id: go-entry
language: go
rule:
  pattern: "func main() { $$$ }"
```

### Java/Kotlin

```yaml
# Methods
id: java-methods
language: java
rule:
  kind: method_declaration

# Classes
id: java-classes
language: java
rule:
  kind: class_declaration

# Imports
id: java-imports
language: java
rule:
  kind: import_declaration
```

## MCP Tool Commands

```bash
# Find functions
mcp__ast-grep__find_code_by_rule(yaml="...", project_folder=$PATH)

# Find by pattern
mcp__ast-grep__find_code(pattern="fn $NAME($$$)", project_folder=$PATH, language="rust")

# Debug AST
mcp__ast-grep__dump_syntax_tree(code=$CODE, language=$LANG, format="cst")
```

---

# PHASE 3: OUTPUT - LLM-Optimized Context

## Output Format

```
<codebase_context path="{path}" depth="{depth}">
PROJECT: {name} | LANG: {languages} | FILES: {count} | LOC: {loc}

ENTRY: {entry_points}

MODULES:
{module_list with file counts}

PUBLIC_API:
{exported functions/classes/types}

TYPES:
{key type definitions}

DEPS:
{external dependencies}

PATTERNS:
{detected patterns: async, error handling, tests}
</codebase_context>
```

## Overview Output Example

```
<codebase_context path="./src" depth="overview">
PROJECT: my-app | LANG: TypeScript 68%, Python 22% | FILES: 142 | LOC: 24,350

ENTRY: src/index.ts:bootstrap() | src/cli.ts:main()

MODULES:
- api/ (12 files) - HTTP endpoints
- services/ (15 files) - Business logic
- models/ (12 files) - Data types

PUBLIC_API: 45 exports (UserService, AuthController...)

DEPS: @nestjs/core, prisma, zod

PATTERNS: async:89 | try-catch:34 | tests:123
</codebase_context>
```

## Detailed Output Example

```
<codebase_context path="./src" depth="detailed">
PROJECT: my-app | LANG: TypeScript | FILES: 45 | LOC: 12,340

FUNCTIONS:
- createUser(dto: CreateUserDto): Promise<User> [src/services/user.ts:45]
- validateToken(token: string): AuthResult [src/auth/auth.ts:23]

CLASSES:
- UserService [src/services/user.ts:10] - methods: create, findById, update
- AuthController [src/controllers/auth.ts:5] - endpoints: login, logout

TYPES:
- User { id, email, name, createdAt } [src/models/user.ts:3]
- CreateUserDto { email, password, name } [src/dto/user.dto.ts:8]

IMPORTS_GRAPH:
- api/* -> services/* -> models/*
</codebase_context>
```

---

# DEPTH LEVELS

| Level | Content | Use Case |
|-------|---------|----------|
| `overview` | Languages, LOC, entry points, modules, patterns | Quick orientation, /plan integration |
| `detailed` | + All functions/classes/types, full API surface | Deep exploration, architecture |

---

# COMMAND INTERFACE

```bash
/contexts [PATH] [OPTIONS]

Arguments:
  PATH              Target directory (default: .)

Options:
  --depth           overview|detailed (default: overview)
  --focus           functions|classes|types|imports|all (default: all)
  --lang            Filter by language: ts,py,rs,go,java
```

---

# VALIDATION GATES

| Gate | Check | Pass Criteria |
|------|-------|---------------|
| Files Found | fd count > 0 | At least 1 code file |
| Language Detected | Extension match | Known language family |
| AST Parse | ast-grep success | At least 1 pattern matches |
| Output Generated | Context format | Valid XML-like structure |

---

# EXIT CODES

| Code | Meaning |
|------|---------|
| 0 | Analysis complete |
| 11 | No code files found |
| 12 | All files failed parsing |
| 13 | ast-grep MCP not available |
| 14 | Path not found |

---

# SKILL INTEGRATION

## Auto-Integration Protocol

When `/plan` or `/review` is invoked, auto-run `/contexts` first:

```
/plan -> /contexts . --depth=overview -> plan with context
/review -> /contexts $PATH --depth=detailed -> review with context
```

---

# REQUIRED OUTPUT

1. **Context Block**
   - `<codebase_context>` with path and depth attributes
   - Compact format optimized for agent consumption

2. **Structured Sections**
   - PROJECT, ENTRY, MODULES, PUBLIC_API, TYPES, DEPS, PATTERNS

3. **File Locations**
   - `[file:line]` format for traceability

$ARGUMENTS
