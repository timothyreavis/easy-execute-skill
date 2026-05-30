---
name: easy-execute
description: "Use when Timothy invokes easy execute or clearly asks Codex to take a goal, plan, spec, docs, roadmap item, launch-readiness item, repo-maintenance task, or review/repair effort through a multi-phase execution workflow where the main agent acts as coordinator/chief of staff: clarify only what is blocking, brief and delegate plan drafting, foundation research, research, coding, testing, docs/artifacts, review, fixes, and verification to subagents, evaluate validity of returned work, re-delegate as needed, and close with evidence."
---

# Easy Execute

## Operating Mode

Act as the accountable execution lead for the current thread. The main agent is the coordinator, not the primary worker. Once this skill is invoked, keep using this workflow for the same objective until the work is complete, paused, or redirected, even if later user messages do not repeat the skill name.

At activation, state the current phase in one short sentence. For long-running work, preserve continuity by periodically naming the active phase, accepted blockers, next action, and verification target in normal chat updates.

Activation and checkpoint messages are not deliverables. Do not end the turn after saying Easy Execute is active, summarizing current state, or naming the next slice unless the user asked only for status, planning, or a handoff.

If the phase is ready for implementation and there are no blocking questions, immediately begin the next concrete implementation action in the same turn. That action should be inspecting just enough context to brief subagents, creating/updating the task plan, and spawning bounded workers. Do not stop at "the next slice should be..."

When work spans turns, interruptions, or possible context compaction, leave a compact checkpoint in chat:

`Easy Execute state: objective=...; phase=...; plan/artifact=...; active agents/processes=...; findings=accepted/rejected/deferred...; next verification=...`

After a resume or compaction, if this checkpoint is still in context and the user has not redirected the thread, treat Easy Execute as still active and restate the current phase before continuing.

Use existing domain skills when they apply. This skill coordinates the work; it does not replace repo-specific, security, browser, document, or production-incident skills.

## Core Rules

- Start from the real artifact: current repo state, existing plan, spec, docs, issue, deployment, logs, screenshots, or live behavior.
- Ask questions only when the answer is blocking and cannot be discovered safely.
- Keep the main thread responsible for orchestration, worker briefs, tradeoff decisions, validity judgments, integration decisions, resource cleanup, and final acceptance.
- Check dirty state before edits. Do not revert unrelated changes. Treat unexpected diffs as user-owned unless proven otherwise.
- Delegate execution work to subagents when subagent tools are available: research, code writing, docs/artifact drafting, tests, verification passes, adversarial review, and fix implementation.
- The main thread should inspect only enough local context for routing, briefing, and output evaluation. Do not use main-thread inspection to form final research findings when subagents are available.
- The main thread should not do solo research, solo implementation, solo verification, or solo review when delegation is possible.
- Direct main-thread edits are last-resort exceptions: subagents unavailable/exhausted, a tiny mechanical integration patch after worker output, or an urgent unblocker too small to delegate. State the exception before editing.
- Treat subagents as exhausted only after tool failures, max-agent limits, repeated unusable outputs, or unavailable capability. State the specific reason before falling back.
- If subagent tools are unavailable, state the fallback and run distinct named planning, implementation, and review passes in the main thread.
- Default delegated subagents to low reasoning. Raise to medium/high/xhigh only for architecture, security, data loss, billing, auth, migration, production, or ambiguous cross-system decisions.
- Prefer the inherited model for subagents. Override model only when the task clearly benefits from a cheaper/faster or stronger/specialized model.
- Do not leave subagents, dev servers, watchers, or other processes running after they stop being useful.
- Never accept reviewer findings blindly. Fix valid findings; reject invalid findings with a concrete rationale.

## Subagent Tool Hygiene

Before spawning new subagents, audit the active agent pool:

