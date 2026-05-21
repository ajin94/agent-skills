---
name: code-review
description: Review code quality, correctness, and style for a specific file or the current diff. Use when the user asks for a code review, PR review, or quality check.
allowed-tools: Read Bash(git diff *)
arguments:
  - name: target
    description: File path or "diff" to review the current git diff (default: diff)
disable-model-invocation: false
---

## Instructions

Review $target (default: current git diff).

Focus on:
1. **Correctness** — logic errors, off-by-one, null/undefined, race conditions
2. **Security** — injection, XSS, secrets in code, improper auth checks
3. **SOLID violations** — single responsibility, open/closed, dependency coupling
4. **Readability** — naming, unnecessary complexity, missing edge case handling

Format output as:
- Severity: `[critical | warning | suggestion]`
- Location: file:line
- Issue and recommended fix

If $target is a file path, read it first. If $target is "diff" or omitted, use `git diff HEAD`.
