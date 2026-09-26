# Evidence guide: where proof lives in a reproduction package

This is the map the rubric's checks read. For each family of proof:
where to look, and what good looks like when you get there.

Two rules that hold across every family:

- **Read the candidate against the issue, never on its own.** Almost
  every family's pass condition is a comparison. An artifact is not
  good or bad; it either shows the behavior this issue describes or it
  shows something else.
- **Absent is not the same as unclear.** If you looked in the place
  this guide names and the proof is not there, that is a `fail`, not
  an `unclear`. `unclear` is for when the package genuinely does not
  let you tell.

## Environment

**Where it lives.** In an eval bundle: the `Environment:` line (or
table) at the top of the candidate repro report, and sometimes a
version claim repeated in the candidate claim comment. Compare it
against the issue section — the reporter's version line, their
environment block, and any axis the issue's body or title singles out
(`(Windows)` in a title, "with a non-English language first", "debug
builds panic, release builds wrap") — plus the `latest release` and
`bug reports` lines in the repo-facts block, which say what the repo
expects a reporter to state. Thread highlights matter too: a
maintainer saying "cannot reproduce with default settings" or
"confirms on 2.3.3 and current main" moves what the issue targets.

In live mode: the environment record is in the student's draft repro
comment. The issue's target is on the issue page (the rendered
bug-report template fields, the body, and later comments), and the
repo's expectations are in `.github/ISSUE_TEMPLATE/` and the repo's
releases page.

**What good looks like.** The record names the tool version and the
platform, plus every axis the issue makes part of the trigger — a
minikube issue titled "on Windows" needs the OS and the driver; a
hyperfine overflow that behaves differently per build profile needs
the build profile; a starship issue reported on macOS + fish needs the
OS and shell. And the environment tested is one the issue's behavior
is claimed on, or the package says otherwise out loud: "The issue was
filed against 13.0.0; behavior is unchanged on 15.2.0" is a pass, and
so is "the report is macOS + fish; shell and OS both differ, starship
version matches". Testing pandas 1.5.3 against an issue the reporter
confirmed on latest and main, with no mention of the gap, is a fail
even though the traceback looks right: the run says nothing about the
reported bug.

## Steps

**Where it lives.** The `Steps:` block of the candidate repro report,
plus everything it points at: the fenced commands, the input files it
creates, the config it says it used, the layout or sketch it built.
Read it beside the issue's own reproduction steps, which is where you
learn what the actual trigger is.

In live mode: the same block in the student's draft, read against the
issue's steps and any minimal reproduction the reporter linked.

**What good looks like.** A stranger holding only this package and the
issue can get from a clean machine to the trigger. Every input is
either given here or is material the package already contains —
"created `test.txt` with the exact 12 lines from the issue" is
followable, because those 12 lines are in the issue section; so is
"wrote a minimal `env.yml` containing a valid `dependencies:` list
plus a `category:` section", because that is enough to rebuild. The
commands appear as they were run, with the flags that matter.

It fails when a step needs something the reader cannot obtain — a
private monorepo, an internal `.golangci.yml` the author says they
cannot share, "our pre-commit hook" — or when the steps quietly skip
the trigger: running `minikube start` on an issue whose repro is
`minikube start --driver vmware`, or running a prefix range on an
issue about offset-from-end syntax. A skipped trigger usually shows up
later as an artifact that does not match; catch it here first.

## Behavior shown

**Where it lives.** The fenced output blocks, log excerpts, tracebacks,
transcripts, produced-CSS panes, and prompt captures inside the
candidate repro report, plus its `Expected:` and `Actual:` lines. The
thing to compare against is in the issue section: the reporter's own
output block, the error text they quote, the exit status, the symptom
they name.

In live mode: the artifacts are in the student's draft; the comparison
target is the issue body and any output a maintainer posted in the
thread.

**What good looks like.** The artifact is first-hand — something a
machine produced during the author's run, pasted in, not summarized.
"The screenshot I took shows the session running with three tabs" is
not an artifact; the pasted terminal output above it is.

And the artifact shows *this* issue's behavior: same trigger input,
same failure signal. Line up the specifics. An issue reporting a
capacity-overflow crash at exit 101 is not shown by a graceful
argument-validation error at exit 1. An issue reporting `Invalid path
expression` is not shown by a compile error from an expression the
author changed. An issue reporting that the terminal process crashes
is not shown by garbled escape-sequence text with the window still
open and the prompt back. An issue reporting a blank unresponsive pane
is not shown by a version banner and a session list proving zellij
runs. These are the packages that read as thorough and prove nothing,
and the only way to catch them is to read the artifact against the
issue's words instead of against the report's confidence.

