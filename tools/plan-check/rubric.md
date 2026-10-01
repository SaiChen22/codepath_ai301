# Rubric: is this plan ready to post and build from?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diagnosis-grounded | The plan's root-cause explanation read against the repro evidence logs and behavior | The diagnosis traces the failure directly to an upstream defect demonstrated in the reproduced behavior, rather than patching a downstream symptom or contradicting the repro logs. | required |
| bounded-scope | The plan's scope section (in-scope and explicit out-of-scope boundaries) read against the touched file list | The change touches only the minimal files necessary to fix the root cause, explicitly states what will NOT be done, and introduces no unrelated refactoring, cosmetic rewrites, or scope creep. Scoped-down deferrals are acceptable if explicitly stated. | required |
| executable-approach | The approach section read against the files to touch and repo architecture | The proposed technical solution specifies exact logic changes and functions/files such that an unfamiliar engineer could begin implementation immediately without guessing missing core details. | required |
| test-decisive | The test plan read against the repro execution steps | Re-runs the reproduction path and defines a concrete, verifiable post-fix outcome (e.g., exact expected exit codes, specific numeric values, or precise log messages) instead of vague assertions like "verify it works". | required |
| comment-and-conventions | The drafted plan comment read against thread highlights and repo-facts | The comment addresses specific maintainer instructions or constraints in the issue thread, respects repo conventions (e.g., no breaking API changes/extra flags if forbidden), and accurately reflects the plan without unverified promises. | required |
| unknowns-acknowledged | The risks, unknowns, or edge-case interactions section of the plan | Potential risks, side effects on dependent callers, or remaining unknowns are explicitly acknowledged rather than asserted as absolute certainty. | preferred |

## Verdict rule

- **accept**: Every `required` check receives a grade of `pass`. `preferred` checks provide diagnostic feedback and do not block an `accept` verdict.
- **reject**: Any `required` check receives a grade of `fail` or `unclear`.
- **unclear handling**: Any evaluation marked as `unclear` (`?`) on a `required` check counts strictly as a `fail`.
