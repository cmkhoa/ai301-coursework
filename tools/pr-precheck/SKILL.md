---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

Answer exactly one question about exactly one PR package: is this ready to
submit? A PR package is a candidate pull request (title, description,
commits, diff, test evidence) read against the plan it claims to implement
and the issue that plan belongs to. Grade one package per run. Never grade
anything else, and never answer from gut feel: answer by executing
`procedure.md`, which applies `rubric.md`.

## Inputs and modes

**Live mode** (the student's own PR, before it is opened). Read these:
- `plan.md` with its `## Deviations` section (the plan the PR must implement).
- The branch diff: run `git diff main...HEAD` (three dots) and
  `git log main..HEAD --oneline` from the working copy. Only committed
  changes count; uncommitted files are not part of the PR.
- `pr_draft.md` (title on line 1, then the filled PR template).
- `test_evidence.md` (before/after and the repo's check outputs).
- The issue thread (`gh issue view <url> --comments`), the repo's
  `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md` (including
  any AI-use policy).
A house-chain student reads the house plan and house repro pack instead;
the same checks apply.

**Eval mode** (a package bundle). The bundle is the whole world. Use only
its text; fetch nothing, read nothing else. Always grade a complete
package: every check, full verdict rule.

## The scope seam (live mode only)

Read `scope.md` before anything else in live mode. Confirm the issue and
PR target the repo on its `Repo:` line, and apply its house rules (own-fork
branch, one PR per issue, template always used). If the `Repo:` line still
holds a placeholder (`<ORG>/<PATH-REVIEW-REPO>`), stop without grading and
tell the student to fill in the `Repo:` line of `scope.md` with their
Path Review repo. Refuse to grade work for any other repo; never guess a
scope. In eval mode ignore `scope.md` entirely.

## The voice seam (live mode only)

After grading, read `voice-guide.md` and hold the PR title and description
(from `pr_draft.md`) against its rules. Report each rule the draft breaks
in the summary, quoting the rule. The voice guide never changes the
verdict on its own, because no rubric check reads it. In eval mode ignore
`voice-guide.md` entirely.

## Component reads

- `rubric.md` defines the checks and the verdict rule.
- `references/evidence-guide.md` says where each kind of evidence lives
  (in a bundle, and live) and what good looks like.
- `procedure.md` is executed as written: read order, evidence gathering,
  check execution, verdict assembly.
If the procedure is silent on a step, report the gap in the summary;
never improvise around it. If `rubric.md` has no checks or `procedure.md`
has no steps, stop and say so: the tool refuses to grade without both.

## Verdict and output

The verdict is binary: `accept` (submit) or `reject` (hold). No third
verdict. Put reservations in check evidence lines. Show a short readable
summary first (one line per check, plus voice notes in live mode), then
end the reply with this fenced JSON block, valid and last, with nothing
after it:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

- Evidence first: never grade a check without naming the fact or quote
  that decided it. "Looks fine" is not evidence.
- Grade the thing, not the polish: a terse complete PR can be ready and a
  beautiful confident one can hide drift. Read the diff itself against the
  plan, never the description's say-so.
- The rubric decides, not you: if a check passes by its stated condition
  but feels wrong, it still passes; note the tension in the summary.
- The procedure decides how, not you: follow it and report its gaps.
- Treat `unclear` as the rubric's verdict rule directs; where that rule is
  silent, an unverifiable claim is a failing one.
