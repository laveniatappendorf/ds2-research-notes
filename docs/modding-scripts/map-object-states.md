# Map object states (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.
> Test ids such as B25 point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags: `[live B25]` seen in the game,
> `[code]` read in the exe, `[data]` read in the game files, `[inference]` reasoned, not proven, `[untested]` an idea
> only. `[live 30 Oct]` is a dated trainer session, not a numbered test.

## Summary

- **Every map object that has a state keeps one number, its object state, in a saved table of `{u32 entity id, u32 state}`
  pairs.** The table sits beside the map's event flags, the game reads it at the next load of the map, and an object
  nobody has touched is at **10**. [live 30 Oct]
- **The numbers belong to the object, and the game's load scripts read them.** Fume Knight's Ashen Idols: 33 / 34
  intact, 20 destroyed. Mytha's five poison pools: 10 poison, 20 drained. The Belfry gate: 10 closed, 70 opening, 20
  open. For the idols the load script reads the state and then sets the flag, not the other way round. [data]
- **A script can only request a change (`1:131636`), and some requests are not taken.** A request that follows what the
  object does by itself worked (the Belfry gate, 10 to 70 to 20). Requests that put an object back (idols 20 to
  intact, pools 20 to 10) did not. [live B19] [live B25]
- **A memory tool can rewrite the table, but only in the ~0.35 s between unload and load.** On unload every live object
  writes its state back into the table, so an early write is lost. A thread that rewrote the wanted states every 2 ms
  restored and destroyed the idols and put the poison back, each in one load. [live 30 Oct]
- **Where the mod ended.** Mytha's room is always drained, by a flag patch that makes the game's own drain run at every
  load, not by a state write. The idols are left alone: the setup gives six Smelter Wedges and the player wedges them
  once, after which they stay wedged. [live B26] [live B49]
- **The main trap: changing the flag instead of the object.** The idols' flags (536000041-44) and Mytha's flags
  (117000055 / 056) are results of the load script, not inputs to the objects. Setting a flag alone should not change
  the object, because the script reads the object. No flag-only test was recorded. [inference from the scripts; the B25
  poison result shows the split]

### 1. What an object state is

An object part in a map's MSB (type 1, for example an idol `o36_1250` or a pool `o17_1060`) carries an **entity id**. The
same number is the object's row in `MapObjectInstanceParam_<map>` (see [Moving a bonfire](../modding-world/moving-a-bonfire.md) for the same
link on bonfires). Event scripts name an object by that id.

Two script commands touch an object's state. Their developer names come from the `.edd` files: [data]

| Id | Developer name | Gloss | Arguments |
|---|---|---|---|
| `1:131636` | 【OBJ】OBJステート遷移リクエスト（インスタンスID指定） | request an object state transition, by instance id | `(object, state)` |
| `f131653` | 【OBJ】OBJステートID比較（インスタンスID指定） | compare an object's state id, by instance id | `(object, state, mode)`; mode 0 tests "equals", 1 "does not equal" [inference from every use in the scripts] |

The state numbers are the object's own. We never saw a table of them. We read them from the scripts that test them
(section "Finding an object's states", below). The ones we met:

| Object (entity id, map) | State | Meaning | How we know |
|---|---|---|---|
| Ashen Idol (50361101-04, m50_36) | 10 | untouched; the load script asks for the intact state | [data] |
| | 33 (idols 1 and 3), 34 (idols 2 and 4) | intact | [data]; written live [live 30 Oct] |
| | 43 / 44 | in use: requested when the player answers YES at the wedge prompt, and the intact state is requested again on NO; found at load, the script sets intact | [data]; the meaning is [inference] |
| | 20, 50, 51, 60, 93 / 94 | the destroyed set (93 for idols 1 and 3, 94 for 2 and 4) | [data]; 20 written live [live 30 Oct] |
| Poison pool (10173000-04, m10_17) | 10 | poison | [live 30 Oct] |
| | 70 | draining: the game's live drain asks for it first | [data] |
| | 20 | drained | [live 30 Oct] |
| Giant windmill (10171500, m10_17) | 20 or 51 | burnt: the load group sets flags 117000055 and 117000056 | [data] |
| Belfry bell gate (10161051, m10_16) | 10 / 70 / 20 | closed / opening / open | [live B19] |
| Throne end objects (20211800, 20211810, m20_21) | 10 / 20 | not usable / usable | [data]: the game's own end-object manager asks for these; meaning [inference] |

