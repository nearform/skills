# Brevity taxonomy

Reference detail for `SKILL.md`. Two lists drive the work: what to protect (keep the meaning) and what to cut (filler). A third covers punctuation. None of these is a closed list. Patterns differ per language; the principle is universal: cut filler, keep load-bearing.

## Load-bearing atoms (protect: keep the meaning)

Before cutting, extract a terse internal checklist of everything the rewrite must honor. Atoms, not prose.

- **Claims:** each distinct assertion.
- **Qualifiers:** words that scope a claim. "usually" is not "always"; also "in most cases", "tends to", "often", "roughly".
- **Scopes and conditions:** "in Postgres 14+", "on Linux", "if X then Y".
- **Numbers, dates, quantities.**
- **Named entities:** people, products, tools, places.
- **The one concrete example** in an otherwise abstract passage.
- **Distinctive voice:** a turn of phrase carrying the author's character. Load-bearing unless also redundant.
- **Signposting:** in genuinely long or complex docs where the reader needs a map.

A cut that drops a ledger atom is deletion, not synthesis. That is the line.

Words that look cuttable but are not:

- "This _usually_ works" → drop "usually" and a hedge becomes a guarantee. Keep.
- "Supported _on Postgres 14+_" → drop the scope and the claim overreaches. Keep.
- "It returned _3_ errors" → the number is the content. Keep.

## Filler (safe to cut)

- **Throat-clearing and meta-narration:** "It's worth noting that", "In this section we will", "As an AI...".
- **Redundant restatement:** say it, then summarize what you just said.
- **Empty intensifiers and hedge-stacking:** "really very quite", "I think it might possibly". Common single-word offenders: just, really, basically, actually, simply, sure, certainly, of course, happy to.
- **Rule-of-three padding** where one item carries the point.
- **Ceremonial transitions:** "Furthermore", "Moreover", "It is important to understand that".
- **Restating the question** before answering it.

Examples:

- "It's worth noting that the API is rate-limited." → "The API is rate-limited."
- "I think it might possibly fail." → "It might fail." (one hedge is load-bearing; the stack is not)
- "We added caching, caching the results so they stay cached." → "We added result caching."

## Punctuation (same test: cut what only decorates)

A mark is filler when the sentence reads the same without it.

- **Em dashes:** usually a decorative pause. Use a colon, comma, period or nothing. Keep only for a genuine aside or the author's voice.
- **Serial (Oxford) commas:** cut where no ambiguity follows. Keep the rare case where dropping it merges two list items: "the lawyers, Bob, and Alice" (three parties) is not "the lawyers, Bob and Alice" (Bob and Alice are the lawyers).
- **Comma splices and stacked clauses:** split into shorter sentences, or drop the clause.
- **Scare quotes, exclamation marks, parenthetical asides:** keep only when they change how the claim reads.

Load-bearing punctuation stays: a comma that disambiguates, voice-carrying marks and the punctuation inside verbatim-preserve content.

Examples:

- "We shipped it — finally." → keep the dash if the relief is the point; otherwise "We shipped it."
- "Install the deps, run the build, and deploy." → "Install the deps, run the build and deploy." (no ambiguity, so the serial comma goes)
