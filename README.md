# ...Thunder Perform

A live performance app for ...Thunder Plus projects, laid out like a DJ setup: clips (project files) on two
decks either side of a mixer. For computers only.

**Status:** clips and transitions. Each side has a column of clips (...Thunder Plus project files). Click one
to launch it on the next beat, bar, 2 or 4 bars; it takes over from the clip playing on its side with its own
transition: a **hard cut**, or an equal-power **crossfade** over 1, 2, 4, 8 or 16 bars. Mixer: channel faders with
meters, crossfader, master. **Edit** shows the playing clip's deck full size; **Keep changes in clip** saves the
edits into the clip.

## How it works

- Each side has **two engines**, each a real ...Thunder Plus page in a hidden frame (`thunder-plus/index.html?deck=a1`,
  `a2`, `b1`, `b2`), so a side can play its current clip and the next one at once during a crossfade.
  - Engine input → engine gain (the transition fades) → side channel fader → crossfader → master.
  - A launch loads the clip into the side's idle engine (in that deck's Worker, so the playing clip isn't
    disturbed), waits if that engine is still fading out, starts it on the next boundary and hands over.
    A clip already loaded in the idle engine starts with no load at all.
  - The first clip launched when nothing plays sets the tempo to its own. After that every clip plays at the
    master tempo (tempo ramps are the next step).
- **Clips** are stored in this app's IndexedDB (`thunderperform`), settings under `thunderperform-settings-v1`.
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
  - `window.__deck`: `ready()`, `open(projectText)`, `play(at, row)`, `stop(at)`, `setBpm(bpm, at)`, `info()`, `save()`.
    Times are on the shared AudioContext clock.
  - `ready()` waits for the deck's Worker to render every pad (falling back to the page thread only without one).
- **The clock** (this page): beats count from an anchor; a tempo change re-anchors so the beat count never
  jumps. A launch waits for the next beat, bar, 2 or 4 bars.

## Verified (headless Chromium, 13 Oct 2026)

- **Sync:** two decks started at one time begin on the same sample; their audio was bit-identical in 9 of 10
  runs. The other run differed in sound, not timing; probably a background pad render landing during the
  first play.
- **Launch:** a deck launched while the other plays starts exactly on the next bar.
- **Crossfade:** a 2-bar crossfade started on the bar 2 bars after the first clip. The gains followed the
  equal-power curve, the old clip stopped exactly 2 bars later, and its engine was free again.
- **Hard cut:** landed on the bar, reusing the already-loaded engine (about 4 ms, no reload).
- **No stalls:** while a clip loaded (about 2 s, in the background), the longest page stall was 62–74 ms, inside
  the sequencer's 150 ms lead.
- **Storage:** nothing is written to ...Thunder Plus's storage; only `thunderperform-settings-v1` and the
  `thunderdeck-*` sample databases.

## Running it locally

```
git clone --recurse-submodules https://github.com/matthettich/thunder-perform-
cd thunder-perform-
python3 -m http.server 8741 --bind 127.0.0.1
```

Then open http://127.0.0.1:8741/ and press **Start audio**.

## Next

1. Tempo ramp transition (pull the master tempo to the incoming clip's over its length).
2. Sequence morph between the outgoing and incoming patterns.
3. Momentary FX per side.
4. MIDI/keyboard launching.

## To update later: ...Thunder Plus changes since the pin (noted 20 Oct 2026)

- **The decks are behind.** The `thunder-plus` pin (`937e278` on `deck`) predates a lot of work on ...Thunder Plus's
  `main` (Arrange as the Song screen, MIDI sequencing, new engines and effects, Renoise-style keys…). Before relying on
  it, merge `main` into `deck`, bump the pin (see How it works) and run the checks under Verified again.
- **More tracks (planned in ...Thunder Plus, not started).** ...Thunder Plus may raise its track limit from 32 to as many
  as 128. The plan keeps song files readable here: tracks 1–32 and the master column stay where they are, and tracks 33+
  (and their block mutes) go in a separate section of the song. When that lands, update this app: merge `main` into
  `deck`, bump the pin, and test a clip that uses tracks above 32 (it plays, mutes hold, a crossfade and a hard cut work,
  CPU with many tracks). This page has no per-track code of its own (the mixer is per side), so the deck is what
  changes. Details: ...Thunder Plus's `HANDOVER.md`, Open items, "More tracks than 32".
