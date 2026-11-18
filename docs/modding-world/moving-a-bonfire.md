# Moving a bonfire (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- **A same-map move is a map-file edit and nothing else.** A bonfire is an `o00_0100` object part in the map's `.msb`. Change its position, its yaw and its draw / display groups. No param, event script or text change is needed for the move itself. [live B6]
- **You land 1.1 m from the part, at the part's own height.** With yaw θ, the landing is `part - 1.1 * (sin θ, cos θ)` in (x, z), and you face the bonfire. A bonfire's origin sits on its floor, so the part's y must be the floor height. [live B6]
- **The trap is a part that sits under the floor.** Our first try was 0.17 m low. The player landed under the map, fell about 300 m, died, respawned at the same bonfire and did it again every ~30 s: a loading-screen death loop. [live B6]
- **Do not trust a computed spot.** Four of the six computed spots we tested were in the way (in a fog gate, inside the room, across the only path, behind spiders). We stood where we wanted the bonfire and saved that instead. [live B10, B11, B13]
- **The rest follows the object.** The travel-menu entry, the warp landing, the death respawn and the "travel sets the respawn point" behaviour all moved with the part. Bonfire **names** are only read when the game starts. [live B6, B10, B12, B15]
- In the shipped mod, 33 bonfire parts were moved (9.8 m to 619 m) and every one passed live. [live B6, B10-B19]

### The bonfire chain

| Step | Where | Field |
|---|---|---|
| The object | `map/<map>/<map>.msb`, list `PARTS_PARAM_ST`, model `o00_0100`, part name `o00_0100_NNNN` | position +0x10, rotation +0x1C (yaw is the Y float at +0x20), scale +0x28, draw groups +0x34, display groups +0x54 (16 bytes each) [code `tools/msbwrite.py`, `tools/datapatch.py`; layout read on all 28 PC maps] |
| Its id | the part's object type data | entity id, u32 at +0xA0 [code `tools/msbwrite.py`] |
| The instance row | `MapObjectInstanceParam_<map>`, row id = the entity id | `param_id` = the bonfire id [data] |
| The bonfire row | `MapObjectBonfireParam`, row id = the bonfire id | `area_id`, `icon_id`, `fog_door_id`, `can_burn_ascetic`, `can_burn_effigy`, `can_burn_sublime_bone`, `menu_sort_id`, `bonfire_lit_flag` [data] |
| The name | `bonfirename.fmg`, string id = the bonfire id | the travel-menu text |

Example, the Forest's Cardinal Tower: part `o00_0100_0002`, entity 10100655, `param_id` 10655, `bonfire_lit_flag` 100971. [data]

(The PS3 map files use a different layout: position at +0x0C and big-endian floats. Everything on this page is the PC layout.)

A move changes only the first row of that table. The instance row, the bonfire row, the lit flag, the area and the name all stay, so the game still knows it as the same bonfire. [live B6]

### FromSoftware did it first

Scholar moved one bonfire in exactly this way. Comparing the original game's `m20_24` map file with the Scholar copy, the Undead Crypt Entrance part (`o00_0100_0001`, entity 20240651, bonfire 24655) is 15.94 m from its original place, at the same height. The other bonfire in that map (Undead Ditch) did not move. [data, compared 17 Nov: both files read with the project's MSB reader]

A note from 4 Nov says the instance row, the map's points and events, and the nearby `EventLocation_m20_24` rows did not change either (only an unrelated row shifted by ~1 m). We did not repeat that second half of the comparison. [data, memory note 4 Nov]

So the warp and respawn landing is not stored anywhere else. It is the object. That was the design bet behind the whole arena, written down before any test, and B6 confirmed it. [data 4 Nov, live B6]

### The landing rule