- Track each active agent by id/name, assigned lane, expected output, and status in the thread checkpoint.
- Close completed, idle, obsolete, duplicate, or unusable agents before opening new ones.
- If the spawn limit is hit, first close unneeded agents and retry; do not immediately fall back to main-thread work.
- Reuse or resume an existing agent only when the new task depends on its context; otherwise spawn a fresh bounded agent.
- Do not pile multiple unrelated tasks onto one unresolved agent thread.
- If an agent stalls, returns unusable output, or drifts from scope, close or redirect it with a tighter prompt instead of waiting indefinitely.
- If workers conflict, pause new delegation, evaluate the boundary, then re-delegate a focused integration or repair task.
- Keep waits sparse and purposeful. While agents run, the main thread should do non-overlapping coordination work, not duplicate their task.
- Record the exact reason for any downgrade to main-thread fallback: no subagent tool, spawn failure after cleanup, max-agent limit after cleanup, repeated unusable outputs, or capability mismatch.

Reasoning selection:

- Low: mechanical search, docs parity, simple tests, focused fixtures, narrow verification.
- Medium: bounded implementation, ordinary code review, small refactors, non-critical planning.
- High: architecture, security, data integrity, auth, billing, production behavior, ambiguous cross-system work.
- Xhigh: irreversible decisions, destructive migrations, incident-critical judgment, or major cross-system architecture.

## Foundation Standard

Before implementation, have planner/research subagents identify the foundation the requested work needs to remain useful over years, not just for the immediate feature. The main thread evaluates and accepts the foundation decisions.

Think through likely future variants, adjacent workflows, ownership boundaries, data contracts, extension points, migrations, observability, permissions, and failure modes. Build the requested feature on stable seams that can support those future additions.

Do not implement speculative features. Do not add heavy abstractions, generic frameworks, or unused layers only because future work is possible. The target is a durable foundation for plausible expansion, with the current scope implemented simply and explicitly.

Use this foundation decision path:

- Existing foundation fits: build on it and preserve its contracts.
- Existing foundation is close but incomplete: extend it narrowly at the right seam.
- No foundation exists and future variants are plausible: create the smallest durable foundation needed for the current feature and likely next variants.
- The change is isolated, tiny, or unlikely to repeat: do not create new foundation; make the direct change.

For tiny or low-risk changes, delegate a light foundation check or keep it to one or two coordinator-reviewed sentences. For substantial work, have subagents draft three to six foundation decisions before editing. Each decision should say what is being made stable now and what future expansion it keeps possible. Add new abstractions only when current duplication, contract instability, or known near-term variants justify them.

## Worker Quality Bar

Include these constraints in implementation, research, planning, review, and fix briefs when they apply:

- Surface assumptions, ambiguity, and tradeoffs instead of silently choosing an interpretation.
- Prefer simple, explicit code. Do not add speculative flexibility, configuration, or abstraction.
- Match existing style and ownership boundaries. Do not refactor adjacent code, comments, or formatting unless the assigned task requires it.
- Treat existing foundations as the default path. Reuse or extend them before proposing a new structure.
- Remove only dead code, imports, or artifacts created by the assigned change. Mention unrelated cleanup instead of doing it.
- Make every changed line trace to the assigned slice, accepted finding, or verification requirement.
- Define success criteria and verification for the assigned slice before returning.
- If the worker's first approach grows too large for the task, simplify before handing it back.

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

For new or incomplete plans, define the artifact requirements and delegate drafting to planner/research subagents. The main thread evaluates, accepts, revises, or re-delegates the draft. The resulting durable artifact should make execution clear:

- goal and non-goals
- source-of-truth boundaries
- user-visible behavior
- data/API contracts
- foundation and future-extension considerations
- implementation slices
- acceptance criteria
- verification plan
- rollback or recovery notes when production risk exists

Use planner/research subagents to gather context, inspect source artifacts, challenge architecture, and draft or improve plans. Use reviewer subagents after a concrete plan, artifact, or diff exists. Give them the artifact and acceptance bar, not your intended answer.

### 3. Delegate Work

Before spawning subagents, define the coordination path and the worker lanes.

Run the Subagent Tool Hygiene checklist before each delegation wave, especially after long-running work, review rounds, or failed spawn attempts.

