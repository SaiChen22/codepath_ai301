# Procedure: how this skill grades a plan package

## Read order

Read the parts of the package in the following strict sequential order before evaluating any check:

1. **Reproduction Evidence & Issue Thread First**: Read the repro command, executed logs, observed failure, and thread comments/repo facts before touching the plan.
   - *What to record*: Note the exact point and type of failure, actual behavior observed vs. expected behavior, and any maintainer-specified constraints (e.g., preserving current APIs, backward compatibility, or no new CLI flags).
   - *Why this order matters*: Evaluating the plan without the repro context biases judgment toward whether the write-up sounds convincing rather than whether it actually addresses the real underlying failure.
2. **Plan's Diagnosis & Approach**: Read the diagnosis statement and the proposed code modifications.
   - *What to record*: Note the identified root cause, whether it is upstream or downstream of the repro failure, and the specific functions/files targeted.
3. **Plan's Scope & Touch List**: Read the declared in-scope, explicit out-of-scope boundaries, and file touch list.
   - *What to record*: Note every touched file path and any deferred work or non-goals explicitly named.
4. **Plan's Test Plan**: Read the verification commands and expected post-fix output.
   - *What to record*: Note the specific expected numeric values, return codes, or output strings, and whether they re-run the repro steps.
5. **Drafted Plan Comment (`comment.md`)**: Read the proposed reply drafted for the issue thread.
   - *What to record*: Note what commitments are made to the maintainers and whether they match what is documented in the plan.

## Evidence gathering

Gather evidence for each rubric check using concrete extraction moves (cross-referencing `references/evidence-guide.md` for specific locations):

1. **For `diagnosis-grounded`**:
   - Extract the plan's root-cause explanation from `plan.md`.
   - Extract the failure stack trace / log output from the repro evidence.
   - Record: Does the explanation identify a defect upstream of the failure, or does it merely patch the symptom where the repro crashed?
2. **For `bounded-scope`**:
   - Extract the file touch list and explicit in-scope / out-of-scope declarations from `plan.md`.
   - Record: Are any untouched files or subtrees modified for formatting, migrations, or "while-I-am-here" refactoring? Note: If the author explicitly defers part of the issue and states clear boundaries, treat it as a valid bounded plan rather than an unbuildable scope.
3. **For `executable-approach`**:
   - Extract the proposed logic changes from the approach section in `plan.md`.
   - Record: Are exact functions, logic branches, or algorithmic adjustments specified with enough detail for an engineer to implement without guessing missing design decisions?
4. **For `test-decisive`**:
   - Extract the test commands and expected output from the test plan section in `plan.md`.
   - Record: Does the test re-run the repro command? Does the expected outcome state concrete, measurable indicators (e.g., exact counts, exit codes, specific error suppression) rather than vague phrases like "verify it passes"?
5. **For `comment-and-conventions`**:
   - Extract the draft comment text from `comment.md`.
   - Extract maintainer constraints and repo conventions from the thread highlights and `repo-facts`.
   - Record: Does the comment promise features beyond the plan? Does it violate maintainer instructions (e.g., introducing a new flag when instructed to keep the API stable)?
6. **For `unknowns-acknowledged`**:
   - Extract the risks, edge cases, and unknowns section from `plan.md`.
   - Record: Are potential side effects or uncertainties openly noted?

## Check execution

Grade each check sequentially against the gathered evidence:

1. **Execution Order**:
   - Grade `diagnosis-grounded` first.
   - Grade `bounded-scope` second.
   - Grade `executable-approach` third.
   - Grade `test-decisive` fourth.
   - Grade `comment-and-conventions` fifth.
   - Grade `unknowns-acknowledged` last.
2. **Absence of Evidence**:
   - If an essential section (e.g., test plan, scope boundary, or root-cause explanation) is missing entirely or contains no actionable statements, mark that check as `fail`.
   - If the text is present but genuinely ambiguous or cannot be verified against the package context, mark the check as `unclear` (`?`).
3. **Context Isolation**:
   - Grade each check solely using its gathered evidence pair without re-reading the entire package, ensuring objective and consistent evaluation across packages.

## Verdict assembly

Synthesize the final verdict and construct the output:

1. **Apply Verdict Rules**:
   - Set verdict to **`accept`** if all `required` checks evaluate to `pass`. (`preferred` checks do not block `accept`).
   - Set verdict to **`reject`** if any `required` check evaluates to `fail` or `unclear`.
2. **Deciding Check Citation**:
   - Identify the deciding check (the first failing `required` check, or the primary passing evidence if all pass).
   - Quote exact lines from the package illustrating the gap (e.g., quote the plan's diagnosis against the repro log, or the extra touched file against the issue scope).
3. **Construct Output**:
   - Output the structured result including the check grades, rationale quotes, and final verdict (`accept` or `reject`) strictly adhering to the skill's output contract.
