# PR Review — PR #2: Add profile screen

**Repo:** nebius-academy-templates/mobile-pr-review-practice
**PR:** https://github.com/nebius-academy-templates/mobile-pr-review-practice/pull/2
**Files changed:** `src/ProfileViewModel.swift` (new file, +14)

---

### src/ProfileViewModel.swift
[HIGH] `ProfileViewModel` calls `URLSession.shared.dataTask` directly instead of going through a repository — violates ADR-003 (Repository Pattern)
[HIGH] `FetchProfile` uses a completion-handler closure instead of `async`/`await` — violates ADR-002 (async/await networking)
[HIGH] ViewModel performs networking directly via `URLSession` rather than through `APIClient` — violates ADR-002 ("APIClient is the only type permitted to perform URL requests")
[HIGH] `ProfileViewModel` imports `UIKit` — violates ADR-001 ("ViewModels must not import UIKit")
[MEDIUM] `id` parameter passed directly into `profileURL(id)` with no validation, sanitization, or encoding — missing input validation on user-supplied data
[MEDIUM] `dataTask` completion handler ignores `URLResponse` and `Error`, silently discarding network failures and non-success HTTP status codes — missing input validation on untrusted response data
[MEDIUM] `Profile.init` is called directly on raw `data` with no visible decoding-error handling — missing input validation on server-supplied data
[LOW] `FetchProfile` is UpperCamelCase; should be `fetchProfile` (lowerCamelCase) — swift-conventions naming rule
[LOW] Property `loading` missing a boolean prefix (e.g. `isLoading`) — swift-conventions naming rule

No SSL/TLS bypass, hardcoded credentials, or insecure local storage were found.

## Summary
4 HIGH, 3 MEDIUM, 2 LOW

**Most important issue:** `ProfileViewModel` bypasses the repository pattern and calls `URLSession` directly with a completion-handler API, violating ADR-001, ADR-002, and ADR-003 simultaneously — this is a structural architecture problem, not just a style nit.

**Verdict:** Significant architecture and validation issues found; this diff needs rework before it's mergeable (this is not an approval).
