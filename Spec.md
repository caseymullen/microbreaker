# Music Practice Timer — Specification

## Concept

A Progressive Web App (PWA) designed for musicians to encourage structured micro-breaks during practice. Modeled on HIIT-style interval training: short focused work bursts alternating with brief mental resets (micro-breaks), grouped into larger timed chunks.

-----

## Intended deployment environment
Optimize for an iPhone or iPhone mini. Use UI techniques that will work reliabily in Safari on an iPhone. Use WebKit-compatible patterns. Ensure that the iPhone status bar color matches the color of the app. 

Try to make it look "native" on the iPhone. For example, you might need to do something like is shown in the code chunk, below. If the app is displaying "under" the status bar, ensure that we don't try to show actual content under the status bar.
```code
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
```
-----

## Timer Structure

### Round (inner loop)

- **Practice phase** — active playing/work. AKA "work"
- **Micro-break phase** — brief mental rest; brain consolidates recent practice. AKA "break"
- rounds repeat automatically until the macro chunk is done
- The length of a chunk is specified in rounds, but also show the user the duration in minutes and seconds.
- After the final work round, do NOT skip the micro-break. The micro-break is the only opportunity the user has to review their recording.


### Chunk (outer loop)

- A timed chunk containing many rounds
- When the chunk expires, play a small "success fanfare" and then begin the "rest" phase.


