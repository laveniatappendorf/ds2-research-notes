# GeneratorParam fields: what each did in our tests (PC Scholar 1.0.3.0, 2026-11-18)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 18 November 2026 from the Boss Arena project's tests.

Tags in square brackets say how we know a claim: `[live B8]` = seen in the game (the id points to the project's test
log, see [Home](../how-to-read.md#the-modding-pages)), `[code]` = read in the exe or in our build tools, `[data]` = read in the game files,
`[inference]` = reasoned, not proven, `[untested]` = an idea only.

## Summary

- Every map has a `GeneratorParam_<map>` param: one 0xA0-byte (160-byte) row per enemy spawner, 2,458 rows in the
  game's 28 maps. [data] Only one field changed the game when we edited it alone: **`SpawnLimit` and
  `SpawnLimitGameClear` set to 255** (no limit) brought a killed boss back three wins in a row. [live B8]
- Two results surprised us. **`ActivationEventID1` did not quiet an ordinary enemy** [live B7], and removing one enemy
  of an **`AggroGroup`** left its mates attacking, because the group links them by number. [live B7]
- **`AppearanceEventID` is two things**: a small number (2 to 8) read as a minimum bonfire intensity, or an event flag.
  A **condition byte beside it, `Unk0D` (+0x0D)**, seems to say which, and for a flag whether the enemy is there
  while the flag is on (7) or off (8). [data] [inference] Vendors stood in Majula once their flag was set and the map
  reloaded. [live B28] [live B30]
- **`HostileEventID1` is an NPC's hostile flag.** Setting the vendors' hostile flags turned every vendor hostile.
  [live B42 run 1] `Unk84` says what kind of NPC spirit a script-made generator is [data]. Bit 7 of the byte at +0x48
  and `MapObjectBonfireID` (+0x4A) are read by the spawn check and the intensity code. [code]
- The main trap: the community definitions give the right offsets, but a few names describe what a field *looks like*
  it does. Test a field on one spawner before building on it.

### Where the rows live

- `GeneratorParam_<map>` is a per-map param, so a loose copy replaces it and the game reads it at each load of the
  map, with no restart. [live B8] See [Editing params](../modding-scripts/editing-params.md). A row's id is the generator id the event
  scripts use (`1:131721(generator, 0)`, see [Removing enemies](removing-enemies.md)).
- A row names its enemy through `GeneratorRegistParam` (+0x08): a row id in `GeneratorRegistParam_<map>`, whose
  `EnemyParamID` is the enemy. Its position is the `GeneratorLocation_<map>` row with the **same id as the
  generator**; positions are map-local, so add the map's offset from its MSB. [data]
- Names and offsets are the community's (Smithbox's DS2S definitions). We read the raw bytes at every offset in the
  table below on four rows of the Forest's file and compared them with the decoder: no mismatch. [data, 18 Nov] The
  number in an `Unk` name is the offset in hex (`Unk84` is at +0x84), with one exception we saw (`Unk71` is at +0x6E).

### The fields

Row size 0xA0, little-endian. "Sure?" is the best evidence we have for the line, details in the sections below.

| Offset | Field | What it does in our tests | Sure? |
|---|---|---|---|
| +0x00 | `SpawnLimit` (u8) | the spawn limit in NG; 255 = no limit; 0 = the game's default (1 for bosses, 12 for ordinary enemies) | live, code |
| +0x01 | `SpawnLimitGameClear` (u8) | the same limit from an intensity total of 2 up (NG+, or a bonfire raised by one Ascetic) | code |
| +0x03 | `AggroGroup` (u8) | spawners with the same number in one map wake together | live |
| +0x04 | `DeathEventID` (u32) | the flag that records the enemy's death: set on 105 rows, nearly all NPCs, and on none of the fight generators | data |
| +0x08 | `GeneratorRegistParam` (u32) | the row in `GeneratorRegistParam_<map>` that names the enemy | data |
| +0x0C to +0x0F | `Unk0C` to `Unk0F` | condition bytes: `Unk0D` goes with +0x10, `Unk0E` with +0x14, `Unk0F` with +0x18 | data, inference |
| +0x10 | `AppearanceEventID` (u32) | a number 1 to 8 = minimum bonfire intensity; a flag = whether the enemy exists | live, data, inference |
| +0x14, +0x18 | `PreviousHostileEventID1`, `2` (u32) | on a boss row, the intro flag; on a flagged row, a further condition or a cycle number | data |
| +0x48 bit 7 | `Unk48_7` | the exemption bit in the spawn check; the switch it needs is off | code, live |
| +0x4A | `MapObjectBonfireID` (u16) | the bonfire whose Ascetic cycles feed this generator's intensity total | code |
| +0x5C, +0x60 | `ActivationEventID1`, `2` (u32) | on a boss row, the fight flag; on an ordinary enemy, no visible effect | live, data |
| +0x64, +0x68 | `HostileEventID1`, `2` (u32) | an NPC's hostile flag | live, data |
| +0x84 | `Unk84` (u8) | the kind of NPC spirit a script makes from this generator | data |

Never touched, so unknown: `Unk02` (+0x02), `LeashDistance` (+0x4C, a float), `AIThinkID` (+0x58),
`ModelMask` (+0x85), `ItemLotID1` / `2` (+0x88, +0x8C), `GraveEventID` (+0x90), the `LocationEventID` slots and the
other `Unk` fields.

### SpawnLimit and SpawnLimitGameClear

- **255 in both is "no limit".** A live probe of the generator objects read the limit the game builds from the row: row
  255 gave 0 (none), row 0 gave 1 on the two Forest boss generators and 12 on ordinary ones, row 1 gave 1. [live B8]
- With both fields at 255 the Last Giant came back on three wins in a row with no other change. [live B8] We then wrote
  255 to 131 generators in 22 maps (44 boss fights and their summoned adds) and every fight played. [live B10 to B19]
- **Which byte applies** is picked when the generator is built, by `FUN_140412e70` (RVA `+0x412e70`): the intensity
  total (the bonfire's Ascetic cycles plus the character's cycle, clamped to 8) below 2 takes `SpawnLimit`, 2 or more
  takes `SpawnLimitGameClear`. [code] The mod writes both, so neither matters. Full account: [NG+ near boss
  arenas](ng-plus-differences.md). The kill count and the spawn check are on [Bringing a boss back](respawning-bosses.md).
- In the vanilla files `SpawnLimit` is 0 on 2,104 rows, 1 on 182, 12 on 62, 24 on 5 and 255 on 105; the other byte is
  255 on exactly the same 105. No row has 255 in only one. [data]
- 255 is not only for player-summon rows. 68 of the 105 carry `Unk84` 1, 3 or 6; the other 37 are enemies in three
  maps (m20_21 6, m20_24 24, m40_03 7) that the game simply never limits. [data]

### AggroGroup

- **Same number, same map = linked.** Two Forest enemies (generators 2830 and 2820, `AggroGroup` 71) were 17 m and
  38 m from the landing. Taking one out left the other, and the two attacked together. [live B7] A horse knight at the
  Looking Glass Knight (generator 3010, group 75, 34 m out) attacked in the first run, with its three mates already
  removed. [live B14] The link was not isolated there. [inference]
- 373 groups hold two or more generators, 1,124 of the 2,458 rows. The largest is m10_23's group 43, 21 rows: the
  Skeleton Lords' skeletons and two of the three Lords. [data]
- **The number means nothing across maps.** 240 is Freja's group in m10_14, the Looking Glass Knight's in m20_21 and the
  Guardian Dragon's in m10_15; close a group over one map at a time. [data]
- A fight's adds do not always share the boss's group. Freja's generator 803 is group 240 and her 12 spiders are group
  100, but all carry her fight flag in `ActivationEventID1`. [data] Look for both.
- 0 and 255 are treated as "no group" by our removal rule. [code] The data: 0 on 28 rows, 255 on 4, one of them the Last
  Giant's boss row. [data] Not tested live. [inference]

### ActivationEventID1 and 2

- **Not an AI switch for ordinary enemies.** Generator 2830 was given a Forest fight-block flag nothing else uses; the
  game loaded the edit (only the new row was found in memory) and the enemy still walked about, then attacked with its
  mate. Giving the same flag to 2820 changed nothing. [live B7]
- **On a boss row it holds the fight flag.** The Last Giant's generator 857 has 110020080, which turns on when you cross
  the fog. [data] 43 of the 44 boss rows' first generator carries one; the Ancient Dragon's is 0. [data] Freja's
  spiders carry hers too. [data] This is how our tools find a fight's generators ([Bringing a boss
  back](respawning-bosses.md)). What the field does there, we never tested. [untested]
