---
name: summarize-changes
description: Summarize uncommitted git changes and flag risks. Use when the user asks what changed, wants a commit message, or needs a diff review before committing.
allowed-tools: Bash(git *)
disable-model-invocation: false
---

## Current changes

!`git diff HEAD 2>/dev/null || echo "No git repository or no changes"`

## Staged only

!`git diff --cached 2>/dev/null`

## Instructions

Summarize the changes in 2–3 concise bullet points, then list any risks:
- Missing error handling
- Hardcoded values (secrets, URLs, IDs)
- Untested code paths
- Breaking API changes

If no VCS is detected, analyze recently modified files instead.
