# Evidence guide: where evidence lives in a PR package

## Plan fidelity (harness category: silent-drift)

**Where it lives.** Eval bundle: `## Plan context` (the plan's Scope, Not in
scope, Files and any deviation or deferral lines) read against
`### Diff`, `### Commits` and `### Description` under `## Candidate PR`.
Live: `plan.md` (Scope, Files, `## Deviations`) in the working copy, read
against `git diff main...HEAD` and `git diff main...HEAD --stat`, and the
Summary and Changes sections of `pr_draft.md`.

**What good looks like.** Every changed file and hunk is named by the plan or
a deviation note, and every promised item (the fix, docs, tests) is in the
diff or disclosed as deferred. Drift runs both ways: extra (a new flag,
option, rename or rewrite the plan scopes out) or less (a promised docs
update missing). A description claiming "exactly as planned" or "docs
updated" is checked against the diff, never trusted. An honest deviation or
deferral recorded in the plan or description re-ties a mismatch.

## Test evidence (harness category: not-tested)

**Where it lives.** Eval bundle: `### Test evidence` read against the plan's
Test plan and the repro evidence's steps. Live: `test_evidence.md` (the
repro before on `main` and after on the branch, plus `make test-unit`,
`make test-integration`, `make lint`, `make typecheck` output), read
against the Test plan in `plan.md` and the Testing section of `pr_draft.md`.

**What good looks like.** The plan's own repro is re-run on the path that
actually breaks (not a control), with a named observable that flips
(exception to `False`, exit status, output line), every failure mode in the
plan's test plan covered or disclosed, and the repo's checks pasted with
outcomes. "tested locally, tests pass" with no command or output is not
evidence. A check that failed or "no tests ran" pasted honestly with its
reason counts.

## Diff quality (harness category: unreviewable)

**Where it lives.** Eval bundle: `### Diff` and `### Commits`. Live:
`git diff main...HEAD` and `git log main..HEAD --oneline`.

**What good looks like.** The fix is visible and nothing rides along. Debris
tells: `print`/`eprintln`/`console.log` leftovers, commented-out code or
earlier attempts, unused or `allow(dead_code)` functions, TODO notes on
imports, import reshuffles, re-indented or reprinted lines, whitespace-only
hunks, and unrelated edits. Commits named "wip", "fmt + cleanup" or "misc
cleanups while debugging" corroborate. A correct fix with debris still fails.

## Standards and comms (harness category: standards-wall)

**Where it lives.** Eval bundle: the `## Repo facts` block (PR-template asks,
contribution policy and AI policy) and `## Thread highlights`, read against
the candidate description and diff. Live: `.github/PULL_REQUEST_TEMPLATE.md`
and `docs/CONTRIBUTING.md` in the repo, the issue thread, and `pr_draft.md`.

**What good looks like.** Each template section has real content (Summary,
`Closes #<n>`, one Changes bullet per change, truthful Testing boxes,
Screenshots or "N/A" with a reason, Notes for Reviewers). Required artifacts
the repo states (changelog/whatsnew entry, tests, checklist) are in the
description or diff. AI use is disclosed in the author's own words whenever
the policy asks; treat every package here as AI-assisted. Maintainer
direction in the thread is followed or answered. Boilerplate or a visibly
ignored ask fails.
