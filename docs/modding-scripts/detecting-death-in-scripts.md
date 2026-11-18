# Detecting a player death in a script (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- **The signal that works is the player's HP.** In an event script, `f130312()` returns the player's current HP. Count a death as `f130312() < 1`, but only after a reading above 0 has been seen in the same map load. HP goes **negative** when damage kills (-592, -394 and -41 in B50c; -218 in B57) and reads **exactly 0** after a kill-plane fall, so test `< 1`, not `< 0`. [live B50c, B57] [live 4 Nov, respawn watcher]
- **A death ends in a loading screen, and the one thing we could carry across it was a global event flag.** The arena script writes "died" into a saved flag; the next map's script reads it. That is how a script in Majula could tell "died in the arena" from "left with the Aged Feather". A script global variable did not do the job. [live B50c]
- **What failed:** the death count `f130308(0)` compared with a copy kept in script global variable 236 (`1:130122` / `f130122`). It sent the player back into the fight after an Aged Feather return (B50), and again when the comparison moved inside the arena (B50b). The B50c log shows the count comparison firing in the first log second after an arrival, with no death and no Feather. Do not use it as a death test. [live B50, B50b, B50c]
- **Ask first whether you need to detect anything.** A death respawns at the respawn record, so a retry point beside the arena (B44) or a real bonfire at the arena (B54, B55) removes the problem. The mod uses those now. The HP-based detection was retired on 10 Nov, and its code stays in `tools/specials.py`, unused. [live B44, B54, B55] [code]
- **The trap:** a death, the Aged Feather, a Homeward Bone and the Darksign all end in the same load of the same map, so seen from the destination they are identical. Only a flag set *before* the load can tell them apart. [live B50c]

Notation: `1:<id>(...)` is a script command (bank 1), `f<id>(...)` a script function, and `5:<id>(cond, ...)` runs command `<id>` only when `cond` is true (the conditional form; FromSoftware's own scripts use it for "set once" tests, and the mod's script writer emits it). [data]

## The problem

The Boss Arena sends the player straight to a fog gate. In almost every arena there is a bonfire, or a retry point set beside one, so a death puts the player back at the fog ([The respawn point](../modding-world/respawn-point.md), pattern C). Two fights had no bonfire of their own until 10 Nov: the Darklurker (Dark Chasm of Old, `m40_03`) and the Giant Lord (his Memory, `m20_10`).

A retry point inside either arena is a softlock. In B19 (5 Nov) the Darklurker's first build put one in the boss room. A death reloaded the room with the fight running, which was the point. But the next load went back into the room as well, and so did the Homeward Bone and the Darksign, and nothing but a win got the player out. [live B19] So both fights kept the respawn on Majula, and a death there went to Majula (B39 step 2, 7 Nov). [live B39]

Players doing very low-level runs may need a hundred attempts, and picking the fight from the Herald's list each time is a chore. B50 asked for two behaviours at once:

| What the player does in the arena | Wanted |
|---|---|
| Dies | Straight back into the fight, no menu |
| Uses the Aged Feather (or a Homeward Bone) | Home, and stay there |

Both end with the game loading Majula, because the respawn record is Majula's. So Majula's script has to know *which* one happened, and the only place that can see a death is the arena, before the load. That is the whole problem.

### The signal: the player's HP

| Call | Developer name (`.edd`) | English gloss | Role here |
|---|---|---|---|
| `f130312()` | 【ステータス】プレイヤーのHP取得 | "[status] get the player's HP" | the death signal that worked |
| `f130308(type)` | 【ステータス】プレイヤーの霊体タイプ別死亡回数を取得 | "[status] get the player's death count by spirit type" | the signal that failed (we only ever passed 0) |
| `1:130122(id, value)` | 【フラグ】グローバル変数設定 | "[flag] set global variable" | the store that failed (variable 236) |
| `f130122(id)` | 【フラグ】グローバル変数取得 | "[flag] get global variable" | its read |
| `1:130101(flag, value)` | 【フラグ】フラグを設定 | "[flag] set an event flag" | carries the death across the load |
| `f6(flag)` | フラグの取得 | "get an event flag" | reads it |
| `f3()` | ステート経過時間の取得 | "get the time elapsed in this state" | Majula's 2 s wait |
| `1:132141(location)` | 【マップ】リトライポイント設定 | "[map] set the retry point" | puts the respawn record on Majula's landing before the warp ([The respawn point](../modding-world/respawn-point.md)) |
| `1:132125(0, 0, map, location, 0)` | 【マップ】プレイヤーをポリ劇ワープ | "[map] warp the player (polygon-drama warp)" | the warp back into the arena ([Warping the player by script](../modding-world/warping-by-script.md)) |

