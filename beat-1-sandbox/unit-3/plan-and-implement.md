# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**GitHub username**

SaiChen22

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72#issuecomment-3356064974

I have reproduced the issue and put together a plan to ensure `verify_password` fails closed on malformed hashes:

### Diagnosis & Root Cause
In `core/security.py:37`, `verify_password` directly invokes `pwd_context.verify(plain_password, hashed_password)` without exception handling. When `hashed_password` is unparseable:
- Strings without a recognizable format (`'not_a_valid_bcrypt_hash'`, `''`, `'plaintext'`) raise `passlib.exc.UnknownHashError`.
- Structurally invalid bcrypt-prefixed strings (`'$2b$notarealhash'`) raise `builtins.ValueError`.

Since `UnknownHashError` subclasses `ValueError`, catching `ValueError` handles both error modes cleanly at the function boundary.

### Proposed Changes & Scope
- **`core/security.py`**: Wrap `pwd_context.verify` in a `try/except ValueError:` block that returns `False`.
- **`tests/unit/test_security.py`**: Drop the `@pytest.mark.xfail` marker from `test_verify_with_wrong_hash_format` (manifest H-05) and assert that malformed hashes return `False`.
- Scope is strictly bounded to these two files; no changes to hashing functions, auth routes, or dependency versions. Note that PR #75 is currently open on this thread; per course guidelines, I am implementing and validating this on an independent branch (`fix/72-verify-password-fail-closed`) grounded in my own repro findings.

### Test Plan
1. Re-run `pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v` (expect `PASSED` instead of `XFAIL`).
2. Run the direct multi-shape verification script (`''`, `'plaintext'`, `'$2b$notarealhash'`, `'not_a_valid_bcrypt_hash'`) and assert all return `False` while valid password controls remain `True`/`False`.
3. Run the full security module suite (`pytest tests/unit/test_security.py -v`) to ensure all 25 tests pass.

Next, I will branch to `fix/72-verify-password-fail-closed`, apply the change, and run the regression tests.

---

## Your branch

**Branch**

`fix/72-verify-password-fail-closed`

**Evidence**

### Before

```text
$ pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v --runxfail
============================== test session starts ===============================
platform darwin -- Python 3.14.6, pytest-9.1.1, pluggy-1.6.0 -- /Users/sai/MyFolder/learning/CodePath/AI301/week2/pathreview-ai301-fa26-s1/.venv/bin/python3.14
cachedir: .pytest_cache
rootdir: /Users/sai/MyFolder/learning/CodePath/AI301/week2/pathreview-ai301-fa26-s1
configfile: pyproject.toml
collected 25 items / 24 deselected / 1 selected                                  

tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format FAILED [100%]

==================================== FAILURES ====================================
________________ TestSecurity.test_verify_with_wrong_hash_format _________________

self = <tests.unit.test_security.TestSecurity object at 0x10b6323d0>

    @pytest.mark.xfail(
        strict=True,
        reason="issue #72 (manifest H-05): password verify raises UnknownHashError instead of returning False",
    )
    def test_verify_with_wrong_hash_format(self):
        """Test verify_password with non-bcrypt hash."""
        wrong_hash = "not_a_valid_bcrypt_hash"
    
        # Should handle gracefully, return False
>       result = verify_password("password", wrong_hash)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

tests/unit/test_security.py:227: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
core/security.py:37: in verify_password
    return bool(pwd_context.verify(plain_password, hashed_password))
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.venv/lib/python3.14/site-packages/passlib/context.py:2343: in verify
    record = self._get_or_identify_record(hash, scheme, category)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.venv/lib/python3.14/site-packages/passlib/context.py:2031: in _get_or_identify_record
    return self._identify_record(hash, category)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <passlib.context._CryptConfig object at 0x10b52dd30>
hash = 'not_a_valid_bcrypt_hash', category = None, required = True

    def identify_record(self, hash, category, required=True):
        """internal helper to identify appropriate custom handler for hash"""
        if not isinstance(hash, unicode_or_bytes_types):
            raise ExpectedStringError(hash, "hash")
        for record in self._get_record_list(category):
            if record.identify(hash):
                return record
        if not required:
            return None
        elif not self.schemes:
            raise KeyError("no crypt algorithms supported")
        else:
>           raise exc.UnknownHashError("hash could not be identified")

E           passlib.exc.UnknownHashError: hash could not be identified

.venv/lib/python3.14/site-packages/passlib/context.py:1132: UnknownHashError
================================ warnings summary ================================
core/config.py:7
  /Users/sai/MyFolder/learning/CodePath/AI301/week2/pathreview-ai301-fa26-s1/core/config.py:7: PydanticDeprecatedSince20: Support for class-based `config` is deprecated, use ConfigDict instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See Pydantic V2 Migration Guide at [https://errors.pydantic.dev/2.13/migration/](https://errors.pydantic.dev/2.13/migration/)
    class Settings(BaseSettings):

-- Docs: [https://docs.pytest.org/en/stable/how-to/capture-warnings.html](https://docs.pytest.org/en/stable/how-to/capture-warnings.html)
============================ short test summary info =============================
FAILED tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format - passlib.exc.UnknownHashError: hash could not be identified
================== 1 failed, 24 deselected, 1 warning in 0.17s ===================
```

