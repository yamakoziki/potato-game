# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

Open `index.html` directly in any modern web browser. No build process, server, or dependencies required.

## Architecture

The entire application lives in a single file: `index.html` (~1,900 lines).

**Layout of index.html:**
- Lines 1–783: HTML structure and CSS styling (mobile-first, max-width 480px)
- Lines 784–795: `state` object — the single source of truth for all game state
- Lines 798–874: `KNOWLEDGE_CARDS` array — 14 educational cards (Japanese)
- Lines 877–1117: `PHASES` array — 11 farming phases, each with choices and score deltas
- Lines 1120–1173: `TRAP_EVENTS` — season-specific random challenge events
- Lines 1176–1287: Sales phase logic — dynamically generated based on current scores
- Lines 1300–1480: `render*()` functions — one per screen, called by `render()`
- Lines 1747–1858: Action handlers (`handleChoice`, `handleTrap`, `handleSales`, etc.)
- Lines 1860–1891: Knowledge album/card display system
- Line 1894: `render()` initialization call

**Game flow (state machine):**
```
setup → phase (×10) → [trap → trap_result] → sales_result → final
                                                           ↘ gameover (harvest ≤ 5)
```

The `state.screen` string controls which render function fires. All score changes are applied as deltas to five 0–100 metrics: `harvest`, `quality`, `health`, `environment`, `community`.

**Key data patterns:**
- Each entry in `PHASES` has a `choices` array; each choice has a `scores` object with metric deltas and optionally a `card` index to award a knowledge card.
- Trap events are triggered between specific phases based on `state.season`.
- Sales options are conditionally unlocked: direct sales requires `quality ≥ 60`, cooperative requires both `harvest ≥ 55` and `quality ≥ 55`.
- Final rank thresholds (average of all five metrics): ≥85 Master, ≥70 Veteran, ≥55 Full-fledged, ≥40 Apprentice, <40 Beginner.
