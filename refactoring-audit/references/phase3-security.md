# Phase 3 — Security & Robustness

Detailed evaluation criteria for authentication/authorization, input validation, error handling, data access security, and test coverage indicators.

## Table of Contents
- [1. Server-Side Handler Security](#1-server-side-handler-security)
- [2. Data Access Security](#2-data-access-security)
- [3. Error Handling Completeness](#3-error-handling-completeness)
- [4. Data Validation](#4-data-validation)
- [5. Test Coverage Indicators](#5-test-coverage-indicators)

---

## 1. Server-Side Handler Security

Server-side handlers include: server actions, API routes, controllers, views, resolvers, or any code that executes on the server in response to client requests. Adapt terminology to the detected stack.

### What to Check

**Authentication verification:**
- Does every server-side handler that accesses or modifies user data verify the user session at the start, before executing any logic?
- Are there handlers that skip authentication checks entirely?
- Is the auth check pattern consistent, or do some handlers implement it differently?

**Authorization verification:**
- When a handler receives an entity ID from the client (e.g., record ID, document ID, project ID), does it verify that the authenticated user has permission to access or modify that entity?
- Or does it trust that if the client sent the ID, the user must have access?
- This is distinct from authentication: a logged-in user shouldn't necessarily access all resources.

**Input handling:**
- Do handlers validate the type, shape, and range of inputs received from the client?
- Or do they pass client input directly to database queries, file system operations, or business logic?
- Are there handlers that receive arbitrary objects from the client and destructure them without validation?

**Server directive compliance** (for frameworks that require explicit server marking):
- Are all handlers that should be server-only properly marked? (e.g., `"use server"` in Next.js, server-only imports in SvelteKit)
- Could any handler accidentally execute on the client?

**Information exposure:**
- Do error messages returned to the client reveal implementation details? (stack traces, database column names, internal paths, query structures)
- Are server-side error details properly sanitized before reaching the client?

### Severity Guide

| Finding | Severity |
|---|---|
| Handler that modifies data without checking authentication | 🔴 Critical |
| Handler that accepts entity IDs without authorization check | 🔴 Critical |
| Client input passed directly to database queries without validation | 🔴 Critical |
| Missing server directive on a handler that should be server-only | 🔴 Critical |
| Stack traces or internal details exposed in client-facing error messages | 🟡 Important |
| Inconsistent auth check pattern across handlers | 🟡 Important |

---

## 2. Data Access Security

### What to Check

**Row-level security or equivalent:**
- If the database supports row-level security (RLS) or the framework provides policy-based access control (Pundit, Policies, CanCanCan, Gates, etc.):
  - Are access policies defined for all tables/models that contain user-specific or sensitive data?
  - Are there tables/models with RLS disabled or no policy defined that should have protection?
- If no built-in policy system exists:
  - Is access filtering applied consistently in the data access layer?
  - Is there a centralized authorization check, or is each query responsible for filtering by user?

**Correct client usage:**
- Is the right database/API client used in the right context? (server-side client for server code, client-side client for client code)
- Are there instances where a server-side client with elevated privileges is used in client-accessible code?
- Are credentials or connection strings hardcoded instead of using environment variables?

**Query safety:**
- Do queries that return a single record handle the case where no record exists? (e.g., `.single()` or `.first()` without null checking)
- Are there queries that could return more data than intended due to missing filters?
- Is there any raw SQL or raw query construction using string concatenation with user input?

**Defense in depth:**
- Does the application rely solely on one layer of protection, or are there multiple?
- Best practice: row-level policies + application-level authorization + handler-level auth checks
- If only one layer exists, flag the absence of defense in depth

### Severity Guide

| Finding | Severity |
|---|---|
| Sensitive table/model with no access control policy | 🔴 Critical |
| Server-side privileged client exposed to client code | 🔴 Critical |
| Raw query with string-concatenated user input | 🔴 Critical |
| Hardcoded database credentials | 🔴 Critical |
| Single record query without null handling | 🟡 Important |
| No defense in depth (only one security layer) | 🟡 Important |
| Query missing user filter while relying solely on policies | 🟢 Minor |

---

## 3. Error Handling Completeness

### What to Check

**Server-side error handling:**
- Do all server-side handlers have try/catch (or equivalent error handling mechanism)?
- Do they return a consistent response structure? (e.g., `{ success: boolean, data?: T, error?: string }`)
- Are there handlers where an unhandled exception could crash the process or return a raw error to the client?

**Silent failures:**
- Are there handlers or async operations where errors are caught but not logged, re-thrown, or communicated?
- Empty catch blocks, catch blocks that only have `console.log`, or catch blocks that swallow errors without any handling

**Client-side error handling:**
- Do client components handle error states from server calls, or only the success path?
- Are there forms or interactions where a server error would leave the UI in an inconsistent state (loading forever, stale data displayed, no error message)?

**Error boundaries / error pages:**
- Does the project have error boundary components or error pages for catching render-time errors?
- Are they placed strategically (per route, per feature), or is there only a single global error boundary?

**Error logging:**
- Are server-side errors logged somewhere (at minimum to server console, ideally to a logging service)?
- Or do errors get caught, formatted for the client, and then lost?

**User feedback accuracy:**
- Do success/error notifications (toasts, alerts, banners) accurately reflect what happened?
- Are there cases where an error could be silently treated as a success? (e.g., a mutation fails but the UI shows a success toast because the error wasn't checked)

### Severity Guide

| Finding | Severity |
|---|---|
| Handler without any error handling (unhandled exceptions reach the client) | 🔴 Critical |
| Error caught but silently swallowed (empty catch, no logging or communication) | 🔴 Critical |
| UI shows success when server returned error | 🔴 Critical |
| Client component doesn't handle error state from server call | 🟡 Important |
| Missing error boundary for a route that performs data mutations | 🟡 Important |
| Errors logged to console only (no persistent logging) | 🟡 Important |
| Inconsistent error response structure across handlers | 🟡 Important |
| Missing error boundary for a read-only route | 🟢 Minor |

---

## 4. Data Validation

### What to Check

**Validation approach:**
- Does the project use a validation library (Zod, Yup, Joi, class-validator, WTForms, etc.) or manual validation?
- If using a library, is it used consistently across all handlers, or only in some?
- If manual, is the validation thorough, or are there obvious gaps?

**Validation layers:**
- Is validation performed on **both** client and server, or only one?
- Client-only validation is a 🔴 Critical finding — client validation is for UX, server validation is for security. They are not interchangeable.
- Server-only validation is acceptable but 🟢 Minor — client validation improves user experience

**Unvalidated inputs:**
- Are there forms or client inputs that reach the server without any validation?
- Are there server-side handlers that accept input and use it without validating type, presence, format, or range?

**Type safety vs runtime validation:**
- Are TypeScript types, Python type hints, or similar static type annotations treated as if they provide runtime validation? (They don't — types are stripped at runtime)
- If the project relies on static types, are there critical paths (user input, external API data, file uploads) where runtime validation is missing?

**Validation completeness:**
- For validated inputs, are all relevant constraints checked?
  - Strings: length, format, allowed characters
  - Numbers: range, integer vs float, positive/negative
  - Enums: membership in allowed values
  - Dates: valid range, correct format
  - Files: size, type, extension
  - Arrays: length limits, item validation

### Severity Guide

| Finding | Severity |
|---|---|
| Client-only validation with no server-side validation on data mutations | 🔴 Critical |
| Server handler accepting unvalidated input for database writes | 🔴 Critical |
| File upload with no type or size validation | 🔴 Critical |
| Relying on TypeScript/type hints as runtime validation for external input | 🟡 Important |
| Validation library used inconsistently (some handlers use it, some don't) | 🟡 Important |
| Missing client-side validation (server-side present) | 🟢 Minor |
| Validation present but missing edge cases (e.g., no max length on strings) | 🟢 Minor |

---

## 5. Test Coverage Indicators

This is not a full test audit. The goal is to assess whether testing exists and where the most critical gaps are.

### What to Check

**Test existence:**
- Does the project have a test directory or test files?
- Are there test scripts in the package/dependency manifest?
- If no tests exist at all, this is a standalone finding

**Critical path coverage:**
- Are there tests for server-side handlers that handle data mutations?
- Are there tests for authentication/authorization logic?
- Are there tests for business logic (calculations, transformations, rules)?
- These three areas are the most important to have covered

**Test health:**
- Are there test files that are outdated (testing functions/components that have changed or been removed)?
- Are there skipped or disabled tests (skip, xit, xdescribe, pending)?
- Do tests actually assert outcomes, or are there tests that only check that code runs without throwing?

### Severity Guide

| Finding | Severity |
|---|---|
| No tests at all in the project | 🟡 Important |
| No tests for data mutation handlers | 🟡 Important |
| No tests for authentication logic | 🟡 Important |
| Multiple broken or skipped tests | 🟡 Important |
| Tests exist but don't cover business logic | 🟢 Minor |
| Tests have weak assertions (no meaningful outcome checks) | 🟢 Minor |
