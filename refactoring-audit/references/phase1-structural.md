# Phase 1 — Structural Analysis

Detailed evaluation criteria for file organization, dead code detection, dependency health, and separation of concerns.

## Table of Contents
- [1. File and Directory Organization](#1-file-and-directory-organization)
- [2. Dead Code and Residual Artifacts](#2-dead-code-and-residual-artifacts)
- [3. Dependency Health](#3-dependency-health)
- [4. Separation of Concerns](#4-separation-of-concerns)

---

## 1. File and Directory Organization

### What to Check

**Directory structure consistency:**
- Does the project follow a recognizable organizational pattern? (feature-based, layer-based, domain-based, or hybrid)
- Is the chosen pattern applied consistently, or do some areas follow different conventions?
- Are there files placed in illogical locations that break the project's own conventions?

**Naming conventions:**
- Do files and directories follow a uniform naming convention? Common patterns: kebab-case for directories/files, PascalCase for component files, camelCase for utilities
- Identify any mixed conventions within the same directory level
- Check for meaningless names (utils.ts that contains unrelated functions, helpers/ as a dumping ground)

**Route/page organization** (if applicable):
- Does each route have the expected companion files for the framework? (e.g., layout, loading state, error boundary, metadata)
- Are there routes missing essential companion files?

### How to Evaluate

Walk the directory tree top-down. For each level, note the organizational pattern you observe. Flag deviations from that pattern. Don't impose an external convention — evaluate consistency with the project's own apparent conventions.

### Severity Guide

| Finding | Severity |
|---|---|
| Files in fundamentally wrong locations (e.g., database logic in UI components directory) | 🔴 Critical |
| Inconsistent naming convention across directories of the same type | 🟡 Important |
| Missing companion files for routes (error boundaries, loading states) | 🟡 Important |
| Minor naming deviations in isolated files | 🟢 Minor |

---

## 2. Dead Code and Residual Artifacts

### What to Check

**Unused files:**
- Files that are not imported by any other file in the project
- Check both direct imports and dynamic imports
- Verify entry points: some files may be used by the framework implicitly (layouts, middleware, config files, route handlers)

**Unused exports:**
- Functions, components, classes, hooks, or constants that are exported but never imported elsewhere
- Check for re-exports in index/barrel files that nobody consumes

**Unused imports:**
- Import statements that bring in modules or symbols not referenced in the file body

**Unused variables and types:**
- Variables declared but never read
- Type definitions or interfaces declared but never used as type annotations
- Constants defined but never referenced

**Orphan routes/pages:**
- Routes or pages that exist in the routing structure but have no navigation links pointing to them from anywhere in the application
- This doesn't apply to routes that are API endpoints or are accessed via direct URL

**Unused styling:**
- CSS classes, custom properties, or utility definitions in global stylesheets that no component references
- Styled components or CSS modules that are imported but not used

**Residual artifacts:**
- Code blocks that are commented out ("kept just in case")
- TODO, FIXME, HACK, XXX comments — list each with its location
- Console.log, debugger statements, or development-only code left in production paths

### How to Evaluate

For each category, use static analysis where possible:
- Trace import graphs to find disconnected files
- Search for export names across the entire project
- Grep for common residual patterns (TODO, console.log, commented blocks)

When you cannot definitively confirm something is dead code (e.g., it might be used dynamically, or by the framework implicitly), mark it as **"suspected dead code"** and explain the uncertainty.

### Severity Guide

| Finding | Severity |
|---|---|
| Entire files that are completely unused (confirmed) | 🟡 Important |
| 10+ unused exports across the project | 🟡 Important |
| Commented-out code blocks (each instance) | 🟢 Minor |
| TODO/FIXME comments (each instance, unless tagged with security/bug risk) | 🟢 Minor |
| Console.log/debugger in production code paths | 🟡 Important |
| Unused dependencies in package manifest | 🟡 Important |

---

## 3. Dependency Health

### What to Check

**Unused dependencies:**
- Packages listed in the dependency manifest (package.json, requirements.txt, Gemfile, composer.json, etc.) that are not imported or required anywhere in the source code
- Check both production and development dependencies

**Duplicate or overlapping dependencies:**
- Multiple packages that serve the same purpose (e.g., two date libraries, two HTTP clients, two validation libraries)
- Different versions of the same conceptual library

**Outdated dependencies with known issues:**
- You don't need to check every package for updates. Focus on:
  - Packages with known security vulnerabilities (if lockfile or audit output is available)
  - Major version gaps that suggest the project fell behind on critical updates

**Dependency weight:**
- Unusually heavy dependencies imported for minimal usage (e.g., importing all of lodash for one function)

### How to Evaluate

Read the dependency manifest. For each dependency, search the source code for imports or requires. If a dependency appears nowhere in the source code, flag it.

For duplicate detection, look for packages in the same category: HTTP clients, state management, date handling, validation, CSS-in-JS, icon sets, testing frameworks.

### Severity Guide

| Finding | Severity |
|---|---|
| Dependency with known security vulnerability | 🔴 Critical |
| 5+ unused dependencies | 🟡 Important |
| Duplicate packages serving the same purpose | 🟡 Important |
| 1-2 unused dependencies | 🟢 Minor |
| Heavy dependency used minimally | 🟢 Minor |

---

## 4. Separation of Concerns

### What to Check

**Server-side logic organization:**
- Are data mutation handlers (server actions, controllers, API routes, views) organized by domain or entity, or are there monolithic files that handle everything?
- Threshold: A single handler file covering more than 3 unrelated domains is a finding

**Business logic in UI components:**
- Do presentation components contain business logic (data transformation, complex calculations, authorization checks)?
- UI components should primarily handle rendering. Business logic belongs in dedicated layers (services, hooks, utilities, handlers)

**Data access layer:**
- Are database queries and API calls centralized (in a data access layer, repository, service, or similar), or scattered across UI components and handlers?
- Threshold: If the same table/endpoint is queried directly from 3+ different locations with different query patterns, that's a finding

**Configuration and constants:**
- Are magic numbers, magic strings, and configuration values hardcoded throughout the codebase, or centralized in config/constants files?

### How to Evaluate

For each UI component, check if it directly:
- Calls the database or ORM
- Performs complex data transformations (more than simple mapping/formatting)
- Contains business rules or authorization logic

For server-side handlers, check if each file covers a single domain/entity or mixes multiple unrelated operations.

### Severity Guide

| Finding | Severity |
|---|---|
| Database queries scattered across UI components (3+ locations querying same table) | 🔴 Critical |
| Monolithic handler file mixing 5+ unrelated domains | 🔴 Critical |
| Business logic embedded in presentation components | 🟡 Important |
| Monolithic handler file mixing 3-4 unrelated domains | 🟡 Important |
| Magic numbers/strings in more than 5 files | 🟡 Important |
| Minor mixing of concerns in isolated cases | 🟢 Minor |
