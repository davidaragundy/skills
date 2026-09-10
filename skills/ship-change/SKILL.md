---
name: ship-change
description: Takes a change through the Next.js playbook's contribution flow — issue, branch, Conventional Commits, pull request, and the check after the merge — reading the rules from the repository's own CONTRIBUTING.md, issue forms, pull request template and hooks. Use for any change to a repository set up with setup-nextjs-playbook, or when asked to open an issue, cut a branch, commit, open a pull request, respond to review, or clean up after a merge.
license: MIT
compatibility: GitHub repositories set up with setup-nextjs-playbook, using git and the gh CLI.
metadata:
  author: davidaragundy
  version: "1.0.0"
---

# Ship a change

Part of the Next.js playbook. Every change to a playbook repository travels the
same road: an issue that says what done looks like, a branch named after it,
commits in Conventional Commits form, a pull request whose title becomes the
squash commit on `main`, and a check after the merge that everything arrived.
`CONTRIBUTING.md` states the rules; this page is the order to apply them in and
the checks between them — the ones that get missed.

Two ideas run through every step:

- **The repository's copy is the rulebook.** Read the rules from the repository
  every time, never from memory or from this page. Its hooks, linter and
  reviewers enforce its copy, so where the two disagree, the repository wins.
- **Ask before anything others can see.** Opening an issue, pushing, opening a
  pull request and deleting a remote branch each wait for the user's go-ahead.
  Show the draft first. Everything local is yours to do.

Read [references/gotchas.md](references/gotchas.md) before you start. Every
entry in it is a trap that looked like success the first time.

## 1. Read the flow

Read each of these, and keep the values you will need:

| File | What to take from it |
| --- | --- |
| `CONTRIBUTING.md` | The types, the scopes, the branch pattern, the commit convention, the issue title rule, the PR flow, who reviews and merges, and the checks to run before a pull request |
| `.husky/pre-push` | The exact branch pattern, prefix included |
| `commitlint.config.mjs` | Every rule it adds to the preset it extends |
| `.github/ISSUE_TEMPLATE/*.yml` | Each form's title prefix, labels and sections, and which sections are required |
| `.github/PULL_REQUEST_TEMPLATE.md` | Its sections and checklist |
| `docs/agents/issue-tracker.md` | Where issues live |
| `docs/agents/triage-labels.md` | When present, the label for the `needs-triage` role |
| `CONTEXT.md` | The domain's words, for titles and descriptions |

If `CONTRIBUTING.md`, the issue forms or the hooks are missing, the repository
was not set up with the playbook: stop, and tell the user that
`setup-nextjs-playbook` sets it up. If the issue tracker is not GitHub, stop and
say so; this flow depends on GitHub Issues.

**Done when** you can state the branch pattern, the accepted types and each
form's sections from what you read.

## 2. Scope the change

A change belongs to exactly one issue.

- **Review follow-up on an open pull request:** no new issue and no new branch.
  Check out that pull request's branch and go to step 4.
- **An issue already covers it:** use it. Search before drafting —
  `gh issue list --state open --search "<words>"` — and read a candidate with
  `gh issue view <n> --comments`.
- **Otherwise, draft one:**
  - Pick the form: something broken is a bug report; everything else is a
    request.
  - The title is the form's prefix followed by the problem in plain English.
    Never a Conventional Commit: the type belongs to the eventual fix.
  - The body has the form's sections as `### <label>` headings, in the form's
    order, with every required section filled. Acceptance criteria are
    checkboxes, one verifiable outcome each.
  - The labels are the form's `labels:`, with `needs-triage` replaced by the
    mapped label when `docs/agents/triage-labels.md` exists.
  - Show the user the draft. On their go-ahead, open it with
    `gh issue create --title "…" --label "…" --body-file <file>`.

**Done when** the issue exists and its acceptance criteria say what done looks
like.

## 3. Cut the branch

