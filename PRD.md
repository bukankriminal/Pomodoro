# PRD: Apple-Style Pomodoro Timer

## How the agent must use this file

1. Read this file and `AGENTS.md` / `knowledge.md` (project rules) fully before starting.
2. Work on **one phase at a time**, in order. Never start a phase until the previous one is marked done.
3. After finishing a phase: run the acceptance checks, tick the checkbox in "Progress" below, then **stop and report** in this format:
   - What changed (one line)
   - Acceptance results (pass/fail per item, and what you could not verify)
   - Open questions (only if any)
   Then wait for the user to reply "next" before starting the next phase.
4. If the user reports a problem with a finished phase, fix only that problem. Do not move on and do not touch other phases' code.
5. If this PRD is unclear, or two readings give different results, ask one short question before coding. Do not guess.
6. Do not add features that are not listed here.

## Product

A single-page pomodoro timer that feels like the iOS Clock app: ring timer, soft synthesized sounds, a gentle end alarm, and a task list under the timer.

- Single file: `index.html` (HTML + CSS + JS inline). No frameworks, no npm packages, no CDN, no external assets, no build step.
- UI copy in English, short and functional (Start, Pause, Resume, Reset, Done).
- Latest Safari, Chrome, Firefox. Works from 360px width up.

## Progress

- [x] Phase 1: Static visual shell
- [x] Phase 2: Working timer and animation
- [x] Phase 3: Sound effects
- [x] Phase 4: Gentle alarm
- [x] Phase 5: Task list and polish

---

## Phase 1: Static visual shell (no logic)

**Goal:** the screen looks right before anything works. No timer logic, no sound, no tasks yet.

**Build**
- Single column, max-width 420px, centered.
- iOS-style segmented control: Focus / Break / Long Break (static, Focus active, white indicator).
- SVG timer ring: 280px, 6px stroke, round caps, thin gray track, accent progress shown at about 70%. Center: `25:00` at 64px, weight 200-300, tabular-nums, with a small label `Focus` beneath.
- One pill primary button `Start` and a plain-text `Reset` button.
- Four small session dots.
- Empty space below for the task list (added in Phase 5).

**Design tokens (CSS variables, auto light/dark via `prefers-color-scheme`)**

| Token | Light | Dark |
|---|---|---|
| --bg | #F2F2F7 | #000000 |
| --surface | #FFFFFF | #1C1C1E |
| --separator | rgba(60,60,67,0.18) | rgba(84,84,88,0.45) |
| --text | #000000 | #FFFFFF |
| --text-secondary | rgba(60,60,67,0.6) | rgba(235,235,245,0.6) |
| --accent | #FF3B30 | #FF453A |
| --accent-break | #34C759 | #30D158 |

Font: `-apple-system, BlinkMacSystemFont, "SF Pro Display", system-ui, Inter, sans-serif`. 8px spacing grid. Radii: 12px rows, 999px pills, 10px segmented control.

**Acceptance**
- [x] Looks correct in both light and dark mode at 360px and 1280px width.
- [x] Every color comes from a CSS variable; no gradients, glow, emoji, or shadows.
- [x] Digits use tabular-nums and do not shift layout.

**Stop.** Report and wait for "next".

---

## Phase 2: Working timer and animation

**Goal:** the timer runs correctly and animates smoothly. Change only what is needed.

**Build**
- Durations as constants at the top: Focus 25, Break 5, Long Break 15.
- Clicking a segment switches mode, resets the timer, slides the indicator (`cubic-bezier(0.32, 0.72, 0, 1)`, 350ms), and transitions the accent color (400ms): red for Focus, green for breaks.
- Start / Pause / Resume toggles label and state. Reset restores the full duration.
- Remaining time is computed from a target timestamp (`Date.now()`), never by summing intervals.
- Ring progress is driven by `requestAnimationFrame` from real elapsed time: continuous, no per-second jumps.
- Digits crossfade vertically on change (translateY 4px + opacity, 200ms), fixed width.
- Buttons: scale 0.96 on press, 120ms. No hover scale.
- Session dots fill after each finished Focus session; after 4, the next break is Long Break.
- At zero: stop and show label `Done` (alarm comes in Phase 4).
- Respect `prefers-reduced-motion`.

**Acceptance**
- [x] Timer stays accurate after switching to another tab for 30 seconds and back.
- [x] Ring moves smoothly with no visible stutter.
- [x] Switching modes mid-run resets cleanly with no leftover state.
- [x] Pause then Resume continues from the correct remaining time.

**Stop.** Report and wait for "next".

---

## Phase 3: Sound effects

