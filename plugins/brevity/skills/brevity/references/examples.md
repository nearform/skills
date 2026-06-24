# Worked pairs

Before/after pairs across registers. Each is annotated: register, ceiling, what was cut, what was protected and why, word delta. They double as the eval set: a correct implementation reproduces these calls.

The word delta is reported as a result, never as a goal. Pair 3 shows the case where the right delta is zero.

---

## 1. Doc paragraph: a load-bearing qualifier kept

**Before**

> It is worth noting that, in most cases, our caching layer will usually reduce latency for read-heavy workloads. We have found that, generally speaking, this tends to help quite a lot, though it is important to understand that results can vary depending on the specific access patterns you happen to have.

**After**

> In most cases our caching layer reduces latency for read-heavy workloads. Results vary with access patterns.

**Detected:** doc / article. **Ceiling:** moderate. **Intensity:** standard.
**Cut:** "It is worth noting that", "We have found that, generally speaking", "this tends to help quite a lot", "it is important to understand that", "the specific ... you happen to have". Also "usually", which only stacked on "in most cases".
**Protected:** "in most cases" stays. It looks like filler but it is the hedge that keeps the claim honest; without it the sentence promises latency always drops. "read-heavy workloads" (scope) and "vary with access patterns" (the conditional claim) stay.
**Delta:** 51 → 16 words, −69%.

---

## 2. Commit message: a verbatim element untouched

**Before**

> This commit basically just fixes the bug where the function getUserById() would sometimes throw `TypeError: cannot read property 'id' of undefined` when the user was not found, returning null instead of throwing. Closes #1423.

**After**

> Fix `getUserById()` throwing `TypeError: cannot read property 'id' of undefined` on missing user; return null instead. Closes #1423.

**Detected:** commit. **Ceiling:** aggressive. **Intensity:** standard.
**Cut:** "This commit basically just", "would", and the vague "sometimes" (the precise condition "on missing user" carries it).
**Protected, verbatim:** the identifier `getUserById()`, the exact error string `TypeError: cannot read property 'id' of undefined`, and the issue number `#1423`. None of these is paraphrased or tightened, even though the error string is long.
**Delta:** 34 → 18 words, −47%.

---

## 3. Slack message: already lean, no-op

**Before**

> deploying the hotfix to staging now, will ping when it's green

**After**

> deploying the hotfix to staging now, will ping when it's green

**Detected:** Slack. **Ceiling:** aggressive.
**Cut:** nothing. Every word is load-bearing: the action, the environment, the follow-up signal.
**Note returned to the user:** "already tight; only trivial trims available."
**Delta:** 0%. Padding this to manufacture a cut would damage it. The aggressive ceiling permits a hard cut; it does not require one.

---

## 4. Email: warmth is load-bearing (gentle register)

**Before**

> Hi Sara, I just wanted to quickly reach out and say thank you so much for taking the time to put together the report yesterday. I really think it was incredibly helpful and I genuinely appreciate all of the hard work that you clearly put into it. I was wondering if you might possibly have a moment at some point this week to walk me through the methodology section, as I would love to understand it a bit better. No rush at all of course. Thanks again, James

**After**

> Hi Sara, thank you for putting together the report yesterday, it was genuinely helpful and the work shows. Could you walk me through the methodology section sometime this week? I would love to understand it better. No rush. Thanks again, James

**Detected:** email. **Ceiling:** gentle. **Intensity:** standard.
**Cut:** the hedge-stacks ("I just wanted to quickly reach out and say", "might possibly have a moment at some point"), empty intensifiers ("so much", "really", "incredibly", "at all of course").
**Protected:** salutation and sign-off, "genuinely helpful", "I would love to", "No rush", "Thanks again". On a gentle ceiling, politeness and warmth are content, not filler; the rewrite is shorter but still reads as a warm note, not a curt demand.
**Delta:** 88 → 41 words, −53%.

---

## 5. Article intro: voice preserved through an aggressive cut

**Before**

> Look, I will be honest with you: for the longest time I genuinely believed that microservices were the answer to basically every single scaling problem under the sun. I drank the Kool-Aid, the whole entire pitcher of it, and I told anyone who would listen. Then we actually went ahead and tried it in production, and let me tell you, reality came knocking pretty hard.

**After**

> I will be honest: for the longest time I believed microservices were the answer to every scaling problem. I drank the Kool-Aid, the whole pitcher, and told anyone who would listen. Then we tried it in production, and reality came knocking.

**Detected:** article. **Ceiling:** moderate. **Intensity:** ungaretti (user-dialed).
**Cut:** "Look ... with you", "genuinely", "basically every single ... under the sun" → "every", "entire", "of it", "actually went ahead and", "let me tell you", "pretty hard".
**Protected:** the confessional voice. The Kool-Aid image and "reality came knocking" are the author's character, not decoration, so they stay verbatim. A cut this hard could have flattened the passage into a neutral summary; preserving those phrases is the difference between brevity and neutralizing.
**Delta:** 65 → 41 words, −37%.