| Observation | Numbers | Test |
|---|---|---|
| Untouched bonfire Place Unbeknownst: part yaw -90 | the player stood 1.10 m **east** of the part, at y -9.79 against the part's -9.805 (+0.015) | B6 |
| Our moved Cardinal Tower: part yaw 90 | the player landed 1.10 m **west** of the part, at the part's height | B6 |
| Cardinal Tower before the move: part yaw 90 | after a warp the player's yaw read 90° (the player faces the part's yaw) | A7 |
| A bonfire placed by data in the Dark Chasm | the player stood 0.30 m from the planned landing, 0.039 above the part height | B2 |

From the two B6 points and the table builder:

```
forward   = (sin yaw, cos yaw)           in (x, z), yaw in degrees as stored at part +0x20
landing   = part - 1.1 * forward         at the part's y
you face  = forward                      so you look at the bonfire
```

To make the player arrive at a spot S facing a direction θ: `part = S + 1.1 * (sin θ, cos θ)`, and `part y = (the y a standing player reads there) - 0.015`. [code `tools/rollout_table.py` (`LAND` 1.1, `SAVED_LIFT` 0.015); live B6]

"You face the bonfire" rests on A7 and on every arena bonfire passing with the player looking at the fog. We never read the facing right after a moved-bonfire landing. [live A7, inference]

### Height: the origin is on the floor

- A bonfire's origin is at its base. The part's y is the floor.
- **0.17 m under the floor: fall-through.** The part was set to the fog's base height (-40.61) while the floor there read -40.30 to -40.44. [live B6, step 1]
- **-40.05, about 0.4 m above the floor:** you land and settle to the floor (-40.44) within 3 s, but the bonfire floats visibly. The tester judged it about 1 m up by eye, at a spot 1 m from the first try. [live B6, step 2]
- **-40.40, 0.04 m above the floor:** sits on the floor, landing right. [live B6, step 3]
- **Floor plus 0.015 from the visual mesh** also worked for the two bonfires we added by data (Dark Chasm, Memory of Jeigh). [live B2, B55]
- Nothing between -0.17 m and 0 m was tried. Stay at or just above the floor. [untested]

The floor must also continue for 1.1 m behind the part (the landing point), and there must be no wall there. [inference from the rule]

### Draw groups

A part is only drawn where its draw / display groups match the area the game has loaded around the player. A moved bonfire keeps its old groups, so at the new place it could be invisible. We copy both 16-byte group sets from a nearby part: the boss's fog gate (an `o00_0500` to `o00_0505` object). [inference for the mechanism; live B6 for the fix]

- Cardinal Tower with the Last Giant's fog's groups: drawn and lit. [live B6]
- All 33 moved bonfires now take the groups of a fog gate of their map. [data `rollout.json`]
- No test reported a bonfire that failed to draw. [live B6, B10-B19]
- The Rotten and Najka first took the groups of a fog 51 m and 42 m away, and both rows passed in B12. We then switched them to the script-tied fog 7-9 m away and rechecked: drawn. [live B12, B15]
- If there is no fog gate nearby, take the groups of the nearest ordinary object. The table code has this fallback; the final table never needed it. [code `tools/rollout_table.py`]

### What moves with the part

| Thing | Result | Test |
|---|---|---|
| Travel-menu entry and warp destination | land at the moved object; the entry stays in its map's menu group because `area_id` is unchanged | B6 |
| Lit | the bonfire shows lit and works (its lit flag is unchanged) | B6 |
| Death respawn | respawns at the moved bonfire | B10 |
| Travelling to it from the menu | sets the respawn point at once, before any rest (17:09:55) | B10, A9 |
| The trainer's own call of the game's warp function | did **not** set the respawn point; the menu does | A3 correction |
| Darksign, Homeward Bone | follow the same respawn point (shown for a Cardinal Tower warp; not repeated at a moved bonfire) | A4; inference |
| The bonfire's "level" byte (NG+ tier for the bosses in its area) | unchanged: it belongs to the bonfire id, not the place | A10 for the byte; inference for the move |

How the respawn point itself works is on [The respawn point](respawn-point.md). How the lit state and the travel list work is on [Bonfire lit state](bonfire-lit-state.md).

### Moving the bonfire away from its usual place has a cost

