# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.0.0] - 2025-02-27

### Added
- Initial release of the Refactoring Toolkit
- `refactoring-audit` skill with 3-phase diagnostic methodology
  - Phase 1: Structural Analysis (file organization, dead code, dependencies, separation of concerns)
  - Phase 2: Code Quality (duplication, naming, complexity, pattern consistency)
  - Phase 3: Security & Robustness (auth, input validation, error handling, test indicators)
  - Consolidated report with severity-based findings
  - Example findings in SKILL.md for agent output calibration
- `refactoring-execute` skill with plan-then-execute workflow
  - Three-tier prioritization (security → maintainability → cleanup)
  - Mandatory user confirmation before code changes
  - Incremental execution with verification checkpoints
  - Fallback instruction when no audit report is available
  - Reinforced version control warning for high-risk actions
  - Example plan actions in SKILL.md for agent output calibration
- Reference materials for severity criteria, report templates, prioritization rules, and execution checklists
- Full documentation (README, CONTRIBUTING, LICENSE)
