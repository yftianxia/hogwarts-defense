# Milestone M1: Core Combat

- **Status**: COMPLETED
- **Target**: Single-level combat prototype with all core systems working
- **Completed**: 2026-06-24

## Goal

Deliver a fully playable single-level prototype proving that the core combat
loop -- player movement, auto-attack, four spells, three enemy types, wave
system, dialogue, audio, and VFX -- all function together in a single-file
HTML5 Canvas game.

## Deliverables

| # | Deliverable | Status |
|---|-------------|--------|
| 1 | Player movement (WASD + arrow keys, facing, robe animation) | DONE |
| 2 | Auto-attack system (nearest-enemy targeting, spell bolt projectiles) | DONE |
| 3 | Q -- Expelliarmus (knockback + AoE damage + screen shake) | DONE |
| 4 | E -- Lumos (cone stun + visual beam) | DONE |
| 5 | R -- Incendio (fire wall DoT at nearest enemy) | DONE |
| 6 | F -- Petrificus Totalus (single-target freeze) | DONE |
| 7 | Enemy Type 1: Death Eater Apprentice (60 HP, basic AI) | DONE |
| 8 | Enemy Type 2: Elite Death Eater (150 HP, purple aura, tougher) | DONE |
| 9 | Enemy Type 3: Acromantula Boss (500 HP, 8 legs, 8 eyes) | DONE |
| 10 | Wave system (4 waves, spawn intervals, inter-wave pause) | DONE |
| 11 | Dialogue system (intro cutscene, skill tutorial) | DONE |
| 12 | Audio engine (procedural Web Audio, per-spell SFX) | DONE |
| 13 | VFX (particles, screen shake, damage flash, spell visuals) | DONE |

## Acceptance Evidence

- **Playable build**: `src/index.html` -- opens in browser, full Level 1 playable
- **All 4 waves**: Completable from dialogue through boss defeat
- **Win/loss states**: Both game-over and victory screens functional
- **No external dependencies**: Single file, no asset files, no server needed

## Risks Retired

- Canvas 2D rendering performance (proven adequate for this scope)
- Procedural audio viability (Web Audio API works without external files)
- Enemy AI pathfinding (gate-rush then chase-Harry behavior works)
- Spell cooldown + HUD integration (all 4 skills function with visual feedback)
