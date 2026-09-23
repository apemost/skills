---
name: use-subagents
description: Decide whether to use subagents and run bounded handoffs with current known and sufficient task context, explicit ownership, collision-safe execution, structured returns, and independently verified output acceptance. Use when delegating, reviewing worker results, or choosing local-only execution.
license: MIT
---

# Use subagents

Use task management to maintain durable parent and child state. Use this skill to decide whether to delegate, define the handoff, isolate edits, collect the result, and verify it.

## Decide

Delegate only bounded workstreams with clear outputs. Run them concurrently only when they can advance without blocking the next critical-path step; otherwise stay local or delegate sequentially.

- Use the simplest available harness operation that safely provides the required isolation and return path.
- Prefer local execution for a factual answer, a tiny tightly-coupled edit, or when the next step depends on one immediate result.
- Run workstreams concurrently only when each has a separate reviewable output, no sequential dependency, and no shared mutable state or overlapping write paths. Otherwise stay local or delegate sequentially.
- If you choose to stay local after considering delegation, state the concrete reason.

## Prepare task context

Before delegating, establish the known task context needed for the assignment. Refresh the task file first when the work is tracked. Include the information a fresh worker would otherwise have to rediscover or guess. Context is sufficient when it supports the assignment; it need not answer every possible question.

Ensure the available context covers:

- Goal, scope, context, acceptance criteria, constraints, non-goals, and definition of done.
- Relevant paths, references, decisions, current state, dependencies, and blockers.
- Decomposition, ownership boundaries, expected outputs, and required verification evidence.

State relevant unknowns, assumptions, evidence gaps, and pending decisions explicitly. Do not turn an unknown into an unstated assumption merely to make the handoff appear complete.

Pass the exact parent task path in every initial or follow-up handoff. Require the worker to read the relevant sections before starting. Require a complete parent-task read when scope, constraints, acceptance criteria, ownership, or prior decisions can affect the work; otherwise identify the sections to read. The task file is the canonical durable context. A conversation fork or prose summary does not replace it.

Keep the parent task file read-only for every worker. The Coordinator records assignments and accepted state; workers return candidate results without editing the parent.

If the task-management skill explicitly skips an ephemeral request, put equivalent context directly in the handoff and state that no parent task file exists.

## Scope the handoff

Use the parent task for durable shared context and the handoff only for assignment-specific additions. Specify:

- **Task context.** Give the exact parent path, the required sections or complete-read requirement, and any explicit unknowns or assumptions; or state why tracking was skipped.
- **Goal.** The exact question or deliverable, phrased so the subagent knows when it is done.
- **Scope.** The files, modules, systems, or behaviors to inspect or edit. Name paths.
- **Boundaries.** What not to touch, especially other agents' ownership areas and anything out of scope.
- **Parent task access.** State `read-only` for every worker.
- **Context overlay.** Include only subtask-specific facts not already captured in the parent.
- **Output.** The expected format, required paths, and conclusions the Coordinator can act on. For searches, require file paths and a direct answer, not a survey.
- **Verification.** The commands, checks, or evidence the subagent must run or return.
- **Ownership.** Assign explicit file or area ownership; use an explicit no-file ownership statement for read-only tasks.

Prefer narrow, answerable handoffs ("does X call Y, and where?") over open-ended surveys ("explore the repo").

## Durable worker state

Use a child task only when a workstream needs durable recovery across sessions or substantial independent evidence. Before delegation, the Coordinator must allocate and create a standard `task-N.md` and link it bidirectionally with the parent. Give one worker exclusive write ownership of the child task file; source-file ownership still follows the handoff. Do not let workers allocate task numbers themselves.

## Concurrent edits

Parallel edits can collide, so prevent overlap:

- Assign disjoint files or modules to each editor; overlap can cause reverts or clobbering.
- Tell every code-editing worker explicitly: *you are not alone in the codebase; do not revert edits made by others, adapt to concurrent changes, and report every path you changed.*
- Serialize genuinely conflicting write surfaces or use the harness's isolated-workspace capability.
- Never use direct parent task-file edits as a worker return channel.
- Treat the changed-paths report as required input to your integration step.

## Worker return contract

Require every worker to return:

```text
Assignment: <id>
Outcome: complete | partial | blocked
Direct answer or deliverable: <result>
Changed paths: <paths> | none
Verification run: <commands and results> | none
Evidence: <paths, logs, or observations>
Open risks or conflicts: <details> | none
Recommended next action: <action>
```

## Output acceptance gate

Treat every worker return as candidate work. The main agent must verify the result independently; a worker's success claim or test summary alone is insufficient.

Before integrating or reporting a worker result:

1. Inspect every changed path or artifact through its diff, content, logs, screenshots, or other evidence.
2. Compare the result with the task's acceptance criteria, scope, boundaries, ownership, and definition of done. Check for unintended changes and missing deliverables.
3. Rerun or reproduce the relevant checks with the project-approved toolchain and collect evidence proportional to the risk.
4. Resolve contradictions between workers and source evidence. Prefer reproducible evidence over unsupported conclusions.
5. Send incomplete, conflicting, or unverified work back with a focused correction request, reassign it, or finish it locally. Repeat this gate after corrections.
6. Record accepted outputs and verification evidence in the task file. For tracked delegation, this is the parent task file; then refresh its current state and remaining work.

If independent verification is impossible, record the exact boundary and do not present the result as verified.

## Main agent responsibilities

The main agent remains accountable for integration:

- Continue any unblocked local work while the worker runs. Do not reimplement the delegated scope; still perform the acceptance checks above.
- Integrate only outputs that pass the acceptance gate. Resolve conflicting findings before integration.
- Keep the parent task file, cross-workstream context, and final verification under main-agent ownership.
- Report verification boundaries explicitly before handing work back to the user.
