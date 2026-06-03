---
name: standard-development-workflow
description: >
  Orchestrate repository development work through the Standard Development Workflow. Use this
  skill for repository-tracked edits, implementation planning, tests, linting, reviews,
  documentation updates, workflow edits, and session closing.
---

# I. Standard Development Workflow (SDW): introduction

This skill defines the workflow only. It must not duplicate project facts that belong in
`README.md`, `CLAUDE.md`, `docs/`, `plans/`, source code, tests, or the issue tracker. At the start
of each session, the orchestrator and assigned agents read the relevant current sources and let
them define the project context.

# II. Non-Negotiables

## II.1. Orchestrator And Agents

- The main Claude process is the orchestrator.
- The orchestrator preserves context, gathers evidence, assigns work, checks outputs, resolves
  contradictions, maintains the contribution ledger, and decides when to stop for user input.
- Implementation, tests, refactors, verification, review, documentation review, and closing are done
  through mandatory agents.
- Required agents are selected from the capabilities available in the current runtime.
- If required agent tooling or authorization is unavailable, stop and ask instead of bypassing
  the workflow.

## II.2. Git Management

- Git shared-history operations are restricted to the first and the last steps.
- Paired `git stash` / `git stash pop` may be used only for SDW verification or session
   closing/integration when needed to protect local work or clear the worktree.
- Stash use must be scoped, reported, and leave no stash behind. It must not bypass review,
hide conflicts, or sweep unrelated changes into the session commit.

## II.3. Deferred Findings Management

- Linear is canonical for deferred findings. Use local fallback persistence only under the
  `Deferred Finding Persistence` rule after a real Linear attempt fails.
- The only approved local fallback path for deferred findings is `plans/deferred-findings.md`.
- Local finding files are repository-tracked docs, not out-of-band notes. Creating, updating,
  migrating, reversing, or deleting entries in `plans/deferred-findings.md` or
  `plans/wont-fix-findings.md` must follow SDW.

# III. Agents

## III.1. Prompt Contract

Every agent prompt must include:

- the user request and assigned SDW step;
- the current branch, worktree, and dirty-state summary;
- the approved file and feature scope;
- relevant docs, code, tests, and issue context already reviewed;
- acceptance criteria and verification expectations;
- linked Linear issues, if any;
- Linear availability, failed Linear-attempt evidence, and local fallback instructions when deferred
  findings may need persistence;
- the required contribution ledger entry for that agent.

Agents must report changed files, findings, commands or checks run, unresolved risks, and whether
their output should be accepted, rejected, or deferred. When they defer a finding, they must report
the Linear issue link or the exact local fallback block, including migration status.

## III.2. Resume / Context-Compaction Guard

After context compaction, resume, interruption, or any uncertainty about the active session, perform a
read-only continuity check before continuing workflow work. Confirm the current task, branch and
worktree, active SDW step, intended file scope, current repository status, and step-relative workflow
evidence. When resuming at active step N, confirm evidence for every prior required SDW step, plus
any already-completed work in the active step. When applicable, also confirm review-fix loop state,
pass number, actionable findings, deferred findings, and pending stop/ask decisions.

Current conversation context or repository evidence may confirm workflow state, branch and worktree,
file scope, repository status, modified files, and prior step artifacts. Repository evidence must not
be used to prove explicit user authorization to spawn agents. Agent authorization must be
confirmed from current conversation context or platform state; if it cannot be confirmed there, stop
and ask the user before proceeding.

If any continuity item cannot be confirmed from the allowed evidence source, stop and ask the user
before proceeding. Do not continue from memory alone, do not infer missing agent authorization,
and do not replace required agent work with local orchestrator work.

## III.3. Contribution Ledger

The orchestrator maintains a ledger for every agent and includes it in the final response.

| Step | Agent role | Scope | Contribution | Accepted output | Duplicate or low-value output | Deferred Linear issue | Score |
| --- | --- | --- | --- | --- | --- | --- | --- |

