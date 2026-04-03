# The .task Modifier and View Lifecycle

## Overview

SwiftUI's `.task` modifier starts an async task when a view appears and automatically cancels it when the view disappears. It is the correct place to begin stream observation and other async work tied to a view's lifetime. Do not start observation tasks in `init()` or `onAppear` with unstructured `Task { }` — these are not cancelled automatically.

## Basic Usage

```swift
struct ContentView: View {
    @StateObject var viewModel: ViewModel

    var body: some View {
        List(viewModel.items) { item in
            Text(item.name)
        }
        .task {
            await viewModel.onAppear()
        }
    }
}
```

The `.task` modifier:
1. Starts the async work when the view appears
2. Cancels the task (and propagates cancellation) when the view disappears
3. Does not require manual task storage or `cancel()` calls

If a flow is fully driven by `.task`/`.task(id:)`, avoid adding a separate `Task` property on the ViewModel just for cancellation.

## ViewModel Pattern

The ViewModel exposes an `async` method. The view calls it from `.task`. The `for await` loop runs for as long as the view is visible.

```swift
@MainActor
class ViewModel {
    private(set) var items: [Item] = []

    func onAppear() async {
        for await items in itemsStream {
            self.items = items
        }
    }
}
```

**Rules:**
- `onAppear()` must be `async` so the `.task` modifier can manage its lifetime
- Do not create a `Task { }` inside `onAppear()` — that task would be unstructured and not cancelled when the view disappears
- Do not start stream observation in `init()` — the task would not be tied to the view

### ❌ Wrong — unstructured task in init

```swift
@MainActor
class ViewModel {
    private var observationTask: Task<Void, Never>?

    init(services: Services) {
        // Task starts immediately, not tied to view lifecycle
        observationTask = Task { [weak self] in
            guard let self else { return }
            for await items in services.itemsStream {
                guard !Task.isCancelled else { break }
                self.items = items
            }
        }
    }
}
```

**Problems:** Task runs even when no view is observing. Must be manually tracked and cancelled. Requires `[weak self]` complexity that `.task` eliminates.

### ✅ Correct — async method called from .task

```swift
@MainActor
class ViewModel {
    private(set) var items: [Item] = []

    init(services: Services) {
        self.services = services
        // No tasks started here
    }

    func onAppear() async {
        for await items in services.itemsStream {
            self.items = items
        }
    }
}
```

## Patterns

### Single stream

```swift
func onAppear() async {
    for await item in stream {
        process(item)
    }
}

// View
.task { await viewModel.onAppear() }
```

### Multiple concurrent streams

Use `TaskGroup` to run multiple `for await` loops concurrently under the same structured task:

```swift
func onAppear() async {
    await withTaskGroup(of: Void.self) { group in
        group.addTask {
            for await item in self.stream1 { self.handle1(item) }
        }
        group.addTask {
            for await item in self.stream2 { self.handle2(item) }
        }
    }
}

// View
.task { await viewModel.onAppear() }
```

### Initial load followed by observation

```swift
func onAppear() async {
    await loadInitialData()

    for await update in updatesStream {
        handleUpdate(update)
    }
}

// View
.task { await viewModel.onAppear() }
```

### Restart on dependency change — task(id:)

`.task(id:)` cancels and restarts the task whenever the `id` value changes:

```swift
func observeUser(id: String) async {
    for await user in userStream(id: id) {
        self.user = user
    }
}

// View
.task(id: viewModel.selectedUserId) {
    await viewModel.observeUser(id: viewModel.selectedUserId)
}
```

## Graceful Cleanup with defer

Use `defer` for cleanup that must run when the task ends, whether due to normal completion or cancellation:

```swift
func onAppear() async {
    defer { cleanup() }

    for await item in stream {
        process(item)
    }
}
```

## Common Mistakes

### ❌ Creating Task inside onAppear()

```swift
func onAppear() {
    Task {                              // Unstructured — not cancelled on disappear
        for await item in stream {
            process(item)
        }
    }
}

.onAppear { viewModel.onAppear() }    // Not .task, so no lifecycle binding
```

**Fix:** Make `onAppear()` async and call it from `.task`.

### ❌ View accessing streams directly

```swift
.task {
    for await item in viewModel.stream {   // View manages stream — wrong layer
        // handle item
    }
}
```

**Fix:** The ViewModel manages stream observation. The view only calls a ViewModel method.

## Summary

| Pattern | Correct tool |
|---|---|
| Start observation when view appears | `.task { await viewModel.onAppear() }` |
| Stop observation when view disappears | Automatic — `.task` cancels on disappear |
| Restart observation when a value changes | `.task(id: value) { ... }` |
| Multiple concurrent streams | `withTaskGroup` inside the `async` method |
| Cleanup on exit | `defer { }` inside the `async` method |
