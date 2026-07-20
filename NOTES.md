# Project Notes

Internal notes for PreCheck. Not user-facing — see README.md for that.

## Current state

Single-file React app (`index.html`, ~840 lines: markup + `<style>` + one
`<script type="text/babel">` block). No build step — React 18, ReactDOM 18,
and Babel Standalone load from unpkg with SRI hashes pinned in the `<script>`
tags. Everything else (validation, prompt assembly, persistence) runs
client-side with no network calls after first load.

State lives in a single `App` component via `useState` — no context, no
external state library. Session data persists to `localStorage` under the key
`precheck.sessions` as a JSON array of records:

```js
{ id, timestamp, problem, attempt, hypothesis, confirmed, passed, promptGenerated }
```

Three commits so far, all README polish plus the initial add — the app itself
hasn't changed since it was first written.

## Known issues / gotchas

- **`passed` is always `true`.** A session only gets written to `localStorage`
  at all once `allValid` is true (all four gate conditions met), so every
  record in history is a pass by construction — there's no failure path or
  partial-credit case. The `passed` field and the `[PASS]` badge in the
  history list are therefore currently redundant with "exists in the array."
  If a "failed attempt" concept is ever wanted, `handleUnlock` needs to allow
  logging without full validation, which changes the whole gate semantics —
  worth deciding deliberately rather than half-adding it.
- **No data export or migration path.** Everything lives in one browser's
  `localStorage`. Clearing site data, switching browsers, or using a different
  machine loses all history and streak progress silently — there's no warning
  about this anywhere in the UI. Matches the "Export history as CSV/JSON" item
  already on the README roadmap.
- **Streak calculation is naive but correct for the single-user case.**
  `computeStreak` walks backward day-by-day from "now" using the browser's
  local timezone via `Date` methods (not UTC). A user traveling across
  timezones or changing their system clock could see the streak jump or reset
  unexpectedly. Not worth fixing unless it's actually reported as an issue.
- **CDN dependency on first load.** The SRI-pinned unpkg scripts mean the app
  can't run fully offline until the browser has cached them once. Worth a
  one-line callout if this ever gets issues from users trying to run it fully
  air-gapped.
- **No tests.** There is no test tooling of any kind in the repo (no
  package.json, no CI config). Given the single-file/no-build philosophy,
  introducing a full test harness would be a real trade-off against the
  project's stated simplicity — if this changes, decide up front whether to
  keep using in-browser Babel or finally take on a build step (Vite, etc.).

## Architecture decisions worth remembering

- **In-browser Babel over a build step is deliberate**, not a shortcut taken
  under time pressure — the whole pitch of the project ("one HTML file, zero
  install") depends on it. Any future feature work should bias toward keeping
  this true rather than reaching for a bundler at the first sign of friction.
- **Terminal aesthetic is a design constraint, not just a skin.** Square
  corners, monospace everywhere, and the locked/unlocked color-temperature
  swap (cold slate vs warm amber) are called out explicitly in the README —
  treat these as intentional when touching CSS, not incidental defaults to
  "clean up."
- **The 40-character attempt minimum (`MIN_ATTEMPT_LENGTH`) is a hardcoded
  constant** at the top of the script block, not a prop or config value — easy
  to tune but currently has no UI for a user to change it themselves.

## Ideas for next steps

- Export history as CSV/JSON (already on the README roadmap; would also
  address the no-migration-path gotcha above).
- Per-language/per-topic tagging on sessions (also on roadmap) — would need a
  new optional field in the session record and a small tag input in the form.
- Consider a lightweight "reset streak" or "clear history" control — right
  now the only way to clear `localStorage` is via browser dev tools or site
  settings, which isn't discoverable for the target (practicing coder)
  audience.
- If test coverage is ever wanted without abandoning the no-build philosophy,
  look at something that can run against the rendered DOM directly (e.g.
  Playwright driving the static `index.html`) rather than a JS unit-test
  framework that would imply a build step.
