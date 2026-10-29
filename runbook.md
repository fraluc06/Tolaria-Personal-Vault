---
type: Type
_icon: book
_color: "#f59e0b"
_order: 1
status: Active
trigger:
verified_against:
---

# Runbook

A Runbook is a repeatable operational procedure: a numbered, checkable sequence of steps that solves one concrete problem, written to be executed again months later without re-reasoning.

Use this type when the note has:

- a **trigger** — the situation that calls for the procedure (frontmatter `trigger:`)
- ordered **steps with verification** — after each step there is a check that tells you it worked
- **failure modes** — a troubleshooting section (symptom → cause → fix)
- an **end state** — you know the runbook is done when a final verification passes

For reference sheets (presets, options), curated lists, or one-off gotchas, keep `type: Note`.

`trigger:` and `verified_against:` come as empty placeholders on new Runbooks: fill `trigger` with the "when do I need this" one-liner, and `verified_against` with the tool version / OS / date the procedure was last tested.
