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