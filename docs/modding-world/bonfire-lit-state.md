# Bonfire lit state, the travel menu and the save's bonfire list (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- **The travel menu lists a bonfire when bit 0 of its level byte is set.** The byte is one byte per bonfire in the save's
  bonfire list, at entry +2. It is `(Ascetic cycles << 1) | lit`. A data write of 0 made three bonfires vanish from the
  menu and a write of 1 brought them back; no world flag was involved [live B22, code].
- **`bonfire_lit_flag` is not the lit state.** It is the area's "boss beaten" world flag, shared by every bonfire of the
  area, and several bonfires carry 0 and still work. Do not set it to light a bonfire [data, live A5, B2].
- **The list is rebuilt from `MapObjectBonfireParam` at every load, every entry unlit; the save then restores levels by
  id.** So a new param row appears in an existing save at level 0 (77 -> 78 -> 79 entries), and a new character starts
  with all 77 rows at level 0 [live B2, B23, B55; code].
- **No script command lights a bonfire.** The game's own lighting function has one direct caller, the bonfire object's
  handler. The only bonfire entries in the script command and function names are two functions that read
  [data, code]. A bonfire is lit by the player, by a save, or by a memory write: the project's Cheat Engine table writes
  0 -> 1 for 38 bonfires [live B22, B55].
- **The upper bits are the Ascetic cycle, and they set the boss's intensity.** Writing 2 to the Last Giant's bonfire took
  his HP from 2,530 to 5,313 on the next load, the same as burning a Bonfire Ascetic [live A10, B56]. Level `n` is not
  tier `n`: the tier is `level >> 1`, so write `(tier << 1) | 1`.
- **Main traps:** the list count at manager +0x28 is a **u32**, not a u64; the list is searched with a binary search on a
  16-bit id, so it must stay sorted and ids must be 1-65,534; the writer has no bounds check against its 256 slots.

### Where the list lives

[code] Addresses are Ghidra `FUN_14xxxxxxx` names with image base 0x140000000; RVAs are given as `+0x...`. All of this
was read by disassembly on 17 November in the stock 1.0.3.0 exe (28,200,992 bytes), not in a patched copy.

`GameManagerImp` is the object whose pointer sits in a global at RVA `+0x16148f0`. From it:

```
EventManager        = [GameManagerImp + 0x70]
EventBonfireManager = [EventManager   + 0x58]
```

| Manager field | Meaning |
|---|---|
| +0x18 | the allocator the builder asks for the entry array |
| +0x20 | pointer to the entry array |
| +0x28 | entry count, **u32**. The constructor (`FUN_14017daa0`) stores four bytes here. |
| +0x2C | a different field (it read 1.8 as a float live [live B2 side find]) |
| +0x44 | s32, 0 or 1: added to the level-byte offset by every reader (see "The second level byte") |

Each entry is 0x18 bytes:

| Entry offset | Size | Meaning |
|---|---|---|
| +0x00 | u16 | bonfire id: the `MapObjectBonfireParam` row id |
| +0x02 | u8 | **level byte**: bit 0 = lit, bits 1-7 = Ascetic cycles |
| +0x03 | u8 | second level byte, cleared at build, read instead of +2 only when manager +0x44 is 1 |
| +0x08 | pointer | the entry's `MapObjectBonfireParam` row in memory |
| +0x10 | u64 | a per-bonfire mask. Helpers test and clear bits 0 to 0x29. We never found what the bits mean. |

### The functions

