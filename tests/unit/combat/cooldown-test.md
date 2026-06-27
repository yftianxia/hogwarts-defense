# Spell Cooldown Tests

- **System**: Combat / Spell Cooldowns
- **Story Type**: Logic
- **Gate Level**: BLOCKING
- **Source File**: `src/index.html`
- **Framework**: Manual verification specification (Canvas game, no test runner)

---

## Test Case: CD-001 -- Q spell has 3-second cooldown

- **Source**: `src/index.html` line 207 (`spellMaxCd: { q: 3, e: 8, r: 12, f: 10 }`), line 374 (`if (player.cooldowns.q > 0) return`)
- **Precondition**: Game is running, not in dialogue, not game over/win. Q cooldown is ready (cooldowns.q == 0).
- **Steps**:
  1. Press Q to cast.
  2. Immediately attempt to press Q again.
  3. Wait and attempt Q at 1.0s, 2.0s, 2.9s, and 3.1s intervals after cast.
- **Expected Result**:
  - Q casts successfully on first press.
  - Q does NOT cast at 0s, 1.0s, 2.0s, or 2.9s (cooldown still active).
  - Q casts successfully at 3.1s (cooldown expired).
- **Pass Criteria**: Q is available exactly when `cooldowns.q` reaches 0, which occurs 3.0 seconds after cast (cooldown decrements by `dt` each frame, per `src/index.html` lines 583-586).

---

## Test Case: CD-002 -- E spell has 8-second cooldown

- **Source**: `src/index.html` line 207 (`spellMaxCd.e: 8`)
- **Precondition**: E cooldown is ready. At least one enemy is within the cone range (260px) and angle for E to have a valid target.
- **Steps**:
  1. Press E to cast.
  2. Immediately press E again -- should not cast.
  3. Wait 8.0 seconds and press E again.
- **Expected Result**:
  - First E cast succeeds.
  - Second E (immediate) is blocked.
  - Third E (after 8.0s) succeeds.
- **Pass Criteria**: E cannot be cast while `cooldowns.e > 0`. E available exactly after 8.0 seconds.

---

## Test Case: CD-003 -- R spell has 12-second cooldown

- **Source**: `src/index.html` line 207 (`spellMaxCd.r: 12`)
- **Precondition**: R cooldown is ready. At least one enemy exists for fire wall placement.
- **Steps**:
  1. Press R to cast.
  2. Press R again immediately -- should not cast.
  3. Wait 12.0 seconds and press R again.
- **Expected Result**:
  - First R cast succeeds (fire wall appears).
  - Second R (immediate) is blocked.
  - Third R (after 12.0s) succeeds.
- **Pass Criteria**: R cannot be cast while `cooldowns.r > 0`. R available exactly after 12.0 seconds.

---

## Test Case: CD-004 -- F spell has 10-second cooldown

- **Source**: `src/index.html` line 207 (`spellMaxCd.f: 10`)
- **Precondition**: F cooldown is ready. At least one enemy is within 300px range (source: `src/index.html` line 525, `nearestDist = 300`).
- **Steps**:
  1. Press F to cast (enemy must be within 300px).
  2. Press F again immediately -- should not cast.
  3. Wait 10.0 seconds and press F again.
- **Expected Result**:
  - First F cast succeeds (enemy becomes frozen for 3 seconds).
  - Second F (immediate) is blocked.
  - Third F (after 10.0s) succeeds.
- **Pass Criteria**: F cannot be cast while `cooldowns.f > 0`. F available exactly after 10.0 seconds.

---

## Test Case: CD-005 -- Auto-attack has 0.35-second interval between shots

- **Source**: `src/index.html` line 626 (`attackCooldown = 0.35`)
- **Precondition**: At least one enemy is within 350px auto-attack range. No dialogue active.
- **Steps**:
  1. Observe the game and count projectile spawns over a 2-second window.
  2. Record the timestamp of each projectile fired.
- **Expected Result**: A new auto-attack projectile fires approximately every 0.35 seconds (about 5-6 shots in 2 seconds).
- **Pass Criteria**: The interval between consecutive auto-attack projectiles is `0.35 +/- 0.02` seconds (allow minor frame timing variation). A projectile is created at `src/index.html` line 625 via `shootProjectile(nearest.x, nearest.y)`.

