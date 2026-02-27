# Phase 2 — Code Quality

Detailed evaluation criteria for code duplication, naming and readability, complexity, and pattern consistency.

## Table of Contents
- [1. Code Duplication](#1-code-duplication)
- [2. Naming and Readability](#2-naming-and-readability)
- [3. Excessive Complexity](#3-excessive-complexity)
- [4. Pattern Consistency](#4-pattern-consistency)

---

## 1. Code Duplication

### What to Check

**Logic duplication:**
- Blocks of logic (5+ lines) that appear with minor variations in 2 or more files
- Common patterns: similar database queries, repeated validation sequences, duplicated data transformations, copied error handling blocks
- Pay special attention to copy-paste code where only variable names or table names changed

**Component duplication:**
- UI components that are near-identical variants of each other
- Components that differ only in a few props, styles, or labels and could be unified with parameters
- Threshold: Two components sharing 70%+ of their structure are candidates for unification

**Pattern duplication:**
- Error handling patterns (try/catch blocks) repeated across multiple handlers with the same structure
- API response formatting repeated instead of abstracted
- Authentication/authorization checks duplicated instead of centralized in middleware or a utility

**Style duplication:**
- Repeated combinations of CSS classes or styles that appear in 3+ components
- These could be extracted into reusable component variants or utility classes

### How to Evaluate

Look for structural similarity, not just textual similarity. Two blocks that do the same thing with different variable names are duplicates even if a diff tool wouldn't flag them.

For each duplication finding, list all files involved so the scope is clear.

### Severity Guide

| Finding | Severity |
|---|---|
| Business logic duplicated across 5+ files | 🔴 Critical |
| Data access pattern duplicated across 3+ files | 🟡 Important |
| Error handling pattern duplicated across 3+ handlers | 🟡 Important |
| Two near-identical components that should be unified | 🟡 Important |
| Style combination repeated in 3+ components | 🟢 Minor |
| Minor logic duplication in 2 files | 🟢 Minor |

---

## 2. Naming and Readability

### What to Check

**Variable naming:**
- Variables with generic or ambiguous names: `data`, `result`, `temp`, `item`, `info`, `val`, `res`, `err` (without qualifying context), `x`, `y`, `obj`, `arr`, `list`, `stuff`
- Exception: Loop iterators (`i`, `j`, `k`) and very short lambda parameters are acceptable in small scopes
- The test: If you need to read the function body to understand what a variable contains, the name is inadequate

**Function naming:**
- Functions whose name doesn't describe their action or return value
- Misleading names (function named `getUser` that also modifies state)
- Overly abbreviated names that sacrifice clarity

**Component naming:**
- Components whose name doesn't reflect their role in the UI
- Generic names that could apply to many components (`Card`, `Item`, `Section` without qualifying context)

**Type and interface naming:**
- Types or interfaces with names that don't communicate their purpose
- Inconsistent naming patterns (some types prefixed with `I`, some with `T`, some with neither)

**Convention uniformity:**
- Verify the project uses a consistent convention:
  - camelCase for functions and variables
  - PascalCase for components, types, interfaces, classes
  - UPPER_SNAKE_CASE for constants and environment variables
  - kebab-case for file and directory names (or whatever the project chose)
- Flag mixed conventions within the same category

### How to Evaluate

Scan files for the patterns listed above. For naming quality, apply this rule: a name is poor if you need to read the implementation to understand what it refers to. Don't flag established domain abbreviations (e.g., `tx` for transaction in a finance app, `req`/`res` in HTTP handlers).

### Severity Guide

| Finding | Severity |
|---|---|
| Misleading function/variable names (name suggests wrong behavior) | 🟡 Important |
| 10+ generic variable names across the project | 🟡 Important |
| Mixed naming conventions within the same category (e.g., some components in camelCase, some in PascalCase) | 🟡 Important |
| Isolated generic variable names in small functions | 🟢 Minor |
| Minor abbreviations that are contextually clear | 🟢 Minor |

---

## 3. Excessive Complexity

### What to Check

**Function length:**
- Functions or methods exceeding **50 lines** of logic (excluding comments and blank lines)
- Server-side handlers or controllers exceeding **80 lines**
- These should be broken into smaller, focused functions

**Component length:**
- UI components exceeding **200 lines** (total file length including template/JSX)
- Components this long typically mix too many responsibilities and should be decomposed

**File length:**
- Any single source file exceeding **400 lines**
- Long files usually indicate missing abstraction boundaries

**Nesting depth:**
- More than **3 levels** of nesting (if/else, ternary chains, nested callbacks, promise chains, nested loops)
- Deeply nested code should be flattened with early returns, guard clauses, or extraction into functions

**Conditional complexity:**
- Complex boolean expressions (3+ conditions combined with AND/OR)
- Chained ternary operators
- Switch/case or if/else chains with 5+ branches that could be replaced with lookup tables or strategy patterns

**Cognitive load indicators:**
- Functions that do more than one thing (fetch data AND transform it AND update state AND handle errors all in one block)
- Long parameter lists (5+ parameters suggest the function does too much or needs a config object)

### How to Evaluate

For each file, note:
- Total line count
- Any functions exceeding the line thresholds
- Maximum nesting depth encountered
- Number of parameters on the longest function signature

Report the specific function/component name and its line count.

### Severity Guide

| Finding | Severity |
|---|---|
| Function exceeding 100 lines | 🔴 Critical |
| Component exceeding 400 lines | 🔴 Critical |
| Function with nesting depth > 4 levels | 🔴 Critical |
| Function between 50-100 lines | 🟡 Important |
| Component between 200-400 lines | 🟡 Important |
| Nesting depth of exactly 4 levels | 🟡 Important |
| Chained ternary operators (2+) | 🟡 Important |
| Long parameter list (5+ params) | 🟡 Important |
| Function between 30-50 lines (review candidate) | 🟢 Minor |

---

## 4. Pattern Consistency

Inconsistent patterns create cognitive overhead — every developer touching the code has to figure out which style this particular file uses. Consistency matters more than any specific pattern choice.

### What to Check

**Error handling patterns:**
- Do all server-side handlers follow the same error handling approach? (try/catch with structured response, Result type, error middleware, etc.)
- Is the error response shape consistent? (e.g., always `{ success, data, error }` or always throwing with error classes)
- Or does each handler do it differently?

**Data access patterns:**
- Is the database/API client used the same way everywhere?
- Are queries structured consistently (same client initialization, same error handling, same response mapping)?
- Or does each file reinvent how to talk to the database?

**State management patterns:**
- Is loading state handled consistently across pages/views?
- Are the same mechanisms used throughout (hooks, context, store, signals), or does each feature choose differently?

**Form patterns:**
- Are forms built consistently? (same validation approach, same submission flow, same error display)
- Or does each form implement its own pattern?

**Type discipline** (for statically typed projects):
- Count all uses of `any`, `as any`, type assertions, and missing return types
- Are interfaces/types defined for all data structures, or are some inlined or skipped?
- Is there a mix of interfaces and types without clear reasoning?
- Provide a total `any` count for the project

### How to Evaluate

Pick the most common pattern in each category — that's the "expected" pattern. Then find deviations from it. The finding isn't that the project uses a specific pattern; the finding is inconsistency in applying whatever pattern was chosen.

For type discipline, run a search for `any` across all source files and count occurrences.

### Severity Guide

| Finding | Severity |
|---|---|
| Error handling inconsistent across handlers (3+ different patterns) | 🔴 Critical |
| Data access patterns vary significantly across the project | 🟡 Important |
| Form handling inconsistent (each form is different) | 🟡 Important |
| 20+ uses of `any` across the project | 🟡 Important |
| Loading state handling varies across pages | 🟡 Important |
| 5-19 uses of `any` | 🟢 Minor |
| Minor pattern deviations in 1-2 files | 🟢 Minor |
