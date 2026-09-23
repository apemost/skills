---
name: task-manager
description: Create and maintain durable task files for actionable work, including scope, lifecycle status, plan-quality checks, assumptions and unknowns, acceptance criteria, verification evidence, single-writer coordination, child-task ownership, recovery, and done state.
license: MIT
---

# Task manager

Create and maintain lightweight task files for actionable work. Each file records the working agreement between the user and agent: scope, current status, assumptions and unknowns, acceptance criteria, decisions, evidence, verification, blockers, and the current definition of done.

## Operating rules

- Treat actionable work as tracked by default. Skip tracking only when the request is clearly ephemeral or the user explicitly opts out.
- Do not use "multi-step" as the deciding test. Most real tasks become multi-step once implementation, verification, and follow-up are included.
- Store task files under `.local/tasks/YYYY-MM-DD/task-N.md` unless the user, project, or workspace specifies another location.
- Store new supporting documents under `.local/docs/YYYY-MM-DD/` unless project conventions say otherwise.
- Keep task files compact. Link to evidence instead of pasting large logs, reports, or transcripts.
- Recovery path: `Summary -> Activity Log -> linked evidence`.

## Tracking decision

Before substantive work, choose exactly one action:

1. Continue an existing task when the user references a task file explicitly, asks to continue or update prior tracked work, or the new request is clearly the next phase of an unfinished tracked task.
2. Create a new task for a new actionable request when no matching task file already exists.
3. Skip task tracking when the request is clearly ephemeral:
   - Simple Q&A
   - Git commit requests
   - Purely conversational replies
   - Tiny one-shot edits with no meaningful follow-up
   - Requests that explicitly say not to create or update a task

For one continuous scope, continue the matching task instead of creating one task per message.

## Minimal context

Before creating or continuing a task, gather the minimum context needed to frame the work correctly:

- The most relevant files, docs, issue links, logs, or error messages
- The current task file, if this work is already tracked
- The minimum success signals or verification expectations, when they are known

Keep this pass bounded. Do not do broad research just to open a task.

## New task

When a new actionable request does not already have a matching task file, follow these steps in order:

1. Do the minimal context pass above
2. Distill the task into a short, durable `Description`
3. Add `### References` when supporting material will help future work. Prefer file paths, URLs, task paths, ticket IDs, or log locations
4. Add `### Success Signals` when there are concrete checks, tests, screenshots, or output expectations that define "done"
5. If the user does not specify another location, determine the task directory as `.local/tasks/YYYY-MM-DD/` using today's date from the system context
6. Create the directory if it does not exist
7. Determine the next task number by listing existing `task-*.md` files in that directory, extracting the maximum numeric suffix, and adding 1. If none exist, start from 1
8. Create the file using the canonical template below
9. Continue with the user's substantive request unless the user asked only for task creation

When checking existing files, ignore names that do not match the `task-N.md` pattern.

## Canonical template

```markdown
# Task N

## Description

[What needs to be done and the intended scope.]

### References

- [Relevant files, tasks, docs, tickets, logs, or links]

### Success Signals

- [Optional tests, screenshots, or output checks that define done]

## Activity Log

> Maintained by Agent: Append records as list items after each meaningful task event. Prefix with the speaker's name followed by a colon (e.g. `Andrew:` / `Claude:` / `Codex:` / `Gemini:`). The Agent must use the active assistant's actual model or tool name, not generic terms like "AI" or "Assistant". Keep entries concise and chronological. Put created artifacts, evidence, and deliverables as indented sub-items.

## Summary

> Maintained by Agent: Treat this section as the current task state. Keep only the currently valid state. Link to evidence instead of pasting long logs or report bodies.

- Status: active | blocked | done | superseded
- Goal: ...
- Constraints: ...
- Assumptions / Unknowns:
  - Assumption: ...
  - Unknown: ...
- Current Decisions:
  - ...
- Verification:
  - Passed: ...
  - Failed: ...
  - Not run: ...
  - Boundary: ...
- Evidence Links:
  - [Doc A](...)
  - [Log B](...)
- Open Questions / Blockers:
  - ...
- Next Steps:
  1. ...
  2. ...
  3. ...
```

Omit `### References` or `### Success Signals` when they would be empty. Under `Verification`, omit unused result categories instead of leaving placeholders. Use `- None.` when `Assumptions / Unknowns` has no current entries.

## Task sections

- `Description`: Preserve the original ask, accepted scope, references, and success signals. Update it only when scope is clarified or materially changes.
- `Activity Log`: Record a concise chronological timeline of meaningful user inputs, agent actions, created artifacts, verification results, and blocker changes.
- `Summary`: Act as the current authoritative task state for fast recovery. It should explain the lifecycle status, current goal, constraints, assumptions and unknowns, decisions, verification state and boundary, evidence, blockers, and next steps without requiring the full history.

## Plan quality gate

Apply this gate when a task contains or links an implementation decomposition. It improves the execution plan without requiring a separate plan document or a second source of truth for lifecycle state.

