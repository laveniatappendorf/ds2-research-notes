# Bringing a boss back: kill flags, kill counts and SpawnLimit (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

Tags in square brackets say how we know a claim: `[live B8]` = seen in the game (the id points to the project's test
log, see [Home](../how-to-read.md#the-modding-pages)), `[code]` = read in the exe, `[data]` = read in the game files, `[inference]` = reasoned, not
proven, `[untested]` = an idea only.

## Summary

- **Clearing a dead boss's event flags is not enough.** The fog gate, the health bar and the intro cutscene come
  back, but the boss does not spawn. [live A2]
- What keeps him away is a **per-spawner kill count** (one byte per generator in `EnemyGeneratorDeadCounter`). A
  spawner only spawns while its kill count is below its limit. [live A2] [code]
- **Memory route:** set the boss generator's kill count to 0, clear his kill flags, reload the map. Only that boss
  returns. [live A2]
- **Data route (what the mod ships):** set `SpawnLimit` and `SpawnLimitGameClear` to **255** on the boss's
  `GeneratorParam` row. 255 means "no limit", so the count no longer matters. No memory writes, nothing to re-arm;
  he came back on every visit: three wins in a row [live B8], all 44 boss rows [live B10-B19], a 39-fight run [live B39].
- The kill is credited **about 6.5-7 s after HP reaches 0**, not at HP 0. A warp or a death inside that window
  cancels it harmlessly; after it the souls are kept. [live A6] [live B10]
- The traps: summoned adds that FromSoftware gave `SpawnLimit` 1 (the Skeleton Lords' skeletons) never come back
  unless unlimited too [live B39]; a kill flag still on at load makes the boss stand down for that load [live B16]
  [live B38]; an unlimited spawner ignores the kill flag, so a boss you want skipped must be deleted by script
  [inference].

### What "dead" is made of

A boss row (`BossBattleParam`) names two flags, and the boss's generator adds a third thing, the kill count. They are
separate, and only the count gates the spawn.

| Piece | Where it lives | Last Giant | What reads it |
|---|---|---|---|
| Kill flag (`boss_destruction_flag`) | map-local event flag; in memory only while that map's flag block is loaded [live A1] | 110000081 | the map's boss script at load; the fog gates follow it [live A2] |
| Map-clear flag (`map_clear_global_event_flag`) | global flag block 10 | 100971 | Majula's scripts, the Herald's boss counter, NPC generators [data]; also the `bonfire_lit_flag` of the area's bonfires |
| Intro flag (`PreviousHostileEventID1` of the boss generator) | map-local | 110000082 | the intro cutscene; set when you first cross the fog [live A2] |
| Fight-state flags | map-local | 110020080 on when you cross the fog, off at HP 0; 110020084 on at HP 0 | the boss bar and the fight script [live A2] |
| **Kill count** | one byte per generator in `EnemyGeneratorDeadCounter` | generator 857 | the spawn check [code] |

The kill flag and the map-clear flag are the pair a kill sets: both turn on in the same tick as the souls (+10,000
for the Last Giant). [live A2] The boss generator's own `DeathEventID` (+0x04) is 0 on the boss rows we read, so no
flag is wired to the spawn in the param. [data]

Event flags outside the loaded blocks cannot be read or written. The global blocks and the current map's blocks are
loaded; a boss's map-local flags exist only while you are in his map. [live A1] Arena flags are sometimes readable
from Majula when a flag block happens to still be loaded, but do not rely on it. [live B38] See
[Event flags and choosing safe ids](../modding-scripts/event-flags-and-free-ids.md).

### The kill counter

`EnemyGeneratorDeadCounter` is reached as `[[[GameManagerImp]+0x40]+0x10]`, that is `EnemyGeneratorManager` +0x10.
The `GameManagerImp` pointer is the static at RVA +0x16148F0 (Ghidra `DAT_1416148f0`). [live A2] [code]

| Offset | Content |
|---|---|
| +0x8 + n x 0xB10, n = 0..42 | map slots; our tools scan 43 (n = 42 is at +0x1D0A8) |
| slot +0x0 | map key, 4 bytes `{00, 00, bb, aa}` for map `m<aa>_<bb>`; m10_14 is `00 00 0e 0a` |
| slot +0x4 | u16, number of generators in the slot |
| slot +0x10 | u16 generator ids, sorted (the game binary-searches them) |
| slot +0x210 | u8 kill counts, same index as the ids |

- The Forest slot (m10_10) holds 149 ids, the 149 `GeneratorParam_m10_10` rows. The Last Giant is generator 857
  (enemy 309600), the Pursuer's boss generator is 853. [live A2]
- The two bytes of the map key were first written the wrong way round (`{aa, bb}`) in our lookup. m10_10 reads the
  same both ways, so the Forest tests passed; the lookup for other maps failed until it was fixed on 30 October.
  [live A2] The respawn record uses the same map-byte order, see [The respawn point](respawn-point.md).
- The id table is 0x200 bytes, so a slot seems to hold at most 256 generators. The largest map we had has 224
  (m50_37). [inference]
- **A map's slot is only in memory once that map has loaded in the current game session.** On a fresh start the Looking
  Glass Knight's count read "not found" from Majula and 0 from his own map. A tool that re-arms from Majula must
  reset again after the arrival load. [live, 4 Nov, recorded in `tools/arenactl.py`]
- The count went up by 1 per boss kill: 1, 2, 3 over three wins in a row [live B8], 5 after more kills in the same
  session [live B9]. Counts survived map reloads within a session. One oddity: the Dragonrider's count read 0 again
  on the next Heide load. [live B11]

### The spawn check

`FUN_140412060` (RVA +0x412060, 13 references: 12 calls and one data reference) decides whether a generator may
spawn its enemy. Its count getter is `FUN_1401f6760` (RVA +0x1F6760): it finds the generator id in the map slot and
returns the byte at +0x210; an id that is not found returns 0. [code, read again for this page on 17 Nov] The rule, in
words:

A generator **may spawn** if any of these is true:

1. its limit byte at generator +0x8D is 0 (no limit);
2. bit 0 or bit 1 of the byte at `[[generator+0x68]+0x76]` is set (what that object is has not been identified);
3. its kill count is below its limit;
4. it is exempt: bit 7 of its param row's byte +0x48 (community name `Unk48_7`) is set **and** bit 0x10 of
   `EnemyGeneratorManager` +0x305 is set. That manager byte was 0x26 in every probe, so the switch is off. [live B8]

Otherwise the spawn is refused. The check's inputs are the generator's map id (`[[generator+0x50]+0x2C]`), its id (u16
at +0x88) and its param row pointer (+0x58). [code]

**Where the live limit at +0x8D comes from.** A read-only probe of the live generator objects (B8) found, for the
Forest: `SpawnLimit` 0 gives **1** on the boss generators 857 and 853 and **12** on the ordinary enemy generators in
the probe (3150 and 2810 among them); `SpawnLimit` 1 (generator 8100) gives 1; `SpawnLimit` 255 (the summon row 527)
gives **0 = no limit**. [live B8] The code that sets +0x8D is `FUN_140412e70` (RVA `+0x412e70`), run when the
generator is built: it takes `SpawnLimit` in NG and `SpawnLimitGameClear` from NG+ on, turns 255 into 0 (no limit), and
for a 0 byte takes a default from another param row found by the enemy's id, which is where the 1-or-12 comes from
[code; details and the NG+ rule on [NG+ differences](ng-plus-differences.md)]. So with the default, a boss spawns once
ever and an ordinary spawner stops after 12 deaths [inference from the limit]. Whatever brings bosses back in the vanilla game (a
new game cycle, a Bonfire Ascetic) must reset these counts [inference].

### GeneratorParam fields that matter here

Row size 0xA0, little-endian, community field names from Smithbox's DS2S definitions. [data]

| Offset | Field | Used for |
|---|---|---|
| +0x00 | `SpawnLimit` (u8) | the limit, 255 = none |
| +0x01 | `SpawnLimitGameClear` (u8) | the limit once the intensity total reaches 2 (the first NG+, or one Bonfire Ascetic at that generator's bonfire); the mod writes both [code, see [NG+ differences](ng-plus-differences.md)] |
| +0x03 | `AggroGroup` (u8) | linked aggro; helps find a fight's adds |
| +0x04 | `DeathEventID` (u32) | 0 on every boss row we read |
| +0x14 | `PreviousHostileEventID1` (u32) | on the boss row = the intro flag (110000082) |
| +0x48 bit 7 | `Unk48_7` | the exemption bit in the spawn check |
| +0x5C | `ActivationEventID1` (u32) | on a boss row = his fight flag (110020080); this is how we find a fight's generators |

Vanilla values we read in the loose per-map files: boss rows 857, 853 (Forest), 860 / 862 / 859 / 910 (Drangleic
Castle) all `SpawnLimit` 0 / `SpawnLimitGameClear` 0. The summoned adds are 1 / 1, except the Looking Glass Knight's
mirror summons 5000 / 5001 (1 / 0). The player-summon rows (for example 527 in the Forest) are 255 / 255. [data]

### A kill, step by step

Times from the Last Giant and the Pursuer's wins. [live A2] [live B8] [live B10]

| When | What happens |
|---|---|
| t = 0 | HP reaches 0; fight-state flag 110020084 turns on (first read as a phase change, then as "HP 0") [inference] |
| t + 6.5 to 7 s | the death animation ends: **kill flag and map-clear flag on, souls and boss soul added, kill count + 1**, all in one tick as far as the 50 ms flag watcher could tell |
| kill + 1 s (mod) | the arena group clears the kill flag |
| kill + 8 s (mod) | the arena group warps the player to Majula |

- Before the credit, a warp cancels the kill: no souls, no flag, no count, and the boss is back at the next visit.
  The Forest unloaded 6.1 s after 110020084 came on (the warp key pressed as the bar emptied) and the kill was
  lost. [live A6]
- Before the credit, a death cancels it the same way (falls at +4 s and +6 s). After the credit a death keeps the
  souls; the flag was already cleared. A death at the moment of the warp's fade ends in Majula with the souls.
  Either way nothing is left stale and the boss is back. [live B10]
- A boss that falls to his death also counts: Flexile Sentry's count went up the moment he fell with the
  player. [live B39]

### The memory route

This is the trainer's route and the first one we proved. It needs a program writing to the game's memory, so it is for
testing, not for shipping.

1. Find the boss's generator id. The fight's generators are the `GeneratorParam` rows whose `ActivationEventID1` is
   the fight flag; if there is none, the rows with the boss's enemy id. [data] (This is `tools/bosstable.py`'s rule.)
2. Write 0 to that generator's count byte: slot address + 0x210 + index of the id in the slot.
3. Clear the kill flag and the map-clear flag (bit writes; the kill flag only while his map is loaded).
4. **Reload the map** if the kill flag was still on when it loaded. Then go near the arena: the boss is not in memory
   after the load, he spawns when the player approaches. [live A10]

Proof: killed the Last Giant (count 0 to 1, flags on, the only non-zero count in the Forest), re-armed with
`tools/arena.py rearm 10_10 857 110000081,100971`, warped to another bonfire and back, teleported to the fog: **a new
Last Giant at 2,530 / 2,530 HP**. [live A2] A second kill on the re-armed boss paid out as the first did
(+10,000 souls about 6.5 s after 110020084). [live A2]

How we found it: flags alone gave fog, bar and (once 110000082 was cleared) the intro cutscene, but memory showed
the boss's `EnemyParam` loaded with **no `CharacterCtrl` pointing at it**. [live A2] The community Cheat Engine table's
"Respawn All Enemies" entry led us to the count: it hooks the instruction that reads the count byte (RVA +0x1F67C5,
inside the getter) and zeroes it on every read, so **every** enemy respawns, not just the boss. With it ticked and the
fog flags cleared, the Last Giant was there. [live A2] Our trainer clears only the boss's own generators.

### The data route

The idea: make the boss generator ignore its count. The GeneratorParam row for the Last Giant is
`GeneratorParam_m10_10` row 857; set `SpawnLimit` and `SpawnLimitGameClear` from 0 to 255. The game reads loose
per-map params at each map load, so a reload is enough. [live B8]

Result of the first test (B8, nothing else re-arming him; the only memory writes were a 1 HP helper for the low-level
test character and read-only probes): the live limit became 0 and the Last Giant was back at kill counts 1, 2 and 3
(three wins in a row, each followed by the warp home). The intro did not replay (110000082 stays on). [live B8] The
param edit was in place before his first kill, so we never saw the data route's own "no boss at count 1" case; A2
(memory route) shows what the default does.

What the shipped mod adds around the param edit, per map, as one arena script group (see
[Event script groups](../modding-scripts/event-script-groups.md)):

| State | Does |
|---|---|
| s0 | nothing but move on (commands in a group's first state did not take) [live B9] |
| s1 | at load: any map-local setup flags (`1:130101`, 【フラグ】フラグを設定, "set flag"), removals (`1:131721`, 【ジェネレータ】敵の強制削除, "force-delete enemy"); if a kill flag is already on, go to s2, else s3 |
| s2 | stale kill flag: clear it (that boss fights from the next load) |
| s3 | wait for a kill flag (`f6(flag)`, フラグの取得, "get flag") |
| s4 | wait 1 s (`f3()`, ステート経過時間の取得, "time in this state"); the souls come in the flag's own tick |
| s5 | clear the kill flags, then wait 7 s more |
| s6 | warp to Majula's bonfire landing: `1:132125(0, 0, 10040000, 200000, 0)` (【マップ】プレイヤーをポリ劇ワープ, "poly-drama warp the player") |

The group has its own `EventParam_<map>` row (event id 990000, flag 0 = runs at every load), because a script group
only runs when the map's EventParam has a row for it. [live B9] The group id 990000 was free in every script we checked.

Scale of the roll-out: 131 generators are unlimited across 22 maps for 44 boss rows (the boss generators and their
adds), written by `tools/buildmod.py` from the roll-out table. [data] Every row was played in batches. [live B10-B19]

The point of the data route over the memory route: it works in a normal install with loose params, with no trainer,
and an aborted or interrupted kill leaves nothing to clean up (see "Stale kill flags" below).

### Summoned adds

A fight's summoned adds are spawners too, and FromSoftware set most of them to `SpawnLimit` 1 because the fight
happens once. After one win they never spawn again. The Skeleton Lords showed it: the fight needs the skeletons
each Lord calls, so a second visit stalled with no kill flag, ever. [live B39]

What we found by data (vanilla `GeneratorParam`, all `SpawnLimit` / `SpawnLimitGameClear` 1 / 1 unless noted): [data]

| Fight (map) | Add generators | Notes |
|---|---|---|
| Skeleton Lords (m10_23) | 5200-5201, 5210-5214, 5220-5231 (19 rows) | aggro group 43, as two of the Lords; enemies 133001 / 157008 / 157013 / 157025 |
| Elana (m50_35) | 8500-8508 (9 rows) | summons, aggro group 230 |
| Looking Glass Knight (m20_21) | 5000, 5001 | mirror summons, aggro group 240; 1 / 0 |
| Covetous Demon (m10_17) | 3300-3303 | switched on by his fight flags 117020085-117020088 |

These 34 generators (19 + 9 + 2 + 4) are `rollout_table.EXTRA_UNLIMIT`. They were found by scanning all 44 rows for
spawners with 1 <= `SpawnLimit` < 255 within 30 m of a boss, and kept only if they share the boss's aggro group or are
switched on by its fight flags. [data] Elana's, the Looking Glass Knight's and the Covetous Demon's fights still end
without their adds, but would have played without them after the first win. Fixed and installed 7 Nov; all 34 read
back 255. [live B39]

Fight generators the automatic search missed (`EXTRA_GENS` in `tools/bosstable.py`):

- **The Skeleton Lords are three generators** (841, 840, 839). The search found only the first, so two Lords had no
  re-arm and would have stayed dead after one win. A scan of all 44 rows for same-kind generators within 40 m found no
  other fight like it. [live B12]
- **The Throne duo is two generators**, 860 and 862, both on fight flag 221020090; only 860 was found at first, so the
  second would not have re-armed. [data]
- **The Lost Sinner's two pyromancer phantoms** (generators 8100 and 8101 in m10_16, woken at about 70 % HP in NG+ or
  at a raised intensity through `ActivationEventID2`) are unlimited as adds. [data] See
  [NG+ near boss arenas](ng-plus-differences.md).

### Stale kill flags at load

If the kill flag is already on when a boss map loads, **the boss stands down for that load**: the fight does not
start, and clearing the flag after the load only helps from the next load. [live B16] [live B38] We think the map's
own boss routine decides from the kill flag at load [inference]. The ways a flag ends up on:

- the boss was killed before the mod, on the same save;
- the player quit within 1 s of a kill (the group clears the flag 1 s after it comes on);
- a trainer or test left it on (the first Nashandra trip stood down for this reason). [live B16]

The shipped fixes:

- **Standard group:** clear the flag at load (s2). The boss fights from the next load. [live B16]
- **Boss rush and the fight list:** every warp to a fight carries an arrival token. A fresh arrival with a stale flag
  clears the flags, waits 1 s, then **reloads the same map at the same spot** (once per arrival, so no loop); the boss
  is there on the second load. Tested live: Heide, the Throne of Want (a gauntlet death leaves stage flags on) and a
  fight-list trip to a boss marked dead. [live B38]
- **Wait for the player to be in the map** (`1:140402(0, 1, 0)`, 【グループ条件：マップ】プレイヤーがマップ内にいる判定,
  "is the player inside the map?", then `f140001(0)`, "group condition met") before deciding anything. A map that is
  only **streaming in next door runs its event scripts too**. After Najka's kill the Royal Rat Authority's map streamed
  in while the player was still next door; its run guard saw "no token" and ended the run. [live B39]

To test a stale arrival on purpose, set the flag back on the moment the group clears it after a real kill
(`tools/stalekill.py --rearm <row>`). The flag usually cannot be set from Majula, because the arena's flag block is not
loaded there. [live B38]

### Clearing the kill flag 1 s after it comes on

Clear it **1 s after** it turns on, not at the warp. If the player leaves during the 8 s wait (by running out, the
Darksign or bonfire travel) with the flag still on, the map would unload with the flag set and the next visit would
have no boss. The souls and the boss soul come in the flag's own tick, so a 1 s wait is safe. [live A2] The first
slice cleared it only at the warp (B8); the early clear was built in B9 and the test walked out after it and found the
boss back at the next visit. [live B9] Likewise, nothing goes in a group's first state; put work in the next one.
[live B9]

### The fog

The fog gate follows the kill flag, with exceptions we saw but did not map:

- With the Last Giant dead at load, clearing 110000081 (written from outside the game, with the map loaded) brought his
  fog back at once, with no reload. [live A2]
- Once inside with the flags cleared and no boss, clearing the fight flags did **not** release the fog: the running
  fight script owns it. [live A2]
- In the load where a boss dies, the Last Giant's fog **did not come back** after the 1 s clear: the player walked out
  before the warp. The log's explanation is that his fog script had finished. [live B9] [inference]
- After the Twin Dragonriders' win the exit fog was down, so the player could leave before the warp. The tester's
  remark was that most other arenas keep their fog up in that gap; we did not test each arena. The Dragonriders' fogs
  are driven inside a shared boss routine (m20_21 group 7000), not by entities in the map's own groups. [live B39]
  [data]

So the fog after a win is per arena. If your mod needs to hold the player in, or let them out, test that arena.
[untested] An idea: leave the kill flag on after a win to keep the fog down for a "stay here" option; whether the fog
is then down is not known (B59, step 5, not yet run). The idea assumes a cleared flag keeps most fogs up, which sits
oddly with the Last Giant case above.

### The map-clear flag and the Bonfire Ascetic

The map-clear flag is **world state**, not part of the re-arm. 100971 is the Last Giant's "boss beaten" flag; the same
number is the `bonfire_lit_flag` of Cardinal Tower, Soldiers' Rest and Crestfallen's Retreat (the whole area shares it;
the Place Unbeknownst has 100972, the Giant Lord's). The values equal each area's `BossBattleParam`
`map_clear_global_event_flag`. [data] It is read by Majula's world-map flames, the Herald's boss counter, and by NPC
generators (for example Melentia's move to Majula). [data] So the mod **never clears it**: a data-route win sets it
and it stays. It is not the lit state of the bonfire: the travel menu follows the bonfire array's level byte alone.
[live B22] See [Bonfire lit state](bonfire-lit-state.md).

(Our first data-route notes said never to clear 100971 because clearing it would drop the travel entry. That was a
precaution. Later tests showed the travel menu follows the level byte alone [live B22], so that reason does not hold;
the rule survives for the world-state reason. The trainer route clears the flag anyway and we noted no ill effect.
[live A2])

The **Bonfire Ascetic**: burning one at the Last Giant's arena bonfire raised his max HP from 2,530 to 5,313, the same
as setting the bonfire's level byte from 1 to 2. [live B56] [live A10] Whether it also re-arms a killed boss was not
tested, because the data mod re-arms him anyway. We think an Ascetic clears that shared map-clear flag and resets the
area's kill counts; neither was observed. [inference]

### Data route (recommended)

1. **Find the boss's generators.** In `GeneratorParam_<map>`, take the rows whose `ActivationEventID1` equals the
   fight flag (for the Last Giant: 857 on 110020080). A multi-boss fight has several. `tools/bosstable.py` does this and
   writes `bosses.json`. [data]
2. **Find the fight's adds.** Look for spawners within about 30 m of the boss with `SpawnLimit` from 1 to 254 that
   share the boss's `AggroGroup` or are switched on by the fight's flags. Add them to the list. (`rollout_table.py`
   scans for them; the four fights above came out of that scan.)
3. **Write 255 to `SpawnLimit` and `SpawnLimitGameClear`** on every one of them:

   ```
   datapatch.py param generatorparam_m10_10_00_00 857:SpawnLimit=255 857:SpawnLimitGameClear=255
   ```

   The loose file is `Param\generatorparam_<map>_00_00.param`; the tool keeps a `.orig`, checks that only the
   intended bytes changed and reads the result back. See [Editing params](../modding-scripts/editing-params.md).
4. **Reload the map.** Params are read at each load; no game restart is needed. [live B8]
5. **Check live (optional).** `tools/slice_lastgiant.py probe` reads the live generators' +0x8D, which should be 0
   for every generator you unlimited.
6. **Add the win flow:** wait for the kill flag, clear it 1 s later, then do whatever the arena should do next (the
   mod warps home 8 s after the kill). Put it in a new group with its own `EventParam` row and nothing in the first
   state. See [Event script groups](../modding-scripts/event-script-groups.md).
7. **Handle the stale flag at load** (clear it, or reload in place), and wait for "player in this map" before
   acting.
8. **Delete bosses that should not appear.** An unlimited spawner spawns whatever the kill flag says, so marking a
   boss "dead" no longer keeps him away. For a stage you want skipped (Velstadt when the Vendrick choice is made),
   delete the enemy at load with `1:131721(generator, 0)`. [inference from B7 / B8; the Vendrick choice passed in
   B16] See [Removing enemies](removing-enemies.md).
9. **Never clear the map-clear flag** in the win flow.

### Memory route (for tests and trainers)

1. Look up the boss's generator ids and map (`tools/bosstable.py`).
2. In his map, `tools/arena.py kills 10_10 857` shows the count; `tools/arena.py rearm 10_10 857 110000081,100971`
   writes 0 and clears both flags. The kill flag clears only while his map is loaded. [live A2]
3. If the kill flag was on at load, or the slot was not in memory before, warp away and back (a map reload). [live A2]
4. Go near the arena; he spawns on approach. [live A10]
5. For the intro cutscene, also clear the intro flag (110000082). [live A2]

The trainer (`tools/arenactl.py`, F5) does it in this order: reset the counts of all the fight's generators and the
map-clear flag from outside, set the respawn point, warp, clear the kill flag once the map is loaded, reset any
count that was "not found" before the load, and reload the map if the boss was dead at arrival.

## Pitfalls and what failed

| What went wrong | Why | Fix | Test |
|---|---|---|---|
| Flags cleared, boss not back | the count is the gate, not the flags; boss `EnemyParam` loaded but nothing spawned | reset the count or unlimit the spawner | A2 |
| Through the fog with everything cleared: no boss, fog trapped the player, boss bar shown | same | same; the fog stays up while the fight script is running | A2 |
| "Respawn All Enemies" re-armed the boss and every other enemy | its hook zeroes the count byte on every read | write only the boss generators' bytes | A2 |
| Warp as the boss dies: no souls, no flag | the kill is credited ~6.5 s after HP 0 | the win flow waits for the **kill flag**, never fires on HP 0 | A6 |
| A death in the credit window cancelled the win | same | accepted: it is what the game does; nothing is left stale | B10 |
| Two of the Skeleton Lords never re-armed | the search found one Lord; the fight is three generators | `EXTRA_GENS` in `tools/bosstable.py` | B12 |
| The Lords' skeletons never came back | their spawners are `SpawnLimit` 1; they died once | `EXTRA_UNLIMIT`, 34 generators | B39 |
| 1 HP test tool killed a boss that was still friendly (Ancient Dragon) | her boss routine starts on a talk flag set after four hits; the **count went to 1 but no fight or kill flag ever came on**, so no souls and no warp | test tool waits for her fight flag; a kill count is not a kill flag | B39 |
| Boss missing on arrival | kill flag still on at load | clear it and reload in place | B16, B38 |
| Run ended after Najka's kill | the next map streamed in and its run guard decided | wait for the player in the map | B39 |
| Risk: a kill flag still on when the player leaves early leaves the next visit without a boss | the slice cleared it only at the warp | clear 1 s after the kill (verified by walking out) | B9 |
| NG+-only adds did not appear with 255 (Flexile's two shadows in NG) | the limit does not override the cycle gate | no fix: not a limit problem | B13, weak: no one reported seeing them, nothing more [live, by absence] |
| A re-arm from Majula found no count slot on a fresh game | the map's slot does not exist until the map has loaded | reset again after the arrival load | 4 Nov, `arenactl.py` |
| Risk: an unlimited spawner brings back a boss you wanted skipped | it ignores the kill flag | delete him by script | design from B7 / B8; B16 passed |
| Writing a map-local flag from another map does nothing | the setter returns 0 while the flag's block is not loaded | write it after arriving | A1 |

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| Kill flags alone give fog, bar, intro, no boss | live: two attempts; boss `EnemyParam` loaded, no `CharacterCtrl` | A2 |
| The kill count is `[[[GameManagerImp]+0x40]+0x10]`, slots of 0xB10 | live memory reads; code | A2, `FUN_1401f6760` |
| Re-arming one generator's count + kill flags + reload gives a full-HP boss | live: killed, re-armed, reloaded, teleported | A2 |
| Bosses spawn when the player nears the arena | live: not in memory after the load, there once near | A10 |
| Kill credited 6.5-7 s after HP 0 | live: flag timelines A2, B8 (7 s), B10 (7 s) | A2, A6, B8, B10 |
| A warp 6.1 s after HP 0 loses the kill; no crash, no stale state | live | A6 |
| Death before the credit cancels, after it keeps the souls | live: six fights | B10 |
| `SpawnLimit` 255 gives live limit 0 (no limit), 0 gives 1 / 12, 1 gives 1 | live probe of the generator objects | B8 |
| Three wins in a row by param edit alone | live | B8 |
| The spawn rule (limit 0 / count < limit / two exemptions) | code, read 17 Nov | `FUN_140412060`, `FUN_1401f6760` |
| 131 generators unlimited, 44 boss rows work | live batches | B10-B19 |
| Clear the kill flag 1 s after it turns on | live: dash-out case | B9 |
| New group with its own EventParam row runs; nothing in the first state | live | B9 |
| Skeleton Lords' skeletons are `SpawnLimit` 1 and block a second fight | data; live (two runs) | B39 |
| Three Lords, one found by the search | live: two Lords had no re-arm | B12 |
| Elana, Looking Glass Knight, Covetous Demon adds are `SpawnLimit` 1 | data (scan) | B39 |
| A stale kill flag at load makes the boss stand down | live | B16, B38 |
| In-place reload gives the boss on the second load | live | B38 |
| A streamed-in neighbour map runs its scripts | live | B39 |
| The kill slot is not in memory before the map has loaded | live (4 Nov) | `tools/arenactl.py` |
| The count is not the kill flag (Ancient Dragon, count 1, no flag) | live | B39 |
| The Ascetic raises HP to the level-2 value | live | B56, A10 |
| The map-clear flag is shared by the area's bonfires | data | `MapObjectBonfireParam` |
| The travel menu follows the level byte, not the lit flag | live | B22 |
| After a win, the fog differs by arena | live, three cases | A2, B9, B39 |

## Tools

All in `tools/`:

| Tool | What it does here |
|---|---|
| `arena.py` | read-only helpers plus the re-arm: `kills <map> <gens>` (read counts), `rearm <map> <gen> <flags>` (count to 0, flags cleared), `flags <ids>` (read flags), `blocks` (loaded flag blocks) |
| `arenalib.py` | `kill_byte` finds a count's address; `reset_kills` writes 0 (used by the trainer) |
| `arenactl.py` | the trainer: F5 re-arms, warps and puts you at the fog |
| `slice_lastgiant.py` | the one-boss data-route test: `build`, `check`, `probe` (live +0x8D, kill counts, the exemption switch), `boss1hp` |
| `datapatch.py` | `param <param> <row>:<field>=<value>` writes loose param fields, keeping `.orig` |
| `bosstable.py` | finds each boss's generators (`ActivationEventID1` = fight flag); `EXTRA_GENS` for the cases it misses |
| `rollout_table.py` | the per-boss table; `EXTRA_UNLIMIT` lists the summoned adds |
| `buildmod.py` | writes 255 to every generator in a row's `unlimit` list |
| `arenamod.py` | builds the per-map arena group: stale clear, 1 s clear, 8 s warp, in-map wait |
| `stalekill.py` | read-only list of stale arrivals; `--set` / `--rearm` make one on purpose |
| `batchwatch.py` | live logger of fight flags, kill flags and kill counts per boss |

## Open questions

- **Do the counts survive a quit to the title and a reload of the save?** We saw them survive map reloads inside a
  session only. [untested]
- Which param holds the 1-or-12 default that `FUN_140412e70` reads for a 0 byte? (The code that fills +0x8D, and which
  of `SpawnLimit` / `SpawnLimitGameClear` applies at which cycle, are now read: see
  [NG+ differences](ng-plus-differences.md).) [code, partly]
- What is the object at `[generator+0x68]` whose byte +0x76 bypasses the limit? [untested]
- Does a Bonfire Ascetic re-arm a killed boss, and through which flag? [untested]
- Does clearing 100971 with the bonfire at level 1 drop anything? The planned check (B22, step 7) has no recorded
  result. [untested]
- The fog after a win, arena by arena, and after leaving the kill flag on. [untested]
- The automatic warp home firing while the bonfire menu is open: the guard's warp worked with the menu open
  [live B38], the win flow's own warp was not separately checked. [untested]
- Why the Dragonrider's count read 0 on the next Heide load. [live B11]

## Related pages

- [The respawn point](respawn-point.md): where a death or the Darksign sends you (it shares the map-byte order of the kill counter)
- [Moving a bonfire](moving-a-bonfire.md) and [Warping by script](warping-by-script.md): the arena arrival and the warp home
- [Removing enemies](removing-enemies.md): quieting and deleting what stands near an arena
- [Event script groups](../modding-scripts/event-script-groups.md): adding the win-flow group and its `EventParam` row
- [Editing params](../modding-scripts/editing-params.md) and [GeneratorParam fields](generatorparam-fields.md): loose params and the other fields
- [Event flags and choosing safe ids](../modding-scripts/event-flags-and-free-ids.md): which flags are readable when
- [Bonfire lit state](bonfire-lit-state.md): the lit flag, the level byte and the Ascetic
- [NG+ near boss arenas](ng-plus-differences.md)
