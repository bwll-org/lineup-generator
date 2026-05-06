# Lineup generator

**[See it in action](https://bwll-org.github.io/lineup-generator/)** — live demo on GitHub Pages.

Single-page tool for youth **rapid lineups**: enter game details and RSVP lists, then generate shuffled **field positions** and **coaching assignments** on printable sheets. Open `index.html` in a browser (no build step). Team logos live in `images/`.

## Lineup prefill form

The top card (hidden when you print) drives the printable pages.

| Field | Purpose |
|--------|--------|
| **Your Team** | Chooses the club for colors, header logos, and footer text. Blank option until you pick a team. |
| **Opponent** | Same team list plus a blank option; fills the **Opponent** line on both pages and the right header logo when selected. |
| **# of innings** | **3–6**. Sets how many inning columns appear on the roster and coaching tables, and updates the “*N*-Inning Rapid Lineup” subtitle. |
| **Date** / **Field** | Copied onto the info row on both printable pages (date is formatted for display). |
| **Players who RSVPed** | One name per non-empty line. These names are **shuffled**, then filled down the roster (up to **18** rows). |
| **Coaches who RSVPed** | One name per line. Used to fill the defensive and offensive coaching grids on page 2 (separate random assignments). |

**Randomize and Fill** shuffles players and coaches, fills opponent/date/field from the form (opponent comes from the opponent dropdown value), and writes positions into the roster inning columns. It does not change your team or inning count.

**Print** opens the browser print dialog. The prefill card uses the `no-print` class so only the two lineup pages are intended to print.

Editable cells on the PDF/sheet (opponent, date, field, roster, coaches) stay editable in the browser; `data-sync` keeps opponent/date/field aligned across both pages.

## How player positions are chosen

### 1. How many players count

Only **non-empty lines** in the players textarea are used. The tool uses `min(RSVP count, 18 roster rows)`. If fewer than **two** players are listed, inning cells are cleared and no position grid is built.

### 2. The label multiset (one set of “slots” per inning)

For a roster of size *N*, the app builds a fixed multiset of **exactly *N* position labels** for every inning. That multiset depends only on *N*, not on inning index.

| Players (*N*) | Labels used (in order they are added as *N* grows) |
|----------------|-----------------------------------------------------|
| 1–4 | First *N* of `P`, `1B`, `2B`, `SS`, `3B` |
| 5 | `P`, `1B`, `2B`, `SS`, `3B` |
| 6 | `P`, `C`, `1B`, `2B`, `SS`, `3B` |
| 7 | previous + `CF` |
| 8 | previous + `RF` |
| 9 | previous + `LF` (full field set in this order) |
| 10 | previous + one `BP` |
| 11 | previous + second `BP` |
| 12+ | 9 field labels + `BP`, `BP` + one `B` (bench) for each player over 11 |

- **BP** = bullpen (two slots only when *N* ≥ 11; one BP when *N* = 10).
- **B** = bench; only appears when *N* > 11. Multiple bench rows are all labeled `B`.

### 3. Assigning labels across innings

For each inning column, the app assigns a **permutation** of those *N* labels to the *N* players (each player gets one label per inning, each label used once per inning).

**Constraint:** a player must **not** receive the **same abbreviation** in two different innings (e.g. the same player cannot be `1B` in inning 1 and `1B` again in inning 3). Two different players **can** both be `B` in the same inning when there are multiple bench slots, because those are distinct roster rows; the constraint is per player, per label string, across time.

The solver shuffles processing order and uses **backtracking** to find a full assignment for that inning. It repeats for all innings (with many random restarts for the whole grid if needed). If no perfect grid is found after many tries, it falls back to a simpler random permutation per inning (a legacy safety net).

## How coach assignments are chosen

Page 2 has two tables (defensive roles and offensive roles), each with **three rows** (Coach A/B/C) and **N inning columns** (same *N* as **# of innings**).

For each inning, three coach names are drawn from your list (with rotation when there are at least three coaches), shuffled, and placed in the three rows. The algorithm tries to avoid giving the **same coach the same row twice** across later innings when a valid shuffle exists; with only two coaches or many innings, repeats can be necessary.

Defense and offense tables are filled with **independent** shuffles.

## Team theming

Choosing **Your Team** sets CSS variables for primary, accent, and page background colors and updates header/footer copy. Logos are loaded from `images/` (e.g. `images/pirates.jpg`). Opponent selection updates the matchup title and synced opponent text when applicable.

## Files

- `index.html` — UI, print layout, and all logic (Tailwind via CDN).
- `images/` — team JPEGs referenced by the team list in the script.

## License

Released under the [MIT License](LICENSE). You may use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the software for any purpose, subject to including the copyright and permission notice in substantial portions.
