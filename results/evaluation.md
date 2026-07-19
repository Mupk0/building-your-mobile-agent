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

| Input | Correct findings? | Hallucinated rule? | Severity correct? | ADR cited (if applicable)? |
| --- | --- | --- | --- | --- |
| PR-01 | [PLACEHOLDER] | | | |
| PR-02 | [PLACEHOLDER] | | | |
| PR-03 | [PLACEHOLDER] | | | |
| PR-04 | [PLACEHOLDER] | | | |
| PR-05 | [PLACEHOLDER] | | | |

## Improvement proposal
[PLACEHOLDER]
