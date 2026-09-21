# sidepocket-intel

Market data and AI briefings for the Sidepocket phone app.

This repo is **public on purpose**: the app reads
`https://raw.githubusercontent.com/MarketBoy809/sidepocket-intel/main/data/…`
with no authentication, and `raw.githubusercontent.com` will not serve a private
repo. Nothing personal lives here — prices, news links, sentiment scores and
market commentary only.

## How it runs

| What | Where | When |
|---|---|---|
| Collect prices, news, sentiment | This PC (Sidepocket Control) | every 15 min |
| Collect — fallback for when the PC is off | GitHub Actions | hourly at :07 |
| AI briefing | This PC, via Claude Code CLI | hourly |

There is **no API key and no OAuth token** anywhere in this repo. The briefing is
written by the Claude Code CLI signed in on the PC.

`sidepocket-intel.zip` in the root is the source of truth for the collector code;
both the PC app and the workflow unpack it on each run. Rebuild it with
`make-zip.py` — never with PowerShell's `Compress-Archive`, which produces an
archive GNU `unzip` rejects, failing the unpack step.

## Adding a mini-app

Apps appear in the hub's **Apps** tab. Add one by pushing two things:

1. A self-contained HTML file at `data/apps/<id>.html` — one file, inline CSS and
   JS, no external requests. It runs in a sandboxed iframe (`allow-scripts`, no
   same-origin), so it cannot read the hub, its settings or the phone.
2. An entry in `data/apps/manifest.json`:

```json
{
  "id": "ladder",
  "name": "Dip ladder",
  "icon": "↓",
  "tagline": "One line shown under the name."
}
```

Optional keys: `"tab": "sim"` opens a built-in view instead of a file;
`"external": true` with `"url"` hands off to the browser; `"file"` overrides the
default `<id>.html`.

No app rebuild is needed — push and it appears. Copy the `:root` colour variables
from an existing app so it follows light and dark mode.

## Asking the PC to run something

The phone cannot reach the PC directly, so the hub's **New briefing** and
**Collect data** buttons open a pre-filled GitHub issue titled `run: briefing` or
`run: collect`. Sidepocket Control polls for open issues **opened by the repo
owner**, runs the job and closes the issue with the result. Issues from anyone
else are ignored, and nothing in the issue text is ever executed — the title only
selects one of two fixed jobs.

## Data layout

```
data/latest.json             prices, 7d change, composite sentiment, top items, source health
data/briefing.json           the current AI briefing
data/items/recent.json       last 14 days of collected items
data/items/archive-*.jsonl   permanent monthly archive
data/prices/                 hourly and daily candles per asset
data/sentiment/              Fear & Greed and news tone history
data/history/                composite snapshots and past briefings
data/analysis/               what the history analyzer found
data/apps/                   mini-apps and their manifest
```

## Known source gaps

- **BLS** (CPI/jobs) returns 403 to every user-agent tried; CPI still arrives
  through news coverage.
- **CFTC** fails TLS from Windows because of a missing intermediate certificate,
  but works on the Linux Actions runner — one reason the hourly cloud run stays.
- **Reddit** serves only its `.rss` feed now, which carries no score, so those
  items rank on recency alone.
- **SEC** requires a contact e-mail in the User-Agent and 403s anything else.
