# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Math Kingdom is a second-grade math game — a **single self-contained HTML file** (`index.html`) with no build system, no package manager, and no dependencies beyond two Google Fonts loaded via CDN. To run it: open `index.html` in a browser. No server required.

## Architecture

Everything lives in `index.html`, organized into three sections:

1. **`<style>` block** — all CSS using CSS custom properties (`--sky`, `--sun`, etc.) defined on `:root` for the color palette. No external stylesheet.
2. **HTML body** — static structural scaffolding: the HUD, topic grid, game area container (`#gameArea`), modal, and confetti wrapper. The quiz card and tutorial panel are injected dynamically via JS.
3. **`<script>` block** — all game logic in vanilla JS (no framework).

### State Management

A single mutable `state` object holds all runtime state. The fields `level`, `xp`, `xpToNext`, `stars`, and `topicStars` are persisted to `localStorage` under the key `mathKingdomSave` via `saveState()` / `loadSave()`. Everything else (`topic`, `streak`, `currentQ`, `triesLeft`, `phase`, `answered`) is session-only and resets on page load.

### Question Generator Pattern

`generators` is a plain object keyed by topic name (`addition`, `subtraction`, `multiplication`, `counting`, `fractions`, `placevalue`). Each generator is a function `(retry=false) => QuestionObject`. A `QuestionObject` has:
- `q` — the question string displayed to the player
- `ans` — the correct answer (string or number; always compared with `String()`)
- `opts` — shuffled array of 4 answer choices
- `vis` (optional) — emoji string shown as a visual hint after a wrong answer
- `steps` — array of `{ text, math, visual }` objects used by the tutorial panel

Difficulty scales with `state.level` inside each generator (e.g., addition uses max 20 below level 3, up to 99 at higher levels).

### Game Flow

```
selectTopic() → renderQuestion() → checkAnswer()
                                      ↓ correct → addXP() → updateHUD() → [showLevelUp()]
                                      ↓ wrong (triesLeft > 0) → show hint / visual
                                      ↓ wrong (triesLeft = 0) → showTutorial() → startRetry() → renderQuestion(isRetry=true)
```

`renderQuestion()` always rebuilds `#gameArea` innerHTML from scratch. The "Next Question" button calls `nextQuestion()` which delegates back to `renderQuestion(false)`.

### XP & Leveling

`addXP(amount)` increments `state.xp`. On overflow past `state.xpToNext`, the level increments and `xpToNext` scales by 1.4×. The `pets` array maps level index to `{ emoji, name, unlock }` — pets update automatically in the HUD and level-up modal.

XP rewards: first-try correct = 15 XP; second-try correct = 10 XP; retry (after tutorial) correct = 5 XP, no stars awarded.
