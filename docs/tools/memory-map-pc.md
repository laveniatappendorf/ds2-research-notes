# Memory map for PC 1.0.3.0 (trainer offsets)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

Tags in square brackets say how we know a claim: `[live B57]` = seen in the game (the id points to the project's test
log, see [Home](../how-to-read.md#the-modding-pages)), `[code]` = read in the exe, `[data]` = read in the game files, `[inference]` = reasoned, not
proven, `[untested]` = an idea only. Every memory write described here was made with Steam in Offline Mode. Do the
same: do not write to the game's memory while it is online.

## Summary

- **One global, one tree.** The exe holds one pointer, at RVA `+0x16148F0`, to the `GameManagerImp` object. Everything the
  arena tools read or write hangs off it: the event manager (+0x70: flags, bonfire list, respawn record), the generator
  manager (+0x40: kill counts), the bag (+0xA8, then +0x10) and the player (+0xD0). A build is recognised by its image
  size, `0x1D76000` for 1.0.3.0. [code] [live A1-A10]
- **This page is the one map**: every offset in one place, one pointer notation, and what must be true before you read
  or write. The rules, failures and recipes for each structure are on the linked topic pages.
- **Position takes three writes and a kick**: the same x, y, z to `+0x1A0`, `+0x1B0` and `+0x1C0` of the Havok body, then a
  velocity of (0, -2, 0), or the map does not draw until the player moves. That part has its own page,
  [Teleport and facing](teleport-and-facing.md). [live A7]
- **Nothing you read stays valid.** `PlayerCtrl`, the physics body and every character are rebuilt at each load, and
  flag blocks, kill-count slots and the bonfire list come and go with the map. Walk the pointers again each time, and
  check the object's class before every write. [live A7, 30 Oct, B57]
- **Traps, each with its own test id:** map ids are stored `{00, 00, bb, aa}`, not `{aa, bb}` (30 Oct); a map's flags exist
  only while it is loaded (A1); a map's kill-count slot is absent until that map has loaded (4 Nov); a warp called while
  the player is dead hangs the respawn (28 Oct); the bonfire count is a u32, not a u64 (B2).

### Notation

| Term | Meaning |
|---|---|
| `exe` | the base address of `DarkSoulsII.exe` in the running process (it changes each launch) |
| RVA `+0x16148F0` | an offset from `exe`. The Ghidra address is `0x140000000` plus the RVA, so this one is `0x1416148F0` |
| `[x]` | read the 8-byte pointer stored at address `x` |
| `[x + 0x70]` | the pointer stored at `x + 0x70` |
| `GameManagerImp` | **the object**, that is `[exe + 0x16148F0]`. Always the object on this page, never the address of the global |
| s32, u16, f32 ... | little-endian numbers: signed 32-bit, unsigned 16-bit, 32-bit float |

Other pages and the tools' docstrings write the same chains in two other ways. [The respawn point](../modding-world/respawn-point.md) and
[Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md) write `[[GameManagerImp] + 0x70]`, with `GameManagerImp` standing for the
global and the first brackets reading it. The tools write `[[[GameManagerImp]+0x70]+0x58]+0x20`, and the community cheat
table writes hex without the `0x`. All of these are the same addresses: `[[GameManagerImp] + 0x70]` there is
`[GameManagerImp + 0x70]` here, which is the `EventManager`.

### The pointer tree

```
exe + 0x16148F0                 the global that holds GameManagerImp
GameManagerImp = [exe + 0x16148F0]
 +0x38    the game's table of maps (the bonfire list builder and the respawn code look maps up in it)
 +0x40    EnemyGeneratorManager
            +0x10   EnemyGeneratorDeadCounter     kill counts, one slot per map
            +0x305  byte; bit 0x10 = the spawn check's exemption switch
 +0x70    EventManager
            +0x20   EventFlagManager              31 hash buckets of flag blocks
            +0x58   EventBonfireManager           the save's bonfire list
            +0x70   the warp object               what the game's warp function works on
            +0x164  respawn map   +0x168 kind   +0x16C id
 +0xA8    [+0x10]   the bag                       lists at bag +0xF0 and +0xF0F4
 +0xD0    PlayerCtrl
            +0x60   world matrix (+0x60 .. +0x9F)
            +0xF8   ChrMotionCtrl     [+0xF0] MorphemeChrCtrl   position +0x20, facing +0x30
            +0x100  ChrPhysicsCtrl    [+0x320] hkpCharacterRigidBody   [+0x20] hkpRigidBody
            +0x168  HP   +0x16C minimum HP   +0x170 max HP   +0x174 cap
            +0x490  PlayerParam       stats, souls, covenant
 +0x22E0  the object the "item received" pop-up draws on
```

Two offsets in the tree come from code readings only (`+0x38` and `+0x305`); every other one was read or written live.
[live A1-A10, B2, B56, B57] [code] GameManagerImp's other fields are not mapped. One more was seen once: `+0x22F0` is null
when a regulation fails the game's tag check, and the boot crashes a second later [live B3]; see
[Editing params](../modding-scripts/editing-params.md).

### Finding the global and checking the build

The static pointer to `GameManagerImp` is found with a byte pattern in the exe's code. The code is not encrypted on disk,
so the scan runs on the file with no game running. The pattern is not ours: the community cheat tables and open-source
Dark Souls II trainers use it. We re-ran it on the stock 1.0.3.0 exe (28,200,992 bytes, image size `0x1D76000`; 17 Nov)
and it gives **one hit**. [code]

| Pointer | RVA | Pattern (`??` = any byte), `mov r64, [rip+disp32]` first |
|---|---|---|
| `GameManagerImp` | `+0x16148F0` | `48 8B 05 ?? ?? ?? ?? 48 8B 58 38 48 85 DB 74 ?? F6` (hit at `+0x3F5219`) |

Two more static pointers are found the same way: the network session manager (`+0x1616CF8`) and the main
application object, `KatanaMainApp` (`+0x16751F8`). Nothing on this page reads them, so their patterns are not given.
[code]

Our tools refuse to write unless the module size equals `0x1D76000`, and also unless Steam is in Offline Mode and the
game has no TCP connection outside the PC. They log every write with the old and new bytes. [code]

Addresses written as `exe+0x....` in community cheat tables are build-specific. The table's hook for the kill count sat at
RVA `+0x1F67C5` on this exe, and the comment in the table named other offsets from another build [live A2 follow-up].
`+0x4750B0` (set event flag) was right as written. Check each one on your build.

### Identifying an object: RTTI

Most objects have an MSVC virtual table, so the class name can be read. The tools do this before they write to the player, to a
character or to the kill counter.

1. `vtable = [obj]`.
2. `locator = [vtable - 8]`. A valid x64 locator starts with the u32 `1`.
3. The type descriptor is at `exe + u32(locator + 0xC)`; its name string starts at +0x10, like `.?AVPlayerCtrl@@`.

Names we met: `GameManagerImp`, `PlayerCtrl`, `CharacterCtrl`, `EnemyGeneratorDeadCounter`, `EnemyGeneratorCtrl`,
`ChrPhysicsCtrl`, `hkpCharacterRigidBody`, `hkpRigidBody`. [live A2, A7, B8, B17] The check is cheap protection: a
pointer that was valid before a load can point at freed memory after it, and a class check refuses a stale one.

### Event flags

`EventFlagManager = [EventManager + 0x20]`. It is a hash table of **blocks**, one block per 10,000 flag ids. [live A1] [code]

| Where | Content |
|---|---|
| `F + 0x20 + 8 * b`, b = 0 .. 30 | bucket head: a pointer to the first node, or 0 |
| node +0x00 | pointer to the block's bytes |
| node +0x08 | s32, size of the block in bytes |
| node +0x0C | s32, the block number |
| node +0x10 | pointer to the next node in this bucket |

To find flag `id`:

1. `block = id / 10000` (integer division), `rest = id % 10000`.
2. `bucket = (block * 137) mod 31`. Take `[F + 0x20 + 8 * bucket]` and follow `+0x10` until the node's `+0xC` equals `block`.
   No such node means the block is not loaded.
3. If `rest >> 3` is not below the node's size, the flag is not there.
4. The flag is bit `7 - (rest & 7)` of byte `rest >> 3` of the block: **most significant bit first**.

Worked examples, computed with this rule:

| Flag | block | rest | bucket | byte | bit |
|---|---|---|---|---|---|
| 110000081 (Last Giant's kill flag) | 11000 | 81 | 28 | 10 | 6 |
| 100971 (his global flag) | 10 | 971 | 6 | 121 | 4 |
| 537000012 (Aava visible: "Eye picked up") | 53700 | 12 | 11 | 1 | 3 |

**Which blocks exist.** Only the two global blocks, 10 and 20, and the **current map's** blocks are in memory. Map
`m<aa>_<bb>` owns blocks `(aa / 10) * 10000 + bb * 100 + 0 .. 2`. [live A1, A8, flag watcher logs]

| Map | Blocks | Seen |
|---|---|---|
| m10_04 Majula | 10400, 10401, 10402 | A1, watcher log |
| m10_10 Forest of Fallen Giants | 11000, 11001, 11002 | A1, A2 |
| m20_10 (the Giant Lord's memory) | 21000, 21001, 21002 | A8 |
| m50_37 Eleum Loyce | 53700, 53701, 53702 | A8 |
| m40_03 Dark Chasm | 40300, 40301, 40302 | flags 403000001 and 403020012 read in 40300 and 40302 (30 Oct, B2); 40301 by the formula |

A flag in a block that is not loaded cannot be read or written, so a boss's own flags exist only while you are in his map.
From Majula an arena's flags read "not loaded" (the stale-kill checker, 7 Nov [live B38]); once, a Heide flag stayed
readable in Majula because its block was still loaded [live B38, 14:15]. [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md)
covers choosing ids.

**A load rewrites the table.** The watcher's log of the Majula-to-Forest warp at 14:37:20 on 28 Oct, polled every 50 ms:

| Time (s) | Blocks present |
|---|---|
| 347.25 | 10, 20, 10400, 10401, 10402 -> **none** (every block gone, the globals too) |
| 347.30 | 10, 20 |
| 347.66 | 10, 20, 11000, 11001, 11002 |

So the new map's blocks arrive 0.35 to 0.41 s after the old ones vanish (0.41 s here, 0.35 s in the Forest-to-Majula warp
a minute earlier; 0.41 s again in the 16:47 warp), and a reload of the **same** map drops them for about 0.3 s [live 28 Oct
logs; `tools/arenalib.py`]. The trainer uses this as its load detector: wait for the map's blocks to leave, then to come
back with HP above 0, then wait 2 s (`wait_any`). [Map object states](../modding-scripts/map-object-states.md) uses the same window.

**Setting a flag.** The game's own setter is at `exe + 0x4750B0`: `rcx = EventFlagManager`, `edx = flag id`, `r8b = value`.
It splits the id by 10,000 with a multiply-by-reciprocal (`0xD1B71759` appears in its first 32 bytes) [code, prologue
re-read 17 Nov]. The community cheat table calls it from a thread and it worked [live A1, A2: it cleared 100971, then
110000081 inside the Forest, and the fog came back at once]. **Our tools do not call it.** They flip the bit with a memory
write (`arenalib.set_flag`), which returns "not loaded" for a missing block. That re-armed the Last Giant [live A2
follow-up], set Aldia's six flags [live 30 Oct] and has run at every arrival since. Whether the setter does anything a
bit write skips, such as telling a script that a flag changed, is not known [untested].

### The respawn record

Three fields in the event manager decide where a death, the Darksign, a Homeward Bone and an Aged Feather send the player.
The whole story is on [The respawn point](../modding-world/respawn-point.md).

| Offset | Type | Field | Notes |
|---|---|---|---|
| `EventManager + 0x164` | 4 bytes | map | the bytes are `{00, 00, bb, aa}` for map `m<aa>_<bb>`: m10_04 = `00 00 04 0a`, m10_14 = `00 00 0e 0a` |
| `+0x168` | s32 | kind | 0 bonfire, 1 event location, 2 map start. **Must be 0** for a bonfire |
| `+0x16C` | s32 | id | a bonfire id (kind 0) or an `EventLocation_<map>` row (kind 1) |

`+0x16C` is the community table's "Last Bonfire". [live A1, A4, 4 Nov] [code]

- **Write all three or none.** The bonfire alone gave an endless loading screen: the game asked the old map for a bonfire
  that is not in it [live A4]. Map and bonfire with a kind of 1 sent every respawn to the map's origin, (0, 0.1, 0), where
  the kill plane killed in 0.9 s [live 4 Nov].
- **The map bytes are in the same order in the kill counter's slot keys** (below). A tool that wrote `{aa, bb}` sent every
  death to a map that does not exist, and its kill-count lookup found nothing [live 30 Oct]. m10_10 reads the same either
  way, which hid it from the Forest tests.
- A remote bonfire warp does not touch the record; the travel menu and a rest do [live A3 correction, A8].

### The bonfire list

`EventBonfireManager = [EventManager + 0x58]`. [live A1, A5, A10, B2] [code] Details, functions and the lit rules are on
[Bonfire lit state](../modding-world/bonfire-lit-state.md).

| Manager offset | Type | Content |
|---|---|---|
| +0x20 | pointer | the entry array |
| +0x28 | **u32** | the entry count (77 in the stock game, 79 with the mod's rows; a read as u64 breaks once +0x2C is set) |
| +0x2C | f32 | another field; it read 1.8 live [live B2 side find] |
| +0x44 | s32 | 0 or 1; added to the level byte's offset by every reader |

| Entry offset (0x18 bytes each) | Type | Content |
|---|---|---|
| +0x00 | u16 | bonfire id (the `MapObjectBonfireParam` row id) |
| +0x02 | u8 | the level byte: bit 0 lit, bits 1-7 the Bonfire Ascetic cycles |
| +0x08 | pointer | the entry's param row |
| +0x10 | u64 | a mask nobody has decoded |

The list is sorted by id, so the game finds an entry with a binary search. The tools scan it linearly and accept a count
of 1 to 200. Writing 1 into a level byte lights that bonfire for the travel menu [live A5, B2]. A10 wrote 2 and the Last
Giant's HP went from 2,530 to 5,313 on the next load, an x2.1 step [live A10]; by the bit reading that byte is one cycle
with the lit bit off, and the HP followed the cycle. Our tools' docstrings call the byte "level n = tier", which ignores the
lit bit; the topic page has the detail.

### The kill counter and generator objects

`EnemyGeneratorDeadCounter = [[GameManagerImp + 0x40] + 0x10]` (class name checked first). [live A2] [code] The rule that
uses it, and the data route that makes it irrelevant, are on [Bringing a boss back](../modding-world/respawning-bosses.md).

| Address | Content |
|---|---|
| `counter + 8 + n * 0xB10`, n = 0 .. 41, and `counter + 0x1D0A8` | map slots: 42 in the run plus one more at the same stride (n = 42) |
| slot +0x00 | map key, 4 bytes `{00, 00, bb, aa}` |
| slot +0x04 | u16, number of generators in the slot |
| slot +0x10 | u16 generator ids, sorted, up to 0x200 bytes |
| slot +0x210 | u8 kill counts, one per id, in the ids' order |

To read or reset a boss's count: scan the 43 slots for the key of his map; find his generator id in the slot's list; the
count byte is at `slot + 0x210 + index`. Write 0 to re-arm. [live A2 follow-up] The Forest slot held 149 ids; the Last
Giant is generator 857 and his count went 0 to 1 at his first kill, the only non-zero count in that map.

**The slot only exists once the map has loaded in this game session.** After a game restart a boss's first lookup finds no
slot: the trainer's log shows `{825: None}` for the Looking Glass Knight at 10:20, `{800: None}` for Velstadt and `{801: None}`
for Mytha, and `{825: 0}` once his map had loaded. The trainer resets the count again after the arrival load. [live 4 Nov]

A live **generator object** (class `EnemyGeneratorCtrl`) holds its id as a u16 at `+0x88`, its spawn limit as a u8 at
`+0x8D` (0 = no limit), a pointer to its param row at `+0x58`, and its map through `[[gen + 0x50] + 0x2C]`. [live B8, A2
follow-up] The limit is not the param's byte: `SpawnLimit` 0 read as 1 on boss generators and 12 on ordinary ones, 255 as 0.
[live B8] To find a generator, scan the player's heap (below) for its id as a u16 at an address with `+0x88`, then check
the class name. The spawn check reads `EnemyGeneratorManager + 0x305` (bit 0x10, an exemption switch that stayed off, 0x26
in every probe) and `[[gen + 0x68] + 0x76]`; see the topic page.

### The player

`PlayerCtrl = [GameManagerImp + 0xD0]`. It is null at the title screen and during a load, and **a new object is built at each
load**. Check its class name. [live B57 step 9, A7] The offsets below were read live unless a row says otherwise.

| Offset | Type | Field | Notes |
|---|---|---|---|
| +0x168 | s32 | HP | goes **below 0** on a death by damage (the floor is -99999); a kill plane sets exactly 0 [live 28 Oct, 4 Nov] |
| +0x16C | s32 | minimum HP | normally -99999. 1 = unkillable. Each load resets it [live B57.9] |
| +0x170 | s32 | max HP | also the max HP of other characters |
| +0x174 | s32 | HP cap | the hollowing cap; HP is clamped to it [live B57] |
| +0x1AC | f32 | stamina | read by the PC project's `player` command |
| +0x1B4 | f32 | max stamina | |
| +0x490 | pointer | `PlayerParam` | below |

The same reader also prints poise at +0x218 and speed at +0x2A8, taken from the community table; the arena tests never used
them. HP, cap and the HP rules are on [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md) and
[Detecting a death in scripts](../modding-scripts/detecting-death-in-scripts.md).

**`PlayerParam` = `[PlayerCtrl + 0x490]`**:

| Offset | Type | Field | Notes |
|---|---|---|---|
| +0x08, +0x0A, +0x0C, +0x0E | u16 | VGR, END, VIT, ATN | community table order |
| +0x10, +0x12, +0x14, +0x16, +0x18 | u16 | STR, DEX, INT, FTH, ADP | |
| +0xD0 | s32 | soul level | **the sum of the nine stats minus 53** |
| +0xEC | s32 | souls held | Give Souls moved it 0 to 100,000 [live A1] |
| +0x1AD | u8 | current covenant | 0 none; 8 Company of Champions (`1:130381(8)` moved it 0 to 8 [live B56.6, B57.4]); 9 Pilgrims of Dark |
| +0x1AE + n | u8 | covenant n discovered | 1 = yes |

- **Soul level = sum - 53.** Raising VGR by 9, INT by 19 and FTH by 19 at the Emerald Herald moved the displayed level from
  180 to 227, which is +47 [live A9]. A character with all nine stats at 99 read SL 838, which is 891 - 53 [live 4 Nov,
  night]. The tools write the stats (clamped to 1-99), then the level; HP, stamina and attack ratings follow at the game's
  next recalculation. The level is a stored number: a writer that changes stats and not the level leaves it wrong.
- **VGR, INT and FTH are confirmed** by A9 reads; END, VIT, ATN, STR, DEX and ADP are from the community table, and the
  all-99 write set all nine. [live A9, 4 Nov]
- **Covenant 9.** The trainer writes 9 for the Darklurker runs because his chasm portal only works in Pilgrims of Dark, and the
  portal then worked [live 30 Oct]. It writes `+0x1AE + n = 1` first, to mark the covenant discovered. `1:130381(8)` set the
  byte to 8 and made enemy damage x1.5 [live B56.6].

### Unkillable: the minimum-HP floor

`PlayerCtrl + 0x16C` is the lowest value HP can reach. Set it to 1 and hits still land, stagger and knock down, but HP stops
at 1. [live B57 step 9, 4 Nov]

- A load builds a new `PlayerCtrl` with the floor back at -99999, so the writer re-applies it: the cheat table's timer runs
  every 0.25 s, the trainer's loop every 50 ms, and a watcher polling every 0.1 s saw it back at 1 at each load [live
  B57.9]. A 242 HP hit left HP at 1; unticked and reloaded, the next hit killed (HP 282 to -218).
- **A fall out of the map did not kill the unkillable player**: the game reloaded the character at the respawn point, with
  no death [live 4 Nov, 11:29 and 11:34; not repeated since]. The first reading that morning, that the floor does not stop a
  kill plane, was taken during the death loop, where each respawn had reset the floor to normal.
- The one write is `+0x16C`. It does not change HP, max HP or the cap.

### Position and facing (teleport)

The player's position lives in three objects, and a teleport writes the **Havok body**: the same x, y, z to `+0x1A0`, `+0x1B0`
and `+0x1C0` of `[[[PlayerCtrl + 0x100] + 0x320] + 0x20]`, then a velocity of (0, -2, 0) at `+0x230`, or the map does not draw until the
player moves. The facing is the quaternion at `[[PlayerCtrl + 0xF8] + 0xF0]` `+0x30` (copy at `+0x50`), and the other copies of the
yaw follow it. [live A7] The full recipe, the tries that failed, the other copies of the position and the yaw, and the enemy
position (`CharacterCtrl +0x90`) are on [Teleport and facing](teleport-and-facing.md).

### Warping with the game's own functions

The travel menu's bonfire warp can be called from a thread of your own: a destination function fills a record and the warp
function queues it. The trainer runs both as a few bytes of code on a remote thread, and the community cheat table's
`BonfireWarp` script does the same. [live A1, A3, A5, A8, B2] [code] How the game finds the bonfire's map is on
[The respawn point](../modding-world/respawn-point.md).

| Step | Detail |
|---|---|
| Destination | `exe + 0x1843B0`: `rcx` = a buffer (the stub uses `rsp + 0x50` in a 0x118 frame), `edx` = the bonfire id (only the low 16 bits are read), `r8d` = 5 as the community table passes it; the 5 is not decoded [code prologue, 17 Nov] |
| Warp | `exe + 0x184830`: `rcx` = `[[GameManagerImp + 0x70] + 0x70]` (the warp object), `rdx` = the same buffer. It first copies the buffer's dwords into the object [code prologue, 17 Nov]. The stub null-checks every pointer on the way and ends if one is null |
| The thread | `OpenProcess` with create-thread, VM operation, read, write and query rights; one 0x1000-byte read-write-execute page from `VirtualAllocEx`; `CreateRemoteThread` at it; wait up to 5 s; release the page afterwards |
| On a timeout | **abandon the page**: never rewrite or free it, because the game's thread may still be running it. The next warp takes a fresh page [30 Oct review] |

Rules we learnt:

- **Never call it while the player is dead or loading.** A warp called while dead left the game stuck in its respawn
  [live 28 Oct]. The trainer refuses, and its "go home" key waits until the player is standing [live 4 Nov, 10:35].
- **A warp asked for too soon after a respawn is dropped.** The trainer waits 5 s and tries up to three times [live 28 Oct].
- **The bonfire must be in the save's list**; the game looks it up there, and `b2.py warp` works only for a listed id [live B2].
  An unlit bonfire still works: the player arrives and lights it [live A5].
- **A warp does not set the respawn record**; write it yourself [live A3 correction].
- **The remote call to the game's *event* warp (warp type 4) is not used.** It worked twice on 28 Oct, then reloaded the
  Forest instead, then the game crashed with heap corruption at 20:58. Later crash dumps put that crash on the debug build's
  add-on, so the warp may not have caused it; it was never retried. The memory and chasm bosses use the game's own entries.
  [live 28 Oct; [Warping by script](../modding-world/warping-by-script.md)]

### The bag

`bag = [[GameManagerImp + 0xA8] + 0x10]`. The topic page is [Giving the player items](../modding-menus/giving-items.md). [live 4 Nov]

| Where | Content |
|---|---|
| bag `+0xF0` | list 1: **3,840 slots of 16 bytes**: weapons, armour, rings, ammo, goods, spells (ends at +0xF0F0) |
| bag `+0xF0F4` | list 2: **256 slots of 16 bytes**: key items and gestures. The 4 bytes before it are unknown |

| Slot offset | Type | Content |
|---|---|---|
| +0x00 | u32 | item id (0 = empty) |
| +0x04 | u32 | 0 |
| +0x08 | u16 **or** f32 | quantity (ammo, goods, spells) **or** durability (weapons, armour, rings) |
| +0x0C | | upgrade level (a Large Club +3 read as 3) |

The **give list** the game's pick-up routine takes is a different layout: `{u32 0, u32 id, f32 durability, u16 quantity, u8
upgrade, u8 infusion}`, with the id at +4. The functions, relative to `exe`: `+0x199CC0` is the pick-up routine, `+0x1ABA90`
`can_give(bag, list, n)`, `+0x1AC3D0` `give(bag, list, n)`, `+0x5D950` `popup_prep(stack, list, n, 1)` and `+0x501080`
`popup([GameManagerImp + 0x22E0], stack)`. [code, 4 Nov; first bytes re-read 17 Nov] The first two start with
`mov rcx, [rcx + 0x10]` and jump on, so the object that does the work is one `+0x10` hop beyond the `bag` the stub passes;
nobody traced it. The stub gives 1 to 8 items; the 8 is the size of the stub's own entry area.

The Estus Flask (item 60155000) sits in list 1. Its slot bytes: `+0x08` u16 (1 on a full flask, not the sips), **`+0x0A` sips
left (u8), `+0x0B` sips at most (u8)**. [live B56.2, B57] The Ashen Mist Heart was refused as "not owned" because the first
reader missed list 2: it sat at bag `+0xF1A4`, slot 11 of the second list [live 4 Nov].

### Other characters

Every `CharacterCtrl` holds the fields below; the human NPC-type bosses are `PlayerCtrl` objects with the same ones.

| Offset | Type | Content |
|---|---|---|
| +0x38 | pointer | its `EnemyParam` row, in the game's loaded copy |
| +0x90 | f32 x3 | world position [live B13] |
| +0x168 | s32 | HP |
| +0x170 | s32 | max HP |

[live A10, B13, B56, 4 Nov] **To find a boss:** get the in-memory address of his `EnemyParam` row (next section), search the
heap that holds the player for it as an 8-byte value, and take `hit - 0x38`. Keep the hit only if its class is `CharacterCtrl`
or `PlayerCtrl` and it is not the player's own. The player, the flags and the map's characters all sat in one **269 MB**
allocation (of 1.9 GB readable): searching only that took **0.8 s against 46 s** for all memory [live 30 Oct]. The Cave of the
Dead trio (EnemyParam 862000-864000) are `PlayerCtrl`s, and a search for `CharacterCtrl` alone missed them [live B17].

- **A boss is not in memory at map load.** The Last Giant did not exist after the reload at Cardinal Tower and appeared as the
  player neared the arena [live A10]; the Burnt Ivory King's default ally likewise [live 30 Oct]. Search after arrival; the
  trainer repeats the search every 1.5 s until the character appears.
- **Kill by writing HP.** HP 0 on every live character of the row makes the game run its own death. Write only if the HP is
  above the target, and re-check the class and row pointer right before each write, because a reload frees the object [live
  4 Nov; 30 Oct review]. The kill is credited about 6.5 s later: [Bringing a boss back](../modding-world/respawning-bosses.md).
- **Max HP** reads the same way: the Last Giant read 2,530 / 2,530 at level 1 and 5,313 / 5,313 after one byte changed [live A10].

### Params in memory

A param table is held in memory as a **flat copy of the file**: the type name is 0xC bytes into the table and each row sits at
its file offset. That is how the tools find a row's copies [live 25 Oct to 4 Nov, `pcmem.eparam_copies`]:

1. Take the row's first 40 bytes from the file (for `EnemyParam`, the bytes before HP at +40) and scan the readable memory for them.
2. Keep a hit only if every byte outside the fields you plan to write matches the file, and the type-name bytes sit at
   `hit - row_file_offset + 0xC`.
3. Cache the hits per process id, but never cache an empty result: a scan at the title screen finds none and would have stuck
   for the whole session [live 4 Nov].

A row can have several copies: write to all of them. The first scan per game session takes 5 to 11 s, so the trainer starts it
in the background when a boss is chosen. In `EnemyParam` we used HP at +40 and `stat_hp` for NG+ 1 to 7, seven s32 at +320 to
+347 [data, community definitions].

**17 of 976 `EnemyParam` rows have a byte-identical twin**, the Darklurker's 506100 and 506101 among them. A search by the row's
bytes cannot tell twins apart and stopped with "not unique". The fix: take the row's file offset from the param reader and keep
only hits that pass the type-name check. [live 4 Nov; old and new methods agreed on all 959 unique rows]

A table of `{u32 entity id, u32 state}` pairs, the saved state of every object in the current map (10 for an untouched one),
sits beside the flags; see [Map object states](../modding-scripts/map-object-states.md).

## How to do it

A minimal external tool that reads and writes safely, in the order our tools do it.

1. **Attach read-only first.** Read the module base and size. Allow writes only if the size is `0x1D76000` and you are offline.
2. **Resolve the global**: `GameManagerImp = u64(exe + 0x16148F0)`, class name checked. It is null while the game starts.
3. **Wait for the player**: `pc = u64(GameManagerImp + 0xD0)`; 0 means title or loading; the class must be `PlayerCtrl`.
4. **Never cache a pointer across a load.** Walk the chain again before each action.
5. **Flags**: use the lookup above and treat "block not loaded" as a normal answer. A write reads the byte, sets or clears the
   bit and writes the byte back.
6. **Respawn**: `em = u64(GameManagerImp + 0x70)`; write `{00, 00, bb, aa}` at `em + 0x164`, `0` at `em + 0x168` and the bonfire
   id at `em + 0x16C`: all three.
7. **Re-arm a boss**: the counter slot by key, the count byte to 0, his kill flags to 0 while his map is loaded, then reload the
   map. A missing slot means redo it after the map has loaded.
8. **Teleport**: [Teleport and facing](teleport-and-facing.md).
9. **Warp**: refuse unless HP is above 0; run the stub on a remote thread; abandon the page on a timeout. Then wait for the
   load (the blocks leave and return with HP above 0) and 2 s more.
10. **Log every write**: time, label, address, old and new bytes. It turned several of our questions into five-minute reads.

`arena.py flags`, `arena.py kills`, `arena.py bonfire` and `teleport.py here` print the flags, one counter slot, one bonfire
entry and the player's position without writing anything: a good first pass.

## Pitfalls and what failed

1. **Map bytes `{aa, bb}` instead of `{00, 00, bb, aa}` (30 Oct; A4).** The trainer wrote the respawn map and read the kill slot's
   key reversed. Every death respawned into a map that does not exist and hung the game on the loading screen, and the re-arm
   never found a boss's count. The Forest (m10_10) reads the same both ways, so it passed the first tests.
2. **Half a respawn record (A4, 4 Nov).** The bonfire alone hung the game on the loading screen (the bonfire animation, a flash
   about every 30 s); writing +0x164 released it. Map and bonfire with `+0x168` left at 1 put every respawn at the map's origin;
   writing 0 while dead cured it. What set the 1 was not proven; the one scripted setter of kind 1 is castle group 17000.
3. **Flags outside the loaded blocks (A1, A2).** The setter does nothing and a raw write has nowhere to go. The first A2 re-arm
   attempt was not valid for a related reason: the table's Set Event Flags ran before the id was typed, so 110000081 stayed 1.
4. **A count slot that is absent (4 Nov).** `{800: None}` before the warp and `{800: 0}` after it. Reset after the arrival load,
   and reload the map once if the boss was found dead.
5. **Warping while dead, too soon after a respawn, or by the event warp (28 Oct).** The game stuck in its respawn, the warp was
   dropped, or (event warp, 20:58) the heap was corrupted, perhaps by another add-on. See "Warping" above.
6. **The bonfire count as a u64 (B2 side find).** The u32 at +0x28 is followed by an unrelated field at +0x2C; once that is set, a
   u64 read is garbage and a tool says "bonfire list not found".
7. **Bag readers (4 Nov, B56).** For weapons, armour and rings the value at slot +0x08 is an f32 durability, and the first reader
   summed it into huge counts. Key items sit in a second 256-slot list, so a reader that stops after 3,840 slots "loses" the
   Ashen Mist Heart. The Estus count is not the u16 at +8 (it read 1 on a full flask): the sips are the bytes at +0xA and +0xB.
8. **A stale pointer after a load.** `PlayerCtrl` is rebuilt, a character is freed, and the physics body's address was reused by an
   enemy body after a warp [live A7]. The class and row-pointer check before each write came out of the 30 Oct review.
9. **A hang after a bonfire warp (25 Oct).** The first no-death loop topped HP up every 0.1 s and the game hung after a warp. The
    cause was not shown. The loop now checks every 0.5 s and checks the class before it writes [PC project note; untested].
10. **Searching all of memory for a character (30 Oct).** 46 s. Search the player's allocation only: 0.8 s.
11. **Twin rows, and human bosses (4 Nov, B17).** Twin `EnemyParam` rows stopped a scan with "not unique", and the error ended the
    fight watcher until it was fixed. The Cave of the Dead trio are `PlayerCtrl`s, not `CharacterCtrl`s.
12. **Trusting a community table's `exe+` offset (A2 follow-up).** The kill-count hook was at a different RVA on this build.
13. **Teleport traps** (no map draw, a 3.2 m slide, a wall) are on [Teleport and facing](teleport-and-facing.md).

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| `GameManagerImp` is at RVA `+0x16148F0`; one hit for the pattern | pattern scan of the stock exe, and the pointer read live | code (17 Nov); A1-A10 |
| Flag blocks: 31 buckets, `(id / 10000) * 137 mod 31`, MSB-first bits | live flags read with the setter's rule | live A1; code `+0x4750B0` |
| Only global blocks and the current map's blocks are loaded | the block lists in the watcher logs | live A1, A8; flag watcher logs |
| Map blocks leave and return 0.35 to 0.41 s apart per load | 50 ms watcher polls | flag watcher logs, 28 Oct |
| The game's setter takes `rcx` = `[EM + 0x20]`, `edx` = id, `r8b` = value | prologue read; the community table calls it | code (17 Nov); live A1, A2 |
| Respawn record at EM `+0x164` / `+0x168` / `+0x16C`, map bytes `{00, 00, bb, aa}` | memory reads, the fixes that followed, the decompile | live A4, 30 Oct, 4 Nov; code (topic page) |
| Bonfire list at EM `+0x58`, entries 0x18, u32 count at +0x28 | reads, the constructor, the B2 side find | live A5, A10, B2; code |
| Kill counter at `[[GM + 0x40] + 0x10]`, slots 0xB10, ids +0x10, counts +0x210 | a kill moved a count 0 to 1; the re-arm worked | live A2 follow-up |
| A slot is absent until its map loads | `{800: None}` then `{800: 0}` | live 4 Nov |
| Generator +0x88 id, +0x8D limit, +0x58 row | probe of the live generators | live B8 |
| `PlayerCtrl` = `[GM + 0xD0]`; HP +0x168, min +0x16C, max +0x170, cap +0x174 | reads and writes in many tests | live B56, B57, 28 Oct |
| Minimum HP 1 holds through a hit and resets at a load | a 242 HP hit left HP 1; the floor read -99999 after a load | live B57.9 |
| An unkillable player survives a fall out of the map | the game reloaded at the respawn point | live 4 Nov, 11:29 and 11:34 |
| `PlayerParam` at +0x490: nine u16 stats, SL +0xD0 = sum - 53, souls +0xEC | stats and level moved together | live A9, 4 Nov, A1 |
| Covenant byte +0x1AD | `130381(8)` and the Darklurker's portal | live B56.6, 30 Oct |
| Warp: `+0x1843B0` then `+0x184830` on `[EM + 0x70]` | used in every trainer F5 and in B2 | live A1, A3, A5, A8, B2; code prologues (17 Nov) |
| Bag lists at +0xF0 (3,840) and +0xF0F4 (256), 16-byte slots | 41 items read; the Heart found at +0xF1A4 | live 4 Nov |
| Estus sips at slot +0xA / +0xB | refill 2 to 4, 1 to 4 | live B56.2, B57 |
| Characters: +0x38 row, +0x90 position, +0x168 HP; a boss is absent until the player nears | spider spawns matched 0.0 m; the Last Giant after a reload | live B13, A10 |
| Param tables sit in memory as the file's bytes; 17 of 976 `EnemyParam` rows are twins | the type-name check found only the right copies | live 25 Oct to 4 Nov |
| Position, facing and the enemy position | see the Teleport page's table | live A7, B13 |

## Tools

| Tool | What it does for this topic |
|---|---|
| `pcmem.py` | The shared memory reader and writer the arena tools import: process, module, static pointers, RTTI, `player`, `bonfires`, `nodeath`, `setstats`, `eparam`. Writes need Steam Offline and the 1.0.3.0 image size, and are logged. It lives in the PC project's tools folder. |
| `tools/arenalib.py` | Every offset on this page in code: flags, kill counts, respawn, bonfire levels, teleport, the warp stub, items and the bag, stats, covenant, minimum HP, characters, object states. |
| `tools/arena.py` | Read and write helpers: `flags`, `blocks`, `kills`, `rearm`, `bonfire`, `lastbonfire`, `setlast`, `hp`. |
| `tools/teleport.py` | The A7 position and facing tool: `here`, `save`, `face`, `go`, `vel`. |
| `tools/flagwatch.py`, `tools/respawnwatch.py` | Read-only change logs: flag bits and blocks, the respawn fields, HP, souls, map, warps. |
| `tools/b56.py`, `tools/b57watch.py`, `tools/b2.py` | Read the player, the Estus bytes, durability, the bonfire list and the respawn record while a test runs. |
| `tools/slice_lastgiant.py` (`probe`) | Finds live generator objects by id and prints +0x8D, +0x58 and `+0x305`. |
| `tools/arenactl.py` | The trainer: hotkeys over all of the above; its `stats` and `give` commands write the stats and the bag. |
| `tools/ce_table.py` | Builds the cheat table's two memory-writing entries (bonfire levels, the minimum-HP floor), with the build and online checks. |

## Open questions

- **Does the game's set-flag function do more than set the bit?** We never compared a raw write with a call [untested].
- **What are `GameManagerImp +0x38`, the warp object at `EventManager +0x70` and the object at `GameManagerImp +0xA8`?** We
  named only what we used. `GameManagerImp +0x38` is the start of the map chain traced on
  [Draw the whole map on PC Scholar](draw-whole-map-pc.md#2-the-chain).
- **What does the `5` passed to the warp's destination function mean?** We copied it from the community table.
- **Why do `can_give` and `give` start with `mov rcx, [rcx + 0x10]`?** The stub's `bag` is then one hop from the object that works.
- **What are the 4 bytes between the bag's two lists, the rest of a kill-counter slot (+0x410 to +0xB10) and the bonfire mask?**
- **Does the minimum-HP floor stop every fall death?** Two falls out of the map did not kill (4 Nov). The cheat table's own
  description still says a pit kills, from the earlier reading. A fall into a death pit with the floor on was never logged as
  such [untested].
- **Do these offsets hold on other builds?** The tests ran on 1.0.3.0 only. The PS3 and older PC builds are not mapped.
- **Is a script-side "set HP" (`1:130311` to `130314`) writing the player's `+0x168` and `+0x16C`?** The code says it writes those
  fields of some object; nobody has called it. See [Script commands](../modding-scripts/script-command-reference.md).
- **Online play.** Every test ran offline; whether the same writes are safe online is unknown, and our tools refuse to try.

## Related pages

- [Teleport and facing](teleport-and-facing.md): the position and yaw writes (this page's other half).
- [The respawn point](../modding-world/respawn-point.md): the record, its kinds and the death loop.
- [Bringing a boss back](../modding-world/respawning-bosses.md): the kill counter, the spawn check and the data route.
- [Giving the player items](../modding-menus/giving-items.md): the bag, the give list and the stub.
- [Bonfire lit state](../modding-world/bonfire-lit-state.md): the list, the level byte and the functions.
- [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md): HP, cap and the Estus bytes by script.
- [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md): which ids are safe to use.
- [Map object states](../modding-scripts/map-object-states.md): the `{entity id, state}` table.
- [Warping by script](../modding-world/warping-by-script.md): script warps, map offsets and the 28 Oct crash.
- [Quieting and removing enemies](../modding-world/removing-enemies.md): kill counts and `+0x90` to find a generator.
- [Live-testing tricks and pitfalls](../modding-scripts/live-testing-notes.md): watchers, write logs and quit-and-load testing.
