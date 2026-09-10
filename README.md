# Skills

Agent skills by [@davidaragundy](https://github.com/davidaragundy), written to the
[Agent Skills](https://agentskills.io) open standard, so they work in any
compatible agent — Claude Code, Codex, Cursor, Gemini CLI, OpenCode and others.

Together they make up the **Next.js playbook**: opinions about how a Next.js
repository is structured and the standards its code is written to, following
Next.js's current best practices. The playbook has no workflow of its own. The
work — from an idea through spec and tickets to implemented code — follows
[Matt Pocock's engineering skills](https://github.com/mattpocock/skills), which
read the playbook's rules as they go. The [workflow guide](docs/workflow.md)
shows where.

## Install

```bash
npx skills add davidaragundy/skills
```

To install a single skill:

```bash
npx skills add davidaragundy/skills --skill setup-nextjs-playbook
```

`setup-nextjs-playbook` runs after
[`setup-matt-pocock-skills`](https://github.com/mattpocock/skills/tree/main/skills/engineering/setup-matt-pocock-skills),
which records the repository's issue tracker, triage labels and domain doc
layout. Install it, and run `/setup-matt-pocock-skills` in the repository first:

```bash
npx skills add https://github.com/mattpocock/skills --skill setup-matt-pocock-skills
```

## Skills

| Skill                                                   | What it does                                                                                                                                   |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| [`setup-nextjs-playbook`](skills/setup-nextjs-playbook/) | Sets up a Next.js repository with the playbook's feature-based structure, code standards, contribution rules, git hooks, CI and documentation. Run once. |
| [`nextjs-playbook-rules`](skills/nextjs-playbook-rules/) | The rules for where code goes and how it is written, consulted by whatever is writing code — Matt's `implement` and `tdd` included.            |

## Layout

Each skill is a directory under `skills/` holding a `SKILL.md`, with optional
`references/` for material the agent loads on demand and `assets/` for files it
copies into the target repository.
