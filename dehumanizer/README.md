# Dehumanizer

A writing skill that removes signs of AI-generated writing without making the output sound like a Wikipedia article or a sales pitch.

Forked from [humanizer](https://github.com/blader/humanizer). Same detection, different voice.

## Installation

Copy the skill into your agent skills directory:

```bash
cp -r dehumanizer/ .agent/skills/dehumanizer/
```

## What's different from the humanizer

| | Humanizer | Dehumanizer |
|---|---|---|
| **Detection** | 24 AI patterns | Same 24 patterns |
| **Aggression** | Strips aggressively — kills em dashes, hedging, rule of three | Recalibrated — flags overuse, leaves appropriate use alone |
| **Voice** | Performative, overwrought | Deadpan, economical, dry |
| **Output** | Tends toward encyclopedia entries | Matches the register of the input |

## 24 Patterns Detected

### Content Patterns

| # | Pattern | Trigger |
|---|---|---|
| 1 | Significance inflation | "marking a pivotal moment in the evolution of..." |
| 2 | Notability name-dropping | Listing publications without saying what was said |
| 3 | Superficial -ing analyses | "symbolizing... reflecting... showcasing..." |
| 4 | Promotional language | "nestled within the breathtaking region" |
| 5 | Vague attributions | "Experts believe..." (which experts?) |
| 6 | Formulaic challenges | "Despite challenges... continues to thrive" |

### Language Patterns

| # | Pattern | Trigger |
|---|---|---|
| 7 | AI vocabulary | "Additionally... testament... landscape... showcasing" |
| 8 | Copula avoidance | "serves as" instead of "is" |
| 9 | Negative parallelisms | "It's not just X, it's Y" (when padding) |
| 10 | Rule of three | Triple-stacking for fake comprehensiveness |
| 11 | Synonym cycling | "protagonist... main character... central figure... hero" |
| 12 | False ranges | "from the Big Bang to dark matter" |

### Style Patterns

| # | Pattern | Trigger |
|---|---|---|
| 13 | Em dash overuse | 3+ em dashes in a paragraph |
| 14 | Boldface overuse | Mechanical bolding of every noun |
| 15 | Inline-header lists | "**Label:** content" bullet points |
| 16 | Title case headings | "Strategic Negotiations And Partnerships" |
| 17 | Emojis | Decorating headings and bullet points |
| 18 | Curly quotes | Typographic quotes instead of straight quotes |

### Communication Patterns

| # | Pattern | Trigger |
|---|---|---|
| 19 | Chatbot artifacts | "I hope this helps! Let me know if..." |
| 20 | Cutoff disclaimers | "While details are limited in available sources..." |
| 21 | Sycophantic tone | "Great question! You're absolutely right!" |

### Filler and Hedging

| # | Pattern | Trigger |
|---|---|---|
| 22 | Filler phrases | "In order to", "Due to the fact that" |
| 23 | Excessive hedging | "could potentially possibly" |
| 24 | Generic conclusions | "The future looks bright" |

## Voice Anti-Patterns

The dehumanizer also flags performative writing patterns:

- Emotional processing ("I genuinely don't know how to feel")
- Performed vulnerability ("Here's what gets me...")
- Permission-seeking ("Can we just talk about...")
- Therapy-speak ("I want to hold space for...")
- Trend slang ("FOMO," "problematic," "I'm cooked")
- Conspiratorial intimacy ("Here's the thing nobody's saying...")

## Version History

- **1.1.0** — Restructured: moved patterns to reference file, removed generational labels, trimmed SKILL.md from 537 to 172 lines.
- **1.0.0** — Initial release. Forked from humanizer 2.1.1 with voice recalibration.
