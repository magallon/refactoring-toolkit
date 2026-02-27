---
name: refactoring-audit
description: >
  Performs a comprehensive 3-phase refactoring audit on any web project, analyzing structure,
  code quality, and security. Produces a consolidated diagnostic report with severity-rated
  findings. Use this skill when the user asks to audit a codebase for refactoring opportunities,
  evaluate technical debt, review code quality, find dead code, detect security issues in
  server-side logic, or assess overall project health. Triggers on phrases like "audit my code",
  "find technical debt", "code review the project", "what needs refactoring", "analyze code quality",
  "find dead code", "check for security issues", or any request to systematically evaluate
  a codebase before refactoring. This is a read-only diagnostic — it does not modify any files.
license: MIT
compatibility: >
  Requires file system read access to the project source code. Works with any web stack
  (frontend, backend, fullstack, monorepo, static sites). No network access needed.
  Compatible with any agent that supports the Agent Skills standard.
metadata:
  version: "1.0.0"
  category: "code-quality"
  phase-count: 3
  modifies-files: false
---

# Refactoring Audit

A read-only, 3-phase diagnostic audit that evaluates any web project's structural health, code quality, and security posture. Produces a single consolidated report with prioritized findings.

## Before You Start

### Stack Detection

Before beginning the audit, identify and document the project's stack. Scan the project root for configuration files and dependency manifests to determine:

- Language(s) and runtime (TypeScript, Python, Ruby, PHP, Go, etc.)
- Framework(s) (Next.js, Laravel, Django, Rails, Express, SvelteKit, etc.)
- Styling approach (Tailwind, CSS modules, styled-components, SCSS, etc.)
- Database and ORM (Supabase, Prisma, Eloquent, ActiveRecord, Mongoose, etc.)
- Authentication method (built-in framework auth, third-party provider, custom)
- Component/UI library if applicable (shadcn/ui, Vuetify, MUI, etc.)
- Package manager (npm, yarn, pnpm, pip, composer, bundler, etc.)
- Monorepo or single-app structure

Record the detected stack at the top of the report. This anchors all subsequent analysis — when the audit references "server-side data mutation handlers," you interpret that as Server Actions, Controllers, Views, or whatever applies to the detected stack.

### Scope Assessment

Count the number of source code files in the project (excluding node_modules, vendor, .git, build output, and similar generated directories).

- **Under 100 files**: Audit the entire project in one pass.
- **100–500 files**: Audit the entire project, but organize findings by module or directory in the report.
- **Over 500 files**: Propose a scope reduction. Suggest auditing by module, feature, or directory. Present the options and wait for user confirmation before starting.

If the user has already specified a scope (e.g., "audit only the auth module"), respect that scope.

## Execution Flow

Run all three phases sequentially without pausing between them. Deliver a single consolidated report at the end.

The three phases are:

1. **Structural Analysis** — File organization, dead code, dependency health, separation of concerns
2. **Code Quality** — Duplication, naming, complexity, pattern consistency
3. **Security & Robustness** — Auth/authz, input validation, error handling, data access security, test indicators

For each phase, read the corresponding reference file for detailed evaluation criteria:

- Phase 1: Read `references/phase1-structural.md`
- Phase 2: Read `references/phase2-quality.md`
- Phase 3: Read `references/phase3-security.md`

## Severity Classification

Use three severity levels for all findings. Read `references/severity-criteria.md` for detailed definitions, thresholds, and examples for each level:

- 🔴 **Critical** — Immediate risk: security vulnerabilities, data loss potential, broken functionality
- 🟡 **Important** — Significant maintainability or reliability concern
- 🟢 **Minor** — Improvement opportunity with low risk

When classifying a finding, refer to the concrete thresholds in the severity criteria reference. Avoid subjective judgment when objective thresholds exist.

## Report Delivery

After completing all three phases, produce a single consolidated report. Read `references/report-template.md` for the exact output format.

The report must include:

1. **Header** — Project name, detected stack, audit scope, date, file count
2. **Executive Summary** — Total findings by severity, top 3 most impactful issues
3. **Phase 1 Findings** — Structural analysis results
4. **Phase 2 Findings** — Code quality results
5. **Phase 3 Findings** — Security and robustness results
6. **Statistics** — Counts and metrics (dead code files, duplication instances, any-type count, etc.)
7. **Recommended Next Steps** — Brief guidance on what to address first

Each finding must include: severity level, category, description, file location(s), and why it matters.

### Example Findings

For calibration, here is what well-formed findings look like:

| # | Severity | Finding | Location | Impact |
|---|---|---|---|---|
| 1 | 🔴 🔒 | Server action `deleteProject` does not verify user session before deleting | `src/actions/projects.ts:45` | Any authenticated user could delete any project by sending a crafted request |
| 2 | 🟡 | Error handling uses 3 different patterns across handlers: raw try/catch, Result type, and unhandled | `src/actions/*.ts` | Developers cannot predict how errors behave, increasing bug risk during changes |
| 3 | 🟢 | Variable `data` used without qualifying context in 4 functions | `src/utils/transform.ts:12,38,67,94` | Requires reading function body to understand what the variable holds |

## Critical Rules

- **Do not modify any files.** This skill is purely diagnostic.
- **Do not skip phases.** Even if the project seems clean after Phase 1, complete all three phases. Problems compound across dimensions.
- **Be specific.** Every finding must reference exact file paths and line numbers when possible. Vague findings like "some components are too complex" are not useful.
- **Flag uncertainty.** If you cannot determine whether something is truly dead code or unused, mark it as "suspected" and explain why you're uncertain.
- **Adapt terminology to the stack.** If the project uses Django, say "views" not "server-side handlers." If it uses Rails, say "controllers" not "mutation endpoints." Use the stack's native vocabulary.
- **No false urgency.** Only classify findings as 🔴 Critical if they meet the specific criteria in the severity reference. Inflating severity erodes trust in the report.

## Edge Cases

- **No TypeScript / no static typing**: Skip type-discipline checks in Phase 2. Note the absence of static typing as a finding in Phase 3 (validation section) since runtime validation becomes more critical.
- **No server-side logic** (static site, SPA with external API): Skip server-side auth and handler checks in Phase 3. Focus on client-side security (exposed secrets, API key handling, XSS surface).
- **No tests**: Note the complete absence of tests as a 🟡 Important finding in Phase 3. Don't skip the rest of Phase 3.
- **Monorepo with multiple apps**: If auditing the full repo, produce findings grouped by app/package. If the user scoped to one app, stay within that scope.
- **Minimal project** (under 10 files): Run all phases but expect fewer findings. Don't pad the report with low-value observations just to fill space.
