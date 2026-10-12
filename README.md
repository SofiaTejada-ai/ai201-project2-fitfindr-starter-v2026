# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->

FitFindr is an agent that finds thrifted clothing and styles it for you. A user describes what they want in plain language — e.g. "vintage graphic tee under $30" — and the agent searches the listings, picks the closest keyword match, suggests an outfit combining it with the user's existing wardrobe (or general styling advice if they have none entered), and writes a short caption ready to post. If nothing matches the search, it stops and says what to change instead of guessing.

---

## Tool Inventory

### `search_listings(description, size, max_price)`
- **What it does:** Searches the listings data for items matching a text description, a size, and a maximum price.
- **Inputs:**
  - `description` (str) — free text describing what the user wants, e.g. "vintage graphic tee"
  - `size` (str, optional) — matched against a listing's `size` field as a case-insensitive substring, not an exact match, since the data's size formats are inconsistent (`"W30 L30"`, `"S/M"`, `"XL (oversized)"`) — a request for `"M"` matches a listing sized `"S/M"`
  - `max_price` (float, optional) — the highest price to allow
- **Returns:** a list of dicts, each the full listing record (`id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, `platform`) — the command line only prints title/price/description/size to the person, but the full dict is what's passed internally
- **Empty case:** an empty list `[]` — never `None`, never a crash

### `suggest_outfit(new_item, wardrobe)`
- **What it does:** Takes the new item and the user's wardrobe, and returns outfit ideas combining the new item with pieces already owned.
- **Inputs:**
  - `new_item` (dict) — the listing dict for the item just found
  - `wardrobe` (dict) — shaped `{"items": [...]}`, each item shaped like a wardrobe entry (`id`, `name`, `category`, `colors`, `style_tags`, `notes`)
- **Returns:** a list of plain-text outfit-suggestion strings, each describing one way to style the new item with existing wardrobe pieces
- **Empty case:** when `wardrobe["items"]` is empty, returns a list with one general-advice string based on the item's own category/colors/style_tags (e.g. "This graphic tee would pair well with dark jeans and sneakers for a casual look.") — not an apology, not an empty list

### `create_fit_card(outfit, new_item)`
- **What it does:** Writes a short, postable caption for the new item styled with the suggested outfit.
- **Inputs:**
  - `outfit` (list of str) — the suggestion(s) from `suggest_outfit`
  - `new_item` (dict) — the listing dict for the new item
- **Returns:** a single string — the caption text
- **Empty case:** if `outfit` is empty, raises a clear error (`ValueError("create_fit_card got an empty outfit — this should never happen if the pipeline is working correctly.")`) rather than failing silently or crashing with a cryptic stack trace — this state should never occur if the branch rule and `suggest_outfit`'s fallback are both working

---

## Planning Loop

**Branch rule:**

If `search_listings` returns an empty list, the loop puts a message in the session naming what to change (price or size) and stops — it does not call `suggest_outfit`. Otherwise, it takes the first result and continues to `suggest_outfit`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex, not the model — `agent.py::parse_query` pulls out a price with `\$(\d+(?:\.\d+)?)`, a size with `\bsize\s+([A-Za-z0-9/]+)`, and treats whatever's left (after stripping filler words like "looking for" and "under") as the description. Chosen over asking the model because parsing doesn't need judgment, it needs reliability, and a regex costs no API call and behaves identically every time. Known limit: a price only counts if it has a `$` — "under 30" is left in the description and no price filter is applied.

**What moves through the session:** `query` and `wardrobe` (set at the start), then `parsed` (the description/size/max_price dict), `search_results` (every match from `search_listings`), `selected_item` (the first result, the one that flows into `suggest_outfit` and `create_fit_card`), `outfit_suggestion`, `fit_card`, and `error` (set only when the loop stops early).

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**
$ python app.py ask '...'


**The three tools, tested one at a time**

```text
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'price': 18.0, ...}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'price': 24.0, ...}, ...]
```

```text
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Pair the vintage Levi's with your white ribbed tank top, the brown leather belt, and chunky white sneakers for an effortless, classic 90s off-duty model vibe. Alternatively, layer your oversized grey crewneck sweatshirt over the tank, cinch the jeans with the brown belt, and lace up the black combat boots for a cozy, grunge-streetwear look.
```

```text
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Found these vintage Levi's 501 jeans on depop for just $38.00 and they honestly fit like a dream. The medium wash has that perfectly broken-in streetwear look that usually takes years to achieve. Just styling them with my favorite white sneakers for an effortless weekend fit.
```


---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I tested `search_listings('a spaceship suit')`, expecting `[]` since nothing like it is in the data, and asked Claude why it returned 10 unrelated listings instead.
- *What came back:* The query tokenized to `{"a", "spaceship", "suit"}`, and "a" appears in almost every listing description — so every listing scored 1 and passed the `score == 0` check on a stopword match alone.
- *What I changed:* I added a small stopword set and a `_content_words()` helper used only for keyword scoring; size matching still uses the raw whole-token `_tokenize()`. The spaceship query now returns `[]`, and "vintage graphic tee" still returns tees first.

**Moment 2**

- *What I asked for:* A spec for `suggest_outfit` and `create_fit_card`, assuming they'd return lists of strings.
- *What came back:* When I pasted the actual stub file, both were already typed to return a single `str`.
- *What I changed:* I kept the stub's types instead of my original plan, since changing a given function signature wasn't worth it.

**Moment 3 (unit 4)**

- *What I asked for:* Help moving `search_listings` onto MCP, adding the trace and the error handler, and running both test runs.
- *What came back:* Claude found that both failed tries in the before run had the same cause: `generate()` didn't retry `500 INTERNAL` errors. It offered two possible fixes, retrying 5xx errors or rewriting the fit card prompt.
- *What I changed:* I picked the 5xx retry, because the fit cards were fine whenever they were generated, so the prompt wasn't the problem.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | FAIL | PASS | PASS | PASS | MET (4/5) |
| 2. Impossible query stops before tool 2 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Session selected_item id = id suggest_outfit receives | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card mentions the item and its price | 4 of 5 | PASS | FAIL | PASS | PASS | PASS | MET (4/5) |
| 5. Absent item type handled like any empty search | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

Diagnostic (not one of my five): empty wardrobe, `denim jacket under $50` — completed 5/5 with general styling advice, no crash.

Full output: [results/run_2026-10-11_2316_before.md](results/run_2026-10-11_2316_before.md)

**Real output from one try**, pasted as text, naming the file and function
that produced it:

**Criterion 1, try 2 (FAIL)**: `agent.py::run_agent`, the error path after `generate.py::generate` raised `ModelUnavailable` inside `tools.py::create_fit_card`:

```
[5] model
      →    unavailable: Couldn't reach the model: 500 INTERNAL. {'error': {'code': 500, 'message': 'Internal error encountered.', 'status': 'INTERNAL'}}
