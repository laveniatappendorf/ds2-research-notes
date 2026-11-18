# Adding a new bonfire (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- **A bonfire can be added to any map with data alone.** No exe or DLL change. Each one is five pieces: an MSB part, a
  `MapObjectInstanceParam_<map>` row, a `MapObjectBonfireParam` row, a `BonfireName` string and a travel-menu picture.
  Two were built and played: **3650 "Darklurker"** in the Dark Chasm (m40_03) and **10685 "Giant Lord"** in the Memory
  of Jeigh (m20_10) [live B2, B54, B55].
- **An existing save picks the new bonfire up at load.** The game builds its bonfire list from the param, so a save
  went from 77 entries to 78, then 79, with the new ones unlit [live B2 step 1, B55 step 5].
- **The new row needs no lit flag.** Set `bonfire_lit_flag` to 0. The list's level byte is the only lit state; the
  field is a world flag for "area boss beaten" [live A5, A10].
- **You land 1.1 m behind the part along its facing, at the part's own height.** Put the part's y about 0.015 m above
  the floor and measure the floor from the visual mesh [live B6, B2].
- **Main traps:** the MSB needs a writer that adds a model and a part and fixes the indices after them; the list's
  count at EventBonfireManager +0x28 is a **u32**, not a u64; the picture is a **new file** that a "changed files
  only" release step drops; and the bonfire id lives in saves, so a save that last rested at a mod-only bonfire is
  not safe without the mod.
- **Not played yet:** a brand-new character's list, a bonfire-menu trip *to* the Giant Lord's bonfire from another
  bonfire, and a Bonfire Ascetic at a new bonfire.

### What a bonfire is

A bonfire is a chain of ids. Everything hangs off one number, the bonfire id.

| Piece | File | Key | Links to |
|---|---|---|---|
| Object part `o00_0100_<nnnn>` | `map/<map>/<map>.msb` | entity id at part +0xA0 | the instance row with that id |
| Instance row | `Param/mapobjectinstanceparam_<map>.param` | row id = the part's entity id | its `param_id` = the bonfire id |
| Bonfire row | `Param/MapObjectBonfireParam.param` | the bonfire id | `area_id` (the map), `icon_id` (the menu group) |
| Travel-menu name | `menu/Text/<language>/bonfirename.fmg` | the bonfire id | |
| Travel-menu picture | `menu/tex/icon/bonfire_list/ic_list_<9-digit id>.tpf` | the bonfire id | |

[data] The chain is read from the game's own rows; [live B2, B54, B55] it works when rebuilt for a map that had no
bonfire. [Moving a bonfire](moving-a-bonfire.md) covers moving an existing one, which touches only the MSB part.

### Why this was ruled out at first, and what changed

The blocker was the rules file. `MapObjectBonfireParam` lives in the regulation, and a changed regulation would not
boot [live B3]. The loose-params layout removed the blocker: every param becomes a loose file and the regulation keeps
only its non-param files [live B3b]. See [Editing params](../modding-scripts/editing-params.md). Mod Engine added nothing for this; it only
serves the same loose files [inference, no test with and without it].

### What the game does with a new row

[code] The save's bonfire list is the `EventBonfireManager` object, at EventManager +0x58 (EventManager is
`[GameManagerImp +0x70]`). Addresses are Ghidra `FUN_14xxxxxxx` with image base 0x140000000.

- **Layout.** Entries start at +0x20, 0x18 bytes each: u16 id, level byte at +2, row pointer at +8, u64 mask at
  +0x10. The count at **+0x28 is a u32**: the constructor (`FUN_14017daa0`) writes four bytes there. +0x2C is a
  different field.
- **Builder `FUN_14017ec50`** (called from `FUN_140450010`). One entry per `MapObjectBonfireParam` row, level
  cleared to 0. A row is dropped if `FUN_1403bf9b0(area_id)` fails.
- **Save writer `FUN_14017e540`** (from `FUN_1402e3a70`). Per entry: u16 id at +0x1598, lit bit = level & 1 at
  +0x1798, cycle = level >> 1 at +0x1898, u64 mask at +0x1998; 256 slots. The load side restores by id.

