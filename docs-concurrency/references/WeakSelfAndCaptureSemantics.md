# Capture Semantics: weak self vs strong self

## Overview

In Swift concurrency, whether to use `[weak self]` depends on whether the closure or task body can outlive the object. Structured concurrency eliminates most retain cycles that required `[weak self]` in callback-based code, but unstructured tasks and stored closures still require careful consideration.

## The Core Rule

**Use `[weak self]` when the task or closure is stored and may outlive the object's intended lifetime. Use strong `self` when the task's lifetime is bounded by the current scope (structured concurrency).**

## Structured Concurrency — Strong Self

In structured concurrency, the task's lifetime is bounded by the function that created it. When the function returns, the task is done. No retain cycle is possible because nothing holds the closure beyond the method's scope.

```swift
func loadData() async throws -> Data {
    // Task lifetime bounded by this function — strong self is fine
    let step1 = try await fetchData()
    let step2 = try await processData(step1)
    return step2
}
```

```swift
func performOperation() async throws -> Data {
    try await retry(maxAttempts: 3) {
        try await self.fetchData()          // Strong self OK
    } shouldRetry: { error in
        self.shouldRetry(error)             // Strong self OK — closure not stored
    }
}
```

### ❌ Wrong — unnecessary weak self in a bounded scope

```swift
func retryOperation() async throws -> Data {
    try await retry(maxAttempts: 3) {
        try await self.performOperation()
    } shouldRetry: { [weak self] error in
        // If self is nil, returns false — aborts valid retries unexpectedly
        self?.shouldRetry(error) ?? false
    }
}
```

Using `[weak self]` here provides no benefit (no retain cycle exists) and introduces a bug: if `self` is deallocated mid-retry, the retry logic silently aborts.

## Unstructured Tasks — Weak Self

Unstructured tasks created with `Task { }` escape the current scope. If the task holds a strong reference to `self` and `self` holds the task, there's a retain cycle. Even without a cycle, the task may keep `self` alive longer than intended.

```swift
@MainActor
class ViewModel {
    private var syncTask: Task<Void, Never>?

    func startBackgroundSync() {
        syncTask = Task { [weak self] in
            while !Task.isCancelled {
                guard let self else { break }
                await self.syncData()
                try? await Task.sleep(for: .seconds(60))
            }
        }
    }

    deinit {
        syncTask?.cancel()
    }
}
```

**When to use `[weak self]` in a Task:**
- The `Task` is stored as a property on `self`
- The task may run longer than `self`'s natural lifetime
- The loop should stop if `self` is deallocated

## Stored Closures and Callbacks

Closures stored by external systems (notification observers, delegate callbacks, completion handlers) must use `[weak self]`:

```swift
func setupNotifications() {
    NotificationCenter.default.addObserver(
        forName: .dataUpdated,
        object: nil,
        queue: .main
    ) { [weak self] notification in       // Weak self required — stored by NotificationCenter
        guard let self else { return }
        Task { await self.handleUpdate(notification) }
    }
}
```

```swift
func setupCallback() {
    legacyAPI.onComplete { [weak self] result in   // Weak self required — closure stored by API
        guard let self else { return }
        Task { await self.handleResult(result) }
    }
}
```

## TaskGroup — Strong Self

Task bodies inside `withTaskGroup` are structured — they don't escape the group's scope. Strong self is appropriate:

```swift
func performComplexOperation() async throws {
    await withTaskGroup(of: Void.self) { group in
        group.addTask { await self.stepOne() }    // Strong self OK
        group.addTask { await self.stepTwo() }    // Strong self OK
    }
}
```

## Decision Tree

```
Does the closure/task escape the current function scope?
│
├─ No (structured concurrency) → use strong self
│   ├─ async function body
│   ├─ TaskGroup addTask closure
│   └─ Inline async operation arguments (retry, timeout, etc.)
│
└─ Yes → Is it stored, and might it outlive self?
    ├─ Yes → use [weak self]
    │   ├─ Task stored as a property
    │   ├─ Completion handlers stored by external APIs
    │   └─ Notification/delegate callbacks
    │
    └─ No (short-lived temporary storage) → use strong self
```

## Common Misconceptions

**"Always use `[weak self]` with `Task`"** — Only when the Task is unstructured and stored. Inline `Task { }` calls inside a function without storing the result don't need it.

**"Async/await eliminates all retain cycles"** — Structured concurrency eliminates most, but stored closures and unstructured tasks still apply the same rules as before.

**"`[weak self]` is always the safe choice"** — Unnecessary `[weak self]` produces unexpected `nil` values and can abort operations prematurely (e.g., aborting a retry because `self` was briefly not retained).

## Summary

| Context | Capture |
|---|---|
| `async` function body | Strong self |
| `TaskGroup` child task | Strong self |
| Inline async operation arguments (retry, timeout) | Strong self |
| Stored `Task` (property on self) | `[weak self]` |
| Notification/delegate/completion handler | `[weak self]` |
| Any closure stored by an external system | `[weak self]` |
