# Evidence Guide: Plan and Build

This guide specifies where to locate evidence in a plan package and what constitutes passing evidence for each rubric check.

---

## 1. Reproduction Evidence & Issue Thread

### Where it lives
* Located in the package under the `## Repro Evidence` section (or Unit 2 repro comment logs).
* Issue thread highlights and repo constraints located under `## Thread Context` or `## Repo Facts`.

### What good looks like
* **Explicit Execution & Output**: Contains real executed commands, full stack traces, terminal logs, or HTTP responses demonstrating the defect.
* **Maintainer Constraints**: Thread notes capture clear boundaries specified by repo maintainers (e.g., "keep existing public API signature", "do not add new CLI flags", "backward compatible").
* **What fails**: Repro evidence that is missing actual terminal output, contains vague summaries like "it crashed", or ignores explicit maintainer instructions.

---

## 2. Diagnosis & Root Cause

### Where it lives
* Located in `plan.md` under the `## Diagnosis` or `## Root Cause` heading.

### What good looks like
* **Upstream Causality**: Explains the mechanical defect upstream of the crash that leads to the observed failure in the repro logs.
* **Direct Explanation**: Directly links the code logic defect to the failure seen in the repro without contradicting the repro facts.
* **What fails**: Symptom painting (e.g., adding a blanket `try/except` or null check at the point of crash instead of fixing the faulty generation logic upstream), or guessing causes that contradict the logs.

---

## 3. Scope & File Touch List

### Where it lives
* Located in `plan.md` under `## Scope` (explicitly checking `In-scope` vs. `Out-of-scope` / `Non-goals`) and `## Files to touch`.

### What good looks like
* **Minimal Footprint**: Only lists the minimum files required to implement the fix and its corresponding test.
* **Clear Boundaries**: Explicitly states what is out-of-scope to prevent scope creep.
* **Permitted Deferrals**: If the issue covers multiple items, explicitly marking secondary aspects as deferred/out-of-scope is acceptable and encouraged.
* **What fails**: Modifying unrelated subtrees, formatting cleanup, renaming unrelated utilities ("while I'm here"), or migrating configurations.

---

## 4. Approach & Technical Design

### Where it lives
* Located in `plan.md` under the `## Approach` or `## Proposed Changes` heading.

### What good looks like
* **Concrete and Executable**: Names specific functions, classes, data structures, and conditional branches being altered or introduced.
* **Unambiguous**: Detailed enough that an unfamiliar engineer or Claude could execute the edits without guessing missing design decisions.
* **What fails**: High-level hand-waving (e.g., "improve error handling across the module"), pseudo-code that lacks target locations, or conflicting algorithmic logic.

---

## 5. Test Plan & Expected Outcomes

### Where it lives
* Located in `plan.md` under the `## Test Plan` or `## Verification` heading.

### What good looks like
* **Re-runs Repro**: Re-executes the exact command from the reproduction evidence against the updated code.
* **Measurable Assertion**: Provides explicit, quantitative expected results (e.g., "page count returns 3 instead of 4", "returns HTTP 200 with `{'status': 'ok'}`", exit code 0).
* **What fails**: Vague assertions such as "run the app and verify it works", "check if tests pass", or tests that do not test the patched code path.

---

## 6. Draft Plan Comment (`comment.md`)

### Where it lives
* Located in the package under the `## Draft Comment` section or in `comment.md`.

### What good looks like
* **Faithful Representation**: Accurately summarizes what the plan actually proposes without introducing unverified claims or extra features.
* **Respects Thread Guidelines**: Engages with existing maintainer feedback and adheres to repository conventions (tone, formatting, no empty claims).
* **What fails**: Generic requests like "Hi, please assign this to me", promising features not included in the plan, or violating constraints set in the thread.
