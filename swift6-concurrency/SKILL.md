---
name: swift6-concurrency
description: Diagnose and fix Swift 6 strict concurrency errors and warnings. Use when seeing errors about MainActor-isolated conformances (Equatable, Codable, Sendable), nonisolated context violations, non-Sendable captures in @Sendable closures, or actor isolation issues. Covers Swift 6.2 Approachable Concurrency and SPM configuration.
---

# Swift 6 Concurrency Diagnostic Guide

This skill helps diagnose and fix Swift 6 strict concurrency compiler errors and warnings.

## Quick Diagnosis

Match your error to the pattern below:

| Error Pattern | Likely Fix |
|---------------|------------|
| `Main actor-isolated conformance of 'X' to 'Encodable/Decodable/Equatable' cannot be used in nonisolated context` | Use `nonisolated struct` for pure data types (preferred), or add explicit `nonisolated` protocol implementations |
| `Main actor-isolated initializer 'init(...)' cannot be called from outside of the actor` | Mark init as `nonisolated` or call from MainActor context |
| `Main actor-isolated static property 'X' can not be referenced from a nonisolated context` | Mark property `nonisolated` or access via `await MainActor.run { }` |
| `Capture of 'X' with non-Sendable type '...' in a '@Sendable' closure` | Make the type properly Sendable, restructure to avoid capture, or extract values before closure |
| `Call to main actor-isolated initializer 'init(from:)' in a synchronous nonisolated context` | Make the calling context `@MainActor` or mark the type's Codable conformance `nonisolated` |

## Common Fixes

### Fix 1: nonisolated Struct for Codable/Equatable

The most common issue: a struct used inside a `@MainActor` context (like SwiftData `@Model`) has its synthesized conformances inferred as MainActor-isolated.

```swift
// Problem: Struct used in @Model has isolated conformances
struct CampaignConfiguration: Codable, Equatable, Sendable {
    var name: String
    var enabled: Bool
}

@Model  // Implicitly @MainActor
final class Campaign {
    var configurationData: Data?
    
    var configuration: CampaignConfiguration {
        get {
            // Error: Main actor-isolated conformance of 'CampaignConfiguration' 
            // to 'Decodable' cannot be used in nonisolated context
            try? JSONDecoder().decode(CampaignConfiguration.self, from: configurationData!)
        }
    }
}
```

**Solution 1 (PREFERRED): Use `nonisolated struct`**

Mark the entire struct as `nonisolated`. All synthesized conformances (Codable, Equatable, Hashable) automatically become nonisolated:

```swift
// ✅ Best solution for pure data types
nonisolated struct CampaignConfiguration: Codable, Equatable, Sendable {
    var name: String
    var enabled: Bool
}
```

This is the cleanest solution because:
- No boilerplate manual implementations
- Synthesized conformances just work
- Clear semantic intent: "this type has no actor isolation"

**Solution 2: Remove `@MainActor` if the struct was explicitly marked**

If **your** struct is marked `@MainActor` but doesn't need it:

```swift
// Before (problematic)
@MainActor
struct CampaignConfiguration: Codable, Equatable { ... }

// After: Just remove @MainActor for pure data types
struct CampaignConfiguration: Codable, Equatable, Sendable { ... }
```

**Solution 3: Manual nonisolated conformances (last resort)**

Only use this if you need @MainActor for some members but also need Codable/Equatable:

```swift
@MainActor
struct CampaignConfiguration: Sendable {
    var name: String
    var enabled: Bool
}

// Explicit nonisolated conformances
extension CampaignConfiguration: Codable {
    nonisolated init(from decoder: Decoder) throws { ... }
    nonisolated func encode(to encoder: Encoder) throws { ... }
}

extension CampaignConfiguration: Equatable {
    nonisolated static func == (lhs: Self, rhs: Self) -> Bool { ... }
}
```

**Solution 4: For classes that must be @MainActor**

```swift
@MainActor
final class CampaignConfiguration: Codable, Equatable {
    let name: String
    let enabled: Bool

    nonisolated init(name: String, enabled: Bool) {
        self.name = name
        self.enabled = enabled
    }

    nonisolated required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        name = try container.decode(String.self, forKey: .name)
        enabled = try container.decode(Bool.self, forKey: .enabled)
    }

    nonisolated func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(name, forKey: .name)
        try container.encode(enabled, forKey: .enabled)
    }

    nonisolated static func == (lhs: CampaignConfiguration, rhs: CampaignConfiguration) -> Bool {
        lhs.name == rhs.name && lhs.enabled == rhs.enabled
    }
}
```

### Fix 2: Static Property Access

