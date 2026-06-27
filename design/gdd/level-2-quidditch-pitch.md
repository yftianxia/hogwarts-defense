# Level 2: Quidditch Pitch Defense — Game Design Document

## 1. Overview

Level 2 takes place at the Hogwarts Quidditch pitch, immediately following
Level 1's successful defense of the castle main gate. Hermione Granger is
studying in the stands when Death Eaters launch a secondary attack through the
pitch grounds. With Harry holding the main gate, Hermione must defend the pitch
alone using her tactical arsenal of charm-based spells.

Hermione's playstyle is **tactical control** — placing hazards, controlling
enemy positioning, and disabling threats — in contrast to Harry's **aggressive
burst-damage** style in Level 1.

## 2. Player Fantasy

The player embodies Hermione's intelligence and resourcefulness. Rather than
overpowering enemies with raw force, the player **outsmarts** them — funneling
enemies through fire hazards, freezing groups for chain combos, lifting
priority targets out of the fight, and detonating clustered enemies. The
feeling is being the **smartest witch in the fight**, turning the battlefield
into a puzzle where every spell sets up the next.

## 3. Detailed Rules

### 3.1 Protagonist: Hermione Granger

| Attribute | Value | Notes |
|-----------|-------|-------|
| HP | 90 | Lower than Harry (100) — Hermione is less durable |
| Speed | 220 px/s | Slightly faster than Harry (200) — relies on positioning |
| Radius | 16 | Slightly smaller hitbox |
| Auto-attack damage | 6 | Lower than Harry (7) |
| Auto-attack range | 380 px | Slightly longer than Harry (350) |
| Auto-attack rate | 0.30s | Slightly faster than Harry (0.35s) |

### 3.2 Hermione's Spells

#### Q: 蓝色风铃草火焰 (Bluebell Flame)
- **Effect**: Places a blue fire patch at the nearest enemy's position.
  Fire persists for 5 seconds. Enemies within 50px radius take 20 damage/sec
  and are slowed by 30% while inside. The fire does NOT damage Hermione.
- **Cooldown**: 10 seconds
- **Visual**: Blue-white flame particles, circular glow on ground
- **Audio**: Soft crackling fire (sine + noise)
- **Strategy**: Zone control — block enemy paths, create kill zones

#### E: 冰冻术 / Immobulus (Freezing Charm)
- **Effect**: Cone-shaped cold blast (half-angle PI/6 = 30°, range 280px).
  All enemies in the cone are frozen for 1.5 seconds. No damage.
- **Cooldown**: 9 seconds
- **Visual**: White-blue cone with frost particles, enemies turn ice-blue
- **Audio**: Sharp frost crack (square wave sweep down)
- **Strategy**: Emergency crowd control when surrounded

#### R: 爆破咒 / Confringo (Blasting Curse)
- **Effect**: Marks a target point (nearest enemy position). After 0.5s delay,
  explodes dealing 60 damage in a 120px radius. If no enemy in range, target
  is 80px in front of Hermione's facing direction.
- **Cooldown**: 8 seconds
- **Visual**: Red-orange expanding ring with explosion particles, screen shake
- **Audio**: Loud explosion (sawtooth with fast decay)
- **Strategy**: Burst damage for clusters, highest single-hit damage

#### F: 漂浮咒 / Wingardium Leviosa (Levitation Charm)
- **Effect**: Lifts the nearest enemy within 300px into the air for 2 seconds.
  The lifted enemy cannot move or attack. At the end of 2 seconds, the enemy
  is slammed down for 35 damage. If no enemy in range, F does NOT trigger
  cooldown (same behavior as Level 1 F).
- **Cooldown**: 12 seconds
- **Visual**: Enemy rises up, swirly white magic aura, then slams down with dust particles
- **Audio**: Rising whistle (sine glide up), then thud (low freq burst)
- **Strategy**: Remove a priority target temporarily, setup for R explosion

### 3.3 Enemy Types

