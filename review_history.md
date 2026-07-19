# Review history

<!--
TODO: episodic memory log. After every completed review, append one line:
  YYYY-MM-DD | PR-[number] | [top finding short description] | [severity]
Example: 2026-05-15 | PR-03 | Hardcoded API token in PaymentService | HIGH
-->

2026-07-19 | PR-01 | ProfileViewModel added but never adopted by ProfileView — incomplete MVVM wiring per ADR-001 | MEDIUM
2026-07-19 | PR-02 | ProfileViewModel calls URLSession directly, bypassing APIClient and repository layer | MEDIUM
2026-07-19 | PR-03 | Hardcoded live API token in PaymentService | HIGH
