# Rubric: Issue Selection

## Checks

| Check | Evidence | Pass condition | Weight |
| :--- | :--- | :--- | :--- |
| `no-assignee` | `repo-facts` and top metadata | Assignees field is `none` and no contributor is formally assigned | required |
| `no-in-progress-pr` | `repo-facts` linked PRs and recent comments | No open, draft, or unmerged pull requests addressing this issue, and no contributor has submitted an active implementation PR in the comments | required |
| `maintainer-active` | `repo-facts` last default-branch commit and PR merges | Repository has default-branch commits or maintainer merges within the last 90 days (excluding pure stale bot automation) | required |
| `ai-policy-permitted` | `repo-facts` contribution policy (`CONTRIBUTING.md`) | Does NOT explicitly ban or restrict LLM / AI-assisted contributions | required |
| `reasonable-scope` | Issue description and title | Pass if the task addresses a concrete bug, a focused enhancement, or an isolated feature with clear reproduction/expected behavior. Reject only if it explicitly demands a complete codebase rewrite, framework migration, system-wide architectural redesign, or vague multi-month roadmap initiative | required |
| `no-blocking-labels` | Issue labels | Does not carry explicit rejection labels such as `wontfix`, `invalid`, `duplicate`, or `stale`. Standard triage or classification labels (e.g. `needs-triage`, `bug`, `enhancement`, `discussion`) pass | required |

## Verdict Rule

- **Reject**: If ANY `required` check evaluates to `Fail (F)` or `Unknown (?)`.
- **Accept**: ONLY if EVERY `required` check evaluates to `Pass (P)`.