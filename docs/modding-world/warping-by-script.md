# Warping the player by script: arrival points (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- **Two script commands move the player.** `1:132125(0, 0, <map id>, <EventLocation row>, 0)` warps to a row in any map,
  **including the map the script is running in, and it always loads**: the map unloads and loads again, about 3 s from the
  command to the new map [live 4 Nov, B34, B38, B39, B52]. `1:132121(<row>)` moves the player inside the current map. The
  game uses it only at the end of cutscenes (eight boss cutscenes and one more), and we never called it [data; live B39].
- **The target is a row of `EventLocation_<map>`**, a per-map param that loads with its map. Its coordinates are in the
  map's local frame: MSB position minus the map's "map offset" event. The player lands on the row's position [live 4 Nov].
- **Our arrival rows:** 38 new rows, ids 990701-990739, in 22 maps. Each is 1.1 m behind its arena bonfire's part along the
  part's facing, 0.1 m above the part, with the part's yaw. Build one from a bonfire part and the map offset, add the row,
  and warp to it [data; live B34, B39, B54, B55].
- **A script warp changes only where the player is.** It does not write the respawn record [live B38, B39]; event flags
  carry across the load.
- **Main traps:** every load runs every load-time script, including those of a map that is only *streaming in* beside you
  [live B39]; a game script that reads a flag once at load can look before your write [live B39]; and a rest at a bonfire,
  a death, a quit and a fall into the Dark Chasm's holes are all loads too [live B38, 4 Nov].

### The commands

| Command | Developer name (`.edd`) | Arguments | Notes |
|---|---|---|---|
| `1:132125` | 【マップ】プレイヤーをポリ劇ワープ, "[map] warp the player (polygon-drama warp)" | `(a, b, map, row, c)` | the warp we use. Game: 25 calls in 18 scripts. [data] |
| `1:132121` | 【マップ】プレイヤーをエリア内でワープ, "[map] warp the player within the area" | `(row)` | game: 9 calls in 8 scripts, all in cutscene-playback groups. [data] |
| `f132101()` | 【マップ】ポリ劇ワープ終了判定, "polygon-drama warp finished?" | none | the game's own wait after a `132125`. We do not use it. [data] |

The commands `1:132122` ("warp an enemy within the area") and `1:132123` ("warp oneself within the area") are named in the
`.edd` files, but no script in the PC build calls them. [data]

**The arguments of `1:132125`:**

