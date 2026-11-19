# Quieting and removing enemies near an arena (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

Tags in square brackets say how we know a claim: `[live B7]` = seen in the game (the id points to the project's test
log, see [Home](../how-to-read.md#the-modding-pages)), `[code]` = read in the exe or in our build tools, `[data]` = read in the game files,
`[inference]` = reasoned, not proven, `[untested]` = an idea only.

## Summary

- **You cannot make an ordinary enemy passive by editing its `GeneratorParam` row.** We gave a generator its own
  `ActivationEventID1` flag: the game loaded the edit and the enemy still attacked. [live B7]
- **What works is an event-script command, run at every map load:** `1:131721(generator, 0)` (force-delete) removes the
  generator's enemy. The enemy is gone and no character object exists for it. `1:131731(generator, 1)` (pause) gave
  the same result in its one test. [live B7] [live B9]
- **Enemies that share an `AggroGroup` wake together.** Removing one enemy of a group is not enough: the others come
  and the player is attacked anyway. The mod removes the whole group. [live B7]
- The mod removes enemies near each arena's arrival point by a rule (20 m across, 8 m up or down, not the fight's own
  spawners, not NPCs, not script-named ones, plus the aggro group), then adds **hand overrides** for what the rule
  misses. In `rollout.json` (regenerated 10 Nov) that is **122 generators across 16 maps**. [data] [code]
- The traps: the rule skips NPC-type enemies and script-named generators, so phantoms, invader NPCs and Dragon Knights
  needed hand overrides [live B12] [live B18]; fight spawners must **not** be removed [live B13]; the
  first state of a script group ignores commands [live B9]; whole script groups (invaders, fireballs) are switched off
  by deleting their `EventParam` row [live B53].
- An unlimited spawner (see [Bringing a boss back](respawning-bosses.md)) ignores kill flags, so **a boss or ally you do
  not want must be deleted by script too**: the Throne of Want's other stages, Velstadt, the Burnt Ivory King's ally
  797. [live B16] [live B24]

### The commands

All are bank-1 event-script commands. Developer names are from the `.edd` file beside each `.esd`. [data]

| Command | Developer name | Gloss | What we saw |
|---|---|---|---|
| `1:131721(gen, 0)` | 【ジェネレータ】敵の強制削除 | force-delete the generator's enemy | enemy gone, no live character; used for every removal in the mod [live B7] [live B9] |
| `1:131731(gen, 1)` | 【ジェネレータ】指定の敵の一時停止 | pause the specified enemy | enemy gone, no live character: it is held **out of the world**, not frozen in place [live B7]. The mod never resumes one; in FromSoftware's Guardian Dragon script the group form pauses with 1 in one state and a later state calls it again with 0 [data] |
| `1:131722(group, 0)` | 【ジェネレータ】ジェネレータグループ指定で敵の強制削除 | the same, for a generator group | never used by the mod [untested]. FromSoftware uses it heavily (48 calls in Majula's script, 70 in all 28 event scripts the project reads loose) [data] |
| `1:131732(group, 1)` | 【ジェネレータ】ジェネレータグループ指定で敵の一時停止 | the same pause, for a group | never used by the mod [untested] |
| `1:131741(gen, ...)` | 【ジェネレータ】NPC霊体生成 | create an NPC spirit (sign or invader) from a generator | how invaders and summon signs are made; not ours [data] |
| `1:131742(gen, ...)` | 【ジェネレータ】NPC霊体削除 | delete an NPC spirit | the command made for removing a spirit; designed, never used [untested] |
| `1:130101(flag, value)` | 【フラグ】フラグを設定 | set an event flag | used before the removals, see below |

FromSoftware's own scripts call `1:131721` 13 times and `1:131731` / `131732` 6 times in the 28 event scripts the
project reads, so these are normal game commands, not debug leftovers. [data]

### Why the param route failed

The first idea (4 Nov, before any test) was a data-only switch: boss generators carry their fight flag in
`ActivationEventID1`, and the Giant Lord stood still until his fight flag was set, so we expected "give every
ordinary generator a flag that is never set" to freeze them.

B7 tested it on the two enemies about 17 m and 38 m back from the Last Giant's fog (generators 2830 and 2820, both in
`AggroGroup` 71). [live B7]

| Step | What we did | What happened |
|---|---|---|
| 1 | 2830 given `ActivationEventID1` = a flag of the Forest's fight block that nothing else uses | it walked about without aggro; then the other enemy saw the player and **both attacked** |
| 2 | 2820 given the same flag | both attacked, three hits 11 s after the landing |
| 3 | both flags removed again; `1:131731(2830, 1)` and `1:131721(2820, 0)` put in the Forest's load-time group | **both gone entirely**; the next enemy up the lift behaved normally |

- The edited rows were really in memory (only the new rows were found, none of the originals), so the loose param was
  read and the field simply is not an AI switch for ordinary enemies. [live B7]
- The Giant Lord's stillness comes from his own boss script: it wakes him within 30 m, or when he is hurt. [live B19]
  [data]
- Enemies that share an `AggroGroup` (+0x03 of the row; 0 and 255 mean "none") are linked: one noticing the player
  wakes the rest. [live B7] See [GeneratorParam fields](generatorparam-fields.md).

### Where the commands go

Put them in a script group that runs at every load of the map, and not in its first state.

- A script group only runs if `EventParam_<map>` has a row whose `event_id` is the group id. The mod gives every arena
  map its own group (id 990000) with its own row, so it never edits a FromSoftware group. [live B9] See
  [Event script groups](../modding-scripts/event-script-groups.md).
- **Commands in a group's first state did not take.** The first build had the removals in the first state: the win
  flow ran, but both enemies were still there. FromSoftware's groups never put commands there (the first state only
  moves on), so ours do the same and the removals sit in the second state. [live B9]
