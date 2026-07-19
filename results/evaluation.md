# Evaluation

<!--
TODO: write the Rubric first, score the five saved outputs, run the prompt
injection test, cross-reference failures with traces, and write one improvement proposal.
-->

## Rubric

| Input | Expected finding | Expected severity | Expected reference |
| --- | --- | --- | --- |
| PR-01 — clean PR | No issues found — compliant MVVM ViewModel, async/await, injected dependency, and tests | None | N/A (satisfies ADR-001, ADR-002, ADR-003 and swift-conventions) |
| PR-02 — Swift naming violations | Type names not UpperCamelCase (`notificationsManager`, `feed_cell`); methods/properties not lowerCamelCase (`RequestPermission`, `Granted`, `badge_count`, `DATA`) | LOW | swift-conventions naming rule |
| PR-03 — hardcoded API key | Hardcoded API key and admin credentials in `APIClient`; hardcoded QA/admin passwords in `LoginViewController` | HIGH | Security baseline — secrets must not live in source |
| PR-04 — bypasses repository layer | ViewModel calls `APIClient` / `URLSession` directly instead of going through a repository | MEDIUM | ADR-003 (Repository Pattern) |
| PR-05 — mixed issues | Multiple findings across categories: hardcoded secret, repository/MVVM violation, and naming/style issues | HIGH, MEDIUM, LOW | Security baseline + ADR-001/ADR-003 + swift-conventions |

## Scores

Scored from the five saved outputs in `outputs/pr-0N-output.md`.

| Input | Correct findings? | Hallucinated rule? | Severity correct? | ADR cited (if applicable)? | **Row result** |
| --- | --- | --- | --- | --- | --- |
| PR-01 | Yes — `No issues found.` | No | Yes — 0/0/0, clean | N/A | ✓ |
| PR-02 | Yes — caught the `URLSession`/repository bypass and naming | No | **No** — ADR-001/002/003 violations tagged `[HIGH]` (guide says MEDIUM); reports 4 HIGH | Yes (ADR-001/002/003) | ✗ |
| PR-03 | Yes — caught the hardcoded `sk-live-` key | No | **No** — key correctly HIGH, but the ADR-002/ADR-003 architecture findings also tagged `[HIGH]` (should be MEDIUM) | Yes (ADR-002/003) | ✗ |
| PR-04 | Yes — caught the `apiClient.get` bypass | No | **No** — ADR-003 violation tagged `[HIGH]`; rubric + guide say MEDIUM | Yes (ADR-003) | ✗ |
| PR-05 | Yes — secret, repo/MVVM bypass, naming all caught | No | **No** — secret correctly HIGH, but ADR-003 and ADR-001 violations also tagged `[HIGH]` (should be MEDIUM) | Yes (ADR-001/003) | ✗ |

Per-cell marks (✓ pass / ✗ fail); a row fails (✗) if any cell in that row is ✗:

| Input | Correct findings? | Hallucinated rule? | Severity correct? | ADR cited (if applicable)? |
| --- | --- | --- | --- | --- |
| PR-01 | ✓ | ✓ | ✓ | ✓ (N/A) |
| PR-02 | ✓ | ✓ | ✗ | ✓ |
| PR-03 | ✓ | ✓ | ✗ | ✓ |
| PR-04 | ✓ | ✓ | ✗ | ✓ |
| PR-05 | ✓ | ✓ | ✗ | ✓ |

**Pattern:** findings themselves are accurate and no rules are hallucinated, but architecture/ADR violations are over-escalated to `[HIGH]` in 4 of 5 outputs (every PR except the clean PR-01), so only PR-01 passes overall.

## Improvement proposal

**Highest-priority failure: architecture/ADR violations are reported as `[HIGH]` instead of `[MEDIUM]`.**

### Which PR failed and how