| Position | What we pass | What we know |
|---|---|---|
| a, b | `0, 0` | In the game's own entries into the giant memories, the Vagrant hawk's flight, the compass ship and the Ancient Dragon's memory, these are six-digit numbers (101070, 101050, 101060; 101010 and 101620; 101820; 101420). They follow the numbering of the cutscene ids the game gives its cutscene command `1:130801` (102310 for the Executioner's Chariot). So they are probably the ids of a cutscene played leaving and arriving, and `0` means none. [inference] |
| map | the packed map id | `aa*1,000,000 + bb*10,000 + cc*100 + dd` for `m<aa>_<bb>_<cc>_<dd>`: m10_04 is `10040000`, m50_37 is `50370000`. [data; `bossrush.map_id`] |
| row | an `EventLocation_<map>` row id | a row that exists in that map's param. [live] |
| c | `0` | The game passes 0 everywhere except two coffin warps: 1 in Things Betwixt's coffin and 2 in the Frigid Outskirts' coffin bobsled. Both land inside their own map, as do other calls that pass 0 (the chasm's drop to the boss room). We never tried either value. Meaning unknown; it may pick a transition style. [data; inference] |

The game's own exits show the pattern we copied. Four maps have a group that ends with `132125(0, 0, 10040000, 200000, 0)`,
Majula's landing row, which is the row our win flow warps to [data]. Its shared library group for `132125` calls the command
in one state and moves on when `f132101()` says the warp has finished. Ours end the state and let the load take the script
away. [data; live B8 onwards]

### What a `1:132125` does

| Step | Measured | Where |
|---|---|---|
| Herald's window closes, then the warp | the group waits 2 s (`f3() > 2`) | B34, `tools/bossrush.py` `majula_spec` |
| command, then the new map in memory | 3 to 3.5 s | B38 (14:01:08 to 14:01:11), B39 (+8.3 s to +11.8 s), B52 (3 s after each pick) |
| the old map's groups meanwhile | **keep running** until the unload | B38: the guard cleared the token 2 s after the reload command, with the warp still pending |
| map unloaded, then the player alive in the new map | about 0.65 s in the debug build | 4 Nov watcher: 11:23:20.563 to 11:23:21.199, 11:23:26.308 to 11:23:26.963 |
| kill flag, then home (the win flow) | 8 s: flag, 1 s, flag off, 7 s, warp | `arenamod.WAIT`; B8: kill 15:52:14, in Majula 15:52:25 |

**It is a full load, in the same map too.** Three live results:
- The Dark Chasm's own hole warp is a `132125` into the chasm's own map (data, below). The watcher saw the map list go
  empty and the chasm load again, then the player appear [live 4 Nov].
- Our stale-arrival reload warps to a fight's own arrival row from inside that map. B38 reported a "double load" at the
  same spot, and the boss was back after the second one [live B38, steps 1 and 2].
- The game's own warp into the Frigid Outskirts (inside m50_37, last argument 2) showed a loading screen that reloaded
  the same flag blocks and no others [live A8 (c)].

**What carries across and what does not:**
- Event flags in the global blocks carry across. The boss rush keeps its state in flags 104700-104707, set before the warp
  and read after it [live B38, B39].
- A stored script variable did not compare equal to the value it stored, so the mod keeps its state in flags. The notes
  are on [Detecting a player death](../modding-scripts/detecting-death-in-scripts.md) and [Event script groups](../modding-scripts/event-script-groups.md). [live B38, B50c]
- The warp restores nothing. The rush is designed so that HP and items carry from fight to fight, and a full 39-fight run
  was played that way [design; live B39]. Restoring is a separate effect: [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md).
- The respawn record is left alone. A run of script warps left Majula's record where the run's start had put it [live B38,
  B39]. Why this is not what A3 showed is in the pitfalls. See [The respawn point](respawn-point.md).

**The landing is exactly on the row.** The castle's summoning circle runs `132125(0, 0, 40030000, 400020, 0)`. Row 400020
plus the chasm's map offset is (-895.05, 20.49, 303.75) in the MSB frame, and the watcher read the player at
(-895.0, 20.5, 303.7) the moment the chasm loaded. The hole's target rows (400003 to 400005) plus the offset give
(-1007.54, 13.15, 272.28); the watcher read (-1007.5, 13.3, 272.3) [data; live 4 Nov, `tools/respawnwatch.py` log].
The player's own y read 0.15 m above the row's, after settling. The yaw was never read. We set each row's yaw to the part's
yaw and assume the warp uses it. [inference]

### `EventLocation_<map>` rows

One param file per map, `Param/eventlocation_<map>.param`, 48 bytes a row. It is not in the regulation, so adding rows
needs no regulation edit ([Editing params](../modding-scripts/editing-params.md)). Layout, with the project's field names, read from the files:
[data]

| Offset | Type | Field | Notes |
|---|---|---|---|
| +0x00 | f32 x3 | `PositionX/Y/Z` | local frame |
| +0x0C | f32 x3 | `RotationX/Y/Z` | yaw is `RotationY`, degrees |
| +0x18 | u8 | `RegionTypeMaybe` | 1, 2 or 3; see below |
| +0x19, +0x1A, +0x1B | u8 | `Unk19`, `Unk1A`, `Unk1B` | unknown. Ours copy Majula's row: 0, 255, 0 |
| +0x1C | f32 x3 | `ScaleX/Y/Z` | box size for type 3. Ours: 0.5, 1, 1 |
| +0x28, +0x2A, +0x2C | u16, u16, u32 | `Unk28`, `Unk2A`, `Unk2C` | unknown. Ours: 0, 2, 0 |

- **Type 1 is a point; type 3 is a box.** The 28 maps hold 4,743 rows: 2,184 of type 1, 404 of type 2 and 2,155 of type 3.
  Every warp target we looked up is type 1: Majula's 200000, the giant memories' 3700000 to 3702000, the chasm's
  400003 to 400005 and 400020, Things Betwixt's 500000, and the castle's 1700000. Our rows copy Majula's shape. A warp to a
  box row was never tried. [data; the names "point" and "box" are our reading of the scales]
- **The same table holds the regions that scripts test.** The chasm's holes are type-3 boxes (400000 to 400002) whose
  warps target the points 400003 to 400005; the Giant Lord's fight-start region 800000 is a 15 x 5 x 4 box. Talk scripts ask
  "is the player in this row?" with `f131101(<row>)` (【ポイント】プレイヤーのポイント内判定, "player inside the point"; 7
  uses in 3 talk scripts). Event scripts do it inside library groups: the Giant Lord's group 8000 passes row 800000 to one.
  [data] Ids are shared, so a new row must not reuse one a script already tests. In the 28 maps no original row has an id from 990000
  to 990999. [data]
- **An arrival inside a trigger region fires it.** The chasm's boss-room rows (400003 to 400005) are one point at the centre
  of the fight-start box 200000, so arriving there starts the fight at once [live B19: "fight on arrival"; data]. Our arena
  arrivals sit outside the fog, and the Giant Lord's lies 12 m before the fog and his start box begins 2.5 m past it, so you
  walk in. [data, from the project's 5 Nov reading of the scripts]

### The coordinate frame

`EventLocation` rows use the map's local frame. The MSB, and the live game, use `local + offset`. (The project's tool notes
say the same of `GeneratorLocation` rows; not re-checked for this page.) The offset is the MSB event named `マップオフセット`
("map offset"), three floats at +0x30 of its entry in `EVENT_PARAM_ST`. [data] Examples:

| Map | Offset | Map | Offset |
|---|---|---|---|
| m10_04 (Majula) | (0, 0, 0) | m10_10 (Forest of Fallen Giants) | (208, 10, -133) |
| m10_16 (Lost Bastille) | (-78, 4, 562) | m40_03 (Dark Chasm of Old) | (-1048, 53, 376) |
| m20_10 (the memories) | (205.996, -4, -175.606) | m50_35 to m50_38, m20_26 | no entry: zero |

22 of the 28 PC maps carry the entry. The other six (Majula, m20_26, m50_35 to m50_38) have none, and the project's reader
returns zero for them. Zero is right for Majula, where the game's own row sits 1.16 m from the bonfire part, and for the DLC
arenas, whose arrival rows were built with zero and passed live [data; live B39]. `tools/bosstable.py` `msb_objects(<map>)`
returns a map's objects and its offset. The 38 rows in `tools/bossrush_points.json` carry the offset of their map, and each
equals the stored MSB-frame landing minus that offset. [data, checked 17 Nov]

### Placing an arrival row

The landing rule of a bonfire ([Moving a bonfire](moving-a-bonfire.md)) is also the rule for our rows:

```
landing  = part - 1.1 * (sin yaw, 0, cos yaw)     in x, z      (MSB frame)
y        = part y + 0.1
row      = landing - map offset                  RotationY = part yaw
```

Worked example, the Last Giant: part at (92.43, -40.40, -146.99), yaw 90. Landing (91.33, -40.30, -146.99). Offset of m10_10
is (208, 10, -133), so the row is (-116.67, -50.30, -13.99), yaw 90: row **990701** in `eventlocation_m10_10_00_00`
[data; code `tools/bossrush.py` `LAND` 1.1, `LIFT` 0.1]. All 38 rows in the debug copy match the numbers
`bossrush.location_rows` computes. [data, checked 17 Nov]

The 0.1 m lift keeps the player off the floor surface. It was our choice, not measured to a limit. [inference] The game's own
Majula row 200000 sits 1.16 m from its bonfire part and 0.09 m above it, about 0.36 m from where a bonfire warp lands.
[data]

**Row ids.** `id = 990700 + the fight's place in the 39-fight list` (`bossrush.loc_id`). 990725 does not exist, because
Vendrick shares Velstadt's 990724. The two special fights have rows too since 10 Nov: 990728 (Giant Lord, bonfire 10685) and
990729 (Darklurker, bonfire 3650). There were 36 rows at the first live test (B34), 37 after B54 and 38 after B55. All 38
are frozen in the id register, because a respawn record can name one (`tools/ids.py`, `loc.<row>`). [data; live B34, B54,
B55] See [Adding a bonfire](adding-a-bonfire.md).

## How to do it

A recipe for "arrive at a spot next to a bonfire, by script".

1. **Choose the spot.** Take it from a bonfire part with the rule above, or read it by standing there. Use the local frame
   for the row: subtract the map's offset.
2. **Add the row.** Copy an existing point row and set all sixteen fields. With the project's tool:
   `tools/paramrows.py add eventlocation_<map> <new id> <copy of id> PositionX=... PositionY=... PositionZ=... RotationY=...`
   (`bossrush.add_locations` does a whole map). It writes into the game copy's `Param` folder and keeps a `.orig` of the
   file. Use an id in 990000-990999. [code; live B34]
3. **Install it.** The per-map param is read when its map loads. B34 installed with the game closed; B54 installed it with
   the game running and the map loaded afterwards. [live B34, B54]
4. **Issue the warp** from a script state that is **not the group's first**: `1:132125(0, 0, <map id>, <row>, 0)`. A group's
   first state ignored entry commands in our first build (B9; see [Event script groups](../modding-scripts/event-script-groups.md)).
5. **Do the set-up in the same state, before the warp.** Set every flag the target's load-time scripts will read. The
   target map's scripts run the moment it loads. The mod puts an arrival token on (and a "reloaded" marker off) in the
   command list directly before every warp to a fight (`bossrush.arrival_cmds`). [live B38]
6. **Make the target's load-time logic wait for the player.** `1:140402(0, 1, 0)` in one state, then move on when
   `f140001(0)` is true, with a 3 s exit. This keeps a neighbouring map from acting for you. [live B39; Event script groups §7]
7. **Treat the warp as terminal.** Leave the state with no commands after it. Everything of the old map goes at the unload,
   and flags are the only thing the next load can read.
8. **Check every warp in the build.** `tools/arrival_check.py` reads the built scripts and fails if a warp to a fight's arrival
   row lacks its token, if an arrival map has no guard, or if a fight has no in-place reload. [code]

To **reload the current map in place** (re-arm a boss, restart a stage): warp to a row *in that map*, with a marker flag
set first so the second load does not reload again. B38's stale-arrival chain does this: clear the stale flags, wait 1 s,
set `RELOADED`, warp to the fight's own arrival row. One reload per arrival. [live B38]

## What the game's own scripts do

A scan of the PC build's 28 `event_*` and 25 `talk_*` scripts (originals where the mod changed one), 17 Nov. [data]

- **`1:132125`, 25 calls in 18 event scripts, none in a talk script.** Thirteen of the 18 carry the shared library group
  "inter-map warp after polygon drama" (【Lib】ポリ劇後マップ間ワープ). The calls are: entries to the memories, their exits and
  their time limits; the Vagrant hawk's flight (ベイグラント鷹タクシー, ["Vagrant hawk taxi"](../network-test/boss-region-names.md#the-pursuer-was-vagrant)) and the compass ship; travel
  between the main game and the DLC; the Pilgrims
  of the Dark's circles (a `132125` into m40_03); the chasm's holes; the two coffins; and bonfire-exit groups in four maps
  (m10_14, 16, 19, 25) that end at Majula's row 200000.
- **`1:132121`, 9 calls in 8 event scripts** (m10_10, 14, 16, 19, 23, 25, m20_21, 24). Each is the end of a cutscene
  group: play the cutscene (`1:130801`, 【ポリ劇】ポリ劇再生, "play polygon drama"), sync remote players (`1:130641`), set a
  flag, `132121(<row>)`, then save (`1:132241`, 【その他】セーブ実行). Eight are boss cutscenes and one is the Vagrant's. So it
  is a move to a point inside the map, with no load. [data]
- **The chasm holes** (m40_03 groups 4000, 4010 and 4020): a library group tests the hole's region and the path flags. Our
  reading, from the project's 5 Nov notes on the scripts: with all three path flags on it runs `132125(0, 0, 40030000, <boss-room row>, 0)`
  and `1:132241` (save); otherwise it warps back to the map you came from, row 400000. The boss room's return holes also warp
  to the map you came from, row 400000, and save. [data]
- **`1:132141`** (the respawn command) is used once, in the castle: [The respawn point](respawn-point.md).

### The respawn record and A3

The project's design notes and the rush tool's docstring cite A3 as the proof that script warps leave the respawn record
alone. A3 tested the trainer's remote bonfire warp, not `1:132125`. The real evidence is B38 and B39: a whole 39-fight run of
script warps left the record where the run's start had put it, on Majula's landing. [live B38, B39] This page cites those.
The topic is on [The respawn point](respawn-point.md).

### The trainer's remote event warp (28 Oct)

The trainer's own call to the game's event warp (warp type 4: a map and an event location, no script involved) worked twice,
then reloaded the Forest instead, then the game crashed with heap corruption (20:58). It was never used again, and the
Giant Lord and the Darklurker were reached through the game's own entries. [live 28 Oct] On 6 Nov the project found the same
crash signature (0xc0000374 at the same ntdll offset) in the Windows event log on 28 Oct, 20:58, before any arena map edit,
and the 6 Nov dump scan put that 28 Oct dump in the same chain as the others, inside the debug build's DebugManager
add-on, not in the warp [live B27 and its correction; see [Diagnosing crashes](../modding-scripts/crash-diagnosis.md)]. So the warp is not
shown to have caused the crash; its wrong-map reloads stand on their own. The call was never retried.
Since the add-on was switched off (B33, 6 Nov) the script warps have run through every test with no crash blamed on them.
[live B34 onwards]

### B39: Flexile Sentry arrived in the sea (7 Nov)

A rush arrival at the Wharf landed in the water and the player died; the boss fell too. The Wharf's ship group (m10_18 group
1000, 鐘による船移動, "ship moved by the bell") looks at the ship flag 118000010 **once, at load**: on = ship docked and deck
shown; off = deck hidden. Our arena group set the same flag at load, and the game's look won the race on a save without the
flag. The ship flag lies in that map's own block, which the project's notes treat as kept only while the map is loaded, so
the arrival cannot set it ahead of the load [inference, per Event script groups §8]. The fix is a patch of the game's group: its first state now sets the flag,
then goes on to the ship preset (`tools/specials.py` `wharf_ship_patch`). [live B39; the cause is inference: the order groups
start in was never read] See [Event script groups](../modding-scripts/event-script-groups.md).

