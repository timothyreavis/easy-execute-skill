---
name: easy-execute
description: "Use when a user invokes easy execute or clearly asks Codex to take a goal, plan, spec, docs, roadmap item, launch-readiness item, repo-maintenance task, or review/repair effort through a multi-phase execution workflow where the main agent acts as coordinator: clarify only what is blocking, brief and delegate plan drafting, foundation research, research, coding, testing, docs/artifacts, review, fixes, and verification to subagents, evaluate validity of returned work, re-delegate as needed, and close with evidence."
---

# Easy Execute

## Operating Mode

Act as the accountable execution lead for the current thread. The main agent is the coordinator, not the primary worker. Once this skill is invoked, keep using this workflow for the same objective until the work is complete, paused, or redirected, even if later user messages do not repeat the skill name.

At activation, state the current phase in one short sentence. For long-running work, preserve continuity by periodically naming the active phase, accepted blockers, next action, and verification target in normal chat updates.

Activation and checkpoint messages are not deliverables. Do not end the turn after saying Easy Execute is active, summarizing current state, or naming the next slice unless the user asked only for status, planning, or a handoff.

If the phase is ready for implementation and there are no blocking questions, immediately begin the next concrete implementation action in the same turn. That action should be inspecting just enough context to brief subagents, creating/updating the task plan, and spawning bounded workers. Do not stop at "the next slice should be..."

When assigned a scoped Linear issue or worker brief, take the slice as far as reasonably possible without waiting for another coordinator prompt. If the plan becomes clear and the brief allows edits, continue through implementation, focused review, fixes, verification, commit/push when permitted, and Linear closeout. Stop only for a real blocker: unsafe data loss, missing access, explicit no-edit/scout-only scope, unrelated dirty state that affects the slice, resource gates, failing verification that cannot be resolved in scope, or a required product decision.

When work spans turns, interruptions, or possible context compaction, leave a compact checkpoint in chat:

`Easy Execute state: objective=...; phase=...; delivery_target=...; plan/artifact=...; active agents/processes=...; findings=accepted/rejected/deferred...; next verification=...`

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
- When this skill runs inside a delegated worker thread, treat the delegation brief as the source of truth for that slice. You may use bounded internal subagents when useful, but do not spawn extra Codex threads unless explicitly asked; finish the assigned slice or report the exact blocker.
- Default delegated subagents to low reasoning. Raise to medium/high/xhigh only for architecture, security, data loss, billing, auth, migration, production, or ambiguous cross-system decisions.
- Prefer the inherited model for subagents. Override model only when the task clearly benefits from a cheaper/faster or stronger/specialized model.
- Do not leave subagents, dev servers, watchers, or other processes running after they stop being useful.
- Treat local resources as constrained. Before spawning several workers, starting browser automation, creating worktrees, installing dependencies, or running broad verification, check disk/memory/process pressure and lower concurrency or pause for cleanup when the system is tight.
- Never accept reviewer findings blindly. Fix valid findings; reject invalid findings with a concrete rationale.

## Delegated Worker Intake

When starting from a Linear issue, worker brief, or delegated Codex thread, normalize the assignment before editing. In the first useful checkpoint, state:

- issue or objective id
- repo/path/worktree and current branch
- whether the thread is active, a continuation, duplicate, superseded, or blocked before edits
- delivery target: scout-only, local-ready, committed, pushed, deployed, or live verified
- commit, push, merge, deploy, and provider-mutation policy
- the source of truth that must prove success: tests, build, live page, provider UI/API, logs, Linear, or a specific artifact

If thread-title tools are available, rename worker threads early to a short issue/objective label. Do not leave the thread title as a pasted delegation block when it can be corrected.

If evidence shows another worker, branch, or commit already completed or superseded the same issue, do not keep parallel-editing by default. Stop with a concise handoff, or continue only as an explicitly scoped reviewer, verifier, or integration worker.

For provider, browser, deploy, ads, analytics, or external-account work, never infer account state from repo code, old notes, or another thread's browser session. If the required UI/API session is not available to the acting thread, record that as the blocker and do not invent state.

## Commit And Push Handling