`f130312()` is not the command `1:130312` (set HP by ratio: it has code in the exe but has never been called; see [Script command reference](script-command-reference.md)). Commands and functions have separate id banks, so the same number can mean two things. [data] [code]

What we know about `f130312()`:

- It returned a value above 0 for a living player: on the first arrival the HP_SEEN flag came on in the first log second, and at every later arrival it was on before the death (a death needs it). [live B50c]
- When the player died to damage, the flag the script set on `f130312() < 1` came on in the same 0.25 s log sample in which the read-only watcher saw HP go negative: -592 (Darklurker), -394 (Darklurker, second death) and -41 (Giant Lord). [live B50c] A cheat-table test also read 282 -> -218 on a death. [live B57 step 9]
- A fall off a ledge, or any kill plane, leaves HP at **exactly 0**, not below it. The 4 Nov watcher saw that 0.9 s after a fall in Majula, and again on a fall death off Majula's cliff. [live 4 Nov] We never ran the script's `< 1` test on a fall death, only on damage deaths. [untested]

The player's HP lives at PlayerCtrl +0x168 (s32); the watcher reads it from there with `tools/arenalib.py` (`hp()`). [live B50c]

### Carrying the death across the load: saved flags

Global event flags (block 10, ids 100000-109999) are saved and can be read from any map, unlike a map's own flags. The arena script sets them and Majula's script reads them 14 s later. [live B50c] [live A1] The working design needs three of the mod's flags (ARM, HP_SEEN, DIED); the rest belong to the failed designs and the log. Ids come from the mod's free range in that block, registered and frozen in `tools/ids.py`:

| Flag | Name in code | Set by | Cleared by | Meaning |
|---|---|---|---|---|
| 104785 / 104786 | ARM (Giant Lord / Darklurker) | the arena's load state, only on a fresh arrival from the fight list with no boss rush running | Majula's check, every branch | "this player came by the list, so a death here means retry" |
| 104789 | HP_SEEN | the arena's `seen` state | the arena's load state | HP above 0 was read in this load |
| 104787 | DIED | the arena's `died` state | Majula's check; the arena's load state | the death, carried to Majula |
| 104788 | COUNT_OK | the arena's load state | the arena's load state | the stored count read back equal (the B50b design only) |
| 104790 / 104791 | MARK_COUNT / MARK_HP0 | marker group 990007 | the arena's load state | diagnostics: what the count signal and the HP signal saw |

See [Event flags and free ids](event-flags-and-free-ids.md) for choosing ids and why ones that live in saves are never reused.

### The two scripts

Both maps' scripts are our own event groups, each with its own EventParam row ([Event script groups](event-script-groups.md)). The special's arena group is 990000 in `m20_10` and `m40_03`; Majula's check is group 990014.

**The arena group (the "fight" part of its win flow, B50c):**

| State | Leaves when | Entry commands |
|---|---|---|
| `load` | at once: to `stale` if the boss's kill flag is already on (the stale-boss reload: [Bringing a boss back](../modding-world/respawning-bosses.md)), else to `fight` | arm the row's ARM flag (conditional: fresh list arrival, no run); clear DIED, COUNT_OK, HP_SEEN and both markers |
| `fight` | the kill flag comes on -> `kill` (the normal win flow); `seen`; `died` | none |
| `seen` | back to `fight` | HP_SEEN = 1 |
| `died` | the kill flag comes on -> `kill` | DIED = 1 |

The two conditions on `fight`:

- `seen`: `ARM on && f6(HP_SEEN) == 0 && f130312() > 0`
- `died`: `ARM on && HP_SEEN on && f130312() < 1`

