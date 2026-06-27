# STORY-003: Write Automated Tests for Combat Systems

- **Status**: Complete
- **Priority**: Must Have
- **Estimate**: 0.5 days
- **Owner**: technical-director
- **Sprint**: 1

## Description

Create automated tests for the core combat system formulas. Since the game
is a single-file HTML/JS application without a test framework, scaffold a
minimal test harness (plain JS assertions in a separate HTML test page) and
write tests covering damage formulas, spell cooldowns, wave spawning logic,
and HP/win/loss state transitions.

## Acceptance Criteria

- [ ] Test harness exists at `tests/unit/combat/test-runner.html`
- [ ] Test: Spell Q (Expelliarmus) deals exactly 45 damage to enemies within 200px range
- [ ] Test: Spell E (Lumos) stuns enemies within 51-degree cone for exactly 1.5 seconds
- [ ] Test: Spell R (Incendio) deals 22 damage/second to enemies in fire wall bounds
- [ ] Test: Spell F (Petrificus Totalus) freezes nearest enemy within 300px for exactly 3 seconds
- [ ] Test: Auto-attack deals exactly 7 damage per bolt at 0.35s fire rate
- [ ] Test: Spell cooldowns match configured values (Q:3s, E:8s, R:12s, F:10s)
- [ ] Test: Wave 1 spawns exactly 12 apprentices, Wave 2 spawns 10 apprentices + 4 elites
- [ ] Test: Player HP reaching 0 triggers gameOver = true
- [ ] Test: All enemies dead after Wave 4 triggers gameWin = true
- [ ] Test: Enemy collision damage formula correct (apprentice: 8/sec, elite: 15/sec, boss: 25/sec)
- [ ] All tests pass with zero failures
- [ ] Tests are deterministic (no Math.random() in assertions)

## Dependencies

- `src/index.html` (extract combat formulas for test assertions)

## Technical Notes

- No framework required -- plain JS with console.assert or visible pass/fail UI
- Extract combat logic into testable pure functions where possible
- Test file should be runnable by opening in a browser (no Node.js dependency)
- Per testing standards: use factory functions for test fixtures, not inline magic numbers (except boundary value tests)