### Reviewable boundaries

- Make each planned unit an independently testable deliverable with a meaningful review boundary. Split where a reviewer could reject one unit while accepting its neighbor.
- Fold setup, configuration, scaffolding, documentation, and cleanup into the deliverable that needs them. Do not create standalone steps that produce no independently useful result.
- Separate genuinely independent subsystems when each can produce working, testable value on its own. Do not split tightly coupled work merely to increase task count.
- Before locking the decomposition, map the files or surfaces each unit creates or changes and state their responsibilities. Follow the codebase's existing structure unless restructuring is part of the accepted scope.

### Interfaces and completeness

- For dependent units, record `Consumes` and `Produces` using exact interface names, signatures, parameter and return types, data shapes, or other constraints that neighboring work relies on.
- Preserve project-wide constraints such as version floors, dependency limits, naming rules, platform requirements, and compatibility promises in one authoritative place and make them apply to every planned unit.
- Map every accepted requirement and success signal to at least one planned unit and to concrete verification evidence.
- Reject placeholders such as `TBD`, `TODO`, "add error handling", "handle edge cases", "write tests", or "similar to another task" when they leave the implementer to guess behavior, scope, or verification.
- Check that names, types, fields, paths, and interfaces stay consistent across units. A consumer must reference exactly what its producer defines.

### Plan self-review

Before implementation begins, review the task or linked plan with fresh eyes:

1. Coverage. Can every requirement and success signal be pointed to a planned unit and verification check?
2. Placeholders. Does any step defer a material decision or omit the content needed to execute it?
3. Interface consistency. Do dependent units agree on names, types, data shapes, and constraints?
4. Execution readiness. Are paths, ownership boundaries, expected outputs, and verification commands or evidence concrete enough to start safely?

Fix discovered gaps before implementation. If the detailed decomposition would make the task file hard to recover from, store it in a linked implementation artifact; keep the task file as the single authority for lifecycle, scope, decisions, acceptance, and current state.

## Multi-agent task ownership

Keep durable task-state ownership separate from delegation mechanics. Use `use-subagents` to decide, scope, hand off, and accept delegated work; use this skill to govern who may change task files and how parent/child state is recorded.

### Parent task ownership

- Designate one active `Coordinator`. Treat the Coordinator as the single writer for the entire parent task file, including `Description`, `Activity Log`, and `Summary`.
- Keep `Activity Log` append-only and named for traceability. This is an append-only history rule, not a concurrent-write primitive.
- Give workers complete read access to the parent task but no write access. Have workers return results through the available harness instead of editing the parent.
- Before delegation, have the Coordinator refresh the parent, record the active assignment in `Summary`, and append a named delegation event to `Activity Log`.

For active delegation, add a compact optional block to `Summary`:

```markdown
- Coordinator: ...
- Delegation:
  - `<assignment-id>`
    - Owner: ...
    - Parent task access: read-only
    - File ownership: ... | none
    - Expected output: ...
    - Required verification: ...
    - Status: pending | in_progress | accepted | rejected | blocked
```

Keep active ownership in `Summary`; keep delegation, acceptance, rejection, and transfer events in `Activity Log`.

### Durable child tasks

- Allocate and create each durable child task before delegation using the next standard `task-N.md` path from the normal numbering procedure.
- Link the child from the parent and the parent from the child's `References`. State the worker's exclusive child-task ownership, parent read-only access, output contract, and acceptance criteria.
- Do not let workers allocate task numbers or create competing child paths concurrently.
- Let the assigned worker write its child task exclusively while active. The Coordinator reads it, receives the return, independently accepts or rejects the result, and then updates the parent.
- Use a child task only when the work needs durable recovery across sessions or substantial independent evidence. Keep bounded work in the parent assignment and worker return.

### Coordinator transfer

Transfer parent ownership sequentially. Stop the previous writer first. The new Coordinator must then read the complete task, record themselves in `Summary`, and append the transfer to `Activity Log` before making other edits.

## Updates

Update the task after meaningful work. Do not wait for the user to ask.

Treat supplementary input, clarifications, and feedback as meaningful task events regardless of when they arrive. Acknowledge and record them promptly, then assess their priority against the current work instead of assuming the latest input comes first.

### Activity log rules

Append `Activity Log` entries only after meaningful task events such as:

- Important user feedback or course corrections
- Material decisions or direction changes
- Created or updated artifacts
- Verification runs and notable results
- Newly discovered or resolved blockers
- Completion, handoff, or pause points that matter for later recovery

Do not log every file read, shell command, or minor thought. Keep entries concise and useful.

#### Handling user feedback and supplementary input

For each user input that supplements, corrects, or extends tracked work:

