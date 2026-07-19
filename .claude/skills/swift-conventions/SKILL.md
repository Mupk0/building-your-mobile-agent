---
name: swift-conventions
description: "Swift (iOS) style and convention rules for the mobile team. Load when reviewing any .swift file in a PR diff — covers naming, concurrency (async/await, GCD), and UIKit/SwiftUI framework usage. Used by the style-reviewer sub-agent."
---

# Swift Conventions

Apply these rules to every `.swift` file in the diff. Report only issues introduced or
touched by the diff. For each finding, emit one line:

```
[SEVERITY] <short description> — <the rule it breaks>
```

## Rules

### Naming — `[LOW]`
Types (class, struct, enum, protocol) use `UpperCamelCase`. Properties, functions,
variables, and enum cases use `lowerCamelCase`. No snake_case, no `ALLCAPS`, no
lower-cased type names. Names must be descriptive — avoid `data`, `DATA`, `getData`.
- Violation: `class feed_cell`, `static var Instance`, `var badge_count`, `func RequestPermission`.

### Concurrency — `[MEDIUM]`
New asynchronous code uses `async/await` (per ADR-002), not completion-handler or raw
`DispatchQueue`/`URLSession` callback pyramids. Hop to the main actor with `@MainActor`
or `await MainActor.run` — not `DispatchQueue.main.async` — when updating UI. Never
force-unwrap errors (`error!`).
- Violation: `func load(completion: @escaping (Bool) -> Void)` wrapping a callback, or
  `DispatchQueue.main.async { self.label.text = ... }` inside an async flow.

### UIKit / SwiftUI usage — `[LOW]`
Keep view code free of business logic and networking — a `UITableViewCell` or SwiftUI
`View` must not fetch data or hold model state (defer to the ViewModel per ADR-001).
Prefer weak captures (`[weak self]`) in escaping closures to avoid retain cycles. Do not
store raw dictionaries (`[String: Any]`) as a cell's model.
- Violation: a cell with a `LoadImage(urlString:)` method that runs `URLSession` directly.

## Template

**Compliant**

```swift
import UIKit

final class FeedCell: UITableViewCell {
    @IBOutlet private var titleLabel: UILabel!

    func configure(with item: FeedItem) {
        titleLabel.text = item.title
    }
}

final class FeedViewModel {
    private let repository: FeedRepository

    init(repository: FeedRepository) {
        self.repository = repository
    }

    @MainActor
    func loadFeed() async throws -> [FeedItem] {
        try await repository.fetchFeed()
    }
}
```

**Non-compliant**

```swift
class feed_cell: UITableViewCell {          // [LOW] type in snake_case + lowercase
    var DATA: [String: Any]?                // [LOW] non-descriptive, raw dictionary model

    func LoadImage(urlString: String) {     // [LOW] method UpperCamelCase; networking in a cell
        guard let url = URL(string: urlString) else { return }
        URLSession.shared.dataTask(with: url) { data, _, _ in   // [MEDIUM] callback, not async/await
            DispatchQueue.main.async {       // [MEDIUM] should hop via @MainActor
                self.imageView?.image = UIImage(data: data!)     // force-unwrap
            }
        }.resume()
    }
}
```
