# Plan: #72 — `verify_password` raises on malformed stored hashes

Issue: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72
Repro comment (mine, unit 2): https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72#issuecomment-5883026020
Base commit: `2f4e82f` (`main` at fork time)

## Diagnosis

`verify_password()` (defined at `core/security.py:27`) hands the stored
value straight to passlib at `core/security.py:37`:

```python
return bool(pwd_context.verify(plain_password, hashed_password))
```

There is no error handling around that call, so anything passlib raises
while identifying or parsing the stored hash escapes to the caller
instead of the function returning `False`.

My reproduction pins down what actually escapes, and it is not one
exception type. My repro comment's step 3 output, in full:

```
control, correct bcrypt hash -> True
control, wrong password vs same hash -> False
the test's value: 'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError: hash could not be identified
empty string: '' -> raised passlib.exc.UnknownHashError: hash could not be identified
truncated $2b$ prefix: '$2b$12$short' -> raised builtins.ValueError: salt too small (bcrypt requires exactly 22 chars)
well-formed sha256_crypt hash: '$5$rounds=535000$...' -> raised passlib.exc.UnknownHashError: hash could not be identified
```

Two things in that output decide the fix:

1. The two controls return `True` and `False` correctly, so the happy
   path is intact and the defect is specific to unusable stored hashes.
2. Values that passlib cannot identify at all raise
   `passlib.exc.UnknownHashError`, but a value whose `$2b$` prefix
   identifies fine and then fails inside the bcrypt handler raises a
   plain `builtins.ValueError`. A guard written as
   `except UnknownHashError` alone leaves that second shape raising.

`UnknownHashError` is itself a subclass of `ValueError`
(`UnknownHashError -> ValueError -> Exception`), so one `except
ValueError` covers both shapes my repro produced.

## Scope

**In scope:** the exception handling of the single `pwd_context.verify()`
call in `verify_password()` (`core/security.py`), the `strict` `xfail`
marker on the covering test, and regression tests for the malformed
stored-hash shapes my repro exercised.

**Not in scope, deliberately:**

- `hash_password()`, `create_access_token()`, `decode_access_token()`
  and the rest of `core/security.py`: not implicated by the repro.
- The caller in `api/routes/auth.py:80`. It already treats a `False`
  return as a failed login, so it needs no change once the function
  fails closed; I am not touching the login flow or its error messages.
- The passlib 1.7.4 / bcrypt 4.3.0 version-probe traceback and the
  `crypt` deprecation warning I flagged in my repro report. Both are
  real, both are noise from this bug's point of view, and bundling a
  dependency bump into a four-line fix is exactly the drive-by this
  issue does not need.
- Any broader "validate stored hashes at write time" or password-policy
  work.

## Files I will touch

- `core/security.py` — `verify_password()` only.
- `tests/unit/test_security.py` — remove the `@pytest.mark.xfail`
  block on `TestSecurity::test_verify_with_wrong_hash_format` (lines
  219-222 at `2f4e82f`), and add the regression cases next to it.

## Approach

1. In `verify_password()`, wrap the `pwd_context.verify()` call so a
   stored hash passlib cannot use returns `False`:

   ```python
   try:
       return bool(pwd_context.verify(plain_password, hashed_password))
   except PasswordValueError:
       raise
   except ValueError:
       return False
   ```

   with `from passlib.exc import PasswordValueError` at the top.

2. The `PasswordValueError` re-raise is the one judgement call here, and
   it is deliberate. `PasswordSizeError` (a `PasswordValueError`, which
   is also a `ValueError`) is raised for a *plaintext* password over
   passlib's 4096-byte limit — a complaint about the caller's input, not
   about the stored hash. Checked on my setup:

   ```
   long plain -> PasswordSizeError password exceeds maximum allowed size
   ```

   Swallowing that into `False` would quietly change behaviour for an
   input this issue says nothing about, so the narrow version keeps
   today's behaviour there and only fails closed on the stored hash.
   If review prefers the blunter `except ValueError: return False`, that
   is a one-line simplification and I will take it.

3. Update the docstring's `Returns:` to say that an unusable stored hash
   returns `False`.

4. Remove the `@pytest.mark.xfail(strict=True, reason="issue #72
   (manifest H-05) ...)` decorator from
   `test_verify_with_wrong_hash_format`. `docs/CONTRIBUTING.md` says
   dropping the marker is part of fixing a seeded bug, and because the
   marker is `strict=True` CI goes red if the test starts passing with
   it still attached.

