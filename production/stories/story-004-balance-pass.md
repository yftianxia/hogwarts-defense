# STORY-004: Balance Pass on Level 1

- **Status**: Complete
- **Priority**: Must Have
- **Estimate**: 0.25 days
- **Owner**: game-designer
- **Sprint**: 1

## Description

Review all gameplay values in Level 1 against the documented design.
Verify that player HP, spell damage, enemy HP, spawn rates, and cooldowns
create the intended difficulty curve. Document findings and any adjustments
with rationale.

## Acceptance Criteria

- [ ] All configurable values documented in the GDD (from STORY-001)
- [ ] Each value has a rationale (e.g., "Q deals 45 so it one-shots apprentices but not elites")
- [ ] Difficulty curve assessed: Wave 1 (easy) through Wave 4 (boss, hard)
- [ ] Player kill time vs death time calculated and documented
- [ ] If values change: source code updated AND GDD updated
- [ ] Balance notes written to `design/gdd/level1-castle-gate.md` Tuning Knobs section

## Current Values (from source)

| Parameter | Value |
|-----------|-------|
| Player HP | 100 |
| Player speed | 200 px/s |
| Auto-attack damage | 7 |
| Auto-attack rate | 0.35s |
| Q damage | 45 (AoE, 200px range) |
| E stun duration | 1.5s (51-degree cone, 260px range) |
| R damage | 22/s (DoT, 3s duration) |
| F freeze duration | 3s (single target, 300px range) |
| Apprentice HP | 60 |
| Elite HP | 150 |
| Boss HP | 500 |
| Wave 1 | 12 apprentices |
| Wave 2 | 10 apprentices + 4 elites |
| Wave 3 | 8 apprentices + 6 elites |
| Wave 4 | 1 boss |

## Dependencies

- STORY-001 (GDD must exist to document balance in)
- `src/index.html` (source of current values)

## Technical Notes

- This is a documentation + review task, not a rebalance task
- Only change values if they are clearly wrong (e.g., impossible to win, trivial to win)
- If no changes needed, document that current values are balanced and why
