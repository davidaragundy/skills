---
name: build-feature
description: Writes code the Next.js playbook's way in a repository set up with setup-nextjs-playbook — placing every file in app/, a feature or shared/ by the repository's own code standards, applying its server code, import and naming rules, speaking the glossary's terms, and checking every Next.js API against the current documentation. Use when adding or changing a page, layout, component, hook, schema, query, server action or any other code in such a repository.
license: MIT
compatibility: Repositories set up with setup-nextjs-playbook — Next.js with the App Router, TypeScript and pnpm.
metadata:
  author: davidaragundy
  version: "1.0.0"
---

# Build a feature

Part of the Next.js playbook. A playbook repository holds its rules in
`docs/code-standards.md`, numbered `CS-1` onwards, its domain's words in
`CONTEXT.md`, and its decisions in `docs/adr/`. This page is the order to apply
them in while writing code, and the points where a decision belongs to the user.

Three ideas run through every step:

- **The repository's copy is the rulebook.** Read the rules from the repository
  every time, never from memory or from this page, and cite them by number. Its
  reviewers enforce its copy, so where the two disagree, the repository wins.
- **The domain decides where code lives.** A file goes where the concept it
  knows about lives. Routes compose; features hold the domain; `shared/` holds
  what carries none.
- **The current Next.js docs decide how.** Every Next.js API and convention is
  checked against the documentation for the version in `package.json`, not
  written from memory. See [references/sources.md](references/sources.md).

The change needs an issue and a branch before its first line of code: the
playbook's `ship-change` skill opens them, and takes the change to merge after
step 6. Read [references/gotchas.md](references/gotchas.md) before you start.

## 1. Read the rules

- `docs/code-standards.md`, in full.
- `CONTEXT.md`, or the layout `docs/agents/domain.md` records.
- Every ADR in `docs/adr/` touching the area, and
  `0001-features-may-import-features.md` always.
- The Next.js version in `package.json`, and the documentation for it.
- The code where the change lands: the routes under `src/app/`, the features
  under `src/features/`, and `src/shared/`. Follow the patterns already there.

If `docs/code-standards.md` or `CONTEXT.md` is missing, the repository was not
set up with the playbook: stop, and tell the user that `setup-nextjs-playbook`
sets it up.

**Done when** you can name the rules and ADRs that govern the change.

## 2. Speak the glossary

List the domain concepts the change touches, each by its `CONTEXT.md` term. A
word listed under a term's `_Avoid_` is replaced by the term, in file names,
identifiers and prose alike.

A concept the glossary does not have is a decision for the user: stop, and
propose the term and its definition in the glossary's format, saying whether it
splits an existing term. Write it into `CONTEXT.md` once they agree.

**Done when** every concept the change touches has a term the user has seen.

## 3. Place every file

For each file the change needs, answer these in order, and stop at the first
yes:

1. **Is it a route, or the assembly of one?** It goes in `src/app/`: routes,
   layouts, metadata, Next's special files, and components from features put
   together (CS-2). State, data fetching and business rules live in a feature
   and are called from here. A page that shows pieces of two features assembles
   both in the route; neither feature imports the other's UI for it (ADR-0001).
2. **Does it know a domain concept?** It goes in the feature that owns the
   concept (CS-3, CS-5), from the list in CS-7. A concept no listed feature owns
   means a new feature, and that is the user's decision: propose the name and
   the reasoning, and once they agree, add it to CS-7 and to the scopes in
   `CONTRIBUTING.md` in the same change.
3. **Otherwise it carries no domain.** It stays in the feature that uses it
   until a second consumer genuinely needs it; then it moves to `src/shared/`
   (CS-6). Never earlier.

Then, inside its owner:

- **Kind folder** from the list in CS-8. A concept no kind names is the user's
  decision; `lib/`, `services/` and `helpers/` are never the answer.
- **Flat.** Subdivision goes in the file name, not in a nested folder (CS-9).
- **File name** in kebab-case (CS-18), with one exported concept per file
  (CS-16). Private helpers stay inside the file.
