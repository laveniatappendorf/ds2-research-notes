# Unused enemy attacks: set up but never requested by the AI

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

PC stage 3, 2026-10-25. The author asked after the Amana sorceress's three-shot volley turned out to be fully set up but never
cast ([Shrine of Amana across builds](../builds/shrine-of-amana-pc.md#2-the-spellcasters-amana-shrine-maidens), section 2): do other enemies have attacks like that?

Tool: `tools\unused_attacks_pc.py`. Output:
- `unused_attacks_pc.tsv`: every spawn kind (enemy row + its two AI scripts) and every attack it has set up, in v1.00,
  1.10 and PC;
- `unused_attacks_pc_gamewide.tsv`: the PC attacks that no spawn of their enemy ever requests;
- `unused_attacks_pc_midfight.tsv` (since 12 Nov): behaviour rows no spawn points at, judged against their model's
  scripts (see "Mid-fight behaviour rows").

**Re-run 2026-11-12** after the Last Giant's "unused" attack 15 turned out to be part of a combo. Two fixes:
- **Combos now count.** 23 attacks the first scan listed were only ever played inside a combo the AI does use, so they
  are not unused moves. The numbers below are the corrected ones; the 25 Oct figures were 183 attacks on 81 rows
  (104 routed), 21 requested by the Network Test.
- **Mid-fight rows are now judged.** These are rows an enemy switches to during the fight, like the Last Giant's arm
  phase. This found one new case (the Varangian Sailor's 15, below) and nothing else that wasn't already listed.

## Short answer

- **Yes, plenty.** On PC, **160 attacks on 71 enemy behaviour rows** are set up but no spawn of that enemy ever asks for
  them, alone or as part of a combo.
  - **81 of them (46 rows)** still have a working animation path: the enemy's animation network routes the attack id
    to an `Attack_NN` animation. Like the volley, these could be brought back with an AI script edit alone.
  - **79 (36 rows)** are data only: the network has no route for them, so they can't play even if requested.
- **18 of them were requested by the 2013 Network Test's AI scripts**, the clearest evidence of attacks cut between
  the Network Test and release. The sorceress's volley is one of them (see below).
- **Mid-fight rows add one:** the **Varangian Sailor's attack 15** (row 113010). The AI requested it in v1.00 and 1.10,
  and the Network Test did too. Scholar rewrote the Sailors' script (ai113000, 160 KB to 396 KB) and stopped requesting
  it.
- **Patch history:**
  - only one attack lost its request in a vanilla patch: the **Grave Wardens' attack 18**, in 1.10;
  - from 1.10 to PC, only the Varangian Sailor's 15 (on a mid-fight row, so the first scan missed it);
  - one attack went unused through a placement change: the **Captive Undead's shot** (attack 20, bullet 112700310). In
    vanilla, 3 Gutter spawns running script ai337000 used it, and [Scholar's Gutter rework](../builds/placements-balance-pc.md#1-enemies) removed those spawns.

## Method

An enemy row (EnemyParam) points at its behaviour rows (`behavior_param_id` → EnemyBehaviorParam, Second, Third). Each
has 10 groups, and group *g* of each param is attack id *g − 1*, *10 + g − 1* or *20 + g − 1*.
- This is how the animation networks route attacks: `SM_Attack` sends `AttackCategory_Enemy` 0-9, 10-19 and 20-29 to
  three state machines, and those pick the single attack.
- The sorceress confirms it: Third group 1 = attack 20 = the orb, group 2 = attack 21 = the volley.

An attack is **set up** when its group names a damage or bullet row.

An attack is **requested** when its id can reach command `1:5204` (start attack) in the spawn record's behaviour script
(`ai<DefaultLogicParamID>.esd`) or logic script (`ai<LogicParamID>.esd`). That is:
- a literal first argument; or
- a value passed down through any chain of sub-machine calls into the argument that a routine hands to 5204.

Checks on the method:
- **5204 is the only way an AI script starts an attack.** A survey of every command in all 538 PC AI scripts finds no
  other command taking attack ids. No 5204 call has a computed first argument (every value resolves to a literal).
- **Spawn kinds without AI ids are not judged** (38 on PC: mostly NPCs, plus the No-man's Wharf hollows, the Varangian
  Sailors and Velstadt, whose AI is set elsewhere).
- **"Never requested" is game-wide**: an attack counts as used if any spawn of that behaviour row, with any scripts,
  requests it. Requests are collected from every state group of a script, even groups a given enemy doesn't run. So the
  list errs towards *used*: it may miss some unused attacks, not invent them.
- **Network**: c<behaviour row ÷ 100>.nmb, PC only (rule checked in [the maidens note](maidens-stay-down-pc.md#4-other-characters-with-the-same-stray-event)). "Routed" = some state machine has a transit on
  `AttackCategory_Enemy` = that id.
- **Network Test**: whether the NT's script of the same id (`DS2_GAME=nt`) requests the attack. The NT's own spawn
  records were not used; [many of these enemies weren't placed in the NT](../network-test/nt-ai-labels-retail-enemies.md).
- **Combos** (added 12 Nov): a script requests a combo as 50, 51, 52 ... The combo param (EnemyComboBehaviorParam)
  group *g* is requested as 50 + *g* − 1, and its `ai_action_id` is the attack whose animations it plays as a chain
  (`Attack_NN_1`, `_2`, `_3`).
  - An attack requested only that way is **"combo only"** and counts as used.
  - The 50 + *g* − 1 numbering is inferred. It fits every case: no script asks for a low combo id that is missing
    while a higher one exists. The misses (88 requests on PC) are shared scripts asking for more combos than one
    weapon variant has.
- **Mid-fight rows** (added 12 Nov): behaviour rows that no spawn points at are judged against the scripts of every
  spawn of the same model (row ÷ 100).
  - Evidence that a row is really switched in: the scripts request an attack that only this row sets up (the Last
    Giant asks for 8, 9 and 19, which only 309610 has), or a time-act event of type 2500 points at it.
  - Type 2500's first value is 1 in the Last Giant's arm tear-off (309600 → 309610), the Smelter Demon's phase change
    (305000 → 305010), the Burnt Ivory King's (690000 → 690010) and one of the Fume Knight's (675000 → 675010). It is
    read as "row + 10 × value".
  - That reading isn't settled. The Fume Knight also uses values 2 and 3, and the Sanctum Knight (c665000) uses 1,
    whose target 665010 is a spawned crossbow variant. c619110, c656000, c678000 and c688000 use values that match no
    row at all.
  - For ordinary enemies (weapon variants such as the Royal Swordsman's crossbow rows, Felkin, Benhart, the Bell
    Keepers), how the row is switched is not known. Neither type 2104 nor 2105 is the switch: they belong to spell
    casts and bow shots.
  - Requests are pooled across all of a model's scripts and phases. So an attack set up in a phase row still counts
    as used if the AI asks for it in any other phase. This errs towards *used*.

## The 18 attacks the Network Test requested

| enemy (behaviour row) | attack | what it is (param / animation) | network on PC |
|---|---|---|---|
| **Amana Shrine Maiden, sorceress** (105010) | 21 | the three-shot volley | routed |
| Black Hollow Mage / White Hollow Mage, incl. Executioner's Chariot (118000) | 12, 20 | a shot (12) and spell 20 | routed |
| Bonewheel Skeleton (133000) | 12 | melee | routed |
| Gyrm (134000) | 12, 13 | melee | routed |
| Prowling Magus (137010) | 21 | spell | routed |
| Stone Knight (150000) | 20 | shot | routed |
| Basilisk (210000) | 21, 22 | shots | routed |
| Giant Acid Horn Beetle (220000) | 22 | shot | routed |
| Giant Acid Horn Beetle (220000) | 21 | shot | not routed |
| Heide Knight, sword and spear (301000 / 301010) | 10 | melee | routed |
| Lion Clan Warrior / Golden Lion Clan Warrior (308000) | 14 | melee | routed |
| Ruin Sentinel (325000) | 16 | melee | not routed |
| **Demon of Song** (602000) | 15, 16 | melee | not routed |

Plus the Varangian Sailor's 15 on its mid-fight row 113010 (routed, a65_15_1010 / 2010). It is outside this count
because the game-wide list covers spawned rows only.

Dropped on 12 Nov as combo moves:
- **The Heide Knights' 13**: the Network Test asked for it alone; retail plays it as their second combo, starting
  with the same hit.
- **The Last Giant's 15**: its damage row 130960510 and its three animations a65_15_1010/1020/1030 are combo 3
  (EnemyComboBehaviorParam 309600 group 3 = action 15, hits 130960510 / 520 / 530), which retail requests as 52 in
  both phases. Only the one-swing request was dropped: the Network Test asked for 15 on its own and had combos 50
  and 51 only.
- **The Last Giant's 16 in his arm phase** (behaviour row 309610, 1.02 name 左腕装備時 "left arm equipped") follows
  the same pattern: it plays only as combo 3 (1510 / 1520 / 1530, a65_16_2010-2030). Every other set-up attack in
  both rows is requested, and all 31 attack animations in his TAE / network are reachable.

**The sorceress in the Network Test** (`ai105000`, sub-machine 2147483615):
- She chose between both spells. State 1 rolls slot 0 to 0-20 (`1:1(0, 0, 20)`).
- A roll between 0 and 10 casts the orb, `6:sub(20, 80, 3, 4, 50)`. Anything else casts the volley,
  `6:sub(21, 80, 3, 4, 50)`. That is roughly half each.
- A timer (`f2(3) ≥ 3.5 × f5101(0)`) spaced the casts.
- The NT's attack row already had the volley (EnemyBehaviorThirdParam 105010 group 2 = bullets 110501310 / 320 / 330).
- Retail replaced this routine with an orb-only one, gated on attack 20 being ready.
- This is the template for the restore in `<local path> patches and fixes.md`.

## Other notable unused attacks (routed, so they have an animation path)

- **Bosses:**
  - The Rotten 20, 21 (two shots);
  - Throne Watcher 17;
  - Looking Glass Knight 7;
  - Dragonrider 12;
  - Freja (Duke's Dear, encounter) 13;
  - Executioner's Chariot 1;
  - DLC (no NT scripts to compare): Aava / Lud & Zallen 0.
- **Common enemies:**
  - Parasite Spider 14 (99 spawns);
  - Captive Undead / Hollow Prisoner shot 20 (about 130 spawns);
  - Basilisk 23 (and 21 / 22 above);
  - Giant Basilisk 21, 22;
  - Ogre 20, 21 (two shots);
  - Manikins 17 and shot 23;
  - Forest of Fallen Giants giants 13-15 (all variants; they share one damage row);
  - Black Hollow Mage 22, 24 (as well as 12 / 20 above);
  - Prowling Magus 23, 25, 26;
  - Leydia Pyromancer 20, 23;
  - Desert Sorceress 15, 16, 22;
  - Gyrm 16, 20;
  - Dark Stalker 13, 16;
  - Undead Aberration 20, 21;
  - Torturer (catch pole) 14;
  - Imperious Knight 14, 18;
  - Grave Wardens 16 (and 18 since 1.10);
  - Primal Knight 13;
  - Laddersmith Gilligan 21.
- **Attacks 0-9** (Acid Horn Beetle 0, Bonewheel 0, Looking Glass Knight 7, Chariot 1, Aava 0) are named
  `Attack_0N_Action` / `OneAction` in the animations. They may be started by the engine rather than the AI (reactions,
  special moves), so treat these as less certain.

## Combo moves (removed from the list on 12 Nov)

These 23 were listed as unused on 25 Oct. The AI never asks for them on their own, but it does ask for a combo that
plays their animations, so they are seen in game. The "hits" column compares the combo with the attack's own row:
- **same first hit**: a requested combo starts with the attack's own damage row, so nothing is unused at all;
- **own rows**: the combo hits with other rows, so the animation plays but the single version's own damage / bullet
  rows are never used. Several of these are stubs (the Dragonrider's 11 = 100000001, the Lost Sinner's 10 =
  162400000), which is why some showed up as "broken on PC" in [attack swap candidates](attack-swap-candidates.md);
- **no rows**: the combo group names no damage row (how it then hits is not known).

| enemy (behaviour row) | attack | hits | combos the AI requests (id = action) |
|---|---|---|---|
| Hollow Soldier, bow (102040) | 11 | no rows | 50 = 11 |
| Royal Soldier, spear and shield (102100) | 11 | own rows | 50 = 11, 51 = 12 |
| Royal Soldier, bastard sword / bow (102110) | 11 | same first hit | 50 = 11, 51 = 12 |
| Giants, unarmed / club (121000 / 121010) | 16 | same first hit | 50 = 16 |
| Giant, pyromancer (121020) | 16 | own rows | 50 = 16 |
| Heide Knights (301000 / 301010) | 13 | same first hit | 50 = 11 or 12, 51 = 13 |
| Undead Jailer (302000) | 10 | same first hit | 50 = 10 |
| Smelter Demon (305000) | 13 | own rows (the single version names only a bullet row, the combo damage rows 130500310 / 320) | 50 = 10, 51 = 10, 52 = 12, 53 = 13 |
| Forgotten Giant (309000) | 10, 11 | same first hit | 50 = 10, 51 = 11 |
| **The Last Giant** (309600) | 15 | same first hit | 50 = 10, 51 = 11, 52 = 15 |
| Grave Warden, sickle (312000) | 10, 11 | same first hit | 50 = 10, 51 = 11 |
| Demon of Song (602000) | 12, 13 | own rows | 50 = 7, 51 = 8, 52 = 12, 53 = 13 |
| Dragonrider (611000) | 11 | own rows | 50 = 10, 51 = 11, 52 = 17 |
| The Lost Sinner (626000) | 10 | own rows | 50 = 10, 51 = 10, 52 = 11, 53 = 12, 54 = 13 |
| Sir Alonne (680000) | 15 | own rows | 50-56 = 10, 10, 11, 12, 15, 10, 15 |
| Sinh (681000) | 16 | no rows | 50-55 = 16, 6, 11, 12, 19, 19 |
| Burnt Ivory King (690000) | 6 | same first hit | 50-56 = 10, 11, 6, 16, 10, 11, 15 |
| Burnt Ivory King (690000) | 16 | own rows | same |

## Mid-fight behaviour rows (added 12 Nov)

On PC, 110 behaviour rows with set-up attacks have no spawn pointing at them:
- 46 belong to models nothing spawns, so they are not judged;
- 64 were judged against their model's scripts;
- 29 of those have evidence that they are switched in. They are boss phases (the Last Giant, the Smelter Demon, the
  Fume Knight, the Burnt Ivory King, the Executioner's Chariot) and weapon or spell variants (Royal Swordsman crossbow
  rows, Lindelt Clerics, Hollow Priests, Undead Steelworkers, Felkin, Benhart, the Bell Keepers, Targray and others).

Attacks on these rows that the scripts never request:

| row | enemy | attack | network | notes |
|---|---|---|---|---|
| **113010** | **Varangian Sailor** (a weapon row) | **15** | routed | requested in the Network Test, v1.00 and 1.10; dropped by Scholar's rewrite of ai113000 |
| 123010 | Suspicious Shadow | 16, 18 (routed); 2, 4-7 (not routed) | | 2-7 name Manikin damage rows (1124xxxxx), so the row looks copied; the Network Test requested 16 |
| 133010 / 133020 | Bonewheel Skeleton | 0, 12 (routed); 16, 20 (not routed) | | 12 is the same move already listed for row 133000 |
| 137000 | Prowling Magus | 13 | not routed | |
| 619100 | Executioner's Chariot | 20 | not routed | |

The other 35 rows have no sign of being switched in, so they may never be used at all. Their unrequested attacks are:
- Heide Knight 10 / 13 / 15 / 19 (rows 301020, 301030);
- Parasite Spider 14 (203001);
- Grave Warden 16 / 18 (312011);
- Prowling Magus 21 / 23 / 25 (137020);
- Rupturing Hollow 20 / 21 (125001-125004).

The Last Giant's arm row 309610 adds nothing new. Its 16 counts as requested because the
scripts ask for 16 in phase 1, where it is a different move (130960610). Requests are pooled across phases, so a move
that is unused in one phase only would not show up.

**Data only (no network route)**:
- the Corrosive Ant Queen has 12;
- Torturers 10;
- Hunting Dogs 6;
- Suspicious Shadow 5;
- Looking Glass Knight 4;
- a handful each on other bosses (Darklurker, Lost Sinner, Old Dragonslayer, Nashandra, Mytha, Najka, Belfry
  Gargoyle, Ruin Sentinel).

## Restoring one

For a routed attack, the param rows, the animation (with its hit or shot events) and the network transition are all
in place, so the only change is in the AI script. Either:
- [add a branch](../modding-scripts/writing-esd-files.md) that calls the enemy's attack routine with the attack id; or
- for a quick test, swap the id in an existing call.

A data-only attack would also need a network edit.

**Proven once:** the Amana sorceress's volley (2026-10-25, PC, live). Swapping the orb's attack id 20 for 21 in the
loaded script (`pcmem.py volley on`) made her cast it, working as designed: three projectiles converging on the
player. The other routed attacks should behave the same way, but each needs its own check.

**37 of the 104 routed attacks are broken or unfinished on PC** (checked 2026-10-25):
- 30 name projectile or damage rows that no longer exist on PC; most were deleted between 1.00 and PC.
- 7 fire projectiles with effect 0 or the placeholder 1300.

Only 12 are both complete and visibly different from the enemy's normal attacks; see [attack swap candidates](attack-swap-candidates.md).

(12 Nov: these counts are from the 25 Oct list of 104. Several of the broken bosses' attacks have since turned out to
be combo moves, whose single versions are stubs because only the combo version is used: the Smelter Demon's 13, the
Demon of Song's 12 / 13, the Dragonrider's 11, the Lost Sinner's 10 and the Burnt Ivory King's 16. The broken check
was not re-run.)

**But "routed" doesn't mean finished.** DebugManager's hitbox view showed the volley firing five projectiles, two of them
invisible: leftover Network Test placeholders that were never removed when the attack was redesigned. Expect some cut
attacks to be half-done like this. A test should look at the hitboxes as well as the animation.

## Not checked

- Live: none of these were seen in game.
- Event scripts (map ESDs) forcing an attack: a sample of their commands found nothing attack-like, but they weren't
  traced.
- PS3 networks (32-bit; `nmb.py` reads only the PC's), so "routed" is PC only.
- What each attack looks like: only the animation names, damage / bullet rows and shot events were read.
