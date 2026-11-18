# Event flags and choosing safe ids (PC Scholar 1.0.3.0, 2026-11-17)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.
> Test ids such as A1 or B16 point to the project's test log (see [Home](../how-to-read.md#the-modding-pages)). Evidence tags: `[live B16]` seen in the
> game, `[code]` read in the exe or the project's tools, `[data]` read in the game files, `[inference]` reasoned, not
> proven, `[untested]` an idea only.

## Summary

- **There are two kinds of event flag.** *Global* flags have six-digit ids (100000-109999 and 200000-209999, "blocks" 10
  and 20). They are saved and read the same in every map. *Map-local* flags have nine-digit ids, three blocks per map, and
  can be read or written **only while that map is loaded**. Anything one map must tell another goes in a global flag.
  [live A1]
- **Free global ids were found by scanning for the number everywhere**: every integer in the 538 scripts, every param field
  and raw word, every MSB and the exe. The run **104500-105299 has no reference at all**, and 202017-202039 in block 20
  has none either. The mod took all its own global flags from those two runs. [data]
- **Ids that live in saves must never change.** A flag, an arrival row or a bonfire id that a save may hold keeps its
  number in every later version of a mod, even after the feature that used it is dropped. A small register of the ids you
  own, with a check that fails the build on a clash, is how the mod kept to this. [code]
- **The main traps:** a map-local flag cannot be set from outside its map [live A1]; the world-state "boss beaten" flags
  (41 values in 100950-101080) are read by Majula and the NPCs, so a mod that resets a boss should leave them on; and a
  flag whose meaning you guessed from its neighbours can turn every vendor hostile [live B42].
- **The 5400-series text "clash" was not one.** The game passes its 5400 / 5410 / 5420 / 5430 messages whole to one
  command; nothing reads 5401-5408, which the mod uses. [data, live B34]

### 1. A flag id is a block and a bit

`[live A1]` The setter at RVA +0x4750b0 (Ghidra `FUN_1404750b0`, image base 0x140000000) and the project's reader agree on
the layout. The `EventFlagManager` is `[EventManager+0x20]`; the `EventManager` is `[GameManagerImp+0x70]`.

| Part | Rule |
|---|---|
| Block | `id / 10000` (integer division) |
| Rest | `id % 10000` |
| Byte in the block | `rest >> 3` |
| Bit in that byte | `7 - (rest & 7)` (bit 7 is the lowest flag of the byte) |
| Lookup | 31 hash buckets at manager +0x20; bucket = `block * 137 mod 31`; each node is `{+0x0 data pointer, +0x8 size in bytes, +0xC block, +0x10 next}` |
| Out of range | if the block has no node, or `rest >> 3` is not below the node's size, the setter returns 0 and writes nothing [code]; the project's reader prints "(no block)" |

Worked examples:

| Flag | Block | Byte | Bit | What it is |
|---|---|---|---|---|
| 104821 | 10 | 602 | 2 | a free global flag, used by the mod for a menu choice |
| 100971 | 10 | 121 | 4 | the Last Giant's "area boss beaten" flag (global) |
| 202020 | 20 | 252 | 3 | a free global flag in block 20, used by the mod for a menu choice |
| 110000081 | 11000 | 10 | 6 | the Last Giant's kill flag (map-local, m10_10) |
| 110020080 | 11002 | 10 | 7 | his fight flag (map-local, m10_10) |
| 221000096 | 22100 | 12 | 7 | Nashandra's kill flag (map-local, m20_21) |

**A map owns three blocks.** For map `m<aa>_<bb>` the blocks are `(aa / 10) * 10000 + bb * 100` plus 0, 1 and 2. [code,
`tools/arenalib.py`; live A1, A8]

