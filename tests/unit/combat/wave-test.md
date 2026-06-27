# Wave System Tests

- **System**: Combat / Wave Spawning
- **Story Type**: Logic
- **Gate Level**: BLOCKING
- **Source File**: `src/index.html`
- **Framework**: Manual verification specification (Canvas game, no test runner)

---

## Test Case: WAV-001 -- Wave 1 spawns exactly 12 apprentices

- **Source**: `src/index.html` line 261 (`{ name: '第 1 波', enemies: 12, elites: 0, boss: null, spawnInterval: 1.0 }`)
- **Precondition**: New game started, dialogue completed. Wave 1 is active.
- **Steps**:
  1. Start Wave 1 (or start a new game -- Wave 1 begins automatically).
  2. Count every enemy that spawns during Wave 1.
  3. Record the type of each spawned enemy.
- **Expected Result**: Exactly 12 enemies spawn, all of type `apprentice`. Zero elites. Zero bosses.
- **Pass Criteria**: Total spawned = 12. All 12 have `type == 'apprentice'`. `enemiesSpawned` reaches 12.

---

## Test Case: WAV-002 -- Wave 2 spawns 10 apprentices + 4 elites

- **Source**: `src/index.html` line 262 (`{ name: '第 2 波', enemies: 10, elites: 4, boss: null, spawnInterval: 0.8 }`)
- **Precondition**: Wave 1 is fully cleared (all Wave 1 enemies killed). Wave 2 is announced.
- **Steps**:
  1. Clear Wave 1 (defeat all 12 apprentices).
  2. Wait for the 3-second inter-wave pause to complete.
  3. Observe Wave 2 spawning and count enemy types.
- **Expected Result**: 14 total enemies spawn: 10 `apprentice` + 4 `elite`.
- **Pass Criteria**: After all Wave 2 enemies spawn: `enemies.filter(e => e.type === 'apprentice').length == 10` and `enemies.filter(e => e.type === 'elite').length == 4`.

---

## Test Case: WAV-003 -- Wave 3 spawns 8 apprentices + 6 elites

- **Source**: `src/index.html` line 263 (`{ name: '第 3 波', enemies: 8, elites: 6, boss: null, spawnInterval: 0.6 }`)
- **Precondition**: Wave 2 is fully cleared. Wave 3 is announced.
- **Steps**:
  1. Clear Wave 2.
  2. Wait for the inter-wave pause.
  3. Observe Wave 3 spawning and count enemy types.
- **Expected Result**: 14 total enemies spawn: 8 `apprentice` + 6 `elite`.
- **Pass Criteria**: After all Wave 3 enemies spawn: `apprentice count == 8`, `elite count == 6`.

---

## Test Case: WAV-004 -- Wave 4 spawns exactly 1 acromantula boss

- **Source**: `src/index.html` line 264 (`{ name: 'BOSS波', enemies: 0, elites: 0, boss: 'acromantula', spawnInterval: 2.0 }`)
- **Precondition**: Wave 3 is fully cleared. Wave 4 is announced.
- **Steps**:
  1. Clear Wave 3.
  2. Wait for the inter-wave pause.
  3. Observe Wave 4 spawning.
  4. Count enemies and identify the boss.
- **Expected Result**: Exactly 1 enemy spawns, of type `acromantula`.
- **Pass Criteria**: `enemiesSpawned == 1`. The single enemy has `type == 'acromantula'`, `hp == 500`, `maxHp == 500`, `radius == 28`, `speed == 35`, `damage == 25`.

---

## Test Case: WAV-005 -- Boss spawns after all normal enemies (if wave has boss + normal)

- **Source**: `src/index.html` line 309 (`if (w.boss && enemiesSpawned === w.enemies + w.elites)`)
- **Precondition**: A wave with both normal enemies and a boss is configured. (Note: current Wave 4 has enemies=0 and elites=0, so the boss spawns first. This test case verifies the mechanism for future hybrid waves.)
- **Steps**:
  1. If a hybrid wave existed (e.g., enemies=5, elites=2, boss=acromantula), observe spawn order.
  2. Verify that normal enemies and elites spawn before the boss.
- **Expected Result**: The boss is the last enemy spawned for the wave. The condition `enemiesSpawned === w.enemies + w.elites` ensures the boss spawns only after all normal and elite enemies are spawned.
- **Pass Criteria**: Boss spawn position in the sequence is `w.enemies + w.elites + 1` (last). For Wave 4 (`enemies=0, elites=0`), the boss spawns as enemy #1.

---

## Test Case: WAV-006 -- 3-second pause between waves

- **Source**: `src/index.html` line 854 (`wavePause = 3.0`)
- **Precondition**: A wave has just been completed (all enemies killed, all enemies spawned).
- **Steps**:
  1. Kill the final enemy of Wave 1.
  2. Start a timer immediately.
  3. Observe the wave announcement text ("第 2 波 即将到来...").
  4. Record the time when Wave 2's first enemy spawns.
- **Expected Result**: The `wavePause` is set to exactly 3.0 seconds (line 854). No enemies spawn during the pause. The first enemy of the next wave spawns approximately 3 seconds after the last enemy of the previous wave died.
- **Pass Criteria**: Time between "wave cleared" and "first enemy of next wave spawned" is `3.0 +/- 0.1` seconds. The wave announcement text is displayed during the pause.

---

## Test Case: WAV-007 -- Wave increment on clear

- **Source**: `src/index.html` lines 279-290 (`startNextWave` increments `wave`), line 850-858 (wave completion check)
- **Precondition**: Wave N is active and all enemies are cleared.
- **Steps**:
  1. Clear Wave 1 -- observe `wave` variable.
  2. Clear Wave 2 -- observe `wave` variable.
  3. Clear Wave 3 -- observe `wave` variable.
  4. Clear Wave 4 -- observe `wave` variable and check game win.
