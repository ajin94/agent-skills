---
name: run-tests
description: Detect the test framework and run tests, optionally filtered by a pattern. Use when the user wants to run tests, check test coverage, or verify a fix.
allowed-tools: Bash Read
arguments:
  - name: pattern
    description: Optional test name or file pattern to filter (e.g. "auth", "*.spec.ts")
disable-model-invocation: false
---

## Project context

!`ls package.json pyproject.toml Cargo.toml go.mod 2>/dev/null | head -5`

## Instructions

Detect the test framework from project files, then run the appropriate command:

| Framework | Command |
|---|---|
| Jest / Vitest | `npm test -- $pattern` or `npx vitest $pattern` |
| pytest | `pytest $pattern -v` |
| Go test | `go test ./... -run $pattern` |
| Cargo | `cargo test $pattern` |
| RSpec | `bundle exec rspec $pattern` |

If `$pattern` is empty, run the full suite.

Report:
- Pass / fail counts
- Any failing test names and the error
- Suggested fixes for failures if obvious
