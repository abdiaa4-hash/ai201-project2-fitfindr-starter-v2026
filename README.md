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

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

A user describes what they want in plain language (e.g. "vintage graphic tee
under $30, size M"). The agent searches a 40-item thrifted listings dataset
for matches, picks the best one, asks a model to suggest an outfit using the
user's wardrobe (or general advice if they have none on file), and writes a
short, ready-to-post caption mentioning the item, its price, and its platform.
If nothing matches the request, the agent stops and tells the user what to
change, instead of guessing or crashing.

---

## Tool Inventory

### `search_listings`

- **What it does:** Filters the 40 listings by price ceiling and exact size
  match, then scores the remainder by counting how many query keywords appear
  in the listing's title, description, category, and style tags.
- **Inputs:** `description` (str, free-text keywords), `size` (str or None),
  `max_price` (float or None).
- **Returns:** A list of listing dicts (id, title, description, category,
  style_tags, size, condition, price, colors, brand, platform), sorted by
  keyword score descending, capped at `config.SEARCH_RESULT_LIMIT` (10).
- **When it has nothing:** Returns `[]` (an empty list) — never `None`, never
  an exception.

### `suggest_outfit`

- **What it does:** Calls the model to suggest one or two outfits pairing the
  new item with the user's existing wardrobe.
- **Inputs:** `new_item` (a listing dict), `wardrobe` (a dict with an `items`
  key holding a list of wardrobe item dicts).
- **Returns:** A non-empty string of outfit suggestions in prose/markdown.
- **When it has nothing:** If `wardrobe["items"]` is empty, it still returns
  a non-empty string — general styling advice for the item instead of
  wardrobe-specific pairings. It never returns an empty string.

### `create_fit_card`

- **What it does:** Calls the model to write a short, casual social-media
  caption for the item, using the outfit suggestion as context.
- **Inputs:** `outfit` (str, from `suggest_outfit`), `new_item` (a listing
  dict).
- **Returns:** A 2-4 sentence caption string mentioning the item, its price,
  and its platform once each.
- **When it has nothing:** If `outfit` is empty or whitespace-only, it returns
  a plain descriptive string ("Found {title} (${price} on {platform}), but
  couldn't generate styling ideas for it.") instead of calling the model or
  raising.

---

## Planning Loop

**Branch rule:** If `search_listings` returns an empty list, put a message in
`session["error"]` explaining what the user could change (a higher price, a
different size, different keywords) and return the session immediately —
`suggest_outfit` and `create_fit_card` are never called. Otherwise, take
`results[0]` as the selected item and continue through both remaining tools.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex. One pattern pulls a price ceiling from
phrases like "$30" or "under 30"; another pulls a size token either from
"size X" or from a short list of common size words (XS/S/M/L/XL/XXL). Both
matched substrings are stripped out of the remaining text, which becomes the
keyword description passed to `search_listings`.

**What moves through the session:** `parsed` (description/size/max_price) →
`search_results` (full list from search) → `selected_item` (the chosen one)
→ `outfit_suggestion` (from `suggest_outfit`) → `fit_card` (from
`create_fit_card`). Any step can instead set `error` and end the run early.

---

## Sample Run

**One full query**