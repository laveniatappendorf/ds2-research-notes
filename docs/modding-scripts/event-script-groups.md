# Event script groups: adding, disabling and patching them (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.
> Test ids such as B9 point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags: `[live B9]` seen in the game,
> `[code]` read in the exe, `[data]` read in the game files, `[inference]` reasoned, not proven, `[untested]` an idea only.

## Summary

- **A group runs only if its map's `EventParam_<map>` has a row whose `event_id` is the group's id.** So the three jobs are:
  *add* a group (a state group in the map's event script plus one 16-byte row), *disable* a FromSoftware group (delete
  its row; the script is not touched), *patch* a FromSoftware group (edit its states in the script). [live B9, B53]
- **Keep the first state empty.** The first state (`s0`) only moves on to the next. Our first build put two enemy-removal
  commands in `s0` and they did nothing; the same commands one state later worked. All 2,183 top-level groups in the
  game's 28 PC event scripts start that way. We do not know the mechanism. [live B9; data]
- **Changes load with the map, with no game restart.** Build while the game runs, then load the map again. [live B13, B54]
- **Use ids 990000-990999 for new groups.** No script of the game (event, talk or AI, 538 files) has a group in that block.
  [data]
- **Logic that belongs to one map must wait for the player to be in that map.** A map that is only *streamed in* next to
  the player runs its event scripts too, and its load groups will fire. The wait is `1:140402(0, 1, 0)` then
  `f140001(0)`. [live B39]
