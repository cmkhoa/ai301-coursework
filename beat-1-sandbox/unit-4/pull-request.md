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

One run: **18/20**, with every category matched
(`clear-accept 5/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3`).
That is the agreement line in the committed `eval-run.txt`:
`agreement: 18/20 scored items  (bar: 18/20: PASS)`.

There was no revise loop, and that is worth being straight about rather than dressing up:
the first full run cleared the bar and the category floor, so I spent no further credit.
The two disagreements (pkg-08 and pkg-13, both gold `accept`, both graded `reject` on
`decisive-test-evidence`) are a real and *known* weakness I did not buy out, not a mystery.
The reason I stopped is the fingerprint rule: `eval-run.txt` records
`SKILL.md sha256:a0218486bae02ddb`, `rubric.md sha256:f504c7193388039a`,
`procedure.md sha256:a6be912d2d3f5a99` and `evidence-guide.md sha256:f1aa2de8db95e781`, and
the files in `tools/pr-precheck/` must still hash to those, so any loosening of
`decisive-test-evidence` would have cost another full run to make submittable. I verified
the four uploaded files still match those fingerprints with `shasum -a 256` before
committing. The fix I would make first is written up under Trade-offs.

**Package analysis**

**`pkg-08`** (`jesseduffield/lazygit#5883`, category `clear-accept`).
My rubric: **reject**, on `decisive-test-evidence`. Gold label: **accept**. A miss.

The plan's test plan asks for three things: "re-run the repro, expect the toast (with the
untracked file still present) and no name prompt; control with a tracked change still
stashes; run the integration test." The PR's evidence answers all three. The problem is
the *form* it answers in:

```
before: s -> "Stash name" prompt -> Enter -> nothing (stash list empty)
after:  s -> toast "No tracked changes to stash. To stash untracked
        files, use stash-include-untracked (default: 'S')." -> no
        prompt; untracked.txt still listed
```

followed by "Control: with a tracked modification staged, `s` still prompts and creates the
stash. Integration test `stash_untracked_only_errors` passes; `go test ./...` passes".

My check's pass condition says the evidence must show "an observable that flips from before
to after" and fails "on 'tested locally / tests pass' with no command and output". Both
halves are in tension here. The observable genuinely does flip, and it is named precisely —
the exact toast string, the absent prompt, `untracked.txt` still listed. But none of it is
pasted output: it is a hand-written transcription of a keystroke flow, and the control and
the suite results are reported as sentences rather than shown. My procedure's step 3 gathers
"the command and observable in the evidence", found a described flow where it expected a
command, and the check's "no command and output" clause fired.

The gold label is right and I am wrong. lazygit is a terminal UI, so pressing `s` and
watching a toast appear produces nothing a contributor *can* paste — there is no stdout for
a TUI keystroke. Writing down what appeared on screen is the most decisive form the evidence
can take for that interaction, and the plan's own repro evidence had the same shape (a
"screen capture in the report"). My check was built against the shape of evidence that
dominates the eval set — pasted CLI transcripts, tracebacks, exit statuses — and it treats
"not pasteable" as "not shown". pkg-13 (gold `accept`, my `reject`, same check) is a
near-identical miss, which is why both land in `clear-accept` and why that category came in
at 5/7 while every other category was perfect.

**Check rationale**

The check, quoted exactly as it reads in the `rubric.md` uploaded to `tools/pr-precheck/`:

> | decisive-test-evidence | The test evidence read against the plan's test plan and the repro's own steps: the before/after, and the outcome of the repo's own checks. | The evidence re-runs the plan's repro (the path that actually breaks, not a control or unchanged path) and shows an observable that flips from before to after, and every failure mode the plan's test plan names is exercised or its omission disclosed. The repo's own checks (tests, lint, typecheck as the repo lists them) are shown with their outcomes, or a not-run check is disclosed with a reason. Fail on "tested locally / tests pass" with no command and output, on evidence that only exercises the unchanged path, or on checks never run and not disclosed. A failing check reported honestly with its reason passes. | required |

