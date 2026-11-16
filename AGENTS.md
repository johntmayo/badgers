# SNL Sketch Ranker — Agent Context

Multi-season sketch tracker (S51 + S52). Single-file app: `index.html` (HTML + CSS + JS, localStorage). No build step.

## Purpose

Rate every *real* sketch / monologue / musical / Weekend Update so a year-end Best Of list is easy to compile. Switch seasons with the S51 / S52 tabs in the sidebar.

## Data baselines

| Const | Season |
|---|---|
| `BASE_S51` | Season 51 episodes (`ep1` …) |
| `BASE_S52` | Season 52 episodes (`s52e1` …) |
| `SEASONS` | Sidebar season tabs |

Persistence key: `snl_ranker_v3` (migrates from `snl_seasons_v1`, `snl52_v2`, `snl51_v3`, etc.).

## What counts as ratable

**Include** (always):
- Cold opens (`cat: 'cold'`)
- Host monologue / stand-up monologue (`cat: 'live'`)
- Live sketches (`cat: 'live'`) — wiki color `darkred`
- Pre-tapes / digital shorts / fake commercials (`cat: 'pre'`) — wiki color `navy`
- Weekend Update desk + desk guests as **one** segment (`cat: 'update'`) — wiki color `#bf7b00`
- Musical performances (`cat: 'musical'`) — wiki color `green`

**Never include** (skip when parsing wiki lineups):
- Opening Montage / theme song / credits montage
- Next Episode Commercial Bumper / promos
- Band Shot / Another Band Shot
- In Memoriam
- Goodbyes / Goodnights / Closing Credits
- Cut for Time (unless the user explicitly asks to add them)

## How to add a new episode

1. Open the episode page on `snl.fandom.com` (date URL, e.g. `December_5,_2026`) → **Edit source**.
2. Copy the wikitable under `==Sketches and Musical Performances==` (and episode host/guest/date from the infobox).
3. Paste into chat and ask to add / fill the episode in `index.html`.
4. Agent should:
   - Append or fill the right baseline (`BASE_S51` or `BASE_S52`): `{ id, ep, date, host, musical, sketches }`
   - Season 52 ids: `s52eN` / sketches `s52eN_K`. Season 51 ids: `epN` / existing sketch ids
   - Map wiki rows → sketch objects (see below)
   - **Filter out** non-ratable titles listed above
   - Bump nothing in localStorage schema unless the object shape changes (then bump `snl_ranker_v3` → `v4` and migrate)
   - Keep sketch `id`s stable once shipped — never renumber existing IDs

### Wiki color → `cat`

| Wiki color | `cat` | Label |
|---|---|---|
| `grey` | `cold` | Cold Open |
| `navy` | `pre` | Pre-tape |
| `#bf7b00` | `update` | Weekend Update |
| `green` | `musical` | Musical |
| `darkred` | `live` | Live Sketch |
| `black` | — | Usually skip (montage, bumper, band, goodnights). Monologue is the exception → `live` |

### Sketch object shape

```js
{
  id: 's7_1',           // stable unique id
  title: 'Cold Open…',
  cat: 'cold',          // cold | pre | update | musical | live
  cast: 'Name, Name',   // comma-separated; extract from summary parentheses + With/Also/Cameo lines
  notes: '',            // user-editable; leave empty in BASE
  rating: '',           // letter grade '' | F | D- | D | D+ | C- | C | C+ | B- | B | B+ | A- | A | A+
  fav: false,
  summary: '…'          // short wiki-sourced description; read-only in UI
}
```

### Episode shell (before sketches land)

```js
{ id:'ep7', ep:'S52E07 (#1015)', date:'Dec 5, 2026', host:'Amanda Seyfried', musical:'Don Toliver', sketches:[] }
```

`ep` display label: `S52E## (#NNNN)` when the absolute episode number is known.

### Title conventions

- Cold opens: short name + “Cold Open” (e.g. `Post-Midterm Cold Open`)
- Monologues: `Monologue` or `Stand-Up Monologue` when the wiki emphasizes stand-up
- Musicals: `Artist: "Song"` (medleys: `Artist: "A" / "B"`)
- Weekend Update: always title `Weekend Update` (one card for the whole desk block)

## UI modes (do not regress)

- **List** — every ratable sketch for the episode on one screen; sort by air order or letter grade
- **Cards** — one sketch at a time; swipe / arrows to move; grade in place
- **Best Of** — season-wide ranked list of graded / favorited sketches

Weekend Update rows are visually distinct (`cat: 'update'`). Non-ratable segments must never appear in List, Cards, or Best Of.

## Persistence

- Key: `snl_ranker_v3`
- Saves ratings, favorites, notes, cast edits, active season/episode, and user-added sketches for all seasons
- `BASE_S51` / `BASE_S52` are factory defaults; localStorage merges onto them on load
- On schema breaks, bump the key and migrate or accept reset

## Files

| File | Role |
|---|---|
| `index.html` | Entire app + season baselines |
| `AGENTS.md` | This file — ingestion + product rules for future agents |
| `snl_tracker_overview.md` | Older Season 51 design notes (partially superseded) |

## Design intent

Dense **broadcast rundown / box-score** UI — not magazine editorial. Compact type, tight rows, letter grades as the primary signal. Prefer information density over decorative chrome.
