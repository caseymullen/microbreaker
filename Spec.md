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


### Block (outer loop)

- A timed block containing many rounds
- When the block expires, play a small "success fanfare" and then begin the "rest" phase.
- A "start practice" button is visible during the Rest phase. Pressing it will start the block. It appears where the "audio player" controls appear during other phases (see below).
- When the app first loads, it essentially starts in the "Rest" phase, as if the "back to work" alarm had already played.
- When in the "back to work", hide the countdown timer. In its place, display "Ready?"


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
- elapsedTime shows the real elapsed clock time and is unaffected by "previous track", "next track" or "play pause".

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
- When the "rest" countdown ring finishes, play an "alarm" evocative of "back to work".
- After the alarm, we are in a rest-waiting-to-start phase. The top label still shows "Rest", but the countdown ring disappears and is replaced by the text "Ready?" in the same font and size used for the countdown.


Make fonts large enough for easy readability. No font should be smaller than 16px. 
Use "DM Mono" for all text other than "messages"

Add subtle and gradual darkness variation to the background so that it is darker at the edges of the screeen than at the center. In the "Rest" phase, this means the base background color should be dark grey (fading to black at the edges).

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


-----

## Recording Audio

Always be recording audio during the work phase. If the user pauses during the work or break phase, display a "Review Recording" button below the "audio player" controls. If user hits "Review Practime", pop up a mini audio player that covers as much of the screen as needed. The audio clip is only for the most-recent work phase. It will have its own "audio player" controls that are not just an analogy, but control the playing of the audio clip. The controls Include:
- Back to the start
- Jump back 5 seconds (a simple small triangular arrow pointing to the left)
- Play/Pause
- Jump forward 5 seconds (a simple small triangular arrow pointing to the left)

A "Close" button at the bottom.

In "settings" there should be a toggle "Record/review practice" that defaults to "on". When it is off, never show the "Review Recording" button.

The controls during review of the recording should be large for easy clicking. The user is probably holding their violin and bow while operating the controls.
While the "Review Recording" audio player is active, the space bar will trigger play/pause and the "Enter" key will restart the audio from the beginning.

While reviewing the recording, make sure that screen does now allow the user to zoom the web page in or out.

When the user pauses during the practice phase, also pause audio recording, and restart it when they un-pause.
When the user hits "Review Recording", stop recording in order to make the audio clip available for playback.
When a new practice phase starts, discard any old recordings.

-----

## PWA / iPhone Notes

- Single self-contained HTML file (no dependencies to download)
- `apple-mobile-web-app-capable` meta tag — runs full-screen when added to home screen
- Install: open in Safari → Share → Add to Home Screen
- Audio context resumes on tap after any suspension

-----

## Other notes

Do not remember the current state of progress through a given block in localStorage.

The rest phase is a bit odd because there are essentially two sub-phases. There is the rest-countdown phase, where the user sees a countdown timer (with the two questions above it). When the rest-countdown phase ends, then the "back to work" alarm goes off (that's the one with the "da-da-da da-da DA!" rhythm). This alarm marks the end of the rest-countdown phase and the start of the rest-waiting phase, where we are just waiting for the user to hit the "Start Practice" button.

Actually, it might be clearer if we actually make these two more distinct phases from the user's perspective, by giving the rest-waiting phase the label "Ready?" to be displayed at the top of the screen.

When the user first opens the app, it starts in the rest-waiting phase. The user sees "Ready?" at the top and in the "countdown timer ring" area, and they contemplate the two questions until they hit the "Start Practice" button.
