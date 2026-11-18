# The respawn point: death, Darksign, Homeward Bone, Aged Feather (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- **One record decides where you come back.** The game keeps a single record `{map, kind, id}` in the EventManager
  (+0x164 map, +0x168 kind, +0x16C id). A death, the Darksign, a Homeward Bone and the Aged Feather all read it. There is
  no separate "Darksign point" and "death point". [live A4, B19, B50c, B54] [code]
- **Kind says what `id` means:** 0 = a bonfire id, 1 = an event location (an `EventLocation_<map>` row), 2 = the start
  of the map. The map half is the packed map id, stored as the bytes `{00, 00, bb, aa}` for map `m<aa>_<bb>`
  (Majula `m10_04` = `00 00 04 0a`). [code] [live A4, 30 Oct]
- **Who writes it:** resting at or lighting a bonfire, the bonfire travel menu, and the script command `1:132141`
  (kind 1). A remote bonfire warp (a trainer or cheat-table warp) and the script warp `1:132125` do **not**.
  [live A3, A8, B10, B19, B39] [code]
- **Write all three fields or none.** Writing only the bonfire id hung the game on the loading screen (A4). Writing the
  map and the bonfire but leaving the kind at 1 sent every respawn to the map's origin, in the void (4 Nov death loop).
- **The trap:** a retry point (kind 1) inside a one-way boss room cannot be left: death, Bone and Darksign all took the
  player back in (B19), and the Feather reads the same record. Set the retry point at an arrival point beside a real
  bonfire, or in the hub.
- **The game sets kind 1 once itself.** Drangleic Castle's event group 17000 sets `{m20_21, 1, 1700000}` after the
  Queen's defeat, once per save. Pre-set its done flag 221000093 or it will fire in your arena. [data] [live 4 Nov, B14]

### The record

`EventManager = [[GameManagerImp] + 0x70]`. In the 1.0.3.0 exe the GameManagerImp pointer is the global at
`0x1416148f0` (RVA `+0x16148f0`), and every function below reads `+0x70` from it. [code]

| Offset | Size | Field | Values |
|---|---|---|---|
| +0x164 | u32 | map | packed map id `aa<<24 \| bb<<16 \| cc<<8 \| dd` for `m<aa>_<bb>_<cc>_<dd>`; for an area map `cc = dd = 0` |
| +0x168 | s32 | kind | 0 = bonfire, 1 = event location, 2 = map start |
| +0x16C | s32 | id | bonfire id (kind 0) or `EventLocation_<map>` row id (kind 1) |

The packing comes from the game's own map-name parser (`FUN_1403bf510`, which turns `"m20_21_00_00"` into the integer).
Stored little-endian, `m<aa>_<bb>` becomes the four bytes `{00, 00, bb, aa}`: [code]

| Map | Bytes at +0x164 | Seen where |
|---|---|---|
| m10_04 (Majula) | `00 00 04 0a` | A4, 4 Nov watcher log |
| m10_10 (Forest of Fallen Giants) | `00 00 0a 0a` | A4 (reads the same both ways, which hid the 30 Oct bug) |
| m10_14 | `00 00 0e 0a` | 30 Oct fix |
| m10_27 | `00 00 1b 0a` | A5 |
| m20_21 (Drangleic Castle) | `00 00 15 14` | 4 Nov watcher log |
| m40_03 (Dark Chasm of Old) | `00 00 03 28` | 4 Nov watcher log |
| m50_37 (Eleum Loyce) | `00 00 25 32` | A8 |

Defaults the game writes itself (`FUN_140040060`): `{m10_02, 2, 0}` for a new game (the start of Things Betwixt) and
`{m10_04, 0, 4650}` when the player is first sent to Majula. [code]

To write it from outside the game (this is what `tools/arenalib.py` does):

```
em = u64(u64(GameManagerImp) + 0x70)
write(em + 0x164, bytes([0, 0, bb, aa]))     # map
write(em + 0x168, int32(0))                  # kind: 0 = bonfire
write(em + 0x16C, int32(bonfire_id))         # id
```

### What kind means to the game

| Kind | Warp the game builds | Notes |
|---|---|---|
| 0 | type 3 (to bonfire `id`) | the bonfire must exist in the save's bonfire list |
| 1 | type 4 (to event location `id` in `map`) | a location row that does not exist in that map puts you at the map's origin |
| 2 | type 2 (the start of `map`) | the game's own new-game default; never written by us |

