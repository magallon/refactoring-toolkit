---
name: refactoring-execute
description: >
  Generates a prioritized refactoring plan from audit findings and executes code changes
  incrementally with user confirmation. Use this skill when the user has a refactoring audit
  report (from refactoring-audit or any structured findings document) and wants to act on it.
  Also triggers when the user says "execute the refactoring plan", "fix the audit findings",
  "refactor based on the report", "apply the refactoring recommendations", "start fixing
  the issues from the audit", or any request to systematically apply code improvements based
  on a diagnostic report. This skill modifies files — it always presents a plan and waits
  for explicit user approval before making any changes.
license: MIT
compatibility: >
  Requires file system read and write access to the project source code. The project should
  have version control (Git recommended) with a clean working tree before execution begins.
  Works with any web stack. Compatible with any agent that supports the Agent Skills standard.
metadata:
  version: "1.0.0"
  category: "code-quality"
  modifies-files: true
  requires-confirmation: true
---

# Refactoring Execute

Transforms audit findings into a prioritized action plan and executes refactoring changes incrementally, with mandatory user confirmation before any code modification.

## Prerequisites

Before starting, verify these conditions:

1. **Audit report exists.** There must be a structured findings document available — either from the `refactoring-audit` skill or any report that contains categorized, severity-rated findings with file locations. If no report is available, do not proceed. Instead, suggest the user run `refactoring-audit` first to generate the diagnostic report, then return to this skill with the results.

2. **Version control is clean.** Warn the user if you detect uncommitted changes. Recommend they commit or stash before proceeding. If the user chooses to proceed anyway, note the risk but don't block execution. However, if the plan includes high-risk actions (modifying shared code, changing auth logic, restructuring data access), issue a second explicit warning before executing those specific actions without a clean git state.

3. **Stack is identified.** If the audit report includes a detected stack, use it. If not, detect the stack the same way `refactoring-audit` does (scan config files and dependency manifests).

## Execution Flow

This skill has two distinct steps that must not be merged:

### Step 1 — Generate the Plan

Read `references/prioritization-rules.md` for the detailed prioritization logic.

Organize all findings from the audit report into three priority tiers:

**Priority 1 — IMMEDIATE** (execute first):
- All 🔴 findings tagged with 🔒 SECURITY
- Findings that could cause data loss or unauthorized access
- Bugs caused by missing error handling on critical paths

**Priority 2 — IMPORTANT** (execute after Priority 1 is complete):
- Remaining 🔴 findings not related to security
- 🟡 findings that affect maintainability
- Significant duplication (3+ files)
- Pattern inconsistencies that create confusion

**Priority 3 — CLEANUP** (execute last):
- All 🟢 findings
- Dead code removal
- Naming improvements
- Readability optimizations

For each action in the plan, specify:
- **What**: Concrete description of the change
- **Where**: Exact files to be modified, created, or deleted
- **Risk**: High / Medium / Low — could this break existing functionality?
- **Dependencies**: Does this action need to happen before or after another action?
- **Grouped with**: Actions that should be done together as a unit (e.g., extracting a utility and updating all its consumers)

### Example Plan Actions

For calibration, here is what well-formed plan actions look like:

```
## Priority 1 — IMMEDIATE

### Group A: Fix authentication gaps in project actions
| # | What | Where | Risk | Dependencies |
|---|---|---|---|---|
| A.1 | Add session verification at the start of `deleteProject` | src/actions/projects.ts | High | None |
| A.2 | Add session verification at the start of `updateProject` | src/actions/projects.ts | High | None |
| A.3 | Add user-ownership check before project deletion | src/actions/projects.ts | High | depends-on: A.1 |
```

### MANDATORY PAUSE

After presenting the plan, **stop and wait for the user to confirm**. Do not proceed to Step 2 until the user explicitly approves the plan or a subset of it.

The user may:
- Approve the full plan → proceed with all priorities in order
- Approve specific priorities only → execute only the approved tiers
- Request modifications → adjust the plan and present again
- Add constraints → respect any files or modules the user says not to touch
- Reject the plan → stop execution, keep the plan as documentation

### Step 2 — Execute

Read `references/execution-checklist.md` for pre/post verification steps.

Once approved, execute the plan following these rules:

**Incremental execution:**
- Apply one refactoring at a time (or one grouped unit at a time)
- After each change, verify the code still compiles/parses without errors
- If a change causes errors, fix them before moving to the next refactoring
- If a fix isn't obvious, revert the change and flag it for the user

**Ordering within each priority:**
1. Changes that create new shared utilities, types, or helpers (other changes may depend on these)
2. Changes that extract or centralize logic
3. Changes that update consumers to use the new centralized code
4. Changes that remove dead code or unused artifacts
5. Changes that rename or restructure

**Before deleting anything:**
- Verify the file, function, export, or variable is truly unused by searching the entire project
- If there's any ambiguity (dynamic imports, framework conventions, reflection), ask the user before deleting

**Before modifying shared code:**
- Identify all consumers of the code being changed
- Ensure the modification doesn't break any consumer
- If the change affects a public API or interface, list all affected consumers in the summary

**Decision points:**
- If a refactoring requires a design decision (e.g., choosing between two valid approaches, naming a new abstraction, selecting a pattern), stop and ask the user
- If a refactoring is riskier than initially assessed, inform the user before proceeding
- If you discover new issues during execution that weren't in the audit, note them but don't act on them unless the user asks

## Progress Reporting

After completing each priority tier, provide a summary:

```
## Priority [N] Complete

### Changes Applied
| # | Action | Files Modified | Files Created | Files Deleted |
|---|--------|---------------|---------------|---------------|
| 1 | [Description] | [paths] | [paths] | [paths] |

### Manual Verification Needed
- [Anything the agent cannot verify automatically — e.g., visual changes, runtime behavior, integration tests]

### New Issues Discovered
- [Any problems found during execution that weren't in the original audit]

### Ready for Priority [N+1]?
```

Wait for user confirmation before proceeding to the next priority tier.

## Critical Rules

- **Never skip the plan confirmation.** This is non-negotiable. The plan must be presented and approved before any file is modified.
- **This is refactoring, not rewriting.** Preserve existing functionality. Don't change behavior, only improve structure, clarity, and safety.
- **One thing at a time.** Resist the temptation to combine multiple unrelated changes in a single step. Each change should be reviewable in isolation.
- **Verify before deleting.** Dead code identification in the audit may have false positives. Always re-verify before removal.
- **Respect the user's constraints.** If the user says "don't touch the auth module" or "skip Priority 3," respect that absolutely.
- **Adapt to the stack's conventions.** When creating new files (utilities, types, helpers), follow the project's existing naming conventions, directory structure, and code style. Don't impose external conventions.

## Edge Cases

- **Audit report from a different tool**: The plan still works as long as findings have descriptions and file locations. Adapt the prioritization to whatever severity system the report uses.
- **Project without version control**: Warn the user strongly. Recommend they initialize git and commit before proceeding. If they insist on proceeding, acknowledge the risk.
- **Conflicting findings**: If two findings suggest opposite changes (e.g., one says centralize, another says split), present both options and let the user decide.
- **Very large refactoring scope**: If the plan has 50+ actions, suggest breaking it into multiple sessions, each covering one priority tier or one module.
