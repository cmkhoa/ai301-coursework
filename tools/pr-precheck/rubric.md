# Rubric: is this pull request ready to submit?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diff-matches-plan | The diff's changed files and hunks read against the plan's stated scope, its "not in scope" line, its files list, and any deviation notes. | Every changed hunk implements something the plan names or a deviation note records, and every work item the plan promises appears in the diff or is recorded as deferred in the plan, a deviation note, or the description. Fail if the diff carries a feature, option, flag, rename, rewrite or edit to a file the plan does not name or scopes out, or if the plan promises a piece (docs, a second fix, a test) the diff lacks with no note. A shortfall or deferral that the plan or description discloses passes. | required |
| description-matches-diff | The description's Summary, Changes bullets and fidelity claims ("exactly as planned", "no other changes", "docs updated") read against the diff. | Every claim in the description can be located in the diff, and nothing the diff does is denied. Fail if the description claims more than the diff delivers (a docs update that is not in the diff), claims fidelity ("no changes beyond the plan") that the diff contradicts, or lists changes that are not there. | required |
| decisive-test-evidence | The test evidence read against the plan's test plan and the repro's own steps: the before/after, and the outcome of the repo's own checks. | The evidence re-runs the plan's repro (the path that actually breaks, not a control or unchanged path) and shows an observable that flips from before to after, and every failure mode the plan's test plan names is exercised or its omission disclosed. The repo's own checks (tests, lint, typecheck as the repo lists them) are shown with their outcomes, or a not-run check is disclosed with a reason. Fail on "tested locally / tests pass" with no command and output, on evidence that only exercises the unchanged path, or on checks never run and not disclosed. A failing check reported honestly with its reason passes. | required |
| diff-clean | The unified diff and the commit list, scanned for debris. | No debug prints or leftover logging, commented-out code, dead or unused functions, TODO noise, formatting or re-indentation churn, import reshuffles, or unrelated drive-by hunks bury the fix. Fail if any such debris is present, even beside a correct fix. Commit messages like "wip" or "misc cleanups" corroborate but do not decide alone. | required |
| template-and-disclosure | The repo-facts block's PR-template asks and stated policy (live: `.github/PULL_REQUEST_TEMPLATE.md`, `docs/CONTRIBUTING.md`, any AI policy), read against the description and the diff. | Every section or checklist item the template requires is filled with real content, required artifacts the repo states (a changelog or whatsnew entry, a "Closes #" line, tests) are present in the description or diff, and an AI-use disclosure is present in the author's own words whenever the repo's policy requires one. Treat every package in this course as AI-assisted work. Fail if a required section is blank or boilerplate, a stated required artifact is missing, or required disclosure is absent. A policy that asks nothing of the PR on a point passes that point. | required |
| thread-direction-engaged | The issue thread's maintainer comments (OWNER, MEMBER, COLLABORATOR, CONTRIBUTOR) read against the description and the diff. | Where a maintainer gave explicit direction, the PR follows it or names it and says why not. No maintainer direction passes. | preferred |
| honest-notes | The description's Notes for Reviewers and Testing section against the diff and evidence. | Anything not done, not run or changed from the plan is stated, and no checkbox is ticked for something not shown. | preferred |

## Verdict rule

Accept if and only if every `required` check passes. Any `required` fail
rejects the package: hold it. `unclear` on a `required` check counts as
fail, because a PR that cannot be verified from the package is not ready to
submit. `preferred` checks never change the verdict; report a `fail` or
`unclear` there as a weakness of an otherwise accepted PR. A PR that
honestly discloses a shortfall can still be ready: disclosure re-ties the
mismatch, so a disclosed deferral is not a fail of diff-matches-plan or
description-matches-diff.
