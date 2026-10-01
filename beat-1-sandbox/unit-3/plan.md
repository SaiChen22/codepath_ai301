# plan.md

## Diagnosis
In `core/security.py`, `verify_password(plain_password: str, hashed_password: str) -> bool` directly returns `bool(pwd_context.verify(plain_password, hashed_password))` without guarding against exceptions raised during hash parsing or identification.

When `pwd_context.verify` receives an unrecognized or malformed stored hash, passlib does not return `False`:
1. If the hash format cannot be identified (e.g., `'not_a_valid_bcrypt_hash'`, `''`, or `'plaintext'`), `passlib.context.CryptContext._identify_record` raises `passlib.exc.UnknownHashError: hash could not be identified`.
2. If the hash has a bcrypt prefix but malformed internal structure (e.g., `'$2b$notarealhash'`), passlib's parser raises `builtins.ValueError: not enough values to unpack (expected 2, got 1)`.

Because `verify_password` lacks an exception handler around `pwd_context.verify`, these exceptions propagate directly to upstream callers rather than failing closed. As verified via `issubclass(UnknownHashError, ValueError)`, `passlib.exc.UnknownHashError` inherits from `builtins.ValueError`. Therefore, catching `ValueError` inside `verify_password` addresses both failure shapes at the point of origin.

## Scope
- **In-scope**:
  - Catch `ValueError` (which covers `passlib.exc.UnknownHashError`) during `pwd_context.verify(plain_password, hashed_password)` in `core/security.py` and return `False`.
  - Remove the `@pytest.mark.xfail(strict=True, reason="issue #72 (manifest H-05): ...")` marker from `test_verify_with_wrong_hash_format` in `tests/unit/test_security.py`.
  - Expand `test_verify_with_wrong_hash_format` to assert that malformed bcrypt-prefixed strings (such as `'$2b$notarealhash'`), empty strings, and arbitrary plain text also return `False`.
- **Out-of-scope (Non-goals)**:
  - Modifying `hash_password`, JWT generation/decoding functions, or authentication dependency routes (`api/` or `core/auth.py`).
  - Upgrading or changing pinned dependencies in `pyproject.toml` (e.g., passlib or bcrypt versions).
  - Silencing unrelated upstream runtime deprecation warnings (such as the passlib bcrypt version probe or Pydantic V2 ConfigDict warning).

## Files to touch
- `core/security.py`
- `tests/unit/test_security.py`

## Approach
1. In `core/security.py`:
   - Locate `verify_password` at line 37:
     ```python
     def verify_password(plain_password: str, hashed_password: str) -> bool:
         try:
             return bool(pwd_context.verify(plain_password, hashed_password))
         except ValueError:
             return False
     ```
   - Wrapping the call with `except ValueError:` ensures `UnknownHashError` and malformed hash `ValueError` cases both fail closed and return `False`, while allowing unrelated system exceptions to propagate.
2. In `tests/unit/test_security.py`:
   - Remove the `@pytest.mark.xfail` decorator above `test_verify_with_wrong_hash_format`.
   - Update `test_verify_with_wrong_hash_format` to verify that `verify_password("password", bad_hash)` evaluates to `False` across representative invalid shapes: `"not_a_valid_bcrypt_hash"`, `""`, `"plaintext"`, and `"$2b$notarealhash"`.

## Test Plan
1. **Targeted covering test**:
   - Run:
     ```bash
     pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v
     ```
   - Pre-fix output: `XFAIL` (or `FAILED` under `--runxfail` with `passlib.exc.UnknownHashError: hash could not be identified`).
   - Expected post-fix output: `1 passed` with exit code `0` (no longer `XFAIL`).
2. **Direct CLI verification against all repro shapes**:
   - Run:
     ```bash
     python3 -c "
     from core.security import verify_password, hash_password
     h = hash_password('password')
     print('control valid:', verify_password('password', h))
     print('control wrong:', verify_password('wrong', h))
     cases = ['', 'plaintext', '\$2b\$notarealhash', 'not_a_valid_bcrypt_hash']
     for bad in cases:
         print(repr(bad), '->', verify_password('password', bad))
     "
     ```
   - Expected post-fix output:
     ```text
     control valid: True
     control wrong: False
     '' -> False
     'plaintext' -> False
     '$2b$notarealhash' -> False
     'not_a_valid_bcrypt_hash' -> False
     ```
3. **Full security suite regression check**:
   - Run:
     ```bash
     pytest tests/unit/test_security.py -v
     ```
   - Expected post-fix output: `25 passed` (all unit security tests pass without regressions).

## Risks and Unknowns
- **Broad exception scope risk**: Catching `ValueError` is safe because `pwd_context.verify` uses `ValueError` exclusively for schema/format parsing failures. It does not mask unhandled `TypeError` (e.g., non-string inputs) or environmental exceptions.
- **Timing side-channel**: While a malformed hash returns immediately rather than performing full bcrypt rounds, an unparseable stored hash represents corrupted or invalid data where failing closed is standard practice.

## Deviations
- **Docstring `Returns` section updated** (not in the original Approach). `docs/CONTRIBUTING.md` requires Google-style docstrings on all public functions, and the fail-closed-on-malformed-hash behavior is part of the function's contract now, so leaving the old "True if password matches, False otherwise" would have been stale. No behavior change.
- **Commit scope is `fix(api)`**, not `fix(security)`. `docs/CONTRIBUTING.md` lists the allowed scopes as `ingestion`, `rag`, `agent`, `safety`, `api`, `frontend` — `security` is not among them, and issue #72 carries the `api` label.
- **Test kept as one test function with a loop** rather than `pytest.mark.parametrize`. The file uses no `parametrize` anywhere, and parametrizing would have changed the module test count away from the 25 the Test Plan asserts.
