# CLAUDE.md

Working notes for this repo. Read before editing.

## What this is

A single-page, offline-capable itinerary for a **5-family road trip to Yellowstone + Grand Teton, July 17–26, 2026**. Deployed to GitHub Pages at https://kd-crafts.github.io/yellowstone-2026/

The audience is four other families who did **not** attend any planning discussion. They read it cold, mostly on a phone, often with no signal. Every design decision below follows from that.

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire itinerary. Self-contained: inline CSS, inline JS, no build step. Only external dep is Google Fonts. |
| `sw.js` | Service worker. Caches the page so it works offline in the parks. |
| `manifest.json` | Makes the page installable ("Add to Home Screen"). |
| `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | App icons. Referenced by `manifest.json` — don't rename. |
| `README.md` | Setup/hosting instructions for humans. |

---

## Hard rules — violating these breaks the site

### 1. Bump the cache version once per push — to one above upstream

`sw.js` line 2:
```js
const CACHE = 'trip-v4';   // ← at push time, set to one above origin/main
```

The rule is **not** "bump on every edit." Bump the `trip-vN` version **only when you're about to push**, and set it to **exactly one above what's live upstream**:

```bash
git show origin/main:sw.js | grep 'CACHE ='   # see the deployed version, then set local = that + 1
```

A multi-edit local batch stays on a single version — don't ratchet v5→v6→v7 as you go. If the working tree is already one above upstream, leave it.

**Why it matters:** the version is what forces already-installed phones to drop the old cached page and fetch your changes. Ship an `index.html` change with a stale (already-deployed) version and those phones keep serving the old copy — the change appears to do nothing.

### 2. Validate tag balance after every edit

`index.html` is one ~800-div blob. Unbalanced tags fail silently — the page renders, just wrong, often far from the edit.

```bash
python3 -c "
h=open('index.html').read()
import re
for t in ['div','iframe','ul','style','script']:
    o=len(re.findall(rf'<{t}[ >]',h)); c=h.count(f'</{t}>')
    print(f'{t:8} {o:>4}/{c:<4} {\"OK\" if o==c else \"MISMATCH\"}')
