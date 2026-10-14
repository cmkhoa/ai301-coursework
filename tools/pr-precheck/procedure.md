# Procedure: how this tool grades a PR package

## Read order

Read everything below before grading any check. The plan is read before the
diff, and the diff before the description, so the diff is measured against
the plan rather than against the description's account of itself.

1. Live mode only: read `scope.md`. Confirm the repo matches the `Repo:`
   line; if it is a placeholder, stop without grading and say so. Eval
   mode: skip.
2. Read `rubric.md` and `references/evidence-guide.md`. Write down the check
   names and the verdict rule; only those are graded.
3. Read the repo facts (eval: the repo-facts block; live: the PR template,
   `docs/CONTRIBUTING.md`, any AI policy). Note each required section,
   required artifact (changelog, "Closes #", tests) and the AI-disclosure ask.
4. Read the plan (eval: plan-context block; live: `plan.md` with
   `## Deviations`). Note: the files it names, its scope and not-in-scope
   lines, every promised work item, and its test plan with each failure mode.
5. Read the thread highlights. Note maintainer direction, or "none".
6. Read the diff and commit list. List every changed file and hunk, and
   mark each as in-plan, in-deviation, or neither. Note any debris.
7. Read the test evidence. Note what was run, the before and after, and the
   repo checks with outcomes.
8. Read the description last (eval: candidate PR description; live:
   `pr_draft.md`). Note each claim and each template section.

## Evidence gathering

For each family, record a short quote plus its location. Go back to the text;
never grade from memory of the read-through.

1. Plan fidelity (`diff-matches-plan`): the list of hunks marked in step 6,
   and the list of plan work items each located in the diff, or absent. Count
   hunks that are "neither" and plan items that are missing without a note.
2. Description claims (`description-matches-diff`): quote each claim of
   scope, fidelity or coverage and locate it in the diff; record unlocated
   claims and anything the diff does that the description denies.
3. Test evidence (`decisive-test-evidence`): quote the plan's test plan; beside
   it quote the command and observable in the evidence. Record whether the
   evidenced path is the path that broke, whether the before/after differs,
   and which of the repo's checks show outcomes.
4. Debris (`diff-clean`): record every debug print, commented-out line,
   dead function, TODO, import reshuffle, whitespace-only or re-indent hunk,
   and unrelated hunk, with its file; or record "none found".
5. Standards (`template-and-disclosure`): per required section or artifact,
   quote the filled content or record "missing". Quote the AI-use sentence or
   record its absence.
6. Thread (`thread-direction-engaged`): quote a sentence engaging maintainer
   direction, or "no direction".
7. Notes (`honest-notes`): quote stated shortfalls, and list ticked boxes
   with the evidence behind each.

## Check execution

1. Grade in the rubric's order, each check only against its own recorded
   evidence, without re-reading the whole package.
2. Apply the pass condition as written; assign `pass`, `fail` or `unclear`
   with one line naming the deciding fact or quote.
3. Grade the outcome, not the polish: length, headings, a ticked checklist and
   confident tone never move a grade. A description saying "exactly as
   planned" proves nothing; the diff decides.
4. A missing statement the check requires (no before/after, no disclosure)
   is `fail`. Use `unclear` only when the package does not let you tell
   either way, after checking every location the evidence guide names.
5. Resolve conditional checks first (maintainer direction exists? policy asks
   for disclosure?). If the condition does not hold, grade `pass` and say so.
6. Grade every check, including `preferred`, before assembling the verdict.
   A required fail does not end the run early.

## Verdict assembly

1. Partition by weight: `required` decides; `preferred` is reported only.
2. `pass` is a pass; `fail` and `unclear` on a required check are not-pass.
3. If every required check passed, the verdict is `accept`; otherwise
   `reject`. No discretion: emit the rubric's verdict even if it feels wrong,
   and say so in the summary.
4. On reject, quote the deciding check: the first failing required check in
   rubric order, with its one-line evidence; list the other failing required
   checks too. On accept, list any failed preferred check as a weakness.
5. Live mode only: hold the title and description against `voice-guide.md`,
   report each broken rule, and never change the verdict for it.
6. Emit the readable summary (one line per check), then the fenced JSON
   block from `SKILL.md` listing every graded check. The JSON is last.
