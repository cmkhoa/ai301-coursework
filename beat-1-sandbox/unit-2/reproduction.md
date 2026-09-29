# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`.

---

## Your identity upstream

**GitHub username**

cmkhoa

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72#issuecomment-5882971266

I'd like to claim this one as my first Path Review contribution. Several classmates have commented here already; per the course house rules I am posting my own claim and will do my own setup, reproduction and write-up rather than adding to theirs.

What the issue describes, as I read the code: `verify_password()` in `core/security.py` passes the stored hash straight to passlib's `pwd_context.verify()`, so when the stored value is not a recognizable hash, passlib's `UnknownHashError` escapes instead of the function returning `False`. The covering test is `tests/unit/test_security.py::test_verify_with_wrong_hash_format`, which carries a strict `xfail` marker for manifest id H-05. That is a reading, not a result — I have not run anything yet.

What I plan to do next:

1. Fork the repo and set it up following `docs/SETUP.md`, recording my OS, Python, passlib and bcrypt versions and the exact commit I test against.
2. Run that test with `--runxfail` so the real exception is visible instead of being swallowed by the marker, and call `verify_password()` directly with a malformed stored hash, using a correctly hashed password as a control run.
3. Check which exception types actually reach the caller for a few different malformed shapes — an empty string, a truncated `$2b$` prefix, and a well-formed hash belonging to a different scheme — since a guard that catches only `UnknownHashError` may not cover all of them.

I will post the environment, the exact commands and the raw output here whether or not it reproduces. I am not promising a fix or a timeline before that report is up.

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72#issuecomment-5883026020

Reproduction report for #72. **Result: reproduced.** `verify_password()` raises instead of returning `False` for a malformed stored hash. One thing I did not expect, and it matters for the fix: not every malformed hash raises `UnknownHashError` — a truncated `$2b$` hash escapes as a plain `ValueError` instead (step 3 below).

**Environment**

- macOS 26.6.2 (build 25G83), Apple Silicon (arm64)
- Python 3.11.16 (Homebrew `python@3.11`, matching the 3.11 that `.github/workflows/ci.yml` pins), fresh `.venv`
- passlib 1.7.4, bcrypt 4.3.0, pytest 9.1.1, installed with `pip install -e ".[dev]"`
- Code: commit `2f4e82f`, current `main`, cloned into my fork with no local changes
- Only the Python side is needed for this bug: no Docker service, no database and no `.env` value is involved. I did not run `docker compose up` or `alembic upgrade head` for this reproduction.

**Steps**

```bash
git clone https://github.com/cmkhoa/pathreview-ai301-fa26-s3.git
cd pathreview-ai301-fa26-s3 && git checkout 2f4e82f
python3.11 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

# 1. As shipped: the strict xfail marker hides the failure
pytest tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format -v

# 2. The same test with the marker ignored, so the real exception is visible
pytest tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format -v --runxfail --tb=short
```

Step 1 output (tail):

```
======================== 1 xfailed, 2 warnings in 2.12s ========================
```

Step 2 output (session header and two unrelated deprecation warnings trimmed):

```
=================================== FAILURES ===================================
_______________ TestSecurity.test_verify_with_wrong_hash_format ________________
tests/unit/test_security.py:227: in test_verify_with_wrong_hash_format
    result = verify_password("password", wrong_hash)
core/security.py:37: in verify_password
    return bool(pwd_context.verify(plain_password, hashed_password))
.venv/lib/python3.11/site-packages/passlib/context.py:2343: in verify
    record = self._get_or_identify_record(hash, scheme, category)
.venv/lib/python3.11/site-packages/passlib/context.py:2031: in _get_or_identify_record
    return self._identify_record(hash, category)
.venv/lib/python3.11/site-packages/passlib/context.py:1132: in identify_record
    raise exc.UnknownHashError("hash could not be identified")
E   passlib.exc.UnknownHashError: hash could not be identified
=========================== short test summary info ============================
FAILED tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format
======================== 1 failed, 2 warnings in 0.20s =========================
```

**Step 3: which exception actually reaches the caller**

The same script also covers the control run. Saved as `probe.py` at the repo root and run with `python probe.py`:

```python
from core.security import hash_password, verify_password

good = hash_password("password")
print("control, correct bcrypt hash ->", repr(verify_password("password", good)))
print("control, wrong password vs same hash ->", repr(verify_password("nope", good)))

cases = [
    ("the test's value", "not_a_valid_bcrypt_hash"),
    ("empty string", ""),
    ("truncated $2b$ prefix", "$2b$12$short"),
    ("well-formed sha256_crypt hash",
     "$5$rounds=535000$X8lpM1Zx0Qk9Yy3T$gqJ0nBu7oEJDcNsl0lVBrvWKKmNsBOSaUNzMvV.Y9G8"),
]
for label, value in cases:
    try:
        print(f"{label}: {value!r} -> returned {verify_password('password', value)!r}")
    except Exception as e:
        print(f"{label}: {value!r} -> raised {type(e).__module__}.{type(e).__name__}: {e}")
```

