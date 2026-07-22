## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/146

**Issue title:** PII scrubber fails to redact parenthesized US phone numbers

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The PII scrubber in `safety/pii_scrubber.py` uses a regex pattern to detect and redact US phone numbers before they reach the AI review pipeline. The current pattern only matches dashed formats like `555-123-4567`, so parenthesized formats like `(555) 123-4567` slip through both `scrub()` and `detect()` completely unredacted. This is a real safety gap since parenthesized formatting is one of the most common ways people write phone numbers, meaning actual PII can leak into logs, prompts, or stored review data without being caught. A successful fix updates the phone-number regex to also match the parenthesized format, redacts it consistently with the dashed format, and passes the four existing failing tests in `tests/unit/test_pii_scrubber.py` (`test_us_phone_number_redaction`, `test_us_phone_formats`, `test_detect_phone_pii`, `test_phone_at_start_of_text`).

**Branch name:** fix/146-pii-scrubber-parenthesized-phone

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger
