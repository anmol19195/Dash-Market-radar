# DASH Market Radar — auto-refresh runbook

This repo holds the DASH Market Radar dashboard. It is a single self-contained
`index.html` (no build step) deployed on Vercel from this repo, so **any commit
to the default branch redeploys the live site automatically.**

A scheduled Cowork task runs every 2 days at 08:00 IST and follows this file.
Editing this file changes what the task does — no need to touch the schedule.

## What each run must do

1. **Read the current dashboard.** `index.html` holds everything, including the
   JS data arrays near the bottom of the file. The ones that carry news are:
   `timeline` (Money & Moves) and `news` (What's New). Others — `indiaListed`,
   `indiaD2C`, `globalCos`, `customPlatforms`, `customBrands`, `customFit`,
   `watchlist`, `social`, `playbook`, `channels` — carry standing profiles and
   should be corrected only when a fact in them actually changed.

2. **Find what is new since the `Data as of` date** in the header (search the
   file for `Data as of`). Research the window from that date to today across:
   - India D2C footwear: Comet, Gully Labs, Neeman's, Solethreads/Mirza, Yoho,
     Thaely, 7-10, Bluorng, Aretto, Agilitas (Lotto, One8, Mochiko)
   - India listed: Bata, Metro Brands, Relaxo, Campus Activewear — quarterly
     results, leadership, strategy
   - India macro: BIS/QCO rules, PLI, Tamil Nadu manufacturing, GST, quick-commerce
   - Global majors: Nike, Adidas, Puma, On, Deckers/HOKA, Crocs, Birkenstock,
     Skechers, New Balance, ASICS — earnings and strategy
   - Custom / 3D-printed: Zellerfeld, Z1R0/SafeSize, Koobz, Fitasy, ARKKY,
     Carbon, Adidas 4D, Nike 3D, On LightSpray, Drifbolt, Vivobarefoot VivoBiome
   - Fit-tech and scanning: SafeSize, Volumental, Aetrex, FindMeAShoe
   - US tariffs and anything that moves footwear supply chains

3. **Write it in.** Add new entries to the top of `timeline` and `news`, matching
   the existing object shape and tone exactly. Prefer concrete numbers over
   adjectives. Never invent a figure or a source — if it cannot be confirmed,
   either leave it out or mark it clearly as unconfirmed. Trim the oldest `news`
   entries so the feed stays around 10–12 items; `timeline` may keep growing.

4. **Update the date stamps.** Replace every occurrence of the old date with
   today's, in the header stamp, the overview footer line, the page footer, and
   the `as_of` field in the JS.

5. **Verify before committing.** Extract the largest `<script>` block and run
   `node --check` on it. A syntax error means a blank dashboard, so do not
   commit unless it passes.

6. **Commit and push** to the default branch. Vercel redeploys on its own.
   Commit message: `Refresh market radar — <today's date>`.

7. **Report back** with a short summary of what changed and the live URL.
   If nothing material happened in the window, say so plainly, bump the date
   stamps, and commit anyway so the dashboard does not look stale.

## Rules

- Do not restructure the dashboard, change its design, or add sections.
- Do not add companies whose business is custom insoles — this radar is
  footwear-only. HILOS was removed deliberately; do not reintroduce it.
- The "OURS" badge in the channels table marks the DASH row. Leave it alone.
- Everything must trace to a real, citable source.