stopped early: yes — Found a listing, but couldn't reach the AI model to style it, so there's no outfit or fit card this time. ...
```

**Criterion 2, try 1 (PASS)**: `agent.py::run_agent`, empty branch:

```
[1] parse_query
      in:  designer ballgown size XXS under $5
      out: dict with keys: description, size, max_price
[2] search_listings (MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    branch: empty, stopping before suggest_outfit
```

**Criterion 3, try 1 (PASS)**: `agent.py::run_agent`. `session["selected_item"]` is `Denim Jacket — Light Wash, Cropped ($42.0, poshmark)`, and that same dict object is what's passed to `suggest_outfit(session["selected_item"], wardrobe)`.

**Criterion 4, try 1 (PASS)**: `tools.py::create_fit_card`, item `Platform Mary Janes — Black Patent ($55.0)`:

```
Channeling serious 90s mall-goth energy with these chunky platform Mary Janes. Score these beauties for just $55.00 off Depop and complete your ultimate Y2K skater rotation. They add just the right amount of attitude to any baggy denim fit.
```

**Criterion 5, try 1 (PASS)**: `agent.py::run_agent`, query `a spaceship suit`:

```
No listings matched. Try a higher price limit, a different size, or more general keywords.
```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Matching query completes all three tools | 4 of 5 | MET (4/5) | 4 tries returned a fit card. Try 2 stopped with no card. 4 passes against a target of 4. |
| 2 | Impossible query stops before tool 2 | 5 of 5 | MET (5/5) | All 5 traces end at step 2 (`search_listings` returned `[]`), and `suggest_outfit` never appears. |
| 3 | Session `selected_item` id = id `suggest_outfit` receives | 5 of 5 | MET (5/5) | In all 5 tries, `run_agent` passes the `session["selected_item"]` dict itself to `suggest_outfit`, so the ids can't differ. Every try logged the same item (Denim Jacket — Light Wash, Cropped). |
| 4 | Fit card mentions the item and its price | 4 of 5 | MET (4/5) | I checked each card for "$55" and a word from the title. 4 cards had both. Try 2 had no card at all, so it counts as a FAIL. |
| 5 | Absent item type handled like any empty search | 5 of 5 | MET (5/5) | All 5 tries of `a spaceship suit` gave `[]` and the same "No listings matched" message as criterion 2. |

**Diagnoses**

Nothing was MISSED, but criteria 1 and 4 each lost a try, and both failures had the same cause.

- **Where:** the model's output, not my tools, branch, or session. Criterion 1 try 2 failed inside `create_fit_card`. Criterion 4 try 2 failed earlier, inside `suggest_outfit`.
- **Mechanism:** Gemini returned `500 INTERNAL` (a temporary server error). `generate.py::generate` only retried 429 rate-limit errors. Every other error was raised as `ModelUnavailable` on the first attempt, so a one-off server hiccup was handled the same way as a bad API key. My handler in `run_agent` caught it and showed a message, so nothing crashed, but the user got no fit card.
- **Pattern:** two failures in different tools with one cause. It's one problem: how model errors are retried. That's what I fixed.
- **Were my targets too low?** Probably. I set 4 of 5 for criteria 1 and 4 to allow for the model wording things badly. But the failed tries weren't wording problems. They were the service failing, and 4 of 5 let that pass as a "MET". If I wrote these again, I'd make criterion 1 5 of 5, since one in five runs giving the user nothing isn't acceptable.



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**: `vintage graphic tee under $30` (from the before run, criterion 1 try 1). Step [2] is the MCP call.

```
[1] parse_query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    branch: 10 results, continuing
[3] select_item
      in:  10 results
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
      →    took the first (best) match
[4] suggest_outfit
      in:  dict with keys: new_item, wardrobe_items
      out: Pair the butterfly baby tee with your baggy straight-leg dark wash jeans and chunky white sneakers for an effo…
[5] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channeling total early 2000s pop princess energy with this cute little butterfly baby tee. Scored it in pristi…
```

**Empty search**: `designer ballgown size XXS under $5`. It stops after step 2.

```
[1] parse_query
      in:  designer ballgown size XXS under $5
      out: dict with keys: description, size, max_price
[2] search_listings (MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    branch: empty, stopping before suggest_outfit
```

**On the MCP move:** I moved `search_listings` behind MCP. `mcp_server.py` registers it with typed inputs (`description: str`, `size: str | None`, `max_price: float | None`) and a description that says the price is in US dollars and that no matches returns `[]`. In `agent.py::run_agent`, the direct call is now `call_tool("search_listings", {...})` from `mcp_client.py`. The results didn't change: `vintage graphic tee under $30` still returns the same 10 listings in the same order, and the empty case still comes back as `[]`, so the branch works without any changes. The one difference is speed. Each call starts the server process, so search is a bit slower than before.

**Failure modes, triggered on purpose:**

- **Empty search** (`spaceship suit under $10`): "No listings matched. Try a higher price limit, a different size, or more general keywords." This already worked before this unit.
- **Empty wardrobe** (`denim jacket under $50 --empty-wardrobe`): returned general styling advice and a full fit card. No crash and no empty string. This also already worked.
- **Model unavailable** (one character of `GEMINI_API_KEY` changed): before this unit it would have shown a raw error. I added a `ModelUnavailable` handler in `run_agent`, and now it says: "Found a listing, but couldn't reach the AI model to style it, so there's no outfit or fit card this time. Check that the API key in .env is correct and that you're online, then try again in a minute. Your top match was: Suede Chelsea Boots — Tan ($44.0)." The user still gets the search result and knows what to check.



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:** In `generate.py::generate`, a 500/503 ("internal" or "unavailable") error now gets the same backoff-and-retry as a 429 rate limit, instead of being raised as `ModelUnavailable` right away. It's about 5 lines, and nothing else changed between the two runs.

**Which failure it was meant to fix:** The only two failed tries in the before run (criterion 1 try 2 and criterion 4 try 2). Both were a single `500 INTERNAL` from Gemini that ended the run without a retry.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before tool 2 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Session selected_item id = id suggest_outfit receives | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card mentions the item and its price | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Absent item type handled like any empty search | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

Full output: [results/run_2026-10-11_2319_after.md](results/run_2026-10-11_2319_after.md)

**Did it help, and how do I know:** Probably yes, but I can't fully prove it. The after run passed 25 of 25 tries, up from 23 of 25. Criteria 1 and 4 went from 4/5 to 5/5. During criterion 1 try 1, the console printed "service pushed back… attempt 1 of 5", which means the model threw an error, the new retry caught it, and the run finished. Without the fix, that error would probably have cost another try. The log doesn't show the error code, though, so I can't be sure it was a 500. Also, 500s are random, so the after run may just have had fewer of them. To really prove the fix works, I'd fake a 500 on purpose and confirm the run retries and finishes.



---

## What's Still Broken

No criterion is still MISSED, but these are real problems I found and didn't fix:

- **Five model errors in a row still crash the run.** When retries run out, `generate()` raises a plain `RuntimeError` ("Still rate limited…"), not `ModelUnavailable`, so my handler doesn't catch it and the user sees an error instead of my message. The fix is to raise `ModelUnavailable` there. I stopped because the unit allows one improvement, and this would be a second change.
- **Search matches the wrong words.** `black leather boots` returns "Platform Mary Janes — Black Patent" first, because the keyword match counts "black" the same as "boots". I'd give the item-type word (boots, tee, jacket) more weight. Criterion 4 still passed because it only checks that the card describes whatever item was picked, but a user wouldn't count that as finding boots.
- **My criterion 3 evidence is weaker than it should be.** The trace shortens dict inputs to "dict with keys: …", so it never prints the item id `suggest_outfit` received. I verified it from the code path, where the same dict object is passed. Next time I'd log the id as a string in the trace so it shows up in the output.
- **Targets:** criteria 1 and 4 at 4 of 5 were too easy (see Diagnoses). I'd raise criterion 1 to 5 of 5.



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**