# Multi-Subscriber Async Streams

## Overview

AsyncStream is a unicast sequence. While multiple for await loops can technically subscribe to the same stream, they compete for values; each yielded element is consumed by exactly one subscriber. For broadcast behavior where every subscriber receives every value, you must use a custom coordinator that manages a unique continuation for each listener.

## The Single-Continuation Limitation

A computed `stream` property creates a new `AsyncStream` each time it's accessed. Each new stream captures the continuation, overwriting the previous one:

### ❌ Wrong — only the last subscriber gets values

````swift
private var continuation: AsyncStream<Int>.Continuation?

var stream: AsyncStream<Int> {
    AsyncStream { self.continuation = $0 }  // Overwrites previous continuation
}

Task { for await val in stream { print("Sub 1: \(val)") } }
Task { for await val in stream { print("Sub 2: \(val)") } }
// Only "Sub 2" receives values — fails silently
````

## Recommended Implementation

```swift
/// A property wrapper that manages a broadcast-capable `AsyncThrowingStream`,
/// allowing multiple consumers to independently observe the same stream of values.
///
/// `ConcurrentThrowingStream` provides a convenient interface for publishing values to an
/// asynchronous, throwing stream. Each access to the wrapped value returns a **new stream**
/// that receives **all future values** published to the stream.
///
/// This is useful in scenarios where multiple parts of your app need to listen to the same stream
/// of events (e.g., notifications, live updates, pub-sub patterns).
///
/// Values are stored and replayed once to new consumers if available. If an error has been published,
/// new consumers will receive the error immediately upon subscription.
///
/// ### Usage:
/// ```swift
/// @ConcurrentThrowingStream var myStream: AsyncThrowingStream<Int, Error>
///
/// // Start listening
/// Task {
///     for try await value in myStream {
///         print("Received: \(value)")
///     }
/// }
///
/// // Send a value
/// _myStream.yield(.success(42))
///
/// // Access the latest value or error
/// let latest = $myStream
/// ```
///
/// > Note: This property wrapper requires `Element` to conform to `Sendable`.
/// > It ensures thread safety internally and supports Swift 6 concurrency restrictions.
///
/// - Parameters:
///   - Element: The type of values yielded by the stream.
@available(iOS 13, *)
@propertyWrapper
public struct ConcurrentThrowingStream<Element: Sendable> {
    // MARK: API

    /// Creates a new `ConcurrentThrowingStream` with an optional initial value.
    /// - Parameter initialValue: A value to emit immediately to all current and future subscribers.
    public init(initialValue: Element? = nil) {
        storage.latestValue = initialValue
    }

    @available(*, unavailable, message: "Use `@ConcurrentThrowingStream(initialValue:)` without providing a stream directly.")
    public init(wrappedValue: AsyncThrowingStream<Element, Error>) {
        fatalError("This initializer is not supported.")
    }

    /// Returns a new `AsyncThrowingStream` instance that receives all future values.
    /// Each call to `wrappedValue` returns a new independent stream.
    public var wrappedValue: AsyncThrowingStream<Element, Error> {
        storage.addStream()
    }

    /// Returns the most recently yielded value or error.
    public var projectedValue: Result<Element?, Error> {
        storage.projectedResult
    }

    /// Publishes a new result to all active stream consumers.
    /// - Parameter result: A `Result` containing either a value or an error.
    public func yield(_ result: Result<Element, Error>) {
        storage.yield(result)
    }

    /// Finishes the stream, signaling completion to all consumers.
    public func finish() {
        storage.finish()
    }

    // MARK: Variables

    private let storage = Storage()

    // MARK: Internal Storage

    /// Thread-safe shared storage and stream management.
    /// Marked `@unchecked Sendable` due to internal use of `NSLock`.
    private final class Storage: @unchecked Sendable {
        private let lock = NSLock()
        private var storedContinuations: [UUID: AsyncThrowingStream<Element, Error>.Continuation] = [:]
        private var latestError: Error?

        var latestValue: Element?

        var projectedResult: Result<Element?, Error> {
            if let error = latestError {
                return .failure(error)
            } else {
                return .success(latestValue)
            }
        }

        func addStream() -> AsyncThrowingStream<Element, Error> {
            AsyncThrowingStream { continuation in
                let uuid = UUID()

                lock.lock()
                storedContinuations[uuid] = continuation
                if let value = latestValue {
                    continuation.yield(value)
                } else if let error = latestError {
                    continuation.yield(with: .failure(error))
                }
                lock.unlock()

                continuation.onTermination = { [weak self] _ in
                    self?.removeContinuation(uuid)
                }
            }
        }

        func yield(_ result: Result<Element, Error>) {
            lock.lock()
            switch result {
            case .success(let value):
                latestValue = value
                for continuation in storedContinuations.values {
                    continuation.yield(value)
                }
            case .failure(let error):
                latestError = error
                for continuation in storedContinuations.values {
                    continuation.yield(with: .failure(error))
                }
            }
            lock.unlock()
        }

        func finish() {
            lock.lock()
            let continuations = storedContinuations.values
            lock.unlock()

            for continuation in continuations {
                continuation.finish()
            }
        }

        private func removeContinuation(_ id: UUID) {
            lock.lock()
            storedContinuations.removeValue(forKey: id)
            lock.unlock()
        }
    }
}
```