So a new row starts unlit, and a save that already exists simply gains it [live B2 step 1: 77 -> 78 entries, 3650 at
level 0]. [Bonfire lit state](bonfire-lit-state.md) has the rest of the lit-state story.

**Rows the builder drops.** A save lists 77 of the game's 83 rows. [data, checked 17 Nov] The six missing rows (650 and
655 on a test map, 62650-62653 on the E3 demo map) share three things: `icon_id` 0, no instance row in any map, and an
`area_id` for a map outside normal play. The code check is on `area_id` (it is not an `AreaParam` lookup: all 85
rows' areas are in it). It turns the `area_id` into a map id and looks that map up, and the six are exactly the rows
whose map has no folder in the game's `map` directory, so the map lookup is the likely reason [code, data; see
[Bonfire lit state](bonfire-lit-state.md)]. Our new rows satisfy all three: a real map, a non-zero `icon_id`, and an
instance row.

### The bonfire row: fields we set

The row is 48 bytes. We copy Shaded Ruins' row 29650 and edit it. All other fields (`Unk01`, `Unk02`, `Unk04`,
`Unk05`, `can_burn_sublime_bone`) stay as the template has them. [data]

| Field | 3650 | 10685 | Notes |
|---|---|---|---|
| `area_id` | 40030000 | 20100000 | The map: m40_03, m20_10. The bonfire warp finds the map through it; the id's own digits mean nothing [live B2, B55]. |
| `icon_id` | 10320000 | 20100000 | The travel menu's group and its area picture `ic_area_<9 digits>.tpf` [inference: the project's reading; every non-zero `icon_id` in the game names an existing area picture]. The Chasm has no area picture, so 3650 uses the Shaded Woods group, as Shaded Ruins does. The memory has its own (20100000). 3650 was listed under Shaded Woods [live B2]. |
| `Unk03` | 10320000 | 20100000 | Equal to `icon_id` in all but three game rows (17665, 19665 and Shaded Ruins' 29650, whose value is 10040000). We set both to the same number, so the live test cannot say which of the two decides the group. Role of `Unk03` unknown. |
| `menu_sort_id` | 83 | 15 | Order in the menu. All 83 game rows have different values; 83 and 15 were unused. Whether duplicates matter is untested. |
| `bonfire_lit_flag` | 0 | 0 | See below. |
| `can_burn_ascetic` | 1 | 1 | As the template. Untested on a new bonfire. |
| `can_burn_effigy` | 1 | 1 | As the Forest's Cardinal Tower row. |
| `fog_door_id`, `Unk10`, `Unk11` | 0 | 0 | Zero, as Shaded Ruins' row. The Forest's rows carry values here; their meaning is unknown. |

**`bonfire_lit_flag` is not the lit state.** [data] Its values equal the per-area "boss defeated" world flags
(`BossBattleParam`'s `map_clear_global_event_flag`), shared by every bonfire in an area: Cardinal Tower and its
neighbours carry 100971, the Last Giant's. Majula, Dragon Aerie and Shrine Entrance carry 0, and a warp to Dragon
Aerie's never-lit bonfire worked with the level byte as its only lit state [live A5]. Reading the field as "the boss an
Ascetic re-arms" is an inference [A10]. The original B2 plan said "a new lit flag"; the test showed none is needed
[live B2].

### The instance row

The row is 40 bytes. We copy the Forest's bonfire row (id 10100655) and set `param_id` to the new bonfire id and
`Unk07` to 7. `map_obj_instance_type_id` stays 100. [data] Of the game's 77 bonfire instance rows, 29 carry `Unk07` 7
and 48 carry 39, even inside one map, so either value looks harmless; ours use 7, as the Chasm's own objects do
[inference]. The row id is the part's entity id: 40030650 and 20100650. Only uniqueness inside the map matters
[inference]; the writer refuses a clash.

### The MSB

[data] A PC MSB2 is a 16-byte header (`MSB `, 1, 0x10) and eight lists, in this order: models, events, points, routes,
layers, parts, `MAPSTUDIO_PARTS_POSE_ST`, `MAPSTUDIO_BONE_NAME_STRING`.

| Part of the file | Layout |
|---|---|
| List | s32 version, s32 count + 1, s64 name offset, s64 entry offsets (count), s64 next-list offset (0 for the last). Then the list name (UTF-16, padded to 8), then the entries back to back. |
| Entry | Every offset inside an entry is relative to that entry. We checked all 28 PC maps: no entry holds an absolute offset. |
| Model entry | +0x08 u8 type (0 map piece, 1 object, 3 collision, 4 navmesh), +0x0A s16 index within its type, name string at +0x20. Objects carry 8 bytes of type data at +0x38. |
| Part entry | +0x08 type, +0x0A index within type, +0x0C u32 **global model index**, +0x10 position, +0x1C rotation (degrees, yaw at +0x20), +0x28 scale, +0x34 draw groups (16 bytes), +0x54 display groups (16 bytes), +0x70 type-data offset, name string at +0x80. Object parts hold the **entity id at +0xA0**. |
| Pose entry | `MAPSTUDIO_PARTS_POSE_ST`: s16 part index at +0x00, s16 bone count at +0x02, s64 bone offset at +0x08 (relative). |

Rules the writer (`tools/msbwrite.py`) follows:

1. **Round trip first.** Every one of the 28 PC maps rebuilds byte-identical from its lists (re-run 17 Nov: 28 maps,
   0 differ).
2. **A new object model** (the Chasm and the memory had no `o00_0100`) goes after the map's last object model, with
   its in-type index set to the object count. Every part whose global model index is at or above the new slot moves up
   by one. In m40_03 the model landed at index 63 of 117.
3. **A new object part** goes after the last object part, with the next free in-type index (m40_03: 37, global part
   540 of 594; m20_10: 534, global 1191, model 210).
4. **Pose entries** at or after the new part's index move up by one. The memory has 66 poses and none had to move; the
   Chasm has none.
5. **Copy, then patch.** The part is copied from the Forest's bonfire part `o00_0100_0002` and the model from the
   Forest's model entry. Name, position, rotation, both group blocks and entity id are patched. The new part name
   must be **as long** as the template's (it is written in place), and a duplicate part name or entity id is refused.
6. **Nothing else changes.** Events, points and every existing part keep their bytes [data: offline tests compare
   them]. Each MSB grew by exactly 256 bytes (118,712 -> 118,968; 366,120 -> 366,376).

**Position frame.** The MSB position is the EventLocation position plus the map's offset event (developer name
`マップオフセット`, "map offset"; three floats at +0x30 of the event entry). m40_03: (-1048, 53, 376). m20_10:
(205.996, -4.0, -175.606). Check: the Giant Lord's part at EventLocation (-73.362, -4.305, 12.639) sits at MSB
(132.634, -8.305, -162.967). [data]

### Placement

The rule from [Moving a bonfire](moving-a-bonfire.md) holds for a new part [live B6, B2, B54, B55]:

- You land **1.1 m behind the part, along its facing, at the part's own height**. Facing is the part's yaw; the
  forward vector is (sin yaw, 0, cos yaw).
- The part's origin sits on the floor. We put it **0.015 m above** the measured floor. A landing under the floor
  falls through and loops the load [live B6].
- Measure the floor from the visual mesh; do not guess it. Both new spots stand on flat floor 1.1 m back.
- **Draw and display groups** come from a part that stands on the same floor: the nearest object with the same
  groups, or the floor's own map piece, found by mesh. The Darklurker's bonfire was drawn and lit [live B2]; the Giant
  Lord's was reported working [live B55].

### Names and the picture

**Name.** `bonfirename.fmg` has the bonfire id as the string id. The new ids were outside every range, so the FMG
writer adds each as its own range (English: 27 -> 29 ranges, 360 -> 362 strings). English reads "Darklurker" and
"Giant Lord". Every other language gets the boss's own official name from that language's `talk/charaname.fmg`
(ids 5061 and 3097), never a translation of ours. All 12 text folders hold both ids [data, checked 17 Nov]. The
first write-up (10 Nov) said "11 languages"; Japanese was added on 16 Nov. See
[Game text and languages](../modding-menus/game-text-and-languages.md). Names load at game start, so restart the game after a change
[live: B1 shows loose menu text loads; later installs of texts needed a restart].

**Picture.** `ic_list_<9-digit id>.tpf` is one picture per bonfire. [data] Our copies are 32,950 bytes: the TPF
signature, one DXT5 texture (256 x 128, from the earlier survey) and the texture name `IC_LIST_<9 digits>` in ASCII
once, at offset 36. To make a new one: copy another bonfire's file and replace that name with the new id, **same
length**. 3650 is Shaded Ruins' picture renamed; 10685 is Place Unbeknownst's picture renamed. It is a **new** file
with no original to compare against, so any release step that ships only files that differ from an original must be
told about it (see Pitfalls). The picture folder now holds 92 files (the game's 90 plus ours). The travel menu opened
and listed the Darklurker with its picture file in place [live B2], but nobody has looked at the picture itself or
tried a bonfire without one [untested].

## How to do it

This is the route the project built. The reference code is `tools/newbonfires.py`, which holds one spec per bonfire
and the build hooks.

1. **Choose the bonfire id and the spot.**
   - The id must not be a `MapObjectBonfireParam` row already (3650 was free; the game's low ids are 650, 655, 2650,
     4650). It becomes the key for the name, the picture and the save.
   - Pick a flat floor with 1.1 m clear behind the spot along the facing. Read the floor height from the visual mesh.
     For the Chasm we used an existing EventLocation point's x and z (row 1010) because its floor was known flat.
   - Work in the EventLocation frame, then add the map offset for the MSB.
2. **Add the MSB model and part** with `msbwrite.add_object(model, model_blob, part, part_blob, pos, rot,
   groups_from, entity)`. Position = EventLocation spot + map offset; rotation = (0, yaw, 0). Check with
   `tools/msbwrite.py roundtrip` first, and re-read the new part (`part_fields`) afterwards.
3. **Add the instance row** to `Param/mapobjectinstanceparam_<map>.param`: copy the Forest's row 10100655, set
   `param_id` = the bonfire id, `Unk07` = 7, and use the entity id as the row id. This is a per-map param: it loads
   with the map.
4. **Add the bonfire row** to `Param/MapObjectBonfireParam.param`: copy row 29650 and set the fields in the table
   above. It is a rules-file param, so it must go out as a loose file (see
   [Editing params](../modding-scripts/editing-params.md)).
5. **Add the name** to `bonfirename.fmg` in every language folder: a new id, the boss's own name per language.
6. **Add the picture**: copy another bonfire's `ic_list_*.tpf`, swap the 17-byte texture name, save as
   `ic_list_<9-digit id>.tpf`.
7. **If scripts need to reach it, add an EventLocation row at the landing.** A script warp (`1:132125`, developer
   name 【マップ】プレイヤーをポリ劇ワープ, "warp the player") names a map and an EventLocation row, not a bonfire
   id. Our row sits at the part's landing, 0.1 m up. In the EventLocation frame, 3650's landing row 990729 is
   (2.788, -41.145, -128.272), yaw 47.6; 10685's row 990728 is (-74.461, -4.205, 12.688), yaw 92.56 [data]. See
   [Warping by script](warping-by-script.md).
8. **Tidy what stands near it.** The Chasm bonfire needed the Shaded Woods white door open (set flag 403020012 with
   `1:130101`, 【フラグ】フラグを設定) and the enemy generator 3 m away removed (`1:131721`, 【ジェネレータ】敵の強制
   削除, "force-delete enemy"), both in the map's load group. The memory needed its corridor enemies removed. See
   [Removing enemies](removing-enemies.md) and [Event script groups](../modding-scripts/event-script-groups.md).
9. **Ship the new picture.** It has no original to differ from.
10. **Test.** Load an existing character and run `tools/b2.py status <id>`: is the id in the list, at level 0?
    Then `tools/b2.py warp <id>`, light it, rest, and check the respawn record.

**When the files take effect.** [live B54] Event scripts and per-map params went in with the game running and loaded
with their maps. Texts (names) load at game start: close and restart [live, install notes]. The bonfire row is a
rules-file param, so plan a restart for it too [inference]. The MSB is read with its map [inference, not tested
alone].

## The two bonfires we built

| | 3650 "Darklurker" | 10685 "Giant Lord" |
|---|---|---|
| Map, part | m40_03, `o00_0100_0000` | m20_10, `o00_0100_0000` |
| Entity id | 40030650 | 20100650 |
| Spot (EventLocation frame) | (3.6, -41.245, -127.53), yaw 47.6 | (-73.362, -4.305, 12.639), yaw 92.56 |
| Floor | mesh -41.259, flat; facing the hole | mesh -4.320, flat arrival platform |
| Draw and display groups from | object `o03_0640_0001` (same groups as the floor pieces) | map piece `m0070_0000` |
| Picture copied from | 29650 (Shaded Ruins) | 10675 (Place Unbeknownst) |
| Landing row (frozen) | EventLocation 990729 | EventLocation 990728 |
| Landing (MSB frame) | (-1045.212, 11.855, 247.728) | (131.535, -8.205, -162.918) |

- **Darklurker.** The Chasm has three sections, each ending in a hole (regions 400000, 400001, 400002). The bonfire
  stands on the Shaded Woods section's floor, facing the hole. The white door `o00_0501_0002`
  (`MapObjectWhiteDoorParam` 400310) is 4.9 m ahead and the drop begins about 8 m on. The door opens on 403020012, the
  section's enemies-dead flag. [data, live B2]
- **Giant Lord.** The bonfire stands 1.1 m *ahead* of the memory's own arrival point (EventLocation 3700000, yaw
  92.56), so you land exactly where the game has always put you. It sits between the two ramps up to the fog. The
  list landing row is the same x and z as the old arrival and 0.037 m higher. [live B55; data]

## Wiring a new bonfire into a mod

These are the choices the Boss Arena made. A different mod can choose differently.

- **Arrival.** The fight list lands at the bonfire's landing row. The map's guard then sets the respawn there
  (`1:132141`, 【マップ】リトライポイント設定, "set retry point"), so a death returns next to the bonfire in one load and the
  Aged Feather returns there too [live B2: both took the player back to the bonfire; B54 and B55 PASS]. After a list
  trip the respawn is a kind-1 record naming the landing row (B54 expected {m40_03, kind 1, 990729}; its result line
  records only the pass). After a rest it is a kind-0 record naming the bonfire (B2 read {m40_03, kind 0, 3650}). See
  [The respawn point](respawn-point.md).
- **Boss rush.** The Darklurker's rush entry still lands in his room (EventLocation 400004). The Chasm's hole is a
  map reload, and the run guard ends a run on a reload with no arrival token [live B38, B39; B54 kept the rule]. The
  Giant Lord's rush entry uses 3700000, the same spot as his landing [code].
- **The old retry went away.** Before the new bonfires, a death in either arena went to Majula and a script sent the
  player back in. With a bonfire in each arena a death respawns at the bonfire like every other arena, so that retry
  group (990014) is no longer built. Its flags 104785 and 104786 stay registered as retired and are never reused
  [live B54, B55; code].
- **Names.** Place Unbeknownst (10675) is no longer renamed "Giant Lord" and shows its own name again in all
  languages [live B55].

## Pitfalls and what failed

1. **Reading the list count as 8 bytes.** [live B2 side find] The first `tools/b2.py`, our Cheat Engine table's list
   finder and four of our other tools (`arenalib`, `arena.py`, `testwatch.py`, the memory reader `pcmem.py`) read the
   count at +0x28 as a u64. Once +0x2C is set (it read 1.8f live on 10 Nov), that is garbage, and the table would
   report "bonfire list not found". All now read a u32. The table's own Lua test now fakes the live float: the old
   table fails 10 of its tests, the new one passes all of them.
2. **Three B2 fears that did not happen.** The plan assumed a new lit flag was needed; it is not. It feared the list
   was built once per character, not from the param at load; step 1 showed an existing character gained the row. It
   kept an escape hatch of "a hard-coded bonfire list in the exe"; nothing needed it.
3. **The old death retry would have fought the new bonfire.** [inference from B2's own test note] Resting at 3650 and
   then using the fight list for the Darklurker in the same session would still send a death via Majula, because that
   retry predated the bonfire. The conflict was planned around and then removed, not seen. Retire any retry that
   sends the player away from a boss room once it has its own bonfire (B54, B55).
4. **Falling under the floor.** [live B6] A part placed at the fog's base height landed the player under the floor and
   looped the load. Measure the floor from the visual mesh and add only about 0.015 m. No fall-through was seen on
   either new bonfire.
5. **The new picture disappears from a "changed files" release.** [code] The release and the test-package builders
   ship files that differ from an original. `ic_list_*.tpf` has none, so both call `newbonfires.NEW_FILES()` and add
   it. The v1.01 package holds both pictures.
6. **The writer says no.** [code] It refuses a part name of a different length, a name that exists, an entity id used
   by another object, a model that is not an object model, and any non-PC MSB.
7. **Checkers that knew the old landing.** [live B56 aside] Two of our arrival checkers still expected the old
   landings for the two specials after B54 and B55. Any test that knows a boss's arrival point must learn the new one.
8. **Not lit on arrival.** [live B2 step 2] A warp to a new bonfire arrives *unlit*, and the player lights it by hand.
   The travel menu follows the lit bit (level & 1) [code]. See [Bonfire lit state](bonfire-lit-state.md) for the Cheat
   Engine route.

## The saves and uninstall risk

The bonfire id lives in saves: the list entry (level byte, by id) and the respawn record {map, kind 0, id} both name
it [code]. A save that last rested at a mod-only bonfire, loaded **without** the mod, names a bonfire the game does
not have. What happens is **untested**.

- **Nearest evidence.** A record of kind 1 that named an event location the game did not have sent every respawn to
  the map's origin, in the void: the 4 Nov death loop [live, 4 Nov session]. The kind-0 case with an unknown bonfire
  id was never tried. The respawn builder (`FUN_140184660`) falls back to Things Betwixt only when bonfires 4650 and
  2650 are both unknown [code]. See [The respawn point](respawn-point.md).
- **The list entry.** An entry with no row to restore into is probably ignored, because the load side restores by id
  [inference].

The practical answer is not to make crossing safe but to keep the two apart: play only new characters with the mod,
never move a character between modded and normal play, and swap the save files back when you uninstall. See
[Saves and installs](../modding-scripts/saves-and-installs.md).

**Freezing the id.** Never renumber a shipped bonfire. The id register keeps the landing rows frozen
(`loc.2010000` = 990728, `loc.4003000` = 990729) because a respawn record can name them. It does **not** hold the
bonfire ids or the entity ids; `tools/ids.py` only checks that a new bonfire id is not already a game row and that the
only new instance rows are ours [code]. Keeping them stable is by convention.

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| A data-placed bonfire works in a map that had none | Warp landed 0.30 m from plan on the floor; it lit; menu listed it; death and Aged Feather returned there | live B2 steps 2-4 (10 Nov) |
| An existing save gains the row | List 77 -> 78, 3650 at level 0 | live B2 step 1 |
| The Giant Lord's memory bonfire works | Reported working; steps played not itemised | live B55 (10 Nov) |
| The list holds 79 entries and both new ones read lit after the CE table | Read back from the save | live B55 step 5 (16 Nov) |
| The list is built from the param; count is a u32 | Constructor writes four bytes; +0x2C read 1.8f | code `FUN_14017daa0`; live B2 side find |
| Lit bit = level & 1; cycle = level >> 1 | Save writer | code `FUN_14017e540` |
| No lit flag needed | Our rows carry 0 and were lit by hand; game rows with 0 exist; the field equals a boss flag | live B2, A5; data |
| The landing rule: 1.1 m behind, part height | Measured on a moved bonfire | live B6 |
| The six dropped rows share `icon_id` 0 and have no instance row; the area check is in code | Read from the params; code per the 10 Nov session | data 17 Nov; code `FUN_1403bf9b0` |
| MSB writer rebuilds all 28 PC maps byte-identical | `tools/msbwrite.py roundtrip` | data, 17 Nov |
| The picture's texture name is in the file once at offset 36 | Read in four files | data, 17 Nov |
| Names exist in all 12 text folders | Read back | data, 17 Nov |
| A new-character list, a travel trip *to* 10685, an Ascetic at a new bonfire | Not played | untested |
| Mod-only bonfire in a save without the mod | Not tried | inference |

## Tools

| Tool | What it does here |
|---|---|
| `tools/newbonfires.py` | The spec for each bonfire (map, part, entity id, spot, yaw, groups, row edits, picture source, door flags, enemies to remove, landing row) and the build hooks: `apply_map`, `pictures`, `bonfire_rows`, `landing`, `NEW_FILES`. |
| `tools/msbwrite.py` | Reads and rebuilds a PC MSB2 from its lists; `add_object` adds a model and a part; `part_fields` reads a part back; `roundtrip` checks all maps. |
| `tools/b2.py` | Live helper (game running): `status [id]` reads the list size, the bonfire's level, the respawn record and the door flag; `warp [id]` makes the game's bonfire warp; `light [id]` sets the level to 1. |
| `tools/paramrows.py` | Adds, replaces and deletes rows in a PC param file; the instance and bonfire rows use it. |
| `tools/fmgwrite.py`, `tools/i18n.py` | Add a new id to an FMG; write the boss's own name in every language. |
| `tools/looseparams.py` | The loose-param layout; `MapObjectBonfireParam` is rebuilt through its rebuild hook. |
| `tools/ids.py` | Register and checks: the new bonfire id is not a game row, only our instance rows are new, landing rows are frozen. |
| `tools/release.py`, `tools/modengine_test.py` | Ship the new pictures. |
| `tools/offline_tests.py` | Checks the MSB writer, the map, the params, the names and the picture. |

## Open questions

- **A brand-new character's list.** Expected to match an existing one (both read the param at load); never played
  [untested].
- **A bonfire-menu trip *to* a new bonfire from another bonfire.** The first one into a memory (10685) is the case
  that might fail; B55's result line did not list which steps were played [untested].
- **An Ascetic at a new bonfire.** `can_burn_ascetic` is 1; whether the area's level goes up is unchecked [untested].
- **The picture.** Is it drawn, and what happens with no file? [untested] Boss portraits instead of copied pictures
  need a TPF writer and a DXT5 encoder [untested].
- **Which of the three conditions drops a row** (map not loaded, `icon_id` 0, no object)? [untested]
- **The role of `Unk03`, `Unk07`, `Unk10` and `Unk11`**, and whether two rows with the same `menu_sort_id` misbehave.
- **The uninstall case itself:** a save at a mod-only bonfire loaded without the mod [untested].
- **Lighting at load.** Commands `1:131631` and `1:131632` ("touch OBJ"; no game script uses them) on the new bonfire
  might light the one in the current map. Left untested by choice [untested]. A brand-new save would still start with
  none lit.
- **More bonfires.** The same pieces would give each Throne of Want fight its own travel entry. Drangleic Castle also
  has one existing bonfire (21665, Under Castle Drangleic) that no arena entry uses, so it could be moved for one
  instead [untested idea].

## Related pages

- [Moving a bonfire](moving-a-bonfire.md): the MSB-only move, the landing rule and the floor survey.
- [The respawn point](respawn-point.md): the record that death, the Darksign, the Homeward Bone and the Aged Feather use.
- [Bonfire lit state, the travel menu and the save's bonfire list](bonfire-lit-state.md): the level byte, the Cheat Engine
  route and why no command lights a bonfire.
- [Editing params](../modding-scripts/editing-params.md): the loose-param layout this depends on.
- [Warping by script](warping-by-script.md): EventLocation warps and arrival points.
- [Removing enemies](removing-enemies.md) and [Event script groups](../modding-scripts/event-script-groups.md): the tidy-up near a new bonfire.
- [Game text and languages](../modding-menus/game-text-and-languages.md): the FMG writer and official names.
- [Saves and installs](../modding-scripts/saves-and-installs.md): what a modded install writes into a save.
  two bonfires.
