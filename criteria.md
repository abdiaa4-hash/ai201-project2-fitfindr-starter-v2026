# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
My search_listings scores by literal keyword overlap between the query and each
listing's title/description/category/style_tags. It doesn't do any fuzzy or
semantic matching, so a query phrased very differently from how a listing
describes itself (e.g. using a synonym the listing never uses) could score
zero and miss, even though a human would call it a match. 4 of 5 leaves room
for exactly that kind of literal-matching miss without pretending the search
is smarter than it is.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
Unlike criterion 1, this isn't a judgment call about whether a match is good
enough — it's a direct code branch: `if not results:`. search_listings either
returns an empty list or it doesn't, and my loop checks that exact condition
before calling suggest_outfit. There's no fuzzy matching involved in the
branch itself, so this should hold every single time, regardless of how the
search's own matching quality varies.

---

## 3. Something about state

For 5 different queries that return at least one result, the `id` field of
`session["selected_item"]` matches the `id` of the listing dict actually
passed into `suggest_outfit()` — in 5 of 5 runs.

**Why this target:**
This is a direct assignment inside my own code (`session["selected_item"] =
results[0]`, then that same value is passed straight into `suggest_outfit`),
not a model decision, so there's no reason it should ever vary. If it fails,
it means state is being dropped or overwritten somewhere between steps, which
is exactly the kind of bug this criterion exists to catch.


---

## 4. Something about the fit card

For 5 different items, the generated fit card mentions the item's price
(as a dollar amount) somewhere in the text — in at least 4 of 5 runs.

**Why this target:**
My create_fit_card() prompt explicitly instructs the model to mention the
price once, but it's still a model call with some temperature, so there's a
real chance it paraphrases the price away entirely (e.g. "a steal" instead of
a number) on an unlucky run. 4 of 5 acknowledges that risk while still holding
the caption to a concrete, checkable standard rather than just "sounds nice."


---

## 5. Your choice

Given an empty wardrobe (`{"items": []}`), `suggest_outfit()` still returns a
non-empty string of general styling advice rather than an empty string,
an exception, or a request for the user's wardrobe — in 5 of 5 runs.

**Why this target:**
The spec explicitly calls out that unit 4 will trigger this path on purpose,
so I wanted a criterion that actually exercises it now rather than assuming
it works. It's checkable with a plain truthiness/length check on the return
value, and I already confirmed it behaves this way once by hand — this
criterion is about confirming that holds consistently, not just on one lucky
run.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
