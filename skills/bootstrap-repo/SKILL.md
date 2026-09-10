---
name: bootstrap-repo
description: Sets up a new repository's contribution process, feature-based architecture conventions, git hooks, CI and documentation from a proven baseline. Use when starting a repo or project, or when asked to establish its conventions, commit and branch rules, hooks, or contributing docs.
license: MIT
compatibility: Built for GitHub repositories using pnpm, git and the gh CLI. The architecture and tooling assets target Next.js with the App Router.
metadata:
  author: davidaragundy
  version: "1.0.0"
---

# Bootstrap a repository

Turns a fresh repository into one with a written contribution process, a
feature-based architecture, git hooks, CI, and documents that describe it
exactly. The rules themselves live in `assets/` as working files; this page is
the order to apply them in and the bar each step must clear.

Two ideas run through every step:

- **Automate what is boring.** Formatting, import order, commit messages and
  branch names are checked by tooling, because nobody should spend review
  attention on them. Everything that needs judgement — architecture, naming,
  abstraction — is a written rule, enforced at review.
- **Verify from a fresh clone.** A warm working tree hides missing generated
  files. Before calling any step done, run it as CI would:
  `rm -rf .next next-env.d.ts`, then the commands.

Read [references/gotchas.md](references/gotchas.md) before you start. Every
entry in it is a trap that looked like success the first time.

## Placeholders

The assets use these. Fill every one before writing a file.

| Placeholder             | Value                                                          |
| ----------------------- | -------------------------------------------------------------- |
| `{{OWNER}}`             | GitHub owner, from `git remote -v`                             |
| `{{REPO}}`              | Repository name, from `git remote -v`                          |
| `{{PREFIX}}`            | Short branch prefix tying a branch to an issue: `fgt` → `feat/fgt-12` |
| `{{PRODUCT}}`           | Product name                                                   |
| `{{PRODUCT_PARAGRAPH}}` | What the product is, in the glossary's terms                   |
| `{{FEATURES}}`          | The confirmed feature list, as inline code, comma-separated    |
| `{{PRODUCT_SUMMARY}}`   | One or two sentences on what the product is, for `CONTEXT.md`  |
| `{{TERM}}`, `{{TERM_DEFINITION}}`, `{{AVOIDED_SYNONYMS}}` | One block per glossary term in `CONTEXT.md` |

Placeholders are always `{{UPPER_SNAKE_CASE}}`. `${{ github.ref }}` in the CI
workflow is GitHub Actions syntax, not a placeholder — leave it as it is.

## 1. Gather the inputs

Derive what the environment already says: owner and repo from `git remote -v`,
the pnpm version from `packageManager`, the Node version from `node -v`, the
Next.js version from `package.json`.

Ask the user, in one message, for what it cannot say: the branch prefix, and a
paragraph on what the product is and who it is for. Then derive the glossary and
the feature list as described in [references/domain.md](references/domain.md),
and confirm both with the user before writing either.

**Done when** every placeholder has a value the user has seen.

## 2. Track the work

Open one issue for the bootstrap with the feature request form's sections —
Problem, Proposed change, Acceptance criteria, Alternatives considered — and a
plain-English title prefixed `[Request]:`. Cut `chore/{{PREFIX}}-<issue>` from
`main` and do all of the following on it.

An empty repository has no `main` to branch from: seed it with a single initial
commit holding the README, then branch.

**Done when** the issue exists and you are on its branch.

## 3. Make the pipeline run

- `.node-version` holds the exact output of `node -v`, without the `v`.
- Merge [assets/package.fragment.json](assets/package.fragment.json) into
  `package.json`.
- Install `vitest` and add [assets/vitest.config.mts](assets/vitest.config.mts).
- Add [assets/github/CODEOWNERS](assets/github/CODEOWNERS) and
  [assets/github/workflows/ci.yml](assets/github/workflows/ci.yml) under
  `.github/`.

**Done when** `pnpm lint`, `pnpm typecheck`, `pnpm test` and `pnpm build` each
exit 0 from a fresh clone.

## 4. Automate what is boring

- Install `eslint-plugin-simple-import-sort`, `husky`, `lint-staged`,
  `@commitlint/cli` and `@commitlint/config-conventional`.