- Start from a clean tree, or one whose uncommitted work belongs to this change.
- `git switch main && git pull --ff-only`, then
  `git switch -c <type>/<prefix>-<issue>`.
- The type is the one you expect the pull request's title to carry. Check the
  name against the pattern in `.husky/pre-push` before the first commit, not at
  the first push.

**Done when** you are on the branch, and it starts from the current
`origin/main`.

## 4. Commit

- One concern per commit. Stage explicit paths, and read `git diff --staged`
  before each commit.
- The subject follows the commit convention; the scope is the feature the change
  touches, or `repo`, `ci` or `agents`, as `CONTRIBUTING.md` says.
- The body says why, not what the diff already shows.
- The hooks run on every commit. When one fails, fix what it points at and
  commit again. Never pass `--no-verify`.

**Done when** every commit passed `commit-msg`, and `git log main..` reads as
one line per concern.

## 5. Verify

- Run the checks `CONTRIBUTING.md` lists before a pull request, including the
  ones it says CI does not run when your change touches what they cover. Read
  the output; a red check is fixed on the branch and the checks run again.
- Go through the issue's acceptance criteria one by one, and check each against
  the branch.

**Done when** every check exits 0, and every criterion is met or named as not
met.

## 6. Open the pull request

- **Title.** It becomes the squash commit on `main`, and no hook sees it. Make it
  a valid commit message for the whole change, then prove it:
  `printf '%s\n' "<title>" | pnpm exec commitlint`.
- **Body.** The template, with every section filled and `Closes #<issue>` in
  place of `Closes #` — the description is what closes the issue on a squash
  merge. "How to verify" lists steps you ran. A checklist box is ticked only when
  it is true; one that is not stays unticked, with the reason beside it.
- Show the user the title and body. On their go-ahead, `git push -u origin
  <branch>`, then `gh pr create --base main --title "…" --body-file <file>`.
- Watch CI with `gh pr checks <n> --watch`, and fix a red run on the branch.

**Done when** the pull request is open and CI is green on it.

## 7. Respond to review

Follow-ups go on the branch already under review, as new commits. When they
change what the pull request does, update its title and body, so the squash
commit still describes it. Rewrite pushed history only when the user asks, with
`--force-with-lease`, and when only messages changed, confirm
`git diff <old-tip> <new-tip>` is empty before pushing.

**Done when** every review comment is answered by a commit or a reply.

## 8. After the merge

Merging is the reviewer's call, as `CONTRIBUTING.md` describes. Do not merge
unless the user asks. Once it is merged:

1. `gh pr view <n> --json state,headRefOid` against `git rev-parse <branch>`.
   If they differ, the merge captured an older head and your last commits never
   reached `main`: they need their own issue, branch and pull request.
2. `git switch main && git pull --ff-only`, and check the squash commit and its
   files are there.
3. `gh issue view <issue> --json state`. An issue still open means the
   description lacked its closing keyword; tell the user.
4. Delete the branch: `git branch -D <branch>` once step 1 matched, the remote
   branch with `git push origin --delete <branch>` if it still exists, then
   `git fetch --prune`.

**Done when** the merged head matches your tip, the issue is closed, and the
branch is gone locally and on the remote.

## Guardrails

Each is a hard rule. Where a rule names what to leave out, it names what to do
instead.

- **Commits, issues and pull requests carry the author's name only.** No
  `Co-Authored-By` trailer and no "Generated with" footer, whatever a default
  template suggests.
- **The repository's rules win over this skill.** Where `CONTRIBUTING.md`, a
  hook or a template says otherwise, follow it, and tell the user this skill is
  out of date.
- **Nothing reaches `main` except through a pull request.** No direct push, no
  `--no-verify`, and no force push without `--force-with-lease`.
- **Every claim in an issue or pull request is true.** A ticked box was checked,
  a verification step was run, and anything not done is named.
- **Your own work is verified, not assumed.** Read the hook's output, CI's
  result, and `main` after the merge.
