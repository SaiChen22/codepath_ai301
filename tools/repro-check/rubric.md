# Rubric: is this reproduction package ready to post?

This rubric answers one question: would a maintainer reading these two
comments get proof they can act on? Every check below reads the thing
itself — the artifact against the issue, the environment against what
the issue targets, the words against what the evidence supports —
never the write-up's shape. A terse four-line report can pass every
check; a formatted report with headings and a summary table can fail
all of them.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| `env-recorded` | The repro report's environment record, read against the axes the issue itself treats as part of the trigger (its version line, its environment block, and any axis its body singles out: OS, driver, shell, browser, build profile, install method). | The report names the tool version and the platform it ran on, and names every axis the issue singles out as part of the trigger. Fail if there is no environment record at all, or if an axis the issue makes central is absent, so a reader cannot place the attempt. | required |
| `env-faithful` | The versions and platform in the report's environment record, read against what the issue targets: the issue's stated version, any "confirmed on latest / on main" note in the issue or thread, and the repo-facts latest release. | The environment tested is one the issue's behavior is claimed on, OR the package names the difference out loud (in the claim comment or the report) so a maintainer can weigh it. Fail when the run happened on a version, OS, or build the issue does not target and the package never says so. | required |
| `steps-rerunnable` | The repro report's steps together with every input, file, config, and setting they depend on, read as a stranger who has only this package and the issue. | A stranger could recreate the starting state and reach the trigger from the package alone: each input is either given here or is material the issue itself already contains. Fail if any step depends on something the reader cannot obtain (a private repo, an unshared config, "our internal setup"), or if the steps never perform the trigger the issue names. | required |
| `artifact-present` | The repro report's output excerpts, logs, transcripts, prompt captures, or parser output — the parts that record what a machine did, as opposed to what the author says happened. | At least one first-hand artifact from the author's own run is shown in the package. Fail when the report is assertion only ("I can confirm this", "I verified the race condition") with nothing produced by a run, or when the artifact is only described ("the screenshot I took shows...") and never shown. | required |
| `behavior-matches-issue` | The shown artifact read line by line against the behavior the issue describes: its error text, exit status, symptom, and the input that triggers it. | Either the artifact exhibits the issue's behavior — same trigger, same failure signal — or the report states it could not reproduce and the artifact shows what happened instead. Fail when the artifact shows a different failure from the one reported (a graceful error where the issue reports a crash, a compile error where the issue reports a runtime one), or shows only that the tool ran and was set up, while the package presents it as the issue's behavior. | required |
| `outcome-honest` | Every certainty word in the claim comment and in the report's conclusion ("reproduced", "confirmed", "verified", "guaranteed reproducible", "not limited to X"), read against what the artifacts actually show. | Every stated conclusion is one the shown evidence supports, including its scope: the versions, platforms, and builds it claims to hold on are the ones that were run. An evidenced "I could not reproduce" that says so plainly passes. Fail when the package asserts a reproduction, a root cause, or a wider scope than its own artifacts show. | required |
| `claim-specific-and-honest` | The claim comment read against the issue and its thread. | The claim says at least one thing that could only be written about this issue — a finding from the attempt, the file or function or thread pointer it will start from, or a named next step — and it commits to nothing the author cannot back. Asking to work on the issue is fine; a delivery date, a guaranteed fix, or a demand that the issue be reserved is not. Fail on interchangeable boilerplate, a bare "+1", or a promise of that kind. | required |
| `ai-policy-satisfied` | The repo-facts contribution-policy line (CONTRIBUTING.md, AI_POLICY.md, AI usage sections), read against both comments. Standing assumption: treat every candidate comment as AI-assisted work, because these drafts are written with AI assistance, so a policy covering "all AI usage in any form" covers them. | Pass when the repo states no AI policy, or its policy places no requirement on issue comments. Where the policy does require disclosure of AI use, the comment names the assistance and its extent. Where the policy requires comments in the contributor's own words, the comments are specific and personal rather than generated-sounding boilerplate. Fail when a stated requirement applies and the comments do not meet it. | required |
| `control-isolates-trigger` | Any second run in the report that changes exactly one thing (dropping the flag, swapping the character, reordering the languages, moving off the boundary). | A control or contrast run is shown that separates the trigger from the surrounding setup, so the artifact proves causation and not coincidence. | preferred |
| `template-asks-covered` | The repo-facts bug-report template asks, read against what the two comments supply. | The package supplies the items the repo's template asks reporters for. | preferred |

## Verdict rule

`accept` when every `required` check is graded `pass`. Any `required`
check graded `fail` **or** `unclear` makes the verdict `reject`:
evidence I cannot verify is evidence that is not ready to post. The
verdict space is binary; there is no hold-and-see.

`preferred` checks never change the verdict. Grade and report them
anyway — they are the difference between a report that survives review
and one that is merely sufficient.

Two clarifications the checks above depend on:

- **A cannot-reproduce can be an accept.** A report that attempted the
  issue's steps, shows what its run actually produced, and names what
  differed from the reporter's conditions passes every required check.
  What fails is an unevidenced cannot-reproduce, or one narrated as a
  reproduction.
- **Claim-only drafts (live mode).** Per SKILL.md, when the package is
  a claim comment with no repro report yet, the checks whose evidence
  is the repro report — `env-recorded`, `env-faithful`,
  `steps-rerunnable`, `artifact-present`, `behavior-matches-issue`,
  `control-isolates-trigger` — are reported `unclear` with evidence
  `not yet applicable: claim-only draft` and are left out of the
  verdict rule entirely. The verdict then rests on
  `claim-specific-and-honest`, `ai-policy-satisfied`, and the parts of
  `outcome-honest` the claim comment alone can decide.