**Goal:** soft synthesized sounds on interactions. Web Audio API only, no audio files.

**Rules**
- One shared `AudioContext`, `resume()` on the first user interaction.
- Sine oscillators with a `GainNode` envelope, master gain 0.15-0.25, attack about 3ms, `exponentialRampToValueAtTime` to near zero before `stop()` so there are no clicks or pops.
- Imitate the character of Apple sounds (soft, short, clean). Do not use or copy real Apple audio.

**Sounds**

| Event | Spec |
|---|---|
| Tap / mode change | 1200Hz, 40ms, gain 0.12 |
| Start | 660Hz then 880Hz, 90ms each, 60ms gap, gain 0.2 |
| Pause | 660Hz sliding to 520Hz, 120ms |
| Reset | single 440Hz, 80ms |

**Build**
- All sound parameters in one config object at the top, with a short comment per field.
- Small mute button (SVG speaker icon, 1.5px stroke) in a top corner. Preference saved in `localStorage` (try/catch). Muted means no sound at all.

**Acceptance**
- [x] No click or pop at the start or end of any sound.
- [x] Mute silences everything and persists after refresh.
- [x] App works normally if `localStorage` is unavailable.

**Stop.** Report and wait for "next".

---

## Phase 4: Gentle alarm

**Goal:** an end-of-timer alarm that informs without startling.

**Build**
- Sound: 3-note rising chime (523, 659, 784Hz), about 400ms each, 40ms attack, about 900ms release, light synthetic reverb (short low-feedback delay or a simple convolver).
- Volume ramps from gain 0.05 to 0.22 over 2 seconds. Never suddenly loud.
- Plays at most 2 times with an 8 second gap, then stops by itself. Respects mute.
- One tap anywhere on the timer screen stops it with a 300ms fade-out.
- Visual: ring pulses once softly (opacity 1 to 0.7 to 1 over 1.2s), label becomes `Done`, two actions fade in: `Start Break` (primary pill) and `Add 5 Minutes` (text button).
- Background tab: tab title changes to `Done · 00:00` and returns to normal after the user acts.
- Request Notification permission only after the user first presses Start, never on page load. Use `silent: true` because the app already plays the sound.
- Respect `prefers-reduced-motion` for the pulse.

**Acceptance**
- [x] Alarm never loops forever and never starts loud.
- [x] One tap stops it cleanly with no click.
- [x] `Start Break` and `Add 5 Minutes` both work.
- [x] No flashing or shaking effects.

**Stop.** Report and wait for "next".

---

## Phase 5: Task list and polish

**Goal:** a tidy task list directly below the timer (32px gap), then a final cleanup pass.

**Build**
- iOS grouped-list style: one rounded surface (12px radius), rows separated by a 0.5px separator inset from the left, no shadows.
- Top row is an `Add task` input. Enter saves; the input clears and stays focused. Empty or whitespace-only titles are ignored silently. Long titles truncate with a single-line ellipsis.
- Each row: check circle on the left, title, small pomodoro counter on the right that increments when a Focus session finishes while that task is active.
- Tap a row to make it the active task: its name shows as the label under the timer digits, with a small accent dot on the row. Only one active task.
- Check-off: circle fills, checkmark draws via `stroke-dashoffset` (200ms), text dims and strikes through, then the row moves to a collapsible `Done` group.
- Delete: swipe left on touch, or a button on hover/focus on desktop, with 5 second undo. Double-click a title to edit (Enter saves, Esc cancels).
- Animation: new row expands in height + fade (250ms); deleted row fades and collapses. Use `cubic-bezier(0.32, 0.72, 0, 1)`.
- Sounds (add to the existing config): soft pop on add (880Hz, 70ms), three quick rising notes on check-off (784, 988, 1175Hz, 55ms each), short lowpass-noise swish on delete (90ms, gain 0.06).
- Empty state: `No tasks yet. Add one to begin.` No illustration.
- Persist in `localStorage` (try/catch). Keyboard accessible with visible focus rings; touch targets at least 44px.

**Final polish pass (after the list works)**
- Remove anything decorative without a function.
- Replace any off-token color with a CSS variable.
- Delete any animation without a purpose.
- Remove any emoji or gradient.
- Check spacing follows the 8px grid.

**Acceptance**
- [x] Add, activate, check off, edit, delete, and undo all work, and tasks persist after refresh.
- [x] Fully usable with keyboard only.
- [x] Works with sound muted, `localStorage` unavailable, and reduced motion on.
- [x] No console errors.
- [x] Looks at home next to the iOS Clock app.

**Stop.** Report that the project is complete.
