# Unit 2: reproduction — write-up

Issue: [codepath/pathreview-ai301-fa26-s1#72](https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72)
— `verify_password` raises `UnknownHashError` on malformed stored hashes
instead of returning `False` (manifest H-05).

<!-- Reflection prompts below are my best guess at what the portal asks;
swap in the exact wording from the Unit 2 Assignment tab if it differs. -->

## Claim comment

Link: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72#issuecomment-5844126669

> Hi, I am a student working on my first open-source contributions, and I would like to take a run at this issue.
>
> I see PR #75 was opened earlier today, but I will still reproduce the failure independently in a clean local environment to document the behavior and verify what exceptions passlib raises across different malformed hash formats.
>
> My next step is setting up the dependencies locally, running test_verify_with_wrong_hash_format under --runxfail, and testing verify_password against both non-bcrypt strings and truncated hashes. I will report back with my exact environment and output. If PR #75 is already set to merge, I am happy to hand over what I find or pivot to another open issue.

**Reflection.** This is a shared house issue — nine classmates had already
claimed it before me, and PR #75 was already open by the time I posted. The
scope's house rule is that a classmate's claim doesn't block mine, so I
posted anyway, but I wrote the claim to be honest about that crowding
rather than pretend I was first: I named PR #75 by number, said I'd still
reproduce independently, and left an explicit out ("happy to hand over
what I find or pivot") instead of asking to have the issue reserved for
me. The one thing I'd tighten next time is naming a concrete fallback
issue up front, since "pivot to another open issue" is vaguer than the
rest of the comment.

## Repro comment

Link: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72#issuecomment-5844314735

> **Environment**
>
> - OS: macOS (arm64)
> - Python: 3.14.6
> - Installed relevant packages: passlib==1.7.4, bcrypt==4.3.0, pytest
> - Branch/Commit: main (forked from codepath/pathreview-ai301-fa26-s1)
>
> **Steps and observed**
>
> 1. Control check (valid hash), to confirm `verify_password` works normally before touching the malformed-hash path:
>
> ```
> $ python3 -c "
> from core.security import verify_password, hash_password
> h = hash_password('password')
> print('control (valid hash):', verify_password('password', h))
> "
> control (valid hash): True
> ```
>
> 2. Direct trigger on the covering test's input:
>
> ```
> $ python3 -c "
> from core.security import verify_password
> verify_password('password', 'not_a_valid_bcrypt_hash')
> "
> Traceback (most recent call last):
>   ...
> passlib.exc.UnknownHashError: hash could not be identified
> ```
>
> 3. Repository test runs, as shipped and then with `--runxfail` to surface the underlying exception:
>
> ```
> $ pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v
> tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format XFAIL
> ```
>
> ```
> $ pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v --runxfail
> tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format FAILED
> ...
> E   passlib.exc.UnknownHashError: hash could not be identified
> ```
>
> 4. Additional malformed hash shapes, to see whether one exception type covers all of them:
>
> ```
> $ python3 -c "
> from core.security import verify_password
> cases = ['', 'plaintext', '\$2b\$notarealhash', 'not_a_valid_bcrypt_hash']
> for bad in cases:
>     try:
>         print(repr(bad), '->', verify_password('password', bad))
>     except Exception as e:
>         print(repr(bad), '-> raised', type(e).__module__ + '.' + type(e).__name__, str(e))
> "
> '' -> raised passlib.exc.UnknownHashError hash could not be identified
> 'plaintext' -> raised passlib.exc.UnknownHashError hash could not be identified
> '$2b$notarealhash' -> raised builtins.ValueError not enough values to unpack (expected 2, got 1)
> 'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError hash could not be identified
> ```
> 5. Exception inheritance check
>
> ```
> python3 -c "from passlib.exc import UnknownHashError; print('issubclass(UnknownHashError, ValueError):', issubclass(UnknownHashError, ValueError))"
> ```
> Output:
>
> issubclass(UnknownHashError, ValueError): True
>
> **Expected vs actual**
>
> Expected: `verify_password` fails closed and returns `False` on any malformed or unrecognized hash string.
>
> Actual: `passlib.exc.UnknownHashError` escapes when the hash is unrecognized, and a bare `builtins.ValueError` escapes when a string has a `$2b$` prefix but truncated structure. `passlib.exc.UnknownHashError` inherits from `ValueError` (checked against the installed passlib), so catching `ValueError` in `core/security.py` covers both shapes.
>
> Next I will check `core/security.py` around line 37 and draft the change to catch `ValueError` during verification, then remove the `xfail` marker from `test_verify_with_wrong_hash_format`.

**Reflection.** The control run (step 1) and the direct trigger (step 2)
are the isolating pair: same function, same valid setup, only the hash
argument changes between "returns True" and "raises." Step 5 is the one
I'm most glad I added — several classmates on the thread asserted that
catching `ValueError` covers both exception shapes without checking it,
and I could have made the same unverified claim. Running
`issubclass(UnknownHashError, ValueError)` directly turned a plausible
guess into a checked fact before I put it in the "Expected vs actual"
line. What I'd still add if I revised this: a `pd.show_versions()`-style
full dependency dump, since I only listed the three packages the bug
path touches, not the exact resolved versions of everything `pip install`
pulled in.

## Eval iteration

Tool graded: `skill/rubric.md` + `skill/references/evidence-guide.md`
(SKILL.md unmodified). Full run recorded in
[`eval/eval-run.txt`](eval/eval-run.txt), model `sonnet` (pinned),
component fingerprints in that file's header.

### Run history

Two runs total, not one. Before spending the ~$4 full run, I spent
~$0.80 on a targeted `--only pkg-20,pkg-16,pkg-09,pkg-19
--include-calibration` probe, picking the four packages my rubric's
riskiest design choices lived or died on: the one-item `disclosure`
category, an unacknowledged-version-delta `wrong-target` package, an
honest cannot-reproduce `clear-accept`, and the claim-only-boilerplate
`unfollowable-comms` package. That probe agreed 4/4, and — more
important than the score — each rejection traced to exactly the check
I'd built it for (pkg-20 failed only `ai-policy-satisfied`, pkg-19
failed only `claim-specific-and-honest`). That let me commit to a full
run instead of guessing. The confirming full run
(`--save-run eval-run.txt --out results.json`) agreed 20/20 on the
first attempt, category floor clean, no revise loop needed:

```
categories: clear-accept 8/8  disclosure 1/1  no-evidence 4/4
            unfollowable-comms 3/3  wrong-target 4/4
agreement: 20/20 scored items  (bar: 18/20: PASS)
```

### Package analysis

The packages that shaped the rubric most were the ones gold labels
`accept` for reasons a naive "did it work" check would miss:

- **pkg-09 and pkg-10** are honest cannot-reproduces. If a check like
  "artifact shows the issue's behavior" is read literally, both fail —
  neither artifact shows the bug. Gold accepts them because they tried
  the real trigger, showed what actually happened, and named what
  differed from the reporter's setup. That forced `behavior-matches-issue`
  and `outcome-honest` to carry an explicit second limb for this shape,
  not just "matches the issue."
- **pkg-19** splits the claim from the report: the repro report is fine,
  but the claim comment is "kindly assign it to me, guaranteed fix in 2
  days." Gold rejects the whole package on the claim alone. That's why
  `claim-specific-and-honest` is `required` and independent of every
  report-side check — a package can fail on comms alone even with a
  perfect reproduction underneath it.
- **pkg-20** is the one-item `disclosure` category the pass bar's
  category floor exists to force. Ghostty's policy requires disclosing
  *all* AI usage; the candidate comments don't. A rubric that only checks
  "does the comment look AI-written" would pass this, since the comment
  reads as ordinary prose — the check has to assume AI assistance and
  ask whether it was disclosed, not detect AI style.
- **pkg-02, pkg-08, pkg-16, pkg-17** are the wrong-target set: each has a
  clean, confident artifact that is quietly not the issue's behavior (a
  graceful arg error instead of a crash, a compile error from a changed
  expression, an old pandas version with no acknowledgment, garbled
  escape text sold as a crash). These are why `behavior-matches-issue`
  reads the artifact against the issue's exact trigger and signal, not
  against whether *something* went wrong.