Three deliberate pieces, and one of them is the bug above.

The parenthetical **"not a control or unchanged path"** is the whole `not-tested` category in
four words. pkg-14 evidences a GET with header casing when the issue's repro is a
single-header POST; pkg-07 runs the single-file control when the plan's repro is the
two-file case. Both are evidence that proves something true about a path nobody claimed was
broken. Without that clause they look like diligent testing and pass. With it, `not-tested`
came in 4/4.

**"every failure mode the plan's test plan names is exercised or its omission disclosed"**
is the clause I argued with myself over, because it is what makes `calib-04` a reject —
a solid fix whose plan names two failure modes and whose evidence re-runs only one. The
assignment calls that one defensible both ways and says the master rejects it. I wrote the
clause to match the master, but the escape hatch "**or its omission disclosed**" is the part
I would defend on its own: an author who says "I did not re-run the `--exec` abort" is in a
different position from one who silently runs half the plan. That hatch is the honest-outcome
rule, fourth week running, written into the condition rather than bolted on in the verdict
rule.

**"A failing check reported honestly with its reason passes"** is the last sentence and it
was not optional — my own PR needs it. `make test-integration` exits `Error 5` on Path
Review because the repo has no integration tests yet, and I pasted that failure as it ran.
A check that read "all repo checks pass" would have rejected my own honest PR, which is the
tell that the condition was about polish rather than about readiness.

What I would revise, and did not, is the clause **"Fail on 'tested locally / tests pass' with
no command and output"**. It is aimed at pkg-04 ("tested locally, colors work now, cargo test
passes") and pkg-10 ("verified working on my machine for a full day"), where nothing
observable is named at all — and it hits those correctly. But it reaches too far, catching
pkg-08 and pkg-13, where a specific observable *is* named and simply cannot be pasted
because the interaction is a TUI. The fix is to make the clause turn on whether a specific
observable is named rather than on whether a command appears: something like *"Fail where no
specific observable is named — 'tests pass', 'works now', 'verified locally' with nothing
identified. A transcribed observable from an interaction that produces no pasteable output
(a TUI keystroke, a rendered frame) counts as shown when the specific state is named."*
That is a loosening, so it needs the canaries in Trade-offs below, and it needs a full run
to be submittable at all.

**Trade-offs**

Nothing changed after the first full run, and here is how I know what it cost.

The check above buys `not-tested` 4/4 and pays for it in `clear-accept`, 5/7. The two
packages it costs me are pkg-08 and pkg-13, and both fail the same way for the same reason:
a named observable that exists only as a transcription. That is the case I am accepting the
rubric will miss, and I can state it precisely rather than as a hunch — a correct,
well-evidenced PR for any terminal-UI or rendered-output change gets held by my tool for
having no stdout to paste. On a repo like lazygit that is not an edge case, it is most of
the surface.

I did not spend the revision, for two reasons. The bar was already met with the category
floor intact (`standards-wall 2/2`, the category the assignment warns is unbuyable at only
two packages), and the fingerprint rule means a rubric edit invalidates the saved run: the
header's four sha256 values have to match the files in `tools/pr-precheck/`, so the only
honest path from a revision is another full run at about $5. Trading a passing 18/20 for a
possible 20/20 at that price, after the deadline, is not a trade I would make.

If I do spend it, the `--only` list is already picked, and it is picked the way the eval
README says: the revision **loosens** `decisive-test-evidence`, so a package that agreed
before can flip. The targets are `pkg-08,pkg-13` (the two misses), plus a canary from every
category that loosening could release — `pkg-04,pkg-07,pkg-10,pkg-14` (all four `not-tested`
packages, since every one of them turns on this exact check, and `not-tested` is where a
loosened evidence check does its damage), plus `--include-calibration` for `calib-04`, whose
gold `reject` on this same check is the free trap: if my loosened wording flips calib-04 to
accept, the loosening went too far and I know it for $0. That is 6 scored packages plus a
calibration at roughly $0.25 each, about $1.75, before committing to the $5 confirming run.
