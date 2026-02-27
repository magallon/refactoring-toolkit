# Prioritization Rules

Detailed logic for organizing audit findings into a prioritized execution plan.

## Table of Contents
- [Priority Tiers](#priority-tiers)
- [Ordering Within Tiers](#ordering-within-tiers)
- [Dependency Resolution](#dependency-resolution)
- [Risk Assessment](#risk-assessment)
- [Grouping Related Actions](#grouping-related-actions)

---

## Priority Tiers

### Priority 1 — IMMEDIATE

**What goes here:** Anything that represents an active risk to the application or its users.

Specific criteria (in execution order within this tier):
1. 🔴 Security findings (tagged 🔒): missing auth checks, unvalidated inputs reaching the database, exposed credentials, SQL injection surfaces, missing RLS/access policies
2. 🔴 Data integrity risks: handlers where errors could cause partial writes, race conditions, or silent data corruption
3. 🔴 Critical error handling gaps: unhandled exceptions in data mutation paths that could crash the server or leave the UI in a broken state

**Expected outcome:** After Priority 1, the application has no known security vulnerabilities or data loss risks.

### Priority 2 — IMPORTANT

**What goes here:** Issues that significantly impact code maintainability, reliability, or developer productivity.

Specific criteria (in execution order within this tier):
1. Remaining 🔴 non-security findings: functions over 100 lines, components over 400 lines, nesting over 4 levels
2. Pattern inconsistency: standardize error handling, data access, form handling, and state management to follow the project's dominant pattern
3. Significant duplication: extract shared logic, unify near-identical components, centralize repeated patterns
4. Missing validation: add server-side validation where it's absent, implement consistent validation approach
5. Error handling completeness: add try/catch to unprotected handlers, ensure consistent error response structure, add error boundaries
6. Type discipline: eliminate `any` usage, add missing type definitions

**Expected outcome:** After Priority 2, the codebase follows consistent patterns, has minimal duplication, and handles errors reliably.

### Priority 3 — CLEANUP

**What goes here:** Improvements that reduce noise, improve readability, and polish the codebase.

Specific criteria (in execution order within this tier):
1. Dead code removal: unused files, exports, imports, variables (re-verify each before deleting)
2. Dependency cleanup: remove unused dependencies, resolve duplicates
3. Naming improvements: rename ambiguous variables, functions, and components
4. Residual artifact cleanup: remove TODO/FIXME comments (after addressing any actionable ones), remove console.log/debugger statements, remove commented-out code
5. Structural reorganization: move misplaced files, add missing companion files (error boundaries, loading states)
6. Style extraction: extract repeated CSS/class patterns into reusable components or utilities

**Expected outcome:** After Priority 3, the codebase is clean, consistently named, and free of dead weight.

---

## Ordering Within Tiers

Within each priority tier, order actions to minimize risk and maximize independence:

1. **Create before consume.** If a refactoring requires a new shared utility, type, or helper, create it first. Then update consumers to use it.

2. **Centralize before deduplicate.** When eliminating duplication, first create the centralized version, then replace all duplicates with references to it. Never delete a duplicate before its replacement is in place.

3. **Isolate before modify.** If a change affects multiple files, make the structural change first (extract function, create new module), then update references. This way, if something goes wrong, only the new code needs to be reverted.

4. **Safe changes before risky ones.** Within the same priority, execute lower-risk changes first. This builds confidence and catches environmental issues early.

5. **Independent changes before dependent ones.** If action B depends on action A, A must execute first. Map dependencies explicitly in the plan.

---

## Dependency Resolution

When building the plan, identify dependencies between actions:

**Hard dependencies** (must respect order):
- Creating a utility → updating consumers to use it
- Extracting a type definition → importing it where needed
- Standardizing an error response format → updating all handlers to use it
- Adding a validation schema → applying it in handlers

**Soft dependencies** (preferred order but not strictly required):
- Fixing auth checks → adding input validation (both are Priority 1 but auth should come first)
- Reducing complexity → improving naming (cleaner code is easier to rename)
- Removing dead code → restructuring directories (less code to move)

**No dependency** (can be done in any order):
- Fixing auth in handler A → fixing auth in handler B
- Renaming variables in file X → renaming variables in file Y
- Removing unused dependency A → removing unused dependency B

Present dependencies in the plan using notation:
- `depends-on: #[action-number]` for hard dependencies
- `recommended-after: #[action-number]` for soft dependencies

---

## Risk Assessment

Assign a risk level to each action in the plan:

### High Risk
The change could break existing functionality if done incorrectly.

Indicators:
- Modifying shared code used by 5+ consumers
- Changing function signatures or interfaces
- Restructuring data access patterns
- Modifying authentication or authorization logic
- Changing error handling in critical paths

Mitigation: Identify all consumers before changing. Test each consumer after the change. Consider creating the new version alongside the old one temporarily.

### Medium Risk
The change is unlikely to break things but requires careful verification.

Indicators:
- Extracting logic into new shared utilities
- Unifying components with minor differences
- Adding validation to existing handlers (could reject previously accepted input)
- Standardizing patterns across multiple files

Mitigation: Verify compilation after each change. Check that the new abstraction covers all edge cases from the original implementations.

### Low Risk
The change is safe and unlikely to affect behavior.

Indicators:
- Removing confirmed dead code (verified unused)
- Renaming local variables
- Removing console.log statements
- Deleting commented-out code
- Adding type annotations to existing code
- Removing unused imports

Mitigation: Standard verification (code compiles, no new errors).

---

## Grouping Related Actions

Some actions form natural units that should be executed together:

**Extract-and-replace group:**
Creating a shared utility and updating all its consumers is one atomic unit. Don't create the utility in one step and update consumers three steps later — they should be adjacent in the plan.

**Pattern standardization group:**
When standardizing a pattern (e.g., error handling), change all instances together. Leaving half the handlers on the old pattern and half on the new is worse than the original inconsistency.

**Type definition group:**
When adding type definitions that are used across multiple files, define all related types together, then update consumers.

In the plan, mark groups clearly:

```
### Group A: Extract data access utility
  - A.1: Create shared query utility
  - A.2: Update handler X to use shared utility
  - A.3: Update handler Y to use shared utility
  - A.4: Update handler Z to use shared utility
  - A.5: Remove duplicated query logic from handlers
```

All actions in a group succeed or fail together. If A.3 causes issues, revert A.1 through A.3 and reassess.
