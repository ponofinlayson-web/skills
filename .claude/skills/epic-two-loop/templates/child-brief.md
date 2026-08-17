# Child Brief Template

Fill every section. The child cannot see the parent conversation, ask clarifying
questions, or read your mind - anything not written here does not exist for it.
Rendered briefs are typically 40-80 lines.

---

## Goal
<One sentence: what this ticket delivers and why it matters to the epic.>

## Context you must load first
<Paths to read BEFORE planning the work: the implementation plan (always),
architecture doc sections that apply, existing code the change touches.>

- Plan: `<path to this ticket's plan file>` (the source of truth - follow it task-by-task)
- Architecture: `<path>` (sections: <which>)
- Code: `<paths>`

## Constraints
<Hard rules: language/framework conventions, files NOT to touch, compat
requirements, test expectations, anything from the parent's hard-won lessons.>

## Definition of done
<Checkable statements. Must include:>

- All tasks in the plan completed
- Full test suite green (name the command to run)
- No regressions outside the ticket's scope

## Report back
<Exactly what to return so the parent can verify without re-reading the diff:>

- Commit hash(es) and branch name
- Tests: command run + pass/fail counts
- Any deviation from the plan, with reason
- Anything discovered that affects sibling tickets (the parent owns routing this)
