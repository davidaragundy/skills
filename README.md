# Skills

Agent skills by [@davidaragundy](https://github.com/davidaragundy), written to the
[Agent Skills](https://agentskills.io) open standard, so they work in any
compatible agent — Claude Code, Codex, Cursor, Gemini CLI, OpenCode and others.

Together they make up the **Next.js playbook**: an opinionated way to build with
Next.js and its current best practices — a feature-based architecture, the rules
code is written by, and the flow every change takes from issue to merge.

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
| [`setup-nextjs-playbook`](skills/setup-nextjs-playbook/) | Sets up a Next.js repository to follow the playbook: the feature-based architecture, code standards, contribution flow, git hooks, CI and documentation. Run once. |
| [`ship-change`](skills/ship-change/)                     | Takes every later change through the playbook's flow: issue, branch, Conventional Commits, pull request, and the check after the merge.        |

## Layout

Each skill is a directory under `skills/` holding a `SKILL.md`, with optional
`references/` for material the agent loads on demand and `assets/` for files it
copies into the target repository.
