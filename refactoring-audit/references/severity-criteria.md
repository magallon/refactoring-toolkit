# Severity Criteria

Concrete definitions, thresholds, and examples for each severity level. When classifying a finding, use the specific thresholds below. If a finding doesn't clearly match a threshold, use the general principles to decide.

---

## 🔴 Critical

### General Principle
The finding represents an **immediate risk** — it could cause security vulnerabilities, data loss, unauthorized access, broken functionality, or significant production issues.

### Automatic Critical Classification
Any finding in these categories is always 🔴 Critical:
- Missing authentication on a data-modifying handler
- Missing authorization check when handling user-submitted entity IDs
- Unvalidated client input reaching database writes
- Raw queries with string-concatenated user input (SQL/NoSQL injection surface)
- Hardcoded secrets, credentials, or API keys in source code
- Server-privileged client accessible from client-side code
- Client-only validation with no server-side counterpart on mutations
- File uploads with no type or size validation

### Threshold-Based Critical Classification
| Metric | Critical Threshold |
|---|---|
| Function length | > 100 lines |
| Component length | > 400 lines |
| Nesting depth | > 4 levels |
| Business logic duplication | Same logic in 5+ files |
| Error handling patterns | 3+ different patterns for the same concern |
| Database queries in UI components | Same table queried from 3+ UI components |
| Monolithic handler file | Mixes 5+ unrelated domains |

---

## 🟡 Important

### General Principle
The finding represents a **significant maintainability, reliability, or quality concern** — it makes the codebase harder to work with, introduces risk of bugs during changes, or creates confusion for developers.

### Threshold-Based Important Classification
| Metric | Important Threshold |
|---|---|
| Function length | 50-100 lines |
| Component length | 200-400 lines |
| Nesting depth | 4 levels |
| Code duplication | Same pattern in 3-4 files |
| `any` type usage | 20+ occurrences in project |
| Unused dependencies | 5+ packages |
| Unused files (confirmed) | Any confirmed unused file |
| Missing error boundaries | On routes with data mutations |
| Inconsistent patterns | Each implementation differs in a given category |
| Console.log in production paths | Any instance |
| Silent error swallowing | Catch block with no logging or re-throw |
| Missing tests for critical paths | Auth, mutations, business rules |

---

## 🟢 Minor

### General Principle
The finding represents an **improvement opportunity** with low risk — fixing it improves readability, consistency, or maintainability but doesn't pose an immediate risk.

### Common Minor Findings
- Isolated naming convention deviations (1-2 files)
- Minor code duplication (2 files)
- TODO/FIXME comments without security implications
- Commented-out code blocks
- Missing client-side validation (when server-side exists)
- Validation present but missing edge cases
- 5-19 uses of `any`
- 1-2 unused dependencies
- Missing error boundary on read-only routes
- Tests with weak assertions
- Minor style duplication (shared CSS patterns not extracted)

---

## Edge Cases

**When a finding spans multiple thresholds:**
Use the highest applicable severity. For example, if a function is 60 lines (🟡) but also has 5 levels of nesting (🔴), the finding is 🔴 Critical.

**When you're unsure between two levels:**
Ask: "If this finding were left unfixed for 6 months while the team actively develops features, what's the worst realistic outcome?"
- Could cause a production incident or security breach → 🔴 Critical
- Would significantly slow down development or cause bugs during changes → 🟡 Important
- Would mildly annoy developers but not cause real problems → 🟢 Minor

**Security findings always get an extra flag:**
Regardless of the severity level assigned, any finding with security implications must be tagged with `🔒 SECURITY` in addition to its severity emoji. This makes security findings scannable in the report.