- The group runs at every load of the map (its `EventParam` row has flag 0, "no done flag"), so the removal repeats
  each time and nothing is stored in the save. [code]
- Changes to a map's script and params are read at each load of that map, so no restart is needed; builds were run with
  the game up while the player stood in another map. Menu text is the exception: it is read only at game start.
  [live B13] [live B54] [live B12]
- **Set the map-local flags before the removals.** An enemy gated on a flag may not exist yet when its removal runs.
  `tools/arenamod.py` writes the flags (`1:130101`) first, then the removals. Built 5 Nov for the Wharf's NG+ phantom
  4550, which needs the ship flag. [code] The Wharf row passed in NG afterwards [live B15], but 4550 itself exists
  only in NG+, so the ordering was never seen to matter. [untested]
- A map that is only streaming in next door runs its own event scripts too, so its removals run when it streams
  in. [live B39]
- Removing a generator that does not exist in the current cycle does nothing, harmlessly: the Smelter Demon, Mytha and
  the Chariot rows carry NG+ generators in their lists and passed in NG. [live B11] [live B12]

### The rule the mod uses

`tools/rollout_table.py` decides, per boss row, which generators of the boss's map to remove. The point of reference
is the **arrival point**: 1.1 m behind the moved bonfire (see [Moving a bonfire](moving-a-bonfire.md)), or the saved spot for
rows whose bonfire stays. [code]

A generator is removed if it passes the **near** test and none of the **never** tests:

| Test | Value |
|---|---|
| near: horizontal distance to the arrival point | under 20 m (`QUIET_R`) |
| near: height difference | under 8 m (`QUIET_DY`) |
| never: one of the boss's fight generators, or a generator of any boss in the same map | kept (see [Bringing a boss back](respawning-bosses.md) for how fights are found) |
| never: a generator the map's script names | kept, see below |
| never: `Unk84` not 0 (NPC spirits: white sign, shade sign, invader) or `SpawnLimit` 255 (mostly player-summon rows, but 37 of the 105 such rows are plain enemies, so the rule keeps those too) | kept |
| never: the `EnemyParam` row is an NPC type (`npc_player_status_param_id` not -1 or 0) | kept |

