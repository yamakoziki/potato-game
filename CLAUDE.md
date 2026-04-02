# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

Open `index.html` directly in any modern web browser. No build process, server, or dependencies required.

## Architecture

The entire application lives in a single file: `index.html` (~1,902 lines).

**Layout of index.html:**
- Lines 1–789: HTML structure and CSS styling (mobile-first, max-width 480px)
- Lines 790–800: `state` object — the single source of truth for all game state
- Lines 803–880: `KNOWLEDGE_CARDS` object (keyed by string ID, e.g. `variety_jyoti`) — educational cards (Japanese)
- Lines 882–1124: `PHASES` array — 11 farming phases, each with choices and score deltas
- Lines 1125–1295: `TRAP_EVENTS` — season-specific random challenge events (keyed by season)
- Lines 1297–1451: Sales phase logic — `renderSalesPhase` and `renderSalesResult`, dynamically generated based on current scores
- Lines 1452–1749: `render()` dispatcher and all `render*()` functions — one per screen
- Lines 1751–1863: Action handlers (`makeChoice`, `makeTrapChoice`, `nextPhase`, `startGame`, `restartGame`, etc.)
- Lines 1865–1896: `showAlbum()` — knowledge card overlay display
- Line 1899: `render()` initialization call

**Game flow (state machine):**
```
setup → phase (×10) → [trap → trap_result] → sales_result → final
                                                           ↘ gameover (harvest ≤ 5)
```

The `state.screen` string controls which render function fires. All score changes are applied as deltas to five 0–100 metrics: `harvest`, `quality`, `health`, `env`, `community`.

**Key data patterns:**
- `state.season`: `'summer'` or `'winter'`; `state.region`: `'terai'`, `'hill'`, or `'mountain'`
- `state.history`: string array of fired trap IDs (format: `'trap_' + trap.id`) — prevents re-triggering
- Each entry in `PHASES` has a `choices` array; each choice has a `result` with a `scores` object (metric deltas) and optionally a `card` key (string ID) to award a knowledge card
- Trap events are triggered between phases based on `state.season` and `state.history`
- Sales options are conditionally unlocked: direct sales requires `quality ≥ 60`, cooperative requires both `harvest ≥ 55` and `quality ≥ 55`
- Final rank thresholds (average of all five metrics): ≥85 Master, ≥70 Veteran, ≥55 Full-fledged, ≥40 Apprentice, <40 Beginner
