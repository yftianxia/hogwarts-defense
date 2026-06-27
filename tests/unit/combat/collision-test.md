# Collision and Edge Case Tests

- **System**: Combat / Collision
- **Story Type**: Logic
- **Gate Level**: BLOCKING
- **Source File**: `src/index.html`
- **Framework**: Manual verification specification (Canvas game, no test runner)

---

## Test Case: COL-001 -- Player takes contact damage when overlapping enemy

- **Source**: `src/index.html` lines 760-773 (collision with Harry)
- **Precondition**: An enemy and the player are positioned so that `distToPlayer < enemy.radius + player.radius` (player radius = 18, apprentice radius = 12, so distance < 30).
- **Steps**:
  1. Move the player into direct overlap with an apprentice enemy.
  2. Measure HP over time while maintaining overlap.
  3. Record the damage per frame.
- **Expected Result**: Player HP decreases by `enemy.damage * dt` each frame while overlap persists. For apprentice: `8 * dt` per frame.
- **Pass Criteria**: Player HP decrement per frame equals `enemy.damage * dt` (line 762). `player.damageFlash` is set to `0.1` (line 763), and the red flash overlay is visible.

---

## Test Case: COL-002 -- Enemy is pushed back on collision (no division by zero)

- **Source**: `src/index.html` lines 765-769 (safeDist guard and pushback)
- **Precondition**: Enemy is overlapping the player (`distToPlayer < enemy.radius + player.radius`).
- **Steps**:
  1. Move the player directly on top of an enemy (center on center, `distToPlayer` approaching 0).
  2. Observe the enemy's position over subsequent frames.
- **Expected Result**:
  - The enemy is pushed away from the player by 3 pixels along the normalized direction vector (line 766-767: `(enemy.x - player.x) / safeDist * 3`).
  - When distance is exactly 0: `safeDist` is clamped to `0.01` (line 765: `Math.max(distToPlayer, 0.01)`), preventing division by zero.
  - No crash, no NaN coordinates.
- **Pass Criteria**: `safeDist >= 0.01` always. No division by zero. Enemy coordinates remain finite numbers. The pushback is always `pushX = (dx / safeDist) * 3`, `pushY = (dy / safeDist) * 3`.

---

## Test Case: COL-003 -- Enemies pushed by Q are clamped to map bounds

- **Source**: `src/index.html` lines 401-403 (`Math.max(15, Math.min(W - 15, enemy.x))`)
- **Precondition**: An enemy is near a map edge (within 80px of the boundary).
- **Steps**:
  1. Lure an enemy to the far left side of the map (x near 0).
  2. Position the player to the enemy's right.
  3. Cast Q to knock the enemy left (further toward the edge).
  4. Observe the enemy's final position.
- **Expected Result**: The enemy's x-coordinate is clamped to 15 (minimum) and `W - 15` (maximum = 945). Same for y-coordinate (15 to `H - 15` = 625).
- **Pass Criteria**: After Q knockback: `15 <= enemy.x <= W - 15` and `15 <= enemy.y <= H - 15`.

---

## Test Case: COL-004 -- Player movement clamped to map bounds

- **Source**: `src/index.html` lines 605-607 (`Math.max(m, Math.min(W - m, player.x))` where `m = 20`)
- **Precondition**: Game is running.
- **Steps**:
  1. Move the player to the far left edge of the map (hold A/ArrowLeft).
  2. Move the player to the far right edge (hold D/ArrowRight).
  3. Move the player to the top edge (hold W/ArrowUp).
  4. Move the player to the bottom edge (hold S/ArrowDown).
- **Expected Result**: Player position is always clamped within `[20, W - 20]` for X and `[20, H - 20]` for Y (`W = 960, H = 640`, so range is `[20, 940]` X and `[20, 620]` Y).
- **Pass Criteria**: `20 <= player.x <= 940` and `20 <= player.y <= 620` at all times.

---

## Test Case: COL-005 -- Dead enemies (hp <= 0) are removed from the enemies array

- **Source**: `src/index.html` lines 416-418 (Q spell cleanup), lines 696-698 (fire wall cleanup), lines 801-802 (projectile hit cleanup)
- **Precondition**: An enemy has just received lethal damage (hp drops to 0 or below).
- **Steps**:
  1. Deal lethal damage via auto-attack projectile.
  2. On the next frame after the hit, inspect the `enemies` array.
  3. Repeat for each damage source (Q, R fire wall, basic auto-attack).
- **Expected Result**: The dead enemy is spliced from the array -- its object is no longer present.
- **Pass Criteria**: After lethal damage, `enemies.length` decreases by 1 (or more if multiple enemies died). The dead enemy does not receive further damage ticks, does not render, and does not participate in collision.

---

