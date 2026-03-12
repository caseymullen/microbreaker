# Music Practice Micro-Break Timer — Specification

## Concept

A Progressive Web App (PWA) designed for musicians to enforce structured micro-breaks during practice. Modeled on HIIT-style interval training: short focused work bursts alternating with brief mental resets, grouped into larger timed blocks.

-----

## Timer Structure

### Micro Cycle (inner loop)

- **Practice phase** — active playing/work
- **Micro-break phase** — brief mental rest; brain consolidates recent practice
- Cycles repeat automatically until the macro block expires

### Macro Cycle (outer loop)

- A timed block containing many micro cycles
- When the block expires, a **Long Rest** begins
- After the long rest, a new block starts

-----

## Default Durations (all adjustable in Settings)

|Timer         |Default|Range    |
|--------------|-------|---------|
|Practice phase|25 sec |5–120 sec|
|Micro-break   |15 sec |5–60 sec |
|Block length  |5 min  |1–15 min |
|Long rest     |1.5 min|0.5–5 min|

-----

## Controls

- **Advance Phase button** — immediately skips to the next phase (work → break → work, or triggers long rest if block expired)
- **Pause button** — freezes all timers
- **Settings button** — opens settings panel; pauses timer while open
- **Keyboard** — Enter or Space = advance phase; P = pause

-----

## Visual Design

- Full-screen color themes per phase:
  - **Practice** — warm purple/amber
  - **Micro-break** — cool green/teal
  - **Long rest** — deep blue/indigo
- Circular countdown ring for the micro timer
- Progress bar for macro block elapsed time
- Block counter (e.g. “Session block · 2”)
- Animated pulse dot during practice phase

-----

## Audio

- Distinct tones for each phase transition
- 3-2-1 countdown beeps before each phase ends
- Mute toggle in settings
- **iOS limitation:** audio requires an initial user tap to unlock (Web Audio API autoplay policy). First tap on Advance Phase unlocks audio for the session.

-----

## PWA / iPhone Notes

- Single self-contained HTML file (no dependencies to download)
- `apple-mobile-web-app-capable` meta tag — runs full-screen when added to home screen
- Install: open in Safari → Share → Add to Home Screen
- Audio context resumes on tap after any suspension

-----

## Known Issues / Open Questions

- Auto-transitions are silent if the AudioContext has been suspended (iOS). A “Tap to Begin” splash screen on first load would reliably pre-unlock audio.
- [ ] Should there be a visual/haptic cue on iPhone when sound is blocked?
- [ ] Should settings persist between sessions (localStorage)?
- [ ] Should block count reset on app reopen, or persist?

-----

## Possible Future Features

- Tap-to-Begin splash screen to unlock audio on iOS
- Haptic feedback (Vibration API) for phase changes
- Session log (how many blocks completed)
- Custom presets (e.g. “Slow practice mode”, “Performance prep”)
- Landscape layout support