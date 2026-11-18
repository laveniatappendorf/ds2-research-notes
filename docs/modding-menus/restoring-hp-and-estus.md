# Restoring HP, Estus, spells and durability by script (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- **Use effect 110000010.** `1:130334(110000010, 19, 4)` from an event script gives the player the bonfire rest's recovery: HP to full (up to the current hollow cap), the Estus Flask refilled, spell uses refilled, statuses cleared. [live B56 step 2, B57 step 5; spells and statuses from the effect's contents, data]
- **No command refills Estus or repairs gear.** None of the 325 script commands does it [data]. Four "set HP" commands, 130311 to 130314, do have real code in the exe (they write HP and minimum HP), but the original scripts never use them, we never called them, and which character they act on is unknown [code, 17 Nov; untested]. A player effect is the only route we have proven.
- Three more effects are useful: 100000020 (HP, spells, repair all, un-hollow, **no Estus**), 60410000 (Repair Powder: repair only) and 100000010 (the Shrine of Amana's return to human). [live B56, B21]
- **Traps.** A script-triggered effect with duration 0 acts **once**, so it cannot be a lasting buff or handicap [live B56 step 4]. HP is refilled only up to the **hollow cap**: 110000010 does not lift it [live B57 logs]. A bonfire rest already restores everything, so test with a quit and load, not a bonfire trip [live B57].
- **Timing.** Wait until the player is inside the map, wait 1 s more, then fire. In our tests the refill landed 1.3 to 1.5 s after the player appeared, at each load and rush arrival we logged [live B57 step 5].
- **No proven "can't die" by script.** `1:130303(1)` stops all damage, but also all hit reactions, and a load resets it [live B56 step 5]. `1:130313` (set minimum HP) writes the same HP floor the cheat table uses, but has never been tried [code; untested].

### Player effects are EMEVD events

Dark Souls II has no SpEffect param table in the usual param tools. The game's effects are events in the `SpEffect*.emevd` files of the regulation, one event per effect, and **the event id is the effect id** [data]. An item names its effect in `ItemParam` (`speffect_id`). For goods the effect id is the item's own id: Repair Powder's effect is 60410000 and the Human Effigy's is 60151000 [data]. Effect events use a small instruction set of their own (banks 1000 and 100040 to 100130). That is not the script command set below.

An event script (the `.esd` files that hold event and talk logic) starts an effect on the player with:

| Command | Developer name | Gloss | Use |
|---|---|---|---|
| `1:130334(effect, 19, 4)` | 【ステータス】プレイヤーの特殊効果を発動 | trigger a special effect on the player | the whole page |
| `1:130336(effect)` | 【ステータス】プレイヤーの特殊効果を解除 | cancel a special effect on the player | never used by the original scripts; see Open questions |
| `1:130333(chr, effect, 19, 4)` / `1:130335(chr, effect)` | 【ステータス】指定の敵の特殊効果を発動 / 解除 | the same two for a chosen enemy | seen in the original scripts, not tested by us |
| `1:140182` | 【グループ条件：キャラ】プレイヤーの特殊効果判定 | group condition: does the player have an effect | seen in the original scripts, not tested |

The original scripts use the player form (`1:130334`) three times, always with `19, 4` after the effect id [data]. We copied the two constants. What they select is not decoded.

| Original use | Effect |
|---|---|
| Shrine of Amana (`event_m20_11`), "return to human" | 100000010 |
| Things Betwixt (`event_m10_02`), run right after character creation | 100000020 |
| The end credits (`event_m20_21`) | 150300000 |

### The effects that restore things

All five effects below were triggered from a script in the live tests. "Contents" lists the effect-event commands each one holds [data, read 17 Nov]. The meaning of a command is read from where else it appears (herbs, waters, the Repair spell) and from what the live test showed.

| Effect (file) | What it is | Contents (command ids) | What the tests showed |
|---|---|---|---|
| **110000010** (`SpEffectWideUse`) | the bonfire rest's recovery [inference, see below] | `100050:1` set HP to 100 %; `100060:0` spell uses to 100 %; `100070:0` cure status states (12 entries); `100110:2` repair; `100090:8` unknown; seven `1000:1` stat entries (stats 0 to 6, 3 s, x100), meaning unread. **No `100050:2`.** | HP full to the current cap, **Estus refilled** [live B56.2, B57.5]. Cap not lifted [live B57]. |
| **100000020** (`SpEffectWideUse`) | run after character creation | `100050:1`; `100060:0` (twice); `100110:2`; `100070:0` (13 entries); `100050:2` with argument 0 | HP, spell uses, durability, hollowing gone. **Estus not refilled** [live B56.1]. |
| **60410000** (`SpEffectActiveItem`) | Repair Powder | three `1000:9` repair entries | durability up [live B56.3] |
| **100000010** (`SpEffectWideUse`) | Shrine of Amana, return to human | two `100040:1`; `100050:2` with argument 1 | the player turns human again, with a "humanity restored" notice [live B21] |
| 21502100 (`SpEffectArmor`) | Black Witch Domino Mask (max HP x0.9) | `100130:2`; `1000:1` stat 7 x0.9 with **duration 0**; five more `1000:1` | a **one-off** cut of current HP x0.9, no lasting change [live B56.4] |

What each effect restores, as measured:

| | HP | Estus | Spell uses | Durability | Hollow cap |
|---|---|---|---|---|---|
| 110000010 | to the cap [live B57] | **yes** [live B56.2, B57] | in its contents [data]; not watched | in its contents [data]; not isolated [inference] | unchanged [live B57] |
| 100000020 | to the **old** cap [live B56.1] | no [live B56.1] | yes, HUD [live B56.1] | yes, weapons [live B56.1] | **lifted** [live B56.1] |
| 60410000 | no | no | no | yes [live B56.3] | not recorded |
| 100000010 | no `100050:1` in it [data] | not measured | not in it [data] | not in it [data] | "human again" [live B21]; cap not logged |

**Why we call 110000010 "the rest's effect".** The exe applies it from one small function, `FUN_1402027b0` (RVA `+0x2027b0`), which loads the id as the immediate `0x68e778a` [code]. A byte scan for direct calls finds three call sites: `0x14017f6df`, `0x1402b8f34` and `0x1404199be` [code, 17 Nov]. The first lies in the bonfire manager's code, near the functions that read and write the bonfire list (`FUN_14017daa0`, `FUN_14017ec50`) [code]. An earlier Ghidra note gave `0x14017f639` for that call. The two addresses are 0xA6 apart and we have not reconciled them. A real rest also behaves the same way: with both options off, a refill in B57 took HP to the current cap (2274, not the full 2394), refilled Estus and repaired the Rapier (log 15:59:43; we take it to be a bonfire rest because both options were off). So the call site and the behaviour both point at the rest [inference]. We never saw the rest call the effect.

### Where the Estus count lives

The Estus Flask is item 60155000 in the bag [live]. Its bag entry is 16 bytes. The numbers sit in the eight bytes from entry +8 [live B56 step 2, B57]:

| Entry offset | Value (our test save) |
|---|---|
| +0 | item id 60155000 (u32) |
| +8 | a u16 that read 1 on a full flask, probably the stack count [inference]. It is **not** the number of sips: it was the first place we looked, and it never changed. |
| +0xA | sips left (u8) |
| +0xB | sips at most (u8) |

The refill took +0xA from 2 to 4 (B56 step 2) and from 1 to 4 (B57); +0xB read 4 throughout. The HUD agreed. Find the entry by scanning the bag for the item id. The bag is `[[[GameManagerImp] + 0xA8] + 0x10]` (`[GameManagerImp]` is the object the static pointer points to; the same notation is used below); the first list starts at bag +0xF0 with 16-byte entries and 3840 slots [code, `tools/arenalib.py`]. The flask was slot 13 in our saves, but the slot is not fixed. More on the bag layout: [Memory map for PC](../tools/memory-map-pc.md).

### Where and when to fire the effect

The mod fires the effect from a small event group in every arrival map (22 maps, group 990006). Each group has an EventParam row copied from one of FromSoftware's load-time rows, so the group starts at every load of the map. See [Event script groups](../modding-scripts/event-script-groups.md).

The group waits for the player to be **inside** the map before it acts. It registers `1:140402(0, 1, 0)` (【グループ条件：マップ】プレイヤーがマップ内にいる判定, "player is inside the map check"), then waits for `f140001(0)` (【グループ条件】グループ条件を満たす, "the group condition is met"). A map that only streams in beside the player runs its scripts too. In B39 a neighbouring arena streamed in during a rush and its run guard ended the run, which is why every arena group now waits for the player to be in its own map [live B39]. The restore group follows the same rule, because without it a neighbour's group could heal the player from next door [inference: the restore group was built with the wait from the start, so we never saw it fail]. We never tried firing earlier [untested].

After that it waits 1 s (`f3() > 1`, `f3` = ステート経過時間の取得, "time in this state") and fires. The 1 s is slack we chose. We did not measure the shortest delay that works [untested].

Logged timings in B57 (the watcher polls every 100 ms; "appears" = its first read of a player after the load):

| Load | Player appears | Refill seen | Gap |
|---|---|---|---|
| Heide, quit and continue | 91.9 s | 93.3 s (HP 1596 to 2154, Estus 1/4 to 4/4 in one tick) | 1.4 s |
| Rush arrival, m20_21 | 285.9 s | 287.2 s (HP 73 to 2154, Rapier 23.5 to 45) | 1.3 s |
| Rush arrival, m10_31 | 359.5 s | 360.8 s | 1.3 s |
| Rush arrival, m50_35 | 407.5 s | 408.9 s | 1.4 s |
| m50_36, after a death | 194.9 s | 196.4 s (HP 1 to 2035, Rapier 43 to 45) | 1.5 s |

The project's own notes give 1.3 to 2.0 s over all runs. These five samples sit at 1.3 to 1.5 s. The refill is instant: HP and Estus jump in a single sample. The 3 s entries inside 110000010 produced nothing we could see.

### The hollow cap

At every load the HP cap reads the full maximum for the first few tenths of a second. About 0.4 s after the player appears it drops to the value the save holds, and HP is clamped to it [live B57 logs; this is the game's own load step, it also happens with both options off]. In B57 (max HP 2394) the cap read 2274, 2154, 2035 and 1915 after successive deaths: 5 % of max HP per death [live B57 logs]. The refill then **fills to the cap and leaves the cap alone**: HP 1596 went to 2154 with the cap at 2154 [live B57]. A rest does the same (above). Only effects that hold the `100050:2` command lift the cap [live B56.1 for 100000020].

Two consequences:

- Effect 110000010 restores "as at a bonfire", not "as a new character". Hollowed players stay hollowed.
- 100000020 fills HP **first** and lifts the cap **after**, so the player ends below the new maximum: HP 2154 of 2394 in B56 step 1 [live]. A following 110000010 took HP to 2393 (B56 step 2). For "fully fresh", apply 100000020 and then 110000010 [inference from those two steps; the pair was not run back to back in one script].

### Other player commands we tested or read

| Command | Developer name | What we found |
|---|---|---|
| `1:130303(1 / 0)` | 【ステータス】プレイヤー無敵設定, "player invincibility setting" | **No damage and no hit reactions.** Reset by a load. The original scripts use it 28 times around cutscenes and scripted moments. [live B56.5, data] |
| `1:130311` to `130314` | HPを直接設定 / HPを割合で設定 / 最低HPを直接設定 / 最低HPを割合で設定, "set HP / HP by ratio / minimum HP / minimum HP by ratio" | Never used by the original scripts and never called by us. **Implemented in the exe**, in an earlier handler than the one first read (see below): 130311 / 130312 write HP, 130313 / 130314 the minimum HP, clamped to the maximum. Which character they act on from an event script is unknown. [code, 17 Nov; untested] See [Script command reference](../modding-scripts/script-command-reference.md). |
| `1:130317(generator, ratio)` | 【ステータス】ジェネレータ指定でHP割合を設定, "set HP ratio by generator" | Used once by the original scripts, on Freja's generator (803), so it reaches a boss. Not tested by us. [data] |
| `1:130381(covenant)` | 【ステータス】プレイヤーの誓約を変更, "change the player's covenant" | `130381(8)` joins the Company of Champions: enemy damage x1.5 exactly, no enemy HP change. `130381(0)` leaves. [live B56.6]. See [Difficulty levers](../modding-world/difficulty-levers.md). |
| `1:130307` | 【ステータス】プレイヤーの落下ダメージを一度だけ無効化, "nullify the player's fall damage once" | Seen in the command list. Not tested. [data] |
| `1:131421` / `1:131401` | エスト瓶強化 / アイテム消費, "strengthen Estus Flask" / "consume item" | The only commands that name Estus or items in this area. Neither refills the flask. [data] |

For the exe check on 130311 to 130314: the B56 probe read the handler `FUN_140461f20` (jump table at RVA `+0x46db80`, byte map at `+0x46dc34`, indexed from command 130307). There, indexes 4 to 9 (commands 130311 to 130316) point at the default entry, which runs `FUN_1404910b0` and then a stub, so B56 concluded "not implemented" [code, B56]. That handler is not the first in the chain. An earlier one, with its byte map at RVA `+0x458b28` indexed from command 130125, has its own cases for 130311 to 130314 (targets `+0x457ff9`, `+0x45808c`, `+0x45813d`, `+0x4581d6`), and only falls back to the handler above for other ids [code, 17 Nov, byte map and targets re-read]. So the commands exist; a live probe would show whom they affect. [untested]

## How to do it

1. **Choose the effect.**

   | You want | Use |
   |---|---|
   | The player refilled like a rest (HP, Estus, spells) | `110000010` |
   | Only gear repaired | `60410000` |
   | A hollowed player made human, with HP, spells and repair | `100000020`, then `110000010` for HP and Estus (the pair was not run together [untested]) |
   | Estus alone | no such effect found: use `110000010` |

2. **Write one event group for each map where you want it.** Our Restore and Repair group (`options.arena_spec()` in `tools/options.py`) looks like this. Conditions are tested in the order listed.

   | State | On entry | Moves on when |
   |---|---|---|
   | s0 | nothing | at once to `load` |
   | load | `1:140402(0, 1, 0)` | `f140001(0)` to `wait` |
   | wait | nothing | `f3() > 1` and the Restore flag is on, to `restore`; else `f3() > 1` to `repair_q` |
   | restore | `1:130334(110000010, 19, 4)` | at once to `repair_q` |
   | repair_q | nothing | the Repair flag is on, to `repair`; else to `done` |
   | repair | `1:130334(60410000, 19, 4)` | nothing |
   | done | nothing | nothing |

   `f6(flag)` (フラグの取得, "get flag") reads the option flags. Both flags are 0 in a new save, so both options start off (Restore 104962, Repair 104963).

3. **Give the group an EventParam row** so it starts at every load, then build and install. How: [Event script groups](../modding-scripts/event-script-groups.md).

4. **Use free flags for any option or test switch.** Our option flags are 104960 to 104968; the probe flags in step 7 are 104950 to 104958. How to pick safe ids: [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md).

5. **Test with a quit and load, not a bonfire trip.** A rest refills and repairs by itself, so a bonfire trip proves nothing [live B57]. Put the player low on HP, with Estus drunk and a weapon worn down, quit to the title and continue. Read HP, the cap, the flask bytes and weapon durability about 2 s after the player appears.

6. **Read the numbers, not only the HUD.** HP, max HP and the cap sit at +0x168, +0x170 and +0x174 of the player's `PlayerCtrl` (`[[GameManagerImp] + 0xD0]`); stamina at +0x1AC and +0x1B4 [live reads in `tools/b56.py`; see [Memory map for PC](../tools/memory-map-pc.md)]. Weapon durability is a float at entry +8 of the weapon's bag entry. The Estus bytes are above. Sample at 100 ms and log changes. Short durability drops (0.5) need logging against the last logged value, or the watcher hides them (the first B57 step 6 try was inconclusive for this reason).

7. **To test a new command, use a probe group.** This is the B56 pattern, and it answered every open question about these commands in one half-hour session (10 Nov, 19:00 to 19:31):
   - One throwaway event group, in the map you will stand in.
   - It waits for the player to be in the map, then loops: wait for a step flag, clear it, run **one** command, go back to waiting.
   - A tool outside the game writes the step flag (a free event flag) and waits for the group to clear it. Every step ran within 0.05 s of the flag being set.
   - The same tool prints HP, max HP, cap, stamina, covenant, the Estus bytes and durability before the step and 4 s after, and what changed.
   - Keep the probe out of any release build. Our packaging scripts refuse to build while it is on.

## Pitfalls and what failed

1. **A duration-0 effect acts once from a script (B56 step 4).** The Domino Mask's effect (max HP x0.9, duration 0, which means "while worn" on armour and rings) took HP from 2393 to 2153 once. Nothing stayed lower: one Estus later HP read 2393 again, and max HP showed no change anywhere. A handicap or buff therefore needs an effect with a real duration, re-applied at each load. That needs new effect events, and we only read those files [untested]. The cancel test was impossible: nothing was left to cancel.

2. **The post-creation effect does not refill Estus (B56 step 1).** 100000020 restored HP, spells and durability and lifted the cap, but the flask stayed at 2 of 4 on the HUD. It looked like the obvious "full restore" before the probe. The Estus refill is in 110000010 only.

3. **100000020 leaves HP under the new maximum (B56 step 1).** HP went to 2154, the old cap, then the cap rose to 2394. See above for the fix.

4. **The rest's effect does not un-hollow (B57 logs).** Several deaths in a row left HP capped at 2154, 2035 and 1915 even with Restore on. This is correct for a rest. It surprises anyone who expects "full HP" to mean max HP.

5. **No proven script "can't die" (B56 step 5).** `1:130303(1)` kept HP flat at 2393 through the Last Giant fight, but the tester reported they could simply stand there and attack: no hit reactions at all. A load reset it: damage got through after the warp. The four set-HP commands have code in the exe, and 130313 writes the same HP floor as below, but they have never been called and their target is unknown [code, 17 Nov; untested]. The only working "can't die" we found is a memory write, not a script: the cheat-table entry sets the player's HP floor, `PlayerCtrl +0x16C` (normally -99999), to 1, so hits still land and stagger but HP stops at 1. Each load makes a new `PlayerCtrl` with the floor back at -99999, so the table writes 1 again within 0.1 s [live B57 step 9]. A 242 HP hit left HP at 1; unticked and reloaded, the player died. Falls are unsettled: the table's own text says a pit still kills, but the project's later note from the same day saw two falls out of the map end in a reload at the respawn point with no death, with the HP floor set [live, trainer test 4 Nov; the two notes disagree, not re-tested]. See [Memory map](../tools/memory-map-pc.md).

6. **A death respawn proves nothing about Restore (B57 step 5).** The game's own respawn already gives full HP, then the cap clamps it. To single out the mod, use a quit and load, or the arrival at a fog from the fight list.

7. **The effect fires on every load of the map, not only at fights.** That includes a bonfire trip there or reloading the game inside an arena. As a design choice the mod documents this in its README: "every time you load into a boss's area". With Restore on, reloading in an arena is a free heal.

8. **We tried "back to human on every load" first and removed it (B21).** Design: two seconds after each load, `1:130334(100000010, 19, 4)` in every arena map and Majula (group 990020, test flags 104840 to 104846, opt-out flag 104832). Result: it worked. The player turned human on the Majula load and after a death in a fight. But the "humanity restored" notice showed on **every** load, which the tester found distracting, so the group is switched off. The mod gives 99 Human Effigies at the arena setup and sells them free instead (see [Giving items](giving-items.md)). The test also recorded `f130307()` (【ステータス】プレイヤーの生者亡者タイプを取得, "get the player's human / hollow type") before and after: it read 0 both times, so its values are still unexplained.

9. **Do not use the DS1 "remove and give the flask" trick (not tried).** `1:131401` consumes an item, but the DS2 flask is one key item whose sips depend on upgrades. Deleting it risks losing them [untested]. The effect route made it unnecessary.

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| `1:130334(effect, 19, 4)` applies a player effect from a script | seen in the game: HP, Estus, durability and covenant changes | B56 (steps 1 to 4), B57, B21 |
| 110000010 refills HP and Estus | Estus byte +0xA 2 to 4 (max +0xB = 4); HUD agreed; HP 2154 to 2393 | live B56.2; B57.5 (Estus 1/4 to 4/4) |
| 110000010 is the bonfire rest's effect | exe call site in the bonfire code; the rest fills HP to the cap | code `+0x2027b0`; live B57 (log 15:59:43) |
| 110000010 holds no `100050:2` | effect events read from the regulation | data, 17 Nov |
| 110000010 leaves the hollow cap alone | cap 2154 after the refill; deaths lower it 5 % each | live B57 logs |
| 100000020 restores HP, spells, durability, un-hollows; no Estus | HP 1813 to 2154, cap 2154 to 2394, Rapier 41 to 45, Estus stayed 2 | live B56.1 |
| 60410000 repairs from a script | Rapier 40.4 to 45.0 | live B56.3 |
| A duration-0 effect acts once | HP x0.9 once, then normal | live B56.4 |
| `1:130303` stops damage and reactions; a load resets it | HP flat through the fight; damage after the warp | live B56.5 |
| 130311 to 130314 have code in the exe (B56's "not implemented" read a later handler) | an earlier handler's byte map gives them their own cases | code (17 Nov); never called live |
| No refill or repair command exists | 325 commands read by developer name | data (survey of the `.edd` names, 10 Nov) |
| Restore lands 1.3 to 1.5 s after the player appears | five logged loads | live B57 step 5 |
| The option group can start twice in one Majula visit | Restore toggled on, off, on, off | live B57 step 3 |
| Both options off: nothing refilled on arrival | HP 1402, Estus 3/4, Rapier 38.2 unchanged 17 s later | live B57 step 7 |
| Auto "return to human" works but its notice repeats | watcher flags and tester report | live B21 (6 Nov) |
| The Estus bytes are +0xA and +0xB, not the u16 at +8 | the u16 read 1 on a full flask | live B56.2; B57 logs |
| Items' effects can be triggered by a script | Repair Powder's effect | live B56.3 |

## Tools

| Tool | What it does for this topic |
|---|---|
| `tools/options.py` | The shipped Restore and Repair options: the arrival-map group (`arena_spec`), the option flags, the Herald's on/off list. |
| `tools/probe56.py` | The B56 throwaway probe group: the step flags and the command run at each step. It stays off in builds. |
| `tools/b56.py` | Runs a probe step from outside the game and prints before and after: HP, max, cap, stamina, covenant, Estus bytes, durability. Also `status` and `boss`. |
| `tools/b57watch.py` | Read-only watcher: logs each change of the option flags, covenant, HP, cap, Estus sips, durability and loaded maps every 100 ms. |
| `tools/_retired/humanity.py` | The retired B21 "back to human" group. |
| `tools/arenamod.py` | The shared "player is inside the map" constants and the EventParam row helper. |
| `tools/ce_table.py` | Builds the cheat table, including the "Can't die (practice)" entry. |
| `tools/arenalib.py` | Reads the player, flags and the bag from the running game. |

## Open questions

- **`19, 4`.** What the two trailing arguments of `1:130334` and `1:130333` select. We copied them from the original scripts.
- **`1:130336` (cancel).** Never exercised. The enemy form is `130335(chr, effect)`, so `130336(effect)` is a guess. The one test (B56 step 4) had nothing to cancel.
- **Which effect command lifts the cap?** 100000020 holds `100050:2` with argument 0 and lifted it. 100000010 holds the same command with argument 1 and made the player human. What each argument does is not settled.
- **Does 110000010 repair, and does it repair armour?** It holds `100110:2`, which repaired weapons in 100000020. In B57 every run with Restore on also had Repair on, so Restore alone was never isolated. Our durability readers track weapons and shields only; armour was never read.
- **Does 110000010 refill spell uses?** It holds the same spell command as 100000020. No B57 step watched it.
- **A lasting effect.** Does an effect with a real duration, started by a script, survive a map load? Needed for any handicap. It needs an EMEVD writer [untested].
- **The shortest delay.** How early the effect can fire after a load (instead of IN_MAP + 1 s) [untested].
- **Running twice.** Whether two firings close together (two quick arrivals) stack or are harmless [untested].
- **A rush past the fourth fight.** B57 stopped there. A full shuffled rush with Restore on is still open.
- **Online play.** The B56 and B57 tests ran with Steam set to Offline. Whether a script-triggered effect behaves the same online is unknown.
- **Alternatives to B21.** The Human Effigy's own effect, 60151000, and the one-param "no health loss" mod via `CriminalAndDeceasedParam` were noted as fallbacks and never tried [untested].

## Related pages

- [Event script groups](../modding-scripts/event-script-groups.md): adding the group and its EventParam row.
- [Event and talk script commands](../modding-scripts/script-command-reference.md): the other commands we proved.
- [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md): choosing the option and probe flags.
- [Memory map for PC](../tools/memory-map-pc.md): the `PlayerCtrl` fields and the bag layout.
- [Live-testing tricks and pitfalls](../modding-scripts/live-testing-notes.md): watchers, quit-and-load testing.
- [Warping the player by script](../modding-world/warping-by-script.md) and [The respawn point](../modding-world/respawn-point.md): what a death or warp does before the refill.
- [Adding new dialogue options](adding-dialogue-options.md): the on/off list that switches the options.
- [Difficulty levers](../modding-world/difficulty-levers.md): Hard mode (`130381(8)`) and the other ways to make fights harder.
- [Giving the player items](giving-items.md): the Human Effigies given at the arena setup.