| Map | Blocks |
|---|---|
| m10_04 (Majula) | 10400-10402 [live A1] |
| m10_10 (Forest of Fallen Giants) | 11000-11002 [live A1] |
| m20_10 (the Giant Lord's memory) | 21000-21002 [live A8 (b)] |
| m20_21 (Drangleic Castle) | 22100-22102 [data] |
| m50_37 (the Burnt Ivory King's map) | 53700-53702 [live A8] |

In the Forest the first block (11000) holds the kill and intro flags and the third (11002) the fight-state flags; the Throne duo
is the same (kill 221000091 in 22100, fight flag 221020090 in 22102). `tools/bosstable.py` takes a boss's fight flags from
the kill flag's block plus 2. [code; live A2]

### 2. Which blocks are in memory

- **Blocks 10 and 20 are always loaded.** The current map's three blocks are loaded with it. Only those. [live A1]
- **A map streamed in next to you may have its blocks loaded too.** At 14:15 in B38 the Heide dragonrider's flag became
  readable while the player stood in Majula with no load; an hour earlier it had not been. [live B38 side note] Majula streams
  its neighbours in, which probably explains it [inference]. Do not rely on it.
- **A same-map reload drops and restores the blocks.** The blocks leave the list for about 0.3 s, then come back. [live
  A8 (c); `tools/arenalib.py` comment, from the B38 watcher]
- **Outside the loaded blocks you can neither read nor write.** The setter returns 0 and writes nothing, and the project's
  trainer refuses a write to a block that is not loaded ("block not loaded, NOT cleared"). [code, A1's decode of the setter]
  A write made while the map was loaded worked: the A2 re-arm cleared the Last Giant's kill flag 110000081 with the Forest
  loaded and his fog gate came back at once. [live A2]
- **Values survive.** A global flag set by the mod is on after a quit and reload [live B47, B58]: a setup flag and an
  edition flag were read on again in a new session, and on a character first set up under the previous version. A
  map-local kill flag left on by the 4 November trainer session was still on at the first Throne of Want trip on 5 November,
  and Nashandra stood down [live B16]. So the *values* are kept (in the save) and only the *addresses* come and go with the
  map. [inference for the save file; the game's own behaviour of dead bosses staying dead says the same]

**Consequences for scripts.** Set a map's own flags in that map's own load group, after the map is loaded. The mod does this in
its arena groups, or in a patch of the game's own load group (the Wharf's ship flag 118000010, Mytha's drain flags 117000055
and 117000056). A game group that reads the flag once at load can still look before your write: see
[Event script groups](event-script-groups.md) section C for the Wharf case. [live B13, B26, B39]

### 3. The flags a boss fight uses

The Last Giant, read from the data and seen live in A2 (a 50 ms watcher):

| Meaning | Flag | Where it comes from | When it changes |
|---|---|---|---|
| Kill flag | 110000081 | `BossBattleParam.boss_destruction_flag`, row 1010010 | on about 6.5 s after HP 0, with the souls, in one tick |
| Map-clear ("area boss beaten") | 100971 | `BossBattleParam.map_clear_global_event_flag`; also the `bonfire_lit_flag` of the area's bonfires | same tick as the kill flag |
| Intro seen / fight started | 110000082 | the boss generator's `PreviousHostileEventID1` | on when the fog is first crossed |
| Fight on | 110020080 | the generator's `ActivationEventID1` (block = kill block + 2) | on at the fog, off at HP 0 |
| HP reached 0 | 110020084 | the fight script | on at HP 0 |

All 44 boss rows with a kill flag (of 45 rows in `BossBattleParam`) have a nine-digit, map-local kill flag. They have 41
distinct map-clear values, all global, in these runs: 100950-100968, 100970-100975, 100977-100980, 101000-101001, 101050-101051,
101053, 101061-101063, 101070-101072 and 101080. Three pairs of rows share a value: the Throne duo's two rows (100974), Nashandra's
two (100973) and the Burnt Ivory King's two (101061). [data, regulation, 17 Nov]

A kill **count** is not a flag: it is one byte per generator in the spawner's own table, and it is what keeps a dead boss away.
That, the 6.5 s delay, and the stale-flag rule are on [Bringing a boss back](../modding-world/respawning-bosses.md). The lit state of a bonfire is
neither of these flags: [Bonfire lit state](../modding-world/bonfire-lit-state.md).

The Throne of Want shows how many flags one fight can have. Its stage flags (221000091 duo dead, 221000096 Nashandra's kill,
221000006 Aldia's kill), cutscene "seen" flags (221000097, 221000098), the once-only retry point (221000093: setting it at
load disables the retry for good) and the two flags that start Aldia's sequence and fight (221000001, 221000007) are all in
block 22100.

### 4. Flags never to clear, and game flags that gate other things

**Never cleared by the arena: the map-clear flags.** Our first notes called 100971 "Cardinal Tower's lit flag" and said never to
clear it, because the three bonfires of that area carry it as their `bonfire_lit_flag`. That reason did not survive: the
travel menu follows the bonfire list's level byte, not the flag [live A5, B22]. The rule stands for another reason. These flags
mean "this area's boss is beaten" to Majula's world-map flames, the Herald's boss counter and several NPCs' moves, so a win
that left them on is the normal game state. [data] The trainer route cleared 100971 and no harm was noted [live A2]; the data
route never does. See [Bringing a boss back](../modding-world/respawning-bosses.md) and [Adding a bonfire](../modding-world/adding-a-bonfire.md).

**A game flag you set is a change to the world.** Before setting one, find every script that sets it and every one that reads
it. The cases we met:

- **Vendors' "attacked me" flags.** 17 flags in 103550-103870 look like "has met" flags. Setting them turned every Majula
  vendor hostile; clearing them is safe. [live B42 run 1]
- **A world-state flag can gate an NPC.** Melentia's Majula generator names 100971, so she was missing on a fresh character
  until it was set. [live B41 run 3; the fix itself unconfirmed]
- **"Heard this line" flags.** 93 flags in 201850-205544 stop the first-meeting speeches; only talk scripts read them. [data]
- **Flags the Throne of Want's own groups read.** The castle's groups read 100972 and 100747 (with 100973, 100974 and 100978) at
  load to decide which stages follow. 208030 is Aldia's "dying talk done" flag; once it is on, the game makes the ending
  objects usable. [code, `tools/choices.py`]
- **"Moved" flags.** A vendor's generator names the flag that stands it in a map; setting it moves the vendor. See
  [NPC menus](../modding-menus/npc-menus.md). [data]

### 5. How the free global flags were found

Before the first menu flag was chosen (5 November) every id in 100000-109999 and 200000-209999 was looked for in four places:

| Source | What was searched | How |
|---|---|---|
| Scripts | 538 ESDs (28 event, 25 talk, 485 AI): the shipped originals and the installed copies | any integer in any command argument or evaluator; 537 ids in block 10 and 429 in block 20 |
| Params | 205 regulation and 371 loose params (and 32 originals) | decoded field by field with the community param definitions (shop `enable` / `disable` flags, `EventFlagListParam` ids, generator death / hostile / appearance / activation ids, bonfire lit flag, boss-battle map-clear flag, NPC death flag, loading-display flag, N-choice `global_event_flag`, chunk phases), **and** as a raw int32 at every byte offset |
| MSBs and the exe | 29 MSBs, `DarkSoulsII.exe` | raw int32 at every byte offset. Mostly noise (floats, code bytes). None of the known global flags 100971, 105528, 106300 or 109990 appears as an exe immediate |
| The project's own files | `.json`, `.py`, `.md` | six-digit numbers |

6,480 distinct ids were referenced by something. Bulk resets (`1:130131(first, last, 0)`, developer name 【フラグ】パラメータに
設定されている指定範囲のフラグをまとめて設定, "set the flags of a range given in a param together") work on `EventFlagListParam`
rows, so every flag those rows name is in the param pass. [data, majula-menu scan, 5 Nov]

**Result.** 104500-105299 holds no script or flag-field reference; the nearest real flags are 104480 and 105300. In block 20,
202017-202039 had no hit. [data] The edge at 105300 is the three giant memories' "visited" flags (105300-105302, named by the
Forest's corpse groups). [data, A8 (b)]

**Recount on 17 November** from the checker's cache of every number the original scripts name (6,772 numbers; it rebuilds
itself if deleted): [data]

| Range | Result |
|---|---|
| Block 10 | 539 numbers referenced. Nothing in 104500-105299; the neighbours are 104480 and 105300. Nothing from 106301 to 109989; the last is 109990 |
| Block 20 | 431 numbers referenced. 202017-202039 has none (neighbours 202001 and 202100); the highest is 208031 |
| The 5 Nov counts | 537 and 429. A difference of 2 in each; we did not trace it, and it touches no free range |

**Inside the block's allocation.** The setter refuses ids past the block's size, and we never recorded that size. Evidence
that ids in these runs fit: the game's own scripts name flags up to 106300 in block 10 (one pardon menu sets it and reads it
back, [data, 5 Nov]) and 208031 in block 20; the mod's flags up to 104968 and 202022 were set and read live [live B16, B24,
B47, B57, B58].

