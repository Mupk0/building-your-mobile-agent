# Observability Table

Diagnostic log of PR-review-agent traces: what sub-agents were invoked, what they
returned, and — when a finding is missing from the final consolidated output —
which failure mode caused it.

Failure modes:
- **subagent not called** — the orchestrator never invoked the sub-agent for this pass.
- **wrong input** — the sub-agent was called, but with the wrong diff/file/context.
- **tool never fired** — the sub-agent ran but a tool call it needed (e.g. a skill load, a file read) never happened.
- **subagent ran but prompt insufficient** — the sub-agent ran with the right input and tools, but the prompt didn't direct it to surface the issue.
- **N/A** — no finding is missing; the pipeline worked as expected.

| Input | Total tokens | Most expensive span | `security-reviewer` called? |
| --- | --- | --- | --- |
| PR-01 | 284,684 | architecture-reviewer | Y |
| PR-02 | 501,211 | architecture-reviewer | Y |
| PR-03 | 486,052 | architecture-reviewer | Y |
| PR-04 | 661,623 | architecture-reviewer | Y |
| PR-05 | 635,974 | architecture-reviewer | Y |

(PR-01 = clean, PR-02 = style violations, PR-03 = hardcoded API key, PR-04 = architecture violation, PR-05 = mixed issues, 2 files. Per-sub-agent token breakdown and tool-call counts are in the "Token/span diagnostics by PR" section below.)

## Token/span diagnostics by PR

Read directly from each sub-agent's trace file for this run (session dir
`70a12d68-1807-47fd-8221-722e9e84070d/tasks/`). "Total tokens" sums
input + cache-read + cache-creation + output tokens across all turns in
that sub-agent's transcript. "Most expensive span" = the sub-agent whose
transcript consumed the most total tokens for that PR.

Notes:
- **security-reviewer was called on every PR** and in every case ran with
  **0 tool calls** — it answered entirely from the diff given in its prompt
  (Read/Grep/Glob were available but unneeded since the full diff was inline).
  This matches its `security-reviewer` agent definition (Tools: Read, Grep, Glob).
- **architecture-reviewer is the most expensive span on every single PR**,
  by a wide margin (roughly 4x style-reviewer, 12x security-reviewer). It
  consistently uses 5 tool calls: loading the `architecture-guidelines` skill
  plus reading all three `docs/adr/*.md` files, then reasoning over them —
  this file-reading + skill-load overhead, not longer output, is what drives
  the token cost.
- PR-05 is the most expensive PR overall because it has two files (one
  Kotlin, one Swift), pushing style-reviewer's cost up (35,846 tokens,
  2 tool calls) since it had to apply both `swift-conventions` and
  `kotlin-conventions`.

## Full span output — PR-03 security-reviewer

```
### PaymentService.kt

[HIGH] Hardcoded live-format API key `sk-live-9f8a7b6c5d4e3f2a1b0c9d8e7f6a5b4c` stored as a public `val` in `PaymentService` — secrets must not live in source
[MEDIUM] `charge(amount: Int, token: String)` performs no validation that `amount` is positive/non-zero before initiating a charge — missing input validation on user-supplied data
[MEDIUM] `charge(amount: Int, token: String)` performs no validation of the `token` (payment/card token) format or presence before use in the charge request — missing input validation on user-supplied data

No SSL/TLS issues found — the Retrofit `baseUrl` uses `https://` and no certificate/hostname validation is disabled or bypassed in this diff.
```

**Diagnostic answers (PR-03):**
- Was security-reviewer called? **Y** — confirmed directly from trace file `70a12d68-1807-47fd-8221-722e9e84070d/tasks/afb87084bcb332b28.output` (2 turns, agent ran to completion).
- Span output: pasted above — `[HIGH]` hardcoded API key finding plus two `[MEDIUM]` missing-validation findings.
- A Failure mode: **tool never fired**.

