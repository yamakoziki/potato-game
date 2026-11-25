# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

Open `index.html` directly in any modern web browser. No build process, server, or dependencies required.

## Architecture

The entire application lives in a single file: `index.html` (~2,900 lines). Sections are marked with `// =====` comments.

**Layout of index.html:**
- Lines 1–799: HTML structure and CSS styling (mobile-first, max-width 480px)
- Lines 800–838: Language helpers (`t()`, `num()`, `setLang()`, `renderLanguage()`)
- Lines 839–852: `state` object — the single source of truth for all game state
- Lines 853–931: `KNOWLEDGE_CARDS` object (keyed by string ID, e.g. `variety_jyoti`)
- Lines 932–1174: `PHASES` array — 11 farming phases, each with choices and score deltas
- Lines 1175–1230: `TRAP_EVENTS` — season-specific random challenge events (keyed by season)
- Lines 1231–1500: Sales phase logic (dynamically generated from current scores) and `SCORE_LABELS`
- Lines 1501–1802: `render()` dispatcher and `render*()` functions — one per screen
- Lines 1803–1916: Action handlers
- Lines 1917–1949: `showAlbum()`
- Lines 1950–2891: `TRANSLATIONS` — `[Japanese, English, Nepali]` triples
- End: `render()` initialization call

**Languages:** Japanese (default), Nepali, English; `state.lang`, saved in localStorage. Game data stays in Japanese, and the Japanese string is the translation key: render functions wrap text in `t()`. When adding or editing Japanese text, update the matching `TRANSLATIONS` entry — unmatched strings fall back to Japanese. Wrap displayed numbers in `num()` (Devanagari digits in Nepali).

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