### Check rationale

Each required check maps to one proof family the lecture named, plus
one check the eval set specifically forces:

- `env-recorded` / `env-faithful` — an environment can be present but
  still off-target (pkg-16 ran pandas 1.5.3 against an issue confirmed
  on latest/main and never said so); splitting "is there a record" from
  "is the record honest about the gap" catches that pkg-16 passes the
  first and fails the second.
- `steps-rerunnable` — pkg-18's reproduction lives in an unshareable
  private monorepo; no artifact problem, no honesty problem, just
  nothing a stranger can rerun. This needed its own check because
  every other check would have passed it.
- `artifact-present` / `behavior-matches-issue` — separated because
  pkg-13/14/15 fail on the first (assertion, no artifact) while
  pkg-02/08/16/17 pass the first and fail the second (artifact
  present, wrong artifact).
- `outcome-honest` — reads the certainty words against the shown
  evidence's actual scope; this is what catches pkg-17's "confirms on
  the Store release, which demonstrates it's not limited to git-main"
  when a maintainer already couldn't reproduce on that release.
- `claim-specific-and-honest` — reads only the claim comment, so a
  strong report (pkg-19) can't compensate for boilerplate or a
  guaranteed-delivery-date claim.
- `ai-policy-satisfied` — the category-floor check; built around the
  "assume AI-assisted" standing assumption described above, since a
  policy requiring disclosure of "all AI usage" is unenforceable if the
  grader first has to decide whether AI was used from the prose alone.
