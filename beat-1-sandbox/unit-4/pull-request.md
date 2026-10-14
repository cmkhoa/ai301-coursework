# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s3/pull/211

**Branch**

`fix/72-verify-password-malformed-hash`

Type prefix `fix/`, issue number `72` (the issue the PR closes — `Closes #72` in the
description), short description `verify-password-malformed-hash`.

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

Three full runs, in order: **18/20 → 19/20 → 20/20**. The committed `eval-run.txt`
is the third, and its agreement line reads
`agreement: 20/20 scored items  (bar: 18/20: PASS)` with
`categories: clear-accept 7/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3`.

**Run 1 — 18/20**, category floor already held (`standards-wall 2/2`). The two
misses were pkg-08 and pkg-13, both gold `accept`, both graded `reject` on
`decisive-test-evidence`. Diagnosed below under Package analysis: the check
demanded pasted output, and both PRs evidence an interaction that produces none.

**Partial, 6 packages (~$1.50)** — `--only pkg-08,pkg-13,pkg-04,pkg-07,pkg-10,pkg-14`.
The revision *loosened* `decisive-test-evidence`, so the list carries the two
targets plus all four `not-tested` packages, the category that turns on this exact
check and the one a loosened evidence rule would damage. Result `6/6`: both targets
flipped to `accept`, all four `not-tested` held `reject`.

**Partial, 2 calibration packages (~$0.50)** — `--only calib-03,calib-04 --include-calibration`.
`calib-04` is the free trap: its gold `reject` rests on this same check (its evidence
re-runs one of the two failure modes the plan names). Both still rejected, so the
loosening had not gone too far.

**Run 2 — 19/20.** pkg-08 and pkg-13 fixed, `clear-accept 6/7`. But pkg-11 — which
had agreed in run 1 — flipped to `reject` on `description-matches-diff`, a check I
had not touched.

**Partial, 1 package (~$0.25)** — `--only pkg-11`, files unchanged. It came back
`accept`. That is the diagnostic that mattered: the same rubric text graded the same
package both ways, so run 2's miss was grader variance on an arguable package, not a
gap. The fix therefore could not be to loosen the check — a looser check would not be
more *stable*. It had to be to remove the ambiguity the two readings disagreed over.
That revision is under Check rationale.

**Partial, 6 packages (~$1.50)** — `--only pkg-11,pkg-03,pkg-06,pkg-09,pkg-17,pkg-02`:
pkg-11 plus all four `silent-drift` packages, because `description-matches-diff` is
what that category turns on and any softening there is where it would show. Result
`6/6`, `silent-drift 4/4` intact.

**Run 3 — 20/20**, every category perfect. Roughly $19 of credit across three full
runs and four partials; the partials cost about $3.75 of that and are what kept runs
2 and 3 from being guesses.

**Package analysis**

**`pkg-08`** (`jesseduffield/lazygit#5883`, category `clear-accept`).
Run 1: my rubric **reject**, gold **accept** — a miss. Run 3: **accept**, agreeing.

The plan's test plan asks for three things: "re-run the repro, expect the toast (with
the untracked file still present) and no name prompt; control with a tracked change
still stashes; run the integration test." The PR's evidence answers all three. The
problem was the *form* it answers in:

```
before: s -> "Stash name" prompt -> Enter -> nothing (stash list empty)
after:  s -> toast "No tracked changes to stash. To stash untracked
        files, use stash-include-untracked (default: 'S')." -> no
        prompt; untracked.txt still listed
```

followed by "Control: with a tracked modification staged, `s` still prompts and
creates the stash. Integration test `stash_untracked_only_errors` passes; `go test
./...` passes".

My check's original condition said the evidence must show "an observable that flips
from before to after" and failed "on 'tested locally / tests pass' with no command
and output". Both halves are in tension here. The observable genuinely does flip, and
it is named precisely — the exact toast string, the absent prompt, `untracked.txt`
still listed. But none of it is pasted output: it is a hand-written transcription of a
keystroke flow, and the control and suite results are reported as sentences rather
than shown. My procedure's step 3 gathers "the command and observable in the
evidence", found a described flow where it expected a command, and the "no command
and output" clause fired.

