---
name: easy-execute
description: "Use when Timothy invokes easy execute or clearly asks Codex to take a goal, plan, spec, docs, roadmap item, launch-readiness item, repo-maintenance task, or review/repair effort through a multi-phase execution workflow: clarify only what is blocking, create or inspect durable plans, design the long-term foundation, delegate subagents, implement, adversarially review, fix valid findings, re-review, verify, and close with evidence."
---

# Easy Execute

## Operating Mode

Act as the accountable execution lead for the current thread. Once this skill is invoked, keep using this workflow for the same objective until the work is complete, paused, or redirected, even if later user messages do not repeat the skill name.

At activation, state the current phase in one short sentence. For long-running work, preserve continuity by periodically naming the active phase, accepted blockers, next action, and verification target in normal chat updates.

Activation and checkpoint messages are not deliverables. Do not end the turn after saying Easy Execute is active, summarizing current state, or naming the next slice unless the user asked only for status, planning, or a handoff.

If the phase is ready for implementation and there are no blocking questions, immediately begin the next concrete implementation action in the same turn. For non-trivial code changes, that action should be inspecting enough context to brief workers, creating/updating the task plan, and spawning bounded workers. Do not stop at "the next slice should be..."

When work spans turns, interruptions, or possible context compaction, leave a compact checkpoint in chat:

`Easy Execute state: objective=...; phase=...; plan/artifact=...; active agents/processes=...; findings=accepted/rejected/deferred...; next verification=...`

After a resume or compaction, if this checkpoint is still in context and the user has not redirected the thread, treat Easy Execute as still active and restate the current phase before continuing.

Use existing domain skills when they apply. This skill coordinates the work; it does not replace repo-specific, security, browser, document, or production-incident skills.

## Core Rules

- Start from the real artifact: current repo state, existing plan, spec, docs, issue, deployment, logs, screenshots, or live behavior.
- Ask questions only when the answer is blocking and cannot be discovered safely.
- Keep the main thread responsible for quality, integration, tradeoffs, and final acceptance.
- Check dirty state before edits. Do not revert unrelated changes. Treat unexpected diffs as user-owned unless proven otherwise.
- For code implementation, delegate-first when subagent tools are available. The main thread owns critical-path decisions, planning, worker briefs, integration, review, verification, and final acceptance; it should not be the primary code author for non-trivial implementation.
- Direct main-thread edits are exceptions: trivial one-file changes, mechanical integration after worker output, merge/test fallout, emergency fixes, or fallback when subagents are unavailable/exhausted. State the exception before editing.
- If subagent tools are unavailable, state the fallback and run distinct named planning, implementation, and review passes in the main thread.
- Default delegated subagents to low reasoning. Raise to medium/high/xhigh only for architecture, security, data loss, billing, auth, migration, production, or ambiguous cross-system decisions.
- Prefer the inherited model for subagents. Override model only when the task clearly benefits from a cheaper/faster or stronger/specialized model.
- Do not leave subagents, dev servers, watchers, or other processes running after they stop being useful.
- Never accept reviewer findings blindly. Fix valid findings; reject invalid findings with a concrete rationale.

Reasoning selection:

- Low: mechanical search, docs parity, simple tests, focused fixtures, narrow verification.
- Medium: bounded implementation, ordinary code review, small refactors, non-critical planning.
- High: architecture, security, data integrity, auth, billing, production behavior, ambiguous cross-system work.
- Xhigh: irreversible decisions, destructive migrations, incident-critical judgment, or major cross-system architecture.

## Foundation Standard

Before implementation, identify the foundation the requested work needs to remain useful over years, not just for the immediate feature.

Think through likely future variants, adjacent workflows, ownership boundaries, data contracts, extension points, migrations, observability, permissions, and failure modes. Build the requested feature on stable seams that can support those future additions.

Do not implement speculative features. Do not add heavy abstractions, generic frameworks, or unused layers only because future work is possible. The target is a durable foundation for plausible expansion, with the current scope implemented simply and explicitly.

For tiny or low-risk changes, keep the foundation check to one or two sentences. For substantial work, record three to six foundation decisions before editing. Each decision should say what is being made stable now and what future expansion it keeps possible. Add new abstractions only when current duplication, contract instability, or known near-term variants justify them.

## Workflow

### 1. Orient

Determine which phase the thread is in:

- Discovery: the goal is still fuzzy.
- Planning: the user wants questions, specs, docs, or cards.
- Plan review: a plan exists and needs critique before build.
- Implementation: the plan/spec is accepted enough to execute.
- Review and repair: code or artifacts exist and need adversarial review.
- Closeout: work is done and needs verification, summary, and handoff.

