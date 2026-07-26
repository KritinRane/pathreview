## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/153

**Issue title:** Faithfulness checker crashes when a context chunk has text:None

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
`FaithfulnessChecker.check()` in `rag/evaluator/faithfulness_checker.py` scores generated
feedback by comparing it against the retrieved context chunks. To build the comparison text,
it does `chunk.get("text", "")` for each chunk, which only falls back to `""` when the `"text"`
key is missing entirely — if a chunk has `"text": None` (e.g. from a source that failed to
extract text during ingestion), `.get()` returns `None` instead of the default, and the
subsequent `" ".join(...)` raises a `TypeError` because it can't join a `None` into a string.
This crashes the entire evaluation run instead of just skipping or scoring around the bad
chunk. A successful fix replaces the `.get("text", "")` call with logic that coerces a `None`
value to an empty string (e.g. `chunk.get("text") or ""`), so the checker degrades gracefully
on malformed chunks instead of raising, and adds a regression test covering a chunk with an
explicit `None` text value.

**Scope check ("Is this right for me?"):**
This is a good first issue: it's isolated to one small method in one file with no cross-module
changes, the bug is fully understood (root cause is the `.get()` default-value gotcha, not a
mystery), it doesn't touch auth, migrations, or infra, and it's easy to verify with a unit test
that passes a chunk with `text: None` before/after the fix. It's scoped small enough to finish
well within Tier 1 expectations while still touching real application code (not just docs/tests).

**Branch name:** fix/153-faithfulness-checker-none-context

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/KritinRane/pathreview/commit/b30477f7ac6ef4e2f8f6b8fefc9296c54b37edf3

**Reproduction summary:**
Ran `.venv/bin/python scripts/repro_issue_153.py` (and the existing unit test
`tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_none_context_chunk_text`),
passing a context chunk of `{"text": None}` to `FaithfulnessChecker.check()`.
Both crash with `TypeError: sequence item 0: expected str instance, NoneType found`
at the `" ".join([chunk.get("text", "") ...])` line in
`rag/evaluator/faithfulness_checker.py`, confirming the `.get(..., "")` default
does not cover an explicit `None` value.

**PLAN.md link:** https://github.com/KritinRane/pathreview/blob/fix/153-faithfulness-checker-none-context/PLAN.md

**Walkthrough video (recommended):**

**Blockers or open questions:**
Need to confirm no other module builds context with the same `.get("text", "")`
pattern, and decide whether to coerce truthy non-string `text` values (likely
out of scope for this issue).