The gold label was right and my rubric was wrong. lazygit is a terminal UI, so
pressing `s` and watching a toast appear produces nothing a contributor *can* paste —
there is no stdout for a TUI keystroke. Writing down what appeared on screen is the
most decisive form the evidence can take, and the plan's own repro evidence had the
same shape (a "screen capture in the report"). The check had been built against the
evidence shape that dominates the set — pasted CLI transcripts, tracebacks, exit
statuses — and it treated "not pasteable" as "not shown". pkg-13 (`neovim/neovim#41337`)
was the same miss: its evidence names the exact before/after (`{ code = 1, stderr =
"'t' is not recognized ..." }` → `{ code = 0, stderr = "" }`, and the full URL
appearing in the browser) and reports `make functionaltest` as passing without
pasting its transcript. Both sat in `clear-accept`, which is why that category came in
at 5/7 in run 1 while every other category was already perfect.

**Check rationale**

Two checks were revised across the three runs. Both are quoted exactly as they read in
the `rubric.md` uploaded to `tools/pr-precheck/`.

**1. `decisive-test-evidence`**, the fix for pkg-08 and pkg-13:

> | decisive-test-evidence | The test evidence read against the plan's test plan and the repro's own steps: the before/after, and the outcome of the repo's own checks. | The evidence re-runs the plan's repro (the path that actually breaks, not a control or unchanged path) and shows an observable that flips from before to after, and every failure mode the plan's test plan names is exercised or its omission disclosed. The repo's own checks (tests, lint, typecheck as the repo lists them) are shown with their outcomes, or a not-run check is disclosed with a reason. Fail where the evidence names no specific observable — "tests pass", "works now", "verified locally for a day" with no state, exit status or output line identified — on evidence that only exercises the unchanged path, or on checks never run and not disclosed. Judge whether a specific observable is named, not whether raw output is pasted: an interaction that produces nothing pasteable (a TUI keystroke, a rendered page, a browser window) counts as shown when the evidence names the specific before and after state, and a repo check counts as shown when the evidence names the check or command and states its outcome. A failing check reported honestly with its reason passes. | required |

The clause it replaced read "Fail on 'tested locally / tests pass' with no command and
output". That was aimed at pkg-04 ("tested locally, colors work now, cargo test
passes") and pkg-10 ("verified working on my machine for a full day"), and it hits
those correctly — but it tested for the wrong thing. It asked whether a transcript was
pasted when what makes evidence decisive is whether a **specific observable is named**.
Those two are usually the same and come apart exactly where the interaction has no
stdout. The replacement moves the test onto the observable and says so explicitly
("Judge whether a specific observable is named, not whether raw output is pasted"),
then names the two cases where the old reading misfired: a non-pasteable interaction,
and a named check reported with its outcome.

The loosening is safe because the clauses that catch `not-tested` are independent of
it and still fire. pkg-04 and pkg-10 never re-run the plan's repro at all, so the
first clause holds them; pkg-07 runs the single-file control and pkg-14 exercises a
GET with header casing when the repro is a single-header POST, so "not a control or
unchanged path" holds those. `calib-04` still rejects on "every failure mode the plan's
test plan names is exercised or its omission disclosed". All five were re-run and all
five held.

