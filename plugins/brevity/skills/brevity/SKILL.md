---
name: brevity
description: "Compress a supplied text (pasted or a file path) to its load-bearing minimum: remove filler, keep every claim, qualifier and scope. EXPLICIT INVOCATION ONLY: use only when the user names the skill; never auto-trigger on generic requests to shorten or tighten text, and never apply to your own chat replies."
metadata:
  author: "Jack Camerano"
  version: 1.0.0
  tags:
    - category/editing
    - domain/writing
    - domain/content
---

# Brevity: every word loaded with meaning

An editing pass that compresses a supplied text to its load-bearing minimum: remove every word that does not change meaning, and no more.

The hard part is the second clause. The value is protecting words that look decorative but are load-bearing: a hedge, a scope, a number, a turn of voice. **Cut filler, keep meaning.** A cut that drops meaning is deletion, not brevity.

## Invocation

**Explicit only.** Engage only when the user names the skill directly: "use brevity on this", "run brevity on this file", "brevity this", or the `/brevity` command. Do not auto-trigger on generic requests to shorten, tighten or trim text, and never apply this to your own conversational replies. It operates only on a text the user hands you.

## Scope

- **In:** any supplied text, pasted or a file path you read. Any language. Prose and short-form alike (commits, PRs, Slack, emails, summaries, docs, READMEs, reports).
- **Out:** fetching URLs, code and your own chat output. Text only.

## Procedure

1. **Ingest:** take the pasted text, or read the file path
2. **Classify register** → set the compression ceiling (below). State the call in one line
3. **Build the ledger:** internally extract the load-bearing atoms the rewrite must preserve. Terse, not narrated. See [references/taxonomy.md](references/taxonomy.md)
4. **Cut** filler up to the ceiling. See [references/taxonomy.md](references/taxonomy.md)
5. **Fidelity check:** verify every ledger atom survives. A dropped atom is restored, or surfaced in the cut-list for the user to confirm
6. **Deliver:** rewrite + at-risk cut-list + word-count delta
7. **Offer the dial:** invite harder or softer, per-section if useful

## Register → compression ceiling

Register sets an automatic ceiling on how hard you may cut. Always on, not a user choice. Infer it from signals, state it in one line, proceed. Ask only when genuinely ambiguous.

| Signals                               | Register                        | Ceiling                                        |
| ------------------------------------- | ------------------------------- | ---------------------------------------------- |
| Bullets, imperative, short, code refs | Commit / PR / changelog         | Aggressive                                     |
| @mentions, casual, short              | Slack / chat                    | Aggressive                                     |
| "Summary of…", recap                  | Summary / notes                 | Aggressive                                     |
| Salutation + sign-off                 | Email                           | Gentle: warmth and politeness are load-bearing |
| Headings, length, narrative           | Doc / article / report / README | Moderate: keep signposting and navigation      |

"Aggressive" never crosses the fidelity line. "Gentle" means tone is content here, not filler.

## Aggressiveness

Register sets the ceiling. Within it, default to an **assertive, fidelity-safe** cut. The shown cut-list is what licenses boldness: nothing is lost silently, so the user can restore.

Optional intensity shortcuts, dialed in conversation, not required upfront:

- `gentle`: conservative, light touch
- `standard`: the assertive default
- `ungaretti`: maximal, still fidelity-bound

The user steers per-section when useful ("full Ungaretti on the intro, softer elsewhere").

## Fidelity check + clarity override + "nothing to do"

**Fidelity check.** After cutting, scan: did any ledger atom vanish? A targeted scan, not a fresh re-derivation. If gone, restore it or surface it as a deliberate removal. The ceiling on aggression _is_ this check: cut until the next cut would drop an atom. Rigor scales with stakes: short-form gets an internal read-back; prose gets an explicit ledger and a shown cut-list; very long docs are chunked by section so the ledger is never held whole.

**Clarity override.** Some content stays full even when words could be cut, because compressing it harms comprehension or safety. Back off the ceiling for: security and risk warnings, irreversible-action confirmations and multi-step instructions where terseness invites misreading. Fidelity protects claims; this protects clarity. When a cut would create ambiguity, don't make it.

**Nothing to do.** If the text is already lean, say so ("already tight; only trivial trims available") and stop. **Word count is a consequence of removing filler, never a target.** Never invent or pad cuts to manufacture a reduction. A skill that always "achieves" a cut will damage good text.