- The Giant Lord stood still because **his boss script** wakes him within 30 m or when he is hurt, not because of a field.
  [live B19] See [Removing enemies](removing-enemies.md).
- `ActivationEventID2` is set on 110 rows. The Lost Sinner's two NG+ pyromancers (8100, 8101) use a flag a script sets at
  about 70 % of her HP. [data, unseen: NG+ only] See [NG+ near boss arenas](ng-plus-differences.md).
- The Burnt Ivory King's ally (generator 797) has 537020024 in `ActivationEventID1` and 537020030 in
  `ActivationEventID2`. We do not know whether either matters. [data] [untested]

### AppearanceEventID and the condition bytes

This field holds one of three kinds of value, which makes it easy to misread.

| Value | Rows | Reading |
|---|---|---|
| 1 to 8 | 149 | minimum bonfire intensity ("at least N"); the community calls 2 NG+, 3 NG+2 and 8 NG+7. Counts and the intensity code: [NG+ near boss arenas](ng-plus-differences.md) |
| 100 and up | 316 | mostly an event flag (307 rows have 6 digits or more); nine rows hold a 4 or 5 digit number instead (6520 and 6020 on the Forest's two bosses, 24000 on three Brightstone Cove rows) |
| 10, 30, 50 | 115 | not tiers: those enemies stand in NG |

**The flag is the vendors' move flag.** Six vendors have a Majula generator whose `AppearanceEventID` is a global
flag. The setup sets it; **reload Majula**; the vendor is there. [live B28] [live B30] [live B42] The flag ids and the
full recipe are on [NPC menus](../modding-menus/npc-menus.md).

**The condition bytes.** Four small bytes sit just before the flag fields (+0x0C to +0x0F). Three of them line up with
+0x10, +0x14 and +0x18 and follow the kind of value in their field: when the field is 0 the byte is 0 or 1, when it
holds 1 to 8 the byte is 22 (or a neighbour), when it holds a flag-sized value the byte is mostly 7 or 8. [data]

| Field | Byte | What the byte holds |
|---|---|---|
| `AppearanceEventID` +0x10 | `Unk0D` +0x0D | field 0: 0 (196 rows) or 1 (1,682). Value 1 in m40_03: 0 or 1 (23 rows). Value 1 elsewhere: 23 (7 rows) or 26 (4). Values 2 to 8: 22 (108 rows) or 25 (7). Values 30 and 50: 1 (106 of 107 rows); value 10: 4 (7 of 8 rows). Flag-sized values (100,000 and up): 7 (186), 8 (84), 2 (18), 11 (8), 13 (11). The nine 4-5 digit values: 13 |
| `PreviousHostileEventID1` +0x14 | `Unk0E` +0x0E | field 0: 0 (2,303 of 2,305 rows). 1 to 8: 22 (3 rows). Flag-sized: 7, 8, 13, 2, 11, 20 or 31 |
| `PreviousHostileEventID2` +0x18 | `Unk0F` +0x0F | field 0: 0 (2,391 of 2,391). 1 to 8: 22 (6 rows). Flag-sized: 7, 8, 13 or 2 (one row 0) |

Our reading of the flag bytes, all [inference]:

- **7 = present while the flag is on; 8 = present while the flag is off.** What fits it:
  - 12 flags appear in `AppearanceEventID` of rows with different bytes (a vendor's Majula row and its home row, for
    example), and in every one of the 12 the two bytes are 7 and 8; [data]
  - all 29 rows whose `AppearanceEventID` equals their own `DeathEventID` carry 8, which only makes sense as "present
    while not dead" (Saulden and Maughlin are among them); [data]
  - live: Majula vendors (7) appear when the flag is set [live B28] [live B30]; the Burnt Ivory King's ally 797 (8) is
    there with its flag 537000029 clear: the map's script clears 537000026 to 29 at every load and he comes. [live 30 Oct,
    trainer]
  - Switching 537000029 on by hand did not remove him [live 30 Oct, trainer], so the condition seems to be read when
    the generator is built or spawns, not every frame.
