# Standard Development Workflow - Starter Kit

This package is a Claude starter kit for adopting a strict Standard Development
Workflow (SDW) in a host repository. It's intended to be as most plug-and-play as possible.

# Getting Started

## 1. Copy files

Copy the package into Claude projects at:

```text
.claude/skills/standard-development-workflow/
```

Copy these files into that directory:

- `SKILL.md`
- `README.md`
- `agents-to-generate.md`

## 2. Generate the required Claude Agents

This workflow depends on generated Claude agents. Use Claude's agent generator to create the required
agents in the host project's `.claude/agents/` directory:

- `software-architect`
- `developer`
- `test-suite-architect`
- `code-reviewer`
- `documentation-specialist`
- `session-closer`

Use `agents-to-generate.md` for each agent's purpose, responsibilities, suggested generator prompt,
expected output, and workflow steps.

## 3. Test

Ask Claude to implement a simple feature: it should use the SDW, and you should see the succession of
agents and steps.

# Custom behavior and tools

By default, this skill will look for:

- `CLAUDE.md` file at the root of the project
- Project guidelines and documentations in a `/docs` folder
- Linear as a ticket manager

Adapt these files, paths and tools directly in the `SKILL.md` file to match your project's requirements.