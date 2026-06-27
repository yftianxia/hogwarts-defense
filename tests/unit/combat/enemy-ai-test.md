# Enemy AI Behavior Tests

- **System**: Combat / Enemy AI
- **Story Type**: Logic
- **Gate Level**: BLOCKING
- **Source File**: `src/index.html`
- **Framework**: Manual verification specification (Canvas game, no test runner)

---

## Test Case: AI-001 -- Enemies walk toward gate when not reached

- **Source**: `src/index.html` lines 725-733 (movement logic -- gate target when `!reachedGate && enemy.y <= GATE_Y`)
- **Precondition**: An enemy has just spawned (at `SPAWN_Y = 60` near the top of the map). The enemy has `reachedGate == undefined` (falsy).
- **Steps**:
  1. Observe a newly spawned enemy's movement.
  2. Track its target destination.
  3. Observe its movement until `enemy.y > GATE_Y`.
- **Expected Result**:
  - The enemy moves toward the gate at `(GATE_X + some jitter, GATE_Y)` where `GATE_X = W * 0.45 ~= 432`, `GATE_Y = H - 50 = 590`.
  - Jitter: target X is offset by `GATE_X * 0.1 * (Math.random() - 0.5)` per frame (line 731).
  - Enemy speed varies by type: apprentice = 75 px/s, elite = 50 px/s, acromantula = 35 px/s (lines 314-316).
- **Pass Criteria**: An enemy with `reachedGate != true` and `enemy.y <= GATE_Y` moves toward gate coordinates each frame. Once `enemy.y > GATE_Y`, the `reachedGate` flag is set to true (line 727).

---

## Test Case: AI-002 -- Enemies chase player after reaching gate

- **Source**: `src/index.html` lines 726-730 (chase logic when `reachedGate == true` or `enemy.y > GATE_Y`)
- **Precondition**: An enemy has `reachedGate == true` or its y-position exceeds `GATE_Y` (590).
- **Steps**:
  1. Allow an enemy to reach the gate (pass `GATE_Y`).
  2. Move the player to various positions on the map.
  3. Observe the enemy's movement.
- **Expected Result**: The enemy moves directly toward the player's current position `(player.x, player.y)` each frame, rather than toward the gate.
- **Pass Criteria**: The target of enemy movement is `(player.x, player.y)` when `reachedGate || enemy.y > GATE_Y`. The enemy does not return to gate-seeking behavior once `reachedGate` is true.

---

## Test Case: AI-003 -- Stunned enemies do not move

- **Source**: `src/index.html` line 722 (`if (enemy.stunned > 0 || enemy.frozen > 0) continue`)
- **Precondition**: An enemy is within the E spell cone (260px range, ~51 degree cone angle) and will be stunned.
- **Steps**:
  1. Press E to stun an enemy (`enemy.stunned = 1.5` at line 466).
  2. Observe the enemy's position over 1.5 seconds.
  3. Observe after the stun expires.
- **Expected Result**:
  - During the 1.5-second stun: enemy position does not change (no movement vector applied).
  - After stun expires: enemy resumes its previous behavior (gate-seeking or player-chasing).
- **Pass Criteria**: The `continue` statement at line 722 skips all movement, shooting, and collision logic for the frame. Enemy's `x` and `y` coordinates remain unchanged for the full 1.5-second stun duration (minus any external forces like Q knockback or collision pushback, which are applied in Q's loop rather than the enemy update loop).

---

## Test Case: AI-004 -- Frozen enemies do not move