#### Dark Wizard (黑巫师) — NEW
| Attribute | Value |
|-----------|-------|
| HP | 80 |
| Max HP | 80 |
| Speed | 45 px/s |
| Radius | 14 |
| Contact damage | 6/sec |
| Projectile damage | 4 (6 × 0.67) |
| Projectile speed | 250 px/s |
| Shoot range | 400 px |
| Behavior | **Ranged attacker** — stops at 250-350px from player and fires. Only approaches gate when player is far. Retreats slightly if player gets too close (< 150px). |
| Visual | Black hooded robe, pale green wand tip, green projectiles |

**Key difference from Level 1 enemies**: The Dark Wizard is a ranged combatant.
He does NOT rush into melee range. He stops at range and fires projectiles, 
only advancing toward the gate when the player is distant. This forces the 
player to close distance or use spells to deal with them.

#### Hellhound (地狱犬) — NEW
| Attribute | Value |
|-----------|-------|
| HP | 200 |
| Max HP | 200 |
| Speed | 90 px/s |
| Radius | 18 |
| Contact damage | 18/sec |
| Projectile damage | N/A (no ranged attack) |
| Behavior | **Fast melee charger** — runs directly at player in a straight line. On death, leaves a **fire pool** (40px radius, 15 dmg/sec, 3 second duration) that damages the player. |
| Visual | Dark fur, three glowing red eyes, fiery trail particles when moving |
| Death effect | Spawns fire pool at death location |

#### Troll (巨怪) — BOSS, NEW
| Attribute | Value |
|-----------|-------|
| HP | 800 |
| Max HP | 800 |
| Speed | 25 px/s |
| Radius | 32 |
| Contact damage | 35/sec |
| Projectile damage | N/A |
| Behavior | **Slow heavy melee** — walks toward player. Every 4 seconds, performs a **ground slam**: deals 20 damage in 150px AoE around the troll, with 0.8s windup (telegraphed by shaking + red flash). After the slam, the troll is stunned for 1 second. |
| Visual | Huge grey-brown body, wooden club, small head, ground cracks on slam |
| Audio | Heavy footsteps (low freq pulses), ground slam (noise burst + screen shake) |

### 3.4 Wave Configuration

| Wave | Name | Dark Wizards | Hellhounds | Boss | Spawn Interval |
|------|------|-------------|------------|------|----------------|
| 1 | 第 1 波 | 8 | 0 | — | 1.2s |
| 2 | 第 2 波 | 8 | 3 | — | 1.0s |
| 3 | 第 3 波 | 5 | 6 | — | 0.7s |
| 4 | BOSS波 | 0 | 0 | 巨怪 (troll) | 2.5s |

**Wave 1**: Introduction — 8 dark wizards, spaced 1.2s apart. Familiarize with ranged enemies.
**Wave 2**: Mixed — dark wizards provide covering fire while hellhounds charge. Player must prioritize.
**Wave 3**: Hellhound swarm — mostly fast chargers, few ranged support. Hellhound death fire pools create battlefield hazards.
**Wave 4**: Troll boss solo fight — dodge ground slams, exploit post-slam stun window.

**Total enemies**: 8 + 11 + 11 + 1 = 31 (Level 1 had 12+14+14+1 = 41, but Level 2 enemies are individually more dangerous)

### 3.5 Map: Quidditch Pitch

| Attribute | Value |
|-----------|-------|
| Size | 960 × 640 px (same as Level 1) |
| Time of day | Daytime (sunny, blue sky) |
| Ground | Green grass with white pitch markings |
| Decorations | Goal hoops (3 on each side, silhouettes), stadium stands shadow at edges, broom shed in top-left corner |
| Defend target | Pitch entrance at bottom-center (same GATE position as Level 1) |
| Theme | Open field — clear visibility, no obstacles, emphasizes positioning |

### 3.6 Story Context

**Setting**: Immediately after Level 1. Harry's defense of the main gate 
succeeded, but Death Eaters have circled around to the Quidditch pitch.

**Opening Dialogue** (5 lines, Hermione + Neville cameo):