### 6. What the game leaves free, and how to choose

| Block | Run with no reference | Nearest numbers the scripts name | Checked in |
|---|---|---|---|
| 10 | 104500-105299 (800 ids) | 104480 below, 105300 above | scripts, params, MSBs and exe [data, 5 and 17 Nov] |
| 10 | 106301-109989 | 106300 below (the pardon menu's flag), 109990 above | scripts only [data, 17 Nov]: run the other three passes before taking an id here |
| 20 | 202017-202039 (23 ids) | 202001 below, 202100 above | scripts, params, MSBs and exe [data, 5 and 17 Nov] |

The highest id any script names is 109990 in block 10 and 208031 in block 20. Rules we follow when picking:

1. **Take a run with no hit in any of the four sources**, and leave a margin from the nearest real flag.
2. **Stay inside what is proven to exist.** The block's size is unknown (open question), so do not go past the game's own
   highest ids (106300 set and read back, 208031 named). [inference]
3. **Leave room.** A run filled exactly has no spare: the next feature, or an overflow, lands on the next owner's ids (council,
   7 Nov, in the pitfalls table).
4. **Never take back an id that one of your own retired builds set.** An old test save may hold it on (section 9).
5. **Scan before you claim.** An early proposal was checked against scripts and loose params only and never built; the full
   scan replaced it.

