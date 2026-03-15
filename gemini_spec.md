Got it. Here is the entire specification enclosed within a single, easily copyable markdown block.

```markdown
# Specification: Micro-breaker Practice Timer

## 1. App Overview
**Micro-breaker** is a Progressive Web App (PWA) designed for musicians. Inspired by Molly Gebrian's research, it facilitates interleaved practice by breaking sessions into focused "chunks" separated by "micro-breaks" to aid neurological consolidation. The app features a robust state machine, a custom Web Audio API synthesizer for notifications, and MediaRecorder integration for self-evaluation.

---

## 2. App States & Phases (Layout Logic)

The app is divided into three vertical flex zones: **Top** (Progress/Labels), **Middle** (Ring/Timer/Prompts), and **Bottom** (Controls). The UI dynamically adapts based on the current phase.

### Core Phases
| Phase | Background (`#app`) | Edge Fill (`#bg-fill`) | Description & Visible Elements |
| :--- | :--- | :--- | :--- |
| **Ready** (`ready`) | `#1e1e1e` (Dark Grey) | `#0d0d0d` | **Top:** Empty. **Mid:** Rest questions visible, "Ready?" displayed inside the ring. Ring strokes hidden. **Bottom:** "Start Practice" pill button. Info (i) icon visible. |
| **Practice** (`work`) | `#b83c08` (Red/Orange) | `#4d1903` | **Top:** "PRACTICE" label, Macro progress bar (chunk progress). **Mid:** Round counter (e.g., "Round 1 of 5"), animated SVG ring tracking phase time, Time remaining text. **Bottom:** Playback controls (Prev, Play/Pause, Next). Mic recording active. |
| **Micro-break** (`break`) | `#141560` (Deep Blue) | `#080928` | **Top:** "MICRO-BREAK", Macro progress bar. **Mid:** Round counter, animated SVG ring, Time remaining, random micro-break reminder message below the ring. "Review" pill button visible if recording exists. **Bottom:** Playback controls. |
| **Rest** (`rest-count`) | `#1e1e1e` (Dark Grey) | `#0d0d0d` | **Top:** "REST". **Mid:** Rest questions, animated SVG ring, Time remaining. **Bottom:** Playback controls. Info (i) icon visible. |

### Overlays
Overlays cover the screen with a high `z-index` and trap focus/interaction.
* **Settings (`#settings-overlay`):** Dark theme (`#1a1a1a`). Contains steppers (+/-) for durations/rounds, toggles for auto-advance/recording/notifications, a volume slider, and inputs to edit custom rest questions and micro-break messages. 
* **Reset Dialog (`#reset-overlay`):** Dimmed semi-transparent background (`rgba(0,0,0,0.75)`). Modal dialog confirming if defaults should be restored.
* **Review (`#review-overlay`):** Deep green (`#1a3d28`). Audio review interface. Displays a dynamically generated canvas waveform of the recorded blob, progress bar, current time/duration, and dedicated playback controls.
* **Info (`#info-overlay`):** Light theme (`#f5efe6`). Scrollable text containing instructions and keyboard shortcuts.

---

## 3. UI Controls

### Main Controls
Icons are rendered using inline SVG. Standard button opacity is `0.88`, dropping to `0.4` and scaling down (`0.90`) on `:active`.

| Control | Location | SVG / Appearance Snippet | Function |
| :--- | :--- | :--- | :--- |
| **Play/Pause** | Bottom (Center) | `<circle cx="34" cy="34" r="32" stroke="white" stroke-width="2" fill="rgba(255,255,255,0.09)"/>` <br> Play: `<path d="M26 20 L50 34 L26 48 Z" fill="white"/>` | Toggles `isPaused`. Swaps path to two `<rect>` elements when paused. |
| **Prev / Next** | Bottom (Flanks) | `<path d="M38 11 L16 23 L38 35 Z" fill="white"/> <rect x="7" y="11" width="5" height="24" rx="2" fill="white"/>` | Restarts current phase / skips to next phase. |
| **Start Pill** | Bottom (Ready) | `border: 1.5px solid rgba(255,255,255,0.40); border-radius: 999px;` | Initiates the first chunk. |
| **Review** | Mid (Break) | `<button id="review-btn">` Contains complex SVG equalizer bars varying in opacity. | Opens the Review overlay to listen to the last recorded passage. |
| **Close (X)** | Bottom Left | `<circle ... /> <path d="M13 13 L25 25M25 13 L13 25" .../>` | Exits chunk, stops recording, returns to Ready phase. |
| **Info (i)** | Bottom Left | `<circle .../> <text ... font-family="Georgia, serif" font-style="italic">i</text>` | Opens the Info overlay. |
| **Settings** | Bottom Right | Complex `<path>` gear with center hole via `fill-rule="evenodd"`. | Opens Settings overlay. |