5. Add a parametrized regression test covering the four stored-hash
   shapes from my repro — `"not_a_valid_bcrypt_hash"`, `""`,
   `"$2b$12$short"`, and a well-formed `sha256_crypt` hash — each
   asserting `verify_password("password", value) is False`. The
   truncated `$2b$` case is the one the issue's own wording would not
   have caught.

6. Run `make check && make test-unit` before committing, per
   `docs/CONTRIBUTING.md`.

## Test plan

My unit-2 repro steps, re-run against the change. Before/after output
goes in the submission.

| # | Command | Before (recorded in my repro comment) | Expected after |
|---|---|---|---|
| 1 | `pytest tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format -v` | `1 xfailed` (marker hides it) | `1 passed` (marker gone, test really passes) |
| 2 | same, `--runxfail --tb=short` | `1 failed` with `passlib.exc.UnknownHashError: hash could not be identified` raised at `core/security.py:37` | `1 passed`, no traceback |
| 3 | `python probe.py` (my repro's four malformed values plus two controls) | controls `True` / `False`; all four malformed values raise (three `UnknownHashError`, one `ValueError: salt too small`) | controls still `True` / `False`; all four malformed values return `False`, nothing raised |
| 4 | `pytest tests/unit/test_security.py -q` | `24 passed, 1 xfailed` | all tests pass, no xfailed, including the new regression cases |
| 5 | `make check && make test-unit` | n/a | clean (lint, format, typecheck, unit suite) |

The observable that decides the fix is row 3: the same four values that
raised in my repro return `False`, while the two controls are unchanged.
Row 1 is the one that catches a half-done fix — with the marker still
attached, a now-passing test makes strict xfail fail CI.

`probe.py` stays out of the commit; it is a scratch script from the
reproduction, and rows 1-5 above are the version that runs in CI.

## Risks and unknowns

- **The `PasswordValueError` re-raise is a judgement call, not a
  measured result.** I know `PasswordSizeError` is raised for an
  over-long plaintext because I ran it; I do not know whether this
  project would rather have that fail closed too. Flagged for review,
  and either way is a one-line change.
- **I have not traced every exception passlib can raise** from
  `verify()` for a stored hash — only the four shapes my repro
  exercised. A `TypeError` is still possible for a non-string stored
  value (`verify_password("password", 12345)` raises `TypeError: hash
  must be unicode or bytes, not int` on my setup). The column is typed
  `str` and no caller passes an int, so I am not guarding types; I am
  recording it rather than pretending the catch is exhaustive.
  `None` already returns `False` without a guard, which I checked.
- **Security direction.** Failing closed is what the issue asks for, and
  it means a corrupted hash column yields "wrong password" rather than a
  500. That is the safer direction for auth, but it does make a
  corrupted row look like a user error in logs. I am not adding logging
  in this change; if maintainers want a warning on an unusable stored
  hash, that is a clean follow-up.
- **Only tested on my environment:** macOS 26.6.2 (arm64), Python
  3.11.16, passlib 1.7.4, bcrypt 4.3.0. CI pins 3.11, so I expect
  parity, but I have not run the other platforms reported in the thread.
  Four classmates posted Windows 11 / Python 3.12 reports on this issue
  (@Aqureshi106, @IDLinares, @AdrianOlm1, @jtan21at) showing the same
  exception types I saw; that is their evidence, not mine, and I have
  not run Windows myself.

## Deviations

Nothing changed. The build is the plan: the four-line `try/except` in
`verify_password()` with the `PasswordValueError` re-raise, the docstring
note, the strict `xfail` marker removed from
`test_verify_with_wrong_hash_format`, and the parametrized regression
test over the four malformed shapes. Both files I said I would touch are
the only two files in the commit (`5d28713`).

Two things worth recording even though neither moved the plan:

- The `PasswordValueError` re-raise survived contact with the code. I
  had flagged it as the one judgement call I might have to give up; it
  cost nothing to implement and `mypy` is clean on it, so it ships as
  planned and the question still stands for review.
- I added the regression test as a *new* parametrized test beside
  `test_verify_with_wrong_hash_format` rather than parametrizing that
  test in place. The issue names that test by name and the manifest
  tracks it as H-05, so leaving its name and body alone keeps the link
  between issue, marker and test readable. The plan did not specify
  either way; this is how it landed.

The test plan ran as written: rows 1-5 all produced the expected
observables, with the before output captured on the same machine by
stashing the change. No surprises, so nothing to amend on the posted
comment.