### 7. The other id spaces

| Space | Free in the game | What guards it |
|---|---|---|
| Script group ids | 990000-990999: no group of the 538 scripts is in the block; the highest top-level event group is 4031010, talk 4100000, AI 9101 [data, 17 Nov] | a build check can fail a new group outside the block. See [Event script groups](event-script-groups.md) |
| `EventParam_<map>` rows | row id = group id; no original row is in 990000-990999, the highest is 4100000 [data, 17 Nov] | `tools/paramrows.py` refuses an existing row. Check rows against groups yourself: they must match |
| `EventLocation_<map>` rows | no original row is in 990000-990999; the highest is 90000002 [data, 17 Nov]. The mod's arrival rows sit at 9907xx | a build check can fail a row that is not yours. See [Warping by script](../modding-world/warping-by-script.md) |
| `mapevent.fmg` text ids | empty slots inside the existing ranges, listed below | the FMG writers refuse an occupied slot; a build check can fail a game text overwritten |
| `PluralSelectMenuParam` rows and `pluralselect.fmg` labels | the game has 3 rows (ids 10-12), so 9000 and up is free; the label id is the row id | the writer refuses an existing row. See [Adding dialogue options](../modding-menus/adding-dialogue-options.md) |
| `ShopLineupParam` rows | each vendor owns a block of 10,000 ids, and no row of the block reaches `+5000` or higher [data] | `tools/shops.py` stops if any row exists at or above the start. See [Vendor stock](../modding-menus/vendor-shops.md) |
| Bonfire id, instance row, entity id | a bonfire id that is not a `MapObjectBonfireParam` row; an entity id unique in its map | a build check fails "new bonfire id used by the game". See [Adding a bonfire](../modding-world/adding-a-bonfire.md) |
| `ItemLotParam2_Other` | a set of 18 lots no script names; the mod re-items four of them (99995005-99995008) | see [Giving items](../modding-menus/giving-items.md) |