Output (one unrelated passlib/bcrypt version-probe traceback trimmed from the top, see the note below):

```
control, correct bcrypt hash -> True
control, wrong password vs same hash -> False
the test's value: 'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError: hash could not be identified
empty string: '' -> raised passlib.exc.UnknownHashError: hash could not be identified
truncated $2b$ prefix: '$2b$12$short' -> raised builtins.ValueError: salt too small (bcrypt requires exactly 22 chars)
well-formed sha256_crypt hash: '$5$rounds=535000$...' -> raised passlib.exc.UnknownHashError: hash could not be identified
```

**Expected:** `verify_password()` returns `False` for every one of those malformed stored hashes, and `True` / `False` for the two control cases.

**Actual:** the controls behave correctly (`True` for the matching password, `False` for the wrong one, so the happy path is intact), and all four malformed values raise. Three of them raise `passlib.exc.UnknownHashError` as the issue describes. The truncated `$2b$12$short` value raises a plain `builtins.ValueError: salt too small (bcrypt requires exactly 22 chars)` instead — it gets past passlib's scheme identification, because the `$2b$` prefix identifies fine, and then fails inside the bcrypt handler.

That is the one thing I would flag for whoever writes the fix: a guard written as `except UnknownHashError` alone still lets the truncated-hash case escape, and a truncated hash in a database column is at least as likely as an unrecognizable one. I have not tried to write the fix yet, so I have no opinion on whether the right catch is `(UnknownHashError, ValueError)`, `passlib.exc.PasswordValueError`, or something narrower.

**Two things I saw that are not this bug**

- `passlib/utils/__init__.py:854` warns `'crypt' is deprecated and slated for removal in Python 3.13`. Harmless on 3.11; it will matter if this project ever moves past 3.12 with passlib 1.7.4.
- The probe script prints `(trapped) error reading bcrypt version` with an `AttributeError: module 'bcrypt' has no attribute '__about__'` traceback before any of my output. That is the known passlib 1.7.4 / bcrypt 4.x version-detection mismatch, passlib traps it itself, and it does not affect any result above (the control hash verifies correctly). I trimmed it from the output block for readability and mention it here so nobody wonders where it went.

Next step for me is the fix itself, on a branch from this commit.

## Eval iterations

**Run history**

Two runs, in order:

1. Targeted partial run, `--only pkg-20,pkg-09,pkg-10,pkg-16,pkg-19,pkg-03` — **6/6**. I ran
   this before spending on a full run, picking the six packages I judged most likely to break
   my first-draft rubric: the single disclosure package (pkg-20), both honest
   cannot-reproduce packages (pkg-09, pkg-10), the silent-version-deviation package (pkg-16),
   the good-report/bad-claim package (pkg-19), and one plain accept as a floor (pkg-03). All
   six agreed, and the categories line showed `disclosure 1/1`, which was the result I most
   wanted before committing $4.
2. Full run, all 20 scored packages, `--save-run eval-run.txt` — **18/20, bar: PASS**, with
   the category floor met: `clear-accept 6/8  disclosure 1/1  no-evidence 4/4
   unfollowable-comms 3/3  wrong-target 4/4`. This is the run in `eval-run.txt`, and 18/20 is
   the agreement line in that file.

I did not revise the rubric after the full run, so the file fingerprints in the `eval-run.txt`
header are the fingerprints of the files uploaded to `tools/repro-check/`.

**Package analysis**

