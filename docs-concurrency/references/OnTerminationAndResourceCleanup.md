# Resource Cleanup and Memory Management

## Overview

This guide focuses on stream and resource cleanup: continuations stored in caches must be removed when their streams end, and external resources (sockets, timers, file handles) must be cleaned up when the associated stream terminates.

## onTermination

`continuation.onTermination` is called when a stream ends for any reason — either because `continuation.finish()` was called, or because the consumer's `for await` loop was cancelled. Use it to remove the continuation from any collection it was added to.

```swift
AsyncStream { continuation in
    Task { await self.register(continuation) }

    continuation.onTermination = { [weak self] _ in
        Task { await self?.unregister() }
    }
}
```

**Rules:**
- Always use `[weak self]` in `onTermination` closures — the handler is stored by the continuation and can outlive the enclosing scope
- The `termination` parameter indicates `.finished` or `.cancelled` — inspect it if different cleanup is needed for each case
- Register cleanup logic at creation time, not after the fact

## Continuation Caches

When continuations are stored in a dictionary (e.g., per-user or per-session streams), pair each addition with an `onTermination` removal:

```swift
actor StreamCache {
    private var streams: [String: AsyncStream<Event>.Continuation] = [:]

    func stream(for key: String) -> AsyncStream<Event> {
        AsyncStream { [weak self] continuation in
            Task { await self?.add(key, continuation) }

            continuation.onTermination = { [weak self] _ in
                Task { await self?.remove(key) }
            }
        }
    }

    func send(_ event: Event, to key: String) {
        streams[key]?.yield(event)
    }

    private func add(_ key: String, _ continuation: AsyncStream<Event>.Continuation) {
        streams[key] = continuation
    }

    private func remove(_ key: String) {
        streams.removeValue(forKey: key)
    }
}
```

## Task Cleanup

SwiftUI note: Tasks started by `.task` are tied to view lifecycle and are cancelled automatically when the view disappears or `task(id:)` changes.

## Checklist

- [ ] `onTermination` registered for every continuation stored in a collection
- [ ] `[weak self]` used in all `onTermination` closures
- [ ] External resources (sockets, timers) closed in `onTermination`
- [ ] Cache size bounded with an eviction policy

## Rules

1. **`onTermination` is your cleanup hook** — register it at stream creation, not later
2. **Use `[weak self]` in `onTermination` closures** — they're stored by the continuation and can outlive the object
3. **Actors for continuation dictionaries** — provides thread-safe access without manual locking
4. **Bound your caches** — streams that are never finished or removed hold memory and background resources indefinitely
