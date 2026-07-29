\## Solution plan



\*\*Issue:\*\* Parenthesized phone numbers not detected/redacted — \[#146](https://github.com/satyapandla/pathreview/issues/146)



\### Understand

The `phone\_us` regex in `safety/pii\_scrubber.py` requires a dash or dot (`\[-.]?`) as the separator after an optional closing parenthesis (`\\)?`). Real-world parenthesized format, e.g. `(555) 123-4567`, uses a space after the `)`, not a dash or dot — so the pattern never matches from the opening `(`. Expected: `(555) 123-4567` should be detected and redacted like any other US phone format. Actual: it passes through `scrub()` untouched and `detect()` returns an empty list for it.



\### Map

\- `safety/pii\_scrubber.py` — contains the `phone\_us` pattern in `PII\_PATTERNS`; this is the only place the fix needs to happen.

\- `tests/unit/test\_pii\_scrubber.py` — contains the 5 currently-failing tests that define expected behavior (`test\_us\_phone\_number\_redaction`, `test\_us\_phone\_formats`, `test\_detect\_phone\_pii`, `test\_phone\_at\_start\_of\_text`, and `test\_mixed\_pii\_and\_text`, which fails for a related reason — see Risks).



\### Plan

1\. Update the `phone\_us` regex so the separator after `\\)?` allows whitespace as well as dash/dot — likely changing `\[-.]?` to `\[-.\\s]?` in that position.

2\. Re-run `test\_us\_phone\_formats` to confirm all four formats (dashed, parenthesized, dotted, `+1` international-style) now redact correctly.

3\. Re-run the full `test\_pii\_scrubber.py` suite to confirm the fix doesn't break passing tests (especially `test\_detect\_no\_false\_positives` and `test\_scrub\_idempotent`).

4\. Investigate `test\_mixed\_pii\_and\_text` separately — it currently fails because the `street\_address` pattern greedily matches into unrelated text ("...developing Python appl" gets partially redacted). Confirm whether this is in scope for #146 or a separate issue.

5\. Add a regression test specifically for the exact issue repro case (`(555) 123-4567` in a sentence) if one doesn't already exist beyond the four listed.



\### Inputs \& outputs

Input: free-form text strings passed to `scrub()` or `detect()`. Output: `scrub()` returns text with matched PII replaced by `\[REDACTED]`; `detect()` returns a list of dicts with `type`, `value`, `start`, `end` for each match. The fix should not change this interface — only which strings the `phone\_us` pattern matches.



\### Risks \& unknowns

\- Loosening `\[-.]?` to `\[-.\\s]?` could cause the pattern to match across sentence boundaries if not scoped carefully (e.g. accidentally spanning a `)` from unrelated parenthetical text followed by a 3-digit number elsewhere in the sentence). Need to test with parenthetical asides near numbers.

\- Unclear whether `test\_mixed\_pii\_and\_text`'s address-pattern-over-matching bug is part of #146's scope or a separate issue (`street\_address` pattern lacks a proper word boundary before matching "5 years... appl"). Will flag for maintainer clarification if it's not clearly in scope.

\- The `phone\_intl` pattern is separate and already passing (`test\_international\_phone\_redaction`) — need to make sure the `phone\_us` fix doesn't create overlapping/duplicate matches with `phone\_intl` on inputs like `+1 555 123 4567`.



\### Edge cases

\- Phone number at the very start of a string (already covered by `test\_phone\_at\_start\_of\_text`).

\- Phone number immediately followed by punctuation (e.g. `(555) 123-4567.`).

\- Multiple phone numbers in different formats in the same string (`test\_us\_phone\_formats` covers this in isolation, but not combined in one string).

\- Parenthesized area code with no space at all, e.g. `(555)123-4567` — should still match given `\\)?` and separator are both optional/flexible.

