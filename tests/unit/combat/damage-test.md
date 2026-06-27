# Damage Calculation Tests

- **System**: Combat / Damage
- **Story Type**: Logic
- **Gate Level**: BLOCKING
- **Source File**: `src/index.html`
- **Framework**: Manual verification specification (Canvas game, no test runner)

---

## Test Case: DMG-001 -- Auto-attack deals exactly 7 damage per hit

- **Source**: `src/index.html` line 347 (`damage: 7`)
- **Precondition**: At least one enemy is alive and within 350px auto-attack range of the player.
- **Steps**:
  1. Wait for an auto-attack projectile to be fired (occurs every 0.35s when an enemy is in range).
  2. Observe the projectile hit the enemy.
  3. Record the enemy's HP before and after the hit.
- **Expected Result**: Enemy HP decreases by exactly 7.
- **Pass Criteria**: `hp_before - hp_after == 7` after exactly one projectile hit (account for no other damage sources active).

---

## Test Case: DMG-002 -- Q spell deals exactly 45 damage per enemy in range

- **Source**: `src/index.html` line 405 (`enemy.hp -= 45`)
- **Precondition**: At least one enemy is within 200px range of the player. Q cooldown is ready (cooldowns.q <= 0).
- **Steps**:
  1. Press Q.
  2. Observe all enemies within 200px of the player.
  3. Record each affected enemy's HP before and after.
- **Expected Result**: Every enemy within 200px loses exactly 45 HP. Enemies beyond 200px are unaffected.
- **Pass Criteria**: For every enemy within 200px range: `hp_before - hp_after == 45`. For every enemy beyond 200px: `hp_before - hp_after == 0`.

---

## Test Case: DMG-003 -- R fire wall deals 22 damage per second

- **Source**: `src/index.html` line 511 (`damagePerSec: 22`)
- **Precondition**: At least one enemy is alive. R cooldown is ready.
- **Steps**:
  1. Press R to place a fire wall at the nearest enemy's position.
  2. Ensure the enemy remains within the fire wall AABB (45px half-width, 14px half-height).
  3. Record HP every 0.25 seconds over a 1-second interval (discrete sampling).
- **Expected Result**: Enemy HP decreases at a rate of 22 damage per second (approximately 5.5 HP per 0.25 seconds).
- **Pass Criteria**: Over a full 1.0 seconds of continuous fire wall overlap, the total damage dealt is `22 +/- 1` (floating-point tolerance due to `damagePerSec * dt` accumulation from `src/index.html` line 683). Enemy outside the AABB takes zero fire damage.

---

## Test Case: DMG-004 -- Enemy contact damage: apprentice deals 8/s

- **Source**: `src/index.html` line 314 (`damage: 8`), line 762 (`player.hp -= enemy.damage * dt`)
- **Precondition**: One apprentice-type enemy is overlapping the player (center distance < enemy.radius + player.radius).
- **Steps**:
  1. Move the player into contact with an apprentice enemy.
  2. Maintain continuous contact (overlap) for exactly 1.0 seconds.
  3. Record player HP at start and end.
- **Expected Result**: Player HP decreases at a rate of 8 per second.
- **Pass Criteria**: Player HP decreases by `8 +/- 1` over 1.0 seconds of continuous contact. (Repeated pushback from `src/index.html` lines 766-769 may interrupt contact; account for that by measuring total overlap time precisely.)

---

## Test Case: DMG-005 -- Enemy contact damage: elite deals 15/s

- **Source**: `src/index.html` line 315 (`damage: 15`)
- **Precondition**: One elite-type enemy is overlapping the player.
- **Steps**:
  1. Move the player into contact with an elite enemy.
  2. Maintain continuous contact for exactly 1.0 seconds.
  3. Record player HP at start and end.
- **Expected Result**: Player HP decreases at a rate of 15 per second.
- **Pass Criteria**: Player HP decreases by `15 +/- 1` over 1.0 seconds of continuous contact.

---

## Test Case: DMG-006 -- Enemy contact damage: acromantula deals 25/s

- **Source**: `src/index.html` line 316 (`damage: 25`)
- **Precondition**: The acromantula boss is overlapping the player.
- **Steps**:
  1. During Wave 4, let the acromantula approach and overlap the player.
  2. Maintain continuous contact for exactly 1.0 seconds.
  3. Record player HP at start and end.
- **Expected Result**: Player HP decreases at a rate of 25 per second.
- **Pass Criteria**: Player HP decreases by `25 +/- 1` over 1.0 seconds of continuous contact.

---

## Test Case: DMG-007 -- Enemy projectile damage equals enemy.damage * 0.5

- **Source**: `src/index.html` line 752 (`damage: enemy.damage * 0.5`)
- **Precondition**: An enemy (any type) is within 350px of the player and has its shoot timer expired.
- **Steps**:
  1. Allow an enemy to fire a projectile at the player (enemies shoot when `distToPlayer < 350` and `shootTimer <= 0`, per line 744).
  2. Let the enemy projectile hit the player.
  3. Record player HP before and after.
- **Expected Result**:
  - Apprentice projectile: player takes `8 * 0.5 = 4` damage
  - Elite projectile: player takes `15 * 0.5 = 7.5` damage
  - Acromantula projectile: player takes `25 * 0.5 = 12.5` damage
- **Pass Criteria**:
  - Apprentice bolt: `hp_before - hp_after == 4`
  - Elite bolt: `hp_before - hp_after == 7.5`
  - Acromantula bolt: `hp_before - hp_after == 12.5`

---

## Test Case: DMG-008 -- Multiple damage sources stack correctly

- **Source**: `src/index.html` lines 683 (fire wall), 762 (contact), 833 (enemy projectile)
- **Precondition**: Player and at least one enemy are in a state where multiple damage sources apply simultaneously.
- **Steps**:
  1. Have an enemy overlap the player (contact damage).
  2. While overlapping, allow the enemy to fire a projectile that hits the player.
  3. Record player HP before and after both sources apply.
- **Expected Result**: Player takes cumulative damage from both sources: `(enemy.damage * dt) + (enemy.damage * 0.5)` for the frame where the projectile hits.
- **Pass Criteria**: Total damage in the frame of projectile impact accounts for the continuous contact damage tick plus the projectile impact.

---

## Test Case: DMG-009 -- Dead enemy takes no further damage (spliced on death)

- **Source**: `src/index.html` lines 416-418 (Q cleanup), lines 696-698 (fire wall cleanup), lines 801-802 (projectile hit cleanup)
- **Precondition**: An enemy has HP <= 0 from any damage source.
- **Steps**:
  1. Deal lethal damage to an enemy.
  2. Observe whether subsequent damage ticks (from fire wall, other projectiles in flight) affect the dead enemy.
- **Expected Result**: The dead enemy is removed from the `enemies` array immediately (via `splice`) and does not receive further damage processing.
- **Pass Criteria**: After lethal damage is applied, the enemy object is no longer present in the `enemies` array on the next frame. No further damage is recorded against that enemy.

---

## Edge Cases Summary

| Case | Description | Expected |
|------|-------------|----------|
| Zero enemies in range for Q | Q still casts, no damage dealt | 0 damage recorded, cooldown triggers |
| Fire wall at map edge | Fire wall placed, visual clipped | Damage still applies to enemies in AABB |
| Enemy projectile out of bounds | Projectile reaches map edge | Projectile removed (`life <= 0 || out of bounds`) |
| HP display floors at 0 | Player HP drops below 0 | `Math.max(0, ...)` at line 863 applies |