1. **Hermione**: "I was just reviewing my Charms notes... what's that noise?" 📖
2. **Neville** (runs in): "Hermione! Death Eaters — they're coming through the Quidditch pitch!" 😨
3. **Hermione**: "Harry's at the main gate. It's up to us. Neville, warn the others!" 🧠
4. **Neville**: "Be careful! I'll fetch help!" (runs off) 🏃
5. **Hermione** (to herself): "Right then. Wingardium Leviosa, Immobulus... I've studied for this." 💪

**Victory message**: "魁地奇球场守住了！赫敏的智慧战胜了黑暗！"

### 3.7 Difficulty Progression

| Metric | Level 1 (Harry) | Level 2 (Hermione) | Change |
|--------|----------------|-------------------|--------|
| Player HP | 100 | 90 | -10% |
| Auto-attack DPS | 20 (7/0.35s) | 20 (6/0.30s) | Same DPS, faster fire rate |
| Spell style | Burst damage | Tactical control | Playstyle shift |
| New threat | — | Ranged enemies | Forces positioning |
| New threat | — | Death hazards | Battlefield awareness |
| Boss mechanic | Passive (spider) | Active (ground slam AoE) | Dodge pattern |

## 4. Formulas

### 4.1 Damage Formulas

| Source | Formula | Value |
|--------|---------|-------|
| Hermione auto-attack | `6` per bolt | 6 |
| Q Bluebell Flame DPS | `20` per second in radius | 20/s × 5s = 100 max |
| Q Slow | `speed × 0.7` | 30% reduction |
| R Confringo | `60` in 120px radius | 60 (AoE) |
| F Leviosa slam | `35` on landing | 35 |
| Dark Wizard contact | `6 * dt` | 6/s |
| Dark Wizard projectile | `6 * 0.67` | 4 |
| Hellhound contact | `18 * dt` | 18/s |
| Hellhound death fire pool | `15 * dt` | 15/s (3s duration) |
| Troll contact | `35 * dt` | 35/s |
| Troll ground slam | `20` (AoE 150px) | 20 |

### 4.2 Cooldown Formulas

| Spell | Cooldown | Uptime |
|-------|----------|--------|
| Q Bluebell Flame | 10s | 5s active / 10s CD = 50% |
| E Immobulus | 9s | Instant / 9s |
| R Confringo | 8s | Instant / 8s |
| F Leviosa | 12s | 2s active / 12s = 17% |

### 4.3 Enemy Speed ≫ Distance Formulas

| Enemy | Speed | Gate distance (~530px) | Time to gate |
|-------|-------|----------------------|-------------|
| Dark Wizard | 45 px/s | 530 | ~11.8s |
| Hellhound | 90 px/s | 530 | ~5.9s |
| Troll | 25 px/s | 530 | ~21.2s |

### 4.4 Balance Check: Kill Time vs Death Time

**Hermione kills Dark Wizard** (80 HP):
- Auto-attack DPS: 20 → 4.0s
- R one-shot: instant
- Q + auto: 80/20 = 4s if standing in fire

**Hermione kills Hellhound** (200 HP):
- Auto-attack DPS: 20 → 10s
- R + auto: (200-60)/20 = 7s
- Q full duration + auto: 100 + 5×20 = 200 → 5s

**Hermione kills Troll** (800 HP):
- Sustained combo (Q + R rotation + auto): ~40-50s
- Must dodge slams while DPSing

**Dark Wizard kills Hermione** (90 HP):
- Projectile DPS: fires every ~2.5s for 4 dmg = 1.6 DPS → ~56s
- Contact: not a factor (ranged enemy)
- Combined with 3 dark wizards: ~19s to death

**Hellhound kills Hermione** (90 HP):
- Contact: 18/s → ~5s if continuously overlapped
- Fire pool: 15/s additional → ~2.7s if standing in fire

**Troll kills Hermione** (90 HP):
- Contact: 35/s → ~2.6s
- Slam: 20 → ~4 hits to die
- Combined: very lethal up close, must stay at range

**Verdict**: Level 2 is harder than Level 1 by design. The ranged enemies and 
death hazards create new threats. Hermione's lower HP (90 vs 100) means more 
careful positioning is required. Her control spells (Q zone, E freeze, F lift) 
compensate.

## 5. Edge Cases

