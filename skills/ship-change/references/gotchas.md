# Gotchas

Traps in the flow that none of the repository's documents mention, because each
one looked like it worked until it did not. Read the section for the step you
are on before you start it.

Each entry: **symptom** → cause → fix.

Each entry was true for the tool versions of its day. Apply a fix only once you
see its symptom, and if the tool's current documentation describes a different
fix, follow the documentation.

## Issues

**An issue opened from the CLI has no labels and no sections.** Issue forms
apply only in GitHub's web interface; `gh issue create` knows nothing about
them. → Write the form's sections into the body yourself, and pass each of the
form's labels with `--label`.

**`gh issue create` fails on a label.** A form can name a label, such as
`needs-triage`, that was never created in the repository. → Tell the user which
label is missing, and ask whether to create it with `gh label create` or to open
the issue without it.

## Commits

**commitlint warns `footer-leading-blank` on a valid message.** A body line
starting with `word:` is parsed as a footer token. → Reword so no body line
starts with a word followed by a colon. A real footer, such as
`BREAKING CHANGE:`, goes after a blank line at the end.

**Re-splitting commits sweeps every change back into the first one.**
`git reset --soft` keeps the index. → Use `git reset --mixed`, then stage each
commit with explicit paths.

**`git add a b` stages nothing.** If one path was already removed with `git rm`,
the whole command aborts. → Stage deletions with `git add -A -- <path>`.

## Pull requests

**A sloppy pull request title lands on `main`.** The squash commit is written by
GitHub from the title, so no local hook ever checks it. → Pipe the title through
`pnpm exec commitlint` before opening the pull request, and again after
changing it.

**`gh pr checks` reports no checks right after the pull request opens.** The
workflow has not started yet. → Wait a few seconds and run it again before
concluding CI is missing.

## After the merge

**Commits you pushed are missing from `main` after the merge.** A pull request
merged while you were still pushing captures the older head; the tail stays
stranded on the branch, and the next branch cut from `main` carries on without
any error. → After every merge, compare `gh pr view <n> --json headRefOid`
against your branch tip. If they differ, the stranded work needs its own issue,
branch and pull request.

**`git branch -d` refuses to delete a merged branch.** A squash merge writes a
new commit on `main`, so git never sees the branch's commits as merged. → Use
`git branch -D`, and only after the head check above matched.

**A message rewrite changed code.** → After any history rewrite,
`git diff <old-tip> <new-tip>` must be empty before you push, and push only with
`--force-with-lease`.
