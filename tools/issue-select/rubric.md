# Rubric: is this a good first issue?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| repo-alive | The `archived:` flag on the repo line and the "last 5 default-branch commits" list in the repo-facts block (live mode: the archived banner and the newest commit date on the repo front page). | The repo is not archived AND the newest default-branch commit is within 12 months of the capture date (live mode: of today). | required |
| maintainer-present | The "maintainer first-response sample" in the repo-facts block, the commit authors in the "last 5 default-branch commits" list, and any owner/member/collaborator comments in this issue's thread (live mode: the evidence-guide Family 1 locations). | At least one of: (a) at least one sampled issue received a first response from an owner, member, or collaborator; (b) at least one of the last 5 default-branch commits is authored by a human (not a `[bot]` account) within 12 months of the capture date; (c) an owner, member, or collaborator commented in this issue's thread within 12 months of the capture date. If every sampled issue shows "no maintainer comment in thread" and neither (b) nor (c) holds, fail. | required |
| unclaimed | The "this issue: assignees / linked PRs" line in the repo-facts block plus the full comment thread (live mode: the Assignees and Development sidebar boxes and the thread). | No assignee is set, AND there is no open linked PR (formally linked or mentioned in the thread), AND there is no unanswered claim comment ("I'll take this", "working on this", "can I work on this") posted within 12 months of the capture date. A claim older than 12 months with no linked PR is stale and does not block; a claim explicitly released or re-opened to takers by a maintainer ("feel free to pick this up") does not block. Closed unmerged PRs do not block here (they are scope evidence). | required |
| bounded-scope | The issue title, body, labels, and comment thread. | The issue asks for one bounded change (a specific bug with described actual-vs-expected behavior, a specific docs or UI task with a stated home, or a small feature whose desired behavior is concretely specified). Fail if ANY of: the issue is an umbrella/tracking/mega issue listing sub-items meant to be split up; the thread shows an unsettled design debate with no maintainer decision; a maintainer says the fix touches core internals; the issue is a pure usage/support question; the issue is a feature wish with no specification of the desired behavior; the issue has 2 or more closed-unmerged PR attempts in its history. A terse body, a checklist of acceptance criteria, or a missing reproduction is NOT by itself a fail: grade the size of the work asked for, not the polish of the write-up. | required |
| policy-allows-ai | The "contribution policy" line in the repo-facts block (live mode: CONTRIBUTING.md in the repo root or `.github/`, any AI policy files, and PR/issue templates). | The repo does not state an outright ban on AI-generated or AI-assisted contributions. Conditions (disclose AI use, personally understand and test changes, human review of AI output) are terms to follow, not bans: pass. No stated policy: pass. An explicit ban ("we do not accept AI-generated code/documentation"): fail. | required |
| recent-release | The "latest release" line in the repo-facts block (live mode: the Releases box on the repo front page). | The latest release is within 18 months of the capture date. A repo with no releases at all is graded `unclear` here (many healthy repos never cut releases; commits carry liveness for them). | preferred |
| maintainer-vouched | The issue's labels and the opener's author association in the issue header. | The issue carries a `good first issue` / `good-first-issue` / `help wanted` label, or was opened by an owner, member, or collaborator of the repo. | preferred |

## Verdict rule

Accept if and only if every `required` check passes. Any `required` fail
rejects the issue. `unclear` on a `required` check counts as fail: a first
issue you cannot verify is not a first issue you should take. `preferred`
checks never change the verdict (an `unclear` or `fail` there costs
nothing); they only rank the issues that are accepted, as tie-breakers in
the order listed.