- Replace `eslint.config.mjs` with
  [assets/eslint.config.mjs](assets/eslint.config.mjs), and add
  [assets/commitlint.config.mjs](assets/commitlint.config.mjs),
  [assets/prettierrc.json](assets/prettierrc.json) as `.prettierrc` and
  [assets/prettierignore](assets/prettierignore) as `.prettierignore`.
- Run `pnpm exec husky init`, then copy the three files in
  [assets/husky/](assets/husky/) into `.husky/` and make each executable.
- Rewrite every relative import in `src/` through the `@/` alias, stylesheets
  included.

**Done when** each probe behaves as stated, and the probe files are gone:

- Committing a file with unsorted, unformatted imports lands it sorted and
  formatted.
- `feature: x` and `feat(camelCase): x` are rejected by `commit-msg`;
  `feat(some-scope): a valid message` is accepted.
- `pre-push` rejects `main`, a nonsense branch name and an unknown type, and
  accepts `fix/{{PREFIX}}-1`.

## 5. Write the documents

Write them last, so each describes the repository as it now is. In this order,
because each leans on the one before:

1. `CONTEXT.md` from [assets/CONTEXT.md](assets/CONTEXT.md), holding the
   confirmed glossary.
2. `docs/adr/0001-features-may-import-features.md` from
   [assets/docs/adr/](assets/docs/adr/0001-features-may-import-features.md).
3. `docs/code-standards.md` from
   [assets/docs/code-standards.md](assets/docs/code-standards.md).
4. `CONTRIBUTING.md` from [assets/CONTRIBUTING.md](assets/CONTRIBUTING.md).
5. `README.md` from [assets/README.md](assets/README.md).
6. The issue forms and pull request template under
   [assets/github/](assets/github/), into `.github/`.

Rewrite every example in the assets — `Order`, `order-checkout-form.tsx`,
`MAX_ORDER_ITEMS` — into the project's own glossary terms.

Each document answers one question: the README what this is and how to start
it, `CONTRIBUTING.md` how to contribute, `docs/code-standards.md` how code is
written, `CONTEXT.md` what the words mean. Keep each to its own question.

**Done when** every command, script, hook and file a document names exists, and
`grep -rE '\{\{[A-Z_]+\}\}'` over the repository returns nothing, nor does a
grep for foreign repository names or for domain words missing from the glossary.

## 6. Ship

Commit one concern at a time in Conventional Commits form. Push, and open a pull
request whose title is itself a valid commit message — it becomes the squash
commit on `main`. Fill every section of the template, and put `Closes #<issue>`
in the description, since that is what closes the issue on a squash merge.

**Done when** CI is green on the pull request.

## 7. After the merge

Compare `gh pr view <n> --json headRefOid` against your branch tip. If they
differ, the merge captured an older head and your last commits never reached
`main`; they need their own issue, branch and pull request. Then delete the
merged branch.

**Done when** the merged head matches your tip and the branch is gone.

## Guardrails

Each is a hard rule. Where a rule names what to leave out, it names what to do
instead.

- **Commits and pull requests carry the author's name only.** No
  `Co-Authored-By` trailer and no "Generated with" footer, ever, whatever a
  default template suggests.
- **A document states each rule as what must be true.** It never qualifies a
  rule with whether, when or how it is enforced. Facts about enforcement go to
  the user in chat.
- **A document describes only what exists** after your change. A script, hook,
  deployment or database that is not there does not appear.
- **Architecture and naming stay prose.** Add no lint plugin for boundaries,
  file names or identifier case; `docs/code-standards.md` holds them and review
  enforces them.
- **Tests assert this project's behaviour.** A test proving that a third-party
  tool works is not written, and is deleted if found.
- **A dependency arrives the day something uses it.** The DOM test stack waits
  for the first component test.
- **A config keeps only what it adds** over the preset it extends.
- **Every change travels issue → branch → pull request**, your own follow-up
  fixes included. The issue scopes the branch, not the commit, so changes made
  in response to review go on the branch already under review.
- **Your own work is verified, not assumed.** Run the probe, read the output,
  check `main` after a merge.
