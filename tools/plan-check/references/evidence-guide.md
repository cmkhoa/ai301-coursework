# Evidence guide: where evidence lives in a plan package

This is the map the rubric's checks read with. For each evidence
family: where to find it in an eval bundle, where to find it live, and
what good looks like when you do.

## Diagnosis and grounding

**Where it lives.** Eval bundle: the first paragraph of `## Candidate
plan`, usually labelled `Diagnosis:` — the sentence naming the cause
and the mechanism. The behavior that cause must explain lives in
`## Repro evidence`: its `Steps`, its artifacts (pasted output,
tracebacks, exit statuses), its `Control run` lines, and its
`Expected:` / `Actual:` pair. The `## Issue` section's own root-cause
paragraph and the `## Thread highlights` are context only: they are
claims, not proof, and a package's evidence can contradict them.
Live: the diagnosis is the opening of `plan.md`; the behavior is the
student's posted repro comment on the issue thread (or the house repro
pack as quoted in the drafts).

**What good looks like.** The cause names a mechanism the artifacts
actually exhibit — the exception that was printed, the stage the
`--debug` output shows running, the constant the output shows stale —
and it survives every control run in the evidence. The decisive
reading is usually a control: a control that reproduces the symptom
with the blamed component removed clears that component, and a
diagnosis still blaming it is ungrounded no matter how confidently it
is written or how many commenters agreed. The question is not whether
the plan traced the mechanism — most plans reason from a symptom to a
cause, and that is normal engineering — but whether the mechanism it
names would produce these artifacts and survives every control. A
diagnosis that merely restates the issue's own root-cause paragraph is
grounded if the repro evidence fits it, and ungrounded the moment a
control cuts against it.

## Scope

**Where it lives.** Eval bundle: the plan's `Scope` / `Change` /
`In scope` / `Not in scope` lines, its numbered approach steps, and
its `Files:` list. Read them against the single behavior in the issue
title and the `Actual:` line of the repro evidence. Live: the
`## Scope` section of `plan.md` and the files-I-will-touch list.

**What good looks like.** One bounded change plus an explicit line
naming what it leaves alone: "saturating clamp at the two subtraction
sites; not in scope, redesigning wide-char wrapping". The bounded
version and the drive-by version look alike at a glance, so count the
fronts: a plan that fixes the bug *and* upgrades a dependency, *and*
adds an option, *and* restructures a module is four fronts, and only
the first is the issue. Deferral is a scope signal in the plan's
favour — "option 1 is a bigger rework than this bug needs, so I am not
touching it" — while "while I am in here" is the phrase that marks
creep. A large fix site is not creep if the plan bounds itself to that
site.

## Executability

**Where it lives.** Eval bundle: the plan's `Approach` steps and its
`Files:` line. Live: the files-I-will-touch and approach sections of
`plan.md`.

**What good looks like.** A reader who has never spoken to the author
can open the named file or module and start the first step. The two
things that must be present are a code location (`src/printer.rs`,
`crates/core/flags/hiargs.rs`, "the client attach path in
`zellij-server`") and one chosen approach stated as a decision, not a
menu. Precision beyond that is a bonus, not a requirement: "the exact
functions get pinned in the PR after tracing with debug logs" is a
working plan, because the decision has been made and only the address
is outstanding. What is not a working plan is a decision deferred to
build time — "gocui? tcell? not sure", "upstream or vendored,
whichever is easier", "profile first and see" — because the first
step is then still a question.

## Test plan

**Where it lives.** Eval bundle: the plan's `Test plan:` section, read
against the `Steps` and artifacts of the `## Repro evidence` block.
Live: the test-plan section of `plan.md`, read against the steps in
the student's posted repro comment.

**What good looks like.** The repro's own steps re-run, with the
specific observable that must flip named in advance: `exit 0` instead
of `exit 101`, the fixture path printed instead of nothing, the named
xfail test passing, no `rgb:` strings across five reattach cycles. A
decisive test plan also keeps the control runs: it says the
known-good case still behaves. The vague ones are recognizable by
what they omit — a suite run with no named case ("`cargo test
--workspace`, make sure nothing regresses"), or a success condition
nobody can observe ("should feel fast", "nothing else should feel
broken"). A test plan that names only tests to be written, with no
statement of what those tests will show, is in the same family.

## Honesty

**Where it lives.** Eval bundle: the plan's `Risk:` / `Risks` /
`open question` lines, its deferral reasons, and any sentence hedging
a claim. Live: the risks-and-unknowns section of `plan.md`, and after
the build, its `## Deviations` section, which records what the build
did differently from the posted plan and why.

**What good looks like.** The gap between what was run and what is
believed is visible in the text: "I cannot test Windows, so I am
leaving that variant out and flagging the shared fix site", "I have
not measured the per-print cost; if it shows in the benchmark I will
move the check". False confidence reads the other way: a long,
polished plan with no stated unknown on a change whose evidence leaves
an obvious question open, or a mechanism asserted flatly where the
evidence only suggests it. Length and polish are not honesty signals
in either direction; look for the marked-unverified sentences.

## Comms

**Where it lives.** Eval bundle: `## Candidate plan comment`, read
against two other places — `## Thread highlights` (each entry carries
its author's association: OWNER, MEMBER, COLLABORATOR, CONTRIBUTOR,
NONE) and the `contribution policy` and `bug reports` lines of the
`## Repo facts` block. Live: the draft in `comment.md`, read against
the live issue thread (`gh issue view <url> --comments`) and the
repo's `CONTRIBUTING.md`, `.github/`, and any AI policy file.

**What good looks like.** Thread-aware means the comment is visibly
written after reading the thread: it adopts the option the
collaborator numbered, engages the open PR that takes the same route,
accepts the owner's confirmed culprit, or names the owner's rejected
approach and explains the detour. Boilerplate is the comment that
would sit unchanged on any issue in any repo, and the worst case is a
comment that proposes a direction a maintainer has already ruled out
in-thread, or quietly routes around a culprit the owner isolated and a
patched binary already tested. On policy: read what the policy asks of
*issue comments* specifically. A policy requiring disclosure of AI use
needs one plain sentence naming the tool and the extent; a policy
asking that comments be in the contributor's own words is satisfied by
voice, not by a disclaimer; a policy that asks only for understanding,
responsibility or testing, or that scopes disclosure to pull requests,
asks nothing of the comment at all.