Then the set is **closed over aggro groups**: every other generator in the map that shares an `AggroGroup` with a
removed one is removed too, unless it is a fight generator or a spirit or summon. [code] [live B7]

- Positions come from `GeneratorLocation_<map>` (same row id as the generator). They are map-local: add the map's
  origin offset, the `マップオフセット` ("map offset") entry in the MSB's event list, to get world coordinates. [data]
- The enemy kind comes from `GeneratorRegistParam` -> `EnemyParamID`. [data]
- "Named by the map's script" is a coarse text test: any number in any line of the map's event script that equals a
  generator id counts as a name. So it can hold back a generator that merely shares a number with a flag or a timer.
  [code]
- Why the arrival point and not the route: with the respawn at the fog, enemies along the way from the old bonfire
  never matter. A 4 Nov count of non-boss generators within 30 m of each fog spot found 0 to 6 for most; the busiest
  were Mytha 18, Ruin Sentinels 17, Congregation 17, Skeleton Lords 14, Freja 12. Many of those are through walls or on
  other floors. [data]
- The result is the `quiet` list of each row in `rollout.json`; the build turns it into one `1:131721` per generator in
  the arena group. [code] The per-boss lists are not published here; this page keeps the
  hand overrides.

### What the rule misses: the hand overrides

Tested one fight at a time, the rule left some enemies that did attack. Each was added by hand (`FORCE_QUIET`) after
the author reported it or after a check. The generator ids are in the table.

| Fight | Added by hand | What they were, why the rule missed them | Test |
|---|---|---|---|
| Executioner's Chariot | 8000, 8001 | red phantom by the fog; EnemyParam 852000, model id 1: an NPC type, so the NPC rule skipped it | built from a 4 Nov note; row passed [live B12] |
| The Rotten | 592, 593 | NPC invaders (`Unk84` 2), 22-25 m out | row passed [live B12] |
| Looking Glass Knight | 3020, 3030, 3040, then 3010 | horse knights with twinblades, 11-26 m, each named only by its own one-line setup group; **3010** (same aggro group 75, 34 m) attacked in the first run, so it was added; gone in the re-check | [live B14] [live B15] |
| Ancient Dragon | 4038, 4039, 4009 | Dragon Knights, NPC type, 2 m (!) and 18 m from the arrival, own setup groups | knights gone [live B18] |
| Smelter Demon (Iron Keep) | 5000, 3071, 3080, 3020, 3060, then 7002, 3025, 7009, 3121, 3120, 3070 | a captain-type enemy (c3180, own setup group) at 13 m and knights 20-29 m out; then, from the hand-placed bonfire, five more unscripted enemies 35-39 m out that were in view (one an Alonne captain, not aggroing); 3070 stays removed at 20.1 m | [live B11] |
| Cave of the Dead trio | 8302 | the last statue-backed bug, 28 m | row passed [live B17] |
| Blue Smelter Demon (DLC) | 8414, 8313, 9502 | the archer is one of two raised enemies (22 m and 28 m out, 5 and 8 m up); 9502 is an NG+2-only NPC phantom 4 m away | row passed [live B17] |
| Duke's Dear Freja | 7000-7003, 7006-7008, 7010, 7032 | area spiders (EnemyParam 203001) between the arrival and the fog; found from kill counts. With the rule's four, 13 removals in all | [live B13] |
| Mytha | 935 | [the Forlorn](../mechanics/forlorn-pc.md) (`Unk84` 2), 21 m from the arrival; its spawn group 4002000 is also switched off (below) | built 5 Nov; Mytha still wins in later runs [live B52]; no check that he is gone [untested] |
| Darklurker's own bonfire (m40_03) | 1000 | the enemy 3 m from the new bonfire (see [Adding a bonfire](adding-a-bonfire.md)); removed at load by the Darklurker's group | built 10 Nov; the bonfire passed [live B2] [live B54]; the enemy's absence was not noted [untested] |
| Giant Lord | 3000, 3010, 3012, 3021, 3022, 3031, 3032, 3100, 3110, 3120, 3121, 3122, 3131, 3132 | the corridor, below | [live B19] |
| NG+ phantoms | see below | exist only from NG+ | not seen [untested] |

