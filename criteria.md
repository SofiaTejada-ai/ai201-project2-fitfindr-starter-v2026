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

My search is a plain keyword/substring match, not a model call, so some
phrasings of a query will miss a listing that a human would still consider a
match. 4 of 5 allows for that without excusing a pattern of misses.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**

This path never reaches the model — `suggest_outfit` and `create_fit_card`
are never called when the search comes back empty. The check is a plain `if`
statement (is the list empty?) that runs the same way every time, so there's
no randomness to account for, unlike criterion 1.

---

## 3. Something about state

In 5 of 5 tries, the `id` of the item stored in `session["selected_item"]`
matches the `id` of the `new_item` that `suggest_outfit` actually receives.

**Why this target:**

This is my own code passing a value through the session, not a model call, so
there's no randomness to account for. If the logic is correct, it should hold
every single time — a miss here would mean a real bug, not an unlucky run.

---

## 4. Something about the fit card

In at least 4 of 5 tries, the fit card mentions the item (a word from its
title or description) and states its price.

**Why this target:**

This one calls a model, so unlike criterion 3, there's real variation — the
model might occasionally phrase something in a way that drops the price or
gets vague about what the item is. 4 of 5 allows one off run without excusing
a pattern.

---

## 5. Your choice

In 5 of 5 tries, a query for an item type absent from the data (e.g. "a
spaceship suit") is handled the same way as any other empty search —
`search_listings` returns an empty list, and the agent stops with the same
message, rather than crashing or behaving differently.

**Why this target:**

Matching a query against the listings data is deterministic code, not a model
call, so there's no reason this should vary. A miss here would mean the
empty-case handling only works for some kinds of "no results," not all of
them — a real bug.

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