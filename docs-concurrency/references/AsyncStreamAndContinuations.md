# AsyncStream and Async Sequences

## Overview

`AsyncStream` and `AsyncThrowingStream` are the core building blocks for creating custom async sequences in Swift. They bridge callback- and event-based systems into the structured concurrency model.

## AsyncStream

### Creating a Stream

Use `AsyncStream.makeStream` to get a stream and its continuation together:

```swift
let (stream, continuation) = AsyncStream.makeStream(of: Int.self)

// Yield values from anywhere
continuation.yield(42)
continuation.yield(100)

// Signal completion
continuation.finish()
```

### Consuming a Stream

```swift
for await value in stream {
    print(value)
}
// Loop exits when the stream finishes
```

### Wrapping Callback-Based APIs

```swift
func timerStream(interval: TimeInterval) -> AsyncStream<Date> {
    AsyncStream { continuation in
        let timer = Timer.scheduledTimer(withTimeInterval: interval, repeats: true) { _ in
            continuation.yield(Date())
        }
        continuation.onTermination = { _ in
            timer.invalidate()
        }
    }
}
```

## Replacing @Published Properties

Use a property with `didSet` and a stored continuation to replicate current-value behavior:

```swift
private(set) var count: Int = 0 {
    didSet { countContinuation?.yield(count) }
}

private var countContinuation: AsyncStream<Int>.Continuation?

lazy var countStream: AsyncStream<Int> = {
    AsyncStream { continuation in
        self.countContinuation = continuation
        continuation.yield(self.count)  // Emit current value immediately
    }
}()
```

**Rules:**
- Emit the current value when the stream is first created so subscribers start with up-to-date state
- Store the continuation as a property; it's the only way to push values later
- Use `lazy` initialization so the continuation is captured before any values are emitted

## Replacing PassthroughSubject

```swift
let (stream, continuation) = AsyncStream.makeStream(of: Int.self)

Task {
    for await value in stream {
        print(value)
    }
}

continuation.yield(42)
```

**Rules:**
- `send()` becomes `continuation.yield()`
- `.sink` becomes a `for await` loop inside a `Task`
- The stream and continuation are separate objects — store both if you need to yield values and consume them from different sites

## Error Handling with AsyncThrowingStream

```swift
let (stream, continuation) = AsyncThrowingStream.makeStream(of: Value.self)

// Yield a value
continuation.yield(value)

// Terminate with an error
continuation.finish(throwing: error)

// Terminate successfully
continuation.finish()
```

**Consuming:**
```swift
do {
    for try await value in stream {
        process(value)
    }
} catch {
    handleError(error)
}
```

## Operators (AsyncAlgorithms)

Most common operators are available via the `AsyncAlgorithms` package with identical or near-identical syntax.

```swift
// Package.swift
.package(url: "https://github.com/apple/swift-async-algorithms", from: "1.0.0")
```

```swift
import AsyncAlgorithms

// Transformation
stream.map { $0 * 2 }
stream.filter { $0 > 10 }
stream.compactMap { Int($0) }
stream.flatMap { createSequence($0) }

// Deduplication and timing
stream.removeDuplicates()
stream.debounce(for: .seconds(0.5))     // No scheduler needed
stream.throttle(for: .seconds(1), latest: true)

// Combining
merge(stream1, stream2)                 // Values interleaved as they arrive
zip(stream1, stream2)                   // Paired: waits for one from each
combineLatest(stream1, stream2)         // Latest from each, re-emitted on any update
```

## Operator Reference Table

| Operator | AsyncAlgorithms API | Notes |
|----------|---------------------|-------|
| map | `.map` | Same syntax |
| filter | `.filter` | Same syntax |
| flatMap | `.flatMap` | Same syntax |
| compactMap | `.compactMap` | Same syntax |
| removeDuplicates | `.removeDuplicates()` | Same syntax |
| debounce | `.debounce(for:)` | No scheduler parameter |
| throttle | `.throttle(for:latest:)` | No scheduler parameter |
| catch / error handling | `do-catch` | Swift error handling |
| merge | `merge(_:_:)` | Free function |
| zip | `zip(_:_:)` | Free function |
| combineLatest | `combineLatest(_:_:)` | Free function |
| scheduler switching | `@MainActor` / actors | Actor isolation model |

## Key Rules

1. **No type erasure needed** — `AsyncStream` is already a concrete type
2. **No schedulers** — actor isolation replaces `.receive(on:)` and `.subscribe(on:)`
3. **Continuations are single-use** — one continuation per stream; a second subscriber overwrites the first (see `AsyncChannelAndMultiSubscriberStreams.md` for solutions)
4. **Always call `finish()`** — streams that are never finished hold resources indefinitely
5. **Use `onTermination`** — register cleanup logic when the stream ends or is cancelled