---

## Test Case: CD-006 -- Casting during dialogue is blocked (all spells)

- **Source**: `src/index.html` lines 219-223 (dialogue mode intercepts keydown), lines 373/425/486/519 (`if (gameOver || gameWin) return`)
- **Precondition**: Dialogue is active (`dialogueActive == true`).
- **Steps**:
  1. Start a new game (dialogue begins active).
  2. Press Q, E, R, F keys.
- **Expected Result**: No spell is cast. The keypress advances dialogue instead (`src/index.html` line 220).
- **Pass Criteria**: While `dialogueActive == true`, `castSkillQ/E/R/F` are not reached from `keydown` handler because `keys[key]` assignment and spell cast calls (lines 225-229) are inside the `else` branch after the dialogue check.

---

## Test Case: CD-007 -- Casting during game over is blocked

- **Source**: `src/index.html` line 373 (`if (gameOver || gameWin) return`)
- **Precondition**: Game over state is true (`gameOver == true`).
- **Steps**:
  1. Allow player HP to reach 0 (trigger game over).
  2. Press Q, E, R, F keys.
- **Expected Result**: No spell is cast. The early return at the top of each cast function prevents execution.
- **Pass Criteria**: All four `castSkill*` functions return without effect when `gameOver == true`.

---

## Test Case: CD-008 -- Casting during game win is blocked

- **Source**: `src/index.html` line 373 (`if (gameOver || gameWin) return`)
- **Precondition**: Game win state is true (`gameWin == true`, all 4 waves cleared).
- **Steps**:
  1. Defeat all waves to trigger game win.
  2. Press Q, E, R, F keys.
- **Expected Result**: No spell is cast.
- **Pass Criteria**: All four `castSkill*` functions return without effect when `gameWin == true`.

---

## Test Case: CD-009 -- Cooldown decrements during wave pause

- **Source**: `src/index.html` lines 583-586 (cooldown decrement runs every frame regardless of wave state)
- **Precondition**: A spell has been cast (cooldown > 0) and the current wave has just been cleared (entering wave pause).
- **Steps**:
  1. Cast Q right before the last enemy of a wave dies.
  2. Observe the 3-second wave pause begins.
  3. Check Q cooldown during the pause.
- **Expected Result**: Q cooldown continues to decrement during the wave pause. By the time the next wave starts (3 seconds later), Q should have at most (3 - (time since cast)) seconds remaining.
- **Pass Criteria**: Cooldown values decrement during `wavePause > 0` because the cooldown update loop (lines 583-586) is inside `update(dt)` which runs every frame regardless of wave pause state.

---

## Test Case: CD-010 -- HUD cooldown display reflects actual cooldown state

- **Source**: `src/index.html` lines 868-879 (HUD update)
- **Precondition**: A spell has been cast.
- **Steps**:
  1. Cast Q and observe the Q skill icon in the HUD.
  2. Monitor both the visual cooldown bar (cd-q width) and the CSS class (`skill-ready` vs `skill-recharging`) over the full 3 seconds.
- **Expected Result**:
  - Immediately after cast: icon has class `skill-recharging`, bar is red (`#c44`), width starts at 0% and fills to 100% over 3 seconds.
  - After 3 seconds: icon has class `skill-ready`, bar is green (`#4a4`), width is 100%.
- **Pass Criteria**: The `cd-q` element's width transitions from `(cooldownRemaining / maxCd) * 100` to 100% over the cooldown duration. The class toggles at the exact boundary where `cooldowns.q <= 0`.

---

## Edge Cases Summary

| Case | Description | Expected |
|------|-------------|----------|
| Cast F with no enemies in 300px | F does NOT trigger cooldown | `player.cooldowns.f` stays 0 (return before assignment at line 537) |
| Multiple spells cast simultaneously | Each spell's cooldown is independent | Q/E/R/F can all be on cooldown simultaneously |
| Cooldown at frame boundary | dt causes cooldown to go slightly below 0 | `Math.max(0, ...)` at line 585 clamps to 0 |
| Frameskip / large dt | dt capped at 0.05s | Max single-frame cooldown decrement is 0.05s (line 565) |
