---
name: gather-requirements
description: Read an input file and transform its content into a structured, LLM-readable requirements document. Prompts the user for an output directory (with a sensible default) before saving. Use when formalizing a spec, plan, rough notes, or brief into structured requirements.
model: sonnet  # Required for Steps 1–4 (exploration, clarification, drafting). Step 5 is mechanical I/O and would run on a smaller model if per-step model selection were supported.
allowed-tools: Read Write Bash(mkdir *) Bash(ls *) Bash(date *) AskUserQuestion
arguments:
  - name: input_file
    description: Path to the input file to convert into a requirements document (e.g. brief.txt, spec.md, notes.txt)
disable-model-invocation: false
---

## Overview

Convert the raw content of `$input_file` into a structured, descriptive requirements document that an LLM agent can read and act on without access to the original file. Save it to a user-confirmed output directory under the project's own subfolder.

### Model usage

This skill declares `model: sonnet` because Steps 1–4 — reading the input, clarifying ambiguities with the user, and drafting the structured requirements — require strong reasoning. Step 5 is purely file I/O and does not benefit from Sonnet; it runs under the same model only because Claude Code skills support exactly one `model:` field in frontmatter. Do not invoke this skill for tasks that consist only of saving or reformatting an existing requirements doc.

---

## Step 1 — Validate the input file

Check that the file exists and is readable:

!`ls "$input_file" 2>/dev/null && echo "FILE_OK" || echo "FILE_NOT_FOUND"`

If the result is FILE_NOT_FOUND, stop immediately. Ask the user:
> "I couldn't find the file at `$input_file`. Please provide the correct path to the input file."

If no `$input_file` argument was provided at all, ask:
> "Which file should I convert into a requirements document? Please provide the file path."

---

## Step 2 — Read and understand the input

Read the full content of `$input_file`. Identify:

- **Input type**: feature brief, user story, meeting notes, product spec, PRD, whiteboard dump, email thread, etc.
- **Domain**: web app, mobile, API, infrastructure, data pipeline, ML model, internal tooling, etc.
- **Primary stakeholders** mentioned or implied
- **Explicit requirements** (stated clearly)
- **Implicit requirements** (strongly implied but not stated)
- **Ambiguities**: anything that could be interpreted in more than one way
- **Gaps**: information that would be needed to implement this but is absent

---

## Step 3 — Clarify ambiguities (if needed)

If the input has 1–4 items that are genuinely ambiguous or missing and would materially affect the requirements, use `AskUserQuestion` to ask them now before writing the document.

Only ask questions that:
- Have a high impact on the requirements (e.g. auth strategy, target platform, scale, integrations)
- Cannot be reasonably inferred from the input

Do NOT ask questions about things that are clear, or that can be noted as assumptions in the document.

If there are more than 4 blockers, proceed with the most reasonable assumptions, document them all in Section 5, and list them as open questions in Section 8.

---

## Step 4 — Generate the requirements document

Derive a short, descriptive title for this requirement (e.g. "User Authentication Flow", "Payment Integration", "Admin Dashboard").
Derive two identifiers from the input:
- **Project name**: a short kebab-case folder name identifying the broader project or product (e.g. `ns-interiors`, `patternix`, `admin-portal`)
- **Slug**: a kebab-case name for this specific requirement, max 40 chars (e.g. `user-auth-flow`, `payment-integration`)

Write a complete requirements document using the template below. Every section must be written as full, standalone prose and structured lists — no shorthand, no "see above", no references to the original file — so the document is self-contained for an LLM reader.

```markdown
# Requirement Document: [Title]

**Source File**: $input_file
**Generated**: [today's date in YYYY-MM-DD format]
**Status**: Draft
**Slug**: [slug]
**Domain**: [domain]

---

## 1. Overview

[2–4 sentences. What is being built or changed, why it exists, and for whom. A reader with no prior context should understand the full scope after reading this section.]

## 2. Goals & Objectives

[Bulleted list of measurable, outcome-oriented goals. Tie each goal to a stakeholder need or business driver where identifiable. Avoid vague goals like "improve UX" — make them specific.]

## 3. Functional Requirements

[Numbered list. Each item must be:
- Atomic — one requirement per line
- Testable — can be verified as pass/fail
- Written as: "The system shall..." or "The feature must..."
Group related requirements under sub-headings (e.g. ### 3.1 Authentication) if the list exceeds 8 items.]

## 4. Non-Functional Requirements

[Only include what is stated or strongly implied by the input. Cover relevant items from: performance targets, security requirements, accessibility (WCAG level), browser/device support, scalability, availability/SLA, data retention, compliance/regulatory.]

## 5. Constraints & Assumptions

[Two sub-lists:
**Constraints**: hard limits (tech stack, third-party systems, deadlines, budgets, team size).
**Assumptions**: things taken as true for planning purposes that need validation (e.g. "Assumes users are already authenticated before reaching this feature").]

## 6. Out of Scope

[Explicit list of things this requirement does NOT include that a reader might assume are covered. This prevents scope creep and sets clear implementation boundaries.]

## 7. Acceptance Criteria

[A checklist of verifiable conditions. When all items are true, the requirements are satisfied. Use Given/When/Then format where useful:
- [ ] Given [context], when [action], then [expected result]
- [ ] ...]

## 8. Open Questions

| # | Question | Owner | Impact if Unresolved |
|---|----------|-------|----------------------|
| 1 | [question] | [team/person] | [what breaks or blocks] |
| ... | | | |

[If no open questions, write: "No open questions — all items resolved or documented as assumptions."]

## 9. Glossary

[Define every domain-specific term, acronym, or concept used in this document. An LLM reading this document later must be able to understand all terminology without external context.]

---

## LLM Context Notes

[Meta-instructions for an AI agent that will read this document to implement, test, or extend the feature. Include:
- Which sections are load-bearing for code generation (usually §3 and §7)
- Priority order if requirements conflict
- Known gaps to handle gracefully
- Any section that represents assumptions vs. confirmed decisions]
```

---

## Step 5 — Save the document

1. Determine the default output directory in this order:
   - If the environment variable `REQUIREMENTS_DIR` is set, use it.
   - Otherwise, if `./knowledgeBase/requirements/` exists relative to the current working directory, use that.
   - Otherwise, fall back to `$HOME/knowledgeBase/requirements/`.

2. Use `AskUserQuestion` to confirm the destination with the user. Present the computed default as the suggested path and allow them to override it with any absolute or relative path:
   > "Where should I save the requirements document? Accept the default or provide a different path."
   > Default: `<computed-default>/[project-name]/`

3. Resolve the chosen path to an absolute path (expand `~` if present). Store it as `$output_dir`.

4. Ensure the project subfolder exists:

!`mkdir -p "$output_dir/[project-name]"`

5. Get today's date:

!`date +%Y-%m-%d`

6. Save the document to `$output_dir/[project-name]/[slug]-[YYYY-MM-DD].md` using the Write tool.

7. Echo the final absolute path back so the user can locate the file.

---

## Step 6 — Report to the user

Summarize in 3–4 lines:
- The file path where the document was saved
- A one-sentence summary of what was captured
- Any open questions or assumptions the user should review