- **22 = compare the intensity total with the number.** It sits only on rows that hold 1 to 8, in all three condition
  slots. The three generator methods that test the total (`>=`, `<=`, `==`, see [NG+ near boss arenas](ng-plus-differences.md))
  may be 22, 23 and 25 or 26. [code] [inference]
- **Values 30 and 50 carry the same byte as unconditional rows (1)**, which fits them not being tiers. [data]
- A worked pair (the numbers in brackets are the byte):

| Row | `AppearanceEventID` | `PreviousHostileEventID1` | `PreviousHostileEventID2` | `Unk0C` |
|---|---|---|---|---|
| Melentia, Majula (row 260) | 102521 (7) | 103711 (8) | 100971 (7) | 3 |
| Melentia, Forest (row 260) | 102521 (8) | 100971 (8) | 103711 (7) | 4 |

102521 is her move flag, 103711 her Forest hostile flag, 100971 the Last Giant's map-clear flag. The Majula row reads
"move flag on, hostile flag off, Last Giant dead", which matches her needing 100971 in a clean install. [live B41 run 3,
B43, B49, not isolated] [inference] The Forest row's last condition (hostile on) reads oddly, and `Unk0C` differs
between the rows; it may say how the three conditions combine. [untested]

**Where our pages seem to disagree.** [NPC menus](../modding-menus/npc-menus.md) and [NG+ near boss arenas](ng-plus-differences.md) read a
flag in `AppearanceEventID` as "present once it is on" (the vendors; the Wharf's phantom 4550, which has 7 and a 2
beside it). [Removing enemies](removing-enemies.md) describes the Burnt Ivory King's ally (generator 797) as present
whatever the flags say: his own `AppearanceEventID`, 537000029, is one of the flags the map's script clears at every
load. Read alone, the two do not fit. With the byte they can both be right: the vendors and 4550 carry 7, the ally
carries 8. That is an inference from the pattern, and the ally's gate stays open (see Open questions).