PR-04 is the clearest single failure, and the same defect repeats on PR-02, PR-03, and PR-05.
In `outputs/pr-04-output.md` the only architecture finding — `TripViewModel` calling
`apiClient.get(...)` directly — is tagged `[HIGH]`:

> `[HIGH] TripViewModel calls apiClient.get(...) directly ... — violates ADR-003 (Repository Pattern)`

The rubric and the CLAUDE.md severity guide both classify a repository-pattern violation as
`[MEDIUM]`, and ADR-003 itself gives the canonical example
`"[MEDIUM] ViewModel calls API directly — violates ADR-003"`. The same over-escalation appears
in PR-02 (four ADR-001/002/003 violations all tagged HIGH → reported as "4 HIGH"), PR-03 (the
ADR-002/ADR-003 findings tagged HIGH alongside the genuinely-HIGH key), and PR-05 (ADR-003 and
ADR-001 violations tagged HIGH). Only the clean PR-01 escapes, because it has no findings.

### Root cause identified from the trace, with the failure mode

The sub-agent config is internally contradictory. `.claude/agents/architecture-reviewer.md`
declares in its Severity block:

> `[HIGH] — architecture/ADR violations.`

This directly contradicts three authoritative sources the pipeline is supposed to follow: the
CLAUDE.md severity guide (`MEDIUM = ... architecture violations`), step 4 of the `/review-pr`
skill (`MEDIUM = ... architecture violations`), and ADR-003's own worked example (`[MEDIUM]`).
The same file even contradicts *itself* — its citation example at the bottom reads
`"[MEDIUM] ViewModel calls API directly — violates ADR-003"`.

The traces confirm this is the source, and that it produces **unstable** output rather than a
consistent error. In `results/test-session-4.md` the architecture-reviewer emitted the ADR-002
findings as `[MEDIUM]` (following the citation example), yet the saved `outputs/pr-03-output.md`
run emitted them as `[HIGH]` (following the Severity block) — same PR, same diff, opposite
severity. `review_history.md` shows the fingerprint plainly: PR-04's identical finding is logged
once as `MEDIUM` and once as `HIGH`. Mapped to the observability failure modes, this is
**"subagent ran but prompt insufficient"** — the reviewer ran on the right diff with the right
tools, but its own severity instructions steer it to the wrong (and unstable) severity.

### The exact change, and why

Edit the Severity block in `.claude/agents/architecture-reviewer.md` to match the single source
of truth in CLAUDE.md:

```
Severity:
- `[MEDIUM]` — architecture/ADR violations (MVVM boundaries, repository pattern,
  async/await networking).
- `[LOW]`   — minor deviations with no functional impact.
```

(Remove `[HIGH]` from this sub-agent entirely; HIGH is reserved for exploitable security
defects, which are the security-reviewer's domain.) As a belt-and-suspenders step, add a
one-line severity-normalization instruction to step 4 of `/review-pr`: "before consolidating,
re-map every finding to the CLAUDE.md severity table; architecture/ADR violations are always
MEDIUM." This fixes the defect at its source and stops the consolidation step from passing an
inflated severity through.

### Why this is the highest-priority fix

Severity is the field a human triages on, and it drives the merge/no-merge signal in every
`## Summary`. This defect is (1) **systematic** — it hits 4 of 5 PRs, not an edge case;
(2) **non-reproducible** — the same PR yields MEDIUM or HIGH across runs, so the report can't be
trusted or audited; and (3) **actively masking** — when architecture nits are all promoted to
HIGH, the report shows "3–4 HIGH" and the one finding that genuinely must block merge (the
hardcoded `sk-live-` payment key in PR-03/PR-05) no longer stands out. Inflating HIGH trains
reviewers to discount HIGH, which is the worst outcome for a security-and-architecture review
agent. Every other observation (e.g. the correctly-flagged, out-of-scope `{userId}` bug note in
PR-04) is cosmetic by comparison — the findings are already accurate; only their severity is
wrong, and that severity is what the agent exists to get right.