Text slots deserve one rule of their own: **an empty slot is not proof it is free.** A script can name the id as a plain number
(a generator id, a message id). The 5 November check looked for every candidate slot as a literal in all the scripts; 5801 and
5802 are empty but were skipped, because m50_35's scripts name them as generator ids. [data]

Slots empty in the original English `mapevent.fmg` (128; the mod later filled 52 of them, and the mod's file still has 76
empty): [data, 17 Nov]

- **Empty and checked against the scripts** (none names the number): 1214-1219, 5301-5309, 5401-5409, 5411-5419, 5421-5429,
  5803-5809, 5811-5819, 5821-5829 and 5831-5839.
- **Empty, not checked** (candidates only; check them the same way): 1317-1319, 1321-1329, 1339, 2001, 2003, 2014-2019, 2302,
  2500, 3008, 4032-4039, 5701-5709 and 9001-9009.
- **Empty but not free**: 5801-5802.

### 8. A register that checks for clashes

A register is one list of every id you own, each with the name of what claims it, and a check that runs on every build. The
mod's (`tools/ids.py`) stops the build when an id is claimed under two names, compares a built copy with the original scripts
(a flag nobody owns, an owned flag the game's scripts name, a group outside your block, a game text overwritten, a row or
bonfire id that is not yours), and keeps the ids that live in saves in a frozen file, so that a later version can add but
never move them. Flags rewritten each time a menu opens are marked *transient*, so a list can grow. [code] Such a register
has blind spots, and ours has three: it finds a flag only in the command forms it knows, so a flag passed to a library group
or named by a bulk reset is invisible; its check of the game's use reads scripts only, so params, MSBs and the exe need the
scan in section 5; and text ids and param rows have no register, because the writers refuse an occupied slot or row and a
clash there fails safe. [code]

### 9. Freezing ids that live in saves

What a save keeps, and so what a later version of a mod must not renumber:

