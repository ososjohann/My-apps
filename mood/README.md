# Mood Check-in

A single-user, mobile-first PWA for a ~15-second nightly mood check-in. It stores
entries locally (works offline, nothing to log into) and exports **CSV rows that
append cleanly to `mood_tracker.csv`** in the Health + Fitness project.

Live app: static site in this `mood/` folder — deploy to Netlify (see below).

## What it does

- Tap **mood (1–7)**, **energy (1–5)**, **stress (1–7)**, **fullness (1–5)** — no sliders.
- **Social contact** as a 3×3 matrix — *type* (strangers / acquaintances / friends & family,
  incl. partner) × *amount of time* (None / Some <2h / A lot 2h+). All-None = an alone day.
- **Alcohol** as a drink-count stepper; **cannabis** as a None/Small/Medium/Large dropdown.
- **Three words for today** — added one at a time, with suggestions ranked by your own
  most-used words (free typing always allowed).
- Optional **health flag** (collapsed by default): quick symptom chips + a free-text detail,
  for the off days (GI, headache, etc.) a wearable can't catch.
- A **Trends** tab: a headline "biggest lever", a "what moves your mood & energy" driver
  table, mood/energy/stress over time, day-of-week, and mood/energy by contact type —
  all inline SVG (no chart library, fully offline).
- Free-text **note** (still here, alongside the three words).
- **Date defaults to the day that just ended**: before 5am local time it defaults to
  *yesterday* (you log at night, often after midnight). Always shown, always editable.
- Entries persist in `localStorage`; a **History** tab lists them (edit / delete /
  mark exported / clear exported).

## Export format (the important part)

**Schema v2.** The single `social` column was replaced by a richer set, and `words` +
health columns were added. The header is now:

```
date,mood,mood_label,stress,fullness,fullness_label,alcohol,alcohol_amount,cannabis,cannabis_amount,note,energy,energy_label,soc_strangers,soc_acquaint,soc_family,words,health_flag,health_tags,health_note
```

Everything up through `energy_label` is unchanged. New columns:
- `soc_strangers` / `soc_acquaint` / `soc_family` — amount of time with each type: `0` None,
  `1` Some (<2h), `2` A lot (2h+). All three `0` = an alone day.
- `words` — up to three day-summary words, comma-separated (CSV-quoted).
- `health_flag` — `0`/`1`; `health_tags` — semicolon-separated symptoms; `health_note` — free text.

The **Export** button (Export tab) emits rows in exactly this order, **without the
header**. `mood_label` / `fullness_label` / `energy_label` are auto-derived (mood 1 Terrible
… 7 Amazing; fullness 1 Starving … 5 Stuffed; energy 1 Drained … 5 Energized). `alcohol_amount`
is a drink count. Notes/words/health detail containing a comma/quote/newline are CSV-quoted.
Rows logged before v2 leave the new columns blank.

Example row:

```
2026-10-18,6,Great,3,4,Full,0,,0,,,4,Good,1,0,2,"productive, calm, sunny",0,,
```

- **Copy** → clipboard (paste into desktop chat). This is the primary path.
- **.csv** → downloads a file (with header) you can drop into the Health + Fitness folder.
- **Pipe** toggle → lightweight `date | mood | stress | fullness | alcohol | cannabis | note`
  format for quick pasting.

Export only includes **un-exported** entries, so you don't double-submit. After
exporting, tap **Mark pending as exported**.

## Import-back workflow (desktop)

Copy the CSV export → paste into a chat in the Health + Fitness project → rows are
appended to `mood_tracker.csv` (dedupe by date, keeping the per-row dates).

## Deploy to Netlify

Plain static site — no build step.

- **Continuous deploy:** connect the repo in Netlify and set the site's
  **publish directory** to `mood` (or base `mood`, publish `.`). Every push auto-deploys.
- **Manual:** `netlify deploy --prod --dir mood`.
- PWA install requires HTTPS — Netlify provides it. After deploy, open the site on your
  phone → *Add to Home Screen*, then confirm it opens offline.

`netlify.toml` in this folder sets the no-cache headers the service worker needs.

## Tech

Vanilla HTML/CSS/JS, `localStorage` only, no framework, no external runtime requests
(system font stack, inline SVG). Files: `index.html`, `sw.js`, `manifest.json`, icons.

## Cloud backup (Option B — not enabled)

v1 is localStorage-only (Option A). A commented hook for **Netlify Forms** cloud backup
is in `index.html` (search for "Netlify Forms hook") — enabling it gives a zero-backend
safety net so submissions also land in the Netlify dashboard as CSV.

## Tests

`../scratchpad`-style checks validate CSV escaping, label derivation, the after-midnight
date rule, and amount-clearing (brief §8 acceptance checks). All pass.