```swift
// Problem
@MainActor
struct Config {
    static let `default` = Config()
}

func process() {
    let config = Config.default // Error: Main actor-isolated static property
}

// Solution 1: Make the property nonisolated (if truly immutable)
@MainActor
struct Config {
    nonisolated static let `default` = Config()

    nonisolated init() { }
}

// Solution 2: Access from MainActor context
func process() async {
    let config = await MainActor.run { Config.default }
}

// Solution 3: Remove @MainActor if the type doesn't need it
struct Config: Sendable {
    static let `default` = Config()
}
```

### Fix 3: Non-Sendable Closure Capture

```swift
// Problem
func update(with closure: (inout CampaignConfiguration) -> Void) {
    Task {
        // Error: Capture of 'closure' with non-Sendable type
    }
}

// Solution 1: Make the closure @Sendable
func update(with closure: @Sendable (inout CampaignConfiguration) -> Void) {
    Task {
        // Now OK
    }
}

// Solution 2: Execute synchronously or restructure
func update(with closure: (inout CampaignConfiguration) -> Void) {
    var config = currentConfig
    closure(&config)
    // Then dispatch the result
    Task { @MainActor in
        self.currentConfig = config
    }
}
```

### Fix 4: Initializer Isolation

```swift
// Problem
@MainActor
struct ScanResult {
    let scanCompletedAt: Date
    let entries: [Entry]

    init(scanCompletedAt: Date, entries: [Entry]) {
        self.scanCompletedAt = scanCompletedAt
        self.entries = entries
    }
}

// Called from nonisolated context
func createResult() -> ScanResult {
    ScanResult(scanCompletedAt: Date(), entries: []) // Error
}

// Solution 1: Mark init as nonisolated
@MainActor
struct ScanResult {
    let scanCompletedAt: Date
    let entries: [Entry]

    nonisolated init(scanCompletedAt: Date, entries: [Entry]) {
        self.scanCompletedAt = scanCompletedAt
        self.entries = entries
    }
}

// Solution 2: Remove @MainActor if it's a pure data type
struct ScanResult: Sendable {
    let scanCompletedAt: Date
    let entries: [Entry]
}
```

## When to Use Each Approach

### Remove @MainActor (Preferred for Data Types)
- Pure data structs (Codable models, DTOs)
- Types that are just containers for values
- Types that don't interact with UI

### Use nonisolated (When @MainActor is Required)
- Types that need some MainActor methods but also need Codable/Equatable
- Computed properties that don't access actor-isolated state
- Initializers that only set immutable properties

## DANGER: Escape Hatches (@unchecked Sendable, @preconcurrency import)

**NEVER use `@unchecked Sendable` or `@preconcurrency import` without explicit user approval.**

These annotations are **compiler safety bypasses** that tell Swift "trust me, this is thread-safe" when the compiler cannot verify it. If you are wrong, the result is:

- **Runtime data races** that the compiler was designed to prevent
- **Undefined behavior** including memory corruption
- **Production crashes** that are intermittent and nearly impossible to reproduce
- **Security vulnerabilities** from race conditions
- **Extreme professional harm** to the developer whose name is on the code

### When Are These Acceptable?

**Only when ALL of the following are true:**

1. Every other solution has been exhausted (restructuring, nonisolated, actor isolation, Sendable conformance)
2. The code is genuinely thread-safe through external means (OS guarantees, internal locks, true immutability)
3. You have explicit, documented proof of thread safety
4. The user has explicitly approved this approach after understanding the risks
5. There is no possible way to fix the underlying type (e.g., third-party binary framework)

### What to Do Instead

Before even considering these escape hatches:

1. **Can you make the type Sendable properly?** Add conformance, make properties immutable
2. **Can you use nonisolated?** Mark specific members that don't need isolation
3. **Can you restructure?** Avoid crossing isolation boundaries with the problematic type
4. **Can you use an actor?** Wrap mutable state in an actor for proper synchronization
5. **Can you use Mutex?** (Swift 6.1+) For synchronous critical sections
6. **Can you copy the data?** Transform to a Sendable representation before crossing boundaries

### If Escape Hatches Are Truly Unavoidable

**You MUST ask the user for explicit approval** before suggesting `@unchecked Sendable` or `@preconcurrency import`. Explain:

- Why no other solution works
- What the specific thread-safety guarantee is
- What could go wrong if that guarantee is violated
- That this defeats the purpose of Swift 6 concurrency safety

```swift
// ONLY with explicit user approval and documented justification:

// For types with proven internal synchronization
final class ThreadSafeCache: @unchecked Sendable {
    // SAFETY: All access synchronized via internal DispatchQueue
    private let queue = DispatchQueue(label: "cache")
    private var storage: [String: Data] = [:]
}

// For legacy frameworks you cannot modify
@preconcurrency import LegacyFramework
// SAFETY: LegacyFramework documentation guarantees thread-safe access to X
```

