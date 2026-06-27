# Level 1: Castle Gate Defense -- Game Design Document

## 1. Overview

Level 1 ("Castle Gate Defense") is the introductory level of *Hogwarts Defense*. The player controls Harry Potter in a 2D top-down arena (960x640), defending Hogwarts' main gate from waves of Death Eaters. Hagrid introduces the level via dialogue. The player uses WASD movement, an automatic basic attack (7 damage, 0.35s cooldown, 350px range, targets nearest enemy), and four spell abilities (Q: Expelliarmus AoE knockback, E: Lumos cone stun, R: Incendio fire wall, F: Petrificus Totalus single-target freeze) to defeat 4 waves totaling 40 enemies plus 1 boss. The level serves as both narrative introduction and mechanical tutorial.

## 2. Player Fantasy

The player should feel like **Harry Potter defending Hogwarts** -- outnumbered but powerful, using iconic spells to control space and manage threats. The fantasy is one of **competence through spell mastery**: each spell has a distinct tactical role (crowd clear, area denial, lockdown, single-target disable), and using them correctly is the difference between being overwhelmed and dominating the field. The dialogue with Hagrid grounds the action in the Harry Potter universe. The aesthetic targets are **Challenge** (surviving escalating waves), **Sensation** (visual and audio feedback from spell effects), and **Fantasy** (embodying Harry Potter).

## 3. Detailed Rules

### 3.1 Arena

| Property | Value |
|----------|-------|
| Canvas size | 960 x 640 px |
| Player spawn | (480, 400) |
| Gate position (enemy target) | (432, 590) |
| Enemy spawn X range | [120, 840] |
| Enemy spawn Y | 60 (+ random 0-30) |
| Player boundary margin | 20 px from all edges |
| Enemy boundary clamp | 15 px from all edges |

### 3.2 Player (Harry Potter)

| Property | Value |
|----------|-------|
| Max HP | 100 |
| Movement speed | 200 px/s |
| Collision radius | 18 px |
| Movement | WASD / Arrow keys, 8-directional normalized |
| Facing | Last movement direction (atan2) |

### 3.3 Auto-Attack

- Automatically fires at the **nearest enemy within 350px range**
- Damage: 7 per hit
- Cooldown: 0.35 seconds
- Projectile speed: 600 px/s
- Projectile lifetime: 1.5 seconds
- Projectile collision radius: enemy.radius + 6 px
- Projectile removed on: hit, lifetime expiry, or leaving canvas bounds
- Fires once per cooldown cycle at the single nearest valid target

### 3.4 Spells

All spells share these rules:
- Cast on keypress (Q/E/R/F) if cooldown is zero
- Cooldown begins on cast, decrements by delta-time each frame
- No resource cost (only cooldown-gated)
- Spells do not interrupt movement
- Cannot cast while game is over, won, or during dialogue

#### Q -- Expelliarmus (除你武器)

| Property | Value |
|----------|-------|
| Cooldown | 3.0 s |
| Damage | 45 (flat, instant) |
| AoE range | 200 px radius from player |
| Knockback | 80 px radially outward from player |
| Visual duration | 0.45 s expanding ring |
| Screen shake | 0.2 s, intensity 8 |

**Rules**: Damages all enemies within 200px. Knocks each hit enemy 80px away from Harry (along the ray from Harry to enemy). Enemies are clamped to canvas bounds [15, W-15] / [15, H-15] after knockback. Killed enemies are removed immediately. Visual is a gold ring expanding from 5px to 200px radius.

#### E -- Lumos (荧光闪烁)

| Property | Value |
|----------|-------|
| Cooldown | 8.0 s |
| Effect | Stun 1.5 s |
| Cone full angle | PI/3.5 radians (~51.4 degrees) |
| Cone range | 260 px |
| Visual duration | 0.55 s |
| Targeting | Aims at nearest enemy; if no enemies, uses player facing |

**Rules**: All enemies within the cone (distance < 260px AND angle difference < coneAngle/2 from aim direction) are stunned for 1.5 seconds. Stunned enemies cannot move or shoot. Angle normalization wraps difference to [-PI, PI] before comparison.

#### R -- Incendio (烈火熊熊)

| Property | Value |
|----------|-------|
| Cooldown | 12.0 s |
| Damage | 22 per second |
| Duration | 3.0 s |
| Half-width | 45 px |
| Half-height | 14 px |
| Targeting | Nearest enemy's position; if no enemies, (player.x, player.y + 60) |
| Screen shake | 0.1 s, intensity 3 |

**Rules**: Places a static fire wall at the target position. Every frame, every enemy whose bounding box overlaps the fire wall rect (check: `abs(enemy.x - fw.x) < fw.halfWidth + enemy.radius && abs(enemy.y - fw.y) < fw.halfHeight + enemy.radius`) takes `22 * dt` damage. Fire wall is removed after 3 seconds. Multiple enemies can burn simultaneously. Total potential damage per enemy if they remain in the wall for the full duration: 66 HP.

#### F -- Petrificus Totalus (统统石化)

| Property | Value |
|----------|-------|
| Cooldown | 10.0 s |
| Effect | Freeze 3.0 s |
| Targeting range | 300 px |
| Targeting | Nearest enemy within 300px |

