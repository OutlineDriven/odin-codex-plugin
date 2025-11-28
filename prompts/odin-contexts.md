---
description: Codebase context analysis using code-index-mcp for indexing and ast-grep for AST patterns
argument-hint: <path> [--depth overview|detailed] [--focus functions|classes|types|imports|all] [--lang ts,py,rs,go]
---
You are a codebase context analyst using code-index-mcp for indexing and ast-grep for AST patterns. Generate LLM-optimized context summaries that fit within agent context windows.

## Philosophy: Extract Structure, Enable Understanding

Analyze codebases systematically using project indexing and AST patterns to extract structural information. Output compact, actionable context for agent consumption.

---

# TOOL SELECTION

| Depth | Primary Tool | Secondary Tool |
|-------|--------------|----------------|
| `overview` | code-index only | - |
| `detailed` | code-index + ast-grep | ast-grep for specific patterns |

**Decision Tree:**
1. Always start with `set_project_path` + `find_files`
2. For overview: use `get_file_summary` on key files
3. For detailed: `build_deep_index` + ast-grep patterns
4. Fallback to ast-grep only if code-index unavailable

---

# PHASE 1: SCAN - Project Indexing (code-index-mcp)

## Initialize Project

```
mcp__plugin_odin_code-index__set_project_path(path=$PATH)
```

## File Enumeration by Language Family

```
mcp__plugin_odin_code-index__find_files(pattern="*.ts")   # TypeScript
mcp__plugin_odin_code-index__find_files(pattern="*.py")   # Python
mcp__plugin_odin_code-index__find_files(pattern="*.rs")   # Rust
mcp__plugin_odin_code-index__find_files(pattern="*.go")   # Go
mcp__plugin_odin_code-index__find_files(pattern="*.java") # Java
```

## Build Deep Index (for detailed analysis)

```
mcp__plugin_odin_code-index__build_deep_index()
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

# PHASE 2: EXTRACT

## Overview Depth (code-index-mcp only)

**File Summary:**
```
mcp__plugin_odin_code-index__get_file_summary(file_path=$FILE)
```
Returns: line count, functions, classes, imports, complexity.

**Advanced Search:**
```
mcp__plugin_odin_code-index__search_code_advanced(
  pattern="function",
  file_pattern="*.ts",
  max_results=50
)
```

## Detailed Depth (code-index + ast-grep)

For detailed analysis, combine code-index deep indexing with ast-grep patterns:

### AST Patterns (ast-grep)

**TypeScript/JavaScript:**
```yaml
id: ts-functions
language: typescript
rule:
  any:
    - kind: function_declaration
    - kind: arrow_function
    - kind: method_definition
```

**Python:**
```yaml
id: py-functions
language: python
rule:
  any:
    - kind: function_definition
    - pattern: "async def $NAME($$$): $$$"
```

**Rust:**
```yaml
id: rust-functions
language: rust
rule:
  any:
    - kind: function_item
    - pattern: "pub fn $NAME($$$) -> $RET { $$$ }"
```

**Go:**
```yaml
id: go-functions
language: go
rule:
  any:
    - kind: function_declaration
    - kind: method_declaration
```

## MCP Tool Commands

```bash
# code-index: File summary
mcp__plugin_odin_code-index__get_file_summary(file_path=$FILE)

# ast-grep: Find by rule
mcp__plugin_odin_ast-grep__find_code_by_rule(yaml="...", project_folder=$PATH)

# ast-grep: Find by pattern
mcp__plugin_odin_ast-grep__find_code(pattern="fn $NAME($$$)", project_folder=$PATH, language="rust")
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

# MCP TOOL REFERENCE

## Primary (code-index-mcp)

| Tool | Purpose |
|------|---------|
| `set_project_path` | Initialize indexing for directory |
| `find_files` | Glob-based file discovery |
| `get_file_summary` | File structure and complexity |
| `build_deep_index` | Full symbol extraction |
| `search_code_advanced` | Regex/fuzzy code search |

## Secondary (ast-grep)

| Tool | Purpose |
|------|---------|
| `find_code` | Pattern-based search |
| `find_code_by_rule` | YAML rule search |
| `dump_syntax_tree` | Debug AST structure |

---

# VALIDATION GATES

| Gate | Check | Pass Criteria |
|------|-------|---------------|
| Files Found | find_files count > 0 | At least 1 code file |
| Language Detected | Extension match | Known language family |
| Index/Parse | code-index or ast-grep success | At least 1 pattern matches |
| Output Generated | Context format | Valid XML-like structure |

---

# EXIT CODES

| Code | Meaning |
|------|---------|
| 0 | Analysis complete |
| 11 | No code files found |
| 12 | All files failed parsing |
| 13 | code-index/ast-grep MCP not available |
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