## Swift 6.2 Approachable Concurrency

Swift 6.2 changes defaults to make concurrency more approachable:

### Enabling in SPM

```swift
// Package.swift
.target(
    name: "MyTarget",
    swiftSettings: [
        .swiftLanguageMode(.v6),
        .enableUpcomingFeature("InferIsolatedConformances"),     // SE-0470
        .enableUpcomingFeature("NonisolatedNonsendingByDefault") // SE-0461
    ]
)
```

### Key Changes

**1. Inferred Isolated Conformances (SE-0470)**

Protocol conformances automatically inherit actor isolation:

```swift
// With SE-0470, this just works:
@MainActor
final class User: Equatable {
    let id: UUID
    // == is automatically @MainActor isolated, no manual annotation needed
}
```

**2. nonisolated(nonsending) by Default (SE-0461)**

Nonisolated async functions stay on the caller's actor:

```swift
// Before: Would hop to global executor
nonisolated func helper() async { }

// After Swift 6.2: Stays on caller's actor by default
// Use @concurrent for explicit parallelism
@concurrent
func parallelHelper() async { }
```

**3. Disabled Property Wrapper Isolation Inference (SE-0401)**

Property wrappers no longer force isolation on the containing type:

```swift
struct ContentView: View {
    @StateObject var model = Model()  // No longer forces View to be @MainActor
}
```

## SPM Configuration Reference

### Swift 5 with Strict Concurrency Warnings

```swift
.target(
    name: "MyTarget",
    swiftSettings: [
        .enableUpcomingFeature("StrictConcurrency")
    ]
)
```

### Swift 6 Mode

```swift
.target(
    name: "MyTarget",
    swiftSettings: [
        .swiftLanguageMode(.v6)
    ]
)
```

### Swift 6.2 with Approachable Concurrency

```swift
.target(
    name: "MyTarget",
    swiftSettings: [
        .swiftLanguageMode(.v6),
        .enableUpcomingFeature("InferIsolatedConformances"),
        .enableUpcomingFeature("NonisolatedNonsendingByDefault")
    ]
)
```

## Decision Tree

When you see a concurrency error:

1. **Is the type a pure data model?** (Codable, Equatable, no UI interaction)
   - YES: Use `nonisolated struct` (preferred) or remove `@MainActor` and add `Sendable`
   - NO: Continue to step 2

2. **Does the specific member need MainActor isolation?**
   - NO: Mark it `nonisolated`
   - YES: Keep isolation, ensure callers are also isolated

3. **Is this a static property or initializer?**
   - YES: Consider `nonisolated` if it doesn't access isolated state
   - NO: Continue to step 4

4. **Is this a closure capture issue?**
   - YES: Make closure `@Sendable` or restructure to avoid capture
   - NO: Continue to step 5

5. **Can you restructure to avoid the boundary crossing?**
   - YES: Refactor to keep data within its isolation domain
   - NO: Continue to step 6

6. **Can you use an actor or Mutex for synchronization?**
   - YES: Wrap mutable state properly
   - NO: Continue to step 7

7. **Can you transform the data to a Sendable representation?**
   - YES: Copy/convert before crossing boundaries
   - NO: Continue to step 8

8. **Is this third-party code you cannot modify?**
   - NO: Go back and find another solution. There is always one.
   - YES: **STOP. Ask the user for explicit approval before proceeding.**
     - Explain why no other solution works
     - Document the thread-safety guarantee
     - Only then consider `@preconcurrency import` or `@unchecked Sendable`

## Reference: Error Messages

Full error messages you might encounter:

- `Main actor-isolated conformance of 'X' to 'Encodable' cannot be used in nonisolated context; this is an error in the Swift 6 language mode`
- `Main actor-isolated conformance of 'X' to 'Decodable' cannot be used in nonisolated context`
- `Main actor-isolated conformance of 'X' to 'Equatable' cannot be used in nonisolated context`
- `Main actor-isolated initializer 'init(...)' cannot be called from outside of the actor; this is an error in the Swift 6 language mode`
- `Main actor-isolated static property 'default' can not be referenced from a nonisolated context`
- `Capture of 'X' with non-Sendable type '...' in a '@Sendable' closure`
- `Call to main actor-isolated initializer 'init(from:)' in a synchronous nonisolated context`
- `Actor-isolated property 'X' can not be referenced from a non-isolated context`
- `Mutation of captured var 'X' in concurrently-executing code`

## Additional Resources

- [Swift.org Migration Guide](https://www.swift.org/migration/documentation/migrationguide/)
- [SE-0470: Inferred Isolated Conformances](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0470-isolated-conformances.md)
- [SE-0461: nonisolated(nonsending)](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0461-async-function-isolation.md)
