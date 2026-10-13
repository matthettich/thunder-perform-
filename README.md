# ...Thunder Perform

A live performance app for ...Thunder Plus projects, laid out like a DJ setup: clips (project files) on two
decks either side of a mixer. For computers only.

**Status:** sync test. Two decks, one shared clock, quantized play/stop, tempo, channel gains and a crossfader.

## How it works

- Each deck is a real ...Thunder Plus page in a frame, opened as `thunder-plus/index.html?deck=a` (or `b`).
- `thunder-plus/` is a **git submodule pinned to one commit of the `deck` branch** of `matthettich/thunder-plus`.
  - Thunder Plus's `main` branch and its live page never get deck mode.
  - Work on Thunder Plus can't break this app. To bring it in, merge `main` into `deck`, then bump the pin
    (`git -C thunder-plus fetch && git -C thunder-plus checkout <new deck commit>`, then commit here).
- **Deck mode** (`?deck=`, the head script and `window.__deck` in the deck branch's `index.html`):
  - The deck plays through this page's AudioContext, into its own input on the mixer.
  - Its settings are kept in memory and its samples in its own database (`thunderdeck-<id>-…`), emptied at
    start, so a deck never reads or changes the saved song, kit or samples of ...Thunder Plus.
  - It has no service worker or update checks.
  - Each audio worklet is added to the shared context once (`__thunderPerform.once`), for both decks.
  - `window.__deck`: `ready()`, `open(projectText)`, `play(at, row)`, `stop(at)`, `setBpm(bpm, at)`, `info()`.
    Times are on the shared AudioContext clock.
- **The clock** (this page): beats count from an anchor; a tempo change re-anchors so the beat count never
  jumps. A launch waits for the next beat, bar, 2 or 4 bars.

## Verified (headless Chromium, 13 Oct 2026)

- Both decks started at one time: the first sound is on the same sample on both, and the audio was
  bit-identical in 9 of 10 runs. The other run differed in sound only, not timing; probably a background pad render
  landing during the first play.
- A deck launched while the other plays starts exactly on the next bar (sample-accurate).
- A tempo change reaches both decks; stop lands on the bar.
- Nothing is written to the page's real localStorage; only the two `thunderdeck-*` sample databases exist.

## Running it locally

```
git clone --recurse-submodules https://github.com/matthettich/thunder-perform-
cd thunder-perform-
python3 -m http.server 8741 --bind 127.0.0.1
```

Then open http://127.0.0.1:8741/ and press **Start audio**.

## Next

1. Clip columns per deck (project files, kept in IndexedDB), with per-clip transition and length.
2. Transitions: hard cut, crossfade, tempo ramp, sequence morph.
3. Momentary FX per side.
4. Live editing: show a deck's frame full size to edit its song in the tracker.
