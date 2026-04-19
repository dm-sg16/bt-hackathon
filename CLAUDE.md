# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repo is a collection of tools and services that support running BT Hackathons — from judging to any future tooling (registration, scheduling, leaderboards, etc.). Each service lives in its own directory (or as a standalone file for simple tools) and is deployed independently via GitHub Pages or other hosting.

### Services

| Service | Location | Description |
|---------|----------|-------------|
| Judging App | `index.html` | In-browser scoring tool for judges — no backend, no dependencies |

As new services are added, document them here with their location, tech stack, and how to run/deploy them.

## Running Locally

Open `index.html` directly in a browser, or serve it with any static file server:

```
python3 -m http.server
npx serve .
```

## Architecture

Everything is in one `index.html` with three "screens" toggled via a `.active` CSS class (`display: flex` vs `display: none`). There is no routing library.

```
screen-setup   → screen-judging   → screen-summary
```

All runtime state lives in a single `st` object:

```js
const st = {
  teams,     // string[] — team names in presentation order (max 8)
  scores,    // object[] — one per team, keyed by criterion id
  idx,       // current team index
  timeLeft,  // seconds remaining on timer
  running,   // timer running?
  iv,        // setInterval handle
};
```

## Scoring Criteria

Defined in the `CRITERIA` array. Each criterion has an `id`, `label`, allowed `values`, and display strings. Current criteria:

| id           | label            | range     |
|--------------|------------------|-----------|
| execution    | Execution        | 0 – 4     |
| difficulty   | Level of Difficulty | 0 – 4  |
| ai           | AI Proficiency   | 0 to −2   |
| ambition     | Ambition         | 0 to +2   |
| presentation | Presentation     | 0 to +1   |

Max possible score per team: **11** (4 + 4 + 0 + 2 + 1).

To add or change a criterion, edit only the `CRITERIA` array — the rendering, scoring, summary table, and clipboard export all derive from it dynamically.

## Timer

`DURATION` (default 360 s / 6 min) is a top-level constant. Timer resets automatically when navigating to a new team. Progress bar and digit colour shift: normal → warning (≤120 s) → critical/pulsing (≤60 s).

## Deployment

The `gh-pages` branch serves the live site. To deploy, push `index.html` to `gh-pages` (or merge/cherry-pick from `main`).

## Key Conventions

- **No framework.** Keep it vanilla JS — no npm, no bundler.
- **CRITERIA-driven rendering.** Never hard-code criterion ids outside the `CRITERIA` array; all screens read from it.
- **Score state is nullable.** `null` means "not yet scored"; `0` is a valid score. The "Score all fields" guard checks `=== null`, not falsiness.
- **Clipboard output is TSV** (tab-separated), intended for pasting into spreadsheets. `buildResultsText()` is the single source of truth for both the in-progress "Copy" button and the final "Copy Results" button.