1. Acknowledge the input so the user knows it has been registered.
2. Distill what changed, what was clarified or added, and any new constraint or preference.
3. Prioritize the input against the current execution context:
   - Blocking corrections ("that approach is wrong", "stop, this is not what I meant"): interrupt immediately and adjust before continuing.
   - Scope or direction changes: record them and revise `Next Steps`; if the current atomic step is nearly complete, it may be reasonable to finish it first.
   - Supplementary context or clarifications that do not invalidate current work: record them in `Activity Log` and integrate them at the next natural breakpoint.
   - Additional tasks or nice-to-haves: record them in `Activity Log`, add them to `Next Steps` according to their priority, and continue the current work.

   Do not default to "last in, first out." Weigh urgency, dependency, and the cost of context-switching against the value of finishing in-progress work.

4. Record an `Activity Log` entry prefixed with the user's name. Summarize the input and the prioritization decision.
5. Update every affected `Summary` field in the same turn. This includes Status, Goal, Constraints, Assumptions / Unknowns, Current Decisions, Verification, Open Questions, and Next Steps. If no field changes, note that decision in `Activity Log` and leave `Summary` unchanged.

This covers answers to open questions, scope or priority changes, acceptance-criteria updates, design preferences, added tasks, and corrections to agent misunderstandings.

Use this format:

```markdown
- Codex: Redesigned the task structure for the skill
  - Updated `skills/task-manager/SKILL.md`
```

Prefix entries with the actual speaker or tool name, such as `Andrew`, `Codex`, `Claude`, or `Gemini`. Do not use generic labels like "AI" or "Assistant".

Put artifacts, deliverables, or evidence as indented sub-items instead of embedding long content inline.

### Summary rules

Treat `Summary` as the current authority, not a status diary.

- Keep only the currently valid state
- Overwrite stale intermediate notes instead of accumulating them
- Refresh the summary after each meaningful phase of work and before ending the turn
- When user feedback changes goal, scope, constraints, decisions, or priorities, update the affected Summary fields in the same turn as the Activity Log entry. Do not defer the update to a later phase
- Ensure the summary can answer: lifecycle status, current goal, constraints, assumptions and unknowns, decisions, verification state and boundary, evidence links, blockers or open questions, and next steps
- Prefer short linked evidence over pasted logs or report bodies

### Persistent state rules

- Treat `Status` as the explicit task lifecycle state:
  - Use `active` while work remains and can proceed.
  - Use `blocked` when a named condition prevents meaningful progress; record that condition under `Open Questions / Blockers`.
  - Set `done` only after the Success Signals are satisfied, accepted outputs are integrated, and the actual verification state and boundary are recorded.
  - Stop work on the old scope and link the replacement task when using `superseded`.
- Keep `Assumptions / Unknowns` current. Prefix each entry with `Assumption:` or `Unknown:`, state its effect on execution or conclusions when material, and remove or convert entries when evidence resolves them.
- Do not turn an unknown into an unstated assumption. Unknowns do not automatically become blockers; classify them as blockers only when they prevent the next meaningful action.
- Record actual verification state under `Verification`:
  - Use `Passed` for checks that ran successfully and name the check or result.
  - Use `Failed` for checks that ran unsuccessfully and retain unresolved impact.
  - Use `Not run` for relevant checks that were skipped or unavailable and give the exact reason.
  - Record the exact verification boundary, including environments, behaviors, or evidence not covered.
- Omit unused result categories instead of leaving placeholders. Evidence links do not imply verification success, and planned checks belong in `Success Signals` or `Next Steps`, not `Passed`.

For `Open Questions / Blockers`, write `- None.` when nothing is unresolved. If the task is complete, set `Status` to `done`, write `1. None. Task complete.` under `Next Steps`, and remove stale follow-up items.

### Evidence rules

When the evidence is too long for the task file, store it elsewhere and link it back:

- Prefer existing outputs, logs, screenshots, PRs, or generated artifacts when they already exist
- If you need a new text artifact and the user did not specify another location, prefer a Markdown document under `.local/docs/YYYY-MM-DD/` unless project conventions require another location
- Link that artifact from both `Activity Log` and `Summary` when it materially affects the current state

## Recovery

When resuming work from an existing task file:

- Read `Summary` first
- Then read the most recent `Activity Log` entries that explain how the task reached its current state
- Open linked evidence only when the summary and recent activity log are insufficient
- If the summary is missing key fields or is obviously stale, repair it before doing deeper work. When continuing an older task without the persistent state fields, add them from current evidence; record unknown or not-run state instead of inferring success.

## Completion and handoff

Before you end a substantive turn on tracked work:

- Ensure `Activity Log` records the latest meaningful actions
- Refresh `Summary` so it reflects the current truth, not the previous state
- Ensure `Status`, `Assumptions / Unknowns`, and `Verification` reflect the latest accepted state
- If the task is complete, set `Status` to `done`, say so explicitly, and capture the final evidence and verification boundary
- If the task is blocked, set `Status` to `blocked`, name the blocker, and make the next step clear
- If there are no further actions, say so directly in `Next Steps` instead of leaving stale planning text

Use task files to reduce context loss across long runs, compaction, and later sessions. Do not let the task file become another place where large, stale logs accumulate.