### Overlay Controls
| Control | Location | Appearance | Function |
| :--- | :--- | :--- | :--- |
| **Stepper (+ / -)** | Settings | Circle, `bg: rgba(255,255,255,0.12)`, `border: 1.5px solid rgba(255,255,255,0.25)` | Adjusts numeric settings. |
| **Toggle Switch** | Settings | Pill track `bg: rgba(255,255,255,0.18)` (active: `#1a3d28`), internal white thumb. | Boolean settings (Recording, Auto-advance). |
| **Review Exit** | Review | Circular outline, `border: rgba(200,40,40,0.40)`, `bg: rgba(200,40,40,0.10)`. Red X. | Closes the Review overlay. |
| **Review Seek** | Review | Arrows with tail lines. | Forward/Back 5 seconds. |

---

## 4. Audio Engine & Sound Notifications

Sounds are generated programmatically using the Web Audio API (OscillatorNode + GainNode). 

**Base Beep Function Snippet:**
```javascript
function beep(freq, dur, gain, type, delay) {
  const t = audioCtx.currentTime + delay;
  const g = audioCtx.createGain();
  const o = audioCtx.createOscillator();
  o.type = type || 'sine';
  o.frequency.value = freq;
  g.gain.setValueAtTime(0, t);
  g.gain.linearRampToValueAtTime(gain, t + 0.012);
  g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
  o.connect(g); g.connect(audioCtx.destination);
  o.start(t); o.stop(t + dur + 0.02);
}

```

### Sound Events Table

| Event | Tones / Frequencies | Duration | Description |
| --- | --- | --- | --- |
| **Countdown (3, 2)** | `880Hz` | `0.11s` | Short beep at 3s and 2s remaining. |
| **Countdown (1)** | `1047Hz` | `0.11s` | Higher pitched beep at 1s remaining. |
| **Work Start** | `660Hz` (delay 0s)<br>

<br>`880Hz` (delay 0.13s) | `0.10s`<br>

<br>`0.18s` | Two-tone rising chime (up a perfect fourth). |
| **Break Start** | `130.8Hz` (C3) | `3.0s` | Low, soothing, long-fading tone indicating rest. |
| **Fanfare (Chunk End)** | `261.6`, `329.6`, `392.0`, `523.3` (staggered 0.16s apart) | `0.18s` (arpeggio)<br>

<br>`0.60s` (final) | Triumphant C-major arpeggio (C-E-G-C). |
| **Back to Work (Rest End)** | `440` (x3), `523` (x2), `660` (final) | `0.10s` (short)<br>

<br>`0.50s` (final) | Rhythmic "da-da-da... da-da... DA!". |
| **Close App** | `196Hz`, `131Hz` | `0.45s`, `0.90s` | Two-tone descending chime indicating cancellation. |

---

## 5. Typography

The primary font is `Inconsolata`, imported from Google Fonts. Variables and fluid typography (`clamp()`) are used extensively to support mobile and tablet sizing.

| Element | Font Family | Size / Clamp | Weight | Color / Opacity | Formatting |
| --- | --- | --- | --- | --- | --- |
| **Global Text** | `'Inconsolata', monospace` | Varies | Varies | `#ffffff` | - |
| **Phase Label (Top)** | `'Inconsolata'` | `clamp(26px, 8vw, 36px)` | `400` | `#ffffff` (0.52 op) | Uppercase, `0.22em` tracking |
| **Progress / Elapsed** | `'Inconsolata'` | `clamp(14px, 3.8vw, 18px)` | `300` / `600` | `#ffffff` (0.55 / 1.0 op) | Elapsed time is `font-weight: 600` |
| **Rest Questions** | `'Inconsolata'` | `clamp(16px, 4.5vw, 22px)` | `300` | `#e8d5b0` | - |
| **Round Counter** | `'Inconsolata'` | `clamp(18px, 5.5vw, 26px)` | `400` | `#ffffff` (0.75 op) | `0.04em` tracking |
| **Time Display** | `'Inconsolata'` | `clamp(42px, 13vw, 60px)` | `300` | `#ffffff` | `-0.02em` tracking |
| **Ready Text** | `'Inconsolata'` | `clamp(38px, 11vw, 54px)` | `300` | `#ffffff` | `0.03em` tracking |
| **Messages (Break)** | `'Inconsolata'` | `clamp(16px, 4.5vw, 22px)` | `300` | `#e8d5b0` | `0.06em` tracking |
| **Info Icon "i"** | `Georgia, serif` | `18px` (inside SVG) | N/A | Current fill | Italicized |