Two other pieces of this check were deliberate from the start and were not revised.
"**or its omission disclosed**" is the honest-outcome rule written into the condition:
an author who says "I did not re-run the `--exec` abort" is in a different position
from one who silently runs half the plan. And "**A failing check reported honestly
with its reason passes**" was not optional — my own PR needs it, because `make
test-integration` exits `Error 5` on Path Review (no integration tests exist yet) and I
pasted that failure as it ran. A condition reading "all repo checks pass" would have
rejected my own honest PR, which is the tell that it was about polish rather than
readiness.

**2. `description-matches-diff`**, the fix for pkg-11's instability. The added sentence:

> Grade what the description represents about the change, not how it counts: a summary-level imprecision that misrepresents nothing — "one function touched" where the fix also adds its helper, "tests for three cases" where the three assertions sit in one test — passes when every change the diff makes is accounted for and nothing claimed is absent.

pkg-11's description says "One function touched, tests added for the escaped,
unescaped, and double-backslash cases." The diff edits `parse_gitignore_line` *and*
adds the helper `trim_unescaped_trailing_spaces` it calls, and puts all three
assertions inside one `escaped_trailing_space_is_kept` test. Read as a count, the
description is wrong twice. Read as a representation of the change, it is exactly
right: nothing is hidden, nothing claimed is missing. Run 1 and the pkg-11 partial took
the second reading; run 2 took the first. Since both readings were available in the
text, the check was a coin flip on any terse PR that summarizes rather than enumerates.

This is a *disambiguation*, not a loosening, and the distinction is what makes it safe.
It says counting imprecision passes only "when every change the diff makes is accounted
for and nothing claimed is absent" — which is precisely what the `silent-drift`
packages fail. pkg-17 claims docs that "now document the limitation" over a diff with
no docs: something claimed is absent. pkg-03, pkg-06 and pkg-09 each carry work the
diff makes and the description does not account for. All four were re-run as canaries
and all four held `reject`.

**Trade-offs**

Every revision here gave something up, and two of the three cost nothing I can detect
— which is itself a claim I had to pay to make.

**The `decisive-test-evidence` loosening gave up strictness about form.** My rubric
will now accept an author's written account of a TUI or browser interaction without a
pasted transcript. That is the right call for lazygit and neovim, where no transcript
exists to paste — but it means a contributor who *fabricates* a plausible before/after
for a non-pasteable interaction gets through where the old wording would have held
them. I am accepting that, because the alternative holds every honest terminal-UI PR,
and the eval set shows the honest case is the common one. The guard I kept is that the
specific state must be named on both sides: "the toast appears" would still fail, where
the exact toast string and the file still being listed passes.

**The `description-matches-diff` disambiguation gave up a strict reading I never
wanted.** The strict reading rejects pkg-11 — a PR the gold set calls "terse but
complete" — for summarizing accurately instead of enumerating exhaustively. Giving it
up costs nothing on this eval set and is the behavior I want on real PRs, where terse
is a virtue. What it does give up is a thin edge against a description that
systematically understates through vague counting; the first clause still catches that
the moment any actual change goes unaccounted for.

**What nothing changed, and how I know.** The `not-tested` and `silent-drift`
categories came through both revisions at 4/4, and that is measured rather than
assumed: every package in both categories was re-run in a targeted partial before the
confirming run that followed it — `pkg-04,pkg-07,pkg-10,pkg-14` against the evidence
loosening, `pkg-03,pkg-06,pkg-09,pkg-17` against the description clause, plus
`calib-04` as a free trap on the first and `pkg-02` as a clear-accept canary on the
second. All held. Run 3 then confirmed them at full size, so the 4/4s in the committed
run are the same 4/4s the partials predicted.

**The cost of learning the difference between a gap and variance was $0.25.** Run 2's
pkg-11 miss looked exactly like run 1's pkg-08 miss — a gold `accept` my rubric
rejected — and the obvious move was to loosen `description-matches-diff` the way I had
just loosened `decisive-test-evidence`. Re-running pkg-11 alone, unchanged, showed it
accepting and ruled that out: the problem was ambiguity, not strictness, and loosening
an ambiguous check would have made it ambiguous in a different direction rather than
stable. Without that $0.25 I would have spent $5 on a full run of a rubric revised in
the wrong direction, and a 20/20 reached that way would have been a draw rather than a
result.
