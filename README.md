# 🕐 Ticky Learns the Clock / Ticky lernt die Uhr

A free, bilingual (English/German), single-page web app that teaches kids how to
tell analog time — from recognizing the hour hand up to reading "twenty-five to
four" — through six short, playful levels plus an interactive explainer screen.

**Live app:** https://koroksengupta.github.io/ticktock/

No installs, no accounts, no ads. Everything runs entirely in the browser and
progress is saved on the child's own device.

---

## Table of contents

- [What it does](#what-it-does)
- [Levels](#levels)
- [The "How Clocks Work" explainer](#the-how-clocks-works-explainer)
- [Personalization](#personalization)
- [Tech stack](#tech-stack)
- [Getting started](#getting-started)
- [Project structure](#project-structure)
- [How the app is built](#how-the-app-is-built)
- [Internationalization (i18n)](#internationalization-i18n)
- [Progress & local storage](#progress--local-storage)
- [Deployment (GitHub Pages)](#deployment-github-pages)
- [Customizing / extending](#customizing--extending)
- [Browser support](#browser-support)
- [Accessibility notes](#accessibility-notes)
- [Support the project](#support-the-project)
- [License](#license)

---

## What it does

Ticky is a small, self-contained "duolingo-style" clock-reading trainer aimed at
early elementary kids (roughly ages 5–8). A child enters their name, picks a
level from a map of stepping stones, and drags the clock hands (or taps an
answer) to match a spoken/written prompt. Correct-on-the-first-try answers earn
stars; each level tracks its own best result so kids can replay and improve.

Key design goals:

- **Zero setup** — it's one `index.html` file. Open it in a browser and it works.
- **Bilingual from the first screen** — every string, prompt, and phrase
  ("quarter past four" / "Viertel nach vier") is available in English and German,
  switchable at any time with a `DE`/`EN` toggle.
- **Kid-friendly, not quiz-like** — no timers pressuring beginners, gentle sound
  effects, star bursts, a "training wheels" hint mode, and a mascot-style rounded
  visual language (Fredoka display font, pastel palette, soft shadows).
- **Actually teaches the mechanism**, not just quiz facts — see
  [The "How Clocks Work" explainer](#the-how-clocks-works-explainer) below.

## Levels

The level map unlocks progressively; each level takes 6 rounds and awards
1–3 stars based on how many rounds were answered correctly on the first try.

| # | Title (EN) | Titel (DE) | What it teaches |
|---|---|---|---|
| 1 | The Hour Hand Hero | Stunden-Helden | Drag the hour hand into the correct numbered "room" of the clock face. |
| 2 | The Half & Quarter Slice | Halb & Viertel | Set the clock to o'clock / quarter past / half past / quarter to, with a pie-slice visual of the elapsed minutes. |
| 3 | The 5-Minute Skip-Counter | 5-Minuten-Zähler | Set any 5-minute mark, with an optional "training wheels" number bubble near the minute hand. |
| 4 | Past or To? | Vor oder Nach? | Distinguish "X minutes past" vs. "X minutes to" the next hour, on a clock face visually split into two colored halves. |
| 5 | The Master Chronometer | Der Zeit-Meister | Given a start time and elapsed minutes, work out the new time — alternating between a hands-on clock round and a text-only "Minute Math" round that walks through the carry-the-hour arithmetic (`start + elapsed = total`, `total = 1 hour + remainder`) with multiple-choice answers. |

Each level has its own pastel color theme and, from level 3 onward, fine
5-minute tick marks on the clock face for extra precision.

## The "How Clocks Work" explainer

Before jumping into quizzes, a kid can open the always-unlocked **"How Clocks
Work" / "Wie die Uhr funktioniert"** tile at the top of the map. This screen:

- Explains the short **hour hand** and the long **minute hand** in plain
  language, with tap-to-highlight chips that glow the matching hand on the
  clock face.
- Lets the child freely **drag the minute hand** around a clock with no
  scoring and no snapping, while a live digital readout and the **hour hand
  visibly creeping forward** demonstrate — rather than just state — that the
  hour hand's position depends on how far the minute hand has traveled.

This was added specifically for kids who are just starting out and need the
underlying mechanism to click before the timed/scored levels make sense.

## Personalization

On first visit the app asks for the child's name (in whichever language is
currently selected) and then greets them throughout — "Mira Learns the
Clock" / "Mira lernt die Uhr" — including in the browser tab title. The name
is remembered locally, and can be changed again at any time via the "Change
name" link under the map header.

## Tech stack

Deliberately minimal:

- **Vanilla JavaScript** (ES5-leaning, no build step, no framework) — a small
  hand-rolled state machine that re-renders screens as HTML strings and
  re-attaches event listeners (`render()` / `attachHandlers()`).
- **Inline SVG** for the clock face, hands, pie slices, and the support QR
  code — no image assets, no external image requests.
- **Plain CSS** with custom properties (`:root` variables) for theming,
  including an automatic dark-mode palette via `prefers-color-scheme` /
  `data-theme`.
- **Google Fonts** (Fredoka for display text, Andika for body text) — the only
  external network dependency.
- **`localStorage`** for progress persistence — no backend, no database, no
  analytics, no cookies.
- **Web Audio API** for tiny procedural success/oops chimes (no audio files).

There is no `package.json`, no bundler, and no dependencies to install.

## Getting started

Because the whole app is one static HTML file, you can simply open it
directly:

```bash
git clone https://github.com/koroksengupta/ticktock.git
cd ticktock
open index.html        # macOS
# or: xdg-open index.html   (Linux)
# or: start index.html      (Windows)
```

For a closer-to-production experience (some browsers restrict certain APIs
on `file://` URLs), serve it over a tiny local HTTP server instead:

```bash
cd ticktock
python3 -m http.server 8000
# then open http://localhost:8000/index.html
```

No install step, no build step — edit `index.html` and refresh.

### Debug helpers

Appending `?debug=1` to the URL exposes a `window.__ticky_debug` object in
the browser console with references to internal state, level config, and
several pure helper functions (`generateRound`, `phraseFor`, `wrapHour`,
etc.), useful for poking at round generation or the clock-angle math without
clicking through the UI.

## Project structure

```
ticktock/
├── index.html                       # The entire application (markup, CSS, JS)
└── .github/
    └── workflows/
        └── static.yml                # GitHub Pages deploy-on-push-to-main workflow
```

Everything — markup, styling, state, rendering logic, level configuration,
and both language files — lives in the single `index.html`. This is
intentional: it keeps the project trivially forkable and hostable anywhere
that can serve a static file.

## How the app is built

`index.html` is organized into clearly commented sections inside one
`<script>` block:

1. **i18n strings** (`STRINGS.en` / `STRINGS.de`) — every piece of UI text,
   including functions for strings that need interpolation (e.g.
   `prompt: function(h){ return "Bring the hand into the room of " + h + "!"; }`).
2. **`phraseFor(hour, minute, lang)`** — converts an hour/minute pair into a
   natural-language phrase ("quarter to five" / "Viertel vor fünf") in either
   language.
3. **`LEVELS`** — a small config array describing each level's mode
   (`hourOnly` vs `minute`), which minute marks are allowed, its color theme,
   and visual flags (`pie`, `scaffold`, `splitFace`, `elapsed`).
4. **Persistence** (`loadProgress` / `saveProgress`) — reads/writes a single
   JSON blob to `localStorage` under the key `tickyUhrProgress_v2`.
5. **Round generation** (`generateRound`, `buildRoundFor`, `buildTextRound`,
   `buildTextOptions`) — produces a fresh, non-repeating target time for each
   round, including the deliberately-constructed "carry the hour" arithmetic
   rounds in level 5.
6. **SVG clock building** (`buildClockSVG`, `updateLiveClock`) — draws the
   clock face, tick marks, numbers, hands, and level-specific overlays (pie
   slice, split-face halves, the "training wheels" number bubble), and
   updates hand positions live during a drag without a full re-render.
7. **Drag controllers** (`attachDrag`, `attachExploreDrag`) — pointer-event
   based dragging that computes the angle from the clock's center, detects
   full rotations (to increment/decrement the hour), and snaps to the
   level's allowed minute values on release. The explore screen uses its own
   drag controller with no snapping and no scoring.
8. **Screen renderers** (`renderWelcome`, `renderMap`, `renderExplore`,
   `renderQuiz`, `renderComplete`, plus modals) — each returns an HTML
   string for the current `state.screen`.
9. **`render()` / `attachHandlers()`** — the tiny "framework": `render()`
   picks the right screen function, injects its HTML into `#app`, then
   `attachHandlers()` wires up event listeners by element ID/class (since
   `innerHTML` replacement discards any previous listeners).

There's no virtual DOM and no diffing — screens are cheap enough to
re-stringify and replace wholesale on every state change, except for the
live clock-hand updates during a drag, which mutate SVG attributes directly
for smoothness.

## Internationalization (i18n)

Adding or editing text means editing the `STRINGS.en` / `STRINGS.de` objects
near the top of the script. Both objects must stay in sync (same set of
keys) since the UI looks up strings dynamically via `T()` (returns the
object for `state.lang`). Strings that need to embed a value (a name, a
number, a computed phrase) are functions instead of plain strings — call
`t.someKey(arg)` at the render call site.

To add a third language, add a new key (e.g. `STRINGS.fr`) with the same
shape as `en`/`de`, then add a button for it in `renderLangSwitch()`.

## Progress & local storage

All progress is stored client-side only, under `localStorage` key
`tickyUhrProgress_v2`, shaped roughly as:

```json
{
  "stars": { "1": 3, "2": 2 },
  "unlocked": 3,
  "lang": "de",
  "bestTimeLevel5": 47,
  "name": "Mira"
}
```

There is no server, no account system, and nothing is ever sent off the
device. Clearing the browser's site data resets progress entirely.

## Deployment (GitHub Pages)

`main` is configured to auto-deploy to GitHub Pages on every push, via
`.github/workflows/static.yml` ("Deploy static content to Pages"), which
uploads the entire repository as a Pages artifact. There is no build step —
the workflow deploys `index.html` as-is.

To deploy your own fork:

1. Push your changes to `main`.
2. In the repo's **Settings → Pages**, set the source to **GitHub Actions**
   (if not already set).
3. The workflow runs automatically and publishes to
   `https://<your-username>.github.io/<repo-name>/`.

## Customizing / extending

A few common tweaks and where to make them:

- **Add a level**: add an entry to `LEVELS`, an EN/DE title block under
  `STRINGS.*.levels`, an icon in `LEVEL_ICONS`, and a case in
  `buildRoundFor()` / `promptTextFor()` for its round-generation and prompt
  logic.
- **Change the color palette**: edit the CSS custom properties under
  `:root` (and the `@media (prefers-color-scheme: dark)` / `[data-theme]`
  blocks for dark mode) near the top of the `<style>` block.
- **Change round count per level**: edit `ROUNDS_PER_LEVEL`.
- **Change the star thresholds**: edit the comparisons in `finishLevel()`.
- **Swap the support link**: `COFFEE_URL` and `COFFEE_QR_SVG` near the top
  of the script — regenerate the QR if you change the URL (see the inline
  comment above `COFFEE_QR_SVG` for how it was built: a hand-rolled,
  rounded-corner QR styled to match the app, verified to decode correctly
  with a QR reader before embedding).

## Browser support

Targets modern evergreen browsers (recent Chrome, Safari, Firefox, Edge) on
both desktop and mobile. Uses Pointer Events for drag interactions, CSS
custom properties, `prefers-color-scheme`, and the Web Audio API — all
widely supported, no polyfills included.

## Accessibility notes

- Respects `prefers-reduced-motion` (animations are skipped entirely for
  users who request reduced motion).
- Interactive elements have visible `:focus-visible` outlines.
- Buttons and the clock's drag handles carry `aria-label`s.
- User-provided text (the child's name) is HTML-escaped before being
  rendered, since it's inserted via string concatenation into `innerHTML`.

There's room to go further here (e.g. full keyboard control of clock-hand
dragging) — contributions welcome.

## Support the project

If Ticky helped your kid learn to tell time, there's an optional "Buy me a
coffee" button below the level map — [buymeacoffee.com/tellkoroke](https://buymeacoffee.com/tellkoroke).
Entirely optional, always appreciated, never required to use the app.

## License

[MIT](LICENSE) — free to use, modify, and share.