Anyone wandering the world now finds the bonfire somewhere new, and the old spot is empty. The mod accepts that because every arena bonfire is a boss's door. A mod that wants both can add a new bonfire instead: see [Adding a bonfire](adding-a-bonfire.md). A moved bonfire is safe to uninstall, because its id still exists in the original game. [inference]

## How to do it

1. **Find the part.** In the map's `.msb`, take the object parts with model `o00_0100`. A map can hold several (the Forest has four). Match the one you want by entity id (+0xA0) and then by the `MapObjectInstanceParam_<map>` row's `param_id`, which is the bonfire id. [code, data]
2. **Choose the arrival spot S and its facing θ.** Stand there in the game and read your position. Face the way you want to look on arrival: the bonfire will be in front of you, 1.1 m away.
3. **Work out the part.** `x = Sx + 1.1 sin θ`, `z = Sz + 1.1 cos θ`, `y = (your standing y) - 0.015`. Check that the bonfire does not stand in a path or a fog gate (see Pitfalls).
4. **Write it.** Write the three position floats at +0x10 and the yaw float at +0x20. Copy the 16 bytes at +0x34 and +0x54 from the nearest fog-gate part. With the project's tool:

   ```
   tools/datapatch.py move m10_10_00_00 o00_0100_0002 92.43,-40.40,-146.99 --yaw 90 --groups-from o00_0501_0001
   ```

   That is the Last Giant's bonfire: from (79.843, 1.610, -183.158) to the fog's entrance side. The fog gate `o00_0501_0001` is at (92.768, -40.607, -143.311). The landing is (91.33, -40.40, -146.99), about 4 m from the fog. [live B6]

   The tool keeps `<file>.orig` before its first change, refuses to write unless only the intended bytes differ (B6: 22 bytes changed, all in that part) and re-reads the result. [code `tools/datapatch.py`]
5. **Rename it (optional).** Put the new name in `bonfirename.fmg` under the bonfire id. The name is read once, when the game starts: **build first, then launch** (see Pitfalls).
6. **Test it in this order.**
   - Travel to it from another bonfire. You should land standing, near the fog, with the bonfire drawn and lit.
   - Check the way to the fog is clear, with the bonfire and with nothing else in the way.
   - Die: you should come back to it.
   - Walk off a ledge or fall if the place has one.
7. **If the load hangs on the loading screen,** you are in the death loop: see Pitfalls.

Loose map files are read each time the map loads. A build can run while the game is up, as long as the player is outside the maps being built. Names are the exception. [live B12, B13]

Smithbox should do steps 1-4 as a hand edit: select the part, set Position and Rotation Y, copy the groups from the fog. We did not use it, because the project's tool is checked and repeatable. [untested]

### Where to put it: the survey

For 44 boss rows we did not want to guess floors. The project's trainer (`tools/arenactl.py`, a memory tool that reads and writes the running game) has a survey mode:

- `tools/arenactl.py run --survey` makes the player unkillable and lands at each boss's fog. 2.5 s after each landing it reads where the player stands (two readings 0.5 s apart, within 2 cm, alive) and saves it as `boss_<row>` in `tools/spots_floor.json`.
- **Home** in survey mode means "put the bonfire here, facing the way I am facing". It saves `bonfire_<row>`.
- `tools/rollout_table.py` then prefers a hand-placed `bonfire_<row>` over everything else: it puts the part where the player stood (y - 0.015, same yaw), so the player lands 1.1 m behind it.
- The final file holds 14 floor readings and 30 hand-placed bonfires. [data `tools/spots_floor.json`; live B10-B13]

Without a trainer, read the floor from the map instead: the height of the visual mesh at the spot gave a landing 0.039 m above the part (B2). The scratch scripts for that are not kept. [live B2]

### What we did for all 44 boss rows