**Rules**: Freezes the single nearest enemy within 300px for 3 seconds. If no enemy is within 300px, the spell does NOT cast (cooldown is NOT consumed). Frozen enemies cannot move or shoot. Visual: ice-blue overlay and ice crystal particles.

### 3.5 Enemy Types

| Property | Apprentice | Elite | Acromantula (Boss) |
|----------|-----------|-------|-------------------|
| Max HP | 60 | 150 | 500 |
| Speed (px/s) | 75 | 50 | 35 |
| Radius (px) | 12 | 16 | 28 |
| Contact damage (per second) | 8 | 15 | 25 |
| Projectile damage (0.5x multiplier) | 4 | 7.5 | 12.5 |
| Projectile speed (px/s) | 200 | 200 | 200 |

### 3.6 Enemy AI

All enemies follow this behavior tree:

1. **Move toward gate**: Walk to GATE position (432, 590). Small random horizontal offset applied each frame: `GATE_X * 0.1 * (random - 0.5)`.
2. **Gate reached**: Once `y > GATE_Y` or `reachedGate` flag is set, switch target to Harry's current position.
3. **Chase Harry**: Move toward Harry's current position each frame.
4. **Shoot**: When distance to Harry < 350px, fire a projectile every 1.5-3.5 seconds (interval: `1.5 + Math.random() * 2`). Projectile speed 200 px/s, lifetime 3s, damage = enemy.baseDamage * 0.5.
5. **Contact damage**: When overlapping with Harry (`dist < enemy.radius + player.radius`), deal `enemy.damage * dt` damage per second of overlap. Enemy is pushed 3 px away from Harry each frame. Push direction uses `safeDist = max(dist, 0.01)` to prevent division by zero.
6. **Stunned/Frozen**: If `stunned > 0` or `frozen > 0`, all movement and shooting is suspended. Both timers count down independently each frame.

### 3.7 Wave Definitions

| Wave | Name | Apprentices | Elites | Boss | Spawn Interval |
|------|------|-------------|--------|------|---------------|
| 1 | 第 1 波 | 12 | 0 | None | 1.0 s |
| 2 | 第 2 波 | 10 | 4 | None | 0.8 s |
| 3 | 第 3 波 | 8 | 6 | None | 0.6 s |
| 4 | BOSS波 | 0 | 0 | 1 Acromantula | 2.0 s |

### 3.8 Wave Mechanics