- **Between loads, keep state in flags.** Flags in the global blocks cross maps (the boss rush's arrival tokens do).
  A death count stored in a script variable did not compare equal to a later read of the same count, so we dropped
  variables. [live B38, B50c]
- **The main trap when patching a FromSoftware group: races.** Two groups of one map start together at load. Our group set
  a flag that a FromSoftware group reads once; the FromSoftware group looked first. The fix was to patch *its* first move,
  not to set the flag from our own group. [live B39]

### 1. The three parts of a group

| Part | Where | What it is |
|---|---|---|
| The group | `event_<map>.esd` (the map's event script) | A state group: states, conditions, commands. Its id is the group id. See [Writing ESD files](writing-esd-files.md) |
| The row | `EventParam_<map>`, a per-map param (loose file `eventparam_<map>.param`) | 16 bytes. Row id = group id, and the `event_id` field holds the same number |
| The start | The map's load (for a load-time row), another group, or a list pick | Section 2 |

The row (paramdef `EVENT_PARAM`, size 16): [data]

| Field | Offset | What we know |
|---|---|---|
| `event_id` | +0x00, s32 | The group the row starts. We set it equal to the row id every time (see Open questions) |
| `flag_id` | +0x04, s32 | **Not always a flag.** Our load rows use 0. On the Giant Lord memory's rows it holds an object instance id (3010: the statue head, 20101050) and a generator id (111247: 656). Gilligan's ladder rows hold 10044000 [inference: "an id the group is about"] |
| `Unk08`, `Unk09` | +0x08, +0x09 | 0 on all rows we use. Non-zero only on 78 rows of the game (all `Unk0B` 4, `Unk0D` 7, ids 80000 and up) |
| `Unk0A` | +0x0A | 0 on ours. 1 on 174 of the game's 2,383 rows |
| `Unk0B` | +0x0B | **Our "kind".** 1 on our load rows, 3 on our list-pick rows. In the game: 1 on 283 rows, 2 on 2,014, 3 on 8, 4 on 78 |
| `Unk0C`, `Unk0D` | +0x0C, +0x0D | 0 and 6 on our load rows; 1 and 5 on our list-pick rows |
| `Unk0E`, `Unk0F` | +0x0E, +0x0F | 0 |

What we copy. Load rows: the Forest's group 14010 (developer label "salamander ground-flame damage immunity", a group the
game runs at every Forest load): flag 0, `Unk0B` 1, `Unk0C` 0, `Unk0D` 6. [live B9] List-pick rows: Gilligan's ladder row
111095: flag 10044000, `Unk0B` 3, `Unk0C` 1, `Unk0D` 5. [live B36c, B57] See
[Adding dialogue options](../modding-menus/adding-dialogue-options.md). We never changed the meaning of a kind byte and do not know what
they mean [untested].

What the game's own files look like, recounted over the 28 PC event scripts and their `EventParam` files (originals): [data]

- 2,183 top-level groups (ids below the compiler's library range, 2147480000 and up). 2,179 of them are named by some
  row's `event_id`. Four are not (a spare queen's portrait, a wyvern fly-by, an armour-shop object placement and a
  herald erase check). We do not know whether those four ever run.
- 2,383 rows. On 200 of them the row id differs from `event_id`. In 155 of those the `event_id` names a **talk-script**
  group, not an event-script group, so rows also start some talk groups [inference from the names].
  Every row we added or deleted has row id = `event_id`.
- 1,385 rows have a non-zero `flag_id`, 998 have 0.
- All 28 files rebuild byte for byte with `tools/paramrows.py`, which is why it is safe to add or delete rows with it.

### 2. What starts a group

| Starter | Row of the started group | Example | Evidence |
|---|---|---|---|
| **The map's load** (every load, since the row has flag 0) | flag 0, kind 1 / 0 / 6 | Every arena group; the Forest's 14010 | [live B9, B10-B19, B39] |
| **A pick in the N-choice list** (the row's `esd_event_id`) | flag 10044000, kind 3 / 1 / 5 | Gilligan's ladder groups; our 56 list rows | [live B36c, B57] |
| **Another group**, with `1:130003` (【システム】他のイベントを実行, "execute another event") | kind 3 | In the Giant Lord's memory, group 2000 starts 2012 and 2013 | [data] |

- `1:130003` is used twice in all the event scripts, so it is a rare route. [data]
- Of the game's 8 kind-3 rows, five are known to be started by something else (the three ladder builders by a list pick,
  the two fireball launchers by group 2000). The other three (two duel starters, a prisoner's song) have names that read
  as "on request". So kind 3 looks like "start on request", and a kind-3 row probably does not start at load.
  [inference] We never checked whether our own kind-3 groups also run at load; their first real state tests a flag that
  only the list sets, so it did not matter.
- **A map that is streamed in runs its event scripts.** While the player was in Najka's map, the Royal Rat Authority's map
  next door loaded in the background and ran its groups. [live B39]
- A group has no memory between loads. At each load it starts again at `s0`. [live B9, B10-B19: every load of an arena
  map ran the removals again]

### 3. The first state holds no commands

FromSoftware's groups all start with a state that only moves on:

- 2,183 of 2,183 top-level groups: the first state (id 0) has **no** entry, exit or while commands. [data, recount 17 Nov]
- 2,180 of 2,183 have exactly one condition on it, the always-true `1`. The other three have one real condition.
  None has commands on that condition. [data]
- Where that one condition goes: to state 2 in 1,538 groups, to state 1 in 385, to state 3 in 246, to state 4 in 9 and to
  state 6 in 2. [data]

Our first new group, B9, followed the same shape but put the removals (`1:131731(2830, 1)`, pause, and
`1:131721(2820, 0)`, delete) in `s0`'s entry. The group ran, because its win flow worked from `s1` onwards, but both
enemies were still there. Moving the removals to `s1` removed both on arrival. The same two commands had worked when
added to state 2 of the game's own group 14010 (B7, B8). [live B7, B9]

What we **do not know** is why. Either the entry of a first state does not run, or it runs before the enemy generators
exist. One observation cannot tell them apart. [inference] The rule stays: nothing in `s0`.
`tools/choices.py` `spec_group()` refuses a group whose first state has commands.

### 4. A group's life: states, conditions, time

- A group starts at `s0` on each load of its map. A state with no conditions is a dead end: the group idles there until
  the map unloads. We use that for "done" and "stale, wait for the next load" states.
- A state's conditions are tried in order and the first true one is taken. The boss rush's arrival-token branches are put
  *ahead* of the other conditions in `s1` for that reason. [design; live B38]
- `f3()` is the time in the *current* state, in seconds, not since the group started. The arena group's clear step
  waits `f3() > 7` in a state it entered 1 s after the kill, so the warp comes about 8 s after the kill flag. [data,
  `tools/arenamod.py`; live B10: kill 17:07:37, flag cleared 17:07:38, Majula 17:07:48 including the load]
- Commands go in a state's entry list. We never used exit or while lists.
- A top-level group returns nothing. It reaches other groups only through flags, through `1:130003`, or by calling a
  library group (bank 6, which does return a value).

**The arena group**, the worked example (one per arena map, id 990000; the win flow itself is in
[Bringing a boss back](../modding-world/respawning-bosses.md) and [Warping by script](../modding-world/warping-by-script.md)):

| State | Entry commands | Moves on |
|---|---|---|
| `s0` | none | to `s1` always |
| `s1` | the map's local flags, then the enemy removals (`1:131721(generator, 0)`) | to `s2` if any of the map's kill flags is already on (stale), else to `s3` |
| `s2` | clear those kill flags | to `s3` always |
| `s3` | none | one condition per boss: its kill flag on, to `s4` |
| `s4` | none | to `s5` after 1 s (the souls come in the flag's own tick) |
| `s5` | the kill flags off (so leaving early leaves nothing stale) | to `s6` after 7 s |
| `s6` | `1:132125(0, 0, map, location, 0)`, the warp to Majula | nowhere |

Flags are set before the removals in `s1` on purpose: an enemy that is gated on a map-local flag may not exist yet when
its removal runs. [design, from the NG+ notes; the order was not isolated in a live test] The group as a whole is
[live B9, B10-B19, B39].

The shipped group has more states than this: chains for the boss rush's next warp, the shuffled rush, the arrival token's
reload and the after-a-win option. They add conditions to `s1` and `s3` and extra states after them; the docstring of
`tools/arenamod.py` `arena_group()` describes each. Without those options it builds exactly the table above.

### 5. When changes apply

Event scripts and per-map params are read when the map loads. We built and installed with the game running and the next
load of the map picked the files up: B13 (21 arena rows built while the player stood in Majula) and B54 (scripts and
per-map params, installed with the game running). [live B13, B54] A map that is already loaded keeps running the old
groups until it loads again. [inference; B56's notes say the same] Texts are different: they are read at game start
[live B12, for the bonfire names], and the game-wide ("rules") params probably are too [inference]. See
[Editing params](editing-params.md).

### 6. Group ids

The block **990000-990999** is ours. No group of any of the game's 538 scripts (28 event, 25 talk, 485 AI) lies in it,
and the highest event-script group is 4031010. [data, recount 17 Nov] `tools/ids.py` fails a build if a new group is
outside the block. The block is script-wide, so the same job gets the same id in every map (990000 is the arena group in
all 22 arena maps). The groups the shipped mod adds:

| Id | Where | Job | Test |
|---|---|---|---|
| 990000 | every arena map (22) | the win flow above; the Giant Lord's and the Darklurker's maps have their own versions | B9, B10-B19, B39 |
| 990001 | the castle's map (m20_21) | the Throne of Want's choices | B16, B50 |
| 990002 | the Lost Bastille (m10_16) | opens the bell gate at load, using the game's own lever commands | B19 |
| 990003 | every arena map (22) | the boss rush's run guard (arrival token) | B38, B39 |
| 990004 | every arena map and Majula | the shuffled rush's picker | B52 |
| 990005 | the Burnt Ivory King's map (m50_37) | the King's fight by choice (knights, portals, ally) | B24 |
| 990006 | every arena map (22) | restore / repair / hard mode at arrival | B57 |
| 990012 | Majula | the shop NPCs' flags, set again at every load | B42 |
| 990013 | Majula | the boss rush's state check | B38, B39 |
| 990200-990204, 990300-990341, 990500-990508 | Majula | one **list-pick** group per row of the N-choice lists | B36c, B50, B57 |
| 990210, 990510 | Majula | load groups that clear the list flags / derive the option flags | B36c, B57 |

Retired and not shipped: 990007 (death markers, B50c), 990010 (the setup, moved into the Herald's Talk, B20d), 990011 and
990014 (the Darklurker's warp and the retry, B19, B50; gone since B54, B55), 990020 (back to human on every load, B21,
switched off) and 990056 (the throwaway probe, B56).

### 7. Waiting for the player to be in the map

A load group starts when its map is loaded, not when the player arrives. In the boss rush, a neighbouring map's guard
group saw a run that was still on and no arrival token (the token had been cleared in the map the player was really in).
So it ended the run and sent the player home (B39, second run, right after Najka's kill). The fix is the wait the game's
own load groups use, copied byte for byte from one of them (Majula's group 111416): [live B39 runs 3-5]

| State | Entry | Moves on |
|---|---|---|
| `load` | `1:140402(0, 1, 0)` (【グループ条件：マップ】プレイヤーがマップ内にいる判定, "group condition: the player is in the map") | `f140001(0)` (【グループ条件】グループ条件を満たす, "the group condition is met") to `go`; `f3() > 3` to your stand-down state |
| `go` | the logic that must run only in this map | |

- The game's 247 uses of `1:140402` are in 242 groups (40 of them top-level, the rest in library groups). The argument
  forms are `(0, 1, 0)` 178 times, `(1, 0, 0)` 34, `(8, 1, 0)` 24 and `(0, 0, 0)` 11. We use the first. [data, recount 17
  Nov] Reading: the first argument is a condition slot and the second says "inside" (1) or not (0). [inference]
- The mod's groups that wait this way: the run guard, each arena group's token and stale reloads (wait, clear, 1 s,
  reload), Majula's rush check (Majula streams in near the Forest, Heide and the Shaded Woods), the arrival restore /
  repair group (990006, B57) and the B56 probe. [live B39, B57, B56]
- Give the wait a time limit (3 s in ours) and a safe exit. A neighbour that never becomes "the player's map" then does
  nothing. [design; live B39]

### 8. State across loads and maps

- **Flags in the global blocks** survive a load and are the same in every map. The rush's flags (run on, the arrival token
  and the "this is the reload" marker, 104700-104707) are set by one map or by Majula and read by the next. [live B38]
  Map-local flags can be read or written only while their map is loaded (their values come back with the map) [live A1;
  see [Event flags and free ids](event-flags-and-free-ids.md)], so the mod sets them in that map's own load group. The Wharf's ship flag 118000010 was the first: B13 passed with it,
  but B39 later showed that such a set can lose a race with a FromSoftware group that reads the flag (section C).
  [live B13, B39] More: [Event flags and free ids](event-flags-and-free-ids.md).
- **Script global variables** (`1:130122` set, `f130122` get): we stored the player's death count in variable 236 and later
  compared it with the live count `f130308(0)`. The comparison fired with no death, in Majula (B50) and again when moved
  inside the same map load (B50b). At B50c the fault was found to lie in the stored-count comparison itself ("does not
  compare equal to a later read, even in the same load"), so the first guess, that variables do not survive a map load,
  was **withdrawn**. [live B50, B50b, B50c] Whether variables survive a map load was never tested on its own. We use
  flags. See [Detecting a player death in a script](detecting-death-in-scripts.md).

### A. Add a new group

1. **Choose the id.** One in 990000-990999 that the map's script and `EventParam` do not use. The writer refuses a
   script that already has the group; `paramrows.py` refuses an existing row.
2. **Plan the states.** First state `s0` empty, moving on always. Do your work from the second state. Add the player-in-map
   wait (section 7) if the group touches the player, the warp or the run's state.
3. **Write the group.** `tools/choices.py` `spec_group(old, gid, spec)` builds it from a list of
   `(state name, commands, [(target state, evaluator text)])` and refuses commands in the first state. It also refuses a
   file that does not write back to the same bytes, and returns `esd.py`'s diff so you can see that only your group
   changed. A small example (flag numbers are placeholders; use your own free flags):

   ```python
   spec = [("s0",    [],                              [("load", "1")]),
           ("load",  [(1, 130101, FLAG_BUSY, 0)],     [("wait", f"f6({FLAG_ASK}) != 0")]),
           ("wait",  [],                              [("work", "f3() > 2")]),
           ("work",  [(1, 130101, FLAG_DONE, 1)],     [])]        # a state with no conditions: stays here
   new, diff = choices.spec_group(old_script_bytes, 990100, spec)
   ```
   A command is `(bank, id, arguments...)`; bank 1 is a plain command (`1:130101` = 【フラグ】フラグを設定, "set flag").
   Conditions are the text of the game's expressions: `f6(flag) != 0`, `f3() > 2`, `1`. Details: [Writing ESD files](writing-esd-files.md).
4. **Add the row.** Copy a load row and set the fields. With the tool:

   ```
   uv run --no-project --python 3.12 tools/paramrows.py add eventparam_m10_10_00_00 990100 14010 \
       event_id=990100 flag_id=0 Unk0B=1 Unk0C=0 Unk0D=6
   ```
   `tools/arenamod.py` `event_row(param name, gid)` does the same with the load row's bytes. The tool edits the loose
   param of the game copy it is pointed at, keeps the original as `.orig`, writes the rows in id order and reads the
   result back through the paramdef. [live B9]
5. **Install** the script and the param as loose files where your install reads them ([Editing params](editing-params.md)).
6. **Load the map again** (travel to it, die, or rest at a bonfire there). No game restart. [live B13, B54]
7. **Check.** Watch a flag your group sets, or an effect you can see. If nothing happens: is the row there, is
   `event_id` the group id, is the map you loaded the one you edited, and is anything in `s0`?

Two other ways to start a group: give a **list row** its `esd_event_id` and a kind-3 row (see
[Adding dialogue options](../modding-menus/adding-dialogue-options.md)), or call it from another group with `1:130003`. [data]

### B. Disable a FromSoftware group

1. **Find it.** Dump the map's event script with `tools/esd.py` and name the groups with `tools/edd.py` (the developer
   labels are Japanese). Follow who starts whom: `1:130003` and calls to library groups.
2. **Trace it.** What starts it, what stops it, which flags and objects it touches, and what other groups wait for its
   result. On the Giant Lord's memory this showed that group 2000 starts 2012 and 2013 (all three had to go) and that
   the head group 3000 leaves the head and three ballistas as placed when removed.
3. **Decide what must stay.** The boss's own group, his summon and the bullet-immunity group were left alone. [live B53]
4. **Delete the rows**, with the group id and nothing else:

   ```
   uv run --no-project --python 3.12 tools/paramrows.py delete eventparam_m20_10_00_00 3000
   ```
5. **Install that one file** and load the map again. The script is unchanged and still holds the group. [live B53]
6. **Test the effect, and the things next to it.**

What we have disabled this way (all by `tools/rollout_table.py` `DISABLE_GROUPS`, applied in `tools/buildmod.py`):

| Map | Group (developer label, gloss) | What it did | Result |
|---|---|---|---|
| m20_10 ([the Giant Lord's memory](../mechanics/giant-memory-fireballs-pc.md)) | 1000-1070 にぎやかし火球発射_n, "ambient fireballs" (8 rows) | A fireball launcher every 8-10 s | **Gone, live B53** |
| m20_10 | 2000 前線火球_発射管理, "front-line fireball manager", with 2012 / 2013 (normal and boss launchers) | Volleys while the player stands in region 200000 and during the boss fight | **Gone, live B53** |
| m20_10 | 3000 火球発射_石像頭ゴロゴロ用, "fireball for the rolling statue head", and 3010 (the navmesh change after it falls) | The statue head: a fireball, the scripted fall 5.6 s later, a 9999 x10 hit | **Gone, live B53** |
| m20_10 | 12000 巨人の記憶の時間制限_王の扉後, "the memory's time limit, after the King's door" | "The ashen mist fades" and a warp out at 300 s | Row deleted since 5 Nov; staying past 5 minutes was an optional step of B53 and no result is recorded [untested] |
| m10_33, m10_34 (the Rat bosses' maps) | 118220 マルチ用NPC：魔導士（男）：黒ファントム出現：オフライン, "NPC: sorcerer (male): black phantom appears: offline" | Spawns an offline invader (`1:131741`) when the player enters region 10000000; in NG+ it comes back up to 12 times | Row deleted 5 Nov; both maps load and fight normally; no invader seen is not recorded [untested] |
| m10_16 (the Lost Bastille) | 111284 NPC：鐘守：黒ファントム出現：オフライン, "NPC: bell keeper: black phantom appears: offline" | The Belfry's offline invader | Row deleted for B19 (PASS); the absence of the invader is not recorded [untested] |
| m10_17 (Earthen Peak) | 4002000 【DC】放浪者A_ランダム抽選と生成, "Forlorn A: random draw and spawn" | A 15 % roll per load for a Forlorn invader, 21 m from Mytha's arrival | Row deleted 5 Nov; the check "no Forlorn on the way to the fog" is not recorded [untested] |

B53 is the live proof: 14 rows deleted from one param file, fireballs and head gone, the boss, summon and immunity groups
kept, and the fight itself unchanged. [live B53] Why the invaders go by group: they are NPC spirits made by `1:131741`
(【ジェネレータ】NPC霊体生成, "create NPC spirit from generator") and removed by `1:131742`, while `1:131721` deletes a
generator's enemy, and the spawn happens when the player enters a trigger box, so a timed delete would not be reliable.
[data; design, from the NG+ notes] See [Removing enemies](../modding-world/removing-enemies.md).

### C. Patch a FromSoftware group

Use this when a FromSoftware group *reads something once at load* and you need that something to be set first. The shape
that fits is the most common one in the game, `s0 -> s2 if 1`. [data: 1,538 of 2,183 groups]

1. **Find the group and check its shape.** Its first state must have one always-true condition and no commands, and the
   state it moves to must be what you expect. Both our patches refuse to run if not (`g42000 s0 is not the expected
   '-> 2 if 1'`).
2. **Add a new state** (the writer gives it the next free id) holding your commands, with one condition `-> s2 if 1`.
3. **Re-target `s0`** to the new state with a **new** condition object: `s0.conds = [ew.cond(new, "1")]`. Do not change
   the old condition in place: the file shares one condition between the states of a group that would otherwise hold
   identical ones, so an in-place change could alter another state too. [data: the writer notes count 1,160 shared
   conditions in the event scripts; our code comments say "the old one may be shared"]
4. **Write, then check.** The new script must write out again byte for byte, and `esd.py`'s diff must touch only that
   group (the tools stop otherwise).
5. **Install and load the map.**

The two patches the mod ships:

| Patch | Group | Why | What the new state does | Result |
|---|---|---|---|---|
| `tools/modes.py` `mytha_drain_patch` | m10_17 g42000 毒水低下, "poison water lowering" | Its drain preset drains the five pools at load only if flag 117000055 is already on; otherwise it waits for it live. A burnt windmill turns the flags on, so a save without it left the pools full | Sets 117000055 and 117000056, then goes on to the preset (state 2) | **PASS, live B26 step 4** (drained at every load); B49 on a clean install, from a note |
| `tools/specials.py` `wharf_ship_patch` | m10_18 g1000 鐘による船移動, "ship moved by the bell" | The group looks at the ship flag 118000010 once at load (library 2147483563, then 2147483566 state 1): on = the ship docked, deck shown; off = deck hidden, wait for the bell. Our arena group set the same flag at load, **but the look came first**: a rush arrival landed in the sea | Sets 118000010, then goes on to the game's ship preset | **Fixed, live B39** (the ship was docked at the arrivals of runs 2-5) |

The cause of the Wharf failure is [inference]: the order in which groups start was never read, and the first run's save
may simply have had the flag off. What is [live] is that the arrival failed once and then worked with the patch.

**The cheaper alternative, adding commands to a state of an existing group.** `tools/datapatch.py addcmd <script> <group>
<state> <id>:<arg>,...` appended the two removals to state 2 of the Forest's group 14010 and both enemies were gone.
[live B7, B8] It needs no new row and no new group, so it was the first thing we proved. We moved to our own groups
because only the Forest had a load-time group to borrow; every other arena map needed a group and a row of its own.
[from the 4 Nov notes]

### D. A probe group you can drive from outside (throwaway)

B56 needed to run script commands on demand in a live game. One group (id 990056, in Majula and the Forest) waited for the
player in the map, then waited for step flags; when a flag came on, the group cleared it, ran that step's command once
and waited again. A tool set the flag and waited for it to go back to 0. Every step ran within 0.05 s. [live B56]
`tools/probe56.py` is a reusable shape for any test of a script command. It was removed from the build after the test.
See [Script command reference](script-command-reference.md).

## Pitfalls and what failed

| Test | What happened | Lesson |
|---|---|---|
| **B9, first build** (4 Nov) | A new group with its own row ran (win flow, warp), but the removals in the first state did nothing: both enemies were there | Nothing in `s0`. Rebuilt with the removals in `s1`: both gone |
| **B39, Wharf ship** (7 Nov) | A rush arrival at Flexile Sentry landed in the sea, ship not docked | Our flag set and a FromSoftware group that reads the flag once started together, and its look came first. Patch that group's first move (section C) |
| **B39, streamed-in map** (7 Nov) | The run ended right after Najka's kill: the neighbouring map, streaming in, ran its guard and found "run on, no token" | Per-map logic waits for the player in the map (section 7) |
| **B50, B50b** (8 Nov) | A retry that compared a stored variable with the live death count looped, in Majula and then inside the arena | The fault was the comparison itself, not the Feather or the map load (B50c's correction). Use flags, and `f130312() < 1` for a death. See [Detecting a player death in a script](detecting-death-in-scripts.md) |
| **B25** (6 Nov) | Preludes added to FromSoftware's own load groups: object-state requests (the Fume Knight's idols, Mytha's poison pools) did not take; a *flag* prelude worked (the drain) | A patch can set flags and start the game's own presets. An object-state request from a prelude did not take effect. Both choices were dropped. See [Map object states](map-object-states.md) |
| **Editing a shared condition** (writer notes) | A condition can be shared between states of one group, so a change in place can reach another state | Build a new condition for a changed target |
| **Cutting a group that others start** (B53 notes) | 2012 and 2013 are started by 2000 and carry a different kind byte (3, not 2) | The notes deleted all three to be safe. Whether deleting 2000 alone would have stopped them was not tested [untested] |
| **B19, a respawn point set by the group** | The Darklurker's first group set a retry point inside the boss room: a softlock until a win | Not a group fault: the respawn rule. See [The respawn point](../modding-world/respawn-point.md) |

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| A new group with its own row runs (first in the Forest, then in all 22 arena maps) | live | B9, B10-B19, B38, B39 |
| Deleting a row stops its group; the script keeps the group | live | B53 (14 rows, fireballs and head gone) |
| Commands in the first state did not take; the same ones in `s1` did | live | B9 |
| The same commands appended to state 2 of FromSoftware's group 14010 worked | live | B7, B8 |
| All 2,183 top-level groups have an empty first state; 2,180 one always-true condition | data (recount) | 28 event scripts, 17 Nov |
| 2,179 of 2,183 groups are named by a row's `event_id`; 4 are not | data (recount) | scripts and EventParam, 17 Nov |
| 990000-990999 has no group in any of the game's 538 scripts | data (recount) | 17 Nov |
| Scripts and per-map params are read at the next load of the map | live | B13, B54 |
| A streamed-in neighbour map runs its event scripts | live | B39 (m10_33 after Najka) |
| The player-in-map wait fixed the early end | live | B39 runs 3-5 |
| Global flags cross maps (arrival tokens) | live | B38 |
| A death count stored in variable 236 did not compare equal to a later read of the count, even in one load | live | B50c (correction) |
| Patching g42000's first move drains Mytha at every load | live | B26 |
| Patching g1000's first move gave a docked ship at the arrivals of runs 2-5 | live | B39 |
| Why the Wharf arrival failed first (a start-order race) | inference | B39 |
| Our two row kinds both start groups (load; list pick) | live | B9; B36c, B57 |
| Deleting the Rat, Bell Keeper and Forlorn spawn groups removes the invaders | built, not observed | rollout notes, 5 Nov |
| Deleting 12000 removes the 5-minute limit | built, not observed | B53 optional step |
| The mechanism behind the empty first state | not known | B9 shows only the effect |
| Rows with a different `event_id` start talk groups | inference | names in the data |

## Tools

All in the project's `tools/` folder.

| Tool | Use for this topic |
|---|---|
| `tools/arenamod.py` | `arena_group()` builds the standard per-map group; `event_row()` adds its row; holds `GROUP`, the row bytes and the player-in-map wait (`IN_MAP`, `IN_MAP_OK`, `IN_MAP_WAIT`) |
| `tools/choices.py` | `spec_group(old, gid, spec)`: a group from a list of states, with the empty-first-state check |
| `tools/esdwrite.py` | Read, edit and write ESD scripts; the file is written back byte for byte before it is trusted ([Writing ESD files](writing-esd-files.md)) |
| `tools/paramrows.py` | `add`, `delete` and `check` for rows of per-map params; keeps `.orig`; reads the result back |
| `tools/specials.py`, `tools/modes.py` | `wharf_ship_patch`, `mytha_drain_patch`: the two first-move patches; `belfry_gate_spec`, the added gate group |
| `tools/rollout_table.py`, `tools/buildmod.py` | `DISABLE_GROUPS` and the build step that deletes the rows |
| `tools/datapatch.py` | `addcmd`: append commands to a state of an existing group (B7) |
| `tools/probe56.py` | The flag-driven probe group (B56), off by default |
| `tools/ids.py` | The id register; fails a build for a group outside 990000-990999 |
| `tools/esd.py`, `tools/edd.py` | Dump a script; the developer names of commands, functions and groups |

## Open questions

- **Why is the first state's work lost?** Put one flag-set in `s0` of a throwaway group and read the flag: if it comes on,
  the removals ran before their generators existed; if it does not, the first entry is skipped. [untested]
- **Do commands on `s0`'s condition run?** (Pass commands: 0 of 2,183 vanilla groups use them.) [untested]
- **What do the kind bytes and `flag_id` do?** Does a non-zero `flag_id` make a load group run once, as a "done" flag? We
  only ever used 0 for load groups. [untested]
- **Does the game match on the row id or on `event_id`?** We never made them differ. [untested]
- **Does a kind-3 group also start at load?** [untested]
- **In which order do the groups of one map start at load?** The Wharf race suggests the game's groups come first.
  [untested]
- **Does disabling group 12000 really remove the 5-minute limit?** And are the three invaders and the Forlorn gone?
  Both are single sessions of waiting or walking. [untested]
- **Do script global variables survive a map load?** [untested; see section 8]
- **Is there an engine route that starts a group with no row** (a bank-6 call, `1:130003`)? [untested]

## Related pages

- [Writing ESD files](writing-esd-files.md): the writer and the file format behind the states
- [Script command reference](script-command-reference.md): `1:130101`, `1:131721`, `1:132125`, `1:140402`, `1:130003` and the rest
- [Event flags and free ids](event-flags-and-free-ids.md): flags that cross maps, free ids, the id register
- [Bringing a boss back](../modding-world/respawning-bosses.md): the arena group's kill flag, stale flags and warp
- [Warping by script](../modding-world/warping-by-script.md): `1:132125` and the player-in-map wait
- [Removing enemies](../modding-world/removing-enemies.md): generator deletion in a load group
- [Editing params](editing-params.md): loose params, adding and deleting rows
- [Map object states](map-object-states.md): what object-state requests did not do
- [Detecting a player death in a script](detecting-death-in-scripts.md): why variables were dropped
- [Adding dialogue options](../modding-menus/adding-dialogue-options.md) and [Asking the player a question](../modding-menus/yes-no-windows.md): the pick groups and their rows
