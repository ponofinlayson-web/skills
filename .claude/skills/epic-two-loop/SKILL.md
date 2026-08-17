---
name: epic-two-loop
description: Runs a complete epic build in two loops - planning (PRD, architecture, ticket slicing) then per-ticket build with parent verification gates, dispatching each ticket to a fresh child conversation or running it parent-direct by size. Use when the user says "run the two-loop pipeline", "full epic build", "plan then build the whole thing", "take this idea through PRD to shipped tickets", or invokes /epic-two-loop.
argument-hint: [product idea] · [optional: paths to research / reference docs to ground in]
---

# Epic Two-Loop Pipeline

Delivers a greenfield feature from idea to merged tickets through two loops: Loop 1 plans in this conversation, Loop 2 builds ticket-by-ticket with verification gates. The end artifact is a fleet of shipped tickets plus a final report table.

This is a **sequencer**: it orchestrates other skills and never restates their instructions. Each step below names the skill to run and the argument to pass. The added value here is the **hand-off contract** (what flows between steps) and the **gates** (when not to proceed).

## When to use

- A feature big enough to deserve a PRD before code (greenfield or a major addition).
- The user wants planning AND execution in one pipeline, not just a plan.
- For a single small task, skip this - run `piv-implement` directly.

## Loop 1 - Planning (all in this conversation)

1. Run `plan-create-prd` (`.claude/skills/plan-create-prd/SKILL.md`) with: **$ARGUMENTS**.
   Note the PRD path for every later step.
2. Run `plan-architecture` (`.claude/skills/plan-architecture/SKILL.md`) with the PRD path.
   Note the architecture doc path. Architecture decides HOW; do not let it leak task lists.
3. Run `piv-slice-epic` (`.claude/skills/piv-slice-epic/SKILL.md`) with PRD + architecture paths.
   Note the ticket IDs and the dependency graph.

**GATE - user sign-off before build.** `piv-slice-epic` writes real tickets to a tracker (side-effecting). Present the ticket list + dependency order and get explicit user approval before Loop 2 starts.

## Loop 2 - Build (per ticket, in dependency order)

For each ticket from the slice, in dependency order:

### Step 1 - Dispatch decision

- Ticket touches one small surface (≤ ~3 files, no new subsystem) → run **parent-direct** in this conversation.
- Bigger than that, or it needs a long focused implementation session → **delegate** to a fresh child conversation / Task subagent. The child cannot see this conversation; before dispatching, write the brief by reading `templates/child-brief.md` and following it exactly. Hand the child the plan path, never a summary of it.

### Step 2 - Per-ticket sequence

1. Run `piv-plan-implementation` (`.claude/skills/piv-plan-implementation/SKILL.md`) with the ticket + architecture doc → note the plan path.
2. Execute the plan: run `piv-implement` (`.claude/skills/piv-implement/SKILL.md`) with the plan path (parent-direct), or have the child execute it (delegated).
3. Run `piv-validate` (`.claude/skills/piv-validate/SKILL.md`). **GATE: suite must be green before continuing.** On failure, fix in place before proceeding.
4. Run `piv-commit` (`.claude/skills/piv-commit/SKILL.md`) on the ticket's own branch.

### Step 3 - Parent verification gate (EVERY ticket, both modes)

Never trust the implementer's self-report - child conversations especially. Before moving to the next ticket:

1. Re-run the validation suite yourself in this conversation.
2. **Live-smoke the deliverable's primary interface** (hit the endpoint, open the page, run the binary) - not just its tests.
3. Only then merge (or open the PR via `piv-create-pr`, `.claude/skills/piv-create-pr/SKILL.md`) and mark the ticket done in the tracker.

If verification fails, fix it here in the parent before starting the next ticket. A ticket that leaves the gate red blocks its dependents.

## Final report

After the last ticket, produce:

| Ticket | Mode (child/parent) | Commit / PR | Verification |
|---|---|---|---|

Plus: epic-level summary, anything descoped, and follow-ups discovered mid-build.

## Gotchas

- **Don't inline child-skill instructions.** If a step needs more than the one-liner here, the detail belongs in that skill, not this sequencer.
- **The child brief is the artifact that determines child quality.** A lazy brief yields a lost child that guesses. Use the template verbatim.
- **Gates are hard stops**, not suggestions. The two that get skipped under time pressure (user sign-off before tickets; parent verification before merge) are exactly the two that prevent the expensive failures.
- **Ticket failure mid-epic**: if a ticket can't go green, stop the loop and report - do not silently continue to dependents or pile up red tickets.

## Resources

- `templates/child-brief.md` - the required shape for delegating a ticket to a child conversation. Read before every delegation.
