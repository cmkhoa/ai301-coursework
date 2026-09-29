# Evidence guide: where proof lives in a reproduction package

A reproduction package has two halves that must be read against each
other: the issue (what somebody says goes wrong) and the candidate
comments (what this author says they saw). Every check in `rubric.md`
names a place in one half and a place in the other. This file is the
map: for each family of proof, where it lives in an eval bundle, where
it lives on GitHub in live mode, and what good looks like when you get
there.

A note that applies to every family: in eval mode the bundle is frozen
on its stated capture date and is the whole world — never reach for the
live issue it names. In live mode the drafts in front of you are the
candidate half, and the issue thread is the other half.

## Environment

**Where it lives.** In the bundle: the repro report's opening
`Environment:` line (or the equivalent sentence in its prose), and,
for what it should be measured against, the issue's own version line,
its title and body (platform-specific issues usually say so in the
first sentence), and the `bug reports:` line in the repo-facts block,
which lists what that repo asks reporters for. In live mode: the
`Environment`/`Version` fields of the draft comment, the issue's
rendered bug-report form on GitHub, and the repo's issue template under
`.github/ISSUE_TEMPLATE/`.

**What good looks like.** A stranger can name the version of the
software under test and the machine it ran on after one read, and every
factor the issue treats as decisive appears. Decisiveness is set by the
issue, not by a fixed list: on an issue whose title says "on Windows",
the OS and the driver are part of the environment; on an issue about a
panic in a debug build, the build profile is; on a shell-prompt issue,
the shell and its version are. One dense line naming version, install
method and OS ("yq 4.53.3 (Homebrew), macOS 15.5 (arm64)") is a good
record. A report with no environment line anywhere is the failure this
family exists to catch: without it nobody can place the attempt, and
nobody can tell a real difference from a coincidence.

## Steps

**Where it lives.** In the bundle: the report's `Steps:` block — the
commands, the config file contents, the input documents and scripts it
quotes — plus the issue's own reproduction steps to compare against, and
the thread highlights, where a maintainer often names the trigger
condition ("only with the `--offline` flag", "only when go.mod is in a
subdirectory"). In live mode: the same block in the draft, and the
issue's steps and comments on GitHub.

**What good looks like.** Followable means executable by someone who
has only this report: the starting state is written down (an empty
directory, a fresh `git init`, a named config file with its contents
shown), the commands appear as they were run rather than described, and
every file the commands read is printed in the report or fetchable from
the issue or the repo. The test is mechanical — walk the steps and ask
at each one whether a reader could perform it. Steps that pass through
material the reader cannot have ("our internal `.golangci.yml`, which I
cannot share", a private monorepo) fail the test however faithful the
result is, because nobody can check them. Steps that silently skip a
factor the issue names as necessary fail it too: they describe some
other run.

## Behavior shown

**Where it lives.** In the bundle: the fenced output blocks, log
excerpts, screenshots-described, and measurements inside the report,
read against the issue's own quoted output, its error text, and its
stated exit status. In live mode: the same blocks in the draft, and the
issue body's own artifacts on GitHub.

**What good looks like.** The artifact shows the issue's failure, not a
neighbour of it. Read the artifact first and the prose afterwards, and
ask what the artifact alone proves. A graceful argument-validation
error exiting 1 is not the capacity-overflow crash exiting 101 that the
issue reported. A compile error from an expression the author rewrote
is not the runtime path error the issue reported. Garbled escape
sequences in a terminal that is still alive is not the crash the issue
reported. A version banner and a session list prove the program
started, and nothing else. Where the artifact and the narration
disagree, the artifact is the evidence and the narration is a claim. A
contrasting run — the same command with the trigger removed, or the
known-good variant the issue names — is the strongest form this family
takes, because it shows both that the failure happens and what makes it
happen.

## Honesty

**Where it lives.** At the seam between the two halves of the package:
every assertion in the claim comment and the report's `Expected:` /
`Actual:` / summary lines, set against the artifacts printed in the
same report. The words are in one place and their backing is in
another; this family is read by moving between them.

**What good looks like.** Each assertion is traceable to something
shown, or is flagged as a guess. "I verified this race condition" with
no transcript is not honest reporting; "a fish shell resolving `PWD`
logically looks necessary to hit this, and I did not have one" is,
because it labels itself. An honest cannot-reproduce is a complete and
passing outcome: it records a real attempt, shows what happened
instead, and names what differed from the reporter's setup. What fails
here is the reverse shape — certainty exceeding evidence: a root cause
diagnosed from reading alone, a result generalized to a build or
platform that was never run, "guaranteed reproducible" attached to no
measurement, or an `Expected:` line that contradicts the output printed
above it. Confidence is not evidence, and polish is not proof: the
long, well-formatted, entirely sure report is the one this family
catches most often.

## Comms

**Where it lives.** In the bundle: the candidate claim comment, read
against the issue's own content, and the `contribution policy:` and
`bug reports:` lines in the repo-facts block. In live mode: the draft
comment, `CONTRIBUTING.md` in the repo root or `.github/`, any
`AI_POLICY.md` / `AI_USAGE_POLICY.md` the guide links to, and the issue
and PR templates.

**What good looks like.** A claim comment is specific when swapping it
onto another issue would make it wrong: it names the behavior
reproduced, a file or function it will read next, or the concrete
question it is going to answer. It is honest when it promises only
investigation and a report back — never a fix, never a date, never a
demand that the issue be held. "+1, any updates?" and "Kindly assign it
to me, I will fix it within 2 days guaranteed" are the two failure
shapes, and they fail for opposite reasons: nothing offered, and more
offered than anyone can deliver.

On policy, the distinction that decides the check is between conditions
and disclosure. Most policies set conditions — understand your change,
test it, take responsibility, have a human review AI output — and those
are terms to follow, not something a comment has to announce; a package
that meets them passes with no disclosure line. A minority go further
and require that AI use be disclosed, in any form, stating the tool and
the extent of the assistance; there, a comment that does not disclose
fails no matter how good the proof above it is, and the work in this
course is AI-assisted, so the requirement always applies to us. A third
shape asks that comments to maintainers be written by the contributor
in their own words: that one is met by voice, not by a disclaimer, and
is failed by text that reads as unedited assistant output. Read the
policy line for which of the three it is before grading the comments
against it.