## Test Case: COL-006 -- Projectiles out of bounds are removed

- **Source**: `src/index.html` lines 782-783 (projectile removal: `p.life <= 0 || out of bounds`)
- **Precondition**: A projectile is in flight.
- **Steps**:
  1. Fire projectiles toward the map edges (auto-attack with enemy near edge).
  2. Observe whether projectiles that leave the canvas (x < 0, x > W, y < 0, y > H) are removed.
- **Expected Result**: Projectiles that exceed canvas bounds are removed from the `projectiles` array even if their `life` timer has not expired.
- **Pass Criteria**: Condition at line 782: `p.x < 0 || p.x > W || p.y < 0 || p.y > H` triggers removal.

---

## Test Case: COL-007 -- Enemy projectiles out of bounds are removed

- **Source**: `src/index.html` lines 825-826 (`ep.life <= 0 || ep.x < 0 || ep.x > W || ep.y < 0 || ep.y > H`)
- **Precondition**: An enemy projectile is in flight.
- **Steps**:
  1. Position the player such that an enemy fires a projectile that can miss and fly off the map.
  2. Observe whether the enemy projectile is removed when it leaves the canvas.
- **Expected Result**: Enemy projectiles exceeding canvas bounds are removed from the `enemyProjectiles` array.
- **Pass Criteria**: Same boundary check as player projectiles: `x < 0 || x > W || y < 0 || y > H` triggers removal.

---

## Test Case: COL-008 -- Player collision with dead enemy does not occur

- **Source**: `src/index.html` lines 801-802 (enemy spliced from array on projectile kill)
- **Precondition**: An enemy is about to die from a projectile hit.
- **Steps**:
  1. Fire a projectile that kills an enemy that is adjacent to the player.
  2. On the frame the enemy dies (projectile hit), observe whether collision damage continues.
- **Expected Result**: The enemy is removed from the array in the same frame it dies (projectile update loop at line 802), so the enemy update loop (lines 715-773) does not iterate over it on subsequent frames.
- **Pass Criteria**: No contact damage from a dead enemy. The splice happens inside the projectile loop which runs before the next frame's enemy update.

---

## Test Case: COL-009 -- Q spell kill cleanup iterates in reverse

- **Source**: `src/index.html` lines 416-418 (`for (let i = enemies.length - 1; i >= 0; i--)`)
- **Precondition**: Multiple enemies are killed by a single Q cast (enemies with HP <= 45 within 200px).
- **Steps**:
  1. Spawn or arrange multiple low-HP enemies within 200px of the player.
  2. Cast Q to kill 2 or more enemies simultaneously.
  3. Verify all dead enemies are removed.
- **Expected Result**: All enemies with `hp <= 0` are removed from the array. Reverse iteration prevents index skipping during splice.
- **Pass Criteria**: After Q cast, `enemies` array contains exactly the enemies that survived. No dead enemies remain. No index-out-of-bounds errors.

---

## Test Case: COL-010 -- Fire wall AABB collision detection

- **Source**: `src/index.html` lines 679-682 (`Math.abs(enemy.x - fw.x) < fw.halfWidth + enemy.radius && Math.abs(enemy.y - fw.y) < fw.halfHeight + enemy.radius`)
- **Precondition**: A fire wall is active (`life > 0`) with halfWidth=45, halfHeight=14.
- **Steps**:
  1. Place a fire wall.
  2. Move an enemy to positions at the exact boundary of the fire wall AABB.
  3. Test positions: exactly at boundary + 1px, exactly at boundary, exactly at boundary - 1px.
- **Expected Result**:
  - Enemy whose center is at `(fw.x + 45 + enemy.radius + 1, fw.y)` does NOT take damage (outside, strict inequality).
  - Enemy whose center is at `(fw.x + 45 + enemy.radius - 1, fw.y)` DOES take damage (inside).
- **Pass Criteria**: Damage only applies when both X and Y distance checks pass the AABB + radius overlap (strict less-than, `<` at line 680 and 681).

---

## Edge Cases Summary

| Case | Description | Expected |
|------|-------------|----------|
| Exact center overlap (dist=0) | Player and enemy at exact same position | `safeDist = 0.01`, no crash, enemy pushed 3px in arbitrary direction |
| Multiple simultaneous deaths | Several enemies die from Q/R in one frame | All removed, no index errors due to reverse iteration |
| Projectile hits two enemies | Single projectile can only hit one enemy | `break` at line 804 stops checking after first hit |
| Fire wall at map corner | Fire wall placed at position (0, 0) or (W, H) | Damage still applies to enemies overlapping the AABB at corners |
| Player at map edge colliding | Contact damage still applies at map boundary | Enemy push may be limited by map clamping (enemies clamped during Q push, not during contact push) |
