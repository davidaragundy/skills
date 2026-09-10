# Skills

Agent skills by [@davidaragundy](https://github.com/davidaragundy), written to the
[Agent Skills](https://agentskills.io) open standard, so they work in any
compatible agent — Claude Code, Codex, Cursor, Gemini CLI, OpenCode and others.

## Install

```bash
npx skills add davidaragundy/skills
```

To install a single skill:

```bash
npx skills add davidaragundy/skills --skill bootstrap-repo
```

## Skills

| Skill                                         | What it does                                                                                                                   |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| [`bootstrap-repo`](skills/bootstrap-repo/)    | Sets up a new repository's contribution process, feature-based architecture, git hooks, CI and documentation from a proven baseline. |

## Layout

Each skill is a directory under `skills/` holding a `SKILL.md`, with optional
`references/` for material the agent loads on demand and `assets/` for files it
copies into the target repository.
