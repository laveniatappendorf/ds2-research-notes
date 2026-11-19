# Shrine of Amana (m20_11): vanilla v1.00 vs vanilla 1.10 vs PC Scholar 1.0.3.0 (2026-10-24)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author): what differs in the Shrine of Amana between vanilla 1.00, vanilla 1.10 and the current PC Scholar:
how many enemies, weapons, spells and other items, and how they changed (for example, "the PC Scholar spellcasters
track less").

"1.10" is the last vanilla PS3 patch, as in [bosses: vanilla vs PC Scholar](../mechanics/bosses-pc.md). Its regulation is PS3 Scholar's byte for byte, and for this
map PS3 Scholar's generators and pickups are also identical to 1.10's (checked by the tool). So "1.10" also stands for
PS3 Scholar here.

Tool: `..\tools\map_compare_pc.py [map]` (works for any map; run with `uv run --no-project --python 3.12 --with openpyxl
python -I -X utf8`, ~60 s). Outputs:
- `map_m20_11_00_00.txt`: the full report;
- `map_m20_11_00_00_generators.tsv`: every generator in the three builds (enemy, position, spawn limits, AI, aggro group,
  leash, item lots);
- `map_m20_11_00_00_items.tsv`: every pickup;
- `map_m20_11_00_00_fields.tsv`: every changed field of the enemy rows, with the patch that made each vanilla change.

The chains are those of `bosses_pc.py` (enemy rows decoded in the PC layout) and `placements_pc.py` (generators,
pickups). Bullets also follow their child bullets. Each 1.00 → 1.10 change is dated against the 1.01, 1.02, 1.03, 1.08 and
1.09 regulations. Patches 1.04-1.07 were never obtainable, so "(1.08)" means "somewhere in 1.04-1.08".

## Short answer

- **Vanilla 1.00 → 1.10: nothing moved.** The map still has 102 generators and 34 pickups, the same enemies in the same
  places. Only the numbers changed:
  - the shrine maidens' homing orb was nerfed in 1.03 and retuned again by 1.08;
  - the maidens' spells hit about 9 % harder from 1.08;
  - poise damage of the Aberrations' and Ogres' attacks [went up ×1.8 (1.02)](1.01-to-1.03.md#101-102-a-balance-patch-calibrations-102-103);
  - three Demon of Song attacks were cut (1.01);
  - co-op time per kill went down (1.10);
  - most enemies' close-range perception value (`detect_distance`) went up, game-wide (1.10), but not the maidens';
  - two beetle attacks and the Ogres' projectile attacks lost their bullet rows (1.04-1.08).
- **1.10 → PC: the placements were reworked.**
  - [Generators 102 → 131](placements-balance-pc.md#1-enemies): 43 added, 14 removed, 30 moved more than 1 m.
  - More maidens (14 → 18), clerics (12 → 16) and Aberrations (48 → 52). New kinds: 4 Basilisks, 3 Pyromancers,
    3 Poison Moths, a Dragonrider, an Old Knight, a Syan Knight, a Forlorn, Lone Hunter Schmidt.
  - The one Prowling Magus is gone.
  - Items barely changed: 2 new pickups (Smooth & Silky Stone; Human Effigy ×2). The weapons, armour, ring and spells are
    the same in all three builds.
- **The spellcasters' tracking did not change between 1.10 and PC.** Every field of the maidens' bullets, their turn
  rates and their attack turning is identical in 1.10 and PC. The tracking cut is a **vanilla-patch** change (1.03,
  then 1.04-1.08), so PS3 and PC Scholar both inherit it. Compared with v1.00 it is real. Compared with 1.10 it is not a
  Scholar change.
  - One thing the data can't rule out is the PC's 60 fps (the PS3 runs at 30), if homing is applied per frame. That
    needs a live test.
- **The maidens' sight range is shorter on PC: 40 m → 30 m.** This is the distance at which they notice you, and so
  start casting. It was 40 m in every vanilla patch and in PS3 Scholar. The PC cut touches only the two maiden
  perception rows (section 2).
- **The sorceress's three-shot volley is never cast**, in any build. It is fully set up (bullets, damage, animation),
  but no AI script ever asks for it (section 2).
- The PC also changed the **healer's** behaviour script: some modes are re-routed, and its approach routine's 15 m /
  30 m thresholds became 200 m. The sorceress's combat, casting and movement routines are identical in 1.10 and PC.

## 1. Enemies: how many of each

Generators per kind (names from the community SOTFS sheet; Japanese labels translated):

| enemy | v1.00 | v1.10 | PC |
|---|---:|---:|---:|
| Amana Aberration | 48 | 48 | **52** |
| Amana Shrine Maiden (sorceress, 105010) | 14 | 14 | **18** |
| Amana Shrine Maiden (healer, 105000) | 2 | 2 | **3** |
| Lindelt Cleric | 12 | 12 | **16** |
| Acid Horn Beetle | 10 | 10 | **9** |
| Giant Acid Horn Beetle | 1 | 1 | 1 |
| Milfanito (2 singers + 2 helper entities with 10 HP) | 4 | 4 | 4 |
| Undead Supplicant | 3 | 3 | 3 |
| Ogre | 2 | 2 | 2 |
| Demon of Song (boss) | 1 | 1 | 1 |
| Butcher phantom (NG+) | 1 | 1 | 1 |
| Felicia the Brave (summon) | 1 | 1 | 1 |
| Peculiar Kindalur (NPC invader) | 1 | 1 | 1 |
| 304010, AI label バンシー（亡者） "banshee (hollow)": 10 HP, no souls | 1 | 1 | 1 |
| Prowling Magus variant 137022 (AI label "senior necromancer (immortal)") | 1 | 1 | **0** |
| Basilisk | 0 | 0 | **4** |
| Pyromancer (human NPC) | 0 | 0 | **3** |
| Poison Moth | 0 | 0 | **3** |
| unequipped human 696000 | 0 | 0 | **2** |
| Dragonrider (halberd) | 0 | 0 | **1** |
| Old Knight (great hammer) | 0 | 0 | **1** |
| Syan Knight (spear) | 0 | 0 | **1** |
| Forlorn (scythe) | 0 | 0 | **1** |
| Lone Hunter Schmidt (shade summon sign; "invader" corrected 2026-11-04) | 0 | 0 | **1** |
| Undead Traveler (unarmed, a statue) | 0 | 0 | **1** |
| **total generators** | **102** | **102** | **131** |

A generator is a spawn point, not an enemy you always meet: the Butcher only appears in NG+, and the invaders and the
summon depend on online / event conditions.

**1.00 → 1.10**: no generator added, removed, moved or given another enemy. One unidentified flag
(GeneratorParam `Unk48_7`) went 0 → 1 on 91 of the 102 generators in 1.10 (not in 1.01-1.09): every ordinary enemy
except the Giant Acid Horn Beetle and the Prowling Magus, and none of the NPCs, the boss or the singers. PC keeps it, adds
it to the giant beetle and sets it on most new enemies (114 of 131). Its meaning is not decoded (see [GeneratorParam fields](../modding-world/generatorparam-fields.md#unk48-bit-7-the-exemption) for a later reading from the spawn check).

**1.10 → PC**:
- **added (43)**: Aberrations 14, maidens 6, clerics 4, Basilisks 4, Pyromancers 3, Poison Moths 3, unequipped humans 2,
  and one each of Schmidt, the Forlorn, a healer maiden, a statue traveller, the Syan Knight, the Old Knight and the
  Dragonrider;
- **removed (14)**: Aberrations 10, maidens 2, a beetle, the Prowling Magus;
- **moved (30)**: mostly maidens, Aberrations and clerics, by 2-32 m. Felicia's summon spot moved 109 m and Kindalur's
  spawn 38.6 m. Kindalur also has a new logic script (820010 → 820024) and a longer leash (100 → 200 m);
- **settings**:
  - 5 clerics have new AI think ids (10 → 101 ×4, 10 → 230 ×1), and the cleric AI (ai131000) gained three modes
    (groups 100, 101, 150) to match;
  - 27 generators (24 existing, 3 new) are linked to a new event region 53000000, and 5 clerics to new regions
    52001000-52001101 (what these events do is not decoded);
  - the Demon of Song's aggro group changed 150 → 240;
  - Felicia's spawn limit changed 0 → 255 (as for the other PC summons / invaders);
  - render groups (DrawGroup) changed on 67 generators;
  - the enemy rows' respawn limit (EnemyParam `spawn_limit`) is unchanged.

## 2. The spellcasters: Amana Shrine Maidens

The sorceress (105010) has two spells, both cast from attack group 3 of its behaviour row:
- **a slow homing orb**: bullet 110501210, lives 10 s, starts at 5 m/s, capped at 10 m/s;
- **a three-shot volley**: bullets 110501310 / 320 / 330, faster (8 → 15-30 m/s) and short-lived (3-5 s). **Never
  cast** in any build; see "The three-shot volley" below.

Through every patch on disk:

| homing orb 110501210 | 1.00-1.02 | 1.03 | 1.08 - 1.10 | PC |
|---|---|---|---|---|
| `max_tracking_angle_change` (SmithBox name) | 45° | **30°** | **90°** | 90° |
| `max_tracking_time` (SmithBox name) | 60° | 60° | **50°** | 50° |
| `Unk40` (unnamed) | 0 | **8** | **1.9** | 1.9 |
| damage | 498 | 498 | **544** | 544 |
| volley damage (each of 3; never cast) | 288 | 288 | **314** | 314 |

No other field of the four bullets changed in any build (speeds, lifetimes, lock-on angles, shot counts).

**What these fields mean** is not documented, and the SmithBox names don't fit the values:
- The two "tracking" fields hold angles, not times. On the player's spells they read like the horizontal and vertical
  cones inside which a cast aims at the locked target: Soul Arrow 60° / 20°, Soul Spear 90° / 60°, Fireball 180° / 50°,
  Poison Mist 360° / 360°.
- `Unk40` is set on only 35 of the PC's 860 enemy bullets (no player spell uses it). All of them are slow, long-lived
  orbs, and the value (1-6 s) is always shorter than the bullet's life. That fits **how long the bullet keeps homing**.
- If that reading is right, the orb:
  - homed for its whole 10 s in 1.00-1.02;
  - homed for 8 s, with a narrower 30° aim cone, in 1.03;
  - **has homed for only its first 1.9 s since 1.08** (a wider 90° aim cone, slightly lower vertical cone).

  The volley never had a homing time.
- The official 1.03 notes (machine-translated in English coverage) say "female mages in Shrine of Amana receiving nerf
  in both damage and their spell's tracking ability". The tracking half matches the 1.03 data. The damage half does
  not: 1.03 only carried the game-wide durability-damage cut (5 → 4), and by 1.08 the maidens' spell damage was 9 %
  higher.
- Not verified live.

**Unchanged in all three builds**: the maidens' HP (sorceress 700, healer 520), defences, souls (380), movement and turn
rates (EnemyMoveParam), turning during attacks (EnemyAttackMoveParam), attack rows (EnemyBehaviorParam and the
Second / Third / Combo rows: cooldowns, speeds) and animations (TAE). Sight changed on PC; see below.

**Other changes:**
- **NG+ table** (1.02): the maidens moved from NG+ table 9 to 19. Damage multipliers are the same; stamina damage on a
  guard is higher in NG+ (NG+1 162 → 200 %, NG+7 400 → 600 %).
- **Co-op time per kill**: 90 → 60 s (1.10) → 36 s (PC).
- **The healer's heal / blast** (damage row 110500110): hitbox radius ×15 → ×18 on PC (+20 %).
- **AI, 1.10 → PC**: the logic script (ai105010) is unchanged. The behaviour script ai105000, which both maiden kinds
  use, has one dispatcher per kind:
  - group 20 is the **sorceress's**: it is the only group that asks for attack 20, the orb, and only the sorceress's
    attack row has the orb;
  - group 10 is the **healer's**: its one spell is attack 11, the heal / blast (bullet 110500110).

  The PC changes are almost all in the healer's group:
  - The modes the logic picks as 12 and 14 now run a routine that is new in the script, called with one argument
    (1.10: another routine, called with two). Mode 17 now runs mode 13's routine (with an extra argument) instead of
    its own.
  - Its approach routine (mode 10) is now called with 200 / 200 instead of 15 / 30. Those arguments are the distances
    (f7001, very likely the distance to the target) at which it walks or runs towards you, so on PC those branches
    practically never fire. A new branch in the same routine (distance ≥ 200 and f7010 ≥ 5) can't fire either. Read
    structurally, the healer no longer closes in on you from 15 m or more. It still backs off inside 10 m.

  The sorceress's group only gained the same extra argument on mode 13's routine. Her combat routine, the cast routine
  and her movement routine are line-for-line the same in 1.10 and PC. The script's functions have no names, so the
  effect in play is a structural reading. Nothing here touches how the spells aim or home.

**Sight range (how far away they notice you), PC only.** The maidens' perception comes from LogicComParam rows 105010 and
105030. The game looks the row up by the spawn record's logic script id (`LogicParamID` 105010, or 105030 for generator
4000; `ChrAi::Init`, PC 0x140425510). The enemy rows' own link, 13000, points at a row that doesn't exist in any build,
which is why an earlier pass of this note called their sight unchanged.

| LogicComParam 105010 / 105030 | 1.00 - 1.10 | PS3 Scholar | PC |
|---|---:|---:|---:|
| `Unk04`: sight range when the enemy stands in light | 40 m | 40 m | **30 m** |
| `sight_distance`: sight range in darkness | 40 m | 40 m | **30 m** |
| `Unk68`: online sync radius 30 → 70 m | 0 | 0 | **1** |
| `detect_distance` (1 m), vision cone 60° × 45° | same | same | same |

**How the game uses these fields** (PC exe, static; sight test `FUN_14043BEF0`, checked in the disassembly):
1. **`detect_distance`**: closer than this, the enemy notices you at once, from any direction: no cone, no light
   check. Raising it (1.10, below) makes sneaking up from behind, e.g. for a backstab, a little harder.
2. **Range by light**: `Unk04` applies when the enemy stands in light, `sight_distance` in darkness (LogicComParam
   `Unk32` = 0, as on 328 of 347 PC rows and the maidens'; 1 swaps them, 2 always uses `Unk04`). Light is a per-character
   flag set by bright-location regions, the floor collision's brightness (map parts start lit; map events switch groups
   to dark) and by a torch-bearer within 27 m. So `Unk04` is the normal range and `sight_distance` the dark range:
   the Lindelt Clerics, for example, see 30 m lit and 16 m dark.
3. **Torch rule**: an enemy in the dark with a range under 27 m sees out to 27 m if you carry a lit torch.
4. **Cone**: within the range, a pure angle test (horizontal / vertical angle). No line-of-sight raycast in this test.
5. **`Unk68`** is network code only. It sets a bit at character setup that `NetEnemyManager` reads: the player's
   distance to the enemy is compared with 30 m, or 70 m with the bit set, to decide which player in a co-op session
   takes or keeps the enemy's network control. It does nothing offline. Rows that have it from 1.10: Freja, the Grave
   Wardens, the Pursuer, Flame Lizards, the Brume Tower Iron Warriors, the Frozen Reindeer and the statue clusters.
   PC adds it to the maidens, the Guardian Dragon, the Executioner's Chariot (new PC rows) and several PC NPC phantoms.
   For a ranged enemy whose sight is 30 m it fits: the PC keeps her synced while she can still shoot at a co-op
   partner from further away.

Confidence: high on 1 and the 30 / 70 m comparison; medium on "lit is the default everywhere" and on what network
control means exactly.

- Only these two rows changed out of the 333 the PC shares with 1.10, so this targets the maidens rather than being a
  game-wide perception pass. PS3 Scholar is identical to 1.10, so the cut came with a PC version. Only the current PC
  regulation (1.0.3.0) is on disk, so which PC patch made it isn't known.
- The script doesn't check distance before the orb: the sorceress casts it whenever it is off cooldown (3-5 s) and she
  is facing you within 20°. So the range at which she starts casting is the range at which she notices you, cut
  by a quarter on PC. The orb itself is unchanged and can travel well beyond 30 m (up to 10 s at up to 10 m/s).
- Not verified live: a sorceress 31-39 m away should ignore you on PC and fire at you on PS3.

**The three-shot volley is configured but never cast**, in 1.00, 1.10 and PC:
- It is all there: the sorceress's attack row (EnemyBehaviorThirdParam 105010) has it as attack 21, with three
  bullets (110501310 / 320 / 330), three damage rows and a 2-6 s cooldown. The animation `Attack_21_Shot` in
  c105010's TAE spawns the three bullets at 4.10 s.
- Nothing asks for it. Attacks are started by command 5204 in the behaviour script. Tracing every value that can reach
  it, through all routine calls, gives only attacks 10, 11 and 20 (plus 30 and 31) in ai105000, in all three builds.
  The logic scripts ai105010 / ai105030 never start an attack. The only combo row (EnemyComboBehaviorParam 105000 /
  105010) chains attack 11 only.
- So the volley's damage (288 → 314 in 1.08) is a param value with no effect in play. Wikis and sheets that list
  "volley damage" read it from the param table.
- **It was used in the 2013 Network Test's script.** The NT's version of the combat routine rolls 0-20: below 10 casts
  the orb (20), otherwise the volley (21), about half each, with a timer between casts. Retail replaced it with the
  orb-only routine. The maidens weren't placed in the NT, so this is the script, not something seen in play. More cut
  attacks like it are in [unused enemy attacks](../mechanics/unused-attacks-pc.md).

**Could it be restored? Yes: only the AI's request is missing** (checked 2026-10-25, PC files):
- **Animation network** (`c1050.nmb`, which both maiden kinds run): `SM_Attack` sends attack categories 20-29 to the
  shot state machine (node 588). There, `AttackCategory_Enemy` = 20 enters `Attack_20_Shot` (the orb) and = 21 enters
  `Attack_21_Shot` (the volley), three transits each. The volley is wired exactly like the orb.
- **Animation events** (c105010 TAE): the orb charges for 3.3 s (cast effect 5350) and fires its one bullet at 3.30 s.
  The volley charges for 4.1 s (same effect, variant 2) and fires bullet slots 0, 1 and 2 together at 4.10 s.
- **Params**: attack 21 = EnemyBehaviorThirdParam 105010 group 2 (bullets, damage, 2-6 s cooldown), as above.
- **The script**: the sorceress's combat routine (PC group 2147483612, state 2) casts the orb when `f5091(20) != 0`
  (attack 20 ready), and state 5 starts it with `6:2147483622(20, 20, 90, 5, 1, 0, 3, 0)` (attack id 20, turn
  until within 20°, give up after 5 s).
  - **Quick test**: change the two `20`s to `21` (PC `ai105000.esd` file offsets 0x1fdc3 and 0x21d80, byte 0x54 →
    0x55). She then casts the volley instead of the orb, on the volley's own cooldown. Same two bytes in 1.10 at
    0xfd1b and 0x11c0b. This can be done in memory, live, without touching the files.
  - **Tested live on PC, 2026-10-25: it works.** With `pcmem.py volley on` (one loaded copy of the script, both bytes
    written and re-read), a sorceress at Amana cast the volley. The author saw three projectiles at once, leaving in
    different directions and converging on the player. So the animation, the shot events, the three bullets and the
    network route all work as shipped; only the AI request was missing. `volley off` puts the orb back.
  - **It "hits off" visually** (the author). The data shows why that may be: the cast fires **five** projectiles, not three.
    - Bullet slot 0 (110501310) is the orb effect (5000), fired three times in a ±25° fan, with a cast and a hit
      effect. Those are the three that were seen.
    - Slots 1 and 2 (110501320 / 330) are two more bullets at ±30°: faster (up to 30 m/s), homing for about 1 s, with
      effect 1300 and **no cast or hit effect**. No other bullet in the game uses effect 1300, and whether that effect
      exists or shows anything is not checked.
    - If those two are invisible or don't match their visuals, you can be hit by projectiles you can't see. A hitbox /
      bullet view in game would settle it. Possibly part of why the volley was cut.
  - **Confirmed with DebugManager's hitbox view (2026-10-25, PC debug copy):** five damage spheres, the two invisible
    ones out ahead of the three visible orbs (they reach 30 m/s, the fan 15).
  - **The volley is unfinished.** The bullet history:

    | bullet | Network Test | 1.00 / 1.10 / PC |
    |---|---|---|
    | orb 110501210 | placeholder effect 1300, 30 m/s, 3 s | orb effect 5000 + cast 5375 / hit 5160, 10 m/s, 10 s |
    | volley slot 0, 110501310 | placeholder 1300, one bullet | orb effect, fired 3× in a ±25° fan |
    | volley slots 1-2, 110501320 / 330 | placeholder 1300 at ±30°, hit effect 800020290 | still placeholder 1300, no hit effect |

    - In the Network Test every maiden spell used the same placeholder effect, 1300.
    - By release the orb was finished, and the volley was redesigned around a three-orb fan from slot 0.
    - The two old side bullets were left attached: never given visuals, never removed (and they even lost their NT
      hit effect). Then the AI stopped asking for the attack.
    - So the finished design looks like the three visible orbs. A restore should drop slots 1-2: set
      EnemyBehaviorThirdParam 105010 `group_2_enemy_bullet_id_1/_2` (and damage ids 1-2) to -1.
  - **A real restore** keeps both spells: a second branch in state 2 (e.g. "volley ready and a random roll" → a new
    state calling the same routine with 21). That adds states and bytecode, so it needs [an ezstate writer](../modding-scripts/writing-esd-files.md) (not built
    yet) or appended bytecode with repointed offsets. The roll already exists: state 1 sets slot 0 to 1-100
    (`1:1(0, 1, 100)`), which the melee choice reads as `f1(0)`.
- PS3 networks are 32-bit and not read by `nmb.py`, so the PS3 wiring is assumed from the PC, not checked.

## 3. The other enemies

All values are v1.00 → 1.10 → PC; the patch that made a vanilla change is in brackets.

**Perception (1.10, game-wide).** [Patch 1.10](1.08-to-1.10.md#3a-the-base-game-patch-archive-patchdata-109-110-independent-of-the-read-order) raised LogicComParam `detect_distance` on 122 of 395 rows (3 went down).
On the Amana enemies: Aberration 1 → 1.1, Lindelt Cleric 1 → 1.2, Undead Supplicant 1 → 1.1, Acid Horn Beetle 1 → 1.3,
Giant Acid Horn Beetle 1 → 2.5, Ogre 1 → 1.8, Syan Knight 1 → 1.3, Prowling Magus 1 → 1.2. Felicia and Kindalur went
1 → 0.8. The maidens' rows were not touched, and PC kept all of these. `detect_distance` is the all-round radius in
metres inside which an enemy notices you from any direction (section 2, "How the game uses these fields"), so 1.10 made
most enemies a little harder to approach unseen from behind. Sight distances themselves did not change in 1.10.
(These lines were missing from the first version of this note; see "Sight range" in section 2.)

- **Amana Aberration** (HP 500, 300 souls): 48 → 48 → 52. Poise damage of its two hits 37.5 / 46.9 → 67.5 / 84.4 (1.02);
  co-op time 18 → 12 → 7 s. Drops unchanged (Human Effigy 5 %, etc.).
- **Lindelt Cleric** (HP 760, 550 souls; Archdrake set / mace / shield 2 % each): 12 → 12 → 16. No stat change apart from
  co-op time. PC: new think ids for 5, three new AI modes, and 5 linked to new event regions (section 1).
- **Acid Horn Beetle / Giant Acid Horn Beetle**: 10 → 10 → 9 and 1 → 1 → 1. Unchanged on PC. The 5-shot spray and the
  36-pellet burst bullet rows (120500110 / 210 / 211, and the giant's 122000410 / 411) were dropped in 1.04-1.08, while
  their behaviour rows still name them. Those attack slots either fire nothing or were never used (not checked).
- **Ogre** (HP 2,740, 2,200 souls): 2 = 2 = 2. Poise damage 52.5 → 94.5 (1.02). Its two projectile attack rows
  (130000410 / 411 / 510) were also dropped in 1.04-1.08. PC adds three AI modes, probably for its new placements
  elsewhere.
- **Undead Supplicant** (HP 380): 3 = 3 = 3. PC gives three of its attacks a second damage part of 15 (type code 6, the
  code of the beetles' acid spit) and adds an AI mode.
- **Demon of Song** (HP 6,180): three attacks [cut in 1.01](v1.00-vs-1.01.md#2-balance-regulation-calibrations-100-102-and-effect-scripts) (256 → 224, 496 → 352, 736 → 672). PC changed nothing but its
  aggro group.
- **Milfanito**: unchanged except PC curse defence 30 → 100 on the two real singers.
- **Peculiar Kindalur** (invader): moved, new logic script, leash 200 m. His Priestess Gloves / Skirt and Cleric's Parma
  drops went up 3 → 10 % on PC.
- **Felicia the Brave** (summon): moved 109 m; spawn limit 255.
- **Butcher phantom (NG+)**: only co-op time changed.
- **Prowling Magus variant 137022** (HP 1,220, 2,500 souls; one generator in vanilla, removed on PC): its spells hit
  9 % harder and switched damage type code 1 → 4 (1.08), and poise damage went up (1.02). It dropped the Warlock Mask
  and Cursed Bone Shield (2 %) and Wilted Dusk Herb (10 %).
- **New on PC** (PC values):
  - **Basilisk** ×4: HP 350, 300 souls; drops Raw Stone 5 % and stones;
  - **Pyromancer** ×3: human NPC, HP 1,830, 450 souls. It casts the enemy versions of Great Combustion, Fire Tempest,
    Acid Surge, Great Chaos Fireball and Lingering Flame (their own NPC spell rows 73040000-73140000, not compared here). Its generators drop
    the Bonfire Ascetic (2 %);
  - **Poison Moth** ×3: HP 280, 60 souls; generator lots of lifegems / herbs (35 / 15 %);
  - **Dragonrider (halberd)**: HP 5,730, 2,700 souls; Titanite Slab 10 %, Petrified Dragon Bone 5 %;
  - **Old Knight (great hammer)**: HP 1,600, 700 souls; Old Knight set and hammer;
  - **Syan Knight (spear)**: HP 1,010, 900 souls; this row exists in vanilla but was never placed in Amana;
  - **[Forlorn (scythe)](../mechanics/forlorn-pc.md#2-the-15-invasion-areas)**: HP 3,460, 16,000 souls;
  - **Lone Hunter Schmidt**: HP 11,400. A small white (shade) summon sign, not an invader (corrected 2026-11-04:
    GeneratorParam Unk84 3 and event name 小白霊：白ファントムサイン表示, see [NPC invaders](../mechanics/npc-invaders-pc.md));
  - **unequipped human 696000** ×2: HP 10; drops Smooth & Silky Stone 80 %;
  - **Undead Traveler** ×1: a statue.

## 4. Items

**Pickups** (MSB object → MapObjectInstanceParam → ItemLotParam2_Other): 34 → 34 → 36.
- 1.00 → 1.10: no change.
- 1.10 → PC: **2 added**, both in `o00_0258` containers:
  - 20116210: Smooth & Silky Stone;
  - 20116220: Human Effigy ×2.
- The other 34 have the same place, container and contents in all three builds. Two glowing pickups (20116050,
  20116180) have empty lots in all three.

| item kind (lot entries) | v1.00 | v1.10 | PC | which |
|---|---:|---:|---:|---|
| weapons / shields / catalysts | 3 | 3 | 3 | Red Iron Twinblade, Helix Halberd, Manslayer |
| armour | 5 | 5 | 5 | Singer's Dress; King's Crown / Armor / Gauntlets / Leggings |
| rings | 1 | 1 | 1 | Life Ring+2 |
| spells | 3 | 3 | 3 | Homing Crystal Soulmass, Sunlight Blade, Fire Tempest |
| ammunition | 1 | 1 | 1 | Magic Arrow ×20 |
| goods | 34 | 34 | 36 | Estus Flask Shard, Bonfire Ascetic, Soul of the King, Twinkling Titanite, Petrified Dragon Bone, Divine Blessing, souls, herbs ... (+ Smooth & Silky Stone, Human Effigy ×2 on PC) |

The chest settings (TreasureBoxParam: no trap, no bullets) are identical.

**Enemy drops** (EnemyParam death lot → ItemLotParam2_Chr): identical for every enemy here except Kindalur (above).
Generator lots:
- the Prowling Magus's lot goes with it;
- PC adds the Pyromancers', the moths' and the Dragonrider's (section 3).

**Enemy-carried spells and weapons**: the maidens' Staff of Amana drop (2 %) and the clerics' Archdrake gear (2 %) are
unchanged in all three builds.

## 5. Not checked / open

- The meaning of bullet `Unk40` (read here as homing duration) and of the two "tracking" angles. A live test on PC
  would settle it (watch a maiden orb stop curving after ~1.9 s). The 60 fps question also needs a live test.
- The generator flag `Unk48_7` (new in 1.10), the event regions 53000000 / 52001xxx on PC, and the render groups.
- What the re-routed healer modes and the new cleric modes do in play (the ezstate functions have no names).
- The maidens' 30 m PC sight, live (section 2); [the light rule](../mechanics/earthen-peak-aggro-pc.md#4-light-vs-dark-same-in-both-builds-checked-2026-11-02) (is every floor lit by default?); whether line of sight
  is checked anywhere outside the sight test; `Unk68` needs a co-op session.
- Whether the beetle and Ogre attacks whose bullet rows were dropped are ever used (see [attack swap candidates](../mechanics/attack-swap-candidates.md#near-amana-you-are-there)).
- The NG+ extra enemies: there is no NG+-only flag in the generator data, so which generators are NG+-only (the
  Butcher, perhaps others) is set by events and was not traced (see [NG+ near boss arenas](../modding-world/ng-plus-differences.md#ng-content-is-extra-generator-rows-data)).

## 6. Knocked-down maidens never stand up

Moved to its own note, [knocked-down Amana maidens](../mechanics/maidens-stay-down-pc.md) (2026-10-25). In short: the maidens' face-up fall animation
(`a00_16_0740` in `c1050.nmb`) carries a stray `DownDuration` event. At the cross-fade into the lying loop its end
switches the character's "down" flag off, so the get-up timer (1.5 s) resets after 0.8 s and never sends `SitUp`.
Proved with a live memory watch; maidens knocked onto their face do get up. Not the AI, the params or a missing
animation. The same event is on the Forest Grotesques', Skeletons' and Leydia Pyromancer's
networks, but the Skeleton Lord skeletons (c1320's network) get up, so it isn't sufficient on its own.

Source for the official 1.03 notes: GamingBolt, "Here's What Dark Souls II v1.03 Brings to the Table", 10 Apr 2014
(machine-translated from the Japanese notes).

## Related pages

- [The name Amana](../network-test/shrine-of-amana-name.md): what the Network Test already holds for this map (the maidens' AI script, the Demon of Song's script, Gavlan's dropped lines) and where the name comes from.