- Follow the brief's commit policy exactly. If the brief forbids commit/push, stop at a reviewed local-ready state unless the coordinator explicitly changes the policy.
- At the start of implementation, classify commit policy as forbidden, allowed, or required. If the brief says "commit and push when verified" or the issue expects a pushed branch, treat push as required after clean verification, not optional.
- If the objective requires committed or pushed work, do not stop at local-ready. After accepted blocker/high/medium findings are resolved and verification passes, commit only the scoped files.
- If commit/push is allowed by the brief, treat it as part of closeout after clean review and verification; do not wait for a separate "commit it" prompt.
- Before committing, prove branch, HEAD, dirty state, staged file names, and staged name-status. Abort if unrelated files are staged or required files are unexpectedly dirty.
- Use a concise commit message tied to the issue. Push only the named branch or target in the brief, then record commit hash, push range, verification, and final status in Linear.
- If an implementation worker, sidecar, or earlier turn already created commits or advanced a remote, verify actual local and remote refs before adding follow-up commits. Closeout must describe the real delivery state, including any unexpected target-branch movement; do not pretend a change stayed branch-only when remote proof shows otherwise.
- Keep deploy separate from commit/push. Use the repo's current deploy owner and record whether deployment and live verification are done, pending, or blocked.
- When a pushed branch is the expected delivery, include a merge-readiness judgment in closeout: target branch, source head, whether the branch is clean and pushed, conflict-check status if run, verification evidence, migration/deploy blockers, and whether the coordinator should merge automatically.
- Do not leave work at "pushed" without naming the next integration action. If the branch appears merge-ready, say so plainly. If not, name the exact blocker.
- Do not mark Linear `Done` merely because a local commit exists. If the branch is not pushed/deployed/live verified and the issue is not explicitly local-only, leave or move the issue to `In Review` and state the exact next integration step.
- If a worker updates Linear itself, its state must match delivery reality: `local-ready`, `committed`, `pushed`, `deployed`, and `live verified` are different closeout states.
- When work is merged and pushed, do not assume the worker branch or worktree disappeared. Treat follow-on branch/worktree cleanup as part of integration closeout. Remove only worktrees proven safe by the resource-hygiene rules below, or report the exact cleanup blocker.

## Subagent Tool Hygiene

Before spawning new subagents, audit the active agent pool:

- Track each active agent by id/name, assigned lane, expected output, and status in the thread checkpoint.
- Close completed, idle, obsolete, duplicate, or unusable agents before opening new ones.
- If the spawn limit is hit, first close unneeded agents and retry; do not immediately fall back to main-thread work.
- Reuse or resume an existing agent only when the new task depends on its context; otherwise spawn a fresh bounded agent.
- Before reusing a prior Codex thread/worktree for a new issue, prove the prior task is complete or idle, the worktree is clean or intentionally continuing, and the new branch base is correct. Rename the thread when its active issue changes.
- Do not pile multiple unrelated tasks onto one unresolved agent thread.
- If an agent stalls, returns unusable output, or drifts from scope, close or redirect it with a tighter prompt instead of waiting indefinitely.
- If workers conflict, pause new delegation, evaluate the boundary, then re-delegate a focused integration or repair task.
- Keep waits sparse and purposeful. While agents run, the main thread should do non-overlapping coordination work, not duplicate their task.
- Record the exact reason for any downgrade to main-thread fallback: no subagent tool, spawn failure after cleanup, max-agent limit after cleanup, repeated unusable outputs, or capability mismatch.

Resource hygiene:

- On constrained local machines, treat under 8 GiB free disk as a warning and under 4 GiB free disk as an implementation pause gate unless the task is an explicit cleanup or emergency fix.
- Under 8 GiB free disk, do not spawn worker sidecars/subagents, browser sessions, dev servers, dependency installs, broad tests/builds, or new worktrees unless the coordinator explicitly authorizes that exception.
- Under 4 GiB free disk, do not spawn workers, sidecars, browser sessions, or verification lanes at all except for explicit cleanup or emergency incident work.
- Prefer one to three concurrent workers on resource-heavy repo work. Six is still only a maximum for genuinely parallel, lightweight lanes.
- Before cleanup, prove process/path ownership. Stop only processes started by the current effort or clearly obsolete worker/browser/dev-server processes; otherwise create a cleanup issue or ask.
- Do not delete worktrees, sessions, logs, caches, `node_modules`, generated files, or untracked artifacts unless the current task explicitly authorizes cleanup and provenance is clear.
- If a project-local or user-installed resource helper is available, use it for a read-only snapshot before large delegation waves. Do not hard-code private machine paths into worker briefs or reusable skill files.

Worktree and branch hygiene:

- A worker should not delete the worktree it is actively running from. Instead, report cleanup readiness in closeout: worktree path, branch, HEAD, clean status, push status, known active processes, and whether the branch has been merged if known.
- A coordinator or cleanup worker may remove a completed worktree only after proving all of the following: the source branch is contained in the pushed target branch, the worktree is clean including untracked files, there are no unpushed commits, no active thread/process is using the path, and no review/deploy/rollback/follow-up still needs the separate worktree.
- Use `git worktree remove <path>` for eligible source worktrees. Use `git worktree prune` only for stale metadata after removal or missing directories. Do not use raw recursive deletion for source worktrees as routine hygiene.
- Delete local or remote branches only when the target branch contains the work, project conventions allow deletion, and no open review, release, automation, issue, or worker reference still needs the branch.
- If cleanup would be valuable but any proof is missing, leave the worktree in place and create or update a cleanup task with the path, branch, and missing proof instead of guessing.

Reasoning selection:

- Low: mechanical search, docs parity, simple tests, focused fixtures, narrow verification.
- Medium: bounded implementation, ordinary code review, small refactors, non-critical planning.
- High: architecture, security, data integrity, auth, billing, production behavior, ambiguous cross-system work.
- Xhigh: irreversible decisions, destructive migrations, incident-critical judgment, or major cross-system architecture.