### The two PreviousHostileEventID fields

- On the Last Giant's boss row `PreviousHostileEventID1` is 110000082, the intro flag, set when you first cross the fog.
  [live A2] [data] The Pursuer's row holds his own (110000087). [data]
- On a vendor's Majula row one slot holds the vendor's home-hostile flag and, for Melentia, Rosabeth and Carhillion, the
  other holds a further global flag. [data] On the Wharf's phantom 4550 it holds a 2 beside a flag. [data] See
  [NG+ near boss arenas](ng-plus-differences.md).
- The field name says "hostile", but the roles above are not all about hostility. [data]

### HostileEventID1 and 2, and DeathEventID

- **`HostileEventID1` is the NPC's hostile flag.** For a vendor's home row it is the vendor's flag `b` at home; for
  the Majula row, flag `b` in Majula (Gilligan 103552, Melentia 103712, and so on). [data] Our first Majula setup set the
  vendors' flags `a` and `b` as if they were "met" flags, and **every vendor turned hostile**. [live B42 run 1] A fix
  clears the 17 flags at every load. [built and installed 8 Nov; no logged retest] Details: [NPC menus](../modding-menus/npc-menus.md).
- Not isolated: whether the generator or the talk script reads the flag, and whether `HostileEventID1` alone is enough.
  [untested]
