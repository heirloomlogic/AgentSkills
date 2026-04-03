# Task Cancellation

## Overview

Swift Tasks support cooperative cancellation. Cancellation is requested, not forced — the task must check for it and exit gracefully. Infinite async sequences (event streams, observation loops) never complete on their own, so explicit cancellation management is required.

## Requesting Cancellation

```swift
let task = Task {
    for await value in eventStream {
        process(value)
    }
}

// Later:
task.cancel()
```

Calling `.cancel()` sets the task's cancellation flag and wakes it if it's suspended. The task is responsible for checking this flag and stopping.

## Checking Cancellation

### In a for-await loop

```swift
for await value in stream {
    if Task.isCancelled { break }
    process(value)
}
```

### After long operations

```swift
for await batch in dataStream {
    if Task.isCancelled { break }

    for item in batch {
        if Task.isCancelled { break }
        processItem(item)
    }
}
```

### In throwing contexts

```swift
for await value in stream {
    try Task.checkCancellation()  // Throws CancellationError if cancelled
    try await processValue(value)
}
```

`Task.checkCancellation()` is preferred when you want cancellation to propagate up as an error rather than breaking out manually.

## Storing Tasks for Later Cancellation

Tasks that observe infinite streams must be stored so they can be cancelled when no longer needed.

### ✅ Correct — stored task with explicit cancellation

```swift
private var observationTask: Task<Void, Never>?

func startObserving() {
    observationTask = Task {
        for await value in eventStream {
            if Task.isCancelled { break }
            process(value)
        }
    }
}

func stopObserving() {
    observationTask?.cancel()
    observationTask = nil
}
```

### ❌ Wrong — awaiting an infinite task

```swift
let task = Task {
    for await value in infiniteStream {
        process(value)
    }
}

await task.value  // Hangs forever — the stream never finishes
```

Never await the `.value` of a task that runs over an infinite stream.

## TaskGroup for Multiple Streams

When observing multiple streams concurrently, use `TaskGroup` for structured cleanup. Cancelling the parent task cancels all child tasks automatically.

```swift
private var observationTask: Task<Void, Never>?

func startObserving() {
    observationTask = Task {
        await withTaskGroup(of: Void.self) { group in
            group.addTask {
                for await value in stream1 {
                    if Task.isCancelled { break }
                    self.handle1(value)
                }
            }
            group.addTask {
                for await value in stream2 {
                    if Task.isCancelled { break }
                    self.handle2(value)
                }
            }
            await group.waitForAll()
        }
    }
}

func stopObserving() {
    observationTask?.cancel()  // Cancels all child tasks
    observationTask = nil
}
```

## Cancellation in deinit

Always cancel stored tasks in `deinit` as a safety net:

```swift
@MainActor
class ViewModel {
    private var observationTask: Task<Void, Never>?

    func startObserving() {
        observationTask = Task { @MainActor in
            for await items in dataStream {
                if Task.isCancelled { break }
                self.items = items
            }
        }
    }

    deinit {
        observationTask?.cancel()
    }
}
```

Note: In most SwiftUI apps, prefer the `.task` modifier over manual task management — it ties cancellation to the view lifecycle automatically (see `TaskModifierAndViewLifecycle.md`).

## SwiftUI .task vs Manual Task Storage

- If work starts from SwiftUI `.task`/`.task(id:)`, do not duplicate it with a stored `Task` property just to cancel later.
- If work is started outside view lifecycle binding (service layer, coordinator, long-lived object), store the task and cancel it explicitly.

## Rules

1. **Always store tasks that run infinite streams** — you must be able to cancel them
2. **Check `Task.isCancelled` in loops** — at the start of each iteration, and after long operations
3. **Never `await task.value` on an infinite task** — it will hang forever
4. **Use `TaskGroup` for multiple concurrent streams** — single cancellation point, structured cleanup
5. **Cancel and nil out in pairs** — `task?.cancel(); task = nil` prevents double-cancellation
6. **Cancel in `deinit`** — final safety net if explicit cleanup is missed
