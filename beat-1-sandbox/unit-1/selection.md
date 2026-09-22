# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72

**Verdict output**

```
issue-select: live mode, scope = codepath/pathreview-ai301-fa26-s3
Rubric: 5 required checks (repo-alive, maintainer-present, unclaimed,
bounded-scope, policy-allows-ai), 2 preferred (recent-release,
maintainer-vouched). Verdict rule: accept iff every required check passes;
unclear on a required check counts as fail.

ACCEPTED (ranked by fit profile):

1. #72 — verify_password raises UnknownHashError on malformed stored hashes
   Fit: smallest bounded fix (est. 1–2 h) in pure Python with a named
   covering test and no infrastructure to stand up — matches a preference
   for backend bug fixes that are reproducible with unit tests alone.
   - repo-alive: pass — archived: no; newest default-branch commit
     2026-09-16, five days before this run.
   - maintainer-present: pass — all of the last 5 default-branch commits
     are authored by the human maintainer Aburke225, newest 2026-09-16.
   - unclaimed: pass — no assignee, no linked PRs open or mentioned; one
     student claim comment (sseid4) is ignored under the Path Review house
     rule in scope.md ("other students' claim comments do not block an
     issue").
   - bounded-scope: pass — one fail-closed change in core/security.py
     ("Verification against a malformed hash should fail closed (return
     False), not raise"), with the covering test named and an xfail marker
     to remove; estimated effort stated as 1–2 hours.
   - policy-allows-ai: pass — no CONTRIBUTING.md and no stated AI policy
     in the repo root; silence passes.
   - recent-release: unclear — the repo has no releases; commits carry
     liveness (preferred check, does not affect the verdict).
   - maintainer-vouched: pass — labels include "good first issue"; opened
     by maintainer Aburke225.

2. #68 — Keyword search raises ZeroDivisionError when the index is empty
   Fit: also pure-Python and unit-testable, but larger (est. 2–4 h) and
   inside the third-party BM25 wrapper, so slightly more code to read
   before the first change.
   - repo-alive: pass (same repo facts as above)
   - maintainer-present: pass (same repo facts as above)
   - unclaimed: pass — no assignee, no linked PRs; one student claim
     comment (acordero4852) ignored under the house rule.
   - bounded-scope: pass — one guard in KeywordSearcher.index() for the
     empty corpus, mirroring how search() already handles it; covering
     test named.
   - policy-allows-ai: pass; recent-release: unclear (preferred);
     maintainer-vouched: pass ("good first issue", maintainer-filed).

3. #62 — Health check references settings.redis_host, which does not exist
   Fit: bounded one-field fix, but reproducing it as written means running
   Redis and calling GET /health, which is the kind of infrastructure
   setup the fit profile prefers to avoid; ranked last of the three.
   - repo-alive: pass; maintainer-present: pass (same repo facts)
   - unclaimed: pass — no assignee, no linked PRs; one student claim
     comment (skonda29) ignored under the house rule.
   - bounded-scope: pass — swap the probe in api/routes/health.py to the
     existing settings.redis_url; reproduction steps stated in the body.
   - policy-allows-ai: pass; recent-release: unclear (preferred);
     maintainer-vouched: pass ("good first issue", maintainer-filed).

REJECTED: none of the graded candidates were rejected.
```

```json
[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72",
    "checks": [
      {"name": "repo-alive", "grade": "pass", "evidence": "archived: no; newest default-branch commit 2026-09-16"},
      {"name": "maintainer-present", "grade": "pass", "evidence": "last 5 default-branch commits authored by human maintainer Aburke225, newest 2026-09-16"},
      {"name": "unclaimed", "grade": "pass", "evidence": "no assignee, no linked PRs; sseid4's claim comment ignored under Path Review house rule"},
      {"name": "bounded-scope", "grade": "pass", "evidence": "single fail-closed fix in core/security.py with named covering test; estimated effort 1-2 hours"},
      {"name": "policy-allows-ai", "grade": "pass", "evidence": "no CONTRIBUTING.md in the repo; no stated policy"},
      {"name": "recent-release", "grade": "unclear", "evidence": "repo has no releases; commits carry liveness"},
      {"name": "maintainer-vouched", "grade": "pass", "evidence": "good first issue label; opened by maintainer Aburke225"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/68",
    "checks": [
      {"name": "repo-alive", "grade": "pass", "evidence": "archived: no; newest default-branch commit 2026-09-16"},
      {"name": "maintainer-present", "grade": "pass", "evidence": "last 5 default-branch commits authored by human maintainer Aburke225, newest 2026-09-16"},
      {"name": "unclaimed", "grade": "pass", "evidence": "no assignee, no linked PRs; acordero4852's claim comment ignored under Path Review house rule"},
      {"name": "bounded-scope", "grade": "pass", "evidence": "one guard in KeywordSearcher.index() for the empty corpus, mirroring search(); covering test named"},
      {"name": "policy-allows-ai", "grade": "pass", "evidence": "no CONTRIBUTING.md in the repo; no stated policy"},
      {"name": "recent-release", "grade": "unclear", "evidence": "repo has no releases; commits carry liveness"},
      {"name": "maintainer-vouched", "grade": "pass", "evidence": "good first issue label; opened by maintainer Aburke225"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/62",
    "checks": [
      {"name": "repo-alive", "grade": "pass", "evidence": "archived: no; newest default-branch commit 2026-09-16"},
      {"name": "maintainer-present", "grade": "pass", "evidence": "last 5 default-branch commits authored by human maintainer Aburke225, newest 2026-09-16"},
      {"name": "unclaimed", "grade": "pass", "evidence": "no assignee, no linked PRs; skonda29's claim comment ignored under Path Review house rule"},
      {"name": "bounded-scope", "grade": "pass", "evidence": "swap health probe to existing settings.redis_url; reproduction steps stated in the body"},
      {"name": "policy-allows-ai", "grade": "pass", "evidence": "no CONTRIBUTING.md in the repo; no stated policy"},
      {"name": "recent-release", "grade": "unclear", "evidence": "repo has no releases; commits carry liveness"},
      {"name": "maintainer-vouched", "grade": "pass", "evidence": "good first issue label; opened by maintainer Aburke225"}
    ],
    "verdict": "accept"
  }
]
```

---

## Eval iterations

**Run history**

Three runs, in order:

1. Smoke run (`--limit 3`, partial, does not count as a submitted run):
   `agreement: 2/3 scored items` — issue-01 disagreed
   (`issue-01  accept  reject   NO     failed: bounded-scope`).
2. Targeted re-run after revising the `bounded-scope` check
   (`--only issue-01,issue-05,issue-10`, partial):
   `agreement: 3/3 scored items`.
3. Confirming full run, the one saved to `eval-run.txt`:
   `agreement: 20/20 scored items  (bar: 18/20: PASS)` with
   `categories: claimed 4/4  clear-accept 8/8  dead-repo 3/3  policy 1/1  scope 4/4`.

**Issue analysis**

