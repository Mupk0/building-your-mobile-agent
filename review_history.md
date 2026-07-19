# Review history

2026-07-19 | PR-01 | ProfileViewModel added but never adopted by ProfileView — incomplete MVVM wiring per ADR-001 | MEDIUM
2026-07-19 | PR-02 | ProfileViewModel calls URLSession directly, bypassing APIClient and repository layer | MEDIUM
2026-07-19 | PR-03 | Hardcoded live API token in PaymentService | HIGH
2026-07-19 | PR-01 | RideRepository and Ride types referenced but not defined anywhere in repo | MEDIUM
2026-07-19 | PR-01 (nebius-academy-templates/mobile-pr-review-practice) | No issues found — clean diff across style, security, architecture | NONE
2026-07-19 | PR-03 (nebius-academy-templates/mobile-pr-review-practice) | Hardcoded live API key in PaymentService | HIGH
2026-07-19 | PR-02 (nebius-academy-templates/mobile-pr-review-practice) | ProfileViewModel imports UIKit, calls URLSession directly, bypassing APIClient and repository — violates ADR-001/002/003 | MEDIUM
2026-07-19 | PR-04 (nebius-academy-templates/mobile-pr-review-practice) | TripViewModel calls ApiClient directly, bypassing repository layer — violates ADR-003 | MEDIUM
2026-07-19 | PR-05 (nebius-academy-templates/mobile-pr-review-practice) | Hardcoded client secret in CheckoutViewModel, also transmitted from client-side code | HIGH