### REST phase
The rest phase is a bit odd because there are essentially two sub-phases that share the same background color and some UI elements. There is the rest-countdown phase, where the user sees a countdown timer (with the two questions above it). When the rest-countdown phase ends, then the "back to work" alarm goes off (that's the one with the "da-da-da da-da DA!" rhythm). This alarm marks the end of the rest-countdown phase and the start of the rest-waiting phase, where we are just waiting for the user to hit the "Start Practice" button.

A "start practice" button is visible during the entire Rest phase (both sub-phases). Pressing it will start the chunk (the first work phase of the chunk). The button appears where the "audio player" controls appear during other phases (see below).

During the rest-countdown phase, the phase label is "REST".
During the rest-waiting phase, omit the phase label. The user will see the large "Ready?" text in the countdown ring (as described elsewhere).

When the user first opens the app, it starts in the rest-waiting phase. The user sees "Ready?" in the "countdown timer ring" area, and they contemplate the two questions until they hit the "Start Practice" button.

In rest-countdown and rest-waiting phases, show an "information" button (a circle with an "i" in it) in the lower-left. Clicking it opens a "nested" "information" phase.

-----

## Information phase

This shows a white screen with black text.

The text should be derived from the contents of this code block:

```md
This music practice timer is inspired by the work of [Molly Gebrian](https://www.mollygebrian.com/) and incorporates some of her suggestions for efficent music practice.

**"Micro-breaks"** are 10-20 second breaks that allow your brain to begin incorporating what you were learning just before the micro-break. 

The app encourages working in **chuncks** of 5-10 minutes, with 1-3 minute rests between chunks.

**Interleaved practice** is encouraged! Choose different pieces/passages/etudes/goals for each practice chunk. See Molly's work for the compelling motivation for this.

**Recording yourself** is encouraged! Turn on "record/review practice" in settings. Your practice will always be recorded, and you can review it during the micro-break. Click the recording waveform to jump around during playback.

There are **keyboard shortcuts** designed to be used with a "page turner" footpedal that can emulate a "space bar" or "enter" key. The "space" generally acts as play/pause. "Enter" triggers "skip to the end". Both keys trigger the "Start Practice" button in the "rest" phase. "Enter" skips back 5 seconds in the "review" phase.

It is impossible to guess the "right" amount of time for a round of practice. If you turn off "Automatically advance" in settings, you will get an audible notification that the phase has ended, but it will wait for you to manually advance to the next phase (by hitting the "next track" button).

If you have feedback or suggestions, email [microbreaker@gmail.com](mailto:microbreaker@gmail.com).
```

Don't let Cloudflare hide the email. Make it a simple mailto link.

-----

## Default Durations (all adjustable in Settings)

|Timer         |Default|Range    |
|--------------|-------|---------|
|Practice phase|45 sec |15–120 sec|
|Micro-break   |15 sec |10–60 sec |
|Chunk length  |5 rounds |3–15 rounds |
|Rest     |2 min|1–5 min|

-----

## Controls

The main controls look like audio-player controls
- A "next track" button skips to the next phase (work → break → work, or rest when you reach the end of the last work phase in the chunk)
- A "Play/Pause" button — freezes all timers
- A "Previous track" button restarts the current phase

Note that when the user hits the "Next track" and "previous track" buttons, the real elapsed time starts diverging from the time that should be used for displaying the chunk progress bar. We have two time concepts: progressTime and elapsedTime
- progressTime gets decremented or incrementted appropriately when the user hits "previous track" or "next track". Hitting "pause" also stops the progressTime
- elapsedTime shows the real-world elapsed clock time and is unaffected by "previous track", "next track" or "play pause" or going into review mode or settings.

Leave space below the "audio-player" controls for the "Review Recording" button to appear. Also leave a little bit of "blank" space at the bottom of the screen, even when the "Review Recording" is showing.

-----

## Settings Controls

A **Settings button** is in the lower right. It should look like a gear. It opens the settings panel; pauses timer while open

In "settings", display the duration of the chunk directly below where you display the text showing the number of rounds.
In "settings", just use the label "Chunk", rather than "Chunk length".

To save/close settings, use a circle button with a check mark in the upper right.

The "Recording" setting should come after the "audio" setting.

The Audio setting section should be labeled "Notifications" and the setting should be "Audible notifications" and should default to "on".

## Keyboard input

Listen for keyboard input. We really only listen for "Space" and "Enter", which are coming from a foot pedal, not a real keyboard.
All editable text fields should support virtual keyboard input with "Assistive Touch" turn on the iPhone/iPad. Basically there should be some way (ideally a standard iOS based way) to bring up the on-screen keyboard even though the system thinks that there is a keyboard attached, because that "keyboard" is likely to just be a foot pedal.

The "keys" act differently in different phases:

 will play/pause. "Enter" acts like "next track" (play/pause)

| Phase         | Space | Enter    |
|--------------|-------|---------|
|Work | Pause/Play |"next track"|
|Break | Pause/Play |"next track" |
|Review | Pause/Play |restart from beginning |
|Rest-coundown     |Pause/Play|"next track"|
|Rest-waiting|"Start Practice"|"Start Practice"|

-----

## Visual Design

- Full-screen color themes per phase:
  - **Practice** — warm rusty deep orange
  - **Micro-break** — deep blue/indigo
  - **Rest** — black
- Circular countdown ring for the timer during work, break, and rest. Either use a CSS-based approach or, if you use a SVG approach, ensure that SVG values are mathematically precise and declared inline rather than via stylesheet. The same applies to the "chunk progress bar", discussed next.
- Display a progress bar near the top of the screen to show progress towards completion of the chunk. In addition to the moving bar:
  - Below the bar show the time progress, e.g. "0:23 of 5:00
  - The graphical progress bar is governed by the progressTime, whilse the textual time progress shows elapsedTime. This way the user can see how much time they actually spent in the chunk, compared to how much time was allocated to the chunk.
  - Halfway between the progress bar and the countdown ring, display the current progress in terms of rounds. For example "Round 2 of 5". The font size of this text should be larger than the size of the time progress text.



Make fonts large enough for easy readability. No font should be smaller than 16px. 
Use "Inconsolata" font for all text.
Use "clamp()" pattern for font sizes. The two target form-factors are an iPhone in vertical layout and an iPad in landscape layout.

Add subtle and gradual darkness variation to the background so that it is darker at the edges of the screeen than at the center. In the "Rest" phase, this means the base background color should be dark grey (fading to black at the edges).

Leave some space between the phase label and the progress bar to give it some visual separation from the phase label.

Add a "close" button in the lower left. (Maybe a circle with an X in it, but don't make it red.) When it is pressed play the "goodbye" notes C1 G2 with the second note being twice as long as the first.
The "Close" button should not appear in the  rest-waiting phase, because pressing it should reset all of the timers and put the app in the rest-waiting phase.

During the rest-waiting phase, display the text "Ready?" in the same font size and color that you use for the countdown timer ring, and in the exact same location as the text in the countdown timer ring. Essentially, the timer ring goes away, but the countdown text remains but changes to "Ready?" instead of displaying a time.


-----

## Audio

- Distinct tones for each phase transition
- 3-2-1 countdown beeps before each phase ends
- A small "success fanfare" plays at the end of the chunk
- The "alarm" at the end of rest timer has a rhythm like "da-da-da da-da DA!"
  - When you play the "da-da-da da-da DA!" sound, each rhythmic group starts on a steady beat and has the same duration. The pause after "da-da-da" should last as long as the sound. The sound+pause for the other two rhythmic groups should have the same duration,  e.g. "DA [pause]" lasts as long as "da-da-da [pause]" and "da-da [pause]".
- When a "work" phase starts, play a crisp double-beep
- The "break" begins with a meditation chime, which should be a calm, low frequency tone fading over 3 seconds.
- Mute toggle in settings
- **iOS limitation:** audio requires an initial user tap to unlock (Web Audio API autoplay policy). First tap on "Start Practice" unlocks audio for the session.

-----

## Messages

During the break phase, display one of the reminders below. Cycle through them, in order, for different breaks.
- Focus on your goal
- Flex your wrist
- Audiate to intonate
- Create emphasis

During "rest" display these "rest questions", on separate lines, in a beige font, halfway between the "countdown timer ring" area and the top phase label:
- What is your goal?
- How will you achieve it?"

Those are the default messages. They can be edited in settings.


-----
## Settings

All settings are stored in local storage and thus stay in place when the app reloads on the same device.

The "rest questions" are editable in the settings. There can be three. It is ok if some blank.

For durations in settings, do not use a slider. Show the value with a button to the left to decrement it and a button to the right to increment. Time durations increment/decrement by 5 seconds.

In settings, label the "break" reminders as "Micro-break reminders"

Have a button to reset all settings to their defaults. Ask the user if they also want to clear the "micro-break messages". Allow the to answer yes, no, or cancel.

At the end of the "durations", add a "Automatically advance" toggle that defaults to "on". When it is "off", in work and break phases, when the countdown timer finishes, play the appropriate sound, but don't actually advance to the next phase. Wait for the user to manually hit the "skip" button.

Below the "
-----

## Recording Audio

Always be recording audio during the work phase (if the settings toggle has the "record/review" toggle on).

- Recording starts when the "work" phase starts and ends when that phase ends. Hitting "pause" does NOT pause the recording. They may pause because they want more time before taking a micro-break. If the elapsed clock time reaches 5 minutes, stop the recording. If a user hits "pause" and goes to lunch, we don't want to keep recording for hours.
- A "Review Recording" button ONLY appears during the "break" phase, and (if the "record/review" toggle is on in settings) it ALWAYS appears in that phase. The user does not have to pause first. They just hit "Review Recording" (and that will automatically pause the phase while they listen to the recording. 
- When they exit the recording, the "break" phase should remain paused.

 The "Review Recording" button appears below the "audio player" controls. If user hits "Review Recording", we enter a nested "review" phase, with its own screen.
 
 The recorded audio clip is only for the most-recent work phase.
 
 The screen for the "review" phase will have its own "audio player" controls that are not just an analogy, but control the playing of the audio clip. The controls Include:
- Back to the start
- Jump back 5 seconds (a simple small triangular arrow pointing to the left)
- Play/Pause
- Jump forward 5 seconds (a simple small triangular arrow pointing to the left)
- A "X" button that will close the screen/phase. It should be a deep red. When the user clicks it, play the notes C2 G3 as a "goodbye", with the G2 twice as long as the C1.

In "settings" there should be a toggle "Record/review practice" that defaults to "off". When it is off, never show the "Review Recording" button.
- If the user turns the "record/review" setting on, immediately trigger a request to access the microphone. If they entered settings from the work phase, then immediately start recording.
- If the user turns off the "record/review" setting, if there is a recording in progress, then stop it immediately and disgard it.

The controls during review of the recording should be large for easy clicking. The user is probably holding their violin and bow while operating the controls.
While the "Review Recording" audio player is active, the space bar will trigger play/pause and the "Enter" key will act like the "jump back 5 seconds" button.

While reviewing the recording, make sure that screen does now allow the user to zoom the web page in or out.

When a new work/practice phase starts, discard any old recordings.

Regarding the recording player itself:
- It would be nice to show progress through the recording as it plays. It would be awesome to display the waveform.
- Clicking on the waveform should jump the playback to that location and started playing.
- The background color should be a rich forest green with the same effect of blending to darker shades near the edge of the screen.
- The phase label should be sized and positioned like the other "phase" labels.
- Show both the current time in the recording and the total time of the recording.
- When we enter the phase, start it in "play" mode

-----

## PWA / iPhone Notes

- Single self-contained HTML file (no dependencies to download)
- `apple-mobile-web-app-capable` meta tag — runs full-screen when added to home screen
- Install: open in Safari → Share → Add to Home Screen
- Audio context resumes on tap after any suspension

-----

## Other notes

Do not remember the current state of progress through a given chunk in localStorage.




## Implementation Notes: Audio Recording

### Recording lifecycle
- Recording starts when the **work phase begins** and stops when that phase ends
  (i.e. when the break phase starts, or the chunk completes).
- Hitting **pause does NOT pause the recorder** — it continues running independently
  of the timer. This keeps the logic simple and avoids state-sync bugs.
- A **hard 5-minute cap** (`setTimeout → stopRecording()`) prevents runaway recording
  if a user pauses and walks away.
- The **Review Recording button** is shown only during the **break phase**,
  always (no pause required), provided a recording blob is available and the
  record/review toggle is enabled. The user does not need to pause first.
- Tapping **Review Recording** automatically pauses the break phase.
  When the review overlay is closed, the break phase remains paused.

### Critical: MediaRecorder async onstop
`MediaRecorder.stop()` is **asynchronous** — the `onstop` event fires after the
browser finishes encoding, which may be many milliseconds after `.stop()` returns.
This has two important consequences:

1. **Never null `mediaRecorder` before `onstop` fires and you still need the mimeType.**
   Capture the recorder in a local `const mr = new MediaRecorder(...)` and reference
   `mr.mimeType` inside the `onstop` closure — not the global `mediaRecorder`, which
   may already be `null` by the time `onstop` runs.

2. **Never call `render()` synchronously after `.stop()` expecting `reviewBlob` to
   be set yet.** Instead, call `render()` from *inside* the `onstop` handler, after
   assigning `reviewBlob`. That way the UI updates (e.g. showing the Review button)
   happen at the exact moment the blob is ready.
```js
function _beginRec() {
  const mr = new MediaRecorder(micStream);
  mediaRecorder = mr;
  mr.ondataavailable = e => { if (e.data?.size > 0) recChunks.push(e.data); };
  mr.onstop = () => {
    // mr (local) is safe here even if mediaRecorder (global) has been nulled
    if (recChunks.length) {
      reviewBlob = new Blob(recChunks, { type: mr.mimeType || 'audio/webm' });
      render(); // ← update UI here, not after calling .stop()
    }
  };
  mr.start(250); // timesliced so ondataavailable fires regularly
}

function stopRecording() {
  if (mediaRecorder && mediaRecorder.state !== 'inactive') {
    mediaRecorder.stop(); // onstop fires async; closure holds ref via mr
  }
  mediaRecorder = null; // safe to null here — onstop uses mr, not this global
}
```

### Safari/iOS: Blob audio duration is Infinity
Safari reports `audio.duration === Infinity` after `onloadedmetadata` fires on
Blob URL audio. The standard workaround is:
1. In `onloadedmetadata`, check `if (!isFinite(duration))` and set
   `audio.currentTime = 1e9` to force a full scan.
2. Listen for `ondurationchange` — it fires with the real duration once
   the scan completes. Seek back to 0 there.
3. Never use `audio.duration` directly for display or seek math; use a
   separate `knownDuration` variable that is only set when `isFinite(duration)`.


## Implementation Notes: iOS PWA Visual & Hardware Edge Cases

### White bar at bottom (safe area background)
When running as an iOS home screen PWA with `apple-mobile-web-app-status-bar-style`
set to `black-translucent`, the system can show a white or mismatched strip at the
bottom of the screen (behind the home indicator) if only the main app element has a
background color set.

Fixes applied:
1. **`html` and `body`** both get `background-color` set and updated dynamically
   to match the current phase, with `overscroll-behavior: none` and
   `position: fixed; inset: 0` to suppress iOS bounce revealing the background.
2. **`#bg-fill`** — a separate `position: fixed; inset: 0; z-index: -1` div sits
   behind everything and is updated to the *vignette-darkened* edge color on each
   phase transition, not the raw base color. Since the vignette is
   `rgba(0,0,0,0.58)` over the base, the edge color = `base * 0.42`. Example values:
   - Rest `#1e1e1e` → edge `#0d0d0d`
   - Work `#b83c08` → edge `#4d1903`
   - Break `#141560` → edge `#080928`
3. **No-cache meta tags** prevent Safari from serving a stale cached version when
   the PWA is killed and reopened from the home screen:
```html
   <meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">
   <meta http-equiv="Pragma" content="no-cache">
   <meta http-equiv="Expires" content="0">
```
   Note: to force a reload of an already-installed PWA, open the URL directly in
   Safari (not via the home screen icon) and reload there, then re-add to home screen.

### iOS microphone status indicator
The orange microphone dot stays visible as long as any `MediaStream` track is live,
even if no `MediaRecorder` is actively recording. To turn it off between work phases:

- Call `micStream.getTracks().forEach(t => t.stop())` and set `micStream = null`
  inside `stopRecording()`, immediately after stopping the `MediaRecorder`.
- Do **not** cache `micStream` between recordings — always call `getUserMedia` fresh
  at the start of each work phase via `acquireMic()`.
- iOS remembers the permission grant for the origin, so subsequent `getUserMedia`
  calls after the first user approval are silent — no re-prompt dialog.
```js
function stopRecording() {
  // ... stop MediaRecorder ...
  mediaRecorder = null;
  // Release stream so the iOS mic indicator turns off
  if (micStream) {
    micStream.getTracks().forEach(t => t.stop());
    micStream = null;
  }
}
```


-----

## Notification Sounds

Here's a spec snippet based on the actual code:

---

## Notification Sounds

All sounds use sine wave oscillators. Timing is specified as seconds from the moment the sound is triggered. Each note is described as `(frequency Hz, duration s, gain)`. Gain is approximate peak amplitude (0–1). Notes fade out exponentially to near-silence by the end of their duration.

---

### Work phase start
Two rising beeps in quick succession, giving a crisp "ready, go" feel.
- 660 Hz, 0.10s, gain 0.32, at t=0.00
- 880 Hz, 0.18s, gain 0.32, at t=0.13

---

### Micro-break start
A meditation chime — One low C2 note
- C2: 65.4 Hz, 3s, gain 0.30, at t=0.00

---

### Countdown beeps (3, 2, 1 before a phase ends)
Single beeps, fired when the phase timer crosses the 3s, 2s, and 1s marks.
- At 3s and 2s remaining: 880 Hz, 0.11s, gain 0.42
- At 1s remaining: 1047 Hz (C6), 0.11s, gain 0.42 — slightly higher to signal the final beat

---

### Chunk complete fanfare
Four ascending notes followed by a held final note. Plays at the end of the final micro-break in a chunk.
- 261.6 Hz (C4), 0.18s, gain 0.35, at t=0.00
- 329.6 Hz (E4), 0.18s, gain 0.35, at t=0.16
- 392.0 Hz (G4), 0.18s, gain 0.35, at t=0.32
- 523.3 Hz (C5), 0.18s, gain 0.35, at t=0.48
- 523.3 Hz (C5), 0.60s, gain 0.40, at t=0.70 — held resolution

---

### Rest countdown complete / "back to work" alarm
Rhythmic "da-da-da · da-da · DA!" pattern. Three rhythmic groups, each group plus its following silence occupying exactly 0.60s (one 3-beat unit at a beat of 0.20s). The groups start on beats 0, 3, and 6.
- Group 1 — "da da da": 440 Hz, 0.10s, gain 0.28, at t=0.00 / 0.20 / 0.40
- Group 2 — "da da": 523 Hz, 0.10s, gain 0.30, at t=0.60 / 0.80
- Group 3 — "DA!": 660 Hz, 0.50s, gain 0.50, at t=1.20 — long held note, louder

---

### App close ("goodbye")
Two descending notes played when the close button is pressed. Low register, gentle.
- G3: 196 Hz, 0.45s, gain 0.30, at t=0.00
- C3: 131 Hz, 0.90s, gain 0.28, at t=0.50

---

### Review close
Same descending two-note shape as app close, but an octave higher and slightly softer.
- G4: 392 Hz, 0.40s, gain 0.28, at t=0.00
- C4: 261.6 Hz, 0.80s, gain 0.25, at t=0.45

---

### Mute behavior
All sounds are suppressed when "Audible notifications" is off in Settings. Sounds also require the audio context to have been unlocked by at least one user gesture before they will play (standard browser autoplay policy).

-----

# Additional spec information
If the following information contradicts information from above, give this information higher priority

Here's a spec section covering layout, typography, and buttons:

---

## Layout

The main screen is a single vertical flex column (`flex-direction: column`, `align-items: center`) that fills the full viewport. Safe-area insets are applied as padding on all sides. The column has no overall `justify-content` — spacing is governed by two flex-grow zones described below. Overflow is hidden.

The elements in order from top to bottom:

1. **Phase label** — `flex-shrink: 0`
2. **Label gap** — a fixed-height spacer equal to approximately one line of the elapsed-time font (`clamp(28px, 7.5vw, 36px)`), always present, creates breathing room below the phase label
3. **Progress section** — `flex-shrink: 0`, hidden (`visibility: hidden`) in ready and rest phases
4. **Mid-zone** — `flex: 1`, `justify-content: flex-end`, `padding-bottom: 16px`. This is the growing space between the progress bar and the ring. Its content (rest questions or round counter) sits in the lower half of the zone, near the top of the ring.
5. **Ring wrap** — `flex-shrink: 0`
6. **Lower-zone** — `flex: 1`, `justify-content: flex-start`, `padding-top: 16px`. This is the growing space between the ring and the controls. The micro-break reminder message sits in the upper half of the zone, just below the ring.
7. **Controls row** (or Start Practice button in ready phase) — `flex-shrink: 0`
8. **Review slot** — `flex-shrink: 0`, fixed `min-height: 48px`, always reserves space so controls don't shift

The mid-zone and lower-zone each get an equal share of remaining vertical space, so the ring sits centered between the progress bar area and the controls.

---

## Progress Section

The progress bar track is 6px tall with `border-radius: 3px`. The fill uses the same radius. The elapsed-time counter below it is centered, not right-aligned.

---

## Typography

All font sizes use `clamp()` for fluid scaling. The font is Inconsolata throughout.

| Element | Size |
|---|---|
| Phase label | `clamp(26px, 8vw, 36px)`, weight 300, uppercase, letter-spacing 0.12em |
| Elapsed time (`0:32 of 5:00`) | `clamp(22px, 6vw, 28px)`, weight 300 |
| Rest questions | 80% of round counter size, i.e. `clamp(22px, 6.6vw, 31px)`, weight 300, color `#e8d5b0` |
| Round counter | `clamp(27px, 8.2vw, 39px)`, weight 300, color `rgba(255,255,255,0.55)`, two lines: "Round" / "N of M" |
| Countdown ring digits | `clamp(38px, 11vw, 54px)`, weight 300 |
| "Ready?" ring text | Same as countdown ring digits |
| Micro-break message | `clamp(15px, 4.2vw, 20px)`, weight 300, color `rgba(255,255,255,0.60)`, letter-spacing 0.06em |
| Start Practice button | `clamp(18px, 5vw, 24px)`, weight 300, letter-spacing 0.10em |

---

## Buttons

### Playback controls (restart / play-pause / skip)

Three buttons in a horizontal row, centered, with `gap: clamp(18px, 5vw, 32px)`. Each is a bare `<button>` with no background or border (`background: none`, `border: none`). Color is `rgba(255,255,255,0.75)`, brightening to `#fff` on press. Padding 8px on all sides.

- **Restart (previous)**: SVG 38×38. A vertical bar (rect, width 3.5, rounded ends, rx 1.5) on the left, and a left-pointing filled triangle to its right. The triangle's right vertex touches the right edge of the viewbox.
- **Play/Pause**: SVG 68×68 — noticeably larger than the flanking buttons. Two states, toggled by showing/hiding:
  - *Play*: a right-pointing filled triangle with generous margins
  - *Pause*: two vertical rounded rectangles (rx 4), width 16 each, at x=10 and x=42
- **Skip (next)**: SVG 38×38. Mirror of restart — vertical bar on the right, right-pointing filled triangle to its left.

### Start Practice button

Pill-shaped (`border-radius: 100px`). Background `rgba(255,255,255,0.12)`, border `1.5px solid rgba(255,255,255,0.30)`. Padding `14px 36px`. Brightens on press to `rgba(255,255,255,0.24)`. Occupies the same vertical slot as the playback controls row (both are `flex-shrink: 0` siblings; only one is visible at a time via `display: none` / `display: flex`).

### Close button (lower-left, fixed)

Circle with X inside. SVG 38×38. Circle is `r=17`, stroke `rgba(255,255,255,0.36)` weight 1.8, no fill. X is two diagonal lines, stroke weight 2, `stroke-linecap: round`. Color `rgba(255,255,255,0.36)`, brightens on press. **Hidden** (`opacity: 0`, `pointer-events: none`) during ready and rest-countdown phases. Sits at `fixed` position `bottom: safe-bot + 16px`, `left: safe-l + 18px`, z-index 10.

### Info button (lower-left, fixed — same slot as close button)

Circle with italic *i* inside. SVG 38×38. Same circle as close button. The *i* is a `<text>` element, font Georgia serif, italic, 18px. Shown only during ready and rest-countdown phases (close button hidden in those same phases, so they don't overlap). Same position as close button.

### Settings gear (lower-right, fixed)

SVG 28×28 rendering a gear/cog shape. Color `rgba(255,255,255,0.36)`, brightens on press. Position `bottom: safe-bot + 18px`, `right: safe-r + 20px`, z-index 10.

### Settings done button (upper-right of settings header)

Circle with a checkmark inside. SVG 34×34. Circle `r=15`, stroke weight 1.8. Checkmark path: `M10 17 L15 22 L24 12`, stroke weight 2.2, `stroke-linecap: round`, `stroke-linejoin: round`. Color `rgba(255,255,255,0.75)`.

### Info overlay close button (upper-right)

Circle with X inside. SVG 34×34. Circle `r=15`, stroke weight 1.8. X path identical in style to the main close button but scaled to the smaller circle. Color `rgba(0,0,0,0.45)` (dark, since the info overlay is white).

### Review Recording button

Appears during micro-break phase only, in the review slot below the controls. Rounded rectangle (`border-radius: 8px`), background `rgba(255,255,255,0.10)`, border `1px solid rgba(255,255,255,0.22)`, padding `9px 20px`.

### Review player controls

Five buttons in a horizontal row inside the green review overlay. All `background: none`, `border: none`, color `rgba(255,255,255,0.80)`. The play/pause button has extra padding (`padding: 12px`) making it the visual focal point.

- **Back to start**: SVG 32×32. Vertical bar on left (rect 3.5 wide, rounded), left-pointing filled triangle.
- **Back 5s**: SVG 26×26. Left-pointing filled triangle only (smaller, no bar).
- **Play/Pause**: SVG 32×32. Same two-state toggle as main play/pause but scaled down. The play triangle and pause rectangles are proportionally sized for the 32×32 viewbox.
- **Forward 5s**: SVG 26×26. Right-pointing filled triangle (mirror of back 5s).
- **Close (red X)**: Two diagonal lines, stroke `#e03030`, weight 2.6, `stroke-linecap: round`. No circle. SVG 22×22.

---

## Visibility Rules

| Element | ready | rest-countdown | work | break |
|---|---|---|---|---|
| Phase label | blank | "Rest" | "Practice" | "Micro-break" |
| Progress section | hidden | hidden | visible | visible |
| Mid-zone content | rest questions | rest questions | round counter | round counter |
| Ring strokes | opacity 0 | visible | visible | visible |
| "Ready?" ring text | visible | hidden | hidden | hidden |
| Countdown digits | hidden | visible | visible | visible |
| Micro-break message | — | — | — | visible |
| Controls row | hidden | visible | visible | visible |
| Start Practice button | visible | hidden | hidden | hidden |
| Close button | hidden | hidden | visible | visible |
| Info button | visible | visible | hidden | hidden |
| Review Recording button | hidden | hidden | hidden | visible (if recording enabled and blob ready) |

---

# Details for the countdown ring timer

## Countdown Ring Timer — Robust Implementation

### SVG Structure

The ring is an SVG `<circle>` element. The circumference is `2 * π * r`. For `r = 110` in a `256×256` viewBox, this is `691.150`. The circle must start at the 12 o'clock position, which requires a **native SVG transform attribute** — not a CSS `transform` — on the `<circle>` element itself:

```html
<circle id="ring-fg" cx="128" cy="128" r="110"
  stroke-width="10" fill="none"
  stroke-dasharray="691.150"
  transform="rotate(-90 128 128)"
  style="stroke-dashoffset: 691.150"/>
```

The initial `stroke-dashoffset` must be set as an **inline style** (not a presentation attribute, and not in the stylesheet), so that CSS transitions see it as a style-layer value from the start.

### Why CSS `transform` Fails

Applying `transform: rotate(-90deg); transform-origin: 50% 50%` via a CSS rule to an SVG element is unreliable in Chrome and Safari — the rotation may be ignored or misapplied. Always use the SVG `transform` attribute with an explicit rotation center: `rotate(degrees cx cy)`.

### Animating the Ring

The `stroke-dashoffset` must be updated via **`element.style.strokeDashoffset`**, not via `element.setAttribute('stroke-dashoffset', value)`. Presentation attributes set via `setAttribute` sit below the CSS cascade and do **not** trigger CSS `transition` in Chrome or Safari. Setting via `.style` places the value in the inline style layer, which does participate in transitions.

```js
const RING_R = 110;
const RING_C = +(2 * Math.PI * RING_R).toFixed(3); // 691.150

// In render(), when updating the ring:
const frac   = phaseTimeLeft / phaseTotalDuration;          // 1.0 = full, 0.0 = empty
const offset = RING_C * (1 - Math.min(1, Math.max(0, frac)));
elRingFg.style.strokeDashoffset = offset.toFixed(3);
```

A `frac` of `1.0` gives offset `0` (full ring drawn). A `frac` of `0.0` gives offset `691.150` (ring fully hidden). This is the correct direction for a countdown.

### CSS for the Ring

```css
#ring-fg {
  fill: none;
  stroke: rgba(255,255,255,0.72);
  stroke-width: 10;
  stroke-linecap: round;
  stroke-dasharray: 691.150;
  transition: stroke-dashoffset 0.4s linear;
  /* No stroke-dashoffset here — set it via inline style on the element */
  /* No transform here — set it via SVG transform attribute on the element */
}
```

Do **not** include `stroke-dashoffset` or `transform` in the CSS rule. Both must live on the element itself (as shown above) to work correctly cross-browser.