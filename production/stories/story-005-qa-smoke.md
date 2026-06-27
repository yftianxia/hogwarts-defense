# STORY-005: QA Smoke Check on Level 1

- **Status**: Complete
- **Priority**: Must Have
- **Estimate**: 0.25 days
- **Owner**: qa-lead
- **Sprint**: 1

## Description

Perform a manual smoke check of the complete Level 1 experience. Verify
all core systems function correctly from game start through victory screen.
Document any bugs found, with severity classification.

## Acceptance Criteria

- [ ] Game loads without errors in browser console
- [ ] Dialogue system: all 10 lines display, advance works (Enter/Space/Click)
- [ ] Player movement: WASD and arrow keys work, player stays in bounds
- [ ] Auto-attack: Harry fires at nearest enemy within 350px range
- [ ] Spell Q: shockwave expands, enemies knocked back, 45 damage dealt, screen shakes
- [ ] Spell E: light cone renders, enemies in cone stunned for 1.5s, stars display
- [ ] Spell R: fire wall appears at nearest enemy, DoT damage applied, fire particles
- [ ] Spell F: nearest enemy frozen 3s, ice overlay renders
- [ ] Cooldown HUD: all 4 skill icons show correct fill state and ready/recharging class
- [ ] Wave 1: 12 apprentices spawn, kill all, wave completes
- [ ] Wave 2: 10 apprentices + 4 elites spawn, kill all, wave completes
- [ ] Wave 3: 8 apprentices + 6 elites spawn, kill all, wave completes
- [ ] Wave 4: Boss spawns, 500 HP, boss defeated
- [ ] Victory screen: "恭喜！城堡守住了！" displays
- [ ] Death: player HP reaches 0, game over screen displays
- [ ] Inter-wave announcements display correctly
- [ ] Damage stats panel updates correctly for all damage sources
- [ ] Audio: all 6 SFX functions produce sound (Q, E, R, F, hit, explosion, chime)
- [ ] No S1 (crash/blocker) bugs found
- [ ] Smoke check report written to `production/qa/smoke-2026-06-25.md`

## Dependencies

- STORY-003 (tests should pass before smoke check)
- STORY-004 (balance should be finalized before smoke check)

## Technical Notes

- Open `src/index.html` in browser to test
- Check browser console for errors throughout
- Test both keyboard and mouse inputs
- Try edge cases: spam all skills, stand still and let enemies reach gate, die intentionally
- Per testing standards: this is a Config/Data type story, smoke check is ADVISORY gate level