### 2. Where the saved table is in memory

- The table holds 8-byte pairs `{u32 entity id, u32 state}`. The four idols sit in four consecutive pairs, so the pairs
  look sorted by id. [live 30 Oct, from the trainer's write log]
- No pointer path to it is known and no exe function was read. The trainer finds it by scanning the one allocation that
  holds `PlayerCtrl` (269 MB in the 30 Oct session, of 1.9 GB readable memory). A hit counts when the word eight bytes
  before or after it, divided by 1,000,000, gives the same number as the id divided by 1,000,000: another object of the
  same map range, which rules out a stray match. [live 30 Oct]
- **There can be two copies.** The Fume Knight's map gave two hits for each idol and Mytha's map one hit for each pool.
  The load reads the copy **nearest the map's event-flag block**; the other copy keeps stale values and is only written
  by us. [live 30 Oct]
- The live state itself was not found as a plain number. The idol's `MapObjStatus` and `MapEntity` objects hold no 33 or
  34. [live 30 Oct]
- The table exists while its map is loaded. The trainer found the slots only for the map it was in, so from another map
  it enters the map first and then reloads it. [live 30 Oct]

### 3. The unload and load window

A reload of the same map makes the map's event-flag blocks (block `(aa / 10) * 10000 + bb * 100` for `m<aa>_<bb>`; see
[Event flags](event-flags-and-free-ids.md)) vanish from the flag manager for about 0.3 s. The state table works on the same
beat. [live 30 Oct]

1. **Unload.** Every live object writes its own state into the table, over whatever you put there.
2. **About 0.35 s later: load.** The game reads the table and builds each object in that state.

The trainer's write log shows this. Three holds, whole-second timestamps (30 Oct, 17:50 to 17:52): [live 30 Oct]

| Run | Warp call | First sweep | Write-back and rewrite | Player placed |
|---|---|---|---|---|
| Idols intact (20 to 33 / 34) | 17:50:09-10 | 17:50:10: both copies of each idol written | 17:50:13: only the copy near the flags, written back to 20 by the unload and rewritten by us | 17:50:17 |
| Idols destroyed (33 / 34 to 20) | 17:50:42 | 17:50:42: both copies | 17:50:45: near copy again | 17:50:49 |
| Mytha, poison (20 to 10) | 17:51:49 | 17:51:49: five pools | 17:51:52: five pools again | 17:51:56 |

The write-back came about three seconds after the warp call, and the player was placed about four seconds after that. The
first sweep is harmless. The rewrite after the write-back is the one that counts. [live 30 Oct]

### 4. Load scripts read the object, then set the flags