For non-trivial work, keep a visible task plan with the current step in progress. Create it after initial artifact inspection. Update it on phase changes, after subagent completion, after accepted findings, and before closeout.

Before promising delegation or review rounds, check whether subagent tools are available. If they are not, use the main-thread fallback and say so.

If orientation shows the plan is accepted, reviews are clean, and no blockers remain, advance directly to Delegate Implementation for non-trivial code work. Do not return a status-only response.

### 2. Inspect and Plan

Read the relevant existing artifacts before drafting or editing. If the user provides an existing plan, treat it as the starting source of truth and check whether it is complete enough to execute.

For new or incomplete plans, produce the smallest durable artifact that makes execution clear:

- goal and non-goals
- source-of-truth boundaries
- user-visible behavior
- data/API contracts
- foundation and future-extension considerations
- implementation slices
- acceptance criteria
- verification plan
- rollback or recovery notes when production risk exists

Use planner subagents before implementation only when scope, architecture, or acceptance criteria are uncertain. Use reviewer subagents after a concrete plan, artifact, or diff exists. Give them the artifact and acceptance bar, not your intended answer.

### 3. Delegate Implementation

Before spawning subagents, define the local critical path and the delegable side work.

Use workers for disjoint implementation slices, research threads, fixture/test additions, docs parity, or focused verification. Assign each worker explicit ownership of files, modules, or responsibilities. Tell workers they are not alone in the codebase and must not revert unrelated edits.

For non-trivial code changes, spawn at least one implementation worker before the main thread writes code. The main thread may inspect files to create good briefs, but should not continue into solo implementation unless one of the direct-edit exceptions applies.

Every subagent brief should include:

- objective
- repo/path scope
- owned files or responsibilities
- files or areas off-limits
- whether edits are allowed
- commands allowed or expected
- constraints and source-of-truth artifacts
- expected output
- evidence required
- instruction not to edit or integrate unless explicitly assigned

Keep no more than the useful number of concurrent agents running. Six is a ceiling, not a target. Prefer fewer agents when coordination cost would exceed parallelism.

When a worker finishes, review its changed files or output before integrating. Inspect each worker boundary before stacking another worker's edits on top. Run `git diff` and targeted tests after integration. The main thread owns the final diff.

### 4. Implement

Build only the accepted scope, using the repo's existing patterns and the foundation decisions from the plan. Keep APIs small, names clear, and behavior explicit.

Treat this phase as worker-led for non-trivial code. The main thread coordinates implementation, reviews worker output, applies or refines integration, and runs verification. If the main thread implements directly, disclose which exception allows it.

If the implementation reveals that the plan is wrong, pause the slice long enough to update the plan and explain the adjustment. Do not silently widen scope.

### 5. Review

Use the `adversarial-review-loop` skill for any non-trivial implementation, production-facing artifact, plan-to-code execution, review/repair phase, or when the user asks for peer review, subagents, voting rounds, or re-review.

Choose reviewer count by risk:

- Tiny low-risk edit: main-thread review or one focused reviewer.
- Normal feature or artifact: two reviewers.
- Cross-cutting, public-facing, data-backed, or release-sensitive work: three reviewers.
- Security, auth, billing, destructive migration, production incident, or major architecture: four to six reviewers with role diversity.

Useful reviewer roles include correctness, architecture/foundation, security/privacy, production/ops, product/UX, docs/contracts, strategy/commercial, and copy/content.

### 6. Triage and Repair

Classify findings as accepted, rejected, or deferred.

- Accepted: concrete issue that affects correctness, maintainability, user experience, launch readiness, safety, or the stated acceptance criteria.
- Rejected: incorrect, already covered, out of scope, or based on a false assumption. Record the rationale.
- Deferred: valid but outside the current scope. Note the follow-up only when it matters.

Fix accepted blocker, high, and medium findings. Re-review changed areas until no accepted blocker/high/medium findings remain. Do not stop merely because tests pass.

If subagents are unavailable, exhausted, or no longer worth the coordination cost, perform a main-thread re-review with the same finding taxonomy and disclose the downgrade.

### 7. Verify and Close

Run the smallest verification set that proves the work:

- typecheck/lint/unit tests for code contracts
- integration or browser checks for user workflows
- build checks for deployability
- migration/dry-run checks for data changes
- live/log checks for production incidents
- direct artifact rereads for docs and plans

Before final response, check active subagents, dev servers/watchers, temp artifacts, verification commands, and unresolved risks. Close unneeded subagents and stop unneeded local processes. Report what changed, what verified it, accepted findings fixed, rejected findings with rationale, deferred findings with next step, remaining blocker/high/medium count, unresolved risks, and the next concrete step if one remains.