Use the `Deferred Linear issue` column for the created Linear issue link. If Linear was unavailable
after a real attempt and the finding was recorded under the fallback rule, use the
`plans/deferred-findings.md` heading and migration status instead. Write `None` only when no
deferred finding or local fallback record was created, updated, migrated, or removed.

Scores:

- `0`: no useful output.
- `1`: confirmation only.
- `2`: useful finding, validation, or cleanup.
- `3`: materially improved the result or prevented a real defect.

# IV. Deferred Findings Persistence

## IV.1. Deferred finding records

Every deferred finding must be persisted in Linear with migration-ready fields: type, affected
files, affected feature, issue/impact/reproduction, suggested fix, severity, priority, estimated
scope, raised during session/provenance, and verification context.

Linear is the canonical system for deferred findings. Local persistence is a strict temporary
fallback, not a normal backlog, and it is not a WONT-FIX record.

Use local fallback only when all of the following are true:

- a deferred finding is valid and should survive the session;
- a real attempt to create or update the Linear issue has failed because Linear is unavailable;
- the failure evidence is recorded in the local finding;
- the finding is persisted only in `plans/deferred-findings.md`;
- the local block includes migration-ready fields: type, affected files, affected feature,
  issue/impact/reproduction, suggested fix, severity, priority, estimated scope, raised during
  session/provenance, verification context, Linear failure evidence, and migration status.
- no other local file, note, TODO, or fallback tracker is created or updated for the deferred
  finding.

Local fallback blocks must be migrated to Linear once Linear is available. After migration, delete
the full migrated local block from `plans/deferred-findings.md` in the same change that records the
Linear destination.

Creating, updating, migrating, or deleting a `plans/deferred-findings.md` block is a
repository-tracked docs change. The change must go through SDW with mandatory agent
implementation, docs readback and verification, review and closing coverage, and final reporting.
This includes deleting migrated blocks after Linear migration and deleting resolved local blocks
that no longer need persistence.

## IV.2. WONT-FIX Finding Records

`plans/wont-fix-findings.md` records review findings that are intentionally not fixed after an
explicit decision. It is not for deferred actionable work, not a substitute for Linear, and not a way
to ignore blockers or introduced regressions. Blockers and introduced regressions must be fixed,
escalated for user direction, or handled under the review-loop rules before any WONT-FIX record is
accepted.

Creating, updating, reversing, migrating from, or deleting a WONT-FIX record is a repository-tracked
docs change. The change must follow SDW with mandatory agent implementation, docs readback and
verification, review and closing coverage, and final reporting. Closing reports must identify any
WONT-FIX records created, changed, reversed, migrated, or removed, or confirm that none changed.

# V. Workflow

## Step 1. Prepare

The orchestrator performs setup before any repository-tracked edit.

1. Check recent history, current branch, worktree, and dirty state.
2. Rebase on `main` before continuing:

   ```bash
   git rebase main
   ```

   If the rebase cannot run cleanly because of local changes, conflicts, missing `main`, or other
   repository state, stop and ask before editing.
3. Read `README.md` first.
4. Discover and read relevant current sources: `CLAUDE.md`, `docs/`, `plans/`,
   ADRs, setup guides, testing guides, feature specs, source code, tests, fixtures, generated
   sources, and linked Linear issues when present.
5. Discover available project commands from repository files instead of assuming tooling.
6. Identify affected files and affected features. Later reviews must cover both.
7. Classify existing dirty paths as in scope, user-owned/unrelated, or blockers.
8. For bugs, reproduce the issue or identify the smallest reliable failure signal before assigning a
   fix.
9. For features, confirm expected end-to-end behavior, acceptance criteria, and user-visible
   semantics.
10. Identify whether deferred findings can be persisted in Linear. If the Linear destination is
   unknown, inspect available issue context or ask before review findings need deferral. If Linear
   is unavailable after a real attempt, prepare to use the `Deferred Finding Persistence` fallback
   and preserve the failure evidence.

## Step 2. Architecture And Work Plan

Agent to use: `software-architect`

A mandatory architecture agent produces a decision-complete plan before edits.

The plan must include:

- behavioral goal and acceptance criteria;
- affected feature map;
- intended file scope;
- relevant invariants from current docs, code, tests, and issue context;
- data and control flow;
- migration, rollout, concurrency, security, privacy, or operational risks when relevant;
- test and verification strategy;
- ordered implementation steps;
- review lenses required for the task.

The orchestrator checks the plan and records the pre-edit checkpoint: branch, worktree, status,
successful initial rebase, docs/code/tests reviewed, approved scope, expected files, verification
tier, linked issues, and known user-owned changes. Stop before editing if any checkpoint item is
unclear.

## Step 3. Implement Via TDD

Agent to use: `developer`

A mandatory implementation agent makes the smallest maintainable change that satisfies Step 2.

The implementation agent:

- follows current project docs and local patterns;
- writes or updates tests first when practical, especially for behavior, security, data, jobs, and
  regressions;
- avoids unrelated refactors;
- keeps edits inside the approved scope;
- returns changed files, rationale, test impact, and unresolved risks.

The orchestrator reviews the result for scope drift before continuing.

## Step 4. Refactor And Local Quality

Agent to use: `developer`

A mandatory refactor and quality agent improves the touched area and affected feature code within
scope.

The agent checks naming, duplication, complexity, dead code, stale comments, consistency with
local patterns, generated artifacts, and documentation around the changed behavior. It runs or
justifies relevant formatters, linters, type checks, static checks, generators, or docs checks
discovered from the repository.

Docs-only or comment-only changes require readback, link checks where practical, terminology checks,
and consistency checks instead of unrelated runtime checks.

## Step 5. Verification

Agent to use: `test-suite-architect`

A mandatory verification agent performs change-aware verification using repository guidance and
discovered tooling.

Verification must be broad enough for the risk:

- Docs/comment-only: read back changed sections and verify formatting, links, terminology, and
  consistency.
- Test-only: run affected and nearby tests; broaden when shared fixtures, helpers, or global setup
  changed.
- Production behavior: run targeted tests plus relevant integration, browser, job, data, security,
  or system checks.
- Config/build/dependency/generated changes: verify the affected toolchain and generated/consumer
  consistency.
- Cross-cutting, security, data model, background job, public API, or release-risk changes require
  broader verification.

Required targeted verification must be green before review. If verification cannot be completed, the
agent reports the exact blocker, reproduction details, and residual risk.

## Step 6. Review Loop One

Agent to use: `code-reviewer` (launched in parallel)

The orchestrator launches mandatory parallel review agents with distinct lenses appropriate to
the task.

Common lenses include:

- correctness and completeness;
- design and maintainability;
- database, migrations, jobs, concurrency, and idempotency;
- security, authorization, privacy, and data visibility;
- UI, accessibility, browser behavior, and client interactions;
- test quality and verification sufficiency;
- documentation and workflow consistency.

Reviews are not limited to the diff. Reviewers inspect the affected files and affected features in
context, including full modified files, direct callers and callees, related tests, relevant docs,
schemas, generated outputs, UI/API paths, and runtime behavior needed to judge quality.

Findings are classified before any fix:

- `Blocker`: the requested change is incorrect, unsafe, or not reviewable.
- `Introduced regression`: this session caused a behavioral, safety, or maintainability regression.
- `Scope-expanding`: the fix would change product semantics, permissions, data models, public APIs,
  broad architecture, workflow policy, or files outside the approved scope.
- `Deferred`: pre-existing, opportunistic, non-blocking, or outside the review budget.

Fix in-scope blockers and introduced regressions using the implementation agent, then rerun the
needed Step 4 and Step 5 checks. Stop and ask before scope-expanding fixes.