- **Folders** appear with their first file, never upfront (CS-7).

A feature may import another (CS-4). Before adding such an import, check the
other feature does not already import this one — `grep -rn "@/features/<this>"
src/features/<other>` — because nothing but review catches a cycle between
features (ADR-0001). When there is one, let the route compose the two instead,
or tell the user.

**Done when** every file has a path, and you can cite the rule that put it
there.

## 4. Write the code

- **Server first.** Components are Server Components unless they need state,
  effects, event handlers or browser APIs; `"use client"` goes on the smallest
  component that does. Check the boundary rules in the current docs.
- **Reads live in `queries/`.** Every file there imports `server-only` (CS-10).
- **Writes live in `actions/`.** Every file there starts with `"use server"`
  (CS-11) and exports async functions only. An action is a public endpoint:
  validate its input with a schema from `schemas/`, check authentication and
  authorization inside it, and return only what the UI needs. An action may
  call a query; a query never writes (CS-12).
- **Logic leaves the component.** State, effects, data fetching and non-trivial
  computation move into a hook named after the component; a component without
  logic gets no hook (CS-17).
- **Imports** go through `@/`, stylesheets included, to the direct path — no
  relative imports and no barrel files (CS-13, CS-14). Leave their order to the
  linter's autofix (CS-15).
- **Names** follow CS-19, in the glossary's terms.
- **Next.js APIs.** Before using an API or file convention, read its page in the
  documentation for the installed version. Where the change touches Cache
  Components or Partial Prefetching, the official Next.js skills named in
  [references/sources.md](references/sources.md) cover them.
- **Dependencies** arrive the day something uses them, installed at their latest
  stable version, and confirmed against their own docs.

**Done when** every file follows the rules that placed it, and every Next.js API
it uses was checked against the documentation.

## 5. Verify

- Run the checks `CONTRIBUTING.md` lists, including the build when the change
  touches configuration or anything that renders at the root.
- Run the change. A green check proves it compiles, not that it works. With the
  official `next-dev-loop` skill installed, use it; otherwise start `pnpm dev`,
  open every affected route, exercise the change, and read the server and
  browser consoles. When `next-dev-loop` is missing and the repository meets its
  requirements, recommend it to the user.
- Review your own diff against the rules, file by file: its path, its imports,
  its names, and the server rules. `grep -rnE "(from|import) ['\"]\.\.?/" src`
  returns nothing — it catches stylesheet imports too — and no `index.ts`
  re-exports a folder.

**Done when** the checks pass, the change works in the running app, and the
review of your diff finds nothing.

## 6. Hand off

Ship the change through `ship-change`, or through the flow `CONTRIBUTING.md`
describes. The pull request names every decision the user made along the way —
a feature, a term, a kind — and the rule behind any placement a reviewer might
question.

When the change settled a decision that is hard to reverse, surprising without
context, and the result of a real trade-off, offer the user an ADR in
`docs/adr/`, numbered after the highest one there.

**Done when** the change is on its way through the flow, and every decision it
made is written down.

## Guardrails

Each is a hard rule. Where a rule names what to leave out, it names what to do
instead.

- **The repository's rules win over this skill.** Where `docs/code-standards.md`
  says otherwise, follow it, and tell the user this skill is out of date.
- **Code bends to the rules, never the rules to the code.** A rule that seems
  wrong goes to the user as a proposal; `docs/code-standards.md` changes only
  with their agreement, in its own change.
- **Decisions about the domain belong to the user.** A new feature, a new kind,
  a new glossary term and an ADR are each proposed with reasoning, and wait for
  a yes.
- **Nothing is speculative.** No `shared/` before a second consumer, no folder
  before its first file, no dependency before its first use, no hook that
  returns props unchanged.
- **The current Next.js documentation wins over memory.** When it cannot be
  reached, say so rather than writing an API from memory.
- **Your own work is verified, not assumed.** Run the change and read what it
  does.
