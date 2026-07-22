# World Time Clock — Live

A single-file SVG rendering of a yellow-gold world-time watch head running on atomic time
with a fully working Cottier-style world-time complication.

No branding, no caption on the dial. The live readout (home city, offset, date, sync source)
goes to the document title instead of the page.

```bash
python3 -m http.server 8000 --directory ~/worldtime-clock
```

or, in Claude Code: launch config `worldtime-clock`.

## The complication

* **City ring** — 24 time zones, one per hour of longitude. The ring rotates so the
  **home city sits at 12 o'clock**. Names alternate between two radii — even slots on the
  outer track, odd on the inner — so long names fit without overlapping.
* **24-hour ring** — each numeral shows that city's *true* current hour via `Intl` + IANA
  zones, so daylight saving is real. The ring drifts continuously through the hour rather
  than jumping. **12 and 24 are never printed**: the gold sun star and crescent moon occupy
  those two slots.
* **Day / night** — the silver / charcoal band and its sun and moon rotate with home local
  time. Each numeral owns a half-slot either side, so day runs 06:30 → 18:30.
* **Centre hands** — gold leaf hands showing home local time.

Because real offsets are used, DST shows honestly: Cairo and Moscow both read the same hour
in July, and Anchorage's numeral is skipped.

## Controls

| Action | Effect |
|---|---|
| Click the 10 o'clock pusher | advance home city one zone |
| Click any city name | make it home |
| `←` / `→` | previous / next home city |
| Triple-click the watch | toggle the dark backdrop |
| `B` | same toggle, from the keyboard |

The page background is transparent by default so it drops straight into a Notion embed or a
web page. Triple-clicking the case (not a city name — those stop the event) switches to the
`#191919` backdrop, same as the Reverso and Centigraphe clocks. Notion's iframe sandbox can
silently block `localStorage`, so the choice is written to **both** `localStorage` and a
cookie, each wrapped independently. Home city persists in `localStorage`.

## Atomic sync

Three independent sources are queried in parallel — `worldtimeapi.org`, `timeapi.io`, and
the serving origin's HTTP `Date` header (sub-second precision recovered by watching for the
second rollover across rapid HEAD requests). A source only wins if a second source agrees
within 1.5 s, so a drifted API gets outvoted. Re-syncs every 10 minutes, with exponential
backoff on failure. The title bar reports the winning source and the half-RTT uncertainty.

`?at=2026-07-23T14:05:00Z` freezes the watch at a given instant for testing.