Every deferred finding is persisted as described in the [Deferred Findings Persistence](#iv-deferred-findings-persistence) section.

## Step 7. Review Loop Two

Agent to use: `code-reviewer` (launched in parallel)

Run a second mandatory review pass after Loop One fixes and verification.

The second pass may be narrower, but it must cover:

- all Loop One fixes;
- unresolved Loop One findings;
- affected feature behavior;
- newly touched files;
- any new risks introduced by fixes.

Fix remaining in-scope blockers and introduced regressions using the implementation agent, then
rerun necessary Step 4 and Step 5 checks. Do not start a third review loop unless the user explicitly
asks. Any unresolved blocker or introduced regression after Loop Two requires user direction.

Deferred findings from this pass are also persisted following the instructions of the [Deferred Findings Persistence](#iv-deferred-findings-persistence) section.

## Step 8. Documentation Review

Agent to use: `documentation-specialist`

A mandatory documentation agent checks whether README references, related docs, plans, public
examples, inline comments, generated docs, and workflow text still match the final behavior.

It updates only in-scope documentation or reports drift that needs user approval. For docs-only
tasks, this still runs as an independent review of clarity, accuracy, links, and consistency.

## Step 9. Session Closing

Agent to use: `session-closer`

A mandatory closing agent verifies the final state before the orchestrator responds.

The closing agent checks:

- final status and dirty-path classification;
- final diff;
- verification evidence and skipped-check rationale;
- review loop outcomes;
- Linear issue updates, including deferred findings and any `plans/deferred-findings.md` fallback
  records created, updated, migrated, removed, or still pending, with Linear failure evidence and
  migration status when applicable;
- WONT-FIX record updates in `plans/wont-fix-findings.md`, including the explicit decision and any
  reversals, migrations, or deletions;
- no unrelated user changes were swept in;
- contribution ledger completeness.

When SDW work is authorized and complete, session closing owns final integration without separate
confirmation for commit, rebase, or fast-forward merge. Before integration, the closing agent
must confirm that final verification and review evidence are green, or that the user has explicitly
accepted the remaining direction.

The final commit, rebase, and integration steps are:

- classify every dirty path as in-scope completed work, intentional workflow output, unrelated
  user-owned work, or stop-for-user-input;
- stage only in-scope completed work and intentional workflow output;
- create the session commit from the staged in-scope changes; each commit message and merge must
  follow our [Git Conventions](/docs/DEVELOPMENT.md#git-conventions).
- rebase the committed current `agent-{N}` branch onto current `main`;
- after rebase, detect whether migrations were introduced by the branch or by updated `main` during
  the integration window;
- when migrations are present after rebase, run the discovered project migration command in the
  agent branch/worktree, inspect schema or generated dirty state, classify generated files as
  in-scope intentional output or blockers, and rerun required verification and readback;
- after rebase, rerun required verification and readback when the base changed, affected files
  changed, or migration-generated state changed;
- merge the completed branch into `main` using fast-forward only;
- after fast-forward merge, inspect and classify dirty state in the `main` worktree before final
  commands;
- when migrations changed in the integrated range or `main` advanced with migrations relevant to
  the integration, run the discovered project migration command in the `main` worktree, then inspect
  and classify schema or generated changes; schema or generated dirty state not already represented
  by the integrated commit must be routed back through the branch SDW flow or treated as
  stop-for-user-input before final verification is trusted;
- after post-merge migration dirty state is resolved or confirmed absent, run the discovered full
  test suite on `main` before the final response after successful fast-forward integration.

If unrelated changes would be included in the commit, rebase conflicts occur, post-rebase
verification fails, fast-forward merge is not possible, unrelated or user-owned `main` dirty state
prevents trustworthy final migration or full-suite verification, stash pop conflicts occur, or
destructive recovery would be required, stop and ask or route the work back through the necessary
SDW fix and review steps.

Do not push unless the push is requested or already part of the explicit session scope. External
tracker status transitions remain explicit scope only.

The final response must include:

- mission summary: what was the purpose, what was implemented and how to test it;
- changed files;
- verification performed, including commands or readback checks and scoped exceptions;
- review loop outcome and unresolved risks;
- Linear issue updates, including deferred-finding links; local fallback records created, updated,
  migrated, removed, or still pending, with headings, Linear failure evidence, and migration status
  when applicable; or confirmation that no deferred findings or local fallback records changed;
- WONT-FIX record updates, including explicit decisions, reversals, migrations, deletions, or
  confirmation that none changed;
- the SDW contribution ledger.