Two functions turn the record into a warp. They agree on the three kinds but differ in the details: [code]

| Function | Called by | What it does |
|---|---|---|
| `FUN_140184660` (RVA `+0x184660`) | wrapper `FUN_140452200` | kind 0 -> `FUN_1401843b0` (type 3; it looks up the **bonfire's own map** from the save's bonfire list); kind 1 -> `FUN_140184610` (type 4); anything else -> `FUN_1401845c0` (type 2). Two special cases: the exact record `{m20_21, 1, 1700000}` goes straight to a type-4 warp, and if neither bonfire 4650 (Majula) nor 2650 has its lit bit set, the record is ignored and the warp goes to the start of m10_02 (Things Betwixt) |
| `FUN_14044ed40` (RVA `+0x44ed40`) | `FUN_14044fde0`, `FUN_1401be677` | kind 0 -> a type-3 warp whose map field is the **stored map** and whose bonfire field is the stored id; kind 1 -> type 4; kind 2 -> type 2. It first checks the stored map id against a table at GameManagerImp +0x38 and, if it is not found, or the kind is not 0, 1 or 2, replaces the whole record with the default `{m10_02, 2}` (`FUN_14039a9a0`) |

Which of the two serves the death respawn, the Darksign, the Bone and the Feather was not traced. Live, all four
behaved the same way (A4, B50c, B54). [code] [inference]

The second function explains A4. With the stored map left at the Forest and the id changed to Majula's bonfire, a
type-3 warp asks the Forest to load and then waits for a bonfire that is not in it. That is the endless loading screen
with the bonfire animation, retrying about every 30 s. [inference, from A4 and the code; the death path was not traced]

The lit bit in the first function is bit 0 of the level byte in the bonfire list ([Bonfire lit state](bonfire-lit-state.md)):
`FUN_14017c2a0` returns `byte & 1`. [code]

### Who writes it

| Event | Record afterwards | How we know |
|---|---|---|
| Rest at a bonfire (lighting one probably too) | `{bonfire's map, 0, bonfire id}` | A5: a rest set `{m10_27, 27650}`. `FUN_1401caf50` and `FUN_1401cb950` call the setter with a bonfire object's map and id; which of them is the rest and which the lighting was not separated [live A5] [code] |
| Bonfire travel menu | `{bonfire's map, 0, bonfire id}`, **at once, before any rest** | A8, B10 (17:09:55). `FUN_14017fdb0` queues the type-3 warp, then calls the setter [live A8, B10] [code] |
| `1:132141(<location>)` | `{the map the script runs in, 1, location}` | B19 (`{m40_03, 1, 400004}`), B38 (`{m10_04, 1, 200000}`), B44 [live] |
| New game / first arrival in Majula | the two defaults above | [code] |
| Remote bonfire warp (trainer, cheat table) | **unchanged** | A3 correction: three warps left it alone. The wrapper (`FUN_140452140`) builds the warp and queues it; it never calls the setter [live A3] [code] |
| `1:132125` script warp | **unchanged** | B38 / B39: a whole 39-fight run of script warps left Majula's record in place [live] |

The setter is `FUN_14044fe30` (RVA `+0x44fe30`). It writes the three ints, and has one special case (below). Its four
direct callers are the travel-menu warp, the two bonfire-object handlers and the new-game defaults. [code] How the
script command `1:132141` reaches the record was not traced: a second look found no call to this setter from the
command handlers [code, 17 Nov; open].

