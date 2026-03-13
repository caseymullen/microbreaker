# Music Practice Timer — Specification

## Concept

A Progressive Web App (PWA) designed for musicians to encourage structured micro-breaks during practice. Modeled on HIIT-style interval training: short focused work bursts alternating with brief mental resets (micro-breaks), grouped into larger timed blocks.

-----

## Intended deployment environment
Optimize for an iPhone or iPhone mini. Use UI techniques that will work reliabily in Safari on an iPhone. Use WebKit-compatible patterns. Ensure that the iPhone status bar color matches the color of the app. 

Try to make it look "native" on the iPhone. For example, you might need to do something like is shown in the code block, below. If the app is displaying "under" the status bar, ensure that we don't try to show actual content under the status bar.
```code
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
```
-----

## Timer Structure

### Round (inner loop)

- **Practice phase** — active playing/work. AKA "work"
- **Micro-break phase** — brief mental rest; brain consolidates recent practice. AKA "break"
- rounds repeat automatically until the macro block is done
- The length of a block is specified in rounds, but also show the user the duration in minutes and seconds.
- After the final work round, do NOT skip the micro-break. The micro-break is the only opportunity the user has to review their recording.


### Block (outer loop)

- A timed block containing many rounds
- When the block expires, play a small "success fanfare" and then begin the "rest" phase.


### REST phase
The rest phase is a bit odd because there are essentially two sub-phases that share the same background color and some UI elements. There is the rest-countdown phase, where the user sees a countdown timer (with the two questions above it). When the rest-countdown phase ends, then the "back to work" alarm goes off (that's the one with the "da-da-da da-da DA!" rhythm). This alarm marks the end of the rest-countdown phase and the start of the rest-waiting phase, where we are just waiting for the user to hit the "Start Practice" button.

A "start practice" button is visible during the entire Rest phase (both sub-phases). Pressing it will start the block (the first work phase of the block). The button appears where the "audio player" controls appear during other phases (see below).

During the rest-countdown phase, the phase label is "REST".
During the rest-waiting phase, omit the phase label. The user will see the large "Ready?" text in the countdown ring (as described elsewhere).

When the user first opens the app, it starts in the rest-waiting phase. The user sees "Ready?" in the "countdown timer ring" area, and they contemplate the two questions until they hit the "Start Practice" button.

-----

## Default Durations (all adjustable in Settings)

|Timer         |Default|Range    |
|--------------|-------|---------|
|Practice phase|45 sec |15–120 sec|
|Micro-break   |15 sec |10–60 sec |
|Block length  |5 rounds |3–15 rounds |
|Rest     |2 min|1–5 min|

-----

## Controls

The main controls look like audio-player controls
- A "next track" button skips to the next phase (work → break → work, or rest when you reach the end of the last work phase in the block)
- A "Play/Pause" button — freezes all timers
- A "Previous track" button restarts the current phase

Note that when the user hits the "Next track" and "previous track" buttons, the real elapsed time starts diverging from the time that should be used for displaying the block progress bar. We have two time concepts: progressTime and elapsedTime
- progressTime gets decremented or incrementted appropriately when the user hits "previous track" or "next track". Hitting "pause" also stops the progressTime
- elapsedTime shows the real-world elapsed clock time and is unaffected by "previous track", "next track" or "play pause" or going into review mode or settings.

Leave space below the "audio-player" controls for the "Review Recording" button to appear. Also leave a little bit of "blank" space at the bottom of the screen, even when the "Review Recording" is showing.

A **Settings button** is in the lower right. It should look like a gear. It opens the settings panel; pauses timer while open

Listen for keyboard input: "Space bar" will play/pause. "Enter" acts like "next track" (play/pause)

In "settings", display the duration of the block directly below where you display the text showing the number of rounds.
In "settings", just use the label "Block", rather than "Block length".

-----

## Visual Design

- Full-screen color themes per phase:
  - **Practice** — warm rusty deep orange
  - **Micro-break** — deep blue/indigo
  - **Rest** — black
- Circular countdown ring for the timer during work, break, and rest. Either use a CSS-based approach or, if you use a SVG approach, ensure that SVG values are mathematically precise and declared inline rather than via stylesheet. The same applies to the "block progress bar", discussed next.
- Display a progress bar near the top of the screen to show progress towards completion of the block. In addition to the moving bar:
  - Below the bar show the time progress, e.g. "0:23 of 5:00
  - The graphical progress bar is governed by the progressTime, whilse the textual time progress shows elapsedTime. This way the user can see how much time they actually spent in the block, compared to how much time was allocated to the block.
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
- A small "success fanfare" plays at the end of the block
- The "alarm" at the end of rest timer has a rhythm like "da-da-da da-da DA!"
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

During "rest" display these questions, on separate lines, in a beige font, halfway between the "countdown timer ring" area and the top phase label:
- What is the goal of your practice?
- How will you achieve it?"

That is the default list of phrases. They can be edited in settings. There can be  All settings are stored in local storage and thus stay in place when the app reloads on the same device.

Also make the "two questions" editable in the settings. There can only be two. It is ok if one or both is blank.

Have a button to reset all settings to their defaults. Ask the user if they also want to clear the "micro-break messages". Allow the to answer yes, no, or cancel.


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

Do not remember the current state of progress through a given block in localStorage.




## Implementation Notes: Audio Recording

### Recording lifecycle
- Recording starts when the **work phase begins** and stops when that phase ends
  (i.e. when the break phase starts, or the block completes).
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