- `control-isolates-trigger` / `template-asks-covered` — `preferred`,
  because they measure report quality, not readiness: a control run
  makes an artifact more convincing, and template coverage is a
  checklist, but neither is why a package should be blocked.

### Trade-offs

- **Strict `unclear` = fail, no partial credit.** The verdict rule
  treats any required check graded `unclear` the same as `fail`. This
  is deliberately unforgiving — proof I can't verify is proof that
  isn't ready to post — but it means a genuinely ambiguous case with no
  clean read gets rejected by default rather than flagged for human
  judgment. Given the eval set's binary verdict space, I chose
  precision (predictable results) over graceful ambiguity handling.
- **The AI-disclosure assumption cuts both ways.** Treating every
  candidate comment as AI-assisted is what makes `ai-policy-satisfied`
  gradeable at all, and it's what pkg-20 needs. But it also means a
  repo with a strict disclosure policy and a candidate who, on the
  actual issue, genuinely wrote every word by hand would still fail the
  check for not disclosing something that didn't happen. I accepted
  that false-positive risk because the alternative (trusting the prose
  to reveal its own provenance) fails the one package the category
  floor exists to catch.
- **`required` checks gate the verdict uniformly; they aren't
  weighted.** A package that fails only `claim-specific-and-honest`
  (pkg-19, a strong report behind a bad claim) rejects exactly as hard
  as a package that fails five required checks at once (pkg-13). That
  loses a useful distinction — "close, one fixable problem" vs.
  "nothing here" — that the `note` column in the eval output partly
  recovers, but the JSON verdict itself does not.
- **Reading the environment/steps families separately from behavior
  costs verbosity for a small number of extra passes.** `env-faithful`
  and `steps-rerunnable` overlap in spirit with `behavior-matches-issue`
  (all three are versions of "is this actually about the same thing"),
  and a leaner rubric could probably merge them. I kept them apart
  because they fail independently in the set (pkg-18 only on steps,
  pkg-16 only on environment-honesty), and merging them would have
  hidden which one failed in the `note` column.