Developer names of the script commands (from the `.edd` files): `1:132141` 【マップ】リトライポイント設定 ("[map] set
retry point"), `1:132125` 【マップ】プレイヤーをポリ劇ワープ ("[map] warp the player (polygon-drama warp)"), `1:132121`
【マップ】プレイヤーをエリア内でワープ ("[map] warp the player within the area"). Of the 28 `event_*` scripts in the
PC build, `1:132141` appears **once**: castle event group 17000. [data, scanned 17 Nov]

### Persistence

The record is saved. After a game restart it loaded as `{m10_10, 10655}` from the save, not the value that had been in
memory before (A7 note). [live A7] It also feeds the save-slot header: the writer `FUN_14019c190` uses `FUN_1401847f0`, a
test for exactly `{m20_21, 1, 1700000}`, to give that record a fixed location value. [code]

### The Throne of Want's own retry point

Castle event group 17000, 王妃撃破後のリトライポイント設定 ("set the retry point after the Queen's defeat"), runs two
sub-machines: a condition check, then `1:132141(1700000)` followed by `1:130101(221000093, 1)` (set the done flag, so it
never runs again). [data] The condition, as the script lists it:

- Nashandra's kill flag 221000096 on, 221000093 off, and flag 100978 (Vendrick's global) or 100747 (the Ashen Mist
  Heart's) off; that is, Nashandra killed when Aldia cannot come; **or**
- Aldia's kill flag 221000006 on and 221000093 off.

[data] The AND / OR reading of the condition groups is ours. [inference]

The game treats `{m20_21, 1, 1700000}` specially in three places: the setter (it also copies location data into a
player-data object; the effect was not tested), the respawn builder `FUN_140184660`, and the save-slot header. [code]

Why it matters for an arena: any save that kills Aldia (or Nashandra without Aldia's conditions) with 221000093 off gets
this record, and anything that then rewrites only the map and the id leaves the kind at 1. See the 4 Nov death loop below.
The mod's castle group sets 221000093 to 1 at every load, so the event cannot fire (`tools/choices.py`, the throne
group's `load` state). [data]

### A. Make a retry point from a script

1. Add an `EventLocation_<map>` row at the spot (a point). See [Warping the player by script](warping-by-script.md) for
   rows and coordinates. Pick a spot **outside** the fight, beside a bonfire the player can rest at (the mod uses a
   point about 1.1 m from the arena bonfire, 0.1 m up).
2. In that map's script, after the player has arrived, run `1:132141(<row id>)`. The record becomes
   `{this map, 1, row id}`. It stores the map the script runs in, so it must run in the arena's own map script. [live B19]
3. Check that the player can get out from there: rest at the bonfire (it sets kind 0 again and opens the travel menu), or
   win and warp out.

### B. Respawn in the hub first, then warp (the safe pattern)

Use this when a death, Bone, Darksign or Feather should send the player home, as in a boss rush, or when the arrival
spot has no bonfire. Majula's landing is `EventLocation_m10_04` row **200000**, a point that already existed (it is the
win warp's target). [data] [live B19: "kind 1 in Majula" works]

```
1:132141(200000)                                  # record = {m10_04, 1, 200000}
1:132125(0, 0, <map id>, <EventLocation>, 0)      # then the warp into the fight
```

The mod's fight list and the boss rush both do exactly this (`tools/fightlist.py`, `tools/bossrush.py`). [live B38,
B39] A death in the fight then loads Majula; so does the Bone or Feather. A later bonfire rest or travel sets kind 0
again by itself.

### C. A fight that retries in place after a death

This is what the fight list does for an ordinary arena since B44. Each arrival map's guard group (990003), once the
player is in that map: if the arrival token is on and no boss rush is running, run `1:132141(<arrival point>)`. The record
becomes `{this map, 1, arrival point}`; a death retries the fight, in one load, at the point beside the arena bonfire. In
a run the record stays Majula's, so a death ends the run. (`bossrush.respawn_points`, `bossrush.guard_spec`;
37 respawn states in 22 maps.) [live B44, B54, B55]

### D. Give the arena a real bonfire

A real bonfire next to the fog makes all of this unnecessary: travel and rest set kind 0, so death, Bone, Darksign and
Feather all land there. The Darklurker (bonfire 3650) and the Giant Lord (bonfire 10685) got their own bonfires on 10 Nov
for this reason, and the retry-through-Majula scripts were retired (B54, B55). See [Moving a bonfire](moving-a-bonfire.md)
and [Adding a bonfire](adding-a-bonfire.md).

### E. Write the record from outside the game

1. Write the map bytes `{00, 00, bb, aa}`, the kind (0) and the bonfire id. All three. (`arenalib.set_respawn` writes the
   map and id, then `fix_respawn_mode` sets the kind to 0.)
2. A write made while you are dead, before the respawn load, is used by that respawn (the trainer does this on every
   death). A write made during a hung loading screen is read on the next retry.
3. Recovery: if the game hangs on a respawn loading screen, write a valid record (Majula: `00 00 04 0a`, kind 0, id
   4650). The next retry (about 30 s) loads Majula [live A4, B6]. If the respawn lands at the map origin, write kind 0
   (`+0x168`) while you are dead; the next respawn is on solid ground [live 4 Nov, 11:08].
