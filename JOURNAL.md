## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/146

**Issue title:** PII scrubber fails to redact parenthesized US phone numbers

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The PII scrubber in `safety/pii_scrubber.py` uses a regex pattern to detect and redact US phone numbers before they reach the AI review pipeline. The current pattern only matches dashed formats like `555-123-4567`, so parenthesized formats like `(555) 123-4567` slip through both `scrub()` and `detect()` completely unredacted. This is a real safety gap since parenthesized formatting is one of the most common ways people write phone numbers, meaning actual PII can leak into logs, prompts, or stored review data without being caught. A successful fix updates the phone-number regex to also match the parenthesized format, redacts it consistently with the dashed format, and passes the four existing failing tests in `tests/unit/test_pii_scrubber.py` (`test_us_phone_number_redaction`, `test_us_phone_formats`, `test_detect_phone_pii`, `test_phone_at_start_of_text`).

**Branch name:** fix/146-pii-scrubber-parenthesized-phone

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/satyapandla/pathreview/commit/8f5228e

**Reproduction summary:**
Ran `scrub()` and `detect()` against `(555) 123-4567` and confirmed the parenthesized format passes through unredacted while the dashed format in the same string gets caught. Ran the full test suite and confirmed 5 related tests fail as described in the issue: `test_us_phone_number_redaction`, `test_us_phone_formats`, `test_detect_phone_pii`, `test_phone_at_start_of_text`, plus `test_mixed_pii_and_text` (which fails for a related but separate reason).

**PLAN.md link:** https://github.com/satyapandla/pathreview/blob/fix/146-pii-scrubber-parenthesized-phone/PLAN.md

**Walkthrough video (recommended):**

**Blockers or open questions:**
Need to confirm whether the `street_address` over-matching bug surfaced in `test_mixed_pii_and_text` is in scope for #146 or should be filed separately — it's an unrelated regex issue that happened to trip a test in the same file.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Fixed the `phone_us` regex in `safety/pii_scrubber.py` — added whitespace as an allowed separator character alongside dash/dot, which fixes the parenthesized phone number format from issue #146. Added a regression test for the exact scenario in the issue.

**Next steps:**
Run full lint/typecheck to document pre-existing failures, finalize PR description, submit.

**Blockers:**
None — fix was smaller than expected, one regex change.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/satyapandla/pathreview/pull/1

**Branch:** `fix/146-pii-scrubber-parenthesized-phone`

**What you built:**
Updated the `phone_us` regex pattern in `PIIScrubber` to accept whitespace as a separator after the optional closing parenthesis, in addition to dash and dot. This fixes `scrub()` and `detect()` failing to catch parenthesized US phone numbers like `(555) 123-4567`.

**Tests added or updated:**
Added `test_issue_146_parenthesized_phone_in_context` to `tests/unit/test_pii_scrubber.py`, directly testing the exact repro case from the issue (a sentence with both a parenthesized and a dashed phone number). The 4 previously-failing tests tied to this issue (`test_us_phone_number_redaction`, `test_us_phone_formats`, `test_detect_phone_pii`, `test_phone_at_start_of_text`) now pass.

**Self-review confirmation:** [x] make test-unit passes (25/26 — see note below) [ ] make check passes (not fully clean — see note below)

**Draft PR feedback received from:** none

**Note on pre-existing failures:** `test_mixed_pii_and_text` fails before and after my change — it's caused by the unrelated `street_address` regex over-matching into adjacent text, not by anything I touched. `ruff check .` and `black . --check` show pre-existing issues across ~52 files unrelated to `pii_scrubber.py` (import ordering, line length, old-style `Optional[X]` typing). `mypy` shows 140 pre-existing errors, mostly missing library stubs for third-party packages not installed in this environment. My one changed line in `pii_scrubber.py` does not introduce any new lint, format, or type errors beyond what already existed on that file (pre-existing: unsorted imports, one unused loop variable, two long lines — none on the line I changed).