| Function | RVA | What it does |
|---|---|---|
| `FUN_14017daa0` | `+0x17daa0` | Constructor. List pointer 0, count 0 (four-byte store). |
| `FUN_14017ec50` | `+0x17ec50` | **Builder**, called from `FUN_140450010`. One entry per param row. |
| `FUN_1403bf9b0` | `+0x3bf9b0` | Turns a row's `area_id` into a map id and looks it up. A negative result drops the row. |
| `FUN_14017e540` | `+0x17e540` | **Save writer**, called from `FUN_1402e3a70`. |
| `FUN_14017eb10` | `+0x17eb10` | **Save loader** (restore), called from a neighbour of the writer's caller. |
| `FUN_14017f1c0` | `+0x17f1c0` | Finds an entry by id: **binary search** on the u16 id. Ids 0 and 0xFFFF are refused. |
| `FUN_14017c110` | `+0x17c110` | Finds an entry's **index** by id, also a binary search. |
| `FUN_14017e6f0` | `+0x17e6f0` | Takes an id, returns the lit bit: `byte & 1`. The bonfire object's handler asks this first. |
| `FUN_14017c2a0` | `+0x17c2a0` | Takes an index, returns the **lit bit**: `byte & 1`. |
| `FUN_14017c1f0` | `+0x17c1f0` | Takes an index, returns the **cycle**: `byte >> 1`. |
| `FUN_14017e320` | `+0x17e320` | **Lights a bonfire**: `(manager, id, notify)`. Sets bit 0 with an OR; if `notify` is set it also posts a notice (code 0xD). Its only direct caller is at `+0x1cb00b`, inside the bonfire object's handler `FUN_1401caf50`. That handler lights the bonfire only if `FUN_14017e6f0` says it is not lit yet, and sets `notify` when the acting character is the local player. |
| `FUN_14017e390` | `+0x17e390` | Counts entries with bit 0 set. This is the shape of the "number of lit bonfires" function (`f132242`) [inference]. |
| `FUN_14005ba50` | `+0x5ba50` | The **intensity total** for a bonfire id: its cycle (`byte >> 1`) plus a number from `FUN_14019ee30` (the character's own cycle [inference]). |

### Building the list at load

[code] `FUN_14017ec50` takes the row count from the param table's header (a u16), allocates `count * 0x18` bytes and
walks every row:

1. It reads the row id (kept as a **u16**) and the row's first dword, which is `area_id` [data: field offset 0].
2. It calls `FUN_1403bf9b0(area_id)`. That splits the number into the four bytes of a map id (`10100000` is
   `m10_10_00_00`) and looks the map up in a table under `[GameManagerImp + 0x38]`. A negative result drops the row and
   the count goes down by one.
3. Otherwise it writes the id, **clears the level byte and the byte after it to 0**, stores the row pointer and zeroes
   the mask.

So the list starts unlit on every load. Nothing in the param can give a bonfire a starting level: the builder discards
all of it, and none of the row's 15 fields is read for that.

**The six dropped rows.** A save lists 77 of the game's 83 rows (79 of our 85). [data, checked 17 Nov] The six missing
rows, 650 and 655 (`area_id` 90000000) and 62650 to 62653 (`area_id` 20620000), are exactly the rows whose
`area_id` names a map with no folder in the game's `map` directory (`m90_00_00_00`, `m20_62_00_00`). They also have
`icon_id` 0 and no instance row, so the data alone cannot say which property matters. The code says the check is a map
lookup, which fits the map folders [code, inference]. A new row must name a real map. See
[Adding a bonfire](adding-a-bonfire.md).

### Saving and loading

[code] The writer (`FUN_14017e540`) copies the list into the save record, one slot per entry, **256 slots**:

| Save record offset | Size per slot | Content |
|---|---|---|
| +0x1598 | u16 | bonfire id |
| +0x1798 | u8 | lit bit: `byte & 1` |
| +0x1898 | u8 | cycle: `byte >> 1` |
| +0x1998 | u64 | the entry's mask |

The loader (`FUN_14017eb10`) does a **merge** over two id-sorted lists. For each list entry it scans the saved ids
forward. If a saved id equals the entry's id it restores the entry; if the saved id is larger, the entry is new and stays
at level 0; if smaller, that saved slot is skipped (the bonfire is no longer in the param). It restores
`byte = (cycle << 1) | lit` and the mask. Three consequences:

- **Restoring is by id, not by position.** That is why a save written with 77 entries loads into a list of 79 and the two
  new ones stay unlit [live B2 step 1, B55 step 5].
- **The list must be sorted by id.** `FUN_14017f1c0` and `FUN_14017c110` binary-search it, and the loader's merge assumes
  it. Param rows are stored sorted by id, and our writer keeps them sorted (the installed param's 85 row ids read in
  ascending order [data, 17 Nov]). A param written out of order is untested and would break lookups
  [inference].
- **The writer has no bounds check.** It loops over the count and the slot arrays hold 256. The game has 83 rows, so
  nothing reaches the limit; a mod with more than 256 listed bonfires would overrun the save record [code,
  inference]. Ids are u16, so ids of 65,535 and above are impossible.

### What lights a bonfire

[code] The lit bit is set in one place, `FUN_14017e320`. Its only direct caller is the bonfire object's handler
`FUN_1401caf50`: the player pressing the prompt. The other things that change the byte are the loader (restoring a save)
and a memory write.

[data] No event or talk script command lights a bonfire. The 53 `.edd` developer-name files of the debug copy name **325
commands and 131 functions**. Exactly two names mention a bonfire, and both are functions that read:

| Function | Developer name | Gloss |
|---|---|---|
| `f132242` | 【その他】点火済み篝火数取得 | number of lit bonfires |
| `f132243` | 【その他】篝火周回数取得 | bonfire cycle count |

The game's own scripted lighting (`event_m10_16`, group 2147483547) lights the four **Primal Bonfires**. They are map
objects that are not in the travel menu, so that recipe does not show that a script can light a travel bonfire [data].
See [Script command reference](../modding-scripts/script-command-reference.md).

Warping does not light a bonfire either. A bonfire warp to an unlit bonfire arrives at an **unlit** bonfire and the
player lights it by hand [live A5 part 2, B2 step 2].

### The travel menu

[live B22] The menu follows the level byte alone. With three arena bonfires written to 0 the menu no longer listed them;
after the Cheat Engine table wrote 1 they were listed and travel worked. A plain data write was enough: no game
function was called. B22 only tried 0 and 1, so that the test is bit 0 rather than "non-zero" rests on the code: the
writer, `FUN_14017c2a0` and the lit counter all use `byte & 1` [code]. We did not identify the menu's own reader.

The menu entry for a bonfire comes from its param row (`icon_id`, `menu_sort_id`) and its name; see
[Adding a bonfire](adding-a-bonfire.md).

The lit bit has a second consumer. The respawn builder (`FUN_140184660`) looks up bonfires 4650 (Majula) and 2650 (Things
Betwixt) with `FUN_14017c110` and `FUN_14017c2a0`. If neither is lit, it ignores the stored respawn record and sends the
player to the start of Things Betwixt [code] (see [The respawn point](respawn-point.md)). So a data write that unlights both
would change where deaths go.

### The level byte's upper bits: the Ascetic cycle and intensity

[code] The lit bit is bit 0. Bits 1-7 count Ascetic cycles for that bonfire:

| Byte | Lit | Cycle |
|---|---|---|
| 0 | no | 0 |
| 1 | yes | 0 (the normal state of a lit bonfire) |
| 2 | no | 1 |
| 3 | yes | 1 |
| 4 | no | 2 |
| 5 | yes | 2 |

`FUN_14005ba50(id)` returns the cycle plus a second number. Three small comparison functions at about `+0x4112f0`
(`>= N`), `+0x411330` (`<= N`) and `+0x411370` do the same sum from a bonfire index kept in the generator object (+0x80)
and compare it with a number they are given. Generators name a bonfire, so the bonfire's cycle decides which enemies
appear, and it raises boss stats [code, data: `GeneratorParam`]. That the number they compare with is the generator's
`AppearanceEventID` is our reading [inference]; see [NG+ near boss arenas](ng-plus-differences.md).

[live A10] The Last Giant's HP was **2,530** at level 1. After `bonfire 10655 2` (Cardinal Tower only), a reload and a
walk to the arena, a new character object read **5,313** (x2.1 for him: the first step differs by enemy, x1.0 to x2.1
over all enemy rows, because the byte picks an `EnemyParam` HP column [data, 18 Nov]; see
[Difficulty levers](difficulty-levers.md)). One bonfire's byte sets the intensity of
the bosses it governs on the next load, without any Ascetic item or boss reset. [live B56] Burning a real Bonfire Ascetic
at that bonfire gave the same 2,530 -> 5,313, and the effect is permanent for that character.

Reading these together: one cycle = +2 in the byte. A10 wrote **2**, which by the code is cycle 1 **and unlit**. The
travel menu was not looked at during A10, and the bonfire went back to 1 afterwards. A real burn should leave the bonfire
lit, so it should read 3 [inference; nobody read the byte after B56's burn].

### What `bonfire_lit_flag` is

[data] The field is 4 bytes at row +0x24. Its values are the per-area "boss beaten" world flags, the same numbers as
`BossBattleParam`'s `map_clear_global_event_flag`. Every bonfire of an area shares one:

| Value | Bonfires |
|---|---|
| 100971 | Forest: 10655, 10660, 10670 (the Last Giant's flag) |
| 100972 | The Place Unbeknownst, 10675 (the Giant Lord's) |
| 100963 | all seven Lost Bastille bonfires |
| 0 | Things Betwixt 2650, Majula 4650; Dragon Aerie 27650 and Shrine Entrance 27655; **our 3650 and 10685** |

Twenty-seven different non-zero values are used across the 85 rows; the full set is in the params.

What reads those flags: [data] Majula's event script (the original `event_m10_04_00_00` has 21 references to five of
them), including the group that lights the world map's flames, which passes both the boss flag and the bonfire id. The
5 November scan also found the Herald's boss counter and the NPC-moves-to-Majula groups [data, findings]. So the flag
means "this area's boss is beaten".

[live A5] Dragon Aerie and Shrine Entrance carry flag 0, and a warp there worked with the array level as their only lit
state. [live B2] Our two new rows carry 0 and were lit by hand.

[code] The exe does read the field, in exactly one place we found: `FUN_14017dc90` (called from `+0x181eca`). It walks
every entry whose row has the same `fog_door_id` as the bonfire asked about, reads `can_burn_effigy`, `can_burn_ascetic`
and (when non-zero) `bonfire_lit_flag`, and calls `FUN_14017f090`. That compares each bonfire's cycle with numbers
looked up through `Unk10` and `Unk11`. So it is part of a rule about burning items, not about the list or the menu
[inference: an Ascetic or Effigy gate; what it allows is unknown]. A bonfire row with flag 0 is skipped by that loop.

### The second level byte

[code] `FUN_14017e890` sets manager +0x44 from a virtual call on a global object (`FUN_1405135f0`, whose meaning we did
not identify) and clears every entry's +3 byte. Every level reader adds +0x44 to the offset: `FUN_14017c2a0`,
`FUN_14017c1f0`, `FUN_14017e320`, the lit counter and `FUN_14017f090`. With +0x44 = 0 they read the byte at +2; with 1,
the byte at +3. The save writer always reads +2. So there is a second, unsaved level byte that the game uses in some other
mode, perhaps an online session [inference]. All our tests were offline and read and wrote +2. If a write to +2 ever
seems to do nothing, check manager +0x44 first [untested].

### Read the list from outside the game

1. Find the `GameManagerImp` pointer at exe RVA `+0x16148f0`; check the object's RTTI name if you can.
2. `[GameManagerImp + 0x70]`, then `[. + 0x58]`: the manager. Read the **u32** count at +0x28 and the array at +0x20.
3. Entry `i` is at `array + 0x18 * i`: u16 id, level byte at +2. Accept a count of 1 to 200.
4. Refuse to work when `[GameManagerImp + 0xd0]` (the player) is null: the title screen and loading screens have no list
   yet or a stale one.

### Light bonfires with a memory write

1. Read the byte. Write `byte | 1`. The game's own function does an OR, so that keeps any cycle bits.
2. Open the travel menu again. It lists what is now lit. The next save writes the live list, so the change is kept
   [code]. On 16 November 36 of the table's 38 bonfires were already lit in the save we tested, ten days after the table
   first ran on a save we believe was the same character [live B55 step 5; the character is not confirmed].
3. To unlight a bonfire for a test, clear bit 0 (we wrote 0 over level 1 in B22).

[data, the community table read 17 Nov] The community Cheat Engine table's "Unlock All Bonfires" entry takes the other
route, and we are grateful for it: it walks the list and calls the game's own lighting function (exe +0x17E320,
`FUN_14017e320`) with notify set for every entry that is still unlit. That also posts the in-game notice, so it is the
more faithful route. Our table writes the byte instead because B22 showed a plain write is enough and it needs no
function call inside the game's process.

### Set a bonfire's intensity

1. Write `(tier << 1) | 1` to the bonfire's byte: 3 for one cycle, 5 for two. Plain `tier` writes the wrong thing
   (2 means cycle 1 but unlit).
2. Reload the area (A10 used a trip away and back) and let the boss spawn again. A10's boss is not in memory after the
   reload; he spawns when the player nears the arena.
3. Put the byte back afterwards if the setting is only for a test. It takes effect on the next load too.

[untested] We never tried 3, 4 or 5. The cycle half is read from the code; the live tests used 2 and a real Ascetic.

### The shipped table: "Light the boss bonfires"

`cheat-table/BossArena.CT` has one Auto Assembler entry built by `tools/ce_table.py`. It is optional: the Herald's fight
list reaches every fight with no tool (see [Adding dialogue options](../modding-menus/adding-dialogue-options.md)); the table keeps the
travel menu as the boss list.

1. Start the game, **offline**, and load the character.
2. Cheat Engine 7.4: File > Open Process > `DarkSoulsII.exe`; File > Load the table; tick the entry.
3. A brief hidden `netstat` runs. The message lists what it lit and what was already lit, for example "Lit now (3) ...
   Already lit: 34".
4. Rest at any bonfire: the travel menu lists the fights.

What the script does, and refuses to do:

- It checks the module size (0x1D76000 for PC 1.0.3.0), the `GameManagerImp` RTTI name at RVA `+0x16148f0`, a non-null
  player pointer, and a **u32** count of 1 to 200.
- It reads each of its 38 ids (36 arena bonfires plus the two we added) and writes **1 only where the byte is 0**.
  Bonfires at 1, or at 2 and above, are left alone. It reads each byte back and reports a failed write.
- It stops if the game process has a TCP connection outside this PC (a safety net, not a guarantee: it cannot see
  Steam's Offline Mode), and asks Yes / No if `netstat` gives no answer.
- Ids that are not in the save's list are named in the message ("not in this save's bonfire list").

Results: [live B22] six-step run, 6 Nov: three unlit, menu without them, table, menu with them. [live B23] a brand-new
character: the watcher logged **77 in the list, 0 lit** before the table, and after it 38 lit and all 37 (then) arena
bonfires lit. [live B55 step 5] the shipped v1.01 table on a save that already had 36 lit: "That lit the bonfires
correctly" for the two new ones; read back, all 38 had the lit bit in a list of 79. [offline, 10 Nov] the table's own Lua
test, run against fake memory, fails 10 of its tests with a u64 count read and passes all with a u32 (re-run not done on
17 Nov).

### What a new character gets

[live B23] The new character's list holds all 77 rows at level 0 from the start, and the player lights each bonfire on
the first visit. The mod therefore does not depend on the travel menu: the Herald's fight list warps straight to each
arena [live B36c] (see [Adding dialogue options](../modding-menus/adding-dialogue-options.md)). Ways to pre-light the list that we rejected:
a pre-lit save file (one file holds every character slot and may carry the account id of whoever made it), a save
patcher (the same trust cost as any tool of ours), and the `bonfire_lit_flag` variant (pitfall 2). See
[Saves and installs](../modding-scripts/saves-and-installs.md).

## Pitfalls and what failed

1. **Reading the count as 8 bytes.** [live B2 side find] The count at +0x28 is a u32 and +0x2C is another field. Once
   +0x2C is set, a u64 read is garbage and a tool says "bonfire list not found". Our first `tools/b2.py`, the Cheat
   Engine table's reader, `tools/arenalib.py`, `tools/arena.py`, `tools/testwatch.py` and the memory reader all read a
   u64; all now read a u32. Cheat Engine's `readQword`/`readInteger` choice matters here.
2. **Using `bonfire_lit_flag` as the lit state.** [data, live A5, B2] The 5 November setup plan said a new row needs "a new
   lit flag", and a variant that set all 26 arena flags was built offline. It would have made every area's boss count as
   beaten (the map flames, the Herald's counter, NPC moves, the King's Gate flag) without adding a single entry to the
   travel menu. It was never installed.
3. **Expecting a warp to light the bonfire.** [live A5 part 2, B2 step 2] A bonfire warp to an unlit bonfire arrives at an
   unlit bonfire. The player lights it, which puts the byte back to 1.
4. **Writing the cycle number as the whole byte.** [code, live A10] A10's 2 gave the HP of one cycle but left the bonfire
   unlit. The project's NG+ notes suggest writing 3 for a second cycle; by the code 3 is the same cycle as 2, lit. Write
   `(cycles << 1) | 1`.
5. **The table skips cycled but unlit bytes.** [code, `tools/ce_table.py`] It lights only bytes equal to 0. A bonfire left
   at 2 (cycle 1, unlit) is "already lit" to the table and stays out of the travel menu. Only direct writes should
   produce that state, because a real Ascetic is burnt at a lit bonfire [inference].
6. **Assuming the community "Unlock All Bonfires" hooks the byte read.** Our early notes said it hooks the read of the
   byte. The table's own script is a one-shot that calls the game's lighting function for each unlit entry (read 17 Nov).
   It is a good route and worth studying.
7. **Expecting "no command lights a bonfire" to rest on a full code count.** The 10 Nov findings say "649 commands"
   without saying what was counted. Our recount of the `.edd` files finds 325 commands and 131 functions (456 names). The
   claim still holds for those names, and the lighting function's single direct caller supports it from the exe side.
   Commands that exist in the exe but have no `.edd` name are not covered by the name check [inference].
8. **Fearing the list was built once per character.** [live B2 step 1] A real fear in the B2 plan; an existing character
   gained the new row at level 0, because the list is rebuilt from the param at each load.
9. **Level `n` is not tier `n`.** [code] Tools and notes that say "level 2 = NG+, level 3 = NG+2" ignore the lit bit.
   `tools/arena.py bonfire <id> <level>` writes the raw byte, so the tier is `level >> 1`.

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| The travel menu follows the level byte alone | Three bonfires written to 0 left the menu; the table wrote 1 and they returned; no flag or game call | live B22 |
| Bit 0 is the lit bit; bits 1-7 the cycle | Writer, `FUN_14017c2a0`, `FUN_14017c1f0` and the loader all split the byte this way | code `+0x17e540`, `+0x17c2a0`, `+0x17c1f0`, `+0x17eb10` |
| Count is a u32 | The constructor writes four bytes; +0x2C read 1.8 live | code `FUN_14017daa0`; live B2 side find |
| Entries are 0x18 bytes; id u16 at +0, level at +2, row pointer at +8, mask at +0x10 | Builder and writer | code `FUN_14017ec50`, `FUN_14017e540` |
| List rebuilt from the param at every load, all level 0 | Existing save gained 3650 at level 0 (77 -> 78); 10685 made 79; builder clears the byte | live B2 step 1, B55; code |
| A new character lists all 77 rows at level 0 | Watcher read the list before any lighting | live B23 |
| The save restores by id; 256 slots | Merge loop; array spacing 0x200 / 0x100 / 0x100 / 0x800 | code `FUN_14017eb10`, `FUN_14017e540` |
| Lookups binary-search a u16 id | Two search functions; ids 0 and 0xFFFF refused | code `FUN_14017f1c0`, `FUN_14017c110` |
| Six rows are dropped | They are the rows with `icon_id` 0, no instance row and a map with no folder; the check is a map lookup | data 17 Nov; code `FUN_1403bf9b0` |
| `bonfire_lit_flag` is an area boss-beaten flag | Values equal `map_clear_global_event_flag`; Majula's script reads them | data; findings 5 Nov |
| A bonfire with flag 0 works with the byte alone | Dragon Aerie and Shrine Entrance; our two new bonfires | live A5, B2 |
| No script command lights a bonfire | 325 command and 131 function names; only two bonfire names, both readers; one direct caller of the lighting function | data 17 Nov; code `+0x17e320` |
| Warping does not light | Unlit on arrival, lit by hand | live A5 part 2, B2 step 2 |
| Level 2 raises the Last Giant's HP 2,530 -> 5,313 | HP read after reload | live A10 |
| A real Ascetic does the same | HP read after burning at his arena bonfire | live B56 |
| The cycle is `byte >> 1` and feeds an intensity total | `FUN_14005ba50` and the comparison functions near `+0x4112f0` | code |
| Only the bonfire object's handler lights a bonfire in code | One direct caller of `FUN_14017e320`; it checks the lit bit first | code `+0x1cb00b`, `FUN_14017e6f0` |
| The table lit 38 on a real save | Watcher read-back of 79 entries | live B55 step 5 (16 Nov) |
| A second level byte exists (+3, chosen by manager +0x44) | Readers add +0x44; the writer ignores it | code; purpose inference |
| The respawn builder needs Majula or 2650 lit | Reads the lit bit of 4650 and 2650 | code `FUN_140184660`; see Respawn-Point |

## Tools

| Tool | What it does here |
|---|---|
| `tools/ce_table.py` | Builds `cheat-table/BossArena.CT`: `build` writes it, `check` compares its ids with `rollout.json`, `test` runs the script against fake memory under a Lua engine. |
| `cheat-table/BossArena.CT` | The shipped table: "Light the boss bonfires" and a separate "Can't die (practice)" entry. |
| `tools/arena.py` | `bonfire <id> [level]` reads or writes one entry's level byte (a write only when the level is given). |
| `tools/arenalib.py` | `set_bonfire_level` (a u32 count read) and the warp helper used by the tests. |
| `tools/b2.py` | `status [id]` reads the list size, the id's level, the respawn record and a door flag; `light [id]` writes 1; `warp [id]` makes the game's bonfire warp. |
| `tools/testwatch.py` | The watcher that logs "N in list, L lit; arena K lit, M not in list". |
| `pcmem.py` | The project's process-memory reader (from the sister Scholar PC project); its `bonfires()` reads the list. |

## Open questions

- **Which half does intensity read, and what does a real Ascetic write?** Write 2, 3, 4 and 5 to one lit bonfire, read
  the boss's HP and the menu each time, then burn a real Ascetic and read the byte. This would settle the "level 3 =
  NG+2" wording [untested].
- **What is manager +0x44, and when is it 1?** [untested] Is the +3 byte an online-session level?
- **Does a mod-added bonfire's lit state survive quit and reload?** The loader restores by id, so it should, but no test
  logged it. A new character's list with the mod, a menu trip to a mod bonfire and an Ascetic there are also unplayed
  (see [Adding a bonfire](adding-a-bonfire.md)) [untested].
- **Which of the three or four properties drops a row** (map unknown, `icon_id` 0, no instance row)? Add a row with a
  real map but `icon_id` 0 and read the list [untested].
- **Does touching a bonfire object light it?** Commands `1:131631` and `1:131632` (【OBJ】OBJをタッチする, "touch OBJ") on
  a bonfire at load might reach the object handler and light the bonfire of the map you are in. No FromSoftware script
  uses them. A brand-new save would still start with none lit. Left untested by choice [untested]. Object state requests
  failed for other objects (see [Map object states](../modding-scripts/map-object-states.md)).
- **What are the mask bits, `Unk10` and `Unk11`?** And does the `fog_door_id` loop gate Ascetics or Effigies? [untested]
- **Order and size.** What happens to a param with rows out of id order, or more than 256 rows? [untested]

## Related pages

- [Adding a bonfire](adding-a-bonfire.md): new rows, the list's growth from 77 to 79, the six dropped rows.
- [Moving a bonfire](moving-a-bonfire.md): a moved bonfire keeps its id, so its level byte stays.
- [Bringing a boss back](respawning-bosses.md): the map-clear flag, kill counts and the Ascetic.
- [The respawn point](respawn-point.md): the record, and the lit bit of bonfires 4650 and 2650.
- [Script command reference](../modding-scripts/script-command-reference.md): the command names that read bonfire state.
- [Adding dialogue options](../modding-menus/adding-dialogue-options.md): the Herald's fight list, which needs no lit bonfire.
- [NG+ near boss arenas](ng-plus-differences.md) and [Difficulty levers](difficulty-levers.md): intensity and the cycle.
- [Saves and installs](../modding-scripts/saves-and-installs.md): the shared save file, and why a pre-lit save was rejected.
- [Memory map for PC 1.0.3.0](../tools/memory-map-pc.md): the other trainer offsets.