4. Never run a warp while dead; the respawn gets stuck (28 Oct, in the trainer's notes).

## What the mod does today

| Way into a fight | Record after arrival | A death, Bone, Darksign or Feather goes to |
|---|---|---|
| Bonfire travel menu to an arena bonfire | `{map, 0, bonfire}` (set by the menu) | that bonfire [live B10, B54] |
| Fight list, ordinary arena | `{map, 1, arrival point}` (set by the guard) | the arrival point beside the bonfire [live B44] |
| Fight list, Darklurker / Giant Lord (since 10 Nov) | the same, at their new bonfires | that bonfire [live B54 for the Darklurker: death and Feather both returned there; B55 for the Giant Lord reported as a pass, steps not itemised] |
| Boss rush (in order or shuffled) | `{m10_04, 1, 200000}` | Majula; the run ends [live B34 (death and Bone), B39 (the full run), B52 (death)] |

Before 10 Nov the two special fights had no bonfire of their own. The record stayed Majula's and a script sent you straight
back in after a death. See the history below.

### A3: a "pass" that was withdrawn (28 Oct)

A warp to Cardinal Tower with no rest, then a death, respawned at Cardinal Tower, so A3 was first marked a pass. The
same afternoon a watcher on the record showed three remote bonfire warps leaving it unchanged; the record had probably
already been Cardinal Tower. The pass was withdrawn. B10 and A8 settled the rest: the **travel menu** sets it, the
**remote warp** does not. The record flipped to Majula once, about 5 s after a warp, with no rest the player knew of
(14:48:55); that was never explained. [live A3 correction, A8, B10]

### A4: writing the bonfire alone hangs the game (28 Oct)

After a warp, the Darksign and a Bone both went to Cardinal Tower: one record, shared. Then only the bonfire id was
written (`Last Bonfire` -> 4650, Majula) and the player died by falling: **endless loading screen**, the bonfire
animation and tips running, a flash about every 30 s as it retried. Writing the map half to Majula released it. The
community cheat table's "Last Bonfire" entry points at the right field (+0x16C), which was a real help, but it is one
third of the record: write the map and the kind too. [live A4]

### The reversed map bytes (30 Oct)

The trainer wrote `{00, 00, aa, bb}` instead of `{00, 00, bb, aa}`. Every death respawned into a map that does not
exist. The game hung on the loading screen, and a restart came back at Things Betwixt (seen at the Darklurker, Freja
twice and Flexile Sentry). The Forest tests of 28 Oct had passed only because `m10_10` reads the same both ways
(`0a 0a`). The kill-counter slot headers use the same `{00, 00, bb, aa}` key, so the re-arm had been finding no kill
count either ([Bringing a boss back](respawning-bosses.md)). Fixed in `arenalib.set_respawn` / `kill_byte`; confirmed
live: "died, warped to bonfire, then auto-warped to the boss fog door". [live 30 Oct]
The restart landing at Things Betwixt fits the default in `FUN_14044ed40` for a map that is not in the table.
[code] [inference]

### The 4 Nov death loop: kind left at 1

At the Burnt Ivory King every respawn killed the player within about a second, first in his map, then in Majula after the
record was moved there. What the reads showed:

- Each respawn put the player at **(0, 0.1, 0)**, the map's own origin. In Majula that is empty space: HP full, a fall,
  then **HP exactly 0 after 0.9 s**. A kill plane, not damage (damage leaves HP below 0). Minimum HP 1 did not stop it.
- The 4 bytes at +0x168, between the map and the bonfire, held **1**. No note or cheat table named that field.
- Writing +0x168 = 0 while dead (11:08:43) fixed it: the next respawn stood on solid ground in Majula.

**Root cause (found in the code):** that field is the kind. The sequence that morning:

1. The first Aldia kill on that save (10:51:38) fired group 17000: record `{m20_21, 1, 1700000}`.
2. The trainer's respawn writes changed only the map and the id, so the kind stayed 1. The record became
   `{m10_04, 1, 4650}` after the warp home, `{m20_21, 1, 21665}` for the Darklurker (his death looked for event location
   21665; the trainer's teleport rescued the player), then `{m50_37, 1, 37670}` at the King: no such location, so the map's
   origin and the loop.
3. The afternoon could not reproduce it: 221000093 had been on since 10:51, so the event never fired again (a read-only
   watcher, `tools/respawnwatch.py`, logged every change of +0x164..+0x16F and saw the kind stay 0 in all five tests).

The first theory, that the Dark Chasm's own in-map warp turned the field on, was wrong. (That warp is a full reload of
the chasm map, `m40_03` unloading and loading, not a teleport.) [live 4 Nov] [code]

The fix is in the trainer: kind 0 at every respawn write and again on every death before the respawn load
(`arenalib.fix_respawn_mode`), logging "respawn +0x168 was 1" if it ever finds it on. The mod-side fix is the pre-set of
221000093. Anyone's save would hit this once, at its first Aldia kill. [code] [data]

The same event fired once more on 5 Nov (B14): at the Twin Dragonriders' load the record jumped to `{m20_21, 1, 1700000}`
because the trainer's leftover Nashandra and Aldia flags were on with 221000093 off. Harmless that time; the next bonfire
travel reset it. [live B14]

### B6: the death loop under the floor (4 Nov)

A moved bonfire placed 0.17 m under the floor dropped the player about 300 m, killed them at y about -316, and respawned
them at the same bonfire every 30 s. The way out was writing the record to Majula (as in A4). See
[Moving a bonfire](moving-a-bonfire.md). [live B6]

### B19: a retry point inside a boss room is a softlock (5 Nov)

The Darklurker's first build put a retry point in his boss room (`EventLocation` 400004 in `m40_03`). The record
`{m40_03, 1, 400004}` worked: a death reloaded the room with the fight running. But the next load went back into the room
too, and so did the Homeward Bone and the Darksign. There is no bonfire in the room, so no travel menu, and quitting
reloads in the room (the record is saved). **Nothing leaves it but a win.** Rebuilt (A'): no retry point in the chasm; the
warp state in Majula sets `132141(200000)` first, so a death goes to Majula. [live B19]

The Aged Feather (item 60355000) is a reusable Homeward Bone and reads the same record: it took the player to Majula
when the record was Majula's (B50c) and back to the bonfire when it was a bonfire (B54). That it would also loop back
into a retry-point room, as the Bone did, follows from that and was never run live; B50's design assumed it. [live B50c,
B54] [inference]

### B44: single-fight deaths went to Majula (8 Nov)

The fight list set the record to Majula before its warp (`132141(200000)`), copied from the boss rush, where a death must
end the run. For a single fight that meant "I die and I'm warped home". Bonfire travel never had the problem, because
the travel menu sets the record to that bonfire. The fix is pattern C above. The Giant Lord and Darklurker kept Majula
(B39 step 2). [live B44]

### B50, B50b, B50c: retrying the specials through Majula (8 Nov)

With no bonfire of their own, the two specials needed "die -> back in, Aged Feather -> stay home". The first two builds
looped on a Feather return. The reason was the death detection, not the Feather: the death count read back correctly
when stored, but the comparison with a later read fired anyway. The B50c log shows the old signal set one second after
arrival, with no death and no Feather.
The working signal was the player's HP below 1 after HP above 0 was seen in the same load. That story belongs to
[Detecting a player death](../modding-scripts/detecting-death-in-scripts.md). It was all retired two days later, when both fights got
bonfires (B54, B55). [live B50c]

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| One record serves death, Darksign, Bone, Feather | game behaviour | [live A4, B19, B50c, B54] |
| Layout: map +0x164, kind +0x168, id +0x16C | memory reads; decompile | [live 4 Nov, b2.py status] [code `FUN_140184660`, `FUN_14044fe30`] |
| Map bytes `{00, 00, bb, aa}` = packed `0xAABB0000` | memory reads; map-name parser | [live A4, 30 Oct] [code `FUN_1403bf510`] |
| Kind 0 / 1 / 2 = type 3 / 4 / 2 warp | decompile | [code `FUN_140184660`, `FUN_14044ed40`] |
| Kinds 0 and 1 seen live; kind 2 never written live | memory reads | [live B19, B54] |
| Bonfire id alone hangs the load | live | [live A4] |
| Reversed map bytes hang; restart at Things Betwixt | live | [live 30 Oct] |
| Kind 1 + a bonfire id puts you at the map origin; kill plane at 0.9 s | live, watcher | [live 4 Nov] |
| +0x168 = 0 while dead cures it | live | [live 4 Nov 11:08:43] |
| Rest / light sets the record | live, decompile | [live A5] [code `FUN_1401caf50`, `FUN_1401cb950`] |
| Travel menu sets it before any rest | live, decompile | [live A8, B10] [code `FUN_14017fdb0`] |
| Remote bonfire warp does not | live, decompile | [live A3 correction] [code `FUN_140452140`] |
| Script warps `1:132125` do not | live | [live B38, B39] |
| `1:132141` stores {running map, 1, location} | live | [live B19, B38, B44] |
| Only castle group 17000 uses `1:132141` among the 28 event scripts | script scan | [data, 17 Nov] |
| Group 17000's condition and flags | script dump | [data `event_m20_21`, 17 Nov] |
| Three special cases for `{m20_21, 1, 1700000}` | decompile | [code `FUN_14044fe30`, `FUN_140184660`, `FUN_1401847f0`] |
| First Aldia kill fired it and started the loop | trainer log, watcher | [live 4 Nov] |
| A kind-1 retry point in a boss room traps the player | live | [live B19] |
| Majula's `EventLocation` 200000 works as a kind-1 point | live | [live B19, B38, B39] |
| The Aged Feather follows the record | live | [live B50c (to Majula), B54 (to the bonfire)] |
| The Darksign and Bone follow the record | live | [live A4 (to the warp's bonfire), B34 (to Majula in a run), B19 (looped into the retry room)] |
| Guard-set kind 1 at the arrival point retries a fight | live | [live B44] |
| A bonfire beside the fog makes death, Bone and Feather land there | live | [live B54; B55 pass reported without itemised steps] |
| The record is saved | live | [live A7] |
| Two builders differ on the map half (A4 mechanism) | decompile | [code] [inference] |
| An unknown bonfire id falls back to Things Betwixt | decompile of the warp queue (`FUN_140184830`) | [code] [untested live] |

## Tools

| Tool | What it does for this topic |
|---|---|
| `tools/arenalib.py` | `set_respawn(map, bonfire)` writes `{00, 00, bb, aa}` and the id, then `fix_respawn_mode()` sets kind 0 (returns the old kind); `respawn()` reads the map bytes and id |
| `tools/setmap.py` | writes only the map half (the A4 experiment; unsafe on its own) |
| `tools/respawnwatch.py` | read-only watcher: logs every change of +0x164..+0x16F, the loaded map, HP crossing 0 and position jumps, to the millisecond |
| `tools/retrywatch.py` | read-only watcher for the B50c retry flags beside the record |
| `tools/b2.py status` | reads the record `{map, kind, id}` and prints the kind in words |
| `tools/arenactl.py` | the trainer: F5 sets the record to the boss's bonfire, F6 sets Majula and warps home; logs when the kind was not 0 |
| `tools/bossrush.py` | `respawn_points(map)` and `guard_spec(respawn)` build the guard group's `1:132141` states |
| `tools/fightlist.py`, `tools/specials.py` | the fight list's and the specials' "record on Majula, then warp" states |

## Open questions

- Which of the two builders serves the death respawn, and which the Bone, Darksign and Feather? Live they behave the
  same. [untested]
- Kind 2 has never been written live. The code says it warps to the start of the map. [untested]
- The record flipped to Majula once on 28 Oct with no known rest (A3, 14:48:55). Cause unknown.
- What happens to a save whose record names a mod-only bonfire (3650, 10685, or a moved bonfire's arena) when the mod is
  removed? The warp queue replaces a warp whose map is unknown by the start of Things Betwixt, but the other builder may
  wait for a bonfire that is not there, as in A4. The README warns against loading an arena save without the mod.
  [untested] See [Saves and installs](../modding-scripts/saves-and-installs.md).
- What does the setter's extra copy for the Throne record do to the player data? [untested]
- The Darklurker's return hole (region 401000) was never checked as an exit during the fight, because the design changed
  after B19. [untested]
- On a fresh character who has not lit Majula (bonfire 4650) or bonfire 2650, the builder ignores the record. Does an
  arena save ever hit that? Majula is lit by the setup, so probably not. [inference]

## Related pages

- [Moving a bonfire](moving-a-bonfire.md): the landing spot, the floor, and the death loop under the floor
- [Adding a bonfire](adding-a-bonfire.md): the real-bonfire answer
- [Bonfire lit state](bonfire-lit-state.md): the level byte, the lit bit, the travel menu
- [Warping the player by script](warping-by-script.md): `1:132125`, EventLocation rows, arrival points
- [Bringing a boss back](respawning-bosses.md): the kill-counter slots use the same map bytes
- [Detecting a player death](../modding-scripts/detecting-death-in-scripts.md): the B50 attempts in full
- [Memory map for PC 1.0.3.0](../tools/memory-map-pc.md): GameManagerImp -> EventManager and the rest of the offsets
- [Saves and installs](../modding-scripts/saves-and-installs.md): the record is in the save
- [Giving the player items](../modding-menus/giving-items.md): how the Aged Feather reaches the player