A control run — the same command with one thing changed, so the
failure appears and disappears on cue — is what turns an artifact from
"this happened" into "this is what causes it". Dropping `-r '$1'` and
getting correct line numbers, swapping 🚧 for ✅ and getting the block
scalar: that is the strongest shape this family takes.

## Honesty

**Where it lives.** Where the package's words and its artifacts meet:
the claim comment's verbs ("reproduced", "confirmed", "fully
reproduced"), the report's opening `Result:` or summary line, and its
`Conclusion`. Read each one against the fenced blocks above it.

In live mode: the same, in the student's two drafts.

**What good looks like.** Every conclusion is one the shown evidence
supports, at the scope it claims. Watch three seams:

- **Claimed vs shown.** "I verified this race condition" with no
  transcript, "guaranteed reproducible" with no measurement, "I have
  completed a thorough, end-to-end reproduction" above an artifact
  that shows the opposite outcome. The confidence is the tell; go
  looking for the run that would back it.
- **Scope creep.** A run on one build narrated as proof about another:
  "my reproduction confirms the bug on the current Store release,
  which also demonstrates the problem is not limited to git-main
  builds", on an issue where a maintainer could not reproduce on that
  release. What was run is the only thing that was shown.
- **Backwards framing.** `Expected:` restating the bug and `Actual:`
  restating the setup, so the report appears to confirm something it
  never observed.

**An honest cannot-reproduce is a pass, not a hedge.** The shape that
earns it: a real attempt at the issue's steps, the artifacts from that
attempt shown, the result stated plainly up front ("Result: I could
NOT reproduce scenario 2"), the scope of what was and was not tried
named, and a concrete account of what differed from the reporter's
conditions and what a triggering setup would likely need. That report
tells a maintainer something they did not know. What fails is the
unevidenced cannot-reproduce, and the failed attempt narrated as a
success.

## Comms

**Where it lives.** The candidate claim comment, read against the issue
and its thread highlights; both comments read against the repo-facts
block's `bug reports` line (the template's asks) and its `contribution
policy` line (CONTRIBUTING.md, AI_POLICY.md, "Use of AI" and
"Generative AI" sections).

In live mode: the repo's `.github/ISSUE_TEMPLATE/`, `CONTRIBUTING.md`,
and any `AI_POLICY.md` or AI-usage section, plus the thread itself to
see what has already been said and by whom.

**What good looks like, for the claim comment.** It says something only
someone who read this issue could say: a finding from the attempt, the
file or function it will start from, the maintainer pointer it is
acting on ("per the pointer above I'll start reading the standard
printer in grep-printer"), or the thread comment it builds on. It asks
to work on the issue without demanding it, and promises nothing it
cannot keep. "Kindly assign it to me, I will fix it within 2 days
guaranteed, please keep this issue reserved for me" fails on every
limb at once, and it fails even when the repro report attached below
it is excellent — the two comments are read as a pair, and this one is
what a maintainer sees first.

**What good looks like, for policy.** Read the policy line literally
and check whether it reaches issue comments at all. Three shapes recur:

- *No stated AI policy*, or a policy about code and pull requests
  only: nothing to satisfy here, pass.
- *Permissive-with-responsibility* ("generative AI tools welcome; you
  are responsible for all contributions and must review and understand
  them"): no disclosure ask, pass.
- *Disclosure required* ("all AI usage in any form must be disclosed,
  stating the tool used and the extent of the assistance"): the
  comment must say so, in the comment. "Per the AI usage policy: I
  used an AI assistant to help me organize this report; I ran and
  verified every step myself" is what a pass looks like.

**Assume the comments are AI-assisted.** These drafts are written with
AI assistance, so a policy covering "all AI usage in any form" covers
them, and silence is a failure to disclose rather than an absence of
anything to disclose. Do not reason "no AI use is visible in the text,
so the rule does not apply" — that reading makes a disclosure rule
unenforceable by construction. Where a policy instead requires that
comments to maintainers be *in the contributor's own words*, the test
is different and softer: a specific, personal, first-hand comment
satisfies it, and generated-sounding boilerplate does not.

Finally, the template's asks are a checklist for what the package
should already contain (version, OS, steps, actual, expected, a
playground link, the output of `conda info`). Treat a gap as a note on
quality, not as a disqualification: the proof families above are what
decide readiness.