| Kind | Rows | What |
|---|---|---|
| Own bonfire moved | 32 | the boss's own bonfire moves to its fog |
| Own bonfire borrowed | 1 | the Pursuer shares Cardinal Tower with the Last Giant, so he takes **Soldiers' Rest** (10660, part `o00_0100_0003`) and moves it 130.7 m (26.6 m of that upwards) |
| Shared with another row | 6 | the four later Throne rows share the first Throne row's bonfire; the second Fume Knight row too; **Vendrick uses Velstadt's moved bonfire (Undead Ditch 24650)**, chosen from a menu in Majula |
| Kept where it is | 3 | Sinh and both Royal Rat bosses: their bonfire already lands within 6 m of the usual fog spot, or the trainer listed them as at-the-bonfire (renamed only) |
| Own bonfire in another map | 2 | the Giant Lord and the Darklurker: see below |

[data `rollout.json`, `tools/rollout_table.py`; live B10-B19]

Distances from the original part to the final one, in a straight line: Mytha 9.8 m (the shortest); the Last Giant 56.8 m (3.7 m from its fog); Ancient Dragon 224.5 m (62.5 m up); Sir Alonne 334.7 m; Lud and Zallen 618.7 m (all in one map file). The height changes run from -106.6 m (Lost Sinner) to +62.5 m (Ancient Dragon). [data, from the final table]

**The Ancient Dragon is the one script-named boss bonfire.** Scanning the original event scripts of the 22 maps with a boss row, the only two bonfire entity ids that any script mentions are 27655 (Ancient Dragon's Shrine Entrance, entity 10270655) and 24655 (Undead Crypt Entrance, entity 20240651, the one Scholar moved). In both maps the same three script groups (4000000, 4002000 and 4003000) pass the entity id to one subroutine; group 4000000 adds four global flag ids (an early survey note tied the Ancient Dragon's to the Ashen Mist Heart). That is probably the bonfire's appearance and effects, but we did not decode it. Moving 27655 by 216 m horizontally (224.5 m in 3D) worked: the arena ran, and the scripts' key-guide and effect groups did not get in the way. [data, scanned 17 Nov; live B18; inference for what the groups do]

### Borrowing a bonfire, and why we never borrowed across maps

A bonfire cannot be in two places. Two pairs of fights share one in their map: Cardinal Tower (the Pursuer and the Last Giant) and Undead Ditch (Velstadt and Vendrick). [data]

- **Pursuer:** borrow another bonfire of the same map. The table claims Cardinal Tower for the Last Giant and gives the Pursuer the nearest unclaimed bonfire of the map, Soldiers' Rest. The hand-placed spot is 5 m back from his fog. B10 passed. [live B10; code `rollout_table.py` (`PIN`)]
- **Vendrick:** first planned to borrow Undead Crypt Entrance (the bonfire Scholar moved). Replaced on 5 Nov by sharing Velstadt's: a menu in Majula sets a flag, the map's load-time script marks Velstadt beaten for that trip and the player starts at the same bonfire. [live B16]
- Soldiers' Rest and Crestfallen's Retreat carry the same `bonfire_lit_flag` (100971) as Cardinal Tower; Undead Ditch and Undead Crypt Entrance share 100975. Lit-flag meaning is on [Bonfire lit state](bonfire-lit-state.md). The Last Giant's flag 100971 is Cardinal Tower's lit flag: never clear it in a mod, or the travel entry drops. [data; live B8]

**Entry bonfires in other maps** are the game's own way into a fight that lives in another map: the Giant Lord's memory (m20_10) is entered from the Forest's Place Unbeknownst (10675), and the Darklurker's chasm (m40_03) from Under Castle Drangleic (21665). The table never borrowed or moved them. A part cannot leave its map file: putting a bonfire into another map means adding a model entry, a part, an instance row and a bonfire row, not editing a position. That was a bigger job, and it was left for later. [code `tools/rollout_table.py`; design]

On 10 Nov we did it, using the same ideas as this page: a new bonfire 3650 "Darklurker" in the chasm before the drop (B2, B54) and 10685 "Giant Lord" in the memory where you arrive (B55). Both passed. Their recipe is on [Adding a bonfire](adding-a-bonfire.md); Place Unbeknownst got its own name back. [live B2, B54, B55]

## Pitfalls and what failed