### After

```text
$ pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v --runxfail
============================== test session starts ===============================
platform darwin -- Python 3.14.6, pytest-9.1.1, pluggy-1.6.0 -- /Users/sai/MyFolder/learning/CodePath/AI301/week2/pathreview-ai301-fa26-s1/.venv/bin/python3.14
cachedir: .pytest_cache
rootdir: /Users/sai/MyFolder/learning/CodePath/AI301/week2/pathreview-ai301-fa26-s1
configfile: pyproject.toml
collected 25 items / 24 deselected / 1 selected                                  

tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format PASSED [100%]

================================ warnings summary ================================
core/config.py:7
  /Users/sai/MyFolder/learning/CodePath/AI301/week2/pathreview-ai301-fa26-s1/core/config.py:7: PydanticDeprecatedSince20: Support for class-based `config` is deprecated, use ConfigDict instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See Pydantic V2 Migration Guide at [https://errors.pydantic.dev/2.13/migration/](https://errors.pydantic.dev/2.13/migration/)
    class Settings(BaseSettings):

-- Docs: [https://docs.pytest.org/en/stable/how-to/capture-warnings.html](https://docs.pytest.org/en/stable/how-to/capture-warnings.html)
================== 1 passed, 24 deselected, 1 warning in 0.16s ===================
```

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these fields.

## Run history

- Run 1: agreement: 19/20 scored items (bar: 18/20: PASS)
- Categories: clear-accept 6/7, scope-creep 4/4, thread-convention 2/2, unbuildable 3/3, wrong-cause 4/4

## Package analysis

- Package: pkg-14
- Category: clear-accept
- Gold verdict: accept
- Skill verdict: reject

### Reasoning

In pkg-14, the skill marked executable-approach as fail because the plan's approach relied on high-level pseudocode outlining the algorithm rather than concrete AST/file-level modifications and exact function names. Because my rubric specifies under executable-approach that an unfamiliar engineer must be able to "begin implementation immediately without guessing missing core details", the skill took a strict stance and treated the pseudocode as insufficient implementation detail. However, the gold label graded this as accept because the logic changes and touched boundaries were bounded enough in context for an engineer to infer the implementation.

## Check rationale

### Exact quote from rubric.md

```text
| bounded-scope | The plan's scope section (in-scope and explicit out-of-scope boundaries) read against the touched file list | The change touches only the minimal files necessary to fix the root cause, explicitly states what will NOT be done, and introduces no unrelated refactoring, cosmetic rewrites, or scope creep. Scoped-down deferrals are acceptable if explicitly stated. | required |
```

### Why it reads this way

I specifically drafted this check with the clause "Scoped-down deferrals are acceptable if explicitly stated" to prevent rejecting valid plans that deliberately choose to fix one part of a multi-faceted problem while deferring secondary issues. Without that explicit qualification, standard strict scope checks frequently misclassify well-scoped partial fixes as incomplete or unbounded.

## Trade-offs

By strictly requiring that the plan "explicitly states what will NOT be done", this check risks rejecting concise bug fixes that touch only the exact single line needing repair simply because the author did not add a verbose "Non-goals" section. However, accepting this trade-off ensures that ambitious PRs attempting drive-by refactorings or cosmetic cleanups (such as pkg-06, pkg-12, pkg-15, and pkg-19, which all correctly evaluated to reject with 4/4 agreement) are systematically caught before code review.