- **Wave start**: 0.5s delay before first enemy spawn. Announcement banner displayed for 2.5s.
- **Spawning**: Each wave spawns enemies at its spawnInterval until all enemies (apprentices + elites + boss) are spawned. Elite/apprentice order is randomized: 50% chance to spawn elite when both types remain; if only one type remains, spawn that type. Boss spawns LAST (after `enemiesSpawned === w.enemies + w.elites`, i.e., after all normal+elite enemies are spawned).
- **Spawn position**: Random X in [120, 840], Y in [60, 90].
- **Wave clear condition**: `enemiesSpawned >= totalToSpawn AND enemies.length === 0`. All enemies must be spawned AND killed.
- **Between waves**: 3.0s pause with "next wave" announcement banner (displayed for 3.0s). Game chime sound plays at pause start.
- **Level win**: Wave 4 cleared (all enemies dead after Wave 4's total complement is spawned and killed).
- **Level lose**: Player HP reaches 0.

### 3.9 Dialogue System

10 dialogue lines presented sequentially before gameplay begins. Game loop (`update`) is entirely skipped while `dialogueActive === true`. Player advances dialogue with any key (Enter, Space, or E) or mouse click.

| Line | Speaker | Text |
|------|---------|------|
| 0 | 海格 (Hagrid) | 波特！食死徒从禁林方向来了！ |
| 1 | 海格 (Hagrid) | 守住城堡大门，我去护着其他新生！ |
| 2 | 哈利 (Harry) | 交给我，海格！ |
| 3 | 旁白 (Narrator) | WASD / 方向键 -- 移动 |
| 4 | 旁白 (Narrator) | 哈利会自动射击最近的敌人 |
| 5 | 旁白 (Narrator) | Q -- 除你武器：金色冲击波震退周围敌人并造成 45 伤害 |
| 6 | 旁白 (Narrator) | E -- 荧光闪烁：朝敌人方向射出锥形光芒，眩晕 1.5 秒 |
| 7 | 旁白 (Narrator) | R -- 烈火熊熊：在最近敌人脚下放火墙，每秒灼烧 22 伤害 |
| 8 | 旁白 (Narrator) | F -- 统统石化：冰冻最近的敌人 3 秒 |
| 9 | 旁白 (Narrator) | 4 波敌人 + BOSS，守住霍格沃茨！ |

When the last line (index 9) is advanced, `dialogueActive` is set to false and gameplay begins on the next frame with Wave 1's initial 0.5s spawn delay.

## 4. Formulas

### 4.1 Auto-Attack DPS

```
DPS_basic = damage_basic / cooldown_basic
          = 7 / 0.35
          = 20 DPS
```

### 4.2 Projectile Collision Detection

```
distance = sqrt( (proj.x - enemy.x)^2 + (proj.y - enemy.y)^2 )
hit = distance < enemy.radius + 6
```

On hit: enemy.hp -= 7, projectile destroyed, hit particles spawned.

### 4.3 Contact Damage (per frame)

```
damage_per_frame = enemy.damage * dt

Per second rates:
  Apprentice:   8 HP/s
  Elite:       15 HP/s
  Acromantula: 25 HP/s
```

### 4.4 Enemy Projectile Damage

```
proj_damage = enemy.damage * 0.5

  Apprentice projectile:   4
  Elite projectile:        7.5
  Acromantula projectile: 12.5
```

Enemy projectile collision with Harry: distance < player.radius + 5.

### 4.5 Expelliarmus (Q) -- AoE Knockback + Damage

```
For each enemy within 200px of player:
  angle = atan2(enemy.y - player.y, enemy.x - player.x)
  enemy.x += cos(angle) * 80
  enemy.y += sin(angle) * 80
  enemy.x = clamp(enemy.x, 15, 945)
  enemy.y = clamp(enemy.y, 15, 625)
  enemy.hp -= 45
  if enemy.hp <= 0: remove enemy
```

### 4.6 Lumos (E) -- Cone Hit Detection

```
aim_angle = atan2(nearest_enemy.y - player.y, nearest_enemy.x - player.x)
             || player.facing (if no enemies exist)

coneHalfAngle = PI / 7   // ~25.7 degrees
coneRange = 260

For each enemy:
  distance = sqrt( (enemy.x - player.x)^2 + (enemy.y - player.y)^2 )
  if distance < coneRange:
    enemy_angle = atan2(enemy.y - player.y, enemy.x - player.x)
    angle_diff = enemy_angle - aim_angle
    // Normalize to [-PI, PI]
    while angle_diff > PI:   angle_diff -= 2 * PI
    while angle_diff < -PI:  angle_diff += 2 * PI
    if abs(angle_diff) < coneHalfAngle:
      enemy.stunned = 1.5
```

### 4.7 Incendio (R) -- Fire Wall Damage

```
Per frame, for each fire wall:
  for each enemy:
    overlap_x = abs(enemy.x - fw.x) < fw.halfWidth + enemy.radius   // 45 + r
    overlap_y = abs(enemy.y - fw.y) < fw.halfHeight + enemy.radius  // 14 + r
    if overlap_x AND overlap_y:
      enemy.hp -= 22 * dt
      if enemy.hp <= 0: remove enemy

Maximum damage per enemy (full 3s duration): 22 * 3 = 66 HP
```

Multiple fire walls can overlap. An enemy takes damage from each independently.

### 4.8 Time-to-Kill (TTK) -- Auto-attack Only

```
TTK_apprentice  = 60  / 20 = 3.0 s   (9 shots)
TTK_elite       = 150 / 20 = 7.5 s   (22 shots)
TTK_acromantula = 500 / 20 = 25.0 s  (72 shots)
```

### 4.9 Time-to-Kill -- With Spell Rotation

```
Apprentice: Q one-shots (45 + remaining 15 from 3 auto-attacks = ~1.0s with Q)
             OR ~3 auto-attacks without Q

Elite:       Q (45) + 15 auto-attacks (105) = ~5.5s
             Q + R full burn (45 + 66 = 111) + 6 auto-attacks = ~3.5s

Acromantula: Q + R (45 + 66 = 111) + ~56 auto-attacks (389) = ~20s
             With full spell rotation cycling: ~15-18s
```

### 4.10 Wave Duration Estimates

```
Wave 1 total:  12 spawns * 1.0s spawn + cleanup  = ~12s spawn + ~8s combat  = ~20s
Wave 2 total:  14 spawns * 0.8s spawn + cleanup  = ~11.2s spawn + ~12s combat = ~23s
Wave 3 total:  14 spawns * 0.6s spawn + cleanup  = ~8.4s spawn + ~14s combat  = ~22s
Wave 4 total:   1 spawn  * 2.0s spawn + boss fight = ~2s spawn + ~25s combat  = ~27s

Total gameplay:        ~92 seconds
Inter-wave pauses:     3 * 3s = 9 seconds
Dialogue (estimated): ~15-20 seconds

Total level duration:  ~116-121 seconds
```

### 4.11 Delta-Time Capping

```
dt = min( (now - lastTime) / 1000, 0.05 )
```

Caps at 50ms (equivalent to 20 FPS minimum). Prevents physics explosions from frame spikes (e.g., browser tab backgrounding).

### 4.12 Contact Pushback -- Division-by-Zero Guard

```
distToPlayer = sqrt( (enemy.x - player.x)^2 + (enemy.y - player.y)^2 )
safeDist = max(distToPlayer, 0.01)
pushX = (enemy.x - player.x) / safeDist * 3
pushY = (enemy.y - player.y) / safeDist * 3
enemy.x += pushX
enemy.y += pushY
```

## 5. Edge Cases

### 5.1 Enemy-Harry Exact Overlap (Division by Zero)

When enemy and Harry occupy the exact same position, `distToPlayer === 0`. The pushback formula uses `safeDist = max(distToPlayer, 0.01)`. At exact overlap (0,0 vector), the division produces `0 / 0.01 = 0` in both axes, so the enemy receives zero push that frame. However, because enemy movement and contact damage both happen in the same frame, the enemy is typically offset by at least one pixel from the next frame's movement, allowing the pushback to take effect on subsequent frames. The enemy cannot remain at exactly (0,0) overlap across multiple frames due to continuous enemy movement toward the player.

### 5.2 Spell Casting with Zero Enemies

- **Q (Expelliarmus)**: Casts normally, creates full visual effect (expanding gold ring), hits nothing. Cooldown is consumed. This is intentional design -- panic-casting Q when enemies are about to spawn is a valid player behavior.
- **E (Lumos)**: Uses `player.facing` (last movement direction) as aim angle since no nearest enemy exists. Cone visual appears in facing direction, hits nothing. Cooldown consumed. No stun is applied.
- **R (Incendio)**: Targets default position `(player.x, player.y + 60)`. Fire wall placed at that location, burns nothing. Cooldown consumed.
- **F (Petrificus Totalus)**: No enemy within 300px. Spell does NOT cast -- no cooldown consumed, no visual effect, no sound. Keypress is silently ignored. This prevents wasting a long-cooldown single-target spell.

### 5.3 Spell Targeting with Dead Enemies

Dead enemies are removed from the `enemies` array immediately when HP reaches 0 (via `enemies.splice(i, 1)` in Q's damage loop and R's fire wall damage loop). Therefore, targeting for E, R, and F always operates on the live enemy array. An enemy killed by an auto-attack projectile between frames is not present in the array when the next spell cast occurs.

### 5.4 Enemy Clamped Out of Bounds After Knockback

After Q knockback pushes an enemy, the position is clamped to [15, W-15] horizontally and [15, H-15] vertically. This prevents enemies from being pushed off-screen. Clamped enemies remain valid targets for all spells and auto-attacks. The clamp is applied per-enemy immediately after the knockback displacement.

### 5.5 Player Boundary Enforcement

Player position is clamped to [20, W-20] horizontally and [20, H-20] vertically every frame after movement. The player cannot leave the playable area under any circumstances. The 20px margin differs from the enemy 15px margin because the player has a larger collision radius (18 vs 12).

### 5.6 Simultaneous Stun and Freeze Overlap

Both `stunned` and `frozen` timers count down independently each frame. An enemy is disabled (no movement, no shooting) while EITHER `stunned > 0` OR `frozen > 0`. If Lumos is cast (1.5s stun) and Petrificus Totalus is immediately cast on the same enemy (3.0s freeze), the enemy is disabled for 3.0s total -- the freeze overrides the remaining stun time. If the order is reversed (freeze first, then stun), the total disable is still max(freeze_remaining, stun_duration). The visual indicators for both effects display simultaneously (stun stars + ice overlay).

### 5.7 Fire Wall Stacking

Multiple fire walls can coexist from separate R casts. The 12s cooldown with 3s duration means the theoretical maximum overlap is 2 walls briefly (one wall has ~3s remaining when the second is placed at t=12s, overlapping for up to 3s). An enemy in the overlapping area takes `22 * dt` from EACH wall, effectively 44 DPS in the overlap zone. This is intentionally rewarded coordinated play but is self-limiting due to the cooldown.

### 5.8 Projectile Single-Target Piercing Prevention

Auto-attack projectiles check enemies in reverse iteration order and break after the first hit. A projectile damages exactly one enemy and is then destroyed. It never pierces through to hit a second enemy behind the first.

### 5.9 Delta-Time Spike Protection

`dt` is capped at 0.05 (50ms, ~20 FPS minimum). If a frame takes longer (e.g., browser tab was backgrounded, OS scheduling delay), the simulation runs at worst-case 20 FPS physics rather than extrapolating. This prevents: enemies teleporting across the arena, player HP dropping from 100 to 0 in one frame, multiple enemies spawning simultaneously when only one was intended, and cooldowns jumping from full to zero.

### 5.10 Wave Completion vs Persistent Enemies

Wave completion requires both conditions: `enemiesSpawned >= totalToSpawn` AND `enemies.length === 0`. If a wave's complement has been fully spawned but enemies remain alive (e.g., a kited elite at low HP), the wave continues indefinitely until the last enemy dies. This prevents the player from being rushed into the next wave before cleaning up.

### 5.11 Boss Spawn Timing

The boss spawns when `enemiesSpawned === w.enemies + w.elites`. For Wave 4 (0 apprentices + 0 elites), enemiesSpawned starts at 0, equals 0+0, so the boss spawns as the first and only enemy of Wave 4 at the initial 0.5s spawn delay.

### 5.12 Game Over Detection Points

`player.hp <= 0` is checked in three places:
1. Contact damage loop (after `player.hp -= enemy.damage * dt`)
2. Enemy projectile collision (after `player.hp -= ep.damage`)
3. Post-processing at end of `update()` function

Game over is set to `true` and the update loop stops on the next frame. All input is disabled. The death message renders. `player.hp` is clamped to a minimum of 0 for display via `Math.round(Math.max(0, Math.min(player.maxHp, player.hp)))`.

### 5.13 Elite/Apprentice Spawn Randomization Algorithm

```
remainingElites = w.elites - elitesSpawned
remainingNormal = w.enemies - (enemiesSpawned - elitesSpawned)

if remainingElites > 0 AND (remainingNormal <= 0 OR Math.random() < 0.5):
    type = 'elite'
else if remainingNormal <= 0:
    type = 'elite'  // fallback: only elites remain
else:
    type = 'apprentice'
```

This produces natural mixed waves. All elites could spawn first, or all apprentices could spawn first, or any mix in between. The distribution tends toward even mixing but is not forced.

### 5.14 Dialogue-to-Gameplay Transition

During dialogue (`dialogueActive === true`), the `update()` function is entirely skipped. Enemies do not spawn, timers do not advance, cooldowns do not tick. When the final dialogue line (index 9) is advanced, `dialogueActive` is set to `false` on that same input event. The next frame, `update()` begins executing. Wave 1 starts with its 0.5s initial spawn delay, and the spawn timer begins counting down from that frame.

### 5.15 Projectile Out-of-Bounds Cleanup

Projectiles (both player and enemy) are removed when `life <= 0` OR `x < 0` OR `x > W` OR `y < 0` OR `y > H`. This prevents orphaned projectiles from persisting indefinitely if they somehow escape the arena (e.g., fired from an edge-case position).

## 6. Dependencies

### 6.1 Systems This Level Depends On

| System | Status | Purpose |
|--------|--------|---------|
| Core Combat System | Not yet documented (GDD pending) | Auto-attack projectile logic, collision detection, damage application |
| Spell/Skill System | Not yet documented (GDD pending) | Cooldown management, spell effect dispatch, targeting resolution |
| Enemy AI System | Not yet documented (GDD pending) | Gate-pathfinding, Harry-chasing, shooting behavior, state management |
| Wave Management System | Not yet documented (GDD pending) | Wave definitions, spawn scheduling, wave transition logic |
| Dialogue System | Not yet documented (GDD pending) | Sequential text display, input interception, speaker rendering |
| HUD System | Not yet documented (GDD pending) | HP display, wave counter, skill cooldown bars, damage stats |
| Particle System | Not yet documented (GDD pending) | Visual feedback particles for hits, spells, death effects |
| Screen Shake System | Not yet documented (GDD pending) | Camera offset on spell casts |
| Audio System | Not yet documented (GDD pending) | Procedural sound effects via Web Audio API oscillators |

### 6.2 Systems That Depend On This Level

| System | Purpose |
|--------|---------|
| Level Select Screen | Entry point -- Level 1 is the first playable level |
| Save/Progress System | Records Level 1 completion state |
| Level 2 GDD | Builds on mechanics and enemy types introduced in Level 1 |
| Damage Statistics Tracker | Level 1 is the initial data source for balance analytics |

### 6.3 Data Flow Diagram

```
Input (Keyboard/Mouse)
  |
  +--> Player Movement (WASD normalization, boundary clamp)
  |
  +--> Spell Casting (Q/E/R/F)
  |      |
  |      +--> Cooldown Manager (decrement, HUD update)
  |      +--> Spell Effect Dispatcher
  |             |
  |             +--> Q: AoE damage + knockback -> Enemy HP, Enemy position
  |             +--> E: Cone check -> Enemy.stunned
  |             +--> R: Fire wall placement -> Per-frame damage -> Enemy HP
  |             +--> F: Nearest-enemy-in-range check -> Enemy.frozen
  |
  +--> Wave Manager
         |
         +--> Spawn Scheduler (timer, interval, remaining count)
         +--> Wave Transition (clear check, pause, next wave init)
                |
                +--> Enemy AI
                       |
                       +--> Movement (gate, chase)
                       +--> Shooting (projectile spawn, timer)
                       +--> Contact damage -> Player HP
                       +--> Death -> particles, array removal

Player HP --> Win/Lose State Machine
Enemies[].length --> Wave clear detection
```

## 7. Tuning Knobs

All values below are sourced from the current implementation. They should be configurable in external data files for balance iteration (currently inlined in `src/index.html` as a prototype).

### 7.1 Player Tuning Knobs

| Knob | Value | Safe Range | Category | Rationale |
|------|-------|------------|----------|-----------|
| `player.maxHp` | 100 | 50-200 | Gate | Determines survival window; 100 HP gives ~12.5s against single apprentice contact |
| `player.speed` | 200 | 150-300 | Feel | Player is 2.7x faster than apprentice; kiting is possible but not trivial |
| `player.radius` | 18 | 14-24 | Feel | Visual and collision size; must match the sprite rendering |

### 7.2 Auto-Attack Tuning Knobs

| Knob | Value | Safe Range | Category | Rationale |
|------|-------|------------|----------|-----------|
| `autoAttack.damage` | 7 | 4-12 | Curve | 20 DPS baseline; TTK_apprentice = 3s, TTK_elite = 7.5s |
| `autoAttack.cooldown` | 0.35 | 0.2-0.6 | Feel | Attack rhythm; 0.35s feels rapid but not frantic |
| `autoAttack.range` | 350 | 250-450 | Feel | ~36% of arena width; large enough to engage before enemies reach gate |
| `autoAttack.projectileSpeed` | 600 | 400-1000 | Feel | Covers max range in ~0.58s; fast enough to hit moving targets reliably |

### 7.3 Spell Tuning Knobs

| Knob | Value | Safe Range | Category | Rationale |
|------|-------|------------|----------|-----------|
| `spell.q.damage` | 45 | 20-80 | Curve | One-shots apprentices (60 HP); elites left at 105 HP requiring follow-up |
| `spell.q.cooldown` | 3.0 | 2.0-5.0 | Gate | Shortest CD; intentional as the primary wave-clear and emergency escape tool |
| `spell.q.range` | 200 | 150-250 | Feel | Larger than auto-attack range (350) for positioning forgiveness |
| `spell.q.knockback` | 80 | 40-120 | Feel | ~40% of Q AoE radius; pushes enemies out of immediate contact range |
| `spell.e.stunDuration` | 1.5 | 1.0-2.5 | Gate | Long enough for repositioning; short enough to require timing skill |
| `spell.e.cooldown` | 8.0 | 6.0-12.0 | Gate | Utility-only (no damage); longer CD reflects high-impact CC with no resource cost |
| `spell.e.coneAngle` | PI/3.5 | PI/5 to PI/2.5 | Feel | ~51 deg; wider = easier multihit, narrower = more skillful placement |
| `spell.e.coneRange` | 260 | 200-350 | Feel | Extends past auto-attack range for pre-engagement CC |
| `spell.r.damagePerSec` | 22 | 12-35 | Curve | 66 max total per wall; meaningful but does not one-shot elites (150 HP) |
| `spell.r.duration` | 3.0 | 2.0-5.0 | Gate | Area denial window; 3s matches one full engagement cycle |
| `spell.r.cooldown` | 12.0 | 8.0-18.0 | Gate | Longest CD; highest total damage potential per cast (66 per enemy) |
| `spell.r.halfWidth` | 45 | 30-60 | Feel | Fire wall width; must catch enemies walking in a line |
| `spell.r.halfHeight` | 14 | 8-24 | Feel | Fire wall thinness; requires placement skill, rewards predicting enemy path |
| `spell.f.freezeDuration` | 3.0 | 2.0-5.0 | Gate | Free damage window; 3s = ~9 auto-attacks = 63 damage |
| `spell.f.cooldown` | 10.0 | 8.0-15.0 | Gate | Between E and R in CD; single-target focus justifies moderate cooldown |
| `spell.f.targetRange` | 300 | 200-400 | Feel | Shorter than auto-attack range (350) for risk/reward positioning |

### 7.4 Enemy Tuning Knobs

| Knob | Value | Safe Range | Category | Rationale |
|------|-------|------------|----------|-----------|
| `apprentice.hp` | 60 | 40-100 | Curve | 3 auto-attacks to kill; defines the baseline enemy |
| `apprentice.speed` | 75 | 50-100 | Curve | 0.375x player speed; player can comfortably kite |
| `apprentice.damage` | 8 | 5-15 | Curve | 12.5s to kill Harry solo; pressure without being lethal alone |
| `elite.hp` | 150 | 100-250 | Curve | 2.5x apprentice HP; requires spell combos for efficient clear |
| `elite.speed` | 50 | 35-70 | Curve | Noticeably slower; telegraphs "tanky" enemy type |
| `elite.damage` | 15 | 10-25 | Curve | ~2x apprentice damage; demands spell response when in contact |
| `acromantula.hp` | 500 | 300-800 | Curve | Boss-level pool; requires 3-4 full spell rotations |
| `acromantula.speed` | 35 | 20-50 | Curve | Slowest enemy; player can kite easily but projectile threat is main danger |
| `acromantula.damage` | 25 | 15-40 | Curve | High contact damage punishes being cornered; 4s to kill Harry alone |
| `enemy.projectileSpeed` | 200 | 150-300 | Feel | 1/3 of player projectile speed; dodgeable with awareness |
| `enemy.shootCooldown` | 1.5 + rand*2 | 1.0-5.0 | Gate | Average 2.5s between volleys; controls incoming damage pacing |
| `enemy.shootRange` | 350 | 250-450 | Gate | Matches player auto-attack range for symmetric engagement |

### 7.5 Wave Tuning Knobs

| Knob | Value | Safe Range | Category | Rationale |
|------|-------|------------|----------|-----------|
| `wave1.enemyCount` | 12 | 8-16 | Gate | Introductory wave; generous spacing for learning |
| `wave1.spawnInterval` | 1.0 | 0.8-1.5 | Gate | 1 enemy/second; very manageable, tutorial pace |
| `wave2.enemyCount` | 10 + 4 elite | - | Gate | Introduces elite type alongside familiar apprentices |
| `wave2.spawnInterval` | 0.8 | 0.5-1.2 | Gate | Slightly faster; pressure increases |
| `wave3.enemyCount` | 8 + 6 elite | - | Gate | Elite-heavy; tests spell rotation management |
| `wave3.spawnInterval` | 0.6 | 0.4-1.0 | Gate | Fastest spawn; most intense non-boss wave |
| `wave4.bossCount` | 1 | - | Gate | Single Acromantula; solo boss fight |
| `wave4.spawnInterval` | 2.0 | 1.0-3.0 | Gate | Boss spawns quickly after wave start (at 0.5s delay) |
| `wavePause` | 3.0 | 2.0-5.0 | Gate | Between-wave breather; allows all cooldowns to reset (longest CD is 12s but 3s pause means player must manage partial resets) |
| `initialSpawnDelay` | 0.5 | 0.3-1.0 | Feel | Brief anticipation before first enemy appears |
| `announceDuration` | 2.5 | 1.5-3.5 | Feel | Wave start banner display time |
| `nextWaveAnnounce` | 3.0 | 2.0-4.0 | Feel | "Next wave incoming" banner matches pause duration |

### 7.6 Delta-Time Protection

| Knob | Value | Safe Range | Category | Rationale |
|------|-------|------------|----------|-----------|
| `dtCap` | 0.05 | 0.033-0.1 | Gate | 50ms cap = 20 FPS minimum simulation; lower = safer but choppier at low FPS |

## 8. Acceptance Criteria

### 8.1 Functional Criteria

**AC-1: Level Initialization**
- [ ] Game loads to Level 1 arena (960x640 canvas, castle gate rendered at bottom center)
- [ ] Harry spawns at position (480, 400) with 100/100 HP displayed
- [ ] Dialogue sequence begins immediately with Hagrid's first line visible
- [ ] No enemies are present on screen during dialogue
- [ ] Damage statistics tracker is initialized to zero for all categories

**AC-2: Dialogue Flow**
- [ ] All 10 dialogue lines display in fixed sequence (Hagrid x2, Harry x1, Narrator x7)
- [ ] Enter key, Space key, E key, and left mouse click each advance dialogue by one line
- [ ] Other keys (WASD, Q, R, F) do NOT advance dialogue and do NOT trigger gameplay actions
- [ ] Harry's sprite does not move and spell cooldowns do not tick during dialogue
- [ ] After advancing past line 9, dialogue box disappears and Wave 1 begins within 0.5s

**AC-3: Player Movement**
- [ ] W, A, S, D keys and Arrow keys (Up, Left, Down, Right) move Harry
- [ ] Diagonal movement (e.g., W+A) produces normalized movement at 200 px/s (not ~283 px/s)
- [ ] Releasing all keys stops Harry immediately (no momentum/inertia)
- [ ] Harry cannot leave the arena boundaries: position clamped to [20, 940] horizontal, [20, 620] vertical
- [ ] Player facing direction updates to the last movement direction when moving

**AC-4: Auto-Attack Behavior**
- [ ] When 1+ enemies are within 350px, Harry fires a projectile at the single nearest enemy
- [ ] Projectile spawns at Harry's position and travels at 600 px/s toward the enemy's position at time of firing
- [ ] Projectile deals exactly 7 damage on collision (verify: apprentice HP goes from 60 to 53)
- [ ] Minimum time between shots is 0.35s (verify with frame counting at 60 FPS: 21 frames)
- [ ] Projectile is destroyed and removed from the array on: (a) enemy collision, (b) lifetime > 1.5s, (c) leaving canvas bounds
- [ ] When zero enemies are within 350px, no projectiles fire

**AC-5: Spell Q -- Expelliarmus**
- [ ] Pressing Q while cooldown is zero and game is active: spell casts
- [ ] All enemies within 200px radius take exactly 45 damage
- [ ] Each hit enemy is displaced 80px radially away from Harry's position
- [ ] Displaced enemy position is clamped to [15, 945] x [15, 625]
- [ ] Enemies reduced to 0 or fewer HP are removed from the enemies array immediately
- [ ] Gold expanding ring visual plays (0.45s duration, 5px to 200px radius)
- [ ] Screen shake triggers (0.2s duration, intensity 8)
- [ ] Cooldown set to 3.0s; pressing Q during cooldown has no effect
- [ ] HUD icon transitions to "recharging" state with red fill bar

**AC-6: Spell E -- Lumos**
- [ ] Pressing E while cooldown is zero and game is active: spell casts
- [ ] If enemies exist: cone aims at nearest enemy; if no enemies: cone aims in player.facing direction
- [ ] All enemies within 260px AND within cone half-angle of PI/7 (~25.7 deg) of aim are stunned
- [ ] Stunned enemies have `stunned = 1.5` and do not move or shoot for exactly 1.5s
- [ ] Stun stars visual orbits stunned enemies' heads
- [ ] White cone visual plays (0.55s duration)
- [ ] Cooldown set to 8.0s; pressing E during cooldown has no effect
- [ ] Spell casts and consumes cooldown even when zero enemies are hit

**AC-7: Spell R -- Incendio**
- [ ] Pressing R while cooldown is zero and game is active: spell casts
- [ ] Fire wall placed at nearest enemy's position; if no enemies, at (player.x, player.y + 60)
- [ ] Fire wall lasts exactly 3.0s, then is removed
- [ ] Each frame: any enemy whose rect overlaps fire wall rect takes `22 * dt` damage
- [ ] Overlap test: `abs(enemy.x - fw.x) < 45 + enemy.radius AND abs(enemy.y - fw.y) < 14 + enemy.radius`
- [ ] Multiple enemies can burn simultaneously in the same wall
- [ ] Enemies reaching 0 HP from fire damage are removed and death particles spawn
- [ ] Cooldown set to 12.0s; pressing R during cooldown has no effect
- [ ] Screen shake triggers (0.1s duration, intensity 3)

**AC-8: Spell F -- Petrificus Totalus**
- [ ] Pressing F while cooldown is zero and game is active: spell casts
- [ ] Targets the single nearest enemy within 300px range
- [ ] If no enemy is within 300px: spell does NOT cast, cooldown is NOT consumed, no visual/sound effect
- [ ] If target found: target.frozen = 3.0, frozen enemy does not move or shoot for exactly 3.0s
- [ ] Ice-blue overlay and crystal particles render on frozen enemy
- [ ] Cooldown set to 10.0s (only when spell successfully casts)
- [ ] Freeze and stun can coexist (enemy disabled while either is active)

**AC-9: Enemy Spawning**
- [ ] Wave 1: Exactly 12 apprentices spawn. Spawn interval: 1.0s. First spawn at 0.5s after wave start.
- [ ] Wave 2: Exactly 10 apprentices and 4 elites spawn. Spawn interval: 0.8s. Types mixed randomly.
- [ ] Wave 3: Exactly 8 apprentices and 6 elites spawn. Spawn interval: 0.6s. Types mixed randomly.
- [ ] Wave 4: Exactly 1 Acromantula boss spawns. Spawn interval: 2.0s. First spawn at 0.5s after wave start.
- [ ] All enemies spawn at random X in [120, 840], Y in [60, 90]
- [ ] Elite/apprentice mixing: when both types remain, each spawn call has 50% chance elite
- [ ] Boss always spawns last in its wave (after all apprentice+elite enemies)
- [ ] `enemiesSpawned` counter increments by 1 per spawn regardless of type

**AC-10: Enemy AI Behavior**
- [ ] On spawn: enemy walks toward gate position (432, 590) with small random X offset
- [ ] When enemy.y > 590 (gate Y): enemy.reachedGate = true, switches to chasing Harry
- [ ] While chasing Harry: moves toward Harry's current position each frame at its type's speed
- [ ] Shooting: fires projectile (200 px/s, 0.5x base damage) toward Harry every 1.5-3.5s when within 350px
- [ ] Contact: when `dist < enemy.radius + player.radius`, Harry takes `enemy.damage * dt` damage per second
- [ ] Contact pushback: enemy pushed 3px away from Harry each frame; safeDist prevents div/0
- [ ] Stunned or frozen enemies: skip movement update and shooting; both timers count down each frame
- [ ] Enemy HP bar renders above sprite and updates each frame as HP changes

**AC-11: Wave Transitions**
- [ ] Wave completion detected when: all enemies spawned AND zero enemies alive
- [ ] On wave clear: 3.0s pause begins, "next wave" announcement displays
- [ ] Chime sound plays at pause start
- [ ] During pause: no enemies spawn, existing projectiles continue, player can move
- [ ] After 3.0s: next wave starts automatically with its initial 0.5s spawn delay
- [ ] Wave counter in HUD updates (e.g., "1/4" -> "2/4")

**AC-12: Win Condition**
- [ ] When Wave 4's boss is killed AND all Wave 4 spawns are complete: victory triggers
- [ ] Victory screen displays: "Congratulations! The castle is saved!" with subtitle
- [ ] Damage statistics panel shows final tallies for basic, Q, E, R, F damage
- [ ] All player input disabled; game loop stops updating

**AC-13: Lose Condition**
- [ ] When Harry's HP reaches 0 via any damage source: game over triggers in same frame
- [ ] Game over screen displays: "Harry has fallen..." with "Press F5 to retry" subtitle
- [ ] All player input (movement, spells) disabled; game loop stops updating
- [ ] Harry's HP display shows 0

**AC-14: Spell Cooldown Tracking**
- [ ] Each spell cooldown decrements by exactly `dt` per frame
- [ ] Cooldowns cannot go below 0
- [ ] HUD skill icon: border glow + green fill bar when ready; red fill bar when recharging
- [ ] Cooldown bar width transitions from 100% (ready) to 0% (just cast) linearly over the cooldown duration
- [ ] HUD displays cooldown seconds remaining (one decimal) in controls hint

### 8.2 Experiential Criteria

**AC-15: Difficulty Pacing**
- [ ] Wave 1: a first-time player with no prior knowledge can clear using only auto-attack and movement (no spells required)
- [ ] Wave 2: elite enemies create enough pressure that spell usage feels necessary, not optional
- [ ] Wave 3: the highest density of elites requires cycling all four spells to avoid being overwhelmed
- [ ] Wave 4: boss fight lasts 15-30 seconds and requires multiple full spell rotations
- [ ] 3s inter-wave pause provides adequate time to read the next-wave announcement and reposition

**AC-16: Spell Feel**
- [ ] Q: knockback visibly and audibly pushes enemies away; gold ring conveys power
- [ ] E: stunned enemies freeze mid-step with star particles; tactical advantage is immediately obvious
- [ ] R: fire wall flame animation is visually distinct; enemies burning inside it take accelerated damage
- [ ] F: single target freeze creates a clear "this enemy is locked down" read with ice visuals

**AC-17: Feedback Clarity**
- [ ] Every player-hit-enemy event produces: hit sound + damage flash on enemy + hit particles
- [ ] Every enemy-hit-player event produces: red damage flash overlay on Harry + red particles
- [ ] Enemy death produces: explosion sound + death particle burst (color varies by enemy type)
- [ ] Enemy HP bar color changes at thresholds: green (>50%), yellow (>25%), red (<=25%)
- [ ] Player HP change is immediately reflected in HUD number and color

**AC-18: First-Time Player Experience**
- [ ] A playtester unfamiliar with the game can complete Level 1 within 3 attempts
- [ ] All 4 spells are explained in dialogue before gameplay begins
- [ ] Movement and auto-attack are explained in dialogue before gameplay begins
- [ ] The wave structure (4 waves + boss) is communicated before gameplay begins
- [ ] No unexplained mechanics exist -- every system visible in gameplay is mentioned in the tutorial dialogue
