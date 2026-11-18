# NG+ near boss arenas: what changes, and how the mod copes (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed in code"
    Read from the game's code or logic (disassembly or data), but not run live. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

Tags in square brackets say how we know a claim: `[live B56]` = seen in the game (the id points to the project's test
log, see [Home](../how-to-read.md#the-modding-pages)), `[code]` = read in the exe, `[data]` = read in the game files, `[inference]` = reasoned, not
proven, `[untested]` = an idea only. **Every live arena test so far was in New Game (NG).** Nobody has yet seen an NG+
enemy at an arena. Most of this page is a reading of the params, the scripts and the exe, plus what the mod does about
it. The NG+ spot check is still open (see Open questions).

## Summary

- **NG+ enemies are extra rows in the same `GeneratorParam`, not a separate param.** The gate is a small number in
  `AppearanceEventID` (1 to 8), or an event flag with a 2 in the next field. 79 rows carry 2, 25 carry 3, 7 carry 8;
  34 carry 1: the 23 in the Dark Chasm of Old behave as "always", the other 11 look conditional. [data] [inference]
- **The gate is an intensity total, per bonfire.** The total is the bonfire's Ascetic cycles (`byte >> 1`) plus the
  character's own cycle number, capped at 8. A generator names its bonfire in `MapObjectBonfireID`, which is not always the
  bonfire the player rests at. [code] That the generator's `AppearanceEventID` is the number it is compared with is our
  reading. [inference]
- **To set a tier by hand, write `(n << 1) | 1` to the bonfire's level byte** for `n` extra cycles: 3, 5, 7 ... 15. The
  older rule "level 2 = NG+, level 3 = NG+2" is wrong: 2 is one cycle and unlit; 3 is the same cycle, lit. [code] See
  [Bonfire lit state](bonfire-lit-state.md).
- **The same total picks which spawn limit a generator uses**: `SpawnLimit` while the total is below 2,
  `SpawnLimitGameClear` from 2 up. The mod writes 255 (no limit) into both on every fight generator, so neither counts at
  any tier. [code]
- **What the mod does:** it unlimits the Lost Sinner's script-woken pyromancers, force-deletes 18 NG+ generators at 12
  arenas, clears Freja's saved-HP flags at every load, switches off the Rat bosses' invader group, and keeps Mytha's
  room drained. All of it was built from the data, and in NG there is little for it to act on. The rows still arrive,
  fight and win [live B11, B12, B15, B17, B19].
- **Main traps:** the scripts of three NG+ features test `f132244 > 2`, which by the code reading is NG+2, not NG+; the
  bonfire that governs a boss can differ from his arena bonfire (the Pursuer); and a fight add woken by a script flag is
  missed by a search on `ActivationEventID1`.

### NG+ content is extra generator rows [data]

A map's `GeneratorParam_<map>` holds every spawner. NG+ content is more rows in that file. Their enemies are usually the
"black phantom" rows of an ordinary enemy (ids such as 302009, 106009, 872009, 138001). There are no NG+ param tables to
patch. Counts over all 28 maps, 2,458 generators (re-run 17 Nov):

| `AppearanceEventID` | Rows | Where | Reading |
|---|---|---|---|
| 0 | 1,878 | 27 maps | no gate |
| 1 | 34 | m40_03 Dark Chasm of Old 23 (the Darklurker's), m10_23 5, m20_11 3, m10_14 2, m10_17 1 | always in the chasm; the other 11 differ [inference] |
| 2 | 79 | 16 maps | NG+ |
| 3 | 25 | m50_36 13, m50_37 11, m10_04 (Majula) 1 | NG+2 (the DLC and one in Majula's basement) |
| 4, 5, 6, 7 | 1 each | all in m10_04 | NG+3 to NG+6 |
| 8 | 7 | m50_36 6, m10_04 1 | NG+7 |
| 10 | 8 | m10_32 7, m10_23 1 | not a tier |
| 30 | 76 | m10_25 Black Gulch | not a tier |
| 50 | 31 | m10_02 30, m10_15 1 | not a tier |
| 100 and up | 316 | 24 maps | an event flag, not a tier |

Why value N reads as "present once the intensity reaches N":

- **Majula's basement** has generator 1100 (value 0) and 1101 to 1107 (values 2 to 8). Its script (the group named 地図書き：
  スケルトン撃破カウント, "map maker: skeleton kill count") compares the cycle with 1 to 8 (`1:140711`) and wants the kill
  flags of 1100 up to 1100 + N - 1. So the basement holds one skeleton in NG and eight at the top. [data: the flag
  reading is the 5 Nov one; the eight compare calls were re-read on 17 Nov]
- **The community's NG Cycles page** says the basement gains one skeleton per cycle up to NG+7, and that Brume Tower gains
  13 enemies at NG+2 and 6 more at NG+7. Our counts match: m50_36 has 13 rows with value 3 and 6 with value 8; m50_37
  has 11 with value 3. [community] [data]
- **The cap is 8**, the highest tier value in the table (see "The intensity total"). [code]
- **Value 1 is "always" only in the Dark Chasm of Old.** Its 23 rows are all value 1 with condition byte 0 or 1, meaning
  no condition, and the Darklurker passes in NG. [data] [live B19] The other 11 value-1 rows carry byte 23 or 26
  instead, and the value-2 rows beside them carry 22 or 25 (the Copse's torturers 5010-5013 beside their black-phantom
  copies 5014-5017, the Chariot's 8000 beside its phantom 8001); see [GeneratorParam fields](generatorparam-fields.md). So an
  earlier project note's "1 = NG only" probably holds for those twins: NG-only against NG+-only, not standing together
  in NG+. The bytes are [data]; that reading is [inference]; nobody has looked in NG+.
- **10, 30 and 50 are not tiers.** If they were compared with the same `>=` against a total that tops out at 8, those 115
  rows could never spawn, yet they are the ordinary population of Black Gulch, Things Betwixt and the Shaded Woods. We do
  not know what these values mean. Their condition byte is 1 for 30 and 50, the byte of an unconditional row, and 4 for
  10 [data], so 30 and 50 probably have no condition; what they gate is unknown. [inference]

**A flag plus a cycle.** When `AppearanceEventID` is an event flag (100 and up), the cycle can sit in
`PreviousHostileEventID1` or `2` (+0x14, +0x18). Seven rows in four maps have a 2 there: the Wharf's 4550 (flag
118000010, the ship), the Forest's 1800 and 9000, the Copse's 4122 and 4200, Things Betwixt's 7000 and 7010. Four castle
rows (1050 to 1053 in m20_21) hold 15, and two of them also a 2. [data] The community enemy sheet tags the Wharf's, the
Forest's and the Copse's five as "(NG+)", so "this flag and intensity 2" is the likely reading. [community] [inference]

### The intensity total [code]

Read by disassembly of the stock exe on 17 November. Names are Ghidra `FUN_14xxxxxxx` (image base 0x140000000); RVAs are
written `+0x...`.

| Function | RVA | What it does |
|---|---|---|
| `FUN_14017c1f0` | `+0x17c1f0` | Takes a bonfire list index, returns the level byte `>> 1`: the bonfire's cycles. |
| `FUN_1403ba1a0` | `+0x3ba1a0` | The same from a bonfire id. |
| `FUN_14019ee30` | `+0x19ee30` | The character's own cycle number: the dword at `[[[GameManagerImp+0xa8]+0xc0]+0x68]`. If an object exists at `GameManagerImp+0x22f0` it may supply the value instead (not followed). |
| `FUN_14005ba50` | `+0x5ba50` | **Total** for a bonfire id: its cycles plus `FUN_14019ee30`. |
| three virtual methods | `+0x4112f0`, `+0x411330`, `+0x411370` | Methods of the generator class (vtable slots at +0xB0, +0xB8, +0xC0 of the table at `+0x1583550`). Each takes the total of the generator's own bonfire (its index is at generator +0x80) and tests it `>=`, `<=` or `==` against the argument. |
| `FUN_140411a40` | `+0x411a40` | The generator's constructor. It reads the u16 at **param row +0x4A** (`MapObjectBonfireID`), looks that bonfire up and keeps its index at +0x80. It then calls `FUN_140412e70`. |
| `FUN_140412e70` | `+0x412e70` | Picks the live spawn limit (next section). |
| the cap | `+0x10c3b20` | A dword holding 8. The limit function clamps the total to it. |

The event-function dispatcher has four consecutive cases, in the order of the `.edd` function names: they call
`FUN_14019ee30` (`f132241` ゲーム周回数取得, "get game cycle"), `FUN_14017e390` (`f132242` 点火済み篝火数取得, "lit
bonfire count"), `FUN_1403ba1a0` (`f132243` 篝火周回数取得, "bonfire cycle") and `FUN_14005ba50` (`f132244` 合計周回数取得,
"total cycle"). The last caller sits at `+0x460182`. The match is by order and shape, not traced through the table. [code]

What we did not find: the code that feeds `AppearanceEventID` to the three methods. A small condition interpreter near
`+0x4125f0` calls those slots by index, but no one traced where its condition records come from. So "value N means total
>= N" rests on the data and the `>=` method, not on a seen call. [inference]

### The tiers: what total a fresh NG has

Four independent things fit a total of **1** in a fresh NG at an unburnt bonfire, so each Ascetic or NG+ adds one:

1. The cap is 8, and the highest `AppearanceEventID` is 8, which the community calls NG+7. [code] [data]
2. The limit function switches to `SpawnLimitGameClear` at a total of 2 or more, the first NG+. [code]
3. Majula's counter compares 1 to 8. [data]
4. The community's per-cycle counts line up with value N = NG+(N - 1). [community] [data]

Nobody has read the character's cycle number live, so it stays an inference. [untested] On that reading:

| Level byte | Cycles at the bonfire | Total in an NG character | Community name | `AppearanceEventID` rows reached |
|---|---|---|---|---|
| 1 | 0 | 1 | NG | 1 |
| 3 | 1 | 2 | NG+ | up to 2 |
| 5 | 2 | 3 | NG+2 | up to 3 |
| 7, 9, 11, 13 | 3, 4, 5, 6 | 4, 5, 6, 7 | NG+3 to NG+6 | up to 4 to 7 |
| 15 | 7 | 8 | NG+7 | up to 8 |
| above 15 | 8 and more | capped at 8 | | nothing new |

A real NG+ character at an unburnt bonfire should have the same total as an NG character with one Ascetic burnt there.
[inference] The level byte also carries the lit bit; see [Bonfire lit state](bonfire-lit-state.md).

[live A10] wrote 2 to the Last Giant's bonfire: his HP went from 2,530 to 5,313 (x2.1 for him; the first HP step differs by enemy, x1.0 to x2.1, because the byte picks an `EnemyParam` HP column: see [Difficulty levers](difficulty-levers.md)) on the next load. [live B56] A real
Bonfire Ascetic burnt at that bonfire did the same, and it is permanent for that character. By the code, A10's 2 was
cycle 1 and unlit.

**The conflict.** Three scripts that mean "from NG+" test `f132244(bonfire) > 2`, and the ESD operator is a true "greater
than" (opcode 0x94). With a total of 1 in NG, that is NG+2 and above. [data] [code] They are the Lost Sinner's add group,
Black Gulch's coal-tar group (named 【再現】コールタールの出現_2周目から出現, "appears from the 2nd lap") and the Rat bosses'
invader group. Either the developers counted from a different base in these scripts, or the character's cycle number is 2
in NG. The four points above favour the first. A live read of the total settles it. [untested]

### The same total picks the spawn limit [code]

`FUN_140412e70` runs when the generator is built, so a change applies from the next map load. It works like this:

1. Clamp the total to 8.
2. Take `SpawnLimit` (param row +0) if the clamped total is below 2. Take `SpawnLimitGameClear` (row +1) if it is 2 or more.
3. If the byte is 255, the live limit (generator +0x8D) is 0: no limit. Any other non-zero byte is the limit.
4. If the byte is 0, take the default from byte +0x25 of a param row found by the enemy's id. The probe in [live B8] read 1
   on the two boss generators and 12 on ordinary ones, so that byte is the "boss or not" default. We did not name the
   param. [inference]

So the earlier open question "which byte applies at which cycle" has an answer: `SpawnLimit` in NG, `SpawnLimitGameClear`
from the first NG+, and also in NG at a bonfire raised by one Ascetic (that last part leans on the total being 1 in NG).
The spawn check itself is on [Bringing a boss back](respawning-bosses.md). The mod writes 255 into both fields, so the check
never refuses an NG+ fight generator. [code] [inference] [live B8, with both fields at 255]

Vanilla pairs on NG+ rows [data]:

| Generators | `SpawnLimit` / `SpawnLimitGameClear` |
|---|---|
| Flexile's Suspicious Shadows 5220, 5221 | 0 / 1 |
| The Copse's black-phantom torturers 5014-5017 (value 2) | 0 / 12 (their NG twins 5010-5013, value 1: 0 / 0) |
| The Chariot's phantoms: 8000 (value 1), 8001 (value 2) | 1 / 1 and 12 / 12 |
| The Rat bosses' invaders: NG copies 630, 640 | 1 / 1 |
| The Rat bosses' invaders: NG+ copies 631, 641 | 12 / 12 |

A limit is not a gate. An unlimited generator whose `AppearanceEventID` is not met stays out of the world: Flexile's two
Shadows, unlimited, were not reported in NG. [live B13, weak: nobody looked for them on purpose]

### What scripts do with the cycle [data]

| Call | Developer name | Gloss |
|---|---|---|
| `f132241` | ゲーム周回数取得 | game cycle |
| `f132243(bonfire)` | 篝火周回数取得 | bonfire cycle |
| `f132244(bonfire)` | 合計周回数取得 | total cycle |
| `1:140711` | 【グループ条件：その他】周回数比較 | compare cycle |

The map scripts of the arena maps use them in a few places. No other boss script checks the cycle; Flexile's NG+ content
is params only (m10_18 has no cycle check).

| Map | Group | What it does with the cycle |
|---|---|---|
| m10_16 | 2147483462 | The Lost Sinner's "reproduce" part returns "do nothing" unless `f132244(16685) > 2`. |
| m10_25 | 2147483489 | Black Gulch's coal-tar group wakes the Darksuckers on `f132244(arg) > 2`. |
| m10_33, m10_34 | 118220 | The Rat bosses' invaders: `f132244(33660 or 34655) > 2` chooses the NG+ copy. |
| m10_14 | 2147483489 | Freja's prelude gate: `1:140711(0, 0, 0, 1, 0)`, read as "game cycle is 1". [inference] |
| m10_02 | 2147483620 | The Firekeeper's house in Things Betwixt: `f132241() > 2`. |
| m10_04, m10_14, m10_16, m10_19 | various | Shop lists and mimic chests call `1:140711` with 2 or 3 (arguments not decoded). |

### Flexile Sentry (m10_18) [data]

| Generator | What | Gate | Distance to the arrival | Mod |
|---|---|---|---|---|
| 5220, 5221 | Suspicious Shadows (EnemyParam 123002, 123022) | value 2, bonfire 18650, `ActivationEventID1` = fight flag 118020080 | 19.8 and 16.5 m | fight adds: unlimited |
| 4550 | Aurous Knight phantom (854000) | flag 118000010 (the ship) in `AppearanceEventID`, 2 in `PreviousHostileEventID1` | 4.5 m across, **10.1 m up** | force-deleted |

The Shadows wake with the fight flag, so `tools/bosstable.py` already counted them as fight generators. The knight is an NPC
type 10 m up, so the removal rule skips it twice (NPC type; above the 8 m window). It needs the ship flag to exist at all.
The mod sets that flag at every load and runs the removal after it. [code] The Wharf's order of flag-then-removal passed in
NG, where 4550 does not exist, so the order has never been seen to matter. [live B13, B15] [untested]

### The Lost Sinner (m10_16) [data]

How the pyromancers come, as the script is built:

1. A group (15020, 【ボス：忘れられた罪人_ザコ起動】, "Lost Sinner: wake the adds") calls a preset with her generator, 848.
2. The preset's "reproduce" part (group 2147483462) returns "do nothing" unless `f132244(16685) > 2`. 16685 is her arena
   bonfire.
3. In that case, while the fight is on (`1:140521`, ボス戦判定 "boss fight check") and her HP compares with 70 %
   (`1:140121`, HP割合比較 "HP ratio compare"), a small group (2147483460) sets flag **116020088**. The two checks and
   the flag-setting group are all there; the chain between them is the 5 Nov reading.
4. That flag is `ActivationEventID2` of generators **8100 and 8101**: two human phantoms (EnemyParam 732000, 733000), value 2,
   bonfire 16685, aggro group 156, 15 to 16 m either side of her and about 29 m from the arrival.

116020088 is in the fight's own flag block, which resets at every load, so the adds re-arm with the boss. They are not
unlimited in the data (0 / 0), and `bosstable.py` finds fight generators by `ActivationEventID1`, so it missed them. The
build lists them by hand (`EXTRA_GENS[1016010]`). Matching `ActivationEventID2` against the map's general temporary-flag
block instead would pull in about 100 unrelated rows. [data] [code]

Two Undead Jailer phantoms (6110, 6310; EnemyParam 302009; value 2) stand 21.4 and 21.1 m from the arrival, 6 m up, just
outside the 20 m removal radius. They are force-deleted by hand. [data] [code]

By the conflict above, the pyromancers come at a total of 3 or more: an NG+2 character, or an NG character with two
Ascetics burnt at 16685. One community page says Ascetics in NG do not bring them. We have not seen either. [untested]

## The other arenas

Generators gated to NG+ within about 45 m of an arrival, and what the current build does. Distances are horizontal /
vertical to the arrival point. The distances were re-read on 17 November. [data]

| Fight | Generators | Gate | Distance | Mod now |
|---|---|---|---|---|
| Heide's Dragonrider | 2000 phantom (847000) | value 2 | 9.7 m / 0 | deleted |
| Heide's Dragonrider | 1042, 1050 old knights | value 2 | 39 and 37 m / +7 | left |
| Old Dragonslayer | 9110, 9120 phantoms (847100), one each side | value 2 | 4.5 m / 0 | deleted |
| Blue Smelter (DLC) | 9502 Astrologer phantom (872009), 9070 Iron Warrior | value 3 | 4.3 m, 12 m | deleted |
| Congregation | 2200 Hollow Priest phantom (106009), on a ledge | value 2 | 29.4 m / +5.5 | deleted |
| Mytha | 5202 Grave Warden | value 2 | 5.3 m / 0 | deleted |
| Smelter Demon | 3070, 3071, 7009 | value 2 | 20.3, 18.5, 35 m | deleted |
| Smelter Demon | 3040 | value 2 | 44 m / -7 | left |
| Executioner's Chariot | 8001 phantom (852000, 12 / 12); its twin 8000 (1 / 1) | value 2; value 1 | 8.1 m | both deleted |
| Fume Knight (DLC) | 9040 Iron Warrior | value 8 | 12.4 m | deleted |
| Lud and Zallen (DLC) | 7202 Rampart Golem phantom | value 3 | 5.3 m | deleted |
| Belfry Gargoyles | 6510, 6511 Stray Dogs (227009) | value 2 | 48.7 and 47.2 m / -6 | deleted |
| Pursuer | 7002 Hollow Soldier phantom | value 2, 0 / 12 | 22.7 m / -6 | **left** |
| Pursuer | 2680, 6200 | value 2 | 28 m / -18, 44 m / -11 | left |
| Sir Alonne (DLC) | 7610 Alonne Captain | value 3 | 34 m / +9 | left |
| Twin Dragonriders | 8000, 8010 Pursuers (1 / 1) | value 2 | 28.5 m / 22 m **below** | left |
| The Rotten | 4402, 4404 Darksuckers | value 2 and the coal-tar group | 27 m / +7 | left |
| Covetous Demon | 3602 Desert Sorceress | value 2 | 32 m / +7 | left |
| Last Giant | 6000, 6001 | value 2 | 6 to 11 m across, **33 m up** | left |

"Deleted" is `1:131721(generator, 0)` (【ジェネレータ】敵の強制削除, "force-delete the generator's enemy") in the arena
group, as on [Removing enemies](removing-enemies.md). The removal is harmless when the generator does not exist: the Smelter,
Mytha and Chariot rows carry NG+ generators in their lists and passed in NG. [live B11, B12] The phantoms (2000, 9110,
9120, 9502, 8000) are NPC types, which the removal rule skips, so they were added by hand. [code] The DLC rows (9040,
9070, 9502, 7202) passed in NG. [live B17] The "left" rows are on another floor, far from the arrival, or a judgement
call: the 5 Nov sweep marked the Pursuer's 7002, the closest of them, as borderline, and it was not added. [data]

**Not every NG+ enemy is a generator row.** The Rat bosses' offline invaders are made by a script (`1:131741`,
【ジェネレータ】NPC霊体生成, "create an NPC spirit") when the player enters a trigger box. In NG the group creates
Bowman Guthry (630, Royal Rat Authority's map) or Rhoy the Explorer (640, Vanguard's) once; in NG+ it uses 631 or 641, 12
times. Rhoy appears 15 m from the Vanguard's arrival, Guthry 58 m from the Authority's. [data] The mod deletes the
`EventParam` row of group 118220 in both maps, so the group never runs; both fights still arrive, fight and win. [live B15]
That the invaders are gone was never checked by walking into the trigger box. [untested]
Other invaders within 45 m of an arrival that the mod leaves alone (spirit generators, `Unk84` 2, conditions not
checked): Navlaan at the Guardian Dragon (28 m), Congregation (44 m) and the Twin Dragonriders (45 m), Forlorn at
Congregation (32 m) and Aava (32 m), Kindalur at the Demon of Song (43 m), Aurheim at Aava (27 m). [data, 5 Nov scan]
Mytha's Forlorn (935, 21 m) and the Belfry's Bell Keeper (generator 541, 4.5 m from the arrival, group 111284) are
switched off. See [Removing enemies](removing-enemies.md).

### Things that a new cycle changes

- **Freja's prelude.** Her map's script has a "prelude" (前哨戦, "outpost battle") that is skipped in the first cycle: the
  gate calls `1:140711(0, 0, 0, 1, 0)`, read as "game cycle is 1". [data] [inference] While it runs, a group tracks her HP
  in 5 % steps from 95 to 50 % and sets ten flags, 114000085 to 114000094; a ten-way check group reads them back. The
  earlier reading is that the real fight then starts at the recorded HP at every load. [data] [inference] The mod clears
  the ten flags at every Brightstone Cove load, before the removals, so a fight starts at full HP. In NG that does nothing,
  because the prelude never runs. A second lever, flag 114000083 ("prelude done", unverified), is not set. [code]
  [untested]
- **Earthen Peak's windmill.** The poison in Mytha's room drains when the windmill is burnt (group 42000, flag
  117000055), and a new cycle brings the poison back [inference]. The mod makes group 42000 start with both windmill flags
  on, so the room is drained at every load. [code] [live B25, B50] A global "remembered" flag was designed and never
  built.
- **The Bonfire Ascetic.** The shipped README tells players to burn one at a boss's bonfire to get him at NG+ strength.
  The data says that also brings that bonfire's NG+ enemies, which the removal lists were built to cover. [inference]

### A. List the NG+ generators near a new arrival

1. Read the map's `GeneratorParam`, `GeneratorRegistParam` and `GeneratorLocation` (positions are map-local; add the
   map offset, as on [Removing enemies](removing-enemies.md)). `tools/bosstable.py`'s `gens_of(map)` does this.
2. Keep the rows whose `AppearanceEventID` is 2 to 8, or whose `AppearanceEventID` is a flag with a 2 in the next two
   fields, **or whose `ActivationEventID2` is a flag**. Keep the 10 / 30 / 50 rows out.
3. For each row note `MapObjectBonfireID`, `SpawnLimit` / `SpawnLimitGameClear`, `Unk84` and the enemy's `EnemyParamID`.
4. Read the map's event script for `f132241`, `f132243`, `f132244` and `1:140711` (the `.edd` names above). A hit in a
   fight's group means the fight changes by script, and no param read will show it.
5. Sort the rows. **A script-woken fight add** is unlimited, as on [Bringing a boss back](respawning-bosses.md). **An area
   phantom or enemy** near the arrival is force-deleted. **An NPC-type phantom** is added by hand, because the rule skips
   NPC types.
6. Put any flag the NG+ enemy needs before the removal in the arena group (the Wharf's ship flag).

### B. Preview NG+ at one arena without a new playthrough (the cheap check; not yet run)

1. Pick the bonfire ids from step 3, not from the arena: the Lost Sinner's 16685, Flexile's 18650, Heide's 31655 (which
   also governs the Old Dragonslayer's two phantoms), the Blue Smelter's 36665.
2. With the game running offline: `tools/arena.py bonfire <id>` reads the byte; `tools/arena.py bonfire <id> 3` writes it
   (use 5 for the DLC's value-3 rows). The lit bit stays on, so the travel menu is unchanged.
3. Reload the map, go near the arena (bosses spawn when the player nears them [live A10]) and look. At 16685, do the
   pyromancers come at 70 %? At 18650, do the Shadows join, and is 4550 gone?
4. Put the byte back to 1. It also takes effect on the next load.
5. This tests the intensity gates only. A script that reads the game cycle alone (`f132241`, and probably Freja's
   prelude gate) does not follow a bonfire's byte, and nor do flags that a new cycle resets. [code] [inference]

## Pitfalls and what failed

| What went wrong | Why | Fix | Test |
|---|---|---|---|
| "Write 3 for NG+2" in the first plan | The byte is `(cycles << 1) | lit`; 3 is cycle 1, lit | Write 5 | code, W16 |
| A10 wrote 2 | One cycle, but unlit | Write 3 | A10 gave the HP step; the menu was not looked at |
| The first reading "a fresh NG has total 2" | It came from a label, "from the 2nd lap", on a script that tests `> 2` | The cap, the GameClear threshold and the data say 1; the three scripts say 2. Open | 17 Nov |
| The Lost Sinner's pyromancers were not unlimited | `bosstable.py` follows `ActivationEventID1`; theirs is `ActivationEventID2` | `EXTRA_GENS` by hand | 5 Nov data |
| Flexile's knight 4550 outside the removal rule | NPC type, and 10 m above the 8 m window | By-hand removal, after the ship flag | 5 Nov data |
| Heide's, the Old Dragonslayer's and the Blue Smelter's phantoms outside the rule | NPC types at 4 to 10 m | By-hand removal | 5 Nov data |
| The Blue Smelter's phantom was never met | It exists only from NG+2 (value 3) | Removed anyway | 5 Nov data |
| NG+ adds absent with `SpawnLimit` 255 in NG (Flexile's Shadows) | A limit is not an appearance gate | None needed | B13, weak |
| The Pursuer's NG+ rows follow the wrong bonfire | The Pursuer borrows Soldiers' Rest (10660); his generator and his area's NG+ rows name Cardinal Tower (10655) | Raise 10655, not his arena bonfire | data |
| The Old Dragonslayer's phantoms follow another bonfire | Arena bonfire 31660; the phantoms name 31655 | Raise 31655 | data |
| A prelude Freja would start every fight at her saved HP | Ten flags are never cleared | Clear them at each load | built 5 Nov; unseen |

We compared each fight's generators with its arena bonfire for all 44 rows on 17 November. Every row matches, except
the rows that share another row's arena, the Pursuer, and the Darklurker (the Herald's menu, no arena bonfire). Two other
cases are on the table above: the Old Dragonslayer's phantoms, and the Rat Vanguard's invaders, which name 34655 while
his fight generators name 34650. [data]

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| Counts by `AppearanceEventID` value (2,458 generators, 28 maps) | read the params again | data, 17 Nov |
| The generator keeps the index of the bonfire named at row +0x4A | constructor | code `FUN_140411a40` |
| Total = bonfire cycles + character cycle, tested `>=`, `<=`, `==` | disassembly | code `+0x5ba50`, `+0x19ee30`, `+0x4112f0` |
| The live limit uses `SpawnLimit` below a clamped total of 2, `SpawnLimitGameClear` from 2; cap 8 | disassembly; the cap is a dword in `.rdata` | code `+0x412e70`, `+0x10c3b20` |
| 255 gives a live limit of 0; 0 gives 1 on boss generators, 12 on ordinary ones | probe of the live generators | live B8 |
| One bonfire's byte sets the boss's intensity on the next load | HP 2,530 -> 5,313 after writing the byte | live A10 |
| A real Ascetic at that bonfire does the same | HP read after burning one | live B56 |
| NG+ rows in a removal list are harmless in NG | Smelter, Mytha, Chariot rows passed | live B11, B12 |
| The NG+ changes (Flexile, Heide, Old Dragonslayer, both Rats, Freja, Congregation) did not break the NG fights | re-checks PASS | live B15 |
| Mytha's room stays drained at every load | the patched group, played twice | live B25, B50 |
| The Lost Sinner passes with the mod | all bosses worked | live B19 |
| The pyromancers wake through group 2147483460's flag, behind the total check | script read | data |
| Group 118220 chooses the Rat invaders' copy on `f132244 > 2` | script read | data |
| Deleting group 118220's row leaves both Rat fights playable | both fights played | live B15 |
| Value 1 is always present in the Dark Chasm of Old (the Darklurker, NG) | the chasm is all value 1, condition byte 0 or 1 | data; live B19 |
| A flag in `AppearanceEventID` with a 2 beside it marks NG+ (the Wharf and others) | the community sheet's tags | inference |
| An NG+ enemy has never been seen at an arena | every live test was in NG | all of them |

## Tools

All in `tools/`:

| Tool | What it does here |
|---|---|
| `bosstable.py` | `gens_of(map)` reads each map's generators with positions and enemy ids; finds fight generators; `EXTRA_GENS` holds the Lost Sinner's 8100 / 8101 |
| `rollout_table.py` | the removal rule and its by-hand lists (`FORCE_QUIET`), `EXTRA_FLAGS` (Freja's ten flags), `DISABLE_GROUPS` (group 118220, 4002000, 111284) |
| `arenamod.py` | writes the arena group: flags first, then one `1:131721` per listed generator |
| `buildmod.py` | writes 255 into both limits on every unlimited generator; deletes the `EventParam` rows of `DISABLE_GROUPS` |
| `modes.py` | Mytha's room always drained |
| `arena.py` | `bonfire <id> [byte]` reads or writes one bonfire's level byte (the raw byte, not a tier) |

The sweep behind the tables (a loop over every map's generators on top of `gens_of`) is not shipped as a tool.

## Open questions

- **The NG+ spot check.** Run recipe B at 16685, 18650, 31655 and 36665 with 3 and 5, then burn a real Ascetic at one, then
  play a real NG+ save. This also settles the total, the tier table and the Lost Sinner's threshold. [untested]
- **What is the character's cycle number in NG?** Read `[[[GameManagerImp+0xa8]+0xc0]+0x68]` live in NG and NG+. [untested]
- **Does `AppearanceEventID` feed the three methods?** The condition records were not traced. [untested]
- **What do 10, 30 and 50 do?** [untested]
- **Does a bonfire raised in NG switch its generators to `SpawnLimitGameClear`?** The code says yes; no one looked.
  [untested]
- **Does the Ascetic at the Pursuer's arena bonfire (10660) change him?** The data says no. [untested]
- **Do the Rat invaders really stay away,** and does `1:131742` stop a spirit? [untested]
- **Freja:** does the prelude's flag set leave her at a lower HP, and does clearing the ten flags at load fix it?
  [untested]
- **The value-1 and value-2 twins** (the Copse's torturers, the Chariot's phantoms): do they stand together in NG+?
  The condition bytes suggest not (23 / 26 against 22 / 25) [inference]; nobody has looked. [untested]
- **The left-alone rows:** the Pursuer's 7002, Sir Alonne's 7610, the Rotten's coal-tar Darksuckers. Do any attack at the
  arrival? [untested]

## Related pages

- [Bonfire lit state](bonfire-lit-state.md): the level byte's two halves and how to write a tier
- [Bringing a boss back](respawning-bosses.md): the spawn check, `SpawnLimit` and the Bonfire Ascetic
- [Removing enemies](removing-enemies.md): the removal rule, the by-hand lists, invader groups
- [GeneratorParam fields](generatorparam-fields.md): every field of a generator row
- [Difficulty levers](difficulty-levers.md): intensity as a difficulty setting
- [Moving a bonfire](moving-a-bonfire.md): moved and borrowed arena bonfires (the Pursuer uses Soldiers' Rest)
- [Event script groups](../modding-scripts/event-script-groups.md): deleting a group's row
- [Event flags and choosing safe ids](../modding-scripts/event-flags-and-free-ids.md): the fight flag block that resets at every load