Two overrides were wrong the first time: see Pitfalls (Freja's own spiders).

### Finding what to remove: kill counts and live positions

When a fight reports "an enemy came for me on the way to the fog", find its generator like this. [live B13]

1. Read the kill counts of the candidate generators (the ones near the arrival) before the test.
   `tools/arena.py kills <map> <ids>` prints each one's count and how many generators in the map have any kills at all.
   See [Bringing a boss back](respawning-bosses.md) for the layout (`EnemyGeneratorDeadCounter`, one byte per generator).
2. Let the stray enemy come and kill it.
3. Read the counts again. The generator whose byte went up is the enemy's generator.
4. Check it against a live position. A character object holds its `EnemyParam` row pointer at +0x38 and its world
   position (x, y, z floats) at +0x90. The generator's `GeneratorLocation` row plus the map offset should land within
   about a metre of the spawn. We matched the spiders at 0.0 m off at rest. [live B13]
5. Also note the enemy's `AggroGroup`: the mates must go too. A fight's own adds often share the boss's group, but not always (see step 7).
6. Add the generator to the hand-override list, rebuild, reload the map and check that no live body is left for that
   `EnemyParam` row. Compare with a control row that you know is alive. B7's control was EnemyParam 601000 with four
   live bodies. [live B7]

What the evidence for each generator was in Freja's case: "the count went up and no live body was left", plus its
position. The enemy sits in no script and no fight, so deleting it cannot spoil the fight. [live B13]

Scan **both** character classes. Human NPC-type enemies and bosses are `PlayerCtrl` objects, not `CharacterCtrl`, with
the same `EnemyParam` pointer and HP offsets. Our 1 HP helper missed the Cave of the Dead trio (EnemyParam 862000-864000)
until the scan took both classes, and it must never take the player's own `PlayerCtrl`. A "no live `CharacterCtrl`" check
would also pass for an NPC-type enemy that is still there. [live B17] [code]

### Invaders and whole script groups

Some "enemies" are not ordinary generators. An NPC invader or sign is made by a map script that calls `1:131741` with a
generator whose `Unk84` says what kind it is (1 white sign, 3 shade sign, 2 dark-spirit invader, 0 the Dragon
Shrine's spirit). [data, from a separate PC Scholar survey of the event scripts, 4 Nov] The mod has three ways to stop
one.

| Way | Used for | State |
|---|---|---|
| `1:131721` on the invader's generator at load | The Rotten's invaders (592, 593), Mytha's Forlorn (935) | built; the fights pass [live B12] [live B52]; whether it stops a *spirit* is not checked, see Open questions [untested] |
| **Switch off the spawning script group**: delete its `EventParam_<map>` row | Rat maps' offline invaders (group 118220 in m10_33 and m10_34: Bowman Guthry and Rhoy the Explorer), Mytha's Forlorn spawn group 4002000, the Belfry's Bell Keeper invader group 111284 | built; both Rat rows still arrive, fight and win [live B15]; the invaders' absence is not separately seen [untested] |
| `1:131742` on the spirit | none | designed for the Rat maps, never built [untested] |

The deleted-row route is the same switch that removes the [Giant Lord memory's time limit and fireballs](../mechanics/giant-memory-fireballs-pc.md), and it is
proven there for the fireballs: 13 rows for the ambient fireballs (1000-1070, eight launchers every 8-10 s), the
front-line and fight fireballs (2000, 2012, 2013) and the rolling statue head (3000, 3010), plus row 12000 (the
five-minute limit). With the 14 rows gone the memory had no fireballs and no head. [live B53] The limit itself was
never timed past five minutes. [untested] The build deletes rows with `paramrows.delete` on the map's `EventParam`
param. See [Event script groups](../modding-scripts/event-script-groups.md) for the mechanism and why a group needs its row.

Why the group rather than the generator: the Rat maps' invaders are spirits made by `1:131741` when the player enters a
trigger box, as an NG copy (generators 630 and 640, `SpawnLimit` 1) or an NG+ copy (631 and 641, `SpawnLimit` 12). A
`1:131721` at load would run before the spirit exists, and a `1:131742` would have to wait for a spawn whose timing
depends on the player. Deleting the group's row stops the spawn at its source. [data] [inference]

### NG+ phantoms near an arrival

All our arena tests were in NG, where the NG+ enemies do not exist. The NG+ rows are extra generators in the same
`GeneratorParam`, gated by `AppearanceEventID` (a small number = minimum bonfire intensity; a flag plus a cycle in the
next field for a few). The ones that sat at an arrival and slipped past the rule are on the hand list: Flexile's Aurous
Knight phantom 4550 (4.5 m across, 10 m up, needs the ship flag), Heide's Dragonrider phantom 2000 (9.7 m), the Old
Dragonslayer's 9110 and 9120 (4.5 m), the Lost Sinner's Jailer phantoms 6110 and 6310 (21 m, just outside the radius),
Congregation's Hollow Priest 2200 (29 m, on a ledge), the Blue Smelter's 9502, and the Belfry's dogs 6510 and 6511.
[data] The reasoning and the open NG+ check are on [NG+ near boss arenas](ng-plus-differences.md). A raised bonfire
intensity (a Bonfire Ascetic) should bring the same enemies into NG. [inference]

### Removing bosses and allies

An unlimited spawner spawns whatever the kill flag says. A fight you want skipped, or an ally you do not want, has to be
deleted by script. Four cases are in the mod, all deleted with `1:131721`.

| Case | What is deleted, when | Test |
|---|---|---|
| Throne of Want, Nashandra alone | the duo (860, 862) and Aldia (910) at the stage's start | all six choices passed [live B16] |
| Throne of Want, Aldia alone | Nashandra (859) and the duo (860, 862) | [live B16] |
| Crypt, Vendrick | Velstadt (800), after his kill flag is preset so his fogs drop | [live B16] |
| Burnt Ivory King, "the King alone" | ally 797 (EnemyParam 688000), **at load and again every 2 s** | [live B24] |

We never ran a choice with the deletes left out, so "the delete is needed" is reasoned from unlimited spawners, not
shown. [inference]

**The Burnt Ivory King's ally.** The knight comes whatever flags say: the map's script clears the "ally spent" flags
537000026-29 at every load, and switching 537000029 on by hand did not remove him. He spawns when the player nears the
fog, not at load. The trainer killed him with an HP write (the game then counted him as gone). The data route deletes
his generator 797 at load and every 2 s while the map is loaded, because the portal flags the same group sets live in a
fight block that resets per load. [live 30 Oct, trainer] [live B24] In B24 all three fights (King alone, one Loyce Knight,
all the Loyce Knights) worked, including switching between them. We did not test whether one delete at load would have
been enough. [untested] 797's own row has `ActivationEventID1` 537020024, one of the portal flags the "King alone" group
turns on; we do not know whether that matters. [data] [untested] Its condition byte (`Unk0D`) is 8, the byte of the
death-flag rows and the home-map rows. If 8 means "present while the flag is off", that fits the 30 Oct note that he
appears while 537000029 is off, and the by-hand "029 on" not removing him would mean the condition is read when he is
built or spawned, not live. [data] [inference] See [GeneratorParam fields](generatorparam-fields.md). His gate is still
open.

**The Giant Lord's corridor.** Between the front fog and the Giant Lord stand 14 generators in `AggroGroup` 30-33:
twelve soldiers and two giants, tied to no fight flag. The trainer's teleport had landed past them, so they were only
met when the memory was entered on foot. The mod removes all 14 at load. The memory's NPC summon signs (656, 657) are
kept. [data] [live B19] The Giant Lord himself wakes within 30 m or when hurt; in B19 a test tool's 1 HP write at load
counted as a hit and he walked to the player across the fog. See pitfalls. [live B19]

## How to do it

For a modder who wants a calm spot around a new arrival point, or a boss alone in his room:

1. **List the generators near the spot.** Read `GeneratorParam_<map>`, `GeneratorRegistParam` and
   `GeneratorLocation_<map>`. Add the map offset from the MSB, then keep those within about 20 m across and 8 m up or
   down. [data]
2. **Leave out what must stay:** the boss's fight generators (found by `ActivationEventID1` = fight flag, plus the
   fight's adds, see [Bringing a boss back](respawning-bosses.md)), summon signs and invaders (`Unk84` not 0, or
   `SpawnLimit` 255), and friendly NPCs. Hold back anything the map's script names until you have read what it does.
3. **Close the list over aggro groups.** Every generator that shares an `AggroGroup` (not 0 or 255) with a listed one
   goes on the list, unless it is a fight generator.
4. **Write the commands into a load-time script group.** In a new group with its own `EventParam_<map>` row (see
   [Event script groups](../modding-scripts/event-script-groups.md)): first state moves on only; the second state's entry has
   `1:130101` for any map-local flags first, then `1:131721(gen, 0)` for each generator.
   For a test on one enemy, `tools/datapatch.py addcmd <script> <group> <state> 131721:<gen>,0` appends one command to a
   state and refuses the write unless the script diff touches that group only.
5. **Reload the map** (travel away and back). No game restart is needed. [live B8]
6. **Walk the route from the arrival to the fog.** For each enemy that still comes, do the kill-count steps above, add
   its generator and aggro mates, rebuild and reload.
7. **Check what you removed was not part of the fight.** A fight's adds share the boss's aggro group *or* are woken
   by his fight flag or a script, and not always both together: Freja's 12 spiders carry her fight flag and group 100,
   while her own row is group 240. Look for both. If a "removed enemy" turns up in the fight script, put it back.
8. **For a whole scripted system** (invaders, ambient fireballs, a time limit): find its script group by the `.edd`
   name, and delete its `EventParam_<map>` row. Keep the rows of groups the fight needs. See
   [Event script groups](../modding-scripts/event-script-groups.md).
9. **For a boss or ally you do not want**, delete its generator at load, and repeat every 2 s if it spawns only near
   the fog (the Burnt Ivory King's ally did).
10. **Do not test with a tool that damages everything at load.** It wakes bosses whose script wakes on damage (see
    pitfalls).

## Pitfalls and what failed

| What went wrong | Why | Fix | Test |
|---|---|---|---|
| `ActivationEventID1` edit did not quiet the enemy | not an AI switch for ordinary enemies; the boss rows' use of it is their fight flag | script commands instead | B7 |
| Removed one enemy, both attacked | the two shared `AggroGroup` 71; one noticing wakes the rest | remove the group | B7 |
| Removals in the new group's first state left the enemies | a group's first state ignores entry commands | put them in the second state | B9 |
| Freja's first arrival: spiders between the bonfire and the fog | the computed bonfire spot, and area spiders the rule did not list | new hand-placed bonfire spot, 9 area spiders added | B13 |
| A build removed six of Freja's own spiders (8010-8015) | `tools/bosstable.py` looked only at the o00_0500 / 0501 fog models and took her far exit fog (61 m away); her real entry fog is an o00_0502, 9 m from the new bonfire. The six are inside her fight (EnemyParam 203002, aggro group 100, tied to her kill flag) | put back; the author knew the fight | B13 |
| The Chariot's red phantom, the Dragon Knights and the Iron Keep captain were not on the list | the rule never removes NPC-type enemies or script-named generators; found from the author's notes after the 4 Nov survey, then built | hand overrides | 4 Nov survey; B11, B12, B18 |
| One horse knight aggroed at the Looking Glass Knight | generator 3010 shares aggro group 75 with the three removed, 34 m out; no rule covers "same group but far" without a removed member | added by hand | B14 |
| Iron Keep: a tester saw "at least a dozen enemies within 10 m" of the landing | the data showed few near the community spot; the computed bonfire then stood inside the room, so the arrival moved to a hand-placed bonfire outside it | recount from the hand-placed spot; five more removed | B11 |
| Enemies 10-28 m **above** the arrival were left | the 8 m vertical window; Freja's high spiders (10-28 m up) stayed put and did not matter | accepted | B13 |
| The Giant Lord walked 100 m to the player at the fog | the test tool's 1 HP at load counted as a hit; his boss script wakes him within 30 m or when hurt. Not the mod | the tool waits for his fight flag | B19 |
| The same 1 HP tool woke Vendrick on arrival, and killed the still-friendly Ancient Dragon on the first hit | the same family: a hit wakes a sleeper, or ends a fight that has not started (her boss routine starts only after four hits) | the tool waits for the fight flag | B39 |
| NPC-type enemies invisible to a memory scan | they are `PlayerCtrl`, not `CharacterCtrl` | scan both | B17 |
| Burnt Ivory King's ally came despite flags | the game sets and clears the "ally spent" flags itself | delete generator 797, repeat every 2 s | B24 |

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| `ActivationEventID1` is not an AI switch for ordinary enemies | live: edit found in memory, enemies attacked | B7 |
| Enemies of one `AggroGroup` wake together | live: two attacks | B7 |
| `1:131721` and `1:131731` in a load-time group remove the enemy; no live character | live: memory scan with a control row | B7 |
| A new group with its own `EventParam` row runs; first-state commands do not take | live | B9 |
| Removals in the arena group work on arrival | live | B9, B10 |
| Pursuer: 7020 and 7021 (aggro group 143) removed, fight unaffected | live | B10 |
| Hand overrides for LGK, Dragon Shrine, Iron Keep, Freja, Chariot, Rotten | live passes; each found by kill count or sight | B11-B15, B17, B18 |
| Freja's fight spiders must stay | live: six of her own removed by mistake, put back | B13 |
| NG+ generators in a removal list are harmless in NG | live: three rows | B11, B12 |
| Giant Lord corridor: 14 generators, groups 30-33 | data; removed | B19 |
| Giant Lord fireballs, head and time limit gone by deleting `EventParam` rows | live | B53 |
| Burnt Ivory King: delete 797 at load and every 2 s; all three modes work | live | B24 |
| Throne and Vendrick choices with deleted stage bosses | live | B16 |
| NPC-type bosses and enemies are `PlayerCtrl` | live: 1 HP missed the trio until both classes were scanned | B17 |
| The rule: 20 m, 8 m, never fight / script-named / `Unk84` / `SpawnLimit` 255 / NPC type, plus aggro group | build tool | `tools/rollout_table.py` |
| 122 generators removed across 16 maps | the build table | `rollout.json`, 10 Nov |
| FromSoftware uses 131721 13 times, 131722 70, 131731 2, 131732 4 | the 28 event scripts the project reads | `.esd` files |
| Rat maps still play with the invader group's row deleted | live: both rows arrive, fight, win | B15 |
| Invader / Forlorn / Bell Keeper switch-offs actually stop the spirits | built; not seen | `tools/rollout_table.py` |

## Tools

All in `tools/`:

| Tool | What it does here |
|---|---|
| `rollout_table.py` | the rule (`QUIET_R`, `QUIET_DY`), the hand overrides (`FORCE_QUIET`), the script groups to switch off (`DISABLE_GROUPS`, `GIANT_LORD_QUIET_EVENTS`); writes `rollout.json` and `rollout.md` |
| `arenamod.py` | builds the per-map arena group: flags first, then one `1:131721` per listed generator, then the win flow |
| `buildmod.py` | writes the groups into the event scripts; deletes the `EventParam` rows of `DISABLE_GROUPS` with `paramrows.delete` |
| `paramrows.py` | add and delete rows in a loose param (`delete <param> <row>`); keeps a `.orig` |
| `datapatch.py` | `addcmd` appends commands to a state's entry list, refusing the write unless only that group changes; `restore all` undoes |
| `slice_lastgiant.py` | the first one-boss build (B8), with B7's `QUIET` list |
| `choices.py`, `specials.py` | the stage deletes (Throne, Vendrick), the Burnt Ivory King's ally 797, the Giant Lord's removals |
| `arena.py` | `kills <map> <gens>`: read generator kill counts (step 1 and 3 above) |
| `arenalib.py` | `CHR_CLASSES` (both character classes), `chrs_of`: find live characters by `EnemyParam` pointer |
| `batchwatch.py` | live logger of kill counts; `WAKE_ON_HURT` (Giant Lord, Vendrick, Ancient Dragon) holds the test 1 HP until their fight flag |
| `bosstable.py` | `gens_of(map)`: generators with position and enemy id; finds fight generators |

## Open questions

- **Is a single delete at load enough for enemies that spawn only near the player?** The Burnt Ivory King's ally was
  deleted every 2 s as a precaution; we never tried once. [untested]
- **Can a deleted enemy return later in the same load,** for example by leaving its area and coming back without a
  map reload? The tests were short. [untested]
- **Does a force-deleted enemy change its generator's kill count?** We read counts only after kills. [untested]
- **Are the invaders really gone?** The Rat maps' group, Mytha's Forlorn and the Belfry's Bell Keeper were switched
  off and the fights pass, but no one walked the invasion trigger and looked. [untested]
- **Does `1:131721` on a spirit's generator at load stop the invasion?** The Rotten's invaders (592, 593) and Mytha's
  Forlorn (935) rely on it. Spirits are made later by `1:131741`, so the delete may find nothing to delete; Mytha's
  Forlorn is also stopped at its group, the Rotten's are not. [untested]
- **The NG+ phantoms on the list** have never been met: all tests were in NG. A raised bonfire intensity should show
  them. [untested]
- **A data-only way to keep an enemy out:** raise `AppearanceEventID` above the bonfire's level. NG+ rows with value 2
  stayed out in NG even with `SpawnLimit` 255 [live B13], so the gate works, but nobody has set it on purpose.
  [untested]
- **Other switches:** `GeneratorRegistParam.SpawnState`, moving the enemy with its `GeneratorLocation` row, the group
  forms `1:131722` / `1:131732`, and `1:131742` for spirits. [untested]
- **Does removing an enemy that has a `DeathEventID` (death opens something) break the thing it opens?** The rule
  avoids such enemies because their scripts name them; no case has been tried. [untested]
- Whether the NPC-type phantoms (Chariot, Dragon Knights) are `PlayerCtrl` like the Cave of the Dead trio: only the
  trio was checked. [untested]

## Related pages

- [Bringing a boss back](respawning-bosses.md): kill counts, `SpawnLimit`, unlimiting the fight's adds, stale flags
- [Event script groups](../modding-scripts/event-script-groups.md): the group and `EventParam` row the removals live in; deleting rows
- [Moving a bonfire](moving-a-bonfire.md) and [Adding a bonfire](adding-a-bonfire.md): where the arrival point is
- [GeneratorParam fields](generatorparam-fields.md): `AggroGroup`, `ActivationEventID1`, `AppearanceEventID`, `Unk84`
- [NG+ near boss arenas](ng-plus-differences.md): the cycle-gated enemies in full
- [Warping by script](warping-by-script.md) and [Script command reference](../modding-scripts/script-command-reference.md)