## Verbatim-preserve list (keep the exact characters)

Distinct from the ledger. The ledger keeps _meaning_; this keeps the _literal text_. Never paraphrase or "tighten" these, even when wordy:

- Quotes (anything the author is quoting)
- Error messages, exactly as written
- Code, commands, anything in backticks
- Identifiers, function and variable names, file paths
- Numbers, versions, dates
- Proper nouns, citations, references
- Defined terms (a term the text explicitly defines)

Conflating the two rules corrupts a citation or rewrites a quote.

## Punctuation

Marks face the same test as words: keep what changes meaning, cut what only decorates. A mark is filler when the sentence reads the same without it.

Examples:

- Em dashes: usually a decorative pause. Use a colon, comma, period or nothing, unless the dash sets off a genuine aside or carries the author's voice
- Serial (Oxford) commas: cut where no ambiguity follows, keep the one case where dropping it merges two list items
- Comma splices and stacked clauses: split into shorter sentences, or drop the clause
- Scare quotes, exclamation marks, parenthetical asides: keep only when they change how the claim reads

Load-bearing punctuation stays: a comma that disambiguates, voice-carrying marks and the punctuation inside verbatim-preserve content, which is never altered.

## Emission discipline

Output tokens are expensive; the naive build emits the text three or four times (original, ledger, rewrite, diff). Build the cheap version:

- **Never re-print the source.** It is already in context.
- **Show cuts, not copies.** The change summary names what was removed, not two full versions side by side.
- **Keep the ledger internal and terse.** Reasoning scratch, mostly unemitted. Apply Ungaretti to the ledger too.
- **Summarize the bulk, enumerate only the risky.** Describe routine filler removal in grouped terms ("cut throat-clearing throughout"), never word-by-word; spell out individually only the at-risk cuts (a dropped qualifier, scope or voice) so they can be confirmed.

## Output format

The rewrite is shown only for pasted input. For a file edit the diff is the rewrite, so never reprint it; emit the change summary, delta and dial only. Count the delta exactly from both versions (e.g. `wc -w`), never estimate.

- **Short-form:** the rewrite (pasted) or a one-line note (file). Fold a terse "what changed" into the note ("cut throat-clearing and hedge-stacking, −22%"). No visible ledger
- **Prose / high-stakes:**
  - the rewrite, for pasted input only;
  - a short **"what changed"** summary, grouped by kind, not a per-word log ("removed throat-clearing and hedge-stacking; collapsed two rule-of-three lists; cut ceremonial transitions");
  - any **at-risk** cut called out separately with a one-clause confirm-this reason ("dropped 'in most cases': confirm the claim is still meant to be hedged");
  - word-count delta (`820 → 540 words, −34%`), reported as a result, never as a goal hit;
  - the dial offer.

## Delivery

Invocation is explicit and name-gated, so act directly. The host's edit-permission prompt and diff view are the confirmation and preview layer, so do not add a bespoke confirm step.

- **Pasted text** → return the rewrite in chat. It is the deliverable.
- **File path** → edit the file in place. The diff is the rewrite, so never reprint the rewritten text; surface only the cut-list and delta. If the user asks to preview without changing the file, return the rewrite in chat instead.

## Voice

Preserve the author's voice by default: it is load-bearing unless _also_ redundant. Offer a **neutralize** mode (plain dense prose, voice removed) only when asked. Never neutralize silently.

## Anti-patterns

- Padding or inventing cuts to hit a word count
- Refusing to return "already lean" when the text is genuinely tight
- Paraphrasing anything on the verbatim-preserve list
- Dropping a hedge, qualifier or scope and calling it concision
- Re-printing the full original alongside the rewrite
- Narrating the full ledger to the user
- Invoking or depending on another skill: brevity runs standalone
- Touching your own chat replies or running unprompted
- Sanding off the author's voice without being asked
- Dropping articles, breaking grammar, reducing prose to telegraphese. Concision preserves grammaticality and register. This is the line between `brevity` and a blunt token-stripper. Output must read as natural, well-edited writing

For the full filler and protect taxonomies see [references/taxonomy.md](references/taxonomy.md); for worked before/after pairs see [references/examples.md](references/examples.md).
