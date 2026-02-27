# Report Template

Use this exact structure for the consolidated audit report. Every section is required. If a section has no findings, include it with "No findings in this category."

---

## Report Structure

```
# Refactoring Audit Report

## Project Summary
- **Project**: [name]
- **Stack**: [detected stack — language, framework, database, styling, etc.]
- **Scope**: [full project / specific module or directory]
- **Date**: [audit date]
- **Source files analyzed**: [count, excluding generated/vendor/dependency directories]

## Executive Summary

**Total findings**: [count] (🔴 [n] Critical | 🟡 [n] Important | 🟢 [n] Minor)

**Top 3 most impactful issues:**
1. [Brief description of the most critical finding]
2. [Brief description of the second most critical finding]
3. [Brief description of the third most critical finding]

---

## Phase 1 — Structural Analysis

### File and Directory Organization
| # | Severity | Finding | Location | Impact |
|---|----------|---------|----------|--------|
| 1 | 🔴/🟡/🟢 | [Description] | [file path(s)] | [Why it matters] |

### Dead Code and Residual Artifacts
| # | Severity | Finding | Location | Impact |
|---|----------|---------|----------|--------|
| 1 | 🔴/🟡/🟢 | [Description] | [file path(s)] | [Why it matters] |

### Dependency Health
| # | Severity | Finding | Location | Impact |
|---|----------|---------|----------|--------|
| 1 | 🔴/🟡/🟢 | [Description] | [file path(s)] | [Why it matters] |

### Separation of Concerns
| # | Severity | Finding | Location | Impact |
|---|----------|---------|----------|--------|
| 1 | 🔴/🟡/🟢 | [Description] | [file path(s)] | [Why it matters] |

---

## Phase 2 — Code Quality

### Code Duplication
| # | Severity | Finding | Files Involved | Impact |
|---|----------|---------|----------------|--------|
| 1 | 🔴/🟡/🟢 | [Description] | [all file paths] | [Why it matters] |

### Naming and Readability
| # | Severity | Finding | Location | Impact |
|---|----------|---------|----------|--------|
| 1 | 🔴/🟡/🟢 | [Description] | [file path(s)] | [Why it matters] |

### Excessive Complexity
| # | Severity | Finding | Location | Metrics |
|---|----------|---------|----------|---------|
| 1 | 🔴/🟡/🟢 | [Description] | [file:function] | [lines, nesting, params] |

### Pattern Consistency
| # | Severity | Finding | Files Involved | Impact |
|---|----------|---------|----------------|--------|
| 1 | 🔴/🟡/🟢 | [Description] | [representative files] | [Why it matters] |

---

## Phase 3 — Security & Robustness

### Server-Side Handler Security
| # | Severity | Finding | Location | Risk |
|---|----------|---------|----------|------|
| 1 | 🔴 🔒 | [Description] | [file path(s)] | [What could go wrong] |

### Data Access Security
| # | Severity | Finding | Location | Risk |
|---|----------|---------|----------|------|
| 1 | 🔴 🔒 | [Description] | [file path(s)] | [What could go wrong] |

### Error Handling Completeness
| # | Severity | Finding | Location | Impact |
|---|----------|---------|----------|--------|
| 1 | 🔴/🟡/🟢 | [Description] | [file path(s)] | [Why it matters] |

### Data Validation
| # | Severity | Finding | Location | Risk |
|---|----------|---------|----------|------|
| 1 | 🔴/🟡/🟢 🔒 | [Description] | [file path(s)] | [What could go wrong] |

### Test Coverage Indicators
| # | Severity | Finding | Details |
|---|----------|---------|---------|
| 1 | 🔴/🟡/🟢 | [Description] | [What's missing] |

---

## Audit Statistics

| Metric | Value |
|---|---|
| Total source files analyzed | [n] |
| Confirmed dead code files | [n] |
| Suspected dead code files | [n] |
| Unused dependencies | [n] |
| Duplication instances | [n groups of duplicated code] |
| `any` type count (if applicable) | [n] |
| TODO/FIXME comments | [n] |
| Console.log/debugger statements | [n] |
| Handlers without auth check | [n] / [total handlers] |
| Handlers without error handling | [n] / [total handlers] |

---

## Recommended Next Steps

1. **Immediate**: [Address all 🔴 security findings first — these represent active risk]
2. **Short-term**: [Address remaining 🔴 and 🟡 findings — these affect maintainability]
3. **Ongoing**: [Address 🟢 findings as part of regular development — these are improvement opportunities]

> To proceed with refactoring, use the **refactoring-execute** skill with this report as input.
```

---

## Formatting Rules

- Every finding gets a unique sequential number within its table (for easy reference in discussions)
- Security findings always include the 🔒 tag after the severity emoji
- File paths should be relative to the project root
- When a finding involves multiple files, list all of them (up to 10; if more, say "and N others")
- The "Impact" or "Risk" column should be one sentence explaining the real-world consequence, not a restatement of the finding
- "Suspected" dead code must be clearly labeled as suspected, not confirmed
- If a phase has no findings in a category, include the table header with a single row: "No findings in this category"