The HP_SEEN gate means a reading that never works (0 before the player object exists, say) cannot pass for a death. In the live runs it came on in the first second after arrival and never blocked a real death, so we never saw it needed. [inference]

**Majula's group 990014:** two seconds after every load (`f3() > 2`, RETRY_WAIT), it makes one decision.

| Condition | Action |
|---|---|
| ARM on, DIED on, no boss rush running (flag 104700 off) | clear the ARM flags and DIED; `1:132141(200000)` (respawn record on Majula's landing); then the fight list's own warp into the arena (`1:132125`) |
| anything else | clear the ARM flags and DIED |

Every outcome clears the flags, so a Feather, a Bone, a Darksign, a win or a quit leaves nothing armed. A boss rush never retries: a death ends the run. The respawn is put back on Majula before the warp, so a second death or a Feather still lands in Majula. [code `tools/specials.py`]

### What the B50c log showed

`tools/retrywatch.py` (read-only, 0.25 s samples) logged the clean-install test copy from 19:32 to 19:40 on 8 Nov. [live B50c]

| Time | What happened | Log |
|---|---|---|
| 19:33:48 | List -> Darklurker, the chasm loads | `where chasm` |
| 19:33:49 | one second later, no death, no Feather | `ARM_DL 1, COUNT_OK 1, HP_SEEN 1, MARK_COUNT 1` |
| 19:33:59 | **Aged Feather** | Majula 19:34:00; `ARM_DL 0` at 19:34:02 (Majula's 2 s check); stayed home |
| 19:34:28 | List -> Darklurker again | `ARM_DL 1` |
| 19:34:52 | **Death** | `hp -592, DIED 1, MARK_HP0 1` |
| 19:35:06 | Majula loads, 14 s after HP went below 1 | `ARM_DL 0, DIED 0` at 19:35:08; the warp starts |
| 19:35:12 | back in the chasm | `where chasm`, `ARM_DL 1` |
| 19:37:45 | **Second death in a row** (HP -394) | `DIED 1`; Majula 19:38:00; back in 19:38:06 |
| 19:38:17 | **Aged Feather** after that retry | Majula; `ARM_DL 0` at 19:38:20; stayed |
| 19:38:45 | List -> Giant Lord (the Memory of Jeigh, `m20_10`) | `ARM_GL 1` |
| 19:39:13 | **Death** (HP -41) | `DIED 1`; Majula 19:39:27; the memory 19:39:33 |
| 19:40:06 | **Aged Feather** | Majula; `ARM_GL 0` at 19:40:09; stayed |

From HP below 1 to being back in the arena took about 20 s (19:34:52 -> 19:35:12): 13-14 s to the Majula load, 2 s of waiting, then the warp. The flag had a 13 s window in which to be set before the respawn load unloaded the map, and it was set in the same sample as the HP change every time. [live B50c]

## How to do it

This is the recipe the mod ran from 8 to 10 Nov. Use it when you cannot give the arena a bonfire or a retry point, for example a fight whose respawn must stay in a hub.

1. **Pick saved event flags** for ARM, HP_SEEN and DIED, outside every id the game or your other groups use ([Event flags and free ids](event-flags-and-free-ids.md)). They must be saved flags, because they have to survive the respawn load.
2. **Arena map, load group.** On a fresh arrival that your own warp made (the mod keeps an arrival token for this; see [Warping the player by script](../modding-world/warping-by-script.md)), set ARM. At every load of the arena, clear HP_SEEN and DIED.
3. **Arena map, fight state.** Add the `seen` and `died` states above to the group that already waits for the boss's kill flag. List the kill flag's condition first, as the mod does, so the win flow takes any tie. [inference]
4. **Hub map, one group.** `f3() > 2` after every load: if ARM and DIED are on and no run is active, clear them, put the respawn record on the hub (`1:132141`), and warp back in; otherwise clear them.
5. **Give each group an EventParam row** and a free group id, or it will not run ([Event script groups](event-script-groups.md)). The groups load with their map, so no restart is needed.
6. **Test with a watcher, not by eye.** Log the map, HP, the respawn record and the three flags every 0.25 s from outside the game, as `tools/retrywatch.py` does. The checks to run:
   - arrive, use the Feather at once: you go home and stay;
   - arrive, die: back in, with no menu;
   - die twice in a row: back in both times;
   - die, come back in, then use the Feather: home, stay;
   - the same at the second arena.

## Without detecting anything: the respawn record and bonfires

Every other fight in the mod needs no death test. The game already sends a death, the Darksign, a Bone and the Feather to the respawn record, so the question is only what the record says when the player is in the arena.

| Technique | What the death does | Where it is used | Test |
|---|---|---|---|
| Set a retry point (`1:132141`) at an arrival point beside the arena bonfire, once the player is in the map | back to the arrival point in one load | every ordinary arena since B44 | [live B44, B54, B55] |
| Give the arena a real bonfire at the arrival spot | back to the bonfire; the Bone, Darksign and Feather land there too | the Darklurker (bonfire 3650) and the Giant Lord (10685) since 10 Nov | [live B54, B55] |
| Keep the respawn on the hub and let the *next load* give it away | the hub loads; a guard sees "a load that is not one of my arrivals" | the boss rush, which wants a death (or a rest, a quit) to end the run | [live B34, B38, B39] |
| Read HP and save a flag (this page) | the hub script retries | the two specials, 8-10 Nov | [live B50c] |

The third row is a death test in everything but name: nothing reads HP, but the run guard turns "the respawn load happened" into the signal. It works when you want a death to *end* something. It cannot separate a death from a Feather, which is why the specials needed the HP test. [live B38, B39]

### B50: the stored death count (8 Nov)

The design: when the fight list sent the player to a special, the arena stored the death count `f130308(0)` in global variable 236 (`1:130122`) and armed the row. Majula, 2 s into its load, compared a fresh `f130308(0)` with `f130122(236)`. A count that had grown meant a death. The id 236 was picked because no FromSoftware event or talk script names it. [code] [data, scanned 8 Nov]

Live result: **FAIL at step 4.** The Feather took the player to Majula, and Majula then warped them straight back into the fight. So Majula saw "count above the stored value" with no death. The note written then said script global variables most likely do not survive a map load, with "236 read 0 in Majula" as the evidence. We found no record of the variable being read directly (the read-only watcher came later, with B50c, and reads flags, not variables), so it stays an inference. [live B50] [inference]

### B50b: the comparison moved inside the arena (8 Nov)

If variables do not cross a load, compare inside the same load. B50b made the arena store the count at every armed load, set COUNT_OK when it read back equal, and set DIED when `f130308(0)` rose above the stored value while the fight ran. Majula retried only on ARM and DIED.

Live result: **FAIL again.** The same symptom on a Feather return. [live B50b]

### B50c: the HP signal, and the correction (8 Nov)

The first guess for B50b's failure was that the death count itself rises on a Feather or Bone return (the return runs the respawn path). B50c therefore dropped the count and used HP, with HP_SEEN, and kept the count comparison as a marker (MARK_COUNT) beside the new one (MARK_HP0) so a log could show which signal saw what. B50c **passed**, and the log then overturned the guess: MARK_COUNT came on at 19:33:49, one second after the first arrival and in the same sample as COUNT_OK, with no death and no Feather. [live B50c]

So the B50 and B50b loops came from the stored-count comparison itself, not from the Feather raising the count. The guess that the count rises on a Feather return is **withdrawn**. Do not use `f130308(0)` against a stored global variable as a death test. [live B50c]

What the log does and does not settle:

- **Settled:** inside one load, within about a second of a store that read back equal (COUNT_OK on), `f130308(0) > f130122(236)` became true with no death. MARK_COUNT was never seen off again at later arrivals. [live B50c]
- **Not settled:** which side moved. Either the variable no longer held the stored count a moment after the store (cleared, or scoped so that another group reads something else: MARK_COUNT lived in group 990007, a different group from the one that stored it), or `f130308(0)` returned a different number a moment later. The script cannot tell; nothing logged the variable's value. [inference]
- The B50 explanation (variables lost at a map load) is not ruled out, but it is not what broke B50b, because B50b compared in the same group and the same load. [inference]

### Edge cases for an HP test

- **Use `< 1`.** Damage deaths go negative (overkill), a kill plane leaves exactly 0. [live B50c, B57] [live 4 Nov]
- **A trainer that locks HP at 1 defeats it.** The project's cheat table has a "Can't die" entry that holds the minimum HP at 1; with it on there is no death and no retry. [live B57 step 9] [inference for the retry]
- **A death in a boss rush must not retry.** Majula's check reads the run flag (104700). [code]

### A retry through Majula costs two loads (10 Nov)

Death -> Majula -> arena is two loading screens and about 20 s. A retry point or a bonfire at the arena is one load. The project's 10 Nov design note for the new bonfires lists their gains over this retry as a travel-menu entry, resting at the boss, and a one-load retry. [live B50c] [design note, 10 Nov]

## Why new bonfires replaced it

On 10 Nov the Darklurker (B2, then B54) and the Giant Lord (B55) each got a real bonfire placed by data ([Adding a bonfire](../modding-world/adding-a-bonfire.md)). The fight list now lands the player beside it, and the arena map's guard group sets the respawn there (`1:132141` at the new arrival point), exactly as for every other arena. A death returns the player beside the bonfire in one load, and the Aged Feather takes them to the same place. Nothing is detected, and nothing can loop. [live B2 steps 2-4 (death and Feather both returned to bonfire 3650)] [live B54 (death: back beside the bonfire, one load; reported as a pass)] [live B55 (reported as a pass; the steps played were not listed)]

What was left behind:

- `RETRY_ARM` in `tools/specials.py` is empty, because both specials' arrival rows are now bonfire rows. The Majula check group (990014) and the marker group (990007) are no longer built, and the arena groups carry no arming or HP states. [code] An offline test asserts this. [code]
- The flag ids 104785-104791 stay registered and frozen (they may sit in saves; never reuse them), and no FromSoftware script names 104785-104791 or variable 236. [code] [data]
- The functions `arm_cmds`, `died_watch`, `marks_spec` and `retry_spec` remain in `tools/specials.py`, unused. `tools/retrywatch.py` no longer imports (it reads `RETRY_ARM[...]`, now empty). [code, checked 17 Nov]

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| `f130312() < 1` after HP > 0 is a working death test (3 damage deaths, 2 arenas) | read-only log, 0.25 s samples | [live B50c] |
| A flag set in the arena is read by Majula's script after the respawn load | log: DIED on before the load, retry after it | [live B50c] |
| Three Feather returns set no DIED and did not retry | log | [live B50c] |
| HP goes negative on a damage death | log, cheat-table test | [live B50c] (-592, -394, -41), [live B57 step 9] (-218) |
| HP is exactly 0 after a kill-plane fall | watcher | [live 4 Nov] |
| The count-vs-variable-236 comparison looped on a Feather return | player report, twice | [live B50, B50b] |
| The count comparison fired a second after arrival, with no death and no Feather | log | [live B50c] |
| "Variable 236 read 0 in Majula" | no direct read | [inference B50] |
| "The count rises on a Feather return" | withdrawn | [inference B50b, withdrawn B50c] |
| The load follows 13-14 s after HP goes below 1; back in about 20 s | log timestamps | [live B50c] |
| A retry point beside the arena retries an ordinary fight | player report | [live B44] |
| A retry point inside the Darklurker's room traps the player | player report | [live B19] |
| Both specials went to Majula on death before B50 | watcher | [live B39 step 2] |
| A death and the Feather both return to the Darklurker's new bonfire | player report, record read back | [live B2 steps 2-4] |
| His list trip lands at the bonfire and a death comes back beside it in one load | player report ("all seemed to work") | [live B54] |
| The Giant Lord's new bonfire passed (the steps played were not listed) | player report | [live B55] |
| The boss rush ends a run on a load that is not an arrival | live | [live B34, B38, B39] |
| Global flags (block 10) are readable from any map; a map's own flags are not | live flag reads | [live A1] |
| The names and ids of `f130312`, `f130308`, `f130122`, `f130303`, `f130302`, `f130321` | `.edd` name tables of the PC event and talk scripts | [data] |
| FromSoftware's 28 event and 25 talk scripts never call `f130312`; one call to `f130308` (the Majula talk script, compared with 100) | script scan of the unmodified PC scripts | [data, scanned 17 Nov] |
| No FromSoftware script names variable 236 or flags 104785-104791 | script scan, offline test | [data, 8 Nov] |
| The retry is retired: `RETRY_ARM` is empty, 990014 and 990007 are not built | read the module; offline test | [code, checked 17 Nov] |

## Tools

| Tool | What it does for this topic |
|---|---|
| `tools/specials.py` | the special rows' scripts: the retired retry code (`arm_cmds`, `died_watch`, `marks_spec`, `retry_spec`) and the constants (flags 104785-104791, variable 236, `RETRY_WAIT`, groups 990007 and 990014) |
| `tools/retrywatch.py` | read-only watcher for the B50c retest: map, HP (PlayerCtrl +0x168), respawn record and the flags every 0.25 s into `tools/retrywatch-<date>.log`; no longer imports |
| `tools/retrywatch-2026-11-08.log` | the B50c log quoted above |
| `tools/respawnwatch.py` | read-only watcher that also logs HP crossing 0 (the 4 Nov kill-plane reads) |
| `tools/arenalib.py` | `hp()`, `flag()`, `respawn()`: the reads the watchers use |
| `tools/bossrush.py` | the arrival token, `guard_spec` (the "a load that is not an arrival" test) and the fight list's warp |
| `tools/ids.py` | the id register: why 104785-104791 stay reserved |
| `tools/esd.py`, `tools/edd.py` | the project's script readers: dump a map's groups and read the developer names |

## Open questions

- **Why did the count comparison fire within a second of the store?** Settling it needs a probe that copies `f130122(236)` and `f130308(0)` into event flags (a few bits at a time) once a second and a watcher that reads them. [untested]
- **Do the other variable kinds hold a value across a load?** The name table lists four kinds: area (`130121`), global (`130122`), character-local (`130125`) and local (`130127`). Only the global kind was tried. [untested]
- **Other death-related functions in the name table were never tried:** `f130303` 【ステータス】死亡判定 ("[status] death judgement"), `f130302` HP割合取得 ("get the HP ratio") and `f130321` プレイヤーを殺した ("killed the player"). FromSoftware's event scripts call none of them. Its talk scripts call `f130301` (HP), `f130302` and `f130321`, but we did not check which character they read. [data, scanned 17 Nov] [untested]
- **What does the argument of `f130308` select?** We passed 0, as the Majula talk script does. Other values are untested. [untested]
- **A fall death.** We know a kill plane leaves HP at 0, but the retry was only run on damage deaths. [untested]
- **A quit or crash between the death and the respawn load.** DIED would be set; the next arena load clears it, but Majula would read it first if the character loaded there. [untested]
- **A Feather used after taking damage.** All three Feather returns in the B50c log came at full HP (718), 11 to 34 s after arrival. A Feather used mid-fight, or while the HP is low, was not logged. [untested]

## Related pages

- [The respawn point](../modding-world/respawn-point.md): the record every death and Feather reads, and the B19 softlock behind this page
- [Warping the player by script](../modding-world/warping-by-script.md): `1:132125`, arrival points and the arrival token
- [Event script groups](event-script-groups.md): groups, EventParam rows, and why state across maps is flags
- [Event and talk script commands](script-command-reference.md): the command and function table
- [Event flags and free ids](event-flags-and-free-ids.md): choosing saved flags, frozen ids
- [Bringing a boss back](../modding-world/respawning-bosses.md): the kill flag and the stale-boss reload the arena group also runs
- [Adding a bonfire](../modding-world/adding-a-bonfire.md): the answer that replaced this page's recipe
- [Moving a bonfire](../modding-world/moving-a-bonfire.md): landing spots
- [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md): restoring HP by script, and the untried "set HP" commands
- [Giving the player items](../modding-menus/giving-items.md): how the Aged Feather reaches the player
- [Memory map for PC 1.0.3.0](../tools/memory-map-pc.md): the player's HP offset
- [Live-testing tricks and pitfalls](live-testing-notes.md): read-only watchers