- `HostileEventID1` is set on 221 rows and `HostileEventID2` on 172. `HostileEventID2` sits 0 to 2 above
  `DeathEventID` on 62 of the 74 rows that have both. [data]
- `DeathEventID` is set on 105 rows in 22 maps; 98 have enemy ids in the 700000s (the NPC characters). Gilligan's is
  104050, Saulden's 104140, Licia's 104320. [data] On Saulden's and Maughlin's rows it equals `AppearanceEventID`. That
  the game sets it on death is our reading. [inference] It is 0 on every fight generator. [data]

### Unk84

- **The kind of NPC spirit** that an event script makes from this generator with `1:131741` (【ジェネレータ】NPC霊体生成,
  "create an NPC spirit"): 1 white sign, 3 shade sign, 2 dark-spirit invader, 0 the Dragon Shrine's spirit. [data] This
  comes from a separate survey of the event scripts (4 Nov); the developer names in the `.edd` agree on all 157 calls
  to the command (119 generators).
- In the 28 maps: 0 on 2,293 rows, 1 on 52, 2 on 52, 3 on 15, 4 on 6, 5 on 1, 6 on 39. [data]
- **4, 5 and 6 are not in our sources.** 4 and 5 appear only in Majula (Saulden, Maughlin's four rows, Rosabeth's row);
  6 on 39 rows in the two Rat boss maps, all `SpawnLimit` 255. [data] [untested]
- Our removal rule never removes a row with `Unk84` not 0. [code] Whether `1:131721` stops a spirit was never checked.
  [untested] See [Removing enemies](removing-enemies.md).

### Unk48 bit 7, the exemption

- The spawn check lets a generator spawn despite its kill count when bit 7 of the row byte +0x48 is set **and** bit
  0x10 of a manager byte (`EnemyGeneratorManager` +0x305) is set. [code] That manager byte read 0x26 in every probe,
  so the switch is off. [live B8]
- Bit 7 is set on 1,737 of the 2,458 rows (71 %) and is **clear on all 131 generators we unlimited**. [data] So even
  if something turned the switch on, the boss rows would not be exempt. [inference] What sets the manager bit, we do not
  know. [untested]

### MapObjectBonfireID

- A u16 at +0x4A, non-zero on every row. The generator's constructor (`FUN_140411a40`, RVA `+0x411a40`) looks the
  bonfire up and keeps its index; the intensity total of that bonfire gates the row and picks the limit. [code]
- It is not always the arena's bonfire. The Pursuer's rows name Cardinal Tower (10655), not his borrowed arena
  bonfire (10660). [data] Details and the bonfire byte: [NG+ near boss arenas](ng-plus-differences.md),
  [Bonfire lit state](bonfire-lit-state.md).

## How to do it

1. **Read the row before you trust a name.** Decode `GeneratorParam_<map>` with the community definitions, then look at
   the generator's `GeneratorRegistParam` row (the enemy) and `GeneratorLocation` row (the place). [data]
2. **Bring a boss back:** write 255 to `SpawnLimit` and `SpawnLimitGameClear` on the boss's generator and on any
   summoned adds. Reload the map. Recipe: [Bringing a boss back](respawning-bosses.md).
3. **Find a fight's generators:** rows whose `ActivationEventID1` is the fight flag, then rows with the boss's
   `AggroGroup` in the same map, then script-woken adds (`ActivationEventID2`, scripts). Do not rely on one of the three.
4. **Quiet an area:** do not edit `ActivationEventID1`. Delete the generator's enemy by script, and delete the whole
   `AggroGroup` with it. Recipe: [Removing enemies](removing-enemies.md).