| In the save | Names |
|---|---|
| Event flag values (blocks 10, 20 and each map's) | the flag id |
| The respawn record `{map, kind, id}` | kind 0 = a bonfire id; kind 1 = an `EventLocation` row. After a warp from the mod's menu the guard writes kind 1 with the arrival row [live B44] |
| The bonfire list | one entry per `MapObjectBonfireParam` row, by id, with a level byte (see [Adding a bonfire](../modding-world/adding-a-bonfire.md)) |

Group ids, text ids, param rows and entity ids are not stored in a save: a group restarts at `s0` at each load, and a text is
looked up at display time. Renumbering them is safe for the save, though the flags a group sets stay. [inference from [Event
script groups](event-script-groups.md) section 4]

The rules the project follows:

1. **Never move or reuse an id a save may hold.** A retired one stays reserved, because the feature may be gone and an old
   test save may still hold its flag on. Do not take an id back because the experiment was dropped.
2. **A flag that is rewritten at every load or every menu opening is transient**, so the list can grow.
3. **Record new save ids before you ship.** A check that compares ids against the record cannot see one that was never
   added (the council's point on 16 November), so make the release step fail on any save id not yet recorded.
4. **A new one-time gift gets a new edition flag.** A save set up earlier then gets the new items once. [live B47] See
   [Giving items](../modding-menus/giving-items.md).

### 10. The 5400-series text "clash" that was not one

A review of 7 November suggested that texts 5401-5404 would collide with the game's own 5400 / 5410 / 5420 "text bases", and
some of our own notes avoided the 54xx slots for that reason. We checked all 538 original scripts. [data]

- The only literal uses of 5400, 5410, 5420 and 5430 in the event and talk scripts are 11 calls in four DLC event scripts
  (m10_19, m10_25, m10_32, m50_35). They go to library groups with the developer name 【DLC】【プリセット】テキスト表示される石碑,
  "DLC preset: stone that shows text".
- Those groups pass the id **unchanged** to `1:130447(id, 0, 0, 0)` (【FE】イベントウィンドウの表示, "show an event window").
  There is no offset, so nothing adds 1 to 5400 and reaches 5401.
- 5401-5404 were empty in the original, and so are 5409, 5411-5419 and 5421-5429. The mod uses 5401-5408, and its texts
  displayed correctly in the first live run. [live B34]

So the slots are as free as any others, and keeping clear of them was over-cautious (the note on [Vendor stock](../modding-menus/vendor-shops.md)
says the same). Slots 5301-5309 and 1214-1219 work too. [live B32]

### A. A new global flag

1. **Decide that it must be global.** A flag that one map reads and writes can be map-local. Anything a menu, a warp or another
   map must see is global, and block 10 (or block 20) is the place. [live A1]
2. **Pick an id** from a free run (section 6), or a run of your own in block 10. Leave the neighbours of the game's own flags alone,
   and do not pick an id that one of your own retired builds set.
3. **Scan for the number.** Look for it as an integer in every script (event, talk and AI: any argument, any `f6(...)`), in
   every param (decoded by the field definitions, plus raw at every 4-byte offset), in the MSBs, in the exe, and in your own
   files. A hit among floats and code bytes can be dismissed with a reason; a hit in a flag field cannot. Only the script pass
   was kept as code (`tools/ids.py`, which over-includes on purpose); the param, MSB and exe passes of the 5 November scan were
   scratch scripts.
4. **Claim it by name** in one place (a constant in the module that uses it) and put that module in your register. A second
   claim of the same id should stop your build (section 8).
5. **Read it live before relying on it** (game running, a character loaded): `tools/arena.py flags <id>` in Majula and in the
   map where it is used. Expect `0`, **not "(no block)"**. [live A1]
6. **Write it with `1:130101(flag, value)` and read it with `f6(flag)`.** A bank-5 form takes a condition as its first argument.
   In a menu, make choice flags one-hot: the choice turns its own flag on and the others in its group off, and **all off is
   the map's default**, so a save that never used the menu still works. [live B16]
7. **Decide frozen or transient.** If a later version could renumber it (a list row, a menu position), mark it transient.
   Otherwise it is a save flag and goes on record before you ship (section 9).
8. **Run your register check** on the built copy, and again on the installed copy.

### B. A new map-local flag

1. Work out the map's blocks (section 1). A flag you choose must be in one of them, and not one the map's scripts name.
2. **Set it from that map's own load group** (after the player is in the map), never from another map. The setter refuses a flag
   whose block is not loaded. [live A1]
3. If a game group reads the flag once at load, patch that group's first move rather than racing it.
   [live B39; [Event script groups](event-script-groups.md)]
4. Expect to clear arena kill flags 1 s after they come on, and at load if one is already on. [live B9, B16, B38; see
   [Bringing a boss back](../modding-world/respawning-bosses.md)]

### C. Another group, row or text id

| For | Do |
|---|---|
| A script group | An id in 990000-990999 that the map's script and `EventParam` do not use; add its row. [Event script groups](event-script-groups.md) |
| An arrival point | An `EventLocation` row in the same free block, one per place; record it if a respawn can name it. [Warping by script](../modding-world/warping-by-script.md) |
| A question or message text | An empty slot inside an existing range of `mapevent.fmg`, named as a literal by no script. The writer fills only ids inside a range; a new id outside every range needs a new range. Write every language you ship, or other languages see an empty window. FMGs are read at game start. [Game text and languages](../modding-menus/game-text-and-languages.md), [Text length limits](../modding-menus/text-length-limits.md) |
| A list row | A run of consecutive `PluralSelectMenuParam` ids (the list opens `first..last`), one label per row in `pluralselect.fmg`, one group per row. [Adding dialogue options](../modding-menus/adding-dialogue-options.md) |
| A shop row | A range inside the vendor's own block that no row reaches. [Vendor stock](../modding-menus/vendor-shops.md) |
| A bonfire | A bonfire id that is not a row, plus an entity id unique in its map. [Adding a bonfire](../modding-world/adding-a-bonfire.md) |

## Pitfalls and what failed

| Test | What happened | Lesson |
|---|---|---|
| **A1** (28 Oct) | A map-local flag cannot be read or written from another map; "(no block)" | Build per-map logic in the map's load group |
| **B42, run 1** (8 Nov) | The setup set 17 flags it had taken for "has met" flags; every Majula vendor turned hostile. They were the vendors' "you attacked me" flags (a talk helper sets the pair, plays the hostile line and saves); the community generator field name `HostileEventID1` had been right | Before setting a game flag, find every script that sets it and every one that reads it. Fix: never set them, and clear them at every Majula load once the setup is done |
| **B41, run 3** (8 Nov) | On a fresh character Melentia was missing from Majula. Her generator there names 100971 (the Last Giant's beaten flag), on in the progressed save and off in the fresh one | A world-state flag can gate an NPC. The setup now sets it; not confirmed by a separate test |
| **B19 era** (5 Nov), **B24, B25** (6 Nov) | A chasm retry group set a flag at every load; B25's two choice flags were built, then the choices dropped. Both can be on in old test saves | Retired ids stay reserved; never take them back |
| **Council, 7 Nov, finding 5** | The shop page flags filled a run of 100 ids exactly, ten per vendor; an 11th vendor, or a page 11, would have shared a flag with the next vendor or the next feature, and no check caught it | The register's double-claim check; the shop writer also stops at more than 10 pages |
| **Council, 7 Nov** | A "5400 text clash" was reported | Not one (section 10) |
| **Council, 16 Nov** | The check of frozen ids only examines ids already on record, so a new save flag could ship unrecorded | An "unrecorded save id" check was added, and the release build refuses |
| **Early proposal, 5 Nov** | The first free-run proposal was checked against scripts and loose params only | Scan all four sources before claiming a run (section 5) |

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| Flag layout: block = id / 10000, 31 buckets, node fields, byte and bit | setter decoded from the exe; live reads | A1, `FUN_1404750b0` |
| Only blocks 10 and 20 and the current map's are loaded | live reads, a block list | A1 |
| A map's blocks are `(aa/10)*10000 + bb*100 + 0..2` | live (Majula, Forest, the memory, the Frigid Outskirts) | A1, A8; `tools/arenalib.py` |
| The map-local kill flag cleared while the map was loaded brought the fog back at once | live | A2, second attempt |
| A same-map reload drops and restores the blocks (~0.3 s) | watcher | A8 (c), B38 |
| Global flags set by the mod survive a quit and a version update | live | B47, B58 |
| A map-local kill flag left on by the trainer session was still on at the next visit | live | B16 |
| 104500-105299 has no script, param, MSB or exe reference | four-source scan; recount of scripts | data, 5 Nov and 17 Nov |
| 202017-202039 has none | same | data |
| The game's scripts name block-10 ids up to 109990 and block-20 ids up to 208031; 106300 is the highest the game sets and reads back | scan of 538 scripts; the 5 Nov reading of Cromwell's pardon | data, 17 Nov and 5 Nov |
| Choice and option flags in the free runs work, one-hot, and are set and read | live; watcher | B16, B24, B57, B58 |
| 41 distinct map-clear flags, 44 kill flags, all nine-digit | read in `BossBattleParam` | data, 17 Nov |
| 100971 is not the travel menu's lit state | live | A5, B22 |
| 990000-990999: no group, `EventParam` row or `EventLocation` row of the originals | scan | data, 17 Nov |
| 128 English `mapevent.fmg` slots empty in the original (52 since filled by the mod, 76 still empty) | read, compared with the original file | data, 17 Nov |
| 5400 / 5410 / 5420 / 5430 are passed whole to `1:130447`; 11 uses in four DLC scripts | scan; group dump | data, 17 Nov |
| Texts 5401-5404 display correctly | live | B34 |
| A script-only register is blind to library calls, bulk resets, param / MSB / exe uses and text or row ids | read in `tools/ids.py` | code |
| Setting game flags without reading who uses them turned vendors hostile | live | B42 run 1 |

## Tools

| Tool | What it does here |
|---|---|
| `tools/ids.py` (with its frozen-id file) | The mod's register and clash check (section 8) |
| `tools/arena.py` | `flags <ids>` reads flags the way the exe's setter finds them (prints "(no block)"); `blocks` lists every loaded block with its size and set bits |
| `tools/arenalib.py` | `flag_addr`, `set_flag` (raw bit write; False when the block is not loaded), `blocks()`, `map_block(<map>)` |
| `tools/flagwatch.py`, `tools/fltrace.py` | Read-only loggers: every ~50 ms (flags, loaded blocks, souls, respawn pair) and ~10 ms (the list window's state) |
| `tools/bosstable.py` | A boss's kill flag, map-clear flag, fight flags and generators, found from `BossBattleParam`, the map script and `GeneratorParam` |
| `tools/esd.py`, `tools/edd.py`, `tools/esdwrite.py` | Dump a script and its developer names; the writer behind the register's script checks |
| `tools/fmgwrite.py`, `tools/paramrows.py` | Add a text or a param row; both refuse an id that is taken |

## Open questions

- **How big are blocks 10 and 20?** The setter's size check was never read out in a log we kept. The highest ids set live are
  104968 and 202022. [untested]
- **Are map-local values saved with the character, or only returned with the map?** B16's stale kill flag and the game's own
  dead bosses say saved; no test isolated it. [untested]
- **Whether another mod's flags overlap ours.** The register knows the game and this mod, not other mods. [untested]
- **The scan as a tool.** The param, MSB and exe passes of the free-id scan were scratch work; only the script pass is kept
  as code. A repeatable `tools/` script for all four would let a register check more than scripts. [open]
- **Is setting 100971 what brought Melentia back on a fresh character?** Later clean installs showed every vendor present
  [live B49, from a memory note of 8 Nov], but no test took 100971 out of the setup. [untested]
- **The other platforms and versions.** The scans are of PC 1.0.3.0 only. [untested]

## Related pages

- [Event script groups](event-script-groups.md): the group id block, the first-state rule, flags across loads, the Wharf race
- [Warping by script](../modding-world/warping-by-script.md): `EventLocation` rows and why a respawn can name them
- [Adding a bonfire](../modding-world/adding-a-bonfire.md): the bonfire ids and entity ids that live in saves
- [Bringing a boss back](../modding-world/respawning-bosses.md): kill flags, map-clear flags, kill counts and stale flags
- [Bonfire lit state](../modding-world/bonfire-lit-state.md): why the lit flag is not what the travel menu reads
- [Adding dialogue options](../modding-menus/adding-dialogue-options.md): `PluralSelectMenuParam` rows and label ids
- [Text length limits](../modding-menus/text-length-limits.md) and [Game text and languages](../modding-menus/game-text-and-languages.md): the text slots
- [Vendor stock](../modding-menus/vendor-shops.md): the shop id blocks and the page flags
- [Giving items](../modding-menus/giving-items.md): setup editions and the lots
- [NPC menus](../modding-menus/npc-menus.md): vendor presence in a map, the hostile and "moved" flags
- [Detecting a player death in a script](detecting-death-in-scripts.md): saved flags for death detection
- [Script command reference](script-command-reference.md): `1:130101`, `f6`, `1:130131`
- [The respawn point](../modding-world/respawn-point.md): the record that names a bonfire or a row
- [Saves and installs](saves-and-installs.md): what a modded install leaves in a save
