# Refactoring Toolkit

A stack-agnostic methodology for auditing and refactoring web projects, packaged as two complementary [Agent Skills](https://agentskills.io).

## What This Solves

Codebases accumulate technical debt gradually — dead code, inconsistent patterns, duplicated logic, missing error handling, security gaps. This toolkit gives AI agents a structured, repeatable methodology to diagnose these problems and fix them safely.

It works with **any web stack**: Next.js, Laravel, Django, Rails, Express, SvelteKit, Nuxt, or anything else. The audit evaluates universal software engineering principles, not framework-specific rules.

## The Two Skills

| Skill | Purpose | What It Does |
|---|---|---|
| **refactoring-audit** | Diagnosis | Runs a 3-phase analysis covering structure, code quality, and security. Produces a consolidated report with prioritized findings. |
| **refactoring-execute** | Treatment | Consumes the audit report, generates a prioritized refactoring plan, and executes changes incrementally with user confirmation. |

They're designed to work together sequentially, but each can be used independently:

- Use **refactoring-audit** alone to generate reports for tech leads or code reviews
- Use **refactoring-execute** with any structured findings document, not just this audit's output

## Workflow

```
┌─────────────────────┐     ┌──────────────────────┐
│  refactoring-audit   │     │  refactoring-execute   │
│                     │     │                       │
│  Phase 1: Structure │     │  Step 1: Plan         │
│  Phase 2: Quality   │────▶│  Step 2: Confirm      │
│  Phase 3: Security  │     │  Step 3: Execute      │
│                     │     │  Step 4: Verify       │
│  ➜ Consolidated     │     │                       │
│    Report           │     │  ➜ Refactored Code    │
└─────────────────────┘     └──────────────────────┘
```

1. **Audit** — The agent reads your codebase and produces a diagnostic report across three phases (structural analysis, code quality, security & robustness). No files are modified.
2. **Review** — You review the consolidated report. Ask the agent to dive deeper into specific findings if needed.
3. **Plan** — The execution skill prioritizes all findings into an actionable plan (security first, then maintainability, then cleanup).
4. **Confirm** — The agent presents the plan and waits for your approval before touching any code.
5. **Execute** — Changes are applied incrementally, one refactoring at a time, with verification after each step.

## Quick Start

### Installation

#### Using the Skills CLI (recommended)

```bash
# Interactive install — choose which skills and agent
npx skills add magallon/refactoring-toolkit

# List available skills before installing
npx skills add magallon/refactoring-toolkit --list
```

#### Install to a specific agent

```bash
# Install both skills to a specific agent
npx skills add magallon/refactoring-toolkit --all -a claude-code

# Install only the audit skill to a specific agent
npx skills add magallon/refactoring-toolkit --skill refactoring-audit -a cursor
```

#### Install all skills to all agents

```bash
# Install everything to every detected agent (no prompts)
npx skills add magallon/refactoring-toolkit --all
```

#### Manual installation

```bash
git clone https://github.com/magallon/refactoring-toolkit.git

cp -r refactoring-toolkit/refactoring-audit /path/to/your/skills/
cp -r refactoring-toolkit/refactoring-execute /path/to/your/skills/
```

### Usage

**Run a full audit:**
> "Run a refactoring audit on this project"

**Run audit on a specific module:**
> "Audit only the authentication module for refactoring opportunities"

**Execute refactoring from an audit report:**
> "Using the audit report, create a refactoring plan and execute it"

## Stack Compatibility

These skills have been designed and tested to work with any web project. The agent automatically detects your stack and adapts its analysis. Examples of supported stacks:

- **Next.js / React** + Supabase, Prisma, or any backend
- **Laravel** + Vue/Blade + MySQL/PostgreSQL
- **Django** + React/HTMX + PostgreSQL
- **Rails** + Hotwire + PostgreSQL
- **Express** + Angular + MongoDB
- **SvelteKit / Nuxt / Remix** + any database
- **Static sites** with any tooling

The methodology evaluates universal principles: file organization, dead code, duplication, naming, complexity, error handling, security, and data validation. Framework-specific patterns are interpreted by the agent based on the detected stack.

## What Gets Audited

**Phase 1 — Structural Analysis**
File and directory organization, naming conventions, dead code detection (unused files, functions, imports, variables, orphan routes), dependency health, separation of concerns.

**Phase 2 — Code Quality**
Code duplication, naming and readability, cyclomatic complexity, pattern consistency (error handling, state management, data access, forms, typing discipline).

**Phase 3 — Security & Robustness**
Authentication and authorization in server-side handlers, input validation (client and server), error handling completeness, data access security, test coverage indicators.

## Severity Levels

Findings use a three-tier severity system:

| Level | Meaning | Example |
|---|---|---|
| 🔴 Critical | Immediate risk — security holes, data loss potential, broken functionality | Server-side handler without auth check |
| 🟡 Important | Significant maintainability or reliability issue | Duplicated logic across 5+ files |
| 🟢 Minor | Improvement opportunity, low risk | Inconsistent naming convention |

## Project Structure

```
refactoring-toolkit/
├── README.md                 # This file
├── LICENSE                   # MIT License
├── CHANGELOG.md              # Version history
├── CONTRIBUTING.md           # Contribution guidelines
│
├── refactoring-audit/        # Diagnosis skill
│   ├── SKILL.md
│   └── references/
│       ├── phase1-structural.md
│       ├── phase2-quality.md
│       ├── phase3-security.md
│       ├── severity-criteria.md
│       └── report-template.md
│
└── refactoring-execute/      # Execution skill
    ├── SKILL.md
    └── references/
        ├── prioritization-rules.md
        └── execution-checklist.md
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on reporting issues, proposing new audit checks, and submitting pull requests.

## License

MIT — See [LICENSE](LICENSE) for details.