| # | Case | Handling |
|---|------|----------|
| 1 | Q fire placed on map edge | Fire circle clamped visually, damage radius still applies within bounds |
| 2 | R explodes with 0 enemies in range | Falls back to position 80px in front of Hermione's facing direction |
| 3 | F cast with no enemy in 300px | No cooldown consumed, spell does not fire (identical to Level 1 F behavior) |
| 4 | Dark wizard retreats into map edge | Clamped to bounds, continues firing behavior |
| 5 | Hellhound dies on map edge | Fire pool clamped within bounds |
| 6 | Multiple hellhounds die in same spot | Fire pools overlap (damage does NOT stack — player takes max(15, ...) in overlapping zones) |
| 7 | Troll slam during stun/freeze | Slam timer pauses if troll is disabled (stun/frozen check before slam) |
| 8 | Troll slam hits player through other enemies | AoE ignores other enemies, only damages player |
| 9 | Hermione's Q + Harry's R comparison | Q is DoT zone (100 max dmg over 5s) vs R instant burst (28×3=84 over 3s). Different use cases. |
| 10 | F lifted enemy in fire pool | Still takes fire damage while lifted (damage sources are independent) |
| 11 | Enemy frozen mid-charge (hellhound) | Freeze stops movement immediately; after thaw resumes normal behavior |
| 12 | R explosion kills caster (dark wizard at range) | Dark wizard projectile mid-flight still exists — projectiles persist after caster death |

## 6. Dependencies

| System | Dependency | Status |
|--------|-----------|--------|
| Canvas 2D | Browser (Chrome/Firefox/Edge) | ✅ Same as Level 1 |
| Web Audio API | Procedural sound generation | ✅ Same as Level 1 |
| Keyboard input | WASD + QERF + Arrow keys | ✅ Same as Level 1 |
| Level config system | Needs refactor from hardcoded to data-driven | 🔧 Required for implementation |
| Level selection screen | New UI component | 🔧 Required for implementation |
| Enemy AI | Extended for ranged + charger + boss behaviors | 🔧 Required for implementation |
| Particle system | Existing particle system | ✅ Reusable |
| Dialogue system | Existing dialogue system | ✅ Reusable |
| HUD / cooldown display | Existing HUD | ✅ Reusable with spell icon changes |

## 7. Tuning Knobs

### Hermione Stats (9 knobs)
| # | Parameter | Default | Range | Notes |
|---|-----------|---------|-------|-------|
| 1 | player.maxHp | 90 | 70-120 | Hermione durability |
| 2 | player.speed | 220 | 180-260 | Movement speed |
| 3 | player.radius | 16 | 14-20 | Hitbox size |
| 4 | autoAttack.damage | 6 | 4-8 | Per-bolt damage |
| 5 | autoAttack.range | 380 | 300-450 | Acquisition range |
| 6 | autoAttack.cooldown | 0.30 | 0.20-0.50 | Fire rate |
| 7 | autoAttack.projectileSpeed | 650 | 500-800 | Bolt velocity |

### Spell Tuning (16 knobs)
| # | Parameter | Default | Range | Notes |
|---|-----------|---------|-------|-------|
| 8 | Q cooldown | 10 | 7-14 | Bluebell Flame CD |
| 9 | Q duration | 5.0 | 3-7 | Fire patch lifetime |
| 10 | Q dps | 20 | 12-30 | Fire damage per second |
| 11 | Q radius | 50 | 35-70 | Fire patch size |
| 12 | Q slow pct | 0.30 | 0.15-0.50 | Movement slow |
| 13 | E cooldown | 9 | 7-12 | Immobulus CD |
| 14 | E freeze duration | 1.5 | 1.0-2.5 | Freeze time |
| 15 | E cone range | 280 | 220-350 | Cone reach |
| 16 | E cone angle | PI/6 (30°) | PI/8 to PI/4 | Cone width |
| 17 | R cooldown | 8 | 6-11 | Confringo CD |
| 18 | R damage | 60 | 40-85 | Explosion damage |
| 19 | R radius | 120 | 80-160 | Explosion AoE |
| 20 | R delay | 0.5 | 0.2-1.0 | Explosion fuse |
| 21 | F cooldown | 12 | 9-16 | Leviosa CD |
| 22 | F lift duration | 2.0 | 1.5-3.0 | Air time |
| 23 | F slam damage | 35 | 20-50 | Landing damage |

