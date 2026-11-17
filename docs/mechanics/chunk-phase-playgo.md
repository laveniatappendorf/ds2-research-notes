# ChunkPhaseParam and the PlayGo doors (PC Scholar)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Written 2026-11-03. The author asked what is known about `ChunkPhaseParam`. Builds checked: the Network Test, v1.00, PS3
Scholar, PC vanilla 1.12 and PC Scholar 1.0.3.0. Tool: `tools\playgo_pc.py` (everything below).

## Short answer

- **Publicly, almost nothing.** SmithBox and DSMapStudio describe it as "parameters related to an unknown system",
  with two rows named "UNKNOWN" and four unnamed fields (three s32, one f32). A web search found nothing else.
- **It exists only in PC Scholar**, the current-generation codebase. No PS3 build (PS3 Scholar included), no vanilla
  PC build and not the Network Test has it.
- **It looks like the phase table of an install-while-playing system**, the kind PS4 calls PlayGo: the game is
  installed in chunks, and doors at the edge of the opening areas stay shut, saying "Cannot pass", until the chunk
  ("phase") behind them is installed, then say "Path opened". A companion param, `MapObjectPlayGoParam` (internal type
  MAP_OBJECT_PLAY_GO_DOOR_PARAM), defines those doors, and the exe has the classes that run them. On a full PC
  install the doors are presumably never shut; that is not tested.
- It has nothing to do with the cut time-travel plan's past / present labels (`..\..\DS2 Vanilla\findings\e3-demo-leftovers.md`
  section 9): "phase" here is an install phase.

## 1. The rows

| Row (phase) | Unk00 | Unk01 | Unk02 | Unk03 |
|---|---|---|---|---|
| 1 | 200060100 "Cannot pass" | 200060200 "Path opened" | 107200 | 0.0 |
| 2 | 200060100 "Cannot pass" | 200060200 "Path opened" | 107210 | 0.0 |

- **Unk00 / Unk01** are message ids in `menu\text\<language>\dconlymessage.fmg`, a PC Scholar file (38 English entries)
  whose other messages are current-gen platform text: voice chat, the DualShock 4 / Xbox One / Xbox 360 controller
  names, and "Unable to start multiplayer session ... due to reputation issues".
- **Unk02** (107200, 107210) is no row id in any PC Scholar param, no field value in any other param, and no literal in
  any event or AI script. The numbers sit in the range of the game's global event flags (the bosses' map-clear flags
  are 100958-101001), so they are probably "phase reached" flags the program sets itself. Unconfirmed.
- **Unk03** is 0.0 in both rows; unknown.

## 2. The doors: MapObjectPlayGoParam

11 rows. Unk00 is the phase the door waits for (1 or 2, ChunkPhaseParam's row ids). Unk01 (285-287) is in the same
range as the fog doors' type field (`MapObjectWhiteDoorParam`, 280-286), so probably the door's look. Unk02-Unk06 are
small floats (3.0, 3.0, 3.0, 1.5 and 5.0 or 10.0; 5.0 / 5.0 / 5.0 in the default row 999999999), meaning unknown.

| Row | Phase | Unk01 | Also a fog-door row (type) | Map |
|---|---|---|---|---|
| 100400, 100410, 100420 | 1 | 286 | | m10_04 Majula |
| 100430 | 1 | 286 | | Majula |
| 100440 | 1 | 287 | | Majula |
| 100450 | 2 | 286 | 286 | Majula |
| 101000 | 2 | 285 | 281 | m10_10 Forest of Fallen Giants |
| 102900 | 2 | 285 | 285 | m10_29 Majula to Shaded Woods passage |
| 103100 | 2 | 285 | 280 | m10_31 Heide's Tower of Flame |
| 103400 | 2 | 286 | 281 | m10_34 Grave of Saints |
| 999999999 | 2 | 286 | | default |

**Where they are placed** (per-map `MapObjectInstanceParam`, param_id = a PlayGo row): 14 door instances.

| Map | Instance | PlayGo row (phase) | PS3 Scholar / v1.00 |
|---|---|---|---|
| Majula | 10040600 | 100450 (2) | the same instance, a fog door |
| Majula | 10040615-10040635 (5) | 100400-100440 (1) | new on PC |
| Majula | 10040640 | 100450 (2) | new on PC |
| Forest | 10100600 | 101000 (2) | the same instance, a fog door |
| Forest | 10100640 | 101000 (2) | new on PC |
| Majula-Shaded Woods passage | 10290600 | 102900 (2) | new on PC |
| Heide's Tower | 10310600 | 103100 (2) | the same instance, a fog door |
| Heide's Tower | 10310605 | 103100 (2) | new on PC |
| Grave of Saints | 10340605 | 103400 (2) | the same instance, a fog door |
| Grave of Saints | 10340620 | 103400 (2) | new on PC |

So 4 are fog doors the PS3 builds already had, now also PlayGo doors, and 10 are new. Phase 1 opens five new doors
in Majula; phase 2 opens the rest: two more in Majula, and the doors in the Forest, the Shaded Woods passage, Heide's
Tower and the Grave of Saints. Which exits the Majula doors block was not checked (needs the MSB positions).

## 3. The program

The PC Scholar exe (`DarkSoulsII.exe`, 1.0.3.0) has:
- RTTI classes `AppFileChunkManager`, `FileChunkSystem`, `EventConditionOther_CompareChunkPhase` (an event condition
  that compares the current phase), `MapAreaPlayGoDoorCtrl` and `MapObjPlayGoDoorComponent`;
- the param paths `param:/ChunkPhaseParam.param` and `param:/MapObjectPlayGoParam.param` in its param load list;
- an `App.UseChunkSys` option among the `App.*` developer options (`App.ShowFps`, `App.DebugModeEx` ...), next to the
  string `logger_config_release.properties`. Where the option is set, and its default, are not established; the Steam
  install ships no file that mentions it.

None of these strings (ASCII or UTF-16) is in the PS3 Scholar EBOOT, the v1.00 EBOOT or the PC vanilla exes 1.02 and
1.12, where control strings such as `App.ShowFps` and `param:/` paths do appear.

## Open points

- Whether the doors ever shut on PC (the chunk system's default, and `App.UseChunkSys`), from the code or a live check.
- What 107200 / 107210 and the door floats are.
- Which Majula exits the five phase-1 doors and the second phase-2 door block (MSB positions of the instances).
- Whether the PS4 and Xbox One releases use the same rows (only the PC files are here).

## Reproduce

`uv run --no-project --python 3.12 python -I -X utf8 tools\playgo_pc.py` from this folder's parent (a few minutes).
