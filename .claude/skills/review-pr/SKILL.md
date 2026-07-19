---
name: review-pr
description: "Run a full PR review in one command. Invoke explicitly with /review-pr <PR number or URL>. Fetches the PR diff, delegates to the style, security, and architecture reviewers, consolidates findings, presents them, and asks for confirmation before posting anything to GitHub."
disable-model-invocation: true
---

# /review-pr

Run the complete PR review workflow. Follow these steps exactly and in order. Do not
skip the confirmation gate, and never post to GitHub without an explicit `yes` or `post`.

## Steps

1. **Fetch the PR via GitHub MCP.** Read the PR number or URL from the invocation, then
   call the GitHub MCP to fetch the PR metadata and the full diff (changed files + hunks).
   Do not proceed without the diff. If no PR is given, ask which PR to review and stop
   until answered. If the GitHub MCP is unavailable or the fetch fails, stop and report
   it — never review from assumed or pasted content unless the user explicitly provides it.

2. **Route files by type.** From the diff, collect the changed `.swift` files and the
   changed `.kt` files. Only consider lines added or modified by the diff — never
   comment on unchanged code or files outside the diff.

3. **Delegate the review** to the three sub-agents, passing each the relevant diff:
   - `style-reviewer` — applies `swift-conventions` to `.swift` files and
     `kotlin-conventions` to `.kt` files (naming, concurrency/coroutines, framework use).
   - `security-reviewer` — hardcoded credentials/secrets, missing input validation,
     insecure storage (`UserDefaults`/`SharedPreferences`/plain text), disabled SSL/TLS.
   - `architecture-reviewer` — loads `architecture-guidelines`; confirm each finding
     against `docs/adr/` and cite the ADR by name (ADR-001 MVVM, ADR-002 async/await
     networking, ADR-003 repository pattern).

4. **Consolidate findings** into the CLAUDE.md output format. Group by file; one line per
   finding: `[SEVERITY] <short description> — <the rule or ADR it breaks>`. Use the
   severity guide: HIGH = hardcoded credentials / disabled SSL; MEDIUM = unencrypted
   sensitive storage, missing validation, architecture violations; LOW = style. If a
   file has no issues, write `No issues found.` under its heading.

5. **Add the summary.** Close with a `## Summary`: total findings by severity
   (e.g. `2 HIGH, 3 MEDIUM, 1 LOW`), the single most important issue, and a one-line
   overall verdict. Do not phrase the verdict as an approval.

6. **Present and gate.** Show the full review in chat. Do NOT post yet. Ask the user to
   confirm before posting to GitHub.

7. **Post only on confirmation.** If — and only if — the user replies `yes` or `post`,
   use the GitHub MCP to post the review as comments. On any other reply, do not post.

8. **Log it.** After a completed review, append one line to `review_history.md`:
   `YYYY-MM-DD | PR-[number] | [top finding] | [severity]`.

## Guardrails

- Never approve, set a review state, or merge the PR. Never edit source files.
- Maximum **3** review passes per PR. If findings remain unresolved after 3 passes,
  stop and hand back to a human reviewer.
- Stay in scope: comment only on the provided diff.
