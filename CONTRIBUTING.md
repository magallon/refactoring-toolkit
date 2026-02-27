# Contributing to Refactoring Toolkit

Thank you for your interest in improving these skills. Contributions from the community make this toolkit better for everyone.

## How to Contribute

### Reporting Issues

If you find a problem — a false positive, a missing check, an unclear instruction, or a bug in the methodology — open an issue with:

1. **Which skill** is affected (`refactoring-audit` or `refactoring-execute`)
2. **What happened** — describe the problem clearly
3. **What you expected** — what should have happened instead
4. **Context** — your stack, project size, and any relevant details
5. **Reproducibility** — can you trigger it consistently?

### Proposing New Audit Checks

If you think the audit should check for something it currently doesn't:

1. Open an issue titled `[New Check] Brief description`
2. Explain **what** the check would detect
3. Explain **why** it matters (what problems does it catch?)
4. Suggest a **severity level** (Critical / Important / Minor) with reasoning
5. Note which **phase** it belongs to (Structural / Quality / Security)
6. Provide an **example** of code that would trigger the finding

### Submitting Pull Requests

1. Fork the repository
2. Create a branch from `main` (`git checkout -b feature/your-change`)
3. Make your changes
4. Ensure your changes follow the conventions below
5. Submit a PR with a clear description of what changed and why

### Conventions

- **Skills and technical content** are written in English
- **SKILL.md files** must follow the [Agent Skills specification](https://agentskills.io)
- **SKILL.md** must stay under 500 lines; move detailed content to `references/`
- **Frontmatter** must have valid `name` and `description` fields
- **Severity criteria** must be objective and include concrete thresholds where possible
- **New checks** should be stack-agnostic — avoid framework-specific language in core instructions

### What Makes a Good PR

- Focused on a single change (don't bundle unrelated fixes)
- Includes a clear explanation of why the change improves the toolkit
- Doesn't break existing functionality or methodology flow
- Follows the progressive disclosure pattern (keep SKILL.md lean, details in references)

## Code of Conduct

Be respectful, constructive, and assume good intent. We're all here to make better software.
