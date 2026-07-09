# Seed Breakout Map — self-updating website

A website that shows AI companies which started at seed and broke out ($100M+),
grouped by category. It **updates itself once a day** and holds still in between,
so the numbers change only when the market changes — not every time you click.

## How it works (the important idea)

There are three parts, and keeping them separate is the whole trick:

1. **`data/market_data.json`** — the saved list of companies. The single source of truth.
2. **`update_data.py` + the GitHub Action** — a robot that runs once a day, searches
   the market, and rewrites that file if something changed.
3. **`index.html`** — the website. It only *displays* the saved file. The "Reload"
   button re-reads the file; it does not re-guess. That is why it's stable.

The website never talks to the AI directly. The scheduled robot does, in the
background, once a day. That's what makes it track the market without wobbling.

## What you need

- A free GitHub account.
- An Anthropic API key (from https://console.anthropic.com). Each daily run makes
  two searches, so the cost is very small — but it is not zero, so use a key you're
  allowed to use.

## Setup — do these in order

**1. Make a new repository.**
On GitHub: New → name it `seed-breakout-map` → Public → Create.

**2. Add the files.**
Upload all four, keeping the folder structure exactly:
```
index.html
update_data.py
data/market_data.json
.github/workflows/update.yml
```
Easiest way: "Add file → Upload files," then drag them in. To recreate the folders
when typing manually, name a new file `.github/workflows/update.yml` and GitHub makes
the folders for you. Same for `data/market_data.json`.

**3. Add your API key as a secret** (never put it in the code).
Repo → Settings → Secrets and variables → Actions → New repository secret.
Name it exactly `ANTHROPIC_API_KEY` and paste your key as the value. Save.

**4. Run the robot once, by hand.**
Repo → Actions tab. If asked, click to enable workflows. Click
"Update market data" → "Run workflow" → Run. Wait ~1–2 minutes for the green check.
This fills `data/market_data.json` with real data.

**5. Turn on the website.**
Repo → Settings → Pages → Source: "Deploy from a branch" → Branch: `main`, folder
`/ (root)` → Save. After a minute GitHub shows your live URL
(like `https://yourname.github.io/seed-breakout-map/`). Open it.

That's it. From now on it refreshes itself every morning. You can also hit
"Run workflow" any time to update on demand.

## Changing things

- **When it runs:** edit the `cron` line in `.github/workflows/update.yml`
  (`0 11 * * *` is 11:00 UTC daily).
- **Which categories / rules:** edit the prompt and `GROUPS` list in `update_data.py`.
- **Which model:** add a repo variable `RADAR_MODEL` (default is `claude-sonnet-5`).

## Honest limits (say these when you present)

- The list is surfaced by AI web search, so it's a **directional map to verify**,
  not audited data. Confirm figures in PitchBook/Affinity before acting.
- To make it audited instead of estimated, point `update_data.py` at Glasswing's
  **PitchBook connector** rather than open web search — same architecture, better data.
- It updates daily, not by the second — which is correct, because "which seed
  companies have broken out" does not actually change second to second.

## If something breaks

- **Website says "no saved data":** the robot hasn't run yet, or it failed. Check the
  Actions tab for a red X, click it to read the error (usually a missing/invalid API key).
- **Action fails on the API step:** check the `ANTHROPIC_API_KEY` secret is set and valid.
- **Page didn't update after a run:** GitHub Pages can take a minute; hard-refresh the page.
