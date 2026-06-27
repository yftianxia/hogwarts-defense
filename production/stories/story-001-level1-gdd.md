# STORY-001: Complete Level 1 GDD

- **Status**: Complete
- **Priority**: Must Have
- **Estimate**: 0.5 days
- **Owner**: game-designer
- **Sprint**: 1

## Description

Expand the existing `design/gdd/game-concept.md` into a full Game Design
Document for Level 1 (Castle Gate Defense) with all 8 required sections.

## Acceptance Criteria

- [ ] Section 1: Overview -- one-paragraph summary of Level 1
- [ ] Section 2: Player Fantasy -- intended feeling (defending Hogwarts as Harry)
- [ ] Section 3: Detailed Rules -- wave composition, enemy AI behavior, win/loss conditions
- [ ] Section 4: Formulas -- all damage, HP, cooldown math with variables defined
- [ ] Section 5: Edge Cases -- HP rounding, collision edge cases, what happens when all enemies die mid-wave
- [ ] Section 6: Dependencies -- engine (Canvas 2D), audio (Web Audio API), input (keyboard)
- [ ] Section 7: Tuning Knobs -- every configurable value listed (player speed, spell damage, enemy HP, spawn intervals, etc.)
- [ ] Section 8: Acceptance Criteria -- testable conditions for Level 1 being "done"
- [ ] All values link to their source formula or rationale
- [ ] Document written to `design/gdd/level1-castle-gate.md`

## Dependencies

- `design/gdd/game-concept.md` (source of truth for current values)

## Technical Notes

- The source code (`src/index.html`) is the authoritative reference for current values
- Balance values must be extracted from the code and documented with rationale