| What | What happened | Fix | Test |
|---|---|---|---|
| **Part at the fog's base height** | the part was 0.17 m under the floor; the player landed under the map, fell ~300 m, died at y ~-316 and respawned at the same bonfire every ~30 s, stuck on the loading screen | put y at the floor. To get out we wrote the respawn pair to Majula with the trainer; the next respawn loaded Majula (both halves of the pair: map bytes at EventManager +0x164 and bonfire id at +0x16C, with the kind at +0x168 left at 0 for a bonfire) | B6, A4 |
| **Part 0.4 m above the floor** | bonfire floats, the player drops a little | lower it to floor + 0.04 or less | B6 |
| **Computed spot: in the fog gate's way** (the Pursuer) | the bonfire was 1.1 m in front of the arrival point, so it stood across the fog; you could not walk through | the tester stood 5 m back, facing the fog, and saved that | B10 |
| **Computed spot: inside the room** (Smelter Demon, Iron Keep) | the bonfire stood inside the boss room | spot outside, facing the fog (-699.486, 176.362, 648.604 as the standing reading) | B11 |
| **Computed spot: blocked the way in** (Mytha) | across the only path | a spot 6.7 m back | B11 |
| **Computed spot: spiders in the way** (Freja) | area spiders between the bonfire and the fog | new spot, 9 spiders removed by script (see [Removing enemies](removing-enemies.md)) | B13 |
| Where a computed spot is built from | the table puts the bonfire 1.1 m in front of the arrival point, which is exactly where a gate or narrow path usually is | stand where you want it | B10, B11, B13 |
| **Narrow arrival: the Belfry** | the arrival was a tiny gap between a gate and the fog | the gate had to be opened first (a new load-time script group); the bonfire went east of it, facing the fog | B19 |
| **The wrong fog for the draw groups** | the table used the nearest fog model `o00_0500`/`0501` only; Freja's entry fog is an `o00_0502`, the Rotten's and Najka's an `o00_0503` | `FOG_MODELS` now covers `o00_0500` to `o00_0505`; rebuilt and rechecked | B13, B15 |
| **Names did not change** | the build ran with the game up; `bonfirename.fmg` is read only at game start | build first, then launch | B12 |
| Names after a restart | on 5 Nov one restart still showed the old names with the correct 14-name file; later restarts showed the new ones; the cause of that one is not known | none found | B13, B15 |
| A crash at the first Lost Bastille load | looked like a bad map edit | it was the unpacked debug build's DebugManager add-on, not the edit; the same crash happened with original files | B12, B15; see [Crash diagnosis](../modding-scripts/crash-diagnosis.md) |
| Bonfire parts leave the usual place | the old place is empty | by design; add a new bonfire instead if you need both | B6 |

**What did not fail:** the draw groups. Copying them from a fog gate worked every time we tried it.

**Floor numbers to remember:** safe from the floor to about +0.05 m; failed at -0.17 m; floats visibly at +0.4 m. [live B6]

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| A bonfire is an `o00_0100` part; entity at +0xA0 is the instance row id; `param_id` links to the bonfire row | read the files and the writer's docstring | [code `tools/msbwrite.py`], [data] |
| A same-map move needs only the part edit | live: travel menu, landing and lit all followed | B6 |
| FromSoftware moved Undead Crypt Entrance 15.94 m, same height | read both map files | [data] compared 17 Nov |
| Only 2 bonfire entity ids are named by scripts in the 22 boss maps; 1 is a boss bonfire | scan of the original event scripts | [data] scanned 17 Nov |
| Landing 1.10 m from the part, on the side of -forward, at the part's height | two measured bonfires | B6 |
| Player's yaw after a warp = the part's yaw (90°) | trainer read | A7 |
| 0.17 m under the floor: fall-through death loop | the first placement | B6 |
| 0.04 m above the floor: fine; about 0.4 m above: floats | placements 3 and 2 | B6 |
| Floor + 0.015 from the visual mesh: fine | two new bonfires | B2, B55 |
| Draw / display groups copied from a fog gate: drawn | live | B6, B15 |
| A death respawns at the moved bonfire | six fights, deaths before and after the kill | B10 |
| The menu trip sets the respawn point | respawn pair read at arrival, 17:09:55 | B10, A9 |
| The warp function called by the trainer does not set it | pair unchanged across three warps | A3 (correction) |
| Computed spots blocked or were in the room | four live failures | B10, B11, B13 |
| All 33 moved bonfires work | 44 boss rows passed live | B10-B19 |
| The script-named Ancient Dragon bonfire can be moved | live | B18 |
| Names load only at game start | build while up, then restart | B12, B15 |
| New bonfires in other maps work | Darklurker, Giant Lord | B2, B54, B55 |

