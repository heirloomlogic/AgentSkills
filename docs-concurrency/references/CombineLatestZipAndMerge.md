# combineLatest and zip

## Overview

`combineLatest` and `zip` from AsyncAlgorithms combine values from multiple async sequences into synchronized, paired updates. Use them whenever multiple streams need to coordinate to update shared state — separate tasks reading from different streams and writing to the same state create race conditions.

## combineLatest

`combineLatest` emits a new tuple whenever **any** of the input sequences emits a value. The tuple contains the latest value from each sequence.

```swift
import AsyncAlgorithms

for await (authState, user) in combineLatest(authStream, userStream) {
    // Both values are synchronized — no stale reads
    isReady = authState.isAuthenticated && user != nil
}
```

**Behavior:**
- Waits until all sequences have emitted at least one value before emitting the first tuple
- After that, emits on every new value from any sequence
- The other sequence's latest value is re-used in the tuple

### ✅ Correct — synchronized shared state update

```swift
import AsyncAlgorithms

@MainActor
class AppContext {
    private(set) var isReady = false

    func observe() {
        Task { @MainActor in
            for await (authState, user) in combineLatest(authStream, userStream) {
                isReady = authState.isAuthenticated && user != nil
            }
        }
    }
}
```

### ❌ Wrong — race condition with separate tasks

```swift
// DON'T DO THIS — creates a race condition
Task {
    for await authState in authStream {
        isReady = authState.isAuthenticated && hasUser  // reads stale hasUser
    }
}
Task {
    for await user in userStream {
        isReady = isAuthenticated && user != nil  // reads stale isAuthenticated
    }
}
```

**Why this is wrong:**
- Each task reads one property while the other task may be writing to it
- Values from different streams get mixed across iterations
- The result can be inconsistent state (e.g., `isReady = true` with outdated user data)
- The bug is nondeterministic and hard to reproduce

## zip

`zip` emits a tuple only when **all** sequences have each produced a new value. It pairs the first value from each, then the second, and so on — like the `zip` function on collections.

```swift
import AsyncAlgorithms

for await (image, metadata) in zip(imageStream, metadataStream) {
    // Guaranteed: these came from the same "round" of emissions
    display(image: image, title: metadata.title)
}
```

**Behavior:**
- Waits for one new value from every sequence before emitting
- Back-pressures slower producers — faster ones queue up
- Use when values are meant to be consumed in lock-step

## Choosing Between combineLatest and zip

| | `combineLatest` | `zip` |
|---|---|---|
| Emits when | Any sequence has a new value | All sequences have a new value |
| Re-uses values | Yes — latest from others | No — each value consumed once |
| Use for | Reactive state derived from multiple sources | Paired/correlated data from multiple sources |

## merge

`merge` is the unordered variant — it emits values from any sequence as they arrive, with no pairing or coordination:

```swift
let allEvents = merge(authEventStream, userEventStream)

for await event in allEvents {
    handle(event)
}
```

Use `merge` when streams are independent and the values don't need to be correlated.

## AsyncAlgorithms Dependency

```swift
// Package.swift
.package(url: "https://github.com/apple/swift-async-algorithms", from: "1.0.0")
```

```swift
import AsyncAlgorithms
```
