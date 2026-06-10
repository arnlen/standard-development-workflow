![Standard Development Workflow banner](/docs/visuals/sdw-banner.png)

# Standard Development Workflow

Standard Development Workflow (SDW) is a Claude skill starter kit for running
repository-tracked work through a strict, agent-orchestrated engineering process.
It is designed for developers who want AI-assisted changes to follow the same
discipline expected from human contributors: context gathering, planning,
implementation, verification, review, documentation checks, and clean session
closing.

This project started as a personal Claude skill and is now published as an open
source starting point that other projects can adapt.

![Example screenshot](/docs/2025-11-28_example_screenshot.png)

## What SDW Provides

SDW defines a workflow for Claude to follow when editing a repository. The main
Claude session acts as the orchestrator, while specialized generated agents handle
architecture, implementation, verification, review, documentation review, and
session closing.

The workflow is intentionally strict. It is meant to reduce common failure modes
in AI-assisted development, including skipped context, vague plans, unverified
changes, shallow reviews, unrelated edits, and messy handoff at the end of a
session.

At a high level, SDW requires Claude to:

- inspect the repository state before editing;
- read the relevant project documentation and source files;
- produce a decision-complete architecture plan;
- implement with TDD or docs-first discipline;
- run change-aware quality checks and verification;
- complete two review loops with focused reviewer agents;
- verify documentation and examples against the final behavior;
- close the session with clean git hygiene and a clear final report.

## Included Files

This starter kit contains:

- `SKILL.md`: the Standard Development Workflow skill definition.
- `agents-to-generate.md`: starter prompts for the required Claude agents.
- `README.md`: this project overview and installation guide.

## Installation

Clone or download this repository, then copy the skill files into the host
project's Claude skills directory:

```bash
HOST_PROJECT=/path/to/host/project

mkdir -p "$HOST_PROJECT/.claude/skills/standard-development-workflow"
cp SKILL.md README.md agents-to-generate.md \
  "$HOST_PROJECT/.claude/skills/standard-development-workflow/"
```

The final host-project layout should include:

```text
.claude/
  skills/
    standard-development-workflow/
      SKILL.md
      README.md
      agents-to-generate.md
```

## Generate the Required Agents

SDW depends on project-specific Claude agents. Generate them in the host
project's `.claude/agents/` directory before enabling the workflow:

- `software-architect`
- `developer`
- `test-suite-architect`
- `code-reviewer`
- `documentation-specialist`
- `session-closer`

Use `agents-to-generate.md` as the source material for each generated agent. The
prompts in that file are intentionally written as starters: adapt them to the
host project's stack, docs, safety constraints, issue tracker, and review
standards before generation.

## How the Workflow Runs

Once installed and adapted, ask Claude to perform repository work as usual. For
example:

```text
Use the Standard Development Workflow to implement this feature...
```

The skill instructs Claude to run the work through these phases:

1. **Prepare**: inspect git state, rebase when appropriate, read current docs and
   sources, classify dirty paths, and identify affected features.
2. **Architecture and work plan**: use `software-architect` to define acceptance
   criteria, scope, risks, implementation steps, and verification.
3. **Implement via TDD**: use `developer` to make the smallest maintainable change.
4. **Refactor and local quality**: use `developer` again to improve the touched area
   and run relevant quality checks.
5. **Verification**: use `test-suite-architect` to select and run the right checks.
6. **Review loop one**: use focused `code-reviewer` passes to find blockers,
   regressions, deferred findings, and scope-expanding issues.
7. **Review loop two**: re-review fixes and affected behavior after the first loop.
8. **Documentation review**: use `documentation-specialist` to verify docs,
   examples, comments, and workflow references.
9. **Session closing**: use `session-closer` to verify final hygiene, commit or
   integrate when in scope, and summarize the session.

The detailed rules live in `SKILL.md`. Treat that file as the source of truth for
the workflow contract.

## Host Project Assumptions

The default skill expects a host project with:

- a `CLAUDE.md` file at the repository root;
- project documentation in a `docs/` directory;
- Linear as the canonical issue tracker for deferred findings;
- local fallback records in `plans/deferred-findings.md` only when Linear is
  unavailable after a real attempt.

These defaults are not universal. Before adopting SDW in another project, review
`SKILL.md` and update paths, commands, issue-tracker rules, git conventions, and
verification expectations so they match that project.

## Customization Guide

At minimum, adapt:

- repository setup and documentation paths;
- branch, rebase, merge, commit, and push conventions;
- required test, lint, type-check, build, migration, and documentation commands;
- agent prompts in `agents-to-generate.md`;
- issue-tracker behavior, especially deferred-finding persistence;
- security and privacy routing for sensitive review findings;
- project-specific stop conditions and approval requirements.

The goal is not to make every project follow the author's exact workflow. The
goal is to give each project a clear, enforceable workflow that Claude can follow
without guessing.

## Verifying Installation

After installation and agent generation, run a small, low-risk repository task
through Claude. A good first test is a docs-only change or a tiny behavior change
with an obvious test.

You should see Claude:

- read the relevant project context before editing;
- invoke or request the required agents;
- keep edits inside the approved scope;
- run the selected verification checks;
- report review outcomes, documentation status, and final git hygiene.

If Claude skips required phases or cannot access the generated agents, stop and
fix the installation before relying on SDW for larger changes.

## Limitations

SDW is intentionally opinionated and can feel heavy for very small changes. It is
best suited to repositories where reviewability, traceability, and disciplined
handoff matter more than raw speed.

The starter kit also assumes Claude agent tooling is available. If the required
agent runtime or authorization is missing, the workflow should stop and ask for
direction rather than silently bypassing the missing step.

## License

This project is released under the MIT License. See `LICENSE` for details.