5. **Stand an NPC in a map:** find its generator in that map's `GeneratorParam`; its `AppearanceEventID` is the flag; set
   the flag, clear the hostile flags, **reload the map**. Check the row's condition byte first: by our reading 7 wants
   the flag on and 8 wants it off. Recipe: [NPC menus](../modding-menus/npc-menus.md).
6. **Watch an enemy live:** the live generator object keeps its limit at +0x8D and a pointer to its param row at +0x58; a
   read-only probe compares them with the file. [code]

## Pitfalls and what failed

| What went wrong | Why | Fix | Test |
|---|---|---|---|
| A flag in `ActivationEventID1` did not quiet an enemy | the field is not an AI switch for ordinary enemies; the earlier idea came from boss rows and from the Giant Lord, whose own script wakes him | delete by script | B7, B19 |
| Removed one enemy, both attacked | same `AggroGroup` | remove the group | B7 |
| 4 Nov note: "`AppearanceEventID` is not a flag, more like a spawn animation" | its example, 6520, is one of nine 4 or 5 digit values that are not flag-sized; most of the 316 values from 100 up are flags | the 5 Nov survey and the vendor tests | 5 Nov data, B28, B30 |
| Value 1 read as "NG only", then as "always" | the value-1 rows are two kinds: the 23 in the Dark Chasm carry no condition (0 or 1), the other 11 carry 23 or 26, and so do the value-1 halves of the twin pairs (the Copse's torturers 5010-5013 beside 5014-5017; the Chariot's 8000 beside 8001), whose value-2 halves carry 22 or 25 | read the byte too; the twins may be "at most 1" against "at least 2" and so never stand together | data |
| Vendors hostile after setup | the "met" flags were the hostile flags (`HostileEventID1` and its partner) | clear them at every load | B42 run 1 |
| 255 read as "player-summon row" | 37 of the 105 rows are ordinary enemies in m20_21, m20_24 and m40_03 | read `Unk84` as well | data |
| Fight adds missed by a search | the Lost Sinner's pyromancers wake on `ActivationEventID2`, so a search on `ActivationEventID1` misses them; Freja's spiders carry her fight flag but not her `AggroGroup` | search both flags and the group | 5 Nov data |
| `SpawnLimit` 255 on a NG+ row did not bring it into NG | a limit is not an appearance gate (Flexile's two Shadows) | none needed | B13, weak: nobody looked for them on purpose |

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| Row layout and offsets as in the table | raw bytes at each offset against the decoder on four rows | data, 18 Nov |
| 255 gives live limit 0, 0 gives 1 or 12, 1 gives 1 | probe of the generator objects | live B8 |
| 255 in both fields brings a boss back, three wins in a row | live | B8 |
| 131 generators in 22 maps unlimited, 44 fights play | live batches | B10 to B19 |
| `SpawnLimit` below a total of 2, `SpawnLimitGameClear` from 2 | disassembly | code `+0x412e70` |
| Vanilla limits: 0 on 2,104 rows, 1 on 182, 12 on 62, 24 on 5, 255 on 105 | read the 28 files | data, 18 Nov |
| `ActivationEventID1` edit did not quiet generator 2830 / 2820 | live: edit found in memory, enemies attacked | B7 |
| Boss rows carry the fight flag in `ActivationEventID1` (43 of 44) | read the first generator of each fight | data, 18 Nov |
| `AggroGroup` links enemies (71: 2830, 2820) | live: both attacked | B7 |
| 373 groups of 2 or more cover 1,124 rows; numbers repeat across maps | read the files | data |
| A flag in `AppearanceEventID` puts a vendor in Majula | live: set, reload, present | B28, B30, B42 |
| The ally 797 is present with its flag clear; flipping it live did not remove him | live, trainer session | 30 Oct |
| Condition bytes follow the kind of value (0, 1 to 8, flag) | read all 2,458 rows | data, 18 Nov |
| 7 and 8 are the two senses of a flag | pattern: 12 shared flags, 29 death-flag rows; live B28, B30 and 30 Oct fit | inference |
| `HostileEventID1` flags turned every vendor hostile | live: all vendors hostile after the setup | B42 run 1 |
| `Unk84` = kind of NPC spirit (0 to 3) | event scripts and `.edd` names, 157 calls | data, 4 Nov, separate survey |
| Exemption needs bit 7 of +0x48 and manager bit 0x10; the manager byte is 0x26 | disassembly; probe | code, live B8 |
| Bit 7 is clear on all 131 fight generators | read the rows | data, 18 Nov |
| `MapObjectBonfireID` names the bonfire for the intensity total | constructor `+0x411a40` | code |

## Tools

All in `tools/`:

| Tool | What it does here |
|---|---|
| `bosstable.py` | `gens_of(map)` decodes every generator of a map with its enemy id and position; finds fight generators by `ActivationEventID1`; `EXTRA_GENS` for those it misses |
| `rollout_table.py` | the removal rule (`AggroGroup` closure, `Unk84`, `SpawnLimit` 255, NPC type) and the fight lists to unlimit |
| `datapatch.py` | `param <param> <row>:<field>=<value>` writes a field into a loose param, keeps a `.orig`, reads it back |
| `buildmod.py` | writes 255 into both limits of every listed generator |
| `slice_lastgiant.py` | `probe` reads a generator object's live limit (+0x8D), param pointer (+0x58) and the manager byte |
| `paramdef.py` | the shared row decoder (the community definitions); it rounds floats to 4 places |

## Open questions

- **Does the condition byte decide what `AppearanceEventID` means?** The pattern is strong but only a few cases were
  seen live (the vendors, the ally 797, Melentia's 100971). A cheap check: on one visible spawner, put a never-set
  map-local flag in `AppearanceEventID`, try `Unk0D` 7 then 8, reload, and see which makes the enemy absent; then set
  the flag before the load. [untested]
- **Generator 797's gate.** By its byte (8) it is present while 537000029 is off, which matches the 30 Oct trainer
  session; the "present once on" reading came from the vendors. Setting the flag *before* the load would settle it.
  [untested]
- **What is `Unk0C`?** It differs between a vendor's two rows and may combine the three conditions. [untested]
- **What do the other condition codes mean:** 23, 25, 26 (twins), 2, 11, 13, 20, 31, and 4 and 5 beside values 10 and
  50? [untested]
- **Do the value-1 and value-2 twins stand together in NG+?** The bytes suggest not. [inference] Nobody has looked.
  [untested]
- **What does `ActivationEventID1` do on a boss row** if its flag is never set? [untested]
- **Does `HostileEventID1` alone make an NPC hostile,** and what reads it? [untested]
- **What sets the manager bit that switches the exemption on?** [untested]
- **What are `Unk84` 4, 5 and 6?** [untested]

## Related pages

- [Bringing a boss back](respawning-bosses.md): the kill count, the spawn check, `SpawnLimit` 255
- [Removing enemies](removing-enemies.md): `ActivationEventID1` (not an AI switch), `AggroGroup`, `Unk84`, the removal commands
- [NG+ near boss arenas](ng-plus-differences.md): `AppearanceEventID` tiers, `MapObjectBonfireID`, the limit function
- [NPC menus](../modding-menus/npc-menus.md): the vendors' `AppearanceEventID` and `HostileEventID1` flags
- [Editing params](../modding-scripts/editing-params.md): loose per-map params and how to write a field
- [Event script groups](../modding-scripts/event-script-groups.md) and [Script command reference](../modding-scripts/script-command-reference.md): `1:131721`, `1:131741`
- [Event flags and choosing safe ids](../modding-scripts/event-flags-and-free-ids.md): which flags are readable when
- [Bonfire lit state](bonfire-lit-state.md): the bonfire byte behind the intensity total
