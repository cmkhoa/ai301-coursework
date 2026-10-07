# Procedure: how this skill grades a plan package

## Read order

Read every part named below before grading any check. The order
matters: the repro evidence is the yardstick for the plan, so it is
read before the plan, and the plan is read before the comment, so the
comment can be checked against the plan it summarizes rather than
read on its own merits.

1. Live mode only: read `scope.md`. Confirm the issue URL belongs to
   the repo on its `Repo:` line. If that line still carries a
   placeholder, stop without grading and say so. Note the house rules.
   In eval mode, skip this step entirely.
2. Read `rubric.md` and `references/evidence-guide.md`. Write down the
   list of check names and the verdict rule; these are the only checks
   that will be graded.
3. Read the repo-facts block (live: `CONTRIBUTING.md`, `.github/`, any
   AI policy file). Note one thing: what the contribution policy asks
   of *issue comments* — disclosure, own words, or nothing.
4. Read the issue context (title, body, labels). Note the one behavior
   being reported and any root cause the reporter asserts, marking it
   as a claim, not evidence.
5. Read the thread highlights (live: `gh issue view <url> --comments`).
   Note every comment whose author is OWNER, MEMBER, COLLABORATOR or
   CONTRIBUTOR, and for each, what direction it gives: a confirmed
   culprit, a proposed approach, a rejected approach, linked prior art,
   or a request. A thread with no such comments gets the note "no
   maintainer direction".
6. Read the repro evidence (live: the student's posted repro comment,
   or the house repro pack as quoted in the drafts). Note three things
   before reading the plan: the behavior the artifacts actually show,
   what each control run rules in or out, and the expected-vs-actual
   pair. This note is the yardstick for `cause-grounded` and
   `test-observable`; forming it after reading the plan's diagnosis
   contaminates both.
7. Read the candidate plan straight through: diagnosis, scope,
   approach, files, test plan, risks.
8. Read the candidate plan comment last, against the notes from steps
   3, 5 and 7.

## Evidence gathering

For each evidence family a check names, pull the fact from the place
below and record it as a short quote plus its location. A check is
never graded from memory of the read-through; go back to the text.

1. **Diagnosis** (`cause-grounded`): quote the plan's cause sentence.
   Beside it, quote the line of the repro evidence that bears on that
   mechanism — the artifact that shows it, or the control run that
   touches it. Record both, and record explicitly whether any control
   run clears the component the plan blames.
2. **Scope** (`scope-bounded`): list every distinct piece of work the
   plan undertakes, one line each, from its approach steps and its
   files list. Separately list what it names as out of scope or
   deferred. Record the count of undertaken work items that are not
   needed to remove the reported behavior.
3. **Locations and approach** (`executable-start`): quote the code
   locations the plan names (file, module, crate or function paths)
   and the sentence that states the approach. Record whether that
   sentence states a decision or offers a choice.
4. **Test plan** (`test-observable`): quote the test plan verbatim.
   Record two things separately: what it says to run, and what
   observable it says must change. Either may be absent; record the
   absence rather than inferring the missing half from the other.
5. **Thread direction** (`thread-direction`): from the step-5 note,
   take the strongest piece of maintainer direction. Then search the
   plan and the comment for any sentence that engages it (follows it,
   cites it, or declines it with a reason) and quote that sentence, or
   record "no engagement found".
6. **Policy** (`conventions-respected`): quote the contribution-policy
   line, and quote any sentence of the comment that discloses tool use.
   Treat every package produced in this course as AI-assisted work.
7. **Unknowns** (`unknowns-honest`): quote the plan's risk and open-
   question sentences, or record their absence.
8. **Comment fidelity** (`comment-represents-plan`): quote the
   comment's statement of diagnosis, scope and next step beside the
   plan's, and quote any date, deadline or guarantee it contains.

## Check execution

1. Grade the checks in the order the rubric's table lists them. Grade
   each one only against the evidence recorded for it in the gathering
   stage; a check is decided by its own evidence, never by the
   impression another check left. Having gathered that evidence, grade
   without re-reading the whole package.
2. For each check, apply the pass condition as written and assign
   `pass`, `fail`, or `unclear`, with one line naming the fact or quote
   that decided it. "Looks fine" is not evidence; the line must name
   something in the package.
3. Grade the outcome, not the write-up. A terse plan that names a
   location, an approach and an observable passes the checks that ask
   for those; a long, polished, confident plan fails them if the
   substance is absent. Never let length, headings, formatting or
   confident tone move a grade.
4. When the evidence a check needs is genuinely absent from the
   package — not merely hard to find — grade `unclear` and say which
   part was missing. Before grading `unclear`, confirm the evidence
   guide's named locations for that family were all checked. Absence
   of a *statement* the check requires (no observable named, no code
   location) is a `fail`, not `unclear`: the check asked for something
   the plan was supposed to contain. `unclear` is for when the package
   does not let you tell either way, such as a cause the repro evidence
   neither exhibits nor contradicts.
5. Where a check's condition is conditional on the package — maintainer
   direction existing, a policy asking something of comments — resolve
   the condition first from the gathered evidence. If the condition
   does not hold (no maintainer comments, no comment-scoped policy ask),
   grade `pass` and say so in the evidence line; do not invent a
   requirement the package does not impose.
6. Grade every check in the rubric, including `preferred` ones, before
   assembling the verdict. A `required` fail does not end the run early:
   the remaining grades are what tells the author everything that needs
   fixing before a re-run.

## Verdict assembly

1. Partition the graded checks by weight. `required` checks decide;
   `preferred` checks are reported but excluded from the decision.
2. Convert each `required` grade to pass/not-pass: `pass` is a pass;
   `fail` and `unclear` are both not-pass, per the rubric's verdict
   rule, which states that a plan whose readiness cannot be verified
   from the package is not ready to build from.
3. If every `required` check passed, the verdict is `accept`.
   Otherwise the verdict is `reject`. There is no third verdict and no
   discretion: do not override the arithmetic because the plan reads
   well or badly. If the rubric's result feels wrong, say so in the
   summary and still emit the rubric's verdict.
4. Name the deciding check(s) in the summary. On a `reject`, that is
   every `required` check that did not pass, each with the one-line
   quote that decided it. On an `accept`, name any `preferred` check
   that failed as a stated weakness of an accepted plan.
5. Live mode only: hold the draft comment against `voice-guide.md` and
   report any rule it breaks, quoting the rule. This note never changes
   the verdict unless a rubric check reads the voice guide; no check in
   the current rubric does.
6. Emit the readable summary first — one line per check with its grade
   and evidence line — then the fenced JSON block in the format
   `SKILL.md` specifies, with every graded check listed and the verdict
   from step 3. The JSON block is last; nothing follows it.
