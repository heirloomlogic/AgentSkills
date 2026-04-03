# AI Writing Patterns Reference

This is the full catalog of AI writing patterns to detect and fix. Each pattern includes words to watch, the problem, and before/after examples.

## Table of Contents

- [Content Patterns](#content-patterns) (1-6)
- [Language and Grammar Patterns](#language-and-grammar-patterns) (7-12)
- [Style Patterns](#style-patterns) (13-18)
- [Communication Patterns](#communication-patterns) (19-21)
- [Filler and Hedging](#filler-and-hedging) (22-24)

---

## Content Patterns

### 1. Undue Emphasis on Significance, Legacy, and Broader Trends

**Words to watch:** stands/serves as, is a testament/reminder, a vital/significant/crucial/pivotal/key role/moment, underscores/highlights its importance/significance, reflects broader, symbolizing its ongoing/enduring/lasting, contributing to the, setting the stage for, marking/shaping the, represents/marks a shift, key turning point, evolving landscape, focal point, indelible mark, deeply rooted

**Problem:** LLM writing inflates the importance of everything. A regional statistics office doesn't "mark a pivotal moment in the evolution of" anything.

**Before:**
> The Statistical Institute of Catalonia was officially established in 1989, marking a pivotal moment in the evolution of regional statistics in Spain.

**After:**
> The Statistical Institute of Catalonia was established in 1989 to collect and publish regional statistics independently from Spain's national office.

---

### 2. Undue Emphasis on Notability and Media Coverage

**Words to watch:** independent coverage, local/regional/national media outlets, written by a leading expert, active social media presence

**Problem:** Name-dropping publications without saying what was actually said.

**Before:**
> Her views have been cited in The New York Times, BBC, Financial Times, and The Hindu. She maintains an active social media presence with over 500,000 followers.

**After:**
> In a 2024 New York Times piece, she argued that AI regulation should focus on outcomes rather than methods. It got the usual cycle of agreement, outrage, and misquotation.

---

### 3. Superficial Analyses with -ing Endings

**Words to watch:** highlighting/underscoring/emphasizing..., ensuring..., reflecting/symbolizing..., contributing to..., cultivating/fostering..., encompassing..., showcasing...

**Problem:** Present participle phrases tacked onto sentences to simulate depth. The -ing clause looks like analysis but says nothing.

**Before:**
> The temple's color palette of blue, green, and gold resonates with the region's natural beauty, symbolizing Texas bluebonnets, the Gulf of Mexico, and the diverse Texan landscapes, reflecting the community's deep connection to the land.

**After:**
> The temple uses blue, green, and gold. The architect said the colors reference local bluebonnets and the coastline.

---

### 4. Promotional and Advertisement-like Language

**Words to watch:** boasts a, vibrant, rich (figurative), profound, enhancing its, showcasing, exemplifies, commitment to, natural beauty, nestled, in the heart of, groundbreaking (figurative), renowned, breathtaking, must-visit, stunning

**Problem:** LLMs write like a tourism brochure left in a hotel lobby.

**Before:**
> Nestled within the breathtaking region of Gonder in Ethiopia, Alamata Raya Kobo stands as a vibrant town with a rich cultural heritage and stunning natural beauty.

**After:**
> Alamata Raya Kobo is a town in the Gonder region of Ethiopia, known for its weekly market and an 18th-century church.

---

### 5. Vague Attributions and Weasel Words

**Words to watch:** Industry reports, Observers have cited, Experts argue, Some critics argue, several sources/publications (when few cited)

**Problem:** Attributing opinions to nobody in particular.

**Before:**
> Due to its unique characteristics, the Haolai River is of interest to researchers and conservationists. Experts believe it plays a crucial role in the regional ecosystem.

**After:**
> The Haolai River supports several endemic fish species, per a 2019 Chinese Academy of Sciences survey.

---

### 6. Outline-like "Challenges and Future Prospects" Sections

**Words to watch:** Despite its... faces several challenges..., Despite these challenges, Challenges and Legacy, Future Outlook

**Problem:** Formulaic "Challenges" sections that acknowledge problems without saying anything about them.

**Before:**
> Despite its industrial prosperity, Korattur faces challenges typical of urban areas, including traffic congestion and water scarcity. Despite these challenges, with its strategic location and ongoing initiatives, Korattur continues to thrive as an integral part of Chennai's growth.

**After:**
> Traffic got worse after 2015 when three IT parks opened. The municipal corporation started a stormwater drainage project in 2022. Whether it works remains to be seen.

---

## Language and Grammar Patterns

### 7. Overused "AI Vocabulary" Words

**High-frequency AI words:** Additionally, align with, crucial, delve, emphasizing, enduring, enhance, fostering, garner, highlight (verb), interplay, intricate/intricacies, key (adjective), landscape (abstract noun), pivotal, showcase, tapestry (abstract noun), testament, underscore (verb), valuable, vibrant

**Problem:** These words appear in post-2023 text at rates that would embarrass a thesaurus. They co-occur in clusters like a fingerprint.

**Before:**
> Additionally, a distinctive feature of Somali cuisine is the incorporation of camel meat. An enduring testament to Italian colonial influence is the widespread adoption of pasta in the local culinary landscape, showcasing how these dishes have integrated into the traditional diet.

**After:**
> Somali cuisine includes camel meat, which is considered a delicacy. Pasta, introduced during Italian colonization, is still common in the south.

---

### 8. Avoidance of "is"/"are" (Copula Avoidance)

**Words to watch:** serves as/stands as/marks/represents [a], boasts/features/offers [a]

**Problem:** Using four words where one works fine.

**Before:**
> Gallery 825 serves as LAAA's exhibition space for contemporary art. The gallery features four separate spaces and boasts over 3,000 square feet.

**After:**
> Gallery 825 is LAAA's exhibition space. Four rooms, 3,000 square feet.

---

### 9. Negative Parallelisms

**Problem:** "Not only...but..." and "It's not just about..., it's..." are overused by LLMs. But note: the construction isn't inherently bad. Flag it when it's padding. Leave it when it's doing real rhetorical work.

**Overuse (cut it):**
> It's not just about the beat riding under the vocals; it's part of the aggression and atmosphere. It's not merely a song, it's a statement.

**Fixed:**
> The heavy beat drives the aggression. That's the point of the track.

**Acceptable use (leave it alone):**
> The problem isn't that the tests are slow. The problem is that nobody runs them.

---

### 10. Rule of Three Overuse

**Problem:** LLMs force ideas into groups of three. But three items in a list is sometimes correct. Flag it when the third item is padding or when multiple triples appear in sequence.

**Overuse (cut it):**
> The event features keynote sessions, panel discussions, and networking opportunities. Attendees can expect innovation, inspiration, and industry insights.

**Fixed:**
> The event has talks and panels, plus time for networking between sessions.

**Acceptable (leave it alone):**
> The API accepts JSON, XML, and CSV.

---

### 11. Elegant Variation (Synonym Cycling)

**Problem:** AI has repetition-penalty code causing excessive synonym substitution. Repeating a word is fine. It's often clearer.

**Before:**
> The protagonist faces many challenges. The main character must overcome obstacles. The central figure eventually triumphs. The hero returns home.

**After:**
> The protagonist faces many challenges but eventually returns home.

---

### 12. False Ranges

**Problem:** "From X to Y" constructions where X and Y aren't on a meaningful scale.

**Before:**
> Our journey through the universe has taken us from the singularity of the Big Bang to the grand cosmic web, from the birth and death of stars to the enigmatic dance of dark matter.

**After:**
> The book covers the Big Bang, star formation, and current dark matter theories.

---

## Style Patterns

### 13. Em Dash Overuse

**Problem:** LLMs overuse em dashes. But em dashes are a real punctuation mark that humans use. The rule: more than two in a paragraph is suspicious. One is fine. Two is fine. A paragraph that looks like a ransom note held together with dashes is not.

**Overuse (fix it):**
> The term is primarily promoted by Dutch institutions—not by the people themselves. You don't say "Netherlands, Europe" as an address—yet this mislabeling continues—even in official documents.

**Fixed:**
> The term is primarily promoted by Dutch institutions, not by the people themselves. You don't say "Netherlands, Europe" as an address, yet this mislabeling continues in official documents.

**Acceptable (leave it alone):**
> The API returns a 200 — even when the operation fails.

#### Em-dash closer (sales-pitch pattern)

**Problem:** The construction `statement — closer` reads like a late-night infomercial. "Detects what changed and persists it — no explicit save calls." This is the "yadda yadda — peace of mind, for you and your family" pattern. It shows up in commercials, ad copy, and Kickstarter pitches. Two sentences are almost always better.

**Sales pitch (fix it):**
> Your reducers mutate state, and Swidux detects what changed and persists it — no explicit save calls, no load/loaded action pairs, no persistence boilerplate in your feature code.

**Fixed:**
> Your reducers mutate state, and Swidux detects what changed and persists it. This removes the need for explicit save calls, load/loaded action pairs, and persistence boilerplate in your feature code.

**More examples of the pattern to watch for:**
- "Handles everything automatically — no muss, no fuss"
- "One command and you're done — that's it"
- "Built for speed — nothing else"

The tell: the clause after the em dash is short, punchy, and tells you how to feel about the preceding statement. It's copywriting, not documentation.

---

### 14. Overuse of Boldface

**Problem:** Mechanical bolding of every proper noun or acronym.

**Before:**
> It blends **OKRs (Objectives and Key Results)**, **KPIs (Key Performance Indicators)**, and visual strategy tools such as the **Business Model Canvas (BMC)** and **Balanced Scorecard (BSC)**.

**After:**
> It blends OKRs, KPIs, and visual strategy tools like the Business Model Canvas and Balanced Scorecard.

---

### 15. Inline-Header Vertical Lists

**Problem:** Lists where every item starts with a bolded header followed by a colon. The `- **Label:** description` format is one of the most recognizable AI writing tells. Choose the right replacement based on content density:

- **Few items, one sentence each:** Collapse to prose. "Faster queries, a cleaner REST API, and end-to-end encryption."
- **Multiple items, each with real substance:** Convert to heading hierarchy (`### Heading` + paragraph or sub-bullets). This is what human-written docs actually look like.
- **Many items, truly just a label + short value:** A plain list without bold headers. "- Faster queries" not "- **Performance:** Faster queries".

Do not output `- **Label:** description` lists. This format is the single most recognizable AI writing pattern. It does not matter that the input used it. Convert it to one of the three formats above.

**IMPORTANT: This rule is about AI-generated bold-bullet lists. Do NOT flatten an existing heading hierarchy (h2/h3 sections with paragraphs) into bold-bullet lists — that makes text *more* AI-looking, not less.** If the source has proper `### Heading` + paragraph structure, preserve it.

Also: if bold-term bullets are appropriate, use a colon after the term, not a period. "**Reducer purity:** Reducers are pure state transformations" — not "**Reducer purity.** Reducers are pure state transformations." A period makes the bold term a sentence fragment.

**Before:**
> - **User Experience:** The user experience has been significantly improved with a new interface.
> - **Performance:** Performance has been enhanced through optimized algorithms.
> - **Security:** Security has been strengthened with end-to-end encryption.

**After:**
> The update improves the interface, speeds up load times, and adds end-to-end encryption.

**Never do this (collapsing headings into bullets):**

If the original has:
> ### Optimistic UI
> State is updated synchronously in the reducer. Persistence happens in the background.
> ### Reducer Purity
> Reducers are pure state transformations. They return `Effect?` for async work.

Do NOT convert it to:
> - **Optimistic UI.** State updates synchronously. Persistence happens in the background.
> - **Reducer purity.** Reducers return `Effect?` for async work.

The heading structure is correct. Leave it alone.

---

### 16. Bad Title Case in Headings

**Problem:** AI capitalizes *every* word in headings, including articles and prepositions (And, The, Of, In). Standard title case — where major words are capitalized but articles/prepositions are lowercase — is normal and expected in technical documentation. Don't flatten it to sentence case.

**AI tell (fix it):**
> ## Strategic Negotiations And Global Partnerships

**Fixed (standard title case):**
> ## Strategic Negotiations and Global Partnerships

**Also acceptable (sentence case, if that's the document's existing convention):**
> ## Strategic negotiations and global partnerships

**Match the existing document's convention.** If the document already uses title case, keep title case. If it uses sentence case, keep sentence case. Don't impose a convention change as part of dehumanizing.

---

### 17. Emojis

**Problem:** Decorative emojis on headings or bullet points.

**Before:**
> 🚀 **Launch Phase:** The product launches in Q3
> 💡 **Key Insight:** Users prefer simplicity
> ✅ **Next Steps:** Schedule follow-up meeting

**After:**
> The product launches in Q3. User research showed a preference for simplicity. Next: schedule a follow-up.

---

### 18. Curly Quotation Marks

**Problem:** ChatGPT uses curly quotes ("\u2026") instead of straight quotes ("...").

**Before:**
> He said \u201cthe project is on track\u201d but others disagreed.

**After:**
> He said "the project is on track" but others disagreed.

---

## Communication Patterns

### 19. Collaborative Communication Artifacts

**Words to watch:** I hope this helps, Of course!, Certainly!, You're absolutely right!, Would you like..., let me know, here is a...

**Problem:** Chatbot correspondence pasted in as content.

**Before:**
> Here is an overview of the French Revolution. I hope this helps! Let me know if you'd like me to expand on any section.

**After:**
> The French Revolution began in 1789 when financial crisis and food shortages led to widespread unrest.

---

### 20. Knowledge-Cutoff Disclaimers

**Words to watch:** as of [date], Up to my last training update, While specific details are limited/scarce..., based on available information...

**Problem:** AI disclaimers about incomplete information left in text.

**Before:**
> While specific details about the company's founding are not extensively documented in readily available sources, it appears to have been established sometime in the 1990s.

**After:**
> The company was founded in 1994, per its registration documents.

---

### 21. Sycophantic/Servile Tone

**Problem:** Overly positive, people-pleasing language.

**Before:**
> Great question! You're absolutely right that this is a complex topic. That's an excellent point about the economic factors.

**After:**
> The economic factors are worth looking at.

---

## Filler and Hedging

### 22. Filler Phrases

**Before → After:**
- "In order to achieve this goal" → "To do this"
- "Due to the fact that it was raining" → "Because it rained"
- "At this point in time" → "Now"
- "In the event that you need help" → "If you need help"
- "The system has the ability to process" → "The system processes"
- "It is important to note that the data shows" → "The data shows"

---

### 23. Excessive Hedging

**Problem:** Over-qualifying everything. Note: some hedging is normal and human. "I think" and "probably" are fine. "It could potentially possibly be argued that it might" is not.

**Before:**
> It could potentially possibly be argued that the policy might have some effect on outcomes.

**After:**
> The policy probably affects outcomes.

---

### 24. Generic Positive Conclusions

**Problem:** Vague upbeat endings that say nothing.

**Before:**
> The future looks bright for the company. Exciting times lie ahead as they continue their journey toward excellence. This represents a major step in the right direction.

**After:**
> The company plans to open two more locations next year.