`pkg-05` (conda/conda#16543). Gold label: **accept**. My rubric: **reject**, on
`steps-rerunnable`; the two preferred checks also came back negative, but preferred checks
never move the verdict, so the required fail is the whole story.

My check's evidence line was: "env.yml contents ('a valid `dependencies:` list plus a
`category:` section') are described but never printed or linked, so the exact trigger file
isn't reproducible by a reader."

That is a faithful execution of what I wrote. My pass condition demanded that "every input the
steps depend on is shown in the report or publicly fetchable", and pkg-05's report genuinely
does not print its `env.yml`; it describes it in a sentence. The check fired exactly as
specified.

The rubric is still wrong here, and the reason is instructive. I built `steps-rerunnable`
around pkg-18, where the reproduction runs inside a private monorepo with an unshared config.
The failure there is that the reader **cannot obtain** the input, at any price. I encoded that
as "is the input shown?", which is a proxy, and pkg-05 is where the proxy comes apart: a
reader who is told the file holds a valid `dependencies:` list plus a `category:` section can
sit down and write that file in fifteen seconds. Unavailable and unprinted are not the same
condition, and I wrote a check that could not tell them apart.

The honest version of the pass condition distinguishes them: fail when the reader cannot
obtain or recreate the input, not when the report failed to paste it. I have left the check as
it ran, because the uploaded rubric must match the fingerprint in `eval-run.txt`; the fix is
the first thing I would change if I spent another confirming run.

`pkg-03` was the other disagreement, and it is a different animal: `deviation-declared` fired
on Arch Linux vs. the issue's Kubuntu, a platform difference the report's own environment line
already discloses and which has no bearing on a ripgrep line-numbering bug. Notably, pkg-03
**agreed** in my partial run and disagreed in the full run with the identical rubric, so that
check is sitting close enough to its threshold that grader variance can tip it — which is its
own kind of finding about the check.

**Check rationale**

The check as it reads in the uploaded `tools/repro-check/rubric.md`:

```
| conventions-respected | The "contribution policy" line in the repo-facts block (live mode: `CONTRIBUTING.md`, `.github/`, any AI policy file, and the issue templates) read against both candidate comments. Treat every package produced in this course as AI-assisted work, because it is. | If the policy requires AI use to be disclosed in issues or comments, at least one of the two comments states that AI assistance was used and the extent of it. If the policy requires comments to maintainers to be in the contributor's own words, the comments read as a person's own writing rather than generic assistant output. A policy that says nothing about AI, or that asks only for understanding, responsibility, testing or human review without a disclosure requirement, passes with no disclosure line present. Fail when a stated disclosure or own-words requirement is unmet by the comments as written. | required |
```

This is the check the eval set exists to force, and it went through three decisions.

First, the sentence "Treat every package produced in this course as AI-assisted work, because
it is." Without it the check has nothing to grade: an AI-disclosure requirement is only
violated if AI was in fact used, and nothing in a bundle says so. Making that a stated premise
rather than an inference is what lets `pkg-20` fail on a package whose proof is otherwise
excellent.

Second, and this is what I rejected: my first instinct was "fail if the repo has an AI policy
and the comments do not disclose". That would have been a disaster on volume. Reading the
`contribution policy` lines across all 24 bundles, five repos have a stated AI policy and only
**one** (ghostty, pkg-20) requires disclosure in comments. conda (pkg-05) asks for
responsibility, p5.js (pkg-07) bans fully-generated contributions, prettier (pkg-12) asks for
understanding and testing, fd (pkg-09) explicitly states *no* disclosure ask for issue
comments. A disclosure-on-any-policy rule would have rejected four gold accepts and dropped me
to 14/20. So the pass condition names the distinction explicitly: conditions
("understanding, responsibility, testing or human review") pass with no disclosure line;
only a stated disclosure requirement can fail.

Third, the own-words clause. ripgrep (pkg-03) and fd (pkg-09) do not ask for disclosure — they
ask that comments to maintainers be written by a human in their own words. That is a real
requirement with a different remedy: it is satisfied by voice, not by a disclaimer, and a
check that only knew about disclosure would have been blind to it. Both are folded into the
one check because both are the same question — does the comment meet what this repo actually
asks of comments? — read off the same evidence line.

**Trade-offs**

The trade-off I took knowingly is in `steps-rerunnable`, and `pkg-05` is the package whose
result it changes: it is the one gold accept my rubric rejects on a required check, costing me
one of my two disagreements. I take "shown or fetchable" over "obtainable by the reader"
because the stricter form is mechanical — a grader walks the steps and asks at each one
whether the reader has the input in hand — while the looser form asks a grader to judge how
hard something would be to recreate, and that is the kind of judgment call the rubric-swap
activity showed two graders answering differently. I bought reliability on pkg-18's
private-monorepo shape and paid for it with pkg-05's described-but-not-pasted `env.yml`. That
is the wrong side of the trade, and I would fix it with a one-clause change — fail only what
the reader cannot obtain or recreate — but not without canaries.

On canaries: I loosened nothing after the full run, so no package could flip, and the
fingerprints in `eval-run.txt` still describe the uploaded files. If I did make that
`steps-rerunnable` change, the `--only` list would have to carry `pkg-18` and `pkg-06` (the
two `unfollowable-comms` packages the loosening could plausibly release) alongside `pkg-05`
itself, and `pkg-20` as the single-package `disclosure` canary — not because the edit touches
disclosure, but because that category has exactly one member and no volume anywhere else can
buy back a miss in it.

The second trade-off is `claim-promises-only-work` as a **required** check. It is the only
required check in the rubric that can reject a package whose proof is flawless; `pkg-19`'s
report is fine and its claim is "Kindly assign it to me, I will fix it within 2 days
guaranteed." I accept that this makes my rubric hold a package that a maintainer might well
accept anyway, because the whole point of the claim beat is that the comment is the part of
the work a stranger reads first.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
