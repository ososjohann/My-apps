# Mood Check-in

A single-user, mobile-first PWA for a ~15-second nightly mood check-in. It stores
entries locally (works offline, nothing to log into) and exports **CSV rows that
append cleanly to `mood_tracker.csv`** in the Health + Fitness project.

Live app: static site in this `mood/` folder — deploy to Netlify (see below).

## What it does

- Tap **mood (1–7)**, **energy (1–5)**, **stress (1–7)**, **fullness (2–5)** — no sliders.
- One-tap **caffeine** (none / before 2pm / after 2pm) and **social contact** (none / some / lots).
- Toggle **alcohol** / **cannabis**; a small/medium/large selector appears only when on.
- A **Trends** tab charts mood & energy over time and the correlations that matter —
  mood by social contact, energy by caffeine timing — plus rolling averages. All drawn
  as inline SVG (no chart library, still fully offline).
- Free-text **note**.
- **Date defaults to the day that just ended**: before 5am local time it defaults to
  *yesterday* (you log at night, often after midnight). Always shown, always editable.
- Entries persist in `localStorage`; a **History** tab lists them (edit / delete /
  mark exported / clear exported).
- Optional: 14-day mood sparkline and a "same as last check-in" prefill.

## Export format (the important part)

The header of the desktop file is:

```
date,mood,mood_label,stress,fullness,fullness_label,alcohol,alcohol_amount,cannabis,cannabis_amount,note,energy,energy_label,caffeine,social
```

The `energy` / `energy_label` / `caffeine` / `social` columns are **appended after
`note`** so every original column keeps its position — add these four headers to the
desktop `mood_tracker.csv` once; rows exported before they existed simply leave them
blank.

The **Export** button (Export tab) emits rows in exactly this order, **without the
header**, one per line. The `mood_label`, `fullness_label`, and `energy_label` columns
are auto-derived (mood 1 Terrible … 7 Amazing; fullness 2 Hungry … 5 Stuffed; energy
1 Drained, 2 Low, 3 OK, 4 Good, 5 Energized). `caffeine` is `none` / `before 2pm` /
`after 2pm`; `social` is `none` / `some` / `lots`. Amount columns are **blank** (not `0`)
when the flag is off. Notes containing a comma/quote/newline are CSV-quoted.

Example row:

```
2026-07-23,5,Good,5,4,Full,0,,1,large,released a substack post today!,4,Good,before 2pm,lots
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