## Tools

| Tool | What it does for this topic |
|---|---|
| `tools/datapatch.py move <map> <part> <x,y,z> [--yaw <deg>] [--groups-from <part>]` | moves a PC MSB part, optionally sets the yaw and copies both group sets. Keeps `.orig`, refuses unintended byte changes. `status` lists changed files, `restore all` puts everything back |
| `tools/rollout_table.py` | computes the table: which bonfire each row moves, borrows or keeps; the landing rule (`LAND` 1.1, `SAVED_LIFT` 0.015); the groups source; the removal list |
| `tools/buildmod.py` | applies the moves per map and batch, and builds the bonfire-name file with every arena name |
| `tools/arenactl.py run --survey` | floor readings and the Home key for hand-placed spots |
| `tools/spots_floor.json` | the saved `boss_<row>` floor readings and `bonfire_<row>` hand-placed spots |
| `tools/bosstable.py` | finds each boss's fog gate and reads the original bonfire names (`FOG_MODELS` o00_0500..o00_0505) |
| `tools/fmgwrite.py` | writes the travel-menu names |
| `tools/msbwrite.py` | rewrites a whole PC MSB (round-trips all 28 maps byte for byte); only needed when you **add** a part |
| `tools/newbonfires.py` | the new-bonfire builder (its landing check uses the same 1.1 m rule) |

## Open questions

- **How low is too low?** Failure at -0.17 m, success from about floor level. Nothing in between was tried. [untested]
- **Resting and the Ascetic at a moved bonfire.** B6's pass list included "you can rest at it", but only "lit" was recorded. Resting is not logged as its own check, and a Bonfire Ascetic at a moved bonfire was never run. [untested]
- **A Homeward Bone at a moved bonfire.** It follows the respawn point (A4), so it should work. B6 listed it as not checked, and no later test checks it. [inference]
- **Which way you face on landing.** We rely on the part's yaw (A7), but never read the facing right after a moved-bonfire landing. [inference]
- **Does the draw-group source matter?** Every bonfire drew with the groups of a fog gate, some of them far from the new spot. We never tested a bonfire with the wrong groups. [untested]
- **Getting out of the death loop without a trainer.** Fix the map file and let the game retry (it re-reads maps at each load), or quit and fix the file before loading. Neither was tried. [untested]
- **Why did Scholar move the Undead Crypt Entrance?** We know it moved 15.94 m and that scripts name it. We do not know why.
- **Smithbox** for the move. [untested]

## Related pages

- [Adding a bonfire](adding-a-bonfire.md): a new part in a map that has none (the Darklurker and the Giant Lord).
- [The respawn point](respawn-point.md): death, the Darksign, the Homeward Bone and the respawn pair.
- [Bonfire lit state](bonfire-lit-state.md): lit flags, the travel list and the save's bonfire list.
- [Removing enemies](removing-enemies.md): clearing the enemies near a new bonfire.
- [Editing params](../modding-scripts/editing-params.md): why none of this needed a param edit.
- [Mod Engine for DS2](../modding-scripts/mod-engine-ds2.md): serving loose map files.
- [Game text and languages](../modding-menus/game-text-and-languages.md) and [Text length limits](../modding-menus/text-length-limits.md): bonfire names.
- [Live-testing notes](../modding-scripts/live-testing-notes.md): the survey and the watcher.
