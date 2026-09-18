# Unit 1 — Issue Selection

Path: beat-1-sandbox/unit-1/selection.md

Record of the issue carried into Unit 2, and of the evaluation runs that produced eval-run.txt. This file is graded at the path above; a copy kept anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong label is not graded.

## Selected issue

Issue link
https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72

Verdict output
```json
{
  "item": "[https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72](https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72)",
  "checks": [
    {"name": "no-assignee", "grade": "pass",
     "evidence": "Issue 72 metadata: \"assignees\":[] — nobody formally assigned"},
    {"name": "no-in-progress-pr", "grade": "pass",
     "evidence": "No CROSS_REFERENCED/CONNECTED PR timeline events, zero comments, and `gh pr list --state all --search 72` returns []"},
    {"name": "maintainer-active", "grade": "pass",
     "evidence": "Latest main commit 2026-09-16 by Andrew Burke (human), repo pushedAt 2026-09-16, isArchived false — within 90 days of today"},
    {"name": "ai-policy-permitted", "grade": "pass",
     "evidence": "No AI_POLICY.md/AGENTS.md and docs/CONTRIBUTING.md contains no ban or restriction on AI-assisted contributions"},
    {"name": "reasonable-scope", "grade": "pass",
     "evidence": "\"Verification against a malformed hash should fail closed (return False), not raise\" — 2 named files, estimated effort 1–2 hours"},
    {"name": "no-blocking-labels", "grade": "pass",
     "evidence": "Labels: bug, good first issue, api, tier-1 — no wontfix/invalid/duplicate/stale"}
  ],
  "verdict": "accept"
}
```

## Eval iterations

Quote source text directly in each field below. Paraphrase does not satisfy them.

Run history
15/20, 16/20, 18/20

Issue analysis
issue-10: The gold label was reject, and our rubric decided reject. The repository's CONTRIBUTING.md contained an explicit ban against LLM and AI-generated code contributions, triggering a failure on the required ai-policy-permitted check.

Check rationale
`ai-policy-permitted`: "repo-facts contribution policy (CONTRIBUTING.md) | Does NOT explicitly ban or restrict LLM / AI-assisted contributions | required"
This check is required because contributing AI-assisted code to a repository with an explicit anti-AI policy leads to immediate PR closure and violates community norms.

Trade-offs
This check gives up issues in strictly anti-AI repositories that might otherwise have well-scoped, bite-sized tasks. For instance, issue-10 was a clean bugfix that our early rubric accepted, but enforcing this check correctly flipped it to reject to comply with repo policies.

## Selection rationale

Graded on whether all three are answered, in your own words. Not on how good the reasoning is, and not on length — a short honest answer to each earns the full marks. This is also the basis for the claim comment you write in Unit 2.

Selection rationale
1. The issue fits my strong interest in Python and backend development. The scope is tight and realistic (fixing unhandled exception handling in password verification), allowing completion within the Unit 2-4 timeline.
2. The verdict correctly verified that the issue is unassigned, active, and free of PR conflicts. Beyond the automated checks, I weighed that the fix has clear test coverage (`tests/unit/test_security.py`) and does not require provisioning heavy third-party services.
3. The anticipated difficulty in claiming it is low because it has no formal assignee, zero active comments, and the house rule permits claiming issues for course work.