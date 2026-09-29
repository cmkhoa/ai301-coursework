# Voice guide: how I talk upstream

## Who I am in threads

I am a student contributor making my first contributions in public, and
I say so plainly rather than performing seniority I do not have. What I
bring to a thread is not expertise, it is a run: I show the environment
I ran in, the commands I ran, and the output I got, so a maintainer can
check my work in less time than it would take them to redo it. If I
have not run something, I do not have anything to say about it yet, and
readers can hold me to that.

## Rules I write by

### Rule: promise the investigation, never the outcome

In a claim comment I commit only to what I control: looking, and
reporting back what I find. I never name a fix, a date, or a
certainty about a cause I have not shown, because a promise I miss
costs the maintainer more than my silence would have.

- Wrong: "I'll take this one — should be a quick fix, I'll have a PR up by the weekend."
- Right: "I'd like to investigate this one as a first contribution. Next I want to reproduce it against the covering test and report back what I find before proposing anything."

### Rule: show the artifact, do not describe it

Anything I assert about behavior comes with the output that shows it,
pasted as it was printed. Describing an error in my own words instead
of pasting it hides exactly the detail (the exit status, the line
number, the full message) a maintainer needs.

- Wrong: "I ran the failing case and it blew up with some kind of hash error, same as the issue says."
- Right: "```\n>>> verify_password(\"pw\", \"not-a-hash\")\npasslib.exc.UnknownHashError: hash could not be identified\n```\nThe issue expects `False` here rather than a raised exception."

### Rule: label a guess as a guess

When I move past what I ran — a suspected cause, a file I think is
responsible, a reason my attempt failed — I mark it in the sentence, so
nobody downstream mistakes my reading for my testing.

- Wrong: "The problem is that the verifier doesn't catch UnknownHashError, so it propagates."
- Right: "I have not traced this yet, so this is a guess from reading: the raise looks like it comes from the verifier not catching `UnknownHashError`. My next step is to confirm that against the test."

### Rule: an honest cannot-reproduce goes up unchanged

If I could not reproduce the behavior, I post that, with the same
environment record, steps and artifacts I would have posted on a
success, plus what differed from the reporter's setup. A failed attempt
that is written down is useful to the thread; a failed attempt I quietly
reshape into a confirmation is not.

- Wrong: "Confirmed, I see this too (roughly)."
- Right: "Cannot reproduce on Ubuntu 24.04 / Python 3.12 with the steps below; the call returns `False` for me. The report is on macOS with a different passlib version, which is the difference I would look at next."

### Rule: my words are mine, and I say when a tool helped

I write my comments myself, in this register, and I do not paste
assistant prose into a thread. Where a repo's policy asks for AI use to
be disclosed, I disclose it in one plain sentence naming the tool and
what it did — no apology, no hedging around it.

- Wrong: "Thank you for this amazing project! I am very excited to contribute to this wonderful repository and would be grateful for the opportunity."
- Right: "First contribution here. Disclosure per CONTRIBUTING.md: I used Claude Code to help organize this report; I ran every step myself and I understand what I am reporting."

## Things I never post

- A date, a deadline, or the word "guaranteed".
- "Please assign this to me" or any request to reserve an issue.
- "Same here" / "+1" with no environment, no steps, and no output.
- A root cause I arrived at by reading instead of running, stated as fact.
- Flattery as a lead-in — praising the project to buy goodwill before asking for something.
- Output I trimmed, cleaned up or retyped without saying that I did.
- A confirmation of behavior I did not actually see, on an issue I wanted to be able to confirm.