- **Source**: `src/index.html` line 722 (`if (enemy.stunned > 0 || enemy.frozen > 0) continue`), line 539 (`nearest.frozen = 3`)
- **Precondition**: An enemy is within 300px range of the player (F spell's acquisition range).
- **Steps**:
  1. Press F to freeze the nearest enemy (3-second freeze duration).
  2. Observe the enemy's position over 3 seconds.
  3. Observe after the freeze expires.
- **Expected Result**:
  - During the 3-second freeze: enemy position does not change.
  - After freeze expires: enemy resumes normal behavior.
- **Pass Criteria**: The `continue` statement at line 722 prevents all movement for the frozen enemy. Coordinates `(x, y)` remain constant for the full 3-second freeze duration.

---

## Test Case: AI-005 -- Enemies shoot at player when within 350px

- **Source**: `src/index.html` line 744 (`if (enemy.shootTimer <= 0 && distToPlayer < 350)`)
- **Precondition**: An enemy is alive, not stunned, not frozen, and has `distToPlayer < 350`.
- **Steps**:
  1. Move the player to a distance of exactly 340px from an enemy (inside shoot range) and hold still.
  2. Wait for the enemy's `shootTimer` to expire (initial timer: `1.5 + random * 2`, lines 720, 755).
  3. Observe whether an enemy projectile spawns.
  4. Move the player to exactly 360px (outside shoot range) and observe.
- **Expected Result**:
  - At 340px (inside range): an enemy projectile spawns when `shootTimer <= 0`.
  - At 360px (outside range): no enemy projectile spawns.
  - Projectile travels at 200 px/s toward the player (line 750-751 `vx/vy: (sdx/slen) * 200`).
- **Pass Criteria**: Enemy projectiles only appear in the `enemyProjectiles` array when `distToPlayer < 350`. The `shootTimer` resets to `1.5 + Math.random() * 2` after each shot.

---

## Test Case: AI-006 -- Stunned enemies do not shoot

- **Source**: `src/index.html` line 722 (`continue` skips all logic including shooting for stunned enemies)
- **Precondition**: An enemy is within 350px of the player and is stunned.
- **Steps**:
  1. Stun an enemy with E.
  2. Move the player within 350px of the stunned enemy.
  3. Wait for at least 1.5 seconds (the stun duration).
  4. Check whether any enemy projectiles were fired from the stunned enemy.
- **Expected Result**: No projectiles are fired from the stunned enemy during the 1.5-second stun. After stun expires, the enemy may begin shooting again.
- **Pass Criteria**: Zero enemy projectiles originate from the stunned enemy while `enemy.stunned > 0`.

---

## Test Case: AI-007 -- Frozen enemies do not shoot

- **Source**: `src/index.html` line 722 (`continue` skips all logic including shooting for frozen enemies)
- **Precondition**: An enemy is within 350px of the player and is frozen (F spell).
- **Steps**:
  1. Freeze an enemy with F.
  2. Move the player within 350px of the frozen enemy.
  3. Wait for at least 3 seconds (the freeze duration).
  4. Check whether any enemy projectiles were fired from the frozen enemy.
- **Expected Result**: No projectiles are fired from the frozen enemy during the 3-second freeze.
- **Pass Criteria**: Zero enemy projectiles originate from the frozen enemy while `enemy.frozen > 0`.

---

## Test Case: AI-008 -- Shoot timer initializes with random offset

- **Source**: `src/index.html` line 720 (`if (enemy.shootTimer === undefined) enemy.shootTimer = 1.5 + Math.random() * 2`)
- **Precondition**: A new enemy has just spawned.
- **Steps**:
  1. Spawn at least 5 enemies.
  2. For each enemy, measure the time from spawn to its first shot (while player is within 350px range).
- **Expected Result**: Each enemy's first shot fires between 1.5 and 3.5 seconds after spawn (initial timer range: `1.5 + [0, 2]`).
- **Pass Criteria**: First shot time for each enemy falls in the range `[1.5, 3.5]` seconds from spawn. Subsequent shot intervals follow the same range after each reset (line 755).

---

## Test Case: AI-009 -- Enemy speed varies by type during gate approach

- **Source**: `src/index.html` lines 314-316 (speed: apprentice=75, elite=50, acromantula=35)
- **Precondition**: One of each enemy type is spawned and moving toward the gate.
- **Steps**:
  1. Spawn an apprentice, an elite, and an acromantula at the same Y position.
  2. Measure the time each takes to travel a known distance (e.g., from Y=100 to Y=500).
- **Expected Result**:
  - Apprentice covers 400px in approximately `400 / 75 = 5.33` seconds.
  - Elite covers 400px in approximately `400 / 50 = 8.0` seconds.
  - Acromantula covers 400px in approximately `400 / 35 = 11.43` seconds.
- **Pass Criteria**: Speed values match: apprentice = 75, elite = 50, acromantula = 35 (pixels per second). The movement formula at line 738 `enemy.x += (edx / elen) * enemy.speed * dt` confirms speed is applied directly.

---

## Test Case: AI-010 -- Enemy does not shoot when stunned or frozen (both conditions)

- **Source**: `src/index.html` line 722 (combined check)
- **Precondition**: An enemy is both within shoot range (<350px) and has `stunned > 0` or `frozen > 0`.
- **Steps**:
  1. Apply E (stun) to one enemy and F (freeze) to another.
  2. Stand within 350px of both.
  3. Observe for the full duration of each CC.
- **Expected Result**: Neither the stunned enemy nor the frozen enemy fires a projectile.
- **Pass Criteria**: The `continue` at line 722 prevents the enemy from reaching the shoot logic (lines 742-758) on any frame where `stunned > 0 || frozen > 0`.

---

## Edge Cases Summary

| Case | Description | Expected |
|------|-------------|----------|
| Stun and freeze simultaneously | Enemy has both `stunned > 0` and `frozen > 0` | Still skips movement (either condition triggers `continue`) |
| Gate target jitter | Random X offset each frame | Enemy path may waver slightly but generally approaches gate |
| Enemy at exact gate Y | `enemy.y == GATE_Y` does NOT trigger chase | Chase only when `enemy.y > GATE_Y` (strict greater-than, line 726) |
| Shoot range exactly 350 | `distToPlayer == 350` | Does NOT shoot (`distToPlayer < 350` is strict less-than, line 744) |
| Enemy spawns past gate | `enemy.y > GATE_Y` from spawn (should not happen normally) | Immediately enters chase mode (line 726 condition met) |
