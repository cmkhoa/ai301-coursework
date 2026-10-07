# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`.

---

## Posted upstream

**GitHub username**

cmkhoa

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72#issuecomment-6029915163

Plan for #72, built on my reproduction above (https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72#issuecomment-5883026020). Several classmates have posted plans here; per the course house rules this is my own, from my own run.

**Diagnosis.** `verify_password()` (`core/security.py:37`) passes the stored value straight to `pwd_context.verify()` with no error handling, so a stored hash passlib cannot use escapes as an exception instead of returning `False`. My repro's controls returned `True` for the matching password and `False` for a wrong one, so the happy path is intact and this is specific to the stored hash.

The part that changes the fix is that it is not one exception type. From my repro's step 3 output (excerpt — the two control lines and the sha256_crypt case are in the full report above, unchanged):

```
the test's value: 'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError: hash could not be identified
empty string: '' -> raised passlib.exc.UnknownHashError: hash could not be identified
truncated $2b$ prefix: '$2b$12$short' -> raised builtins.ValueError: salt too small (bcrypt requires exactly 22 chars)
```

A guard written as `except UnknownHashError` — which is what the issue's wording suggests — leaves that third shape raising. `UnknownHashError` is a subclass of `ValueError`, so one `except ValueError` covers both.

**Scope.** One bounded change: the exception handling around that single `verify()` call, the `strict` xfail marker on `test_verify_with_wrong_hash_format` (removing it is part of the fix per `docs/CONTRIBUTING.md`), and regression tests. Not in scope: the login caller in `api/routes/auth.py`, the rest of `core/security.py`, and the passlib/bcrypt version-probe noise I flagged in my repro — that last one is real, but it is a dependency bump, not this bug.