print('li      ', len(re.findall(r'<li[ >]',h)), h.count('</li>'))
"
```
All must match. (Note: naive `h.count('<li')` also matches `<link>` — use the regex above.)

**Reference counts as of last known-good state:** 815 div, 10 iframe, 12 ul, 26 li, 1 style, 2 script.

### 3. Never estimate drive times or distances

If a number isn't from Google Maps or a verified source, **don't invent one**. A wrong number here means five families sitting in a parking lot at the wrong hour.

This rule has already caught one real bug: Day 10 was published as "~430 mi · ~6.5 hrs" when Elko→Santa Clara is actually **~520 mi · ~8 hrs** (Elko→Reno alone is ~290 mi). If you can't verify, say so in the text rather than guessing.

**Two sources of truth that must agree:**
- The `day-stat` line at the top of each day
- The sum of the `tl-drive` segments inside that day's timeline

If you change one, change the other.

### 4. Segments must match their maps

Each day has **two** map URLs that must reflect the timeline's actual stop order:
- the `map-cta` button href (`google.com/maps/dir/?api=1&origin=…&waypoints=…`)
- the embedded `<iframe>` src (`maps.google.com/maps?saddr=…&daddr=…`)

Days 3–6 and 8 are **round trips** (origin == destination). This has been wrong before — Day 4's map once listed Tower Fall *before* Lamar Valley, backwards from the timeline.

---

## Content conventions

### Honest verdicts, with the reasoning

Stops aren't just listed — they're **judged**. If something isn't worth it, say so plainly and say why. The doc has a distinct voice: direct, specific, no hedging. Examples of the register:

> **The rule:** check the Great Fountain prediction at the visitor center. If it doesn't line up with a ~2:30pm arrival, drive straight past. **There is no version of this where waiting is the right call.**

> **Don't bother going inside for the view.** The outdoor back terrace has the same view and is open to anyone.

Don't soften these into "you may wish to consider." The whole value of the doc is that it makes calls.

### Write for a cold reader

No references to planning discussions, no "as we said," no "the group agreed." Explain what a thing *is* before saying whether to do it. Someone opening this for the first time at a trailhead should understand it.

### Mobile is the primary surface

Body text 16px, descriptions 14px, line-height 1.6, tap targets ≥44px. Breakpoints at 640px and 380px. Test narrow before wide.

---

## Deliberately excluded — do NOT re-add these

Each was considered and cut for a documented reason. If someone asks "why isn't X here," the answer is in the doc.

**Closed, not optional:**
- **Biscuit Basin** — hydrothermal explosion July 2024, still shut. Takes Sapphire Pool, Jewel Geyser, Black Diamond/Black Opal/Mustard/Wall Pool, and Mystic Falls (trailhead is inside the closed lot) with it.
- **Boiling River** — permanently gone. 2022 floods reshaped the riverbed; the site now sits ~3 mi from any road. **The only legal swim in Yellowstone is the Firehole Swimming Area (Day 3).**

**Time traps — cut on purpose:**
- **Firehole Lake Drive** (Day 3) — Great Fountain Geyser erupts every 9–15 hrs with a 2-hr prediction window. People "wait it out" and burn 90 min. Marked SKIP BY DEFAULT; only go with a confirmed prediction.
- **Virginia Cascade Drive** (Day 6) — 60-ft forested trickle, no platform, no parking. Meaningless after the 308-ft Lower Falls the same day. RVs turn it into a 5-mph crawl.
- **Moose-Wilson Road** (Day 8) — great moose odds, but **five cars cannot stay together on it**. One moose stops traffic both directions with no shoulders; cars 3–5 end up stranded around a blind curve with no cell service. Listed as optional, not in the timeline.
- **Leeks Marina** (Day 7) — removed entirely. Same lake-and-peaks view available at Colter Bay, Jackson Lake Lodge, and Signal Mountain the same day.
- **Winnemucca** (Day 10) — dropped. No Costco, nothing else to justify a stop.

**Marked optional, droppable without regret:**
- **Grand View + Inspiration Point** (Day 6) — the canyon payoff is Artist Point + Brink of the Lower Falls. Chasing every pullout causes fatigue, not better views.

---

## Known-good structure of a day block

```html
<div class="day-card" id="dN">
  <div class="day-header">…day-num, day-date, day-title, day-stat, badge…</div>
  <div class="day-body">
    <div class="geo-note">🧭 Where in the park…</div>     <!-- optional -->
    <div class="map-block">
      <a class="map-cta" href="…">🗺️ Open Day N route in Google Maps →</a>
      <div class="map-embed"><iframe src="…"></iframe></div>
      <div class="route-caption">…</div>
    </div>
    <div class="timeline">…tl-item blocks: tl-time / tl-name / tl-dur / tl-desc, alternating with tl-drive…</div>
    <div class="craters-box">…warning or explainer…</div>  <!-- optional, repeatable -->
    <div class="stops-section">…sublabel + ul.stops (optional/tip items)…</div>
    <div class="sleep-row">…lodging + directions link…</div>
  </div>
</div>
```

Timeline items alternate: **stop → drive → stop → drive → …** Every stop needs a time, a name, a duration, and a description. Every drive needs a duration and a route hint.

---

## Confirmed lodging — all booked, don't change

| Nights | Where |
|---|---|
| Jul 17 (1) | Quality Inn & Suites Twin Falls North, 1910 Fillmore Street North, Twin Falls ID |
| Jul 18–22 (5) | VRBO, 4778 Fir Road, Island Park ID — ~30 min to West entrance |
| Jul 23–24 (2) | Super 8 by Wyndham Driggs, 1361 North Highway 33, Driggs ID |
| Jul 25 (1) | Days Inn by Wyndham Elko, 1500 Idaho Street, Elko NV |

---

## Still soft — improve if you can

- **In-park drive times on Days 3–8 are estimates** (~30–35 mph effective, which is realistic for Yellowstone), not Google pulls. Days 1–2 and 9–10 use real numbers. Pinning down Days 3–8 with actual Google routes would be the single biggest accuracy win left.
- **Map button and map embed waypoints don't perfectly mirror each other** on a few days (e.g. Day 5). Both produce correct routes; they're just not identical lists.

## Deploy

Push to `main`. GitHub Pages serves from root. Live in ~1 min.
**Don't forget rule #1** — bump `sw.js` cache version or nobody sees the change.
