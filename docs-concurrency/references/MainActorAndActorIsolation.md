# @MainActor and Actor Isolation

## Overview

`@MainActor` is a global actor that guarantees code runs on the main thread. Applying it to a class or function provides compile-time enforcement that all accesses happen on the main thread — no explicit `DispatchQueue.main` calls needed.

## When to Use @MainActor

**Apply `@MainActor` to any class that owns UI state** — ViewModels, Coordinators, and similar types. This ensures all reads and writes to published properties happen on the main thread, which is required for SwiftUI and UIKit.

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []
    @Published var isLoading = false

    func loadItems() async {
        isLoading = true
        items = try await apiClient.fetchItems()
        isLoading = false
    }
}
```

**Why this is safe:** Async calls like network requests suspend execution — they don't block the thread. The main thread is released during the suspension and resumes when the result arrives. There is no performance penalty for calling async APIs from `@MainActor`.

## Moving Work Off the Main Actor

For CPU-intensive work (image processing, large data parsing, complex computation), explicitly move work to a background context.

### Option 1: Task.detached

```swift
@MainActor
class ViewModel {
    var processedImage: UIImage?

    func processImage(_ data: Data) async {
        let image = await Task.detached(priority: .userInitiated) {
            expensiveImageProcessing(data)  // Runs on background thread
        }.value

        processedImage = image  // Back on main actor
    }
}
```

### Option 2: Custom Actor

```swift
actor ImageProcessor {
    func process(_ data: Data) async -> UIImage {
        expensiveImageProcessing(data)  // Runs on actor's executor
    }
}

@MainActor
class ViewModel {
    private let processor = ImageProcessor()
    var processedImage: UIImage?

    func processImage(_ data: Data) async {
        processedImage = await processor.process(data)
    }
}
```

Custom actors are preferred when the background logic is reusable or complex, as they provide a named, isolated context with clear ownership.

## Rules

✅ **Default to `@MainActor` for ViewModels and UI-related classes**
- Prevents accidental UI updates from background contexts
- Eliminates the need for `DispatchQueue.main.async` calls
- Matches SwiftUI's expectations

✅ **Async network and database calls are safe to await from `@MainActor`**
- They suspend, not block, the thread
- The runtime handles context switching internally

✅ **Use `Task.detached` or a custom actor for CPU-heavy work**
- Image processing and manipulation
- Large JSON/data parsing
- Cryptographic operations
- Anything that takes noticeable time on the thread

❌ **Don't use `Task.detached` for simple async calls**
- Adds unnecessary complexity and context-switch overhead
- Modern async APIs already manage their own threading

❌ **Don't use `MainActor.run {}` from within a `@MainActor` class**
- If the class is already isolated to `@MainActor`, all methods and property accesses are already on the main thread
- `MainActor.run {}` is for use from non-isolated contexts when you need to hop to the main actor

## Nonisolated Methods

Use `nonisolated` to opt specific methods out of actor isolation, useful for pure computation or conformance requirements:

```swift
@MainActor
class ViewModel {
    nonisolated func formatDate(_ date: Date) -> String {
        DateFormatter.medium.string(from: date)  // No actor-isolated state accessed
    }
}
```

## Summary

The threading model in Swift concurrency is based on **what isolation a type needs**, not **where to switch threads**.

- `@MainActor` for UI-owning classes
- Custom actors for heavy background work
- `nonisolated` for pure functions that don't touch shared state
- Let async APIs handle their own threading