- **Expected Result**:
  - After Wave 1 clear: `wavePause = 3.0`, `wave` is still 1 (next wave not started yet).
  - After pause: `wave` increments to 2 (`startNextWave` called at line 633).
  - After Wave 4 clear: `wave >= WAVES.length` (line 851), `gameWin = true` (line 852).
- **Pass Criteria**: `wave` counts 1 through 4 correctly. `gameWin` is true after Wave 4 is cleared. `WAVES.length == 4`.

---

## Test Case: WAV-008 -- Announcement displayed at wave start

- **Source**: `src/index.html` line 288-289 (`waveAnnouncement = w.name; announceTimer = 2.5`), line 854-856 (inter-wave announcement)
- **Precondition**: A new wave is starting (either initial Wave 1 or after inter-wave pause).
- **Steps**:
  1. Observe the screen when Wave 1 starts.
  2. Observe the screen during the inter-wave pause before Wave 2.
  3. Observe the screen when Wave 2 starts.
- **Expected Result**:
  - Wave start: announcement shows the wave name (e.g., "第 1 波") for 2.5 seconds (`announceTimer = 2.5`).
  - Inter-wave pause: announcement shows "[next wave name] 即将到来..." for 3.0 seconds (`announceTimer = 3.0`).
- **Pass Criteria**: Announcement text is rendered (`drawAnnouncement` at line 910) while `announceTimer > 0`. The text fades as alpha is `Math.min(1, announceTimer / 0.8)` (line 1942).

---

## Test Case: WAV-009 -- Spawn interval is wave-specific

- **Source**: `src/index.html` lines 261-264 (`spawnInterval: 1.0 / 0.8 / 0.6 / 2.0`)
- **Precondition**: Each wave is active sequentially.
- **Steps**:
  1. During Wave 1: measure time between enemy spawns.
  2. During Wave 2: measure time between enemy spawns.
  3. During Wave 3: measure time between enemy spawns.
  4. During Wave 4: measure time between enemy spawns.
- **Expected Result**:
  - Wave 1: spawns every 1.0 seconds.
  - Wave 2: spawns every 0.8 seconds.
  - Wave 3: spawns every 0.6 seconds.
  - Wave 4: spawns every 2.0 seconds (only 1 spawn, so interval not visibly repeated).
- **Pass Criteria**: `spawnTimer` resets to `w.spawnInterval` after each spawn (line 645). Timing matches `1.0 / 0.8 / 0.6 / 2.0`.

---

## Test Case: WAV-010 -- Game win triggers after Wave 4 cleared

- **Source**: `src/index.html` lines 851-852 (`if (wave >= WAVES.length) { gameWin = true; }`)
- **Precondition**: Wave 4 is active, acromantula boss is the last remaining enemy.
- **Steps**:
  1. Kill the acromantula boss (last enemy of Wave 4).
  2. Observe the game state.
- **Expected Result**: `gameWin = true`. The game win message ("恭喜！城堡守住了！") is displayed (line 911). No further enemies spawn.
- **Pass Criteria**: After killing the Wave 4 boss: `gameWin == true`, `waveAnnouncement` does not get set for a Wave 5, and `drawMessage` with "霍格沃茨感谢你的英勇战斗" is rendered.

---

## Test Case: WAV-011 -- Enemies spawned from SPAWN_X_RANGE and SPAWN_Y

- **Source**: `src/index.html` lines 276-277 (`SPAWN_X_RANGE = [120, W - 120]`, `SPAWN_Y = 60`)
- **Precondition**: A new enemy is about to spawn.
- **Steps**:
  1. Record the spawn position of every enemy in Wave 1 (12 enemies).
  2. Verify X and Y coordinates.
- **Expected Result**: Every enemy spawns with X in `[120, 840]` (`W - 120 = 840`) and Y in `[60, 90]` (`SPAWN_Y + Math.random() * 30`).
- **Pass Criteria**: `120 <= enemy.x <= 840` and `60 <= enemy.y <= 90` for all freshly spawned enemies (lines 321-322).

---

## Test Case: WAV-012 -- HUD wave display updates correctly

- **Source**: `src/index.html` lines 865, 881 (wave display update)
- **Precondition**: Game is running.
- **Steps**:
  1. Observe the HUD wave display during Wave 1.
  2. Observe after each wave transition.
- **Expected Result**:
  - During Wave 1: `wave-display` shows `1/4` (line 865).
  - After Wave 4 clear: `wave-display` shows `4/4`.
  - (Note: line 881 overwrites line 865's value, setting `wave-display` to just `wave` without `/WAVES.length`. The first assignment at line 865 shows "N/4" format but line 881 replaces it with just "N". This is a minor display inconsistency.)
- **Pass Criteria**: Wave number is always visible and increments correctly through 1, 2, 3, 4.

---

## Edge Cases Summary

| Case | Description | Expected |
|------|-------------|----------|
| Game win at exact wave boundary | `wave >= WAVES.length` when `wave == 4` (0-based index 3, after WAVES[3] cleared) | `gameWin = true`, no `startNextWave` call |
| All enemies killed before all spawned | Wave still needs to spawn remaining enemies | `enemiesSpawned < totalToSpawn` prevents wave completion (line 850 check requires both spawn complete AND array empty) |
| First enemy of wave spawns quickly | `spawnTimer = 0.5` at wave start | First enemy appears ~0.5s after wave begins, not at the full interval |
| Boss enemy stats | Acromantula in Wave 4 | hp=500, maxHp=500, radius=28, speed=35, damage=25 (line 316) |
| Dialogue blocks wave start | `dialogueActive == true` prevents update loop | No enemies spawn during dialogue |