### Enemy Tuning (15 knobs)
| # | Parameter | Default | Range | Notes |
|---|-----------|---------|-------|-------|
| 24 | Dark wizard HP | 80 | 50-120 | Ranged enemy durability |
| 25 | Dark wizard speed | 45 | 30-65 | Approach speed |
| 26 | Dark wizard contact dmg | 6 | 3-10 | Per-second contact |
| 27 | Dark wizard projectile dmg | 4 | 2-8 | Per-hit ranged |
| 28 | Dark wizard shoot range | 400 | 300-500 | Engagement distance |
| 29 | Hellhound HP | 200 | 120-300 | Charger durability |
| 30 | Hellhound speed | 90 | 60-120 | Charge speed |
| 31 | Hellhound contact dmg | 18 | 10-28 | Per-second contact |
| 32 | Hellhound death fire dmg | 15 | 8-25 | Fire pool DPS |
| 33 | Hellhound death fire radius | 40 | 25-60 | Fire pool size |
| 34 | Hellhound death fire duration | 3.0 | 2-5 | Fire pool lifetime |
| 35 | Troll HP | 800 | 500-1200 | Boss durability |
| 36 | Troll contact dmg | 35 | 20-50 | Per-second contact |
| 37 | Troll slam dmg | 20 | 10-35 | AoE damage |
| 38 | Troll slam AoE | 150 | 100-200 | Slam radius |

### Wave Tuning (4 knobs)
| # | Parameter | Default | Range | Notes |
|---|-----------|---------|-------|-------|
| 39 | Wave 1: dark wizards | 8 | 5-12 | Intro wave count |
| 40 | Wave 2: dark wizards / hellhounds | 8 / 3 | Vary | Mixed wave |
| 41 | Wave 3: dark wizards / hellhounds | 5 / 6 | Vary | Hellhound heavy |
| 42 | Wave 4: boss type | troll | — | Boss identity |

## 8. Acceptance Criteria

### Gameplay
- [ ] Hermione spawns with correct stats (90 HP, 220 speed, 16 radius)
- [ ] Q Bluebell Flame: fire patch appears, persists 5s, damages enemies 20/s, slows 30%
- [ ] E Immobulus: cone freezes enemies in range for 1.5s, 9s cooldown
- [ ] R Confringo: 0.5s delay, 60 damage in 120px radius, 8s cooldown, screen shake
- [ ] F Leviosa: lifts enemy 2s, slams for 35 damage, no CD if no target
- [ ] Auto-attack: 6 damage at 0.30s rate, 380px range
- [ ] Dark Wizard: stops at range, fires green projectiles, retreats when player approaches
- [ ] Hellhound: charges player, leaves fire pool (40px, 15 dmg/s, 3s) on death
- [ ] Troll: slow advance, ground slam every 4s (150px AoE, 20 dmg, 0.8s windup), stunned 1s after
- [ ] All 4 waves spawn correctly with correct enemy types and counts
- [ ] Victory screen displays correct message
- [ ] Death screen displays game over

### Technical
- [ ] Game supports multiple levels via data-driven config (not hardcoded)
- [ ] Level selection screen works: player can choose Level 1 or Level 2
- [ ] Level 1 still plays correctly after refactor (no regressions)
- [ ] All 5 spell cooldown HUD icons display correctly for Hermione's spells
- [ ] Dialogue displays correctly with new Level 2 script
- [ ] Procedural audio plays for all 4 new spells + new enemy sounds
- [ ] Test runner covers new damage formulas

### Visual
- [ ] Quidditch pitch map renders: green grass, goal hoops, daytime sky
- [ ] Hermione character sprite is visually distinct from Harry
- [ ] New enemy sprites (dark wizard, hellhound, troll) are visually distinct
- [ ] Spell VFX for all 4 Hermione spells (blue fire, frost cone, explosion, levitation)