## Foundation Standard

Before implementation, have planner/research subagents identify the foundation the requested work needs to remain useful over years, not just for the immediate feature. The main thread evaluates and accepts the foundation decisions.

Think through likely future variants, adjacent workflows, ownership boundaries, data contracts, extension points, migrations, observability, permissions, and failure modes. Build the requested feature on stable seams that can support those future additions.

Do not implement speculative features. Do not add heavy abstractions, generic frameworks, or unused layers only because future work is possible. The target is a durable foundation for plausible expansion, with the current scope implemented simply and explicitly.

Default to simple now, scale-safe later. Keep MVP behavior narrow, but choose data shapes, permissions, contracts, ownership boundaries, and migration paths that can support the obvious next versions without forcing a known rewrite. If the simple implementation would create likely refactor debt, call that out before building.

Prefer reusable building blocks where adjacent or future features logically need the same behavior. Before adding a new UI pattern, data contract, API helper, permission check, workflow primitive, or integration adapter, look for an existing one to reuse or extend. Standardize repeated patterns at the lowest sensible layer, but avoid broad generic abstractions for one-off behavior.

For frontend/UI work, prefer existing design-system primitives and tokenized styling over standalone elements. Use established tokens, CSS variables, theme values, or utility primitives for color, spacing, typography, radius, borders, shadows, states, and motion when the repo provides them. If a new primitive is justified, make it token-ready and reusable for adjacent cases; avoid hard-coded one-off visual variants unless the existing system cannot reasonably express the requirement. If no token system exists, match local conventions and note the follow-up instead of inventing a broad design system inside a small task.

Use a minimum-complexity check before adding code or dependencies:

1. Can this requirement be removed, deferred, or handled by existing behavior?
2. Does the standard library, platform, database, browser, framework, or already-installed dependency cover it?
3. Can an existing local component, helper, schema, event, permission primitive, or workflow primitive be reused or narrowly extended?
4. Can the current scope be solved with a smaller explicit implementation?
5. Only then add new code, files, abstractions, dependencies, or configuration.

If a deliberately simple implementation has a known ceiling, document the ceiling and the trigger to upgrade it. Prefer a short code comment only when that ceiling would not be obvious from the code itself; otherwise name it in the plan, review, or closeout. Do not let "later" become untracked debt.

Use this foundation decision path:

- Existing foundation fits: build on it and preserve its contracts.
- Existing foundation is close but incomplete: extend it narrowly at the right seam.
- No foundation exists and future variants are plausible: create the smallest durable foundation needed for the current feature and likely next variants.
- The change is isolated, tiny, or unlikely to repeat: do not create new foundation; make the direct change.
- Cross-repo contract change: name the producer and consumer before editing. If a producer starts emitting/storing new fields, verify the consumer accepts them or create/comment the consumer follow-up before closing the producer issue.
- Customer-visible shell/navigation change with unclear layout: do a design/scout pass before production implementation. The scout should settle placement, mobile behavior, priority relative to existing chrome, empty/action states, and implementation boundaries.

For tiny or low-risk changes, delegate a light foundation check or keep it to one or two coordinator-reviewed sentences. For substantial work, have subagents first name three to ten realistic adjacent features or future variants that could reuse the same foundation. Then have them draft three to six foundation decisions before editing. Each decision should say what is being made stable now, what immediate feature it supports, and what future expansion it keeps possible. Add new abstractions only when current duplication, contract instability, or known near-term variants justify them.

The adjacent-feature scan is a design calibration tool, not a scope expansion step. Workers must not implement those adjacent features unless the issue explicitly asks for them. If the scan does not reveal realistic reuse, keep the implementation direct.

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
- resource constraints, cleanup permissions, and expected process ownership
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

Include an over-engineering review lens for non-trivial code changes. This complements correctness/security review; it does not replace them. Ask what can be deleted, replaced with standard library or native platform behavior, handled by an existing dependency, collapsed from an abstraction with one implementation, or expressed with fewer files and simpler data flow. Reject simplification findings that would remove trust-boundary validation, data-loss protection, security, accessibility, required tests, explicit user requirements, or meaningful observability.

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

For deploy, tracking, analytics, ads, provider, and customer-visible site work, distinguish each state explicitly: configured, rendered, deployed, provider-received, and live verified. A branch can be correct and still not be launch-ready if deploy or provider receipt is unproven. Name the exact unproven state as the blocker instead of collapsing it into Done.

Before final response, check active subagents, dev servers/watchers, temp artifacts, verification commands, and unresolved risks. Close unneeded subagents and stop unneeded local processes. Report what changed, what verified it, accepted findings fixed, rejected findings with rationale, deferred findings with next step, remaining blocker/high/medium count, unresolved risks, and the next concrete step if one remains.

For Linear closeout, include:

- delivery state: local-ready | committed | pushed | deployed | live verified
- branch and commit hash when applicable
- push state and remote ref proof when applicable
- whether Linear should be `In Review`, `Done`, or blocked, with the reason
- downstream contract consumers or deploy owners still needing action
