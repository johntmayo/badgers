# SNL Season 51 Sketch Tracker
**Developer Overview & Project Documentation**
_Last updated: April 2026 · Owner: John (Altagether / Supercaster)_

---

## 1. What Is This

A personal web app for tracking, rating, and reviewing every sketch from Saturday Night Live Season 51 (2025–26). The goal is to maintain a running ranked list of the best sketches of the season so that at year's end, a "Best of Season 51" list is easy to compile.

The app is a single self-contained HTML file with no backend, no build step, and no dependencies. It runs entirely in the browser.

---

## 2. Tech Stack

| | |
|---|---|
| **Format** | Single `.html` file — HTML + CSS + vanilla JS, no frameworks |
| **Storage** | `localStorage` — all data persists in the user's browser |
| **Hosting** | None required — open the file directly in any browser |
| **Dependencies** | Zero external dependencies or CDN calls |
| **Build process** | None — edit the file directly |

---

## 3. Data Model

All data lives in two JavaScript structures: a baseline const (`BASE_EPISODES`) and a mutable runtime array (`episodes`) that gets saved to localStorage.

### 3.1 Episode Object

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique ID, e.g. `'ep1'`. Used as stable key for localStorage. |
| `ep` | string | Display label, e.g. `'S51E01'` or `'S51E12 (#1000)'` |
| `date` | string | Human-readable air date, e.g. `'Oct 4, 2025'` |
| `host` | string | Episode host name |
| `musical` | string | Musical guest name |
| `sketches` | array | Array of sketch objects (see below) |

### 3.2 Sketch Object

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique ID, e.g. `'s1_1'`. Stable — never changes once set. |
| `title` | string | Sketch title, editable inline by the user |
| `cat` | string | Category key: `cold` \| `pre` \| `update` \| `musical` \| `live` |
| `cast` | string | Comma-separated cast members, editable inline |
| `notes` | string | User's personal notes, editable inline |
| `summary` | string | Brief wiki-sourced description (read-only flavor text) |
| `rating` | number | 0–5 stars (0 = unrated) |
| `fav` | boolean | Whether the user marked it as a favorite |

### 3.3 Sketch Categories

| Key | Label |
|---|---|
| `cold` | Cold Open |
| `pre` | Pre-tape / digital short |
| `update` | Weekend Update segment |
| `musical` | Musical performance |
| `live` | Live sketch (default) |

---

## 4. Rating System

Ratings follow a 1–5 scale:

| Stars | Meaning |
|---|---|
| ★ (1) | Downright offensive — the worst of the worst |
| ★★ (2) | Below average — didn't work |
| ★★★ (3) | Pretty good — nothing special _(the default for most sketches)_ |
| ★★★★ (4) | Really good — memorable, but short of the all-time pantheon |
| ★★★★★ (5) | Outright classic — all-time great sketch |

A rating of `0` means the sketch has not yet been rated. Unrated sketches do not appear in the Best Of view.

Clicking an already-selected star clears the rating back to 0.

The grades reflect the owner's opinion (and occasionally a second viewer's). Disagreements are rare and rarely exceed one point.

---

## 5. UI Structure

The app has two main views, toggled by the tabs in the top-right header.

### 5.1 Episode View

The default view. Shows all sketches for the selected episode as stacked cards. Each card includes:

- Editable title (click to type)
- Star rating (1–5, click to set, click again to clear)
- Rating label shown inline (e.g. "Pretty good — nothing special")
- Category dropdown
- Favorite toggle (♥ button)
- Editable cast field
- Editable notes field
- Summary (wiki-sourced — read-only italic text at bottom of card)
- Delete button (with confirmation)

At the bottom of the sketch list, a "+ Add sketch" button opens an inline form for adding new sketches manually.

### 5.2 Best Of View

Shows every rated or favorited sketch across all episodes, ranked by star rating (descending), then by favorite status as a tiebreaker. Each row shows rank number (gold/silver/bronze for top 3), sketch title, episode info, cast, notes, star rating, favorite badge, and category pill.

### 5.3 Sidebar

Lists all 15 Season 51 episodes. Each entry shows episode number, host, date, and a count of total sketches / rated / favorited. The active episode is highlighted with a gold left border.

---

## 6. Data Persistence

All user data (ratings, notes, cast edits, added sketches) is saved to `localStorage` under the key `snl51_v2`. Data is saved on every change — there is no explicit save button.

`BASE_EPISODES` is the factory default. On load, the app attempts to read from localStorage first. If nothing is saved, it falls back to a deep copy of `BASE_EPISODES`.

> **Important:** localStorage is browser-specific. Data does not sync across devices or browsers. If the user clears localStorage or uses a different browser, data is lost.

The localStorage key is versioned (`snl51_v2`) — bump to `v3` if the schema changes in a breaking way.

---

## 7. Loading Episode Sketch Data

Episode data is sourced from the SNL Fandom wiki (`snl.fandom.com`). The wiki uses a wikitext format where each sketch is a row in a wikitable. The owner pastes the raw wikitext source for each episode into a Claude chat session, which parses the data and outputs an updated HTML file with the episode's sketches pre-loaded.

### 7.1 Current Episode Status

- **Episode 1** (Bad Bunny / Oct 4, 2025): Fully loaded — 14 sketches with titles, cast, and summaries
- **Episodes 2–15**: Shell entries only — host and musical guest loaded, sketches empty

### 7.2 How to Add an Episode

1. Go to `snl.fandom.com/wiki/[Episode_Date]` (e.g. `October_11,_2025`)
2. Click "Edit source" to get the raw wikitext
3. Paste the wikitext into a Claude chat session alongside the current HTML file
4. Claude extracts sketch titles, cast, categories, and summaries and produces an updated HTML file

The relevant section in the wikitext is the wikitable under `==Sketches and Musical Performances==`. Each row follows the pattern:

```
{{lineup3|[color]|[Title]|[Image]|[Summary with cast]}}
```

---

## 8. Visual Design

| | |
|---|---|
| **Theme** | Dark sidebar (`#1A1A1A`) with light main area (`#F5F4F0` bg, white cards) |
| **Accent color** | Gold/amber (`#E8C84A`) — stars, favorites, active episode, top-ranked items |
| **Typography** | System font stack (`-apple-system, BlinkMacSystemFont, Segoe UI`) |
| **Card accents** | 5-star sketches get a gold left border; 4-star get green (`#A0C878`) |
| **Category colors** | Cold Open = blue; Pre-tape = green; Weekend Update = amber; Musical = pink; Live = purple |

---

## 9. Known Limitations & Future Ideas

### 9.1 Current Limitations

- No cloud sync — data is browser-local only
- No export — can't export ratings to CSV, JSON, or shareable format
- No search or filter in the Episode view
- Sketch order is fixed after loading — can't drag to reorder
- The `summary` field is not user-editable (by design — it's wiki-sourced flavor text)

### 9.2 Possible Future Enhancements

- Export to JSON or CSV for year-end list publishing
- Filter/search within the Best Of view by category or episode
- A "second reviewer" rating field so two opinions can be tracked independently
- Auto-load sketch data from the Fandom wiki via a scraper or API
- Move to a hosted app with a real backend if multi-device access becomes needed

---

## 10. File Reference

| File | Description |
|---|---|
| `snl_s51_tracker.html` | The entire app — open in any browser. Everything is in this one file. |
| localStorage key: `snl51_v2` | JSON array of episode objects with all user data |

---

_Questions? Reach John at john@supercaster.us_
