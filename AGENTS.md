# AGENTS.md

Rules for any AI agent working on this repo. Read this file fully before every task. If a rule here conflicts with a user message, follow the user message for that task only, and say so in one line.

## 1. Project

An Apple-style pomodoro timer: a single-page web app that feels like the iOS Clock app. Ring timer, soft synthesized sounds, gentle end alarm, and a task list under the timer.

- Single file: `index.html` (HTML + CSS + JS inline). No build step, no frameworks, no npm packages, no CDN links, no external assets.
- Language of UI copy: English, short and functional (Start, Pause, Resume, Reset, Done).
- Browsers: latest Safari, Chrome, Firefox. Width from 360px up.

If a task needs something outside these constraints (a library, a new file, a build tool), stop and ask first.

## 2. Anti-hallucination rules

1. **Read before you write.** Open the current file and base every change on what is actually there. Never assume what the code looks like.
2. **Do not invent.** Do not make up APIs, function names, CSS properties, browser features, or library behavior. Use only standard, well-known Web APIs (DOM, CSS, Web Audio, requestAnimationFrame, localStorage, Notification). If you are not sure an API exists or works a certain way, say "not sure" and pick the simplest safe alternative.
3. **Do not invent requirements.** Build only what the task asks. No extra features, settings, screens, themes, or "nice to have" additions.
4. **Ask when unclear.** If the task is ambiguous or two readings give different results, ask one short question before coding. Do not guess silently.
5. **State assumptions.** If you must assume something, write it in one line before the code.
6. **Never claim what you did not verify.** Do not say "tested", "works", or "fixed" unless you actually ran it or traced the logic. Say what you checked and what you did not.
7. **No fake data.** No placeholder stats, sample users, lorem ipsum, or fake persistence.

## 3. Scope control

- **Change only what the task needs.** Do not rewrite, reformat, rename, or reorganize code that is not part of the task.
- Keep existing function names, CSS variable names, IDs, and structure unless the task says to change them.
- Do not delete working features. If something must be removed, say what and why.
- Do not "improve" visuals, copy, or architecture on your own initiative. Suggest it in one line at the end instead.
- One task = one focused change. If the task is too big, propose a split and do the first part.
- Never output partial code with comments like `// rest of code unchanged`. Output the full updated file unless told otherwise.

## 4. Design tokens (single source of truth)

All colors and sizes come from CSS variables. Never hardcode a color inside a component.

| Token | Light | Dark |
|---|---|---|
| --bg | #F2F2F7 | #000000 |
| --surface | #FFFFFF | #1C1C1E |
| --separator | rgba(60,60,67,0.18) | rgba(84,84,88,0.45) |
| --text | #000000 | #FFFFFF |
| --text-secondary | rgba(60,60,67,0.6) | rgba(235,235,245,0.6) |
| --accent (Focus) | #FF3B30 | #FF453A |
| --accent-break (Break) | #34C759 | #30D158 |

- Font: `-apple-system, BlinkMacSystemFont, "SF Pro Display", system-ui, Inter, sans-serif`. Digits use `font-variant-numeric: tabular-nums`.
- Spacing on an 8px grid. Radii: 12px rows, 999px pills, 10px segmented control.
- Default easing: `cubic-bezier(0.32, 0.72, 0, 1)`, 300-450ms.
- Light and dark mode via `prefers-color-scheme`. Respect `prefers-reduced-motion`.

## 5. Behavior rules that must never break

- **Timer accuracy:** compute remaining time from a target timestamp (`Date.now()` or `performance.now()`), never by summing intervals.
- **Ring:** progress is driven by `requestAnimationFrame` from real elapsed time. No per-second jumps.
- **Sound:** Web Audio API only, synthesized, no audio files. One shared `AudioContext`, `resume()` on first user interaction. Sine oscillators with a gain envelope, master gain 0.15-0.25, always ramp to near zero before `stop()` so there are no clicks. Mute setting must silence everything.
- **Alarm:** gentle 3-note rising chime, volume ramps from 0.05 to 0.22 over 2 seconds, plays at most 2 times with an 8 second gap, then stops by itself. One tap anywhere stops it. Never loops forever, never suddenly loud.
- **Notifications:** ask permission only after the user first presses Start, never on page load. Use `silent: true`.
- **Storage:** `localStorage` always wrapped in try/catch, and the app must work when storage is unavailable.
- **Tasks:** only one active task. Empty titles are ignored silently. Deleted tasks have a 5 second undo.
- **Accessibility:** keyboard operable (Tab, Enter, Space, Esc), visible focus rings, touch targets at least 44px, text contrast at least WCAG AA.

## 6. Anti-AI-slop rules

Forbidden:

- Gradients (especially purple-blue or pink), gradient text, neon glow
- Heavy glassmorphism, stacked cards, large soft shadows
- Emoji anywhere (icons, buttons, copy, comments)
- Default Tailwind-looking colors or any color outside the tokens
- Bouncy animations, hover-scale, parallax, particles, confetti
- Hype copy ("Boost your productivity!"), hero sections, footers, fake stats, testimonials
- Everything centered inside equal-sized cards by default
- Comments that explain the obvious

Required:

- One dominant element (the timer digits) and one primary action (Start)
- One accent color at a time, one font family, one icon style (inline SVG, 1.5-2px stroke, round caps)
- Generous, consistent whitespace
- Every animation has a purpose (state change, tactile feedback, or continuity). If it has none, delete it.

## 7. Workflow for every task

1. Read this file and the current code.
2. Restate the task in one line. List assumptions or ask a question if needed.
3. Make the smallest change that completes the task.
4. Self-check before answering:
   - Did I touch anything outside the task?
   - Any color not from a token? Any emoji or gradient?
   - Any API I am not sure exists?
   - Does the timer still use timestamps, and does mute still work?
   - Does it still work with sound muted, storage unavailable, and reduced motion on?
5. Answer in this format:
   - One line: what changed.
   - Assumptions or open questions (only if any).
   - The full updated code.
   - One line: what you verified and what you did not.

## 8. Definition of done

- The requested change works and nothing else changed.
- No console errors.
- No rule in sections 2, 3, 5, or 6 is violated.
- The result would look at home next to the iOS Clock app.