### B39: a neighbouring map ended the run (7 Nov)

After Najka's kill the run ended at once: 16:10:40 map m10_33 (the next arena along) streamed in while the player was still in
m10_32, and at 16:10:44 the player was in Majula. A streamed-in map **runs its event scripts**, so m10_33's guard saw "run on, no
token" and sent the player home. B38 never met it because no arena had a neighbour that streams in. Fix (B39c): the guard,
each fight's stale reload and Majula's run check all wait for `1:140402(0, 1, 0)` then `f140001(0)`, with a 3 s exit.
It is byte for byte the wait the game's own groups use: 176 states in 25 original scripts have the entry and the test
(Majula's group 111416 is one). [live B39; data]

### B38: what counts as a load

- **A rest reloads the map, even with the bonfire menu open.** At the Dragonrider's arena bonfire the player rested twice,
  the second time staying in the bonfire menu until the warp home came: the guard fired both times. So a rest (and a death,
  and a quit and continue, and bonfire travel into an arena) is a load with no token. [live B38 step 3]
- **A quit mid-run is a load too.** By design the guard ends a run on a quit and continue, because it sees a load with no
  token. Not cleanly isolated live: the one in-map ending (14:33) may have been a rest. [live B38, partly]
- **The token is cleared 3 s after load, not at load.** Every group in the map must read it first, and the guard clears it.
  A quit within 3 s of an arrival still counts as that arrival. [live B38; code]

### The Dark Chasm's holes are a reload (4 Nov, B38, B54)

The chasm's own drop to the Darklurker is a `132125` into its own map: the map unloads and loads again (the
watcher's map list went empty, then showed m40_03 again; 4 Nov). Two consequences for the mod:
- The **boss rush arrives in the boss room** (row 400004) and does not use the bonfire's landing. A fall into a hole is a
  load with no token, so the guard ends a run on it, as on a death. [live B38, B39, B54]
- The **fight list lands at the chasm bonfire** (row 990729) and its guard sets the respawn there first. A drop is a reload,
  yet a death after it comes back to the bonfire in one load, because the record survives the load. [live B54]

### B19: a retry point in the boss room

Not a warp fault, but it came from the same arrival: the Darklurker's first build warped to 400004 and its chasm group set
the respawn to that row, and nothing then led out of the room. The respawn half is on
[The respawn point](respawn-point.md). [live B19]

### A code review of 7 Nov: "no save stores a new row id"

The review stated that the rush only warps and never sets a respawn, so uninstalling cannot leave a save pointing at a
missing arrival row. That was true on 7 Nov. Since B44 (8 Nov) the guard sets `1:132141(<arrival row>)` after a list trip, so a
save's respawn record can name one of our rows. Those rows are frozen in the id register. What a vanilla game does with
such a record is untested. [live B44; see Respawn-Point and [Saves and installs](../modding-scripts/saves-and-installs.md)]

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| `132125` into another map loads it | every rush leg | [live B34, B39] |
| `132125` into the current map reloads it | stale-arrival "double load" with the boss back | [live B38] |
| The game's own same-map hole warp is a reload | map list empty, then the same map | [live 4 Nov]; [data g4000, g4010, g4020 of m40_03] |
| The Frigid Outskirts' same-map warp (last argument 2) loads | loading screen, same flag blocks reloaded | [live A8 (c)] |
| Landing = row + map offset, at the row's height | two live readings in the chasm | [live 4 Nov] [data] |
| Command to new map: 3 to 3.5 s | timed warps | [live B38, B39, B52] |
| Old map's groups run until the unload | token cleared while a warp was pending | [live B38] |
| Script warps leave the respawn record alone | a 39-fight run; list deaths go to Majula | [live B38, B39] |
| `132121` is used only in cutscene-playback groups | scan, 9 calls in 8 scripts | [data, 17 Nov] |
| A cutscene's own move is not a load | no run ended by one in 39 fights | [live B39] [inference] |
| First two arguments of `132125` are cutscene ids | numbering matches `1:130801` ids | [inference] |
| 38 arrival rows equal the computed landings | rows read back from the debug copy | [data, 17 Nov] |
| No original row has an id 990000-990999 | scan of 28 per-map params | [data, 17 Nov] |
| Region types: 2,184 / 404 / 2,155 of 4,743 | scan | [data, 17 Nov] |
| A streamed-in map runs its scripts | run ended at Najka by m10_33 | [live B39] |
| The wait pair is the game's own | 176 states in 25 original scripts | [data, 17 Nov] |
| The Wharf ship flag is read once at load | script read; fix passed | [data] [live B39] |
| A rest reloads the map with the menu open | guard fired twice | [live B38 step 3] |
| The trainer's remote event warp was unsafe | crash 28 Oct, cause not proved | [live 28 Oct] [live B27] |
| Respawn rows can be named by a save | guard sets kind 1 after a list trip | [live B44] |

## Tools

| Tool | What it does for this topic |
|---|---|
| `tools/bossrush.py` | `map_id`, `loc_id`, `make_points` (landings and offsets into `bossrush_points.json`), `location_rows` and `add_locations` (the rows), `warp_cmds`, `reload_cmds`, `arrival_cmds`, `guard_spec`; `bossrush.py points` recomputes the points |
| `tools/bossrush_points.json` | each arrival: map, map offset, MSB-frame landing, yaw |
| `tools/paramrows.py` | adds a row to `eventlocation_<map>.param`, copied from an existing row, with field edits; checks that an unchanged rebuild is byte-identical |
| `tools/bosstable.py` | `msb_objects(<map>)` returns a map's object parts and its map offset |
| `tools/arrival_check.py` | checks guards, reloads and tokens in a built copy (`--game` for the debug copy, read-only) |
| `tools/ids.py` | the id register; `loc.<row>` freezes the 38 arrival rows |
| `tools/esd.py`, `tools/edd.py` | dump a script and read the developer names used on this page |
| `tools/respawnwatch.py` | read-only watcher: map list, player position and the respawn record to the millisecond (the 4 Nov readings) |
| `tools/specials.py` | `wharf_ship_patch`; the special fights' own warp states |

## Open questions

- **The last argument of `1:132125`** is 0 in all of ours and 1 or 2 in the game's two coffin warps. Does it change how the
  load looks or happens? Both coffin warps stay in their own map, and the Frigid Outskirts one (2) still showed a loading
  screen [live A8 (c)]. A same-map warp with no reload would be useful. [untested]
- **The first two arguments.** Does a non-zero first argument play a cutscene on the way? [untested]
- **Which way does the player face on arrival?** We set the row's yaw to the part's yaw and never read the facing after a
  script warp. [untested]
- **A warp to a row that does not exist in the target map.** A kind-1 respawn to a missing row sent the player to the map's
  origin (4 Nov). A script warp to one was never tried. [untested]
- **A warp to a type-3 (box) row.** Every game target is a point. [untested]
- **`1:132121` as a no-load move** for our own use. We never called it. It would not re-arm anything the way a reload does.
  [untested idea]
- **Where the command enters the loader.** A first look at the callers of the warp builders the bonfire and respawn paths share
  found no script-command handler among them, so the route of `1:132125` into the loader was not traced.
  [code, partial]
- **A script warp while the player is dead,** and one in a co-op session. The trainer's remote warp stuck when called while
  dead, and the game's cutscene groups sync remote players with `1:130641` before `132121`. We play offline. [untested]
- **Does a warp save?** The game's hole and cutscene warps are followed by `1:132241`; ours are not. Whether a quit just
  after a warp resumes at the arrival is unchecked. [untested]

## Related pages

- [The respawn point](respawn-point.md): what a death, the Darksign, a Homeward Bone and the Aged Feather return to, and
  `1:132141`
- [Moving a bonfire](moving-a-bonfire.md): the landing rule and the floor
- [Adding a bonfire](adding-a-bonfire.md): the arrival rows 990728 and 990729 and the map offset on a new part
- [Event script groups](../modding-scripts/event-script-groups.md): groups, the first-state rule, the player-in-map wait, the Wharf patch
- [Detecting a player death](../modding-scripts/detecting-death-in-scripts.md): state across loads
- [Editing params](../modding-scripts/editing-params.md): why a per-map param needs no regulation edit
- [Script command reference](../modding-scripts/script-command-reference.md): the commands used here
- [Saves and installs](../modding-scripts/saves-and-installs.md): respawn records that name mod rows