`issue-01` (conda/conda#16475, category clear-accept). Gold label:
`accept` — the gold note reads "docs task with a stated home and scope;
active repo, unclaimed". My rubric's final decision: `accept`, but its
first draft rejected it on `bounded-scope`, and the reasoning behind both
results is instructive. The issue's body is a multi-section plan: "Add a
new task page", then "Update `manage-pkgs.rst`", "Update
`pip-interoperability.rst`", "Update `new-features.md`", and "Consider a
global `troubleshooting.rst` entry". My original check failed any issue
that was "an umbrella/tracking/mega issue listing sub-items meant to be
split up", and a five-heading task list pattern-matched that clause, so
the grader read the docs plan as an umbrella issue and rejected. The
distinction the first draft missed is who the sub-items are for: an
umbrella issue's checklist is independent work items meant to be picked
up separately, while issue-01's sections are steps of one deliverable —
one new docs page plus updates to the existing pages that must point at
it, all landing together in a single PR. After I rewrote the clause to
say that "a single task that touches several named files in service of
one deliverable ... is still ONE bounded change and passes", the rubric
graded it `accept`, agreeing with gold, while the true umbrella issues
(issue-05's codebase-wide typing effort and issue-10's self-described
megaissue) still rejected.

**Check rationale**

The check I want to defend is `unclaimed`, quoted from the `rubric.md`
uploaded to `tools/issue-select/` as it is currently written:

> No assignee is set, AND there is no open linked PR (formally linked or
> mentioned in the thread), AND there is no unanswered claim comment ("I'll
> take this", "working on this", "can I work on this") posted within 12
> months of the capture date. A claim older than 12 months with no linked
> PR is stale and does not block; a claim explicitly released or re-opened
> to takers by a maintainer ("feel free to pick this up") does not block.
> Closed unmerged PRs do not block here (they are scope evidence).

The reasoning behind its current form: "is anyone on it?" has three
distinct signals that fail differently. An assignee or an open linked PR is
a hard, current claim, so either alone rejects. A claim comment, though,
decays: people announce intent and disappear, so an unanswered "I'll take
this" from years ago should not fence off an issue forever — hence the
12-month staleness window, and the explicit carve-out for a maintainer
re-opening the issue to takers. Closed unmerged PRs are deliberately
excluded here and counted under `bounded-scope` instead, because a pile of
abandoned attempts says the issue is harder than it looks, not that
someone currently owns it.

**Trade-offs**

What the 12-month staleness window gives up: a contributor who claimed an
issue eleven months ago and is still quietly working without a linked PR
would be treated as an active claim by my rubric and the issue passed
over, while a contributor who claimed thirteen months ago and pushed a
draft branch yesterday (without linking a PR or commenting again) would be
treated as stale and the issue accepted. I accept the second miss: a claim
that old with no visible artifact is indistinguishable from abandonment
using the evidence sources the rubric names, and requiring a visible
artifact (open PR or fresh comment) to keep a claim alive is the same
convention most maintainers apply when they re-invite takers on old
issues.

---

## Selection rationale

**Selection rationale**

1. **Fit to my interests and time.** I picked #72 (`verify_password`
   raises `UnknownHashError` instead of returning `False`) because it is a
   backend Python bug with a security flavor, exactly the kind of bounded
   fail-closed fix I want more practice with, and the smallest of my three
   accepted candidates (estimated 1–2 hours). It needs no infrastructure
   to reproduce — the covering test in `tests/unit/test_security.py`
   already exists under an `xfail` marker, so the whole loop is runnable
   with pytest alone, which fits the time I have between other coursework.

2. **What the verdict got right, and what I weighed that the rubric could
   not.** The verdict correctly saw the things a rubric can see: the repo
   is alive (commits five days old), the issue is maintainer-filed and
   labeled good-first-issue, nothing is assigned or linked, and the one
   claim comment doesn't block under the Path Review house rule. What the
   rubric could not weigh is which of three equally-accepted issues
   teaches me the most: #72 touches authentication code, where the
   fail-closed idiom is a transferable security habit, and its existing
   xfail test means I get to practice the remove-the-marker,
   make-it-green workflow that real projects use. Fit ranking captured
   that; the verdict alone could not.

3. **Anticipated difficulty in claiming it.** Another student (sseid4)
   has already posted a detailed claim comment on #72. The house rule says
   shared issues are normal in Path Review and course credit attaches to
   my own PR, so the claim doesn't block me — but it does mean my claim
   comment (written in Unit 2) should acknowledge the shared work rather
   than pretend to be first, and I should expect the maintainer's
   attention on the thread to be split. The actual claim mechanics look
   easy; the social part — claiming politely on an issue someone else is
   also working — is the difficulty I anticipate.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