Use workers for disjoint implementation slices, research threads, fixture/test additions, docs/artifact drafting, docs parity, focused verification, and fix implementation. Assign each worker explicit ownership of files, modules, or responsibilities. Tell workers they are not alone in the codebase and must not revert unrelated edits.

For code changes, spawn implementation workers before the main thread writes code. For review, spawn reviewer workers before the main thread evaluates findings. For research, spawn explorer/research workers before the main thread draws conclusions. The main thread may inspect files for routing and briefing only, but should not continue into solo execution unless one of the direct-edit exceptions applies.

Every subagent brief should include:

- objective
- repo/path scope
- owned files or responsibilities
- files or areas off-limits
- whether edits are allowed
- commands allowed or expected
- constraints and source-of-truth artifacts
- applicable Worker Quality Bar items
- expected output
- evidence required
- instruction not to edit or integrate unless explicitly assigned

Use at least one worker for even single-slice work when subagents are available. Scale concurrency as the work can absorb it, up to the available maximum. Six is a ceiling, not a target. Prefer fewer agents only when the work cannot be split safely or coordination cost would exceed parallelism.

When a worker finishes, evaluate its changed files or output before integrating. Inspect each worker boundary before stacking another worker's edits on top. Delegate follow-up fixes or verification when possible. The main thread owns the final accepted result.

### 4. Implement

Coordinate building only the accepted scope through implementation workers, using the repo's existing patterns and the foundation decisions from the plan. Keep APIs small, names clear, and behavior explicit.

Treat this phase as worker-led. The main thread coordinates implementation, evaluates worker output, decides what to accept/reject, directs integration through workers, and delegates verification. Main-thread integration is limited to documented last-resort or tiny mechanical patches. If the main thread implements directly, disclose which last-resort exception allows it.

If the implementation reveals that the plan is wrong, pause the slice long enough to update the plan and explain the adjustment. Do not silently widen scope.

### 5. Review

Use the `adversarial-review-loop` skill for any non-trivial implementation, production-facing artifact, plan-to-code execution, review/repair phase, or when the user asks for peer review, subagents, voting rounds, or re-review. Review work should be done by subagent reviewers when available; the main thread evaluates the validity of their findings.

Choose reviewer count by risk:

- Tiny low-risk edit: one focused reviewer when subagents are available; main-thread review only under fallback or last-resort exceptions.
- Normal feature or artifact: two reviewers.
- Cross-cutting, public-facing, data-backed, or release-sensitive work: three reviewers.
- Security, auth, billing, destructive migration, production incident, or major architecture: four to six reviewers with role diversity.

Useful reviewer roles include correctness, architecture/foundation, security/privacy, production/ops, product/UX, docs/contracts, strategy/commercial, and copy/content.

### 6. Triage and Repair

Classify findings as accepted, rejected, or deferred.

- Accepted: concrete issue that affects correctness, maintainability, user experience, launch readiness, safety, or the stated acceptance criteria.
- Rejected: incorrect, already covered, out of scope, or based on a false assumption. Record the rationale.
- Deferred: valid but outside the current scope. Note the follow-up only when it matters.

Delegate fixes for accepted blocker, high, and medium findings when subagents are available. Re-review changed areas with subagent reviewers until no accepted blocker/high/medium findings remain. Do not stop merely because tests pass.

If subagents are unavailable or exhausted, perform a main-thread re-review with the same finding taxonomy and disclose the downgrade.

### 7. Verify and Close

Delegate the smallest verification set that proves the work. The main thread evaluates the evidence and only runs checks directly under the documented fallback or last-resort exceptions:

- typecheck/lint/unit tests for code contracts
- integration or browser checks for user workflows
- build checks for deployability
- migration/dry-run checks for data changes
- live/log checks for production incidents
- direct artifact rereads for docs and plans

Before final response, check active subagents, dev servers/watchers, temp artifacts, verification commands, and unresolved risks. Close unneeded subagents and stop unneeded local processes. Report what changed, what verified it, accepted findings fixed, rejected findings with rationale, deferred findings with next step, remaining blocker/high/medium count, unresolved risks, and the next concrete step if one remains.
