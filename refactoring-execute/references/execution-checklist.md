# Execution Checklist

Pre-flight checks, per-action verification, and post-completion procedures.

---

## Pre-Flight Checks (Before Any Execution)

Run these checks once before starting Step 2:

- [ ] **Version control status**: Is the working tree clean? Run `git status` or equivalent. Warn the user if there are uncommitted changes.
- [ ] **Build/compile check**: Does the project currently build without errors? Run the project's build or type-check command. If it fails before you start, inform the user — existing errors are not your responsibility, but you need to know the baseline.
- [ ] **Test baseline** (if tests exist): Run the test suite. Record how many tests pass and fail. This is the baseline to compare against after refactoring.
- [ ] **Plan confirmed**: Has the user explicitly approved the plan or a subset of it?

Record the pre-flight results. You'll compare against them after execution.

---

## Per-Action Verification

After each individual refactoring action (or grouped unit), verify:

### For All Changes

- [ ] **Syntax valid**: The modified files parse without syntax errors
- [ ] **No new import errors**: All imports resolve correctly
- [ ] **Build passes**: If the project has a build step, it still completes successfully
- [ ] **Type check passes** (if applicable): No new type errors introduced

### For Code Extraction / Centralization

- [ ] **New utility works**: The extracted function, component, or module is syntactically valid and exports what it should
- [ ] **All consumers updated**: Every file that used the old inline version now imports the shared version
- [ ] **Old code removed**: The duplicated inline versions have been deleted from their original locations
- [ ] **No orphaned imports**: The old code's specific imports (if any) are cleaned up from consumer files

### For Dead Code Removal

- [ ] **Re-verified unused**: Before deleting, searched the entire project one more time for references to the code being removed
- [ ] **Framework conventions checked**: Verified the file isn't used implicitly by the framework (middleware, layouts, plugins, configuration, route conventions)
- [ ] **Dynamic usage checked**: Searched for dynamic imports, string-based references, or reflection that might reference the code

### For Pattern Standardization

- [ ] **All instances updated**: Every occurrence of the old pattern has been migrated to the new pattern
- [ ] **No hybrid state**: The project doesn't have a mix of old and new patterns (unless intentionally phased)
- [ ] **Edge cases preserved**: Any special handling in the old pattern's outlier cases is preserved in the new pattern

### For Security Fixes

- [ ] **Auth check added**: The handler now verifies authentication before any logic
- [ ] **Auth check tested mentally**: Walk through the code path — is there any way to reach the protected logic without passing the auth check?
- [ ] **Authorization verified**: If entity IDs are received, the handler checks the user's permission on that entity
- [ ] **Validation added**: Input is validated before use, using the project's validation approach
- [ ] **Error response sanitized**: No internal details leak to the client

---

## Per-Priority Completion

After completing all actions in a priority tier:

- [ ] **Build passes**: Full build/compile check
- [ ] **Type check passes** (if applicable): Full type check
- [ ] **Tests pass**: Run the full test suite. Compare results against the pre-flight baseline. If tests that previously passed now fail, investigate before proceeding.
- [ ] **Summary delivered**: Present the completion summary to the user (see report format in SKILL.md)
- [ ] **User confirmation**: Wait for user to confirm before starting the next priority tier

---

## Post-Completion (After All Priorities)

After all approved priorities are executed:

### Final Verification

- [ ] **Full build passes**: Clean build from scratch
- [ ] **All tests pass**: Full test suite matches or exceeds pre-flight baseline
- [ ] **No regression in test count**: Same number of tests pass (or more, if tests were fixed). Flag any decrease.

### Deliverables

Provide the user with:

1. **Change summary**: Total files modified, created, and deleted
2. **What to verify manually**: Things the agent cannot verify (visual changes, runtime behavior, integration with external services, user-facing flows)
3. **New issues discovered**: Any problems found during execution that weren't in the original audit
4. **Remaining items**: Any plan items that were skipped, deferred, or need user decisions
5. **Recommended follow-up**: Suggest running the `refactoring-audit` skill again after the changes have settled to verify improvement and catch any new issues introduced

### Commit Recommendation

Suggest the user commit the changes with a descriptive message. Recommend one commit per priority tier for easy rollback:

```
refactor: priority 1 — security fixes (auth checks, input validation)
refactor: priority 2 — pattern standardization, deduplication, error handling
refactor: priority 3 — dead code removal, naming, cleanup
```

If the user prefers more granular commits, suggest one commit per action group.
