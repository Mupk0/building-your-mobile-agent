# PR Review — PR #4: Add trip list screen

**Repo:** nebius-academy-templates/mobile-pr-review-practice
**PR:** https://github.com/nebius-academy-templates/mobile-pr-review-practice/pull/4
**Files changed:** `src/TripViewModel.kt` (new file, +22)

---

### src/TripViewModel.kt
[HIGH] `TripViewModel` calls `apiClient.get(...)` directly instead of going through a repository — violates ADR-003 (Repository Pattern); ADR-003 states "A ViewModel must not import or reference `APIClient`"
[MEDIUM] `userId` parameter is used to build the request path with no validation or sanitization before being passed to `apiClient.get` — missing input validation on user-supplied data

No naming, coroutine-usage, Jetpack/lifecycle, or MVVM-boundary (ADR-001) violations found. `viewModelScope.launch` with a suspending `apiClient.get` call satisfies ADR-002's async requirement (mapped to Kotlin coroutines).

**Note (out of scope, flagged for awareness, not a review finding):** the path string `"/users/{userId}/trips"` contains a literal, unsubstituted `{userId}` placeholder — the `userId` parameter passed into `loadTrips` is never actually interpolated into the request path. This looks like a functional/correctness bug (the endpoint would be called with the literal text `{userId}`), separate from the style/security/architecture criteria this review covers.

## Summary
1 HIGH, 1 MEDIUM, 0 LOW

**Most important issue:** the ViewModel bypasses the repository layer and calls `ApiClient` directly, violating ADR-003 — the same architecture boundary should be enforced here as elsewhere in the codebase.

**Verdict:** Architecture violation and a validation gap found; this diff needs rework before it's mergeable (this is not an approval).