---

## 6. Information Overlay Text

Here is the extracted content of the `#info-overlay`:

This music practice timer is inspired by the work of [Molly Gebrian](https://www.mollygebrian.com/) and incorporates some of her suggestions for efficient practice.

**“Micro-breaks”** are 10–20 second pauses that allow your brain to consolidate what you were just practicing.

The app encourages working in **chunks** of 5–10 minutes, with 1–3 minute rests between chunks.

**Interleaved practice** is encouraged — choose a different piece, passage, etude, or goal for each chunk. See Molly’s work for the compelling research behind this.

**Recording yourself** is encouraged! Turn on “Record / review practice” in Settings. Your playing will be recorded each round, and you can review it during the micro-break.

The **keyboard shortcuts** work with a “page turner” footpedal (that can emulate a “SPACE” or “ENTER” key.) They eliminate most of your need to touch the screen during a normal practice flow.

| Phase | SPACE | ENTER |
| --- | --- | --- |
| Practice | Pause / Play | End Round |
| Micro-Break | Pause / Play | Review Recording |
| Review | Pause / Play | Close |
| Rest “Countdown” | Pause / Play | End Countdown |
| Rest “Ready?” | Start Practice | Start Practice |

If you turn off **“Automatically advance”** in Settings, you will get an audible notification when a practice round or micro-break ends, and the timer will wait for you to skip to the next phase manually.

To add this app to your iPhone or iPad home screen, tap the **Share** button (the box with an arrow pointing up) in Safari, then scroll down and tap **“Add to Home Screen.”** The app will open full-screen without the Safari browser controls.

If the record/review feature is not working on iPhone or iPad, go to **Settings → Safari → Microphone** and make sure it is not set to “Deny.” To avoid being prompted for microphone permission every time you use the app, you can set this to “Allow.”

On Chrome and Android, the microphone indicator stays on between practice rounds — this is intentional, to avoid requesting permission at the start of each round. Recording only occurs during the Practice phase.

If you have feedback, email microbreaktimer@gmail.com.

---

## 7. Implementation Hints (PWA & iOS considerations)

To ensure smooth operation as a PWA, especially on iOS/Safari, the app uses several specific implementation strategies:

1. **Audio Context Unlocking:** Web Audio API requires a user gesture to start. The app listens for `touchstart` and `click` events on main buttons to instantiate or `resume()` the `AudioContext`. It also listens to `visibilitychange` to resume audio if the app was backgrounded.
2. **iOS/Safari Microphone Handling:** Safari exhibits an aggressive persistent "red pill" mic indicator. If left active between practice phases, it annoys users. The app explicitly checks `navigator.userAgent` for Safari (`/^((?!chrome|android).)*safari/i.test(navigator.userAgent)`). If true, it calls `micStream.getTracks().forEach(t => t.stop())` during breaks to kill the red pill. Chrome keeps the stream alive to prevent constant re-prompting.
3. **Bfcache (Back/Forward Cache) Restore:** iOS Safari frequently caches page states. Returning to the app might load a stale DOM (e.g., an overlay left open). The app binds to the `pageshow` event to explicitly remove the `.open` class from all overlays (`#review-overlay`, `#settings-overlay`, etc.) and resume the audio context.
4. **Auto-Play Review (iOS):** iOS requires `HTMLAudioElement.play()` to be called synchronously within a user gesture. In the `openReview` function, the object URL is mapped to `src` and `.play()` is chained *immediately* without awaiting any metadata to prevent the play request from being blocked.
5. **Dynamic Apple Touch Icon:** Instead of requiring an external `.png`, the app generates its home screen icon programmatically using a `<canvas>`. It draws the orange background, radial vignette, semi-transparent rings, and a Palatino 'μ' symbol, converts it via `.toDataURL('image/png')`, and injects it into the `<link rel="apple-touch-icon">` tag dynamically on load.
6. **Theme Color Syncing:** The `<meta name="theme-color">` and CSS variables (`env(safe-area-inset-...)`) are aggressively synced to the active phase's background edge color (`#bg-fill`) via JavaScript to ensure the iOS status bar seamlessly blends into the app UI, preventing ugly white/black bars at the top notch.

```

```