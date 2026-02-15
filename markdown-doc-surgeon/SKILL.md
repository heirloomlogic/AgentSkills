---
name: markdown-doc-surgeon
description: "Rebalance and tighten markdown documentation that has grown unwieldy. Compresses bloated edge-case sections, verbose code examples, and lengthy DO/DON'T addendums back into proportion with core content — without losing information. Works on READMEs, system prompts, agent skills, API docs, and any markdown reference material. Activate on 'clean up docs', 'tighten documentation', 'rebalance markdown', 'compress docs', 'documentation cleanup', 'doc is too long', 'simplify docs'. NOT for: writing new documentation from scratch (use skill-writer), copyediting prose style, or converting between formats."
---

# Markdown Doc Surgeon

Rebalance markdown documents that have accumulated weight unevenly — where the core is clean but addendums, edge-case lists, and verbose examples have blown out of proportion.

## When to Use

- A doc's edge-case sections are longer than the core explanation
- Code examples repeat the same pattern with minor variations
- DO/DON'T lists have grown into exhaustive catalogs
- A system prompt or skill has been edited many times and feels sprawling
- A README has accumulated "one more thing" sections that dilute the signal

## Audience Detection

Before touching anything, determine who reads this document:

| Audience | Characteristics | Compression thresholds |
|----------|----------------|----------------------|
| **LLM** | System prompts, skills, agent instructions | Aggressive. LLMs parse structure well; favor tables, terse rules, and structured lists over prose. Remove hedging language. |
| **Human developer** | READMEs, API docs, onboarding guides | Moderate. Preserve narrative flow and motivation ("why") alongside the "what". Keep 1-2 illustrative examples per concept. |
| **Mixed** | Skills read by both agents and humans | Balance. Use structured format but keep enough prose for human scannability. |

## Diagnostic Step — Assess Before Cutting

Run this assessment before making any changes:

1. **Map the skeleton** — List every H1/H2/H3 and its line count
2. **Classify each section** using this tier system:

| Tier | Definition | Target ratio |
|------|-----------|-------------|
| **Core** | Essential concepts, primary instructions, the "happy path" | 50–60% of doc |
| **Supporting** | Important context, common variations, key examples | 25–30% |
| **Edge** | Corner cases, advanced scenarios, troubleshooting | 10–15% |
| **Verbose** | Redundant examples, excessive DO/DON'T pairs, repeated warnings | 0% — compress or cut |

3. **Measure current ratio** — If Edge + Verbose > Core, the doc needs surgery
4. **Identify redundancy** — Flag sections that say the same thing differently

## The Rebalancing Workflow

### Step 1: Preserve — Never Delete Blindly

Before compressing anything:
- Read the full document end-to-end
- Identify every *unique insight* — the non-obvious warning, the subtle gotcha, the hard-won lesson
- These survive any compression; they just get relocated or reformatted

### Step 2: Extract the Core Skeleton

Write a mental (or literal) outline of what the document *should* communicate, in order of importance. This becomes the target structure.

### Step 3: Compress Verbose Sections

Apply these techniques in order of impact:

#### Consolidate DO/DON'T Lists

```
BEFORE (8 lines):
❌ Don't use var for constants
❌ Don't use var when let works
❌ Don't declare mutable variables unnecessarily
❌ Don't use var if the value never changes
✅ Do use let for values that don't change
✅ Do prefer let over var
✅ Do default to let
✅ Do use var only when mutation is required

AFTER (2 lines):
- Prefer `let` over `var` — use `var` only when mutation is required
- Default to immutability; the compiler will tell you when you need `var`
```

**Pattern**: When multiple items express the same rule from different angles, collapse to one clear statement + one concrete signal.

#### Collapse Repetitive Code Examples

```
BEFORE: 4 code blocks showing the same pattern with minor variations

AFTER:
[One canonical example]
Variations: use `TypeB` / `TypeC` / `TypeD` in place of `TypeA`.
Edge case: when `TypeD`, also set `flag = true`.
```

**Pattern**: Keep the single best example. Describe variations in a sentence or table row, not another full code block.

#### Tables Over Prose for Structured Information

If a section lists items with consistent attributes (name, description, when-to-use, gotcha), convert to a table. Tables compress 3–5× vs. prose with headers.

#### Inline Edge Cases With Their Parent Concept

Instead of a separate "Edge Cases" section at the bottom:
- Attach each edge case as a sub-bullet or `> [!NOTE]` block under the concept it relates to
- This keeps context local and eliminates a sprawling appendix

### Step 4: Restructure for Progressive Disclosure

After compression, organize remaining content:

```
# Title + one-line purpose

## Quick Reference (the 80% case)
   - Core rules, primary workflow, defaults

## Detailed Instructions
   - Step-by-step with inline edge cases

## Reference (lookup, not reading)
   - Tables of options/flags/parameters
   - Anti-pattern catalog
   - Troubleshooting
```

The reader should be able to stop at "Quick Reference" and be productive. Everything below is depth, not breadth.

### Step 5: Final Pass — Tighten Prose

- **Cut hedging**: "It might be worth considering" → "Consider"
- **Cut filler**: "It's important to note that" → (delete, just state the thing)
- **Active voice**: "The file should be saved" → "Save the file"
- **Merge short sections**: If two H3s are each 2–3 lines, combine or promote to bullets

## Anti-Patterns — What NOT to Do

### Skeleton Summary
Don't reduce a document to a bare outline with no examples. Examples teach; they just shouldn't dominate. **Keep at least one canonical example per major concept.**

### Lossy Compression
Don't silently drop subtle warnings or non-obvious gotchas because they're "edge cases." Compress the format, not the insight. If a warning is genuinely important, it stays — just as a tighter sentence, not a paragraph.

### Over-Tabling
Don't convert narrative explanations ("here's *why* we do this") into tables. Tables are for structured data. Motivation and rationale belong in prose.

### Uniformity Trap
Don't make every section the same length. A critical concept deserves more space than a minor one. Balance means proportional to importance, not equal.

## Output Checklist

After cleanup, verify:

- [ ] No unique information was lost (insights preserved, format changed)
- [ ] Core content is ≥50% of the document
- [ ] Every code example earns its space (no near-duplicates)
- [ ] DO/DON'T lists have ≤1 entry per distinct rule
- [ ] Edge cases live next to their parent concept, not in a distant appendix
- [ ] The document can be understood by reading only the first section
- [ ] Section headings form a scannable table of contents
- [ ] Prose is active voice, no filler phrases
- [ ] Format matches audience (terse/structured for LLM, narrative for human)

## Working With the User

1. **Show the diagnostic first** — Present the skeleton + tier classification before proposing cuts
2. **Propose, don't surprise** — Show what you plan to compress and why; let the user veto
3. **Offer a diff** — When possible, show before/after for the most significant compressions
4. **Preserve originals** — If the user is nervous, suggest keeping the original as `[filename].bak.md`