**The Ashen Idols.** Brume Tower (m50_36) has **11 idol objects** (model `o36_1250`, entity ids 50361100-50361111 without
50361105), each with a companion part (model `o36_1253`, ids 50368500 and up) whose display and collision the scripts
switch with `1:130702`. [data] Each idol has a load group of the same shape (developer label 煤の像を破壊, "break the
soot statue"). The group calls a preset, which calls a "reproduce" sub-machine. That sub-machine does this: [data]

Before anything else it checks whether the player is a guest (an online summoned phantom; the function is `f130602`,
【ネット】ゲストユーザー判定, "guest user check"). A guest only gets the navmesh attribute added, and the sub-machine
stops. For the host it goes on:

1. If the object's state is 10, request the intact state and wait for it.
2. If it is the in-use state (43 / 44), request the intact state.
3. If the state is 20, the idol's own destroyed state (93 / 94), 50, 51 or 60: **set the idol's flag** (`1:130101`) and
   delete a navmesh attribute (`1:132132`, 【マップ】ナビメッシュ属性削除, "delete navmesh attribute", arguments region
   and 2) over the idol's region.
4. Otherwise (intact): switch the companion (`1:130702`, 【描画】オブジェクトインスタンスID指定で表示・当たりの切り替え,
   "switch display and collision by object instance id") and **add** the navmesh attribute (`1:132131`). It does not
   clear the flag.

So the flag is an output of the object's state. The eleven groups:

| Group | Developer label (gloss) | Idol | Flag | Intact / destroyed (own) / in use / intact after NO | Region |
|---|---|---|---|---|---|
| g43000 | ザコボス① (minor boss 1) | 50361100 | 536000040 | 33 / 93 / 43 / 33 | 4300000 |
| g22000 | ボス部屋左_手前 (boss room, left, near) | 50361101 | 536000041 | 33 / 93 / 43 / 33 | 2200000 |
| g22010 | ボス部屋左_奥 (left, far) | 50361102 | 536000042 | 34 / 94 / 44 / 34 | 2201000 |
| g22020 | ボス部屋右_手前 (right, near) | 50361103 | 536000043 | 33 / 93 / 43 / 33 | 2202000 |
| g22030 | ボス部屋右_奥 (right, far) | 50361104 | 536000044 | 34 / 94 / 44 / 34 | 2203000 |
| g44000 | ザコボス② (minor boss 2) | 50361106 | 536000046 | 34 / 94 / 44 / 34 | 4400000 |
| g45000 | 玉座前 (in front of the throne) | 50361107 | 536000047 | 33 / 93 / 43 / 33 | 4500000 |
| g6000 | ロビー (lobby) | 50361108 | 536000048 | 34 / 94 / 44 / 34 | 600000 |
| g5000 | 孤島 (isolated island) | 50361109 | 536000049 | 33 / 93 / 43 / 33 | 500000 |
| g46000 | アイリー (label not translated) | 50361110 | 536000050 | 34 / 94 / 44 / 34 | 4600000 |
| g39000 | 頂上 (summit) | 50361111 | 536000051 | 30 / 91 / 41 / 31 | 3900000 |

Other groups read the idol flags later. Four groups named 煤の像による回復球 ("recovery orb by the soot statue",
g23000-g23030) take the flags 536000041-44. Nine groups (g38100-g38190) call one sub-machine with each idol's flag; the
four for the boss-room idols are labelled 煤の像の破壊変数加算 ("add to the soot-statue destruction variable"), and
group 38000 is the matching reset. [data] We did not trace what the orbs and the variable do. The 30 Oct write-up says
the idols heal the Fume Knight until a Smelter Wedge (item 53200000) goes into each, and the group labels agree. No test
measured the healing. [inference]

**Mytha's pools.** The group is g42000 (毒水低下, "poison water lowering") in `event_m10_17`. Its preset runs a
"reproduce" sub-machine at load. [data]

- Flag **117000055 on** at load: the reproduce requests state 20 on the five pools (10173000-04) and switches six ground
  materials (`1:132161`, 【マップ】グランドマテリアルを切り替える, "switch the ground material": materials 20, 30, 40, 50, 60
  and 21). The room is drained at once.
- Flag off: the preset waits for the flag live, then plays the drain (state 70 on each pool, then 20 in turn, with the
  ground materials switched between timed waits).
- The windmill group (g31100, 【非同期】巨大風車, "giant windmill") sets 117000055 and 117000056 at every load in which
  windmill 10171500 is in state 20 or 51. A save whose windmill is burnt therefore turns the flags back on at each load.

The **ground material** is a runtime effect and is not saved. The **pool objects'** states are saved. They are two
different things, and B25 ran into the gap (section "Pitfalls").

### 5. What survives: the save

- Object states persist across loads: the idols the player wedged in B26 were still wedged on the next trip. [live B26]
  Like the game's own doors and levers, they are almost certainly part of the character's save. [inference] A game
  restart was in the B26 plan; the result line records only "the next trip".
- Because the pools' state 20 persists, **drained stays drained**. The mod's patch also leaves flags 117000055 / 056 on in
  that save [inference]. Undoing the drain takes both: the flags off, and the pools back to 10. The trainer's poison mode
  did exactly that. [live 30 Oct]
- A save shared with a vanilla install therefore keeps a drained Mytha room (its flags are saved); whether an opened
  Belfry gate stays open is not known. See [Saves and installs](saves-and-installs.md). [inference] [untested]

### Choosing a route

| You want | Use | Works? |
|---|---|---|
| An object to go **forward** the way it normally does (a gate opening) | a request, `1:131636` then wait for `f131653` | Yes: Belfry gate [live B19] |
| A state the game's own load script already produces, from a flag | set the flag it reads, before that script runs | Yes: Mytha drained [live B26] [live B49] |
| An object put **back** (idols intact, pools poison) | a request from a script | **No** [live B25] |
| An object put back, with a memory tool | rewrite the saved table inside the unload-load window | Yes [live 30 Oct] |
| A state the player can set once and keep | give the player the means (Smelter Wedges) | Yes: the idols stayed wedged [live B26] |

### Route 1: make the game's own load script do it (Mytha)

1. Read the object's load group in the map's event script. Find what it tests: here flag 117000055 in the "reproduce".
2. Patch that group's first move so it sets the flags (`1:130101(117000055, 1)`, `1:130101(117000056, 1)`) and then goes on
   to the game's preset. The patch method, and why the first state has to stay empty, are in
   [Event script groups](event-script-groups.md) (`tools/modes.py` `mytha_drain_patch`).
3. Load the map. The pools are drained at every load, including a new character's first one. [live B26 step 4] [live B49]

### Route 2: a forward request from a script

```
1:131636(object, wanted_state)          # request the transition
wait until f131653(object, wanted_state, 0)   # or give up after a few seconds
```

The Belfry gate group does this: if `f131653(10161051, 10, 0)` (closed), request 70 (opening), wait for 20 (open). The
game's own lever uses the same three numbers. The gate opened at the Bastille's next load and stayed open. [live B19]
For the failed B25 requests, nobody logged whether `f131653` ever reported the new state or the wait timed out. See the
open questions.

### Route 3: rewrite the saved table in the window (memory tool)

Needs a tool that writes game memory. The project runs it only with Steam in Offline mode and logs every write.

1. **Know the ids and numbers.** The entity ids come from the MSB parts. The state numbers come from the scripts that test
   the object (below).
2. **Find the slots.** Scan the allocation that holds the player for the little-endian u32 id. Keep a hit when a neighbour
   pair 8 bytes before or after belongs to the same id range (section 2). Remember the address of the state word (hit + 4).
3. **Pick the copy the load will read**: the one nearest the map's event-flag block. If it already holds the wanted
   states, you do not need a hold.
4. **Start the hold before the load.** A thread, every 2 ms: for each slot, read the 8 bytes at (state address - 4); if
   the id is still there and the state differs, write the u32. Each pass also reads the flag manager's block list. When
   the map's block vanishes, note it; when it returns, run 2 s more and stop. Give it a 60 s limit and a stop event.
   Never write a slot whose id has moved: a reload can reuse the memory.
5. **Trigger the load.** In the map already: a bonfire warp to that map's bonfire reloads it (one load). From another map:
   enter it first, then reload (two loads). The trainer also holds through a death's respawn load, but no log line shows
   that path in use. [untested]
6. **Check after the load.** Read the near copy back. The trainer logs "objects ... are not in this mode's states after
   the load (F5 again)" when it differs.

What this gave: the four idols destroyed with 20 (their flags came on [live 30 Oct]; the load script's destroyed branch
sets them [data]), the idols restored with 33 / 34 (the trainer cleared the flags beside them, because the intact branch
does not), and Mytha's pools back to poison with 10 (flags 117000055 / 056 off beside them). Each took one load.
[live 30 Oct]

### Finding an object's states from the scripts

1. Dump the map's event script (`tools/esd.py`) and read the developer names (`tools/edd.py`).
2. Find the group that takes your entity id as an argument (for example g22000's `6:sub(50361101, 536000041, ...)`).
3. Follow the sub-machines. **The text dump hides the called sub-machine's id** (it prints `6:sub(...)`); use `esd.parse()`
   and read the entry commands, which are `(6, id, args)`. [data]
4. Collect every `f131653(obj, N, mode)` test and `1:131636(obj, N)` request below it. The numbers the game tests are the
   numbers the object has. The idol's check lists 20, its own destroyed state, 50, 51 and 60 as destroyed.

## Pitfalls and what failed

| Test | What happened | Lesson |
|---|---|---|
| 30 Oct, first table writes | A write to the table stayed until the unload, then every live object wrote its own state back | Write after the unload, inside the window; hold and re-write |
| 30 Oct, drained to poison | The trainer set the poison flags, but the room stayed drained | The pools' state 20 is saved. Poison also needs the pools at 10 (10 is "untouched": the idol script treats 10 as fresh) |
| 30 Oct, a second copy of the table | One copy was never written back; it kept old values | Choose the copy nearest the flag block, and check it after the load |
| **B25, Fume Knight** (6 Nov) | Preludes in FromSoftware's own load groups g22000-22030 requested the intact or destroyed state and waited up to 3 s. The idols stayed as the save had them, whichever choice | Script requests did not restore the idols [live B25] |
| **B25, Mytha "poison"** (6 Nov) | Poison damage came back (splash sound, poison meter rising) but there was no poison texture | The request 20 to 10 on the pools did not take, so the pool objects stayed drained. The poison **ground material** did come back, because with the flags off the drain never ran. Two separate things: saved object state and runtime ground material [live B25] [inference for the split] |
| **B25, the mid-fight guard** | The prelude only acted while the fight flags 536020080 / 083 were off, to protect an idol wedged during the fight | The idols stand outside the arena, so the guard rested on a wrong premise [live B25 note] |
| **B25, outcome** | Both choices (the Herald's questions for the idols and the pools) were dropped | Few players fight these two with the idols or the poison; a flag patch for Mytha was reliable and the poison direction was not |
| Throne end objects, every castle load | The mod requests state 10 on both objects at each load. No test recorded their state before or after the request, and none ran with the request removed | Do not assume a restore request works; see the open questions |

The failing requests are a fact for two objects only. The idol's own prompt asks for the intact state again after a NO
(43 to 33), and the game's end-object manager asks for 10 on the Throne objects after the ending. [data] Whether the
game's own backward requests take effect was not tested. We do not claim that backward requests fail for every object.

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| Saved table of `{u32 id, u32 state}` beside the flags; untouched is 10 | memory scan and write log | [live 30 Oct] |
| The copy nearest the flag block is the one the load reads; a second copy stays stale | write log: two copies written, one written back | [live 30 Oct] |
| The unload writes live states back, about 3 s after the warp call; the load follows the flag-block drop by ~0.35 s | write log and block polling | [live 30 Oct] |
| Holding 20 through a reload broke all four idols and turned their flags on; 33 / 34 and 20 each worked in one load | trainer, in game | [live 30 Oct] |
| Putting the pools to 10 through a reload brought the poison back | trainer, in game; the note says so | [live 30 Oct], evening note |
| The idol load group decides from the object's state, then sets the flag | event script m50_36, g22000-22030 and the preset sub-machines | [data] |
| 11 idol objects in Brume Tower | MSB of m50_36 | [data] |
| 117000055 on at load drains the pools; the windmill's state 20 or 51 turns both flags on | event script m10_17, g42000 and g31100 | [data] |
| Script requests to restore the idols and the pools did not take | Herald choices, in game | [live B25] |
| A request 10 to 70 opened the Belfry gate and it went on to 20 | in game | [live B19] |
| Mytha drained at every load with the flag patch | in game; again on a clean install and a new character | [live B26 step 4] [live B49] |
| Player-wedged idols stay wedged on the next trip | in game | [live B26] |
| The idols heal the boss until wedged | the 30 Oct write-up and group labels; no measurement | [inference] |
| `f131653` mode 1 means "does not equal" | read from every use in the scripts | [inference] |
| The live object state, and the function that writes back, were not found | memory search; exe not read | [live 30 Oct] |
| A saved 20 on the Throne objects comes back at later loads; the lock resets it | the Throne findings infer the first part; the lock's effect from 20 was never isolated | [inference] [untested] |

## Tools

| Tool | What it does here |
|---|---|
| `tools/arenalib.py` | `obj_slots(ids)` finds the saved-state words; `obj_states(slots)` reads them; `hold_obj_states(slots, wanted, block)` is the 2 ms thread (stops on its event, at 60 s, or 2 s after the block returns) |
| `tools/arenactl.py` | The trainer. A mode's fourth item is its object states. `obj_check` picks the copy nearest the flags and reports whether the table differs; `hold` / `end_hold` run the thread through F5's own warp (one load if you are in the map), through an extra reload, and through the respawn after a death; after every load it warns when the states did not take. The Fume Knight and Mytha modes live in `MODES` |
| `tools/modes.py` | `mytha_drain_patch`: the flag patch that keeps Mytha's room drained |
| `tools/_retired/` copy of `modes.py` (8 Nov) | the failed B25 preludes, kept for reference |
| `tools/offline_tests.py` | a fake-memory test of the hold: it writes the wanted state and stops on its event |
| `tools/esd.py`, `tools/edd.py` | read the load groups and their developer names |

## Open questions

- **Where is the live state, and what writes it back at the unload?** No exe function and no pointer path to the table
  is known. A pointer path would remove the heap scan. [untested]
- **What is the second copy of the table?** It might be the in-memory image of the save data. If so, whether the save
  holds the destroyed or the live states at that moment is unknown. [inference]
- **Why did the backward requests fail?** The request may have been refused, or ignored, or overwritten later. B25 did
  not log whether `f131653` ever reported the new state. A probe group in the pattern of the B56 probe (see
  [Event script groups](event-script-groups.md)) that requests and logs the state would tell. [untested]
- **Does a forward request work from a script on every object?** The pools' drain at load is one (the game's own
  reproduce asks for 20, and a new character's pools drained, B49). One live example is not a rule. [inference]
- **Does the Throne lock work from a saved 20?** Start a save whose Throne objects are at 20 and load the castle with the
  mod. If the throne still offers the ending, the lock does not work and flag-based guards must carry it. [untested]
- **Do saved states survive a game restart?** B26 records "the next trip". Quit to the desktop and look again. [untested]
- **Other objects the same route could set** (levers, breakable walls, other bosses' hazards near fogs): suggested after
  30 Oct, never built. [untested]
- **Do the idols heal the Fume Knight?** Fight him with all four intact and with all four wedged and log his HP.
  [untested]

## Related pages

- [Event script groups](event-script-groups.md): the Mytha drain patch, the empty first state, and the probe group.
- [Script command reference](script-command-reference.md): the `1:131636` and `1:131638` rows.
- [Event flags and free ids](event-flags-and-free-ids.md): the flag blocks that vanish at an unload.
- [Memory map for PC 1.0.3.0](../tools/memory-map-pc.md): the flag manager and the other trainer offsets.
- [Giving items](../modding-menus/giving-items.md): the six Smelter Wedges (lot 60019000).
- [Bonfire lit state](../modding-world/bonfire-lit-state.md): why "touch object" was left untested.
- [Saves and installs](saves-and-installs.md): drained pools stay in a shared save.
- [Bringing a boss back](../modding-world/respawning-bosses.md): the other state a reload keeps or resets.
