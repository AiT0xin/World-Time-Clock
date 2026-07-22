# Patek Philippe Ref. 5110 J — World Time, live

A single-file SVG rendering of the yellow-gold Patek Philippe 5110 *Heure Universelle*,
running on atomic time with a fully working Cottier world-time complication.

The watch head only — no lugs, no strap, no caption. The live readout (home city, offset,
date, sync source) goes to the document title instead of the page.

```bash
python3 -m http.server 8000 --directory ~/patek-worldtime-clock
```

or, in Claude Code: launch config `patek-clock`.

## The complication

* **City ring** — the 24 engraved zones, one per hour of longitude, in the classic Patek
  order (London 0 → Auckland +12, Midway −11 → Azores −1). The ring rotates so the
  **home city sits at 12 o'clock**, exactly as pressing the pusher does on the watch.
  Names alternate between two radii — even slots on the outer track, odd on the inner —
  which is how the real dial fits "L. ANGELES" and "MOSCOW" next to each other.
* **24-hour ring** — each numeral shows the *true* current hour in the city it sits under,
  read from the IANA database via `Intl`, so daylight saving is real. The whole ring drifts
  continuously through the hour rather than jumping. **12 and 24 are never printed**: the
  gold sun star and the crescent moon occupy those two slots, as on the watch.
* **Day / night** — the silver / charcoal band and its sun and moon rotate with home local
  time. Each numeral owns a half-slot either side, so day runs 06:30 → 18:30.
* **Centre hands** — gold leaf hands showing home local time.

Because real offsets are used rather than the engraved nominal ones, DST shows honestly:
in July, Cairo and Moscow both read the same hour, and Anchorage skips a numeral. That is
the truth of the zones, not a rendering bug.

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
backoff on failure. The caption reports the winning source and the half-RTT uncertainty.

`?at=2026-07-23T14:05:00Z` freezes the watch at a given instant for testing.