**Approach.** Wrap the `verify()` call: re-raise `PasswordValueError` (passlib raises `PasswordSizeError` for an over-long *plaintext*, which is a complaint about the caller's input, not about the stored hash), and return `False` on any other `ValueError`. That one I ran separately:

```
long plain -> PasswordSizeError password exceeds maximum allowed size
```

Then drop the xfail marker and add a parametrized regression test over the four malformed stored-hash shapes from my repro, including the truncated `$2b$` one.

That re-raise is the one judgement call in here, and I would rather be told than guess: if the project would prefer the blunter `except ValueError: return False`, with an over-long password also failing closed, that is a one-line simplification and I will take it.

**Test plan.** My repro steps re-run against the change, with before/after output in the PR: the H-05 test goes from `1 xfailed` to `1 passed` with the marker gone; the same test under `--runxfail` goes from the `UnknownHashError` traceback at `core/security.py:37` to a pass; my four-value probe goes from three `UnknownHashError`s plus one `ValueError: salt too small` to `False` four times, with both controls unchanged; then `make check && make test-unit`.

**What I have not done.** I have not traced every exception `verify()` can raise, only the four shapes I ran — a non-string stored value still raises `TypeError` on my setup, and I am not guarding types since the column is typed `str`. I have only run this on macOS 26.6.2 / Python 3.11.16 / passlib 1.7.4 / bcrypt 4.3.0 — the Windows / Python 3.12 reports in this thread match on the exception types, but that is other people's evidence, not mine. No date from me; I will report back here when the branch is up.

I used Claude Code to help organize this plan; the reproduction it is built on is mine, I ran every step, and I understand the change I am proposing.

---

## Your branch

**Branch**

`fix/72-verify-password-malformed-hash`

**Evidence**

My unit-2 repro steps re-run against the built change (commit `5d28713`). Both
sides were captured on the same machine in the same venv: macOS 26.6.2 (arm64),
Python 3.11.16, passlib 1.7.4, bcrypt 4.3.0. The "before" was produced by
stashing the two changed files and re-running, and it matches what I posted in
unit 2.

**Step 1 — the covering test as shipped**

```bash
.venv/bin/pytest tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format -v --disable-warnings
```

Before:

```
======================== 1 xfailed, 2 warnings in 0.28s ========================
```

After:

```
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format PASSED [100%]

======================== 1 passed, 2 warnings in 0.47s =========================
```

**Step 2 — the same test with the marker ignored**

```bash
.venv/bin/pytest tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format --runxfail --tb=short --disable-warnings -q
```

Before:

```
.venv/lib/python3.11/site-packages/passlib/context.py:2343: in verify
    record = self._get_or_identify_record(hash, scheme, category)
.venv/lib/python3.11/site-packages/passlib/context.py:2031: in _get_or_identify_record
    return self._identify_record(hash, category)
.venv/lib/python3.11/site-packages/passlib/context.py:1132: in identify_record
    raise exc.UnknownHashError("hash could not be identified")
E   passlib.exc.UnknownHashError: hash could not be identified
=========================== short test summary info ============================
FAILED tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format
1 failed, 2 warnings in 0.20s
```

After:

```
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format PASSED [100%]

======================== 1 passed, 2 warnings in 0.17s =========================
```

**Step 3 — my repro's probe script (the deciding observable)**

```bash
.venv/bin/python probe.py
```

Before:

```
control, correct bcrypt hash -> True
control, wrong password vs same hash -> False
the test's value: 'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError: hash could not be identified
empty string: '' -> raised passlib.exc.UnknownHashError: hash could not be identified
truncated $2b$ prefix: '$2b$12$short' -> raised builtins.ValueError: salt too small (bcrypt requires exactly 22 chars)
well-formed sha256_crypt hash: '$5$rounds=535000$X8lpM1Zx0Qk9Yy3T$gqJ0nBu7oEJDcNsl0lVBrvWKKmNsBOSaUNzMvV.Y9G8' -> raised passlib.exc.UnknownHashError: hash could not be identified
```

After:

```
control, correct bcrypt hash -> True
control, wrong password vs same hash -> False
the test's value: 'not_a_valid_bcrypt_hash' -> returned False
empty string: '' -> returned False
truncated $2b$ prefix: '$2b$12$short' -> returned False
well-formed sha256_crypt hash: '$5$rounds=535000$X8lpM1Zx0Qk9Yy3T$gqJ0nBu7oEJDcNsl0lVBrvWKKmNsBOSaUNzMvV.Y9G8' -> returned False
```

All four malformed values now return `False` and both controls are unchanged,
which is the pair of facts the fix had to produce. `probe.py` is a scratch
script from the reproduction and is not in the commit.

**Step 4 — the whole security test file**

```bash
.venv/bin/pytest tests/unit/test_security.py -q --disable-warnings
```

Before:

```
.....................x...                                                [100%]
24 passed, 1 xfailed, 2 warnings in 5.95s
```

After (the `x` is gone and the four new parametrized cases are in):

```
.............................                                            [100%]
29 passed, 2 warnings in 5.98s
```

**Step 5 — the repo's own gate**

```bash
make check && make test-unit
```

After:

```
.venv/bin/ruff check .
All checks passed!
.venv/bin/black .
All done! ✨ 🍰 ✨
110 files left unchanged.
.venv/bin/mypy api/ core/ ingestion/ rag/ agent/ safety/
Success: no issues found in 76 source files
...
================= 380 passed, 52 xfailed, 2 warnings in 9.87s ==================
```

The 52 remaining xfails are the repo's other seeded bugs, untouched by this change.

## Eval iterations

**Run history**

Four runs, in order. Only the last is a full run.

1. Targeted partial, `--only pkg-20,pkg-04,pkg-09,pkg-14,pkg-03` — **4/5**. I spent
   $1 here before $4 because these five are where I expected my first draft to
   break: both `thread-convention` packages (the 2-package category that no
   volume elsewhere can buy back) plus the three `clear-accept` packages whose
   gold notes call them terse or arguable. Both thread-convention packages
   agreed. `pkg-14` did not: my rubric rejected a gold accept on `cause-grounded`.
2. Canary partial after the revision, `--include-calibration --only
   pkg-14,pkg-01,pkg-07,pkg-11,pkg-16,calib-03` — **5/5 scored, plus calib-03
   agreeing**. The revision loosened `cause-grounded`, so the canaries are all
   four `wrong-cause` packages (the category a loosened cause check could
   release into false accepts) and the calibration trap package, which costs
   nothing to re-read and is the one designed to catch a rubric that believes a
   confident diagnosis over the evidence. `pkg-14` flipped to accept, nothing
   else moved.
3. Spot-check partial, `--only pkg-06,pkg-12,pkg-10,pkg-17,pkg-02` — **5/5**. Two
   `scope-creep`, two `unbuildable`, one plain accept as a floor: the three
   categories I had not yet put a single package of in front of the rubric.
4. Full run, all 20 scored packages, `--save-run eval-run.txt` — **20/20, bar:
   PASS**, category floor met: `clear-accept 7/7  scope-creep 4/4
   thread-convention 2/2  unbuildable 3/3  wrong-cause 4/4`. This is the run in
   `eval-run.txt`, and 20/20 is its agreement line.

I made no changes after the full run, so the four `sha256` fingerprints in the
`eval-run.txt` header are the fingerprints of the files in `tools/plan-check/`.
I verified that rather than assuming it: `shasum -a 256` over the uploaded
`rubric.md`, `references/evidence-guide.md`, `procedure.md` and `SKILL.md`
reproduces `2a9ca9437df26e96`, `8654191539adc45f`, `67a85a23a65b5a3f` and
`4688d0d0f4cf6cb3`.

**Package analysis**

`pkg-14` (zellij/zellij#4306, category `clear-accept`). Gold label: **accept**.
My rubric, on run 1: **reject**, on `cause-grounded`. On runs 2 and 4, after the
revision: **accept**. It is the only package my rubric ever got wrong, and it is
the reason there was a second run at all.

The plan's diagnosis is that on reattach zellij issues OSC color queries and
wires the client's stdin to the session before the responses are consumed, so
the responses arrive as ordinary input and echo into the pane. The repro shows
raw `rgb:1c1c/1c1c/1c1c` strings on every reattach, a clean fresh attach, a clean
0.44.1, and a cache-clear control. What it does not show is the query traffic
itself — no debug trace of an OSC query being issued and not consumed. The plan
reasons from symptom to mechanism.

My first pass condition read: "The mechanism the plan names is one the repro
evidence **actually exhibits**, and nothing in that evidence rules it out... A
cause the evidence neither shows nor contradicts is `unclear`." With `unclear`
counting as fail, that rejected pkg-14 for not having traced what it inferred.

The check was wrong, and reading the gold notes for the whole `wrong-cause`
category is what showed me why. Every one of those four rejects is a
*contradiction*: pkg-01's control parses the same items fine without `-v`, ruling
out the tokenizer it blames; pkg-07's control prints the friendly error in the
same build, ruling out tree-shaking; pkg-11's control shows `collect` already
synthesizing null; pkg-16's step-4 evidence shows the zeros gone before the cast
it blames. Not one of them is a package that merely failed to demonstrate its
mechanism. I had encoded "ungrounded" as "undemonstrated", when the eval set
defines it as "contradicted" — and since almost every real plan infers its cause
from a symptom, the demonstration standard rejects good plans as a matter of
course. pkg-14 is just the package where that gap is visible.

**Check rationale**

The check as it reads in the uploaded `tools/plan-check/rubric.md`:

```
| cause-grounded | The plan's stated diagnosis (its cause sentence and the mechanism it names) read line by line against the repro-evidence block: its artifacts, its control runs, and its expected-vs-actual. The issue body and thread highlights are context, never the proof. | The mechanism the plan names would produce the behavior the artifacts show, and every control run in the repro evidence is consistent with it. A cause the plan reached by inference rather than by tracing passes, provided the artifacts it explains are the ones the package actually shows; the plan does not have to have demonstrated the mechanism directly. Fail if a control run already clears the component the plan blames (the symptom persists without it, or the same symptom appears in a configuration the named mechanism cannot reach), if the evidence locates the behavior at a stage before or after the one the plan blames, or if the plan adopts a cause from the issue or the thread that the package's own evidence contradicts. Grade `unclear` only when the artifacts are too thin to tell whether the named mechanism would produce them at all. | required |
```

Three decisions are load-bearing here.

The sentence I revised *in* is "A cause the plan reached by inference rather than
by tracing passes, provided the artifacts it explains are the ones the package
actually shows." That is the pkg-14 fix stated as a rule rather than as an
exception, so it generalizes: it makes the check ask whether the mechanism
*explains* the evidence, not whether the author proved it.

What I rejected in its favour is the demonstration standard described above
("the mechanism the evidence actually exhibits"). It is the more mechanical of
the two — a grader can check whether a trace is present far more reliably than
whether a mechanism is plausible — and I gave it up anyway, because its failure
mode is systematic rather than occasional. Catching every undemonstrated cause
means catching almost every plan.

The third decision is the evidence column's last line: "The issue body and thread
highlights are context, never the proof." Without it this check reads the thread
as corroboration, and that is exactly the trap calib-03 is built from — a
polished plan adopting the thread's confident key-binding diagnosis while the
package's own timing matrix points elsewhere. The clause is what lets the fail
condition name "a cause from the issue or the thread that the package's own
evidence contradicts" and have that mean something.

**Trade-offs**

The loosening in `cause-grounded` buys pkg-14 at the cost of a plan that invents
a mechanism nothing in the evidence contradicts simply because the evidence is
thin. My check will pass that plan. The `unclear` escape hatch is written
narrowly on purpose — "only when the artifacts are too thin to tell whether the
named mechanism would produce them at all" — which means a thin package
now leans toward accept where the strict version leaned toward reject. I take
that direction knowingly: on this eval set, inference is what good plans do and
contradiction is what bad ones do, so the check should be sensitive to
contradiction and tolerant of inference. The failure I have accepted is a
confident guess on a quiet package, and no package in the set has that shape, so
I cannot claim the set proves me right about it.

How I know nothing else moved: because loosening a check can release packages
that agreed before, run 2 was built as a canary run rather than a re-grade of
pkg-14 alone. All four `wrong-cause` packages went into `--only` — the entire
category the change could plausibly release — plus `calib-03` through
`--include-calibration`, free of score and purpose-built as the trap for exactly
this edit. All four held at reject, calib-03 held at reject, and pkg-14 flipped,
for $1.20 instead of a $4 full run finding it. Run 4 then confirmed it at
`wrong-cause 4/4`.

The second trade-off is one I did not revise and would not: `thread-direction` is
`required`, and it is the only check that can reject a plan whose engineering is
flawless. pkg-20 is the case — a genuinely excellent bounded plan, rejected
because ghostty's stated policy requires AI disclosure in comments and the
comment has none, which my `conventions-respected` catches; pkg-04 is the other
half, a coherent docs-only plan whose comment never engages the owner's isolated
culprit and patched test binary. Both of those are the plan being *right* and the
contribution still being wrong, and that is a real way for real contributions to
die. With only two packages in that category, a rubric with no comms checks
cannot buy the misses back anywhere else, which is why I put a check on each half
of it rather than one check on both.

One thing the run surfaced that I did not fix: grading my own plan live, the
skill flagged a gap in `procedure.md` step 3, which lists `CONTRIBUTING.md` and
`.github/` as policy locations but not `docs/`, where Path Review actually keeps
its contributing guide. It also does not say how to read a thread where every
commenter is `NONE`. Both are real gaps and both are one-line fixes. I left them
alone deliberately: `procedure.md` is one of the four files fingerprinted in the
`eval-run.txt` header, and editing it after the saved run would mean the skill I
uploaded is not the skill that produced the run I submitted. The fix belongs at
the top of the next revision, before the next full run, not after this one.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
