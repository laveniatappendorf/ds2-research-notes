# Difficulty levers: making fights harder or easier (PC Scholar 1.0.3.0, 2026-11-18)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 18 November 2026 from the Boss Arena project's tests.

Tags in square brackets say how we know a claim: `[live B56]` = seen in the game (the id points to the project's test
log, see [Home](../how-to-read.md#the-modding-pages)), `[code]` = read in the exe, `[data]` = read in the game files, `[inference]` = reasoned, not
proven, `[untested]` = an idea only. **Every live reading of an enemy's stats on this page is the Last Giant's, in New
Game.**

## Summary

- **Two levers make bosses harder with no new data.** Raise a bonfire's Ascetic cycle, by a memory write or a real
  Bonfire Ascetic: the Last Giant went from 2,530 to 5,313 HP [live A10, B56 step 7]. Join the Company of Champions by
  script, `1:130381(8)`: enemy damage x1.5, no HP change [live B56 step 6; the toggle again in B57 step 4].
- **The Ascetic does not multiply HP: it picks a column.** `EnemyParam` holds `stat_hp` and `stat_hp_ng1` to
  `stat_hp_ng7`. The Last Giant's row, 309600, reads 2,530 and 5,313 in the first two: the two numbers we saw live
  [data, read 18 Nov]. The first step is not always x2.1: it runs from x1.2 to x2.1 over the arena's 63 boss rows.
- **Easier:** `1:130334(110000010, 19, 4)` refills HP and Estus, `1:130334(60410000, 19, 4)` repairs gear [live B56,
  B57]. `1:130303(1)` stops all damage, but also every hit reaction, and a load resets it [live B56 step 5].
  The details are on [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md).
- **Never run by us:** `1:130317(generator, ratio)` sets an enemy's HP percentage. FromSoftware use it once, on Freja,
  with whole percents from 95 down to 50 [data].
- **A DS1-style handicap (your HP x0.7, your damage x0.85) is not built.** The multiplier commands exist, but DS2 has
  no `SpEffectParam` table. Effects are events in `SpEffect*.emevd` inside the regulation, so a new effect needs an EMEVD
  writer. And a script-triggered effect with duration 0 acts **once** [live B56 step 4].
- **Main traps:** the bonfire byte is `(cycles << 1) | lit`, not a tier number; `130381(8)` replaces any covenant the
  player holds; and a real Ascetic cannot be undone on that character.

### The levers at a glance

| Lever | What it changes | How it is pulled | Status |
|---|---|---|---|
| Bonfire level byte, or a Bonfire Ascetic | boss HP column; NG+ spawn rows of that bonfire | memory write, or the item | live, one boss |
| Company of Champions | enemy damage x1.5 | script `1:130381(8)` | live, one boss |
| `EnemyParam` fields (HP columns, defences, damage multipliers) | the enemy itself | a param edit | data only, never tried |
| `1:130317` | HP percentage of a generator's enemy | script | data only, never run |
| Restore and repair effects | player HP, Estus, spell uses, durability | script `1:130334` | live |
| `1:130303` | all damage and all hit reactions | script | live, too blunt for a mode |
| A handicap effect | player max HP, attack, damage taken | a new effect event, then `1:130334` | not built |

### 1. The bonfire level and the Ascetic

[Bonfire lit state](bonfire-lit-state.md) has the byte's layout and the code. [NG+ near boss arenas](ng-plus-differences.md)
has the generator side. This is what they mean for difficulty.

**What the byte is.** [code] Bit 0 is the lit bit; bits 1 to 7 count Ascetic cycles. To set `n` cycles and keep the
bonfire lit, write `(n << 1) | 1`: 3, 5, 7 and so on. A plain `n` writes the wrong thing.

**What we saw.** [live A10, 28 Oct] The Last Giant's HP read 2,530 at level 1. We wrote 2 to Cardinal Tower's byte
(bonfire 10655, nothing else), went away and came back, and walked to the arena. A new character object read 5,313
(x2.1). He is not in memory after the reload; he spawns when the player nears the arena. [live B56 step 7, 10 Nov] A
real Bonfire Ascetic burnt at the same bonfire gave the same 2,530 to 5,313. It is permanent for that character.

**Where the number comes from.** [data, read 18 Nov] The Last Giant's generator (m10_10, row 857) names bonfire 10655 in
its `MapObjectBonfireID` field. Its enemy row, `EnemyParam` 309600, holds `stat_hp` 2,530 and `stat_hp_ng1` 5,313. The
two live readings are those two columns. So the byte does not scale HP: it chooses which of eight columns the game
reads [inference: we did not trace the code that reads the column. The exact match of both numbers is the evidence].

The other six columns, `stat_hp_ng2` to `stat_hp_ng7`, follow a shared shape. In 810 of the 976 rows they are x1.2,
x1.4, x1.5, x1.6, x1.7 and x1.9 of the first step. The other 121 rows use other schedules, and 45 rows hold one HP in all
eight columns. No row's columns fall. The **first step** differs by enemy. Over the 63 enemy rows of the arena's 44
boss fights it is x1.2 (14 rows), x1.3 (4), x1.4 (12), x1.5 (1), x1.6 (10), x1.7 (2), x1.8 (6), x1.9 (4), x2.0 (8) and
x2.1 (2). **The Last Giant's x2.1 is the top of that range**, so do not quote it as "the NG+ step" for other bosses.
[data, read 18 Nov]

The likely mapping is cycle 0 = `stat_hp`, cycle 1 = `stat_hp_ng1`, up to cycle 7 = `stat_hp_ng7`. That fits the cap of
8 on the intensity total and a total of 1 in New Game [inference; see [NG+ near boss arenas](ng-plus-differences.md)]. We
only saw cycles 0 and 1.

**What each part rests on.**

| Statement | Basis |
|---|---|
| Writing byte 2 gave the cycle-1 HP | live A10 (HP read after reload) |
| A real burn gave the same HP | live B56 step 7 |
| The byte is `(cycles << 1) \| lit` | code, see [Bonfire lit state](bonfire-lit-state.md) |
| A10's 2 left the bonfire unlit | code. The travel menu was not looked at, and the byte went back to 1 afterwards |
| A real burn leaves 3 (cycle 1, lit) | inference. Nobody read the byte after B56's burn |
| Bytes 3, 4, 5 and above | untested. Only 2 and a real burn were tried |
| The boss's HP follows the bonfire named in his generator | inference. The Last Giant's generator and his arena bonfire are both 10655, so A10 cannot tell them apart. The Pursuer is the case where they differ, see [NG+ near boss arenas](ng-plus-differences.md) |

**What else a raised bonfire changes.** We read HP only. The same total picks the NG+ spawn rows near that bonfire and
the live spawn limit ([NG+ near boss arenas](ng-plus-differences.md)). `EnemyParam` also has `defence_*` fields, four
`*_damage_mult` fields and a `speffect_id`; we did not check whether anything there changes with a cycle [untested].
B56 step 7 burnt an Ascetic at the Last Giant's arena and reported nothing odd, but no NG+ spawn row stands near him.

**What a mod can ship.** [code] The bonfire list is rebuilt at every load and the builder clears every byte, so no param
row can give a bonfire a starting cycle. [data] No script command writes the byte ([Bonfire lit state](bonfire-lit-state.md)).
That leaves three routes:

1. **The item.** A shop row can sell the Bonfire Ascetic (item 60527000 [data, project's 10 Nov survey]; see
   [Vendor shops](../modding-menus/vendor-shops.md)). The player burns it at a lit bonfire. It is one-way and permanent per character.
2. **A memory write** from a trainer or cheat table, offline only. Reversible, and a tier can be set directly.
3. **A param edit** of the `EnemyParam` columns. It needs no memory write at all. [untested]

### 2. Company of Champions by script

| Command | Developer name | Gloss |
|---|---|---|
| `1:130381(covenant)` | 【ステータス】プレイヤーの誓約を変更 | change the player's covenant |
| `f130361()` | 【ステータス】プレイヤーの誓約を取得 | get the player's covenant |

`1:130381(8)` joins the Company of Champions and `1:130381(0)` leaves it. [live B56 step 6] The covenant, read from
memory, went 0 to 8 and back to 0 from an event group. [live B57 step 4] The same again from the shipped build's options
list. FromSoftware's event scripts never use `130381`. Their talk scripts do, 48 times: `130381(0)` 12 times, and the rest
with a chosen or computed covenant [data, 18 Nov]. Our calls came from event groups and worked.

**What it does.** [live B56 step 6] Enemy damage was **x1.5**. Three pairs of hits from one attack set of the Last
Giant: 398 to 597, 267 to 401 and 299 to 448. That is x1.5 to within one point of rounding. The Victor's Stone in Majula
still offers to join the covenant afterwards. [data] That NPC row already allows joining (`NpcEventParam` 78601000, menu
bits 16 and 32), so players can also join in the game with no mod menu. The covenant's ring text says it strengthens
enemies in your world and gives no figure.

**No HP change, read indirectly.** [live B56 step 6] The Last Giant respawned with the covenant on at max HP 5,313. That
is the Ascetic value from A10 and step 7, so the covenant added nothing on top. We did not read his HP at level 1 with
the covenant alone [inference].

**What it costs.** The covenant is one field, so joining replaces any covenant the player holds, and `130381(0)` leaves
the player with none. It cannot give the old one back [inference; 8 to 0 was seen, a character with another covenant was
never used]. We never checked what happens to rank or progress in the covenant replaced [untested]. We measured one boss
and three hits; ordinary enemies, spells and other bosses are not measured [untested]. We did not find where x1.5 lives. (A param value that matches, DamageAdjustParam `enemy_damage_mult_coc` 1.5, is on
[balance: vanilla vs PS3 Scholar](../builds/balance-vanilla-vs-scholar.md#6-damage-pvp-and-new-game).)
`VowParam` (10 rows of three small numbers and a zero) has nothing that reads as a multiplier, and we did not look in the
exe [data, 18 Nov].

### 3. Boss HP by generator: `1:130317`

| Command | Developer name | Gloss |
|---|---|---|
| `1:130317(generator, ratio)` | 【ステータス】ジェネレータ指定でHP割合を設定 | set the HP ratio, by generator |

[data, read 18 Nov; the call chain is our reading] FromSoftware use it **once**, in Freja's map script (m10_14). A small
group (2147483551, named 【実行】ボスのＨＰ設定, "execute: boss HP setting") runs `1:130317(803, arg0)`, where 803 is
Freja's generator. The group that calls it passes whole percents: 95, 90, 85 and so on down to 50 in steps of five. It
picks one by a ten-way check of ten saved flags (114000085 to 114000094), which an earlier group sets as her HP falls in
her "outpost battle" (前哨戦). So the original use is: **put the boss back at the HP percentage the earlier fight saved.**

Two details from the script. The ratio is a **percent**, not 0 to 1, in every original call. And the setter group runs
the command only when `f130602()` (【ネット】ゲストユーザー判定, "guest user check") is not 1, so a guest in someone
else's world does not set the boss's HP. [data] The exe has its own case for the command in the handler that covers
130307 to 130511, so it is implemented [code, see [Script command reference](../modding-scripts/script-command-reference.md)].

We never called it. We do not know what a value above 100 or of 0 does, whether it works on any boss or only on Freja,
or whether a boss then plays its later phase [untested]. The mod clears Freja's ten flags at every load so she starts at
full HP ([NG+ near boss arenas](ng-plus-differences.md)). The reverse is an idea: set one flag to start her lower.
[untested]

### 4. Player effects: the easier levers

A script starts a player effect with `1:130334(effect, 19, 4)`. Effects, their contents, where the Estus count lives and
the timings are all on [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md). What matters for difficulty:

| Lever | Effect | What the tests showed | Cost |
|---|---|---|---|
| Rest-style refill | `110000010` | HP to the current cap and the Estus Flask refilled [live B56 step 2, B57] | a free heal on every load of the map; it does not lift the hollow cap |
| Repair | `60410000` (Repair Powder) | a worn weapon back to full [live B56 step 3, B57] | none seen |
| Post-creation restore | `100000020` | HP, spell uses, durability, un-hollow; **not Estus** [live B56 step 1] | leaves HP under the new maximum |
| Invincibility | `1:130303(1)` | no damage, **no hit reactions**, reset by a load [live B56 step 5] | the fight stops being a fight |
| An HP floor | a memory write, `PlayerCtrl +0x16C` set to 1 | hits land and stagger, HP stops at 1 [live B57 step 9] | cheat table only; see [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md) |

Four "set HP" commands, `130311` to `130314`, have real code in the exe, but nothing has called them [code, see
[Script command reference](../modding-scripts/script-command-reference.md)]. The B56 write-up called them "not implemented". That was too strong.

The mod's three settings are one on/off list at an NPC: Restore (`110000010`), Repair (`60410000`) and Hard mode
(`130381`). How to build such a list: [Adding dialogue options](../modding-menus/adding-dialogue-options.md).

### 5. Why a handicap needs an EMEVD writer

A handicap effect would normally be a new row in a SpEffect param. **DS2 has no such table.** The regulation holds 205
params, none of them a SpEffect one, and 11 `SpEffect*.emevd` files [data; see [Editing params](../modding-scripts/editing-params.md)]. An
effect is one **event** in those files, and the event id is the effect id. [data]

The multiplier command is `1000:1`. [data, 18 Nov] Its arguments are an int32 stat index, a float32 duration in seconds
and a float32 multiplier (12 bytes). We checked it by decoding all 899 events in the 11 files. The indexes read from the
game's own effects:

| Stat | Meaning | Examples in the game's effects |
|---|---|---|
| 7 | max HP | rings from x0.5 to x1.15 (duration 0, "while worn"); a mask x0.9 (21502100); effects x0.7 for 60 s (91290030, 31290001) |
| 8, 9 | max stamina, equip load | rings x1.05 to x1.2 |
| 25 | attack | a weapon effect x1.05 for 20 s (1990010); enemy states from x0.7 to x1.5 |
| 26 | damage taken | a spell effect x0.85 for 20 s (34100010); many enemy states, some very low |
| 59 | physical damage taken | three rings: x0.9, x0.9 and x0.875 |

The meaning of an index is read from the items that use it [data, project's 10 Nov survey], so a name here is a reading,
not a label from the game. A handicap is two or three `1000:1` lines in a new event: for example stat 7 x0.7 and stat 25
x0.85 with a long duration, or stat 26 x1.3. Nothing in the data rules that out. Three things stand in the way:

1. **No writer.** We only read these files. A new event must be written into `SpEffectWideUse.emevd` (or another file).
   The layout is simple, but we have not written one. See "How to do it".
2. **The game may not read it.** The mod ships the regulation as a plain DCX holding the 12 FMG and 11 EMEVD files. Whether
   the game reads a *changed* EMEVD from there is open ([Editing params](../modding-scripts/editing-params.md)) [untested].
3. **Duration 0 acts once.** [live B56 step 4] We fired the Black Witch Domino Mask's effect (21502100, max HP x0.9,
   duration 0) from a script. Current HP went 2,393 to 2,153 once. No lower maximum showed anywhere, and one Estus later
   HP read 2,393 again. On a worn mask, duration 0 means "while worn"; from a script it is a one-off. A handicap needs a
   real duration, re-fired at each load. Whether a timed effect survives a map load is open [untested].

**Gear stat effects.** [data] The same files hold the stat changes of armour, rings and weapons, one event per item (the
event id is the item's own id). Attribute changes use `1000:0 (stat, 0, +n)` with 6 = Vigor, 7 = Endurance, 8 =
Vitality, 9 = Attunement, 10 = Strength, 11 = Dexterity, 12 = Intelligence, 13 = Faith and 14 = Adaptability
[data, project's 31 Oct survey]. A scan of all 120 uses on 18 Nov found none with index 9, so **no gear raises Attunement**.
`1000:19` is a second form: a stat index, two bounds (0 to 99, or 6 to 60, in the files) and two signed bytes; we did not
decode the bounds. Its indexes are five lower than `1000:0`'s (1 = Vigor to 9 = Adaptability), and its 14 uses hold no
index 4, so Attunement is not raised through it either [data, 18 Nov]. Note that **the index spaces differ by command**:
stat 7 means Endurance in `1000:0` and max HP in `1000:1`. Reading these events shows what an item does, and they are the
command shapes a handicap would use.

### A. Raise one boss for a test or a "hard" setting

1. Find the boss's generator row (`GeneratorParam_<map>`) and read `MapObjectBonfireID`. The game compares that bonfire's
   cycle for the generator's spawn rows [code]. Start with it. It is not always the bonfire he is fought at, and which
   one moves his HP where they differ is unproven ([NG+ near boss arenas](ng-plus-differences.md)).
2. Offline, read the bonfire's byte (`tools/arena.py bonfire <id>`). Write `(n << 1) | 1` (`tools/arena.py bonfire <id>
   <byte>`). Keep bit 0 set so the travel menu keeps the bonfire. [code]
3. Leave the area and come back, so the generator is built again, then go near the arena. He spawns with the new HP.
   [live A10]
4. Read his HP from memory to check it against the `EnemyParam` columns. Put the byte back when the test is done; it
   takes effect on the next load either way. Do not test with a real Ascetic on a character you care about. [live B56]

### B. A Hard mode toggle by script

1. Run `1:130381(8)` from an event group that starts at a load or from a talk menu result. It needs no data change.
2. Offer `1:130381(0)` as the off switch, and say in your text that it replaces the covenant.
3. Derive the shown state from the covenant (`f130361() == 8`), not from your own flag, so that joining at the Victor's
   Stone shows as on. The mod does this at each Majula load. [live B57 step 4]

### C. A handicap effect (a design, not built)

1. Choose the lines: `1000:1 (7, d, 0.7)` and `1000:1 (25, d, 0.85)`, or `(26, d, 1.3)`. Use a real duration `d`, long
   enough to cover a fight, and re-fire at every load of the arena maps. [inference from B56 step 4]
2. Write a new event with those lines. Use an event id that no effect file uses. Keep the file's counts and offsets
   consistent (see the file notes in "Tools"). [untested]
3. Put the file in the regulation and check that the game loads a changed EMEVD. This is the first thing to test.
   [untested]
4. Fire it with `1:130334(effect, 19, 4)` once the player is inside the map. The wait and the group layout are on
   [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md).
5. Read max HP and damage taken from memory before and after, and after a map load, as the B56 probe did.

## Pitfalls and what failed

1. **Writing the cycle as the whole byte.** [code, live A10] A10 wrote 2: one cycle, but unlit. The HP step worked; the
   bonfire would have left the travel menu. Write `(n << 1) | 1`. See [Bonfire lit state](bonfire-lit-state.md).
2. **Quoting x2.1 as "the NG+ step".** [data] It is the Last Giant's first step. The first step is x1.2 for 14 of the 63
   boss rows we checked.
3. **A script-triggered duration-0 effect acts once.** [live B56 step 4] The Domino Mask's cut came and went. Plan a
   timed effect and a re-fire. `mask_off` (`1:130336`) could not be tested: nothing was left to cancel.
4. **Invincibility is not "can't die".** [live B56 step 5] `1:130303(1)` kept HP flat through the Last Giant fight, but
   the tester could stand still and attack: no hit reactions. A load reset it, and damage got through after the warp.
5. **"Not implemented" for `130311` to `130314`.** [code] B56 read a later handler and stopped. An earlier handler has
   cases for all four. Nothing was called, so "no script can't-die" means "none tried".
6. **The covenant replaces the player's own.** [inference; `tools/options.py` and the README say so] Hard mode off leaves
   no covenant (8 to 0 seen, live B56 step 6). A player who held another one loses it. Say so in your text.
7. **The covenant's "no HP change" is indirect.** [live B56 step 6] The boss was already at the Ascetic value. If you
   rely on it, read HP at level 1 with the covenant alone.
8. **A real Ascetic cannot be undone.** [live B56 step 7] It is permanent on that character. B56 used a disposable
   character for this reason.
9. **A boss's tier may follow another bonfire.** [data, code] A generator names its own bonfire. The Pursuer is the known
   case ([NG+ near boss arenas](ng-plus-differences.md)); we did not test whether his HP follows it.
10. **The post-creation effect does not refill Estus.** [live B56 step 1] Use `110000010`. See
    [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md).

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| Byte 2 at bonfire 10655 gave the Last Giant 5,313 HP (from 2,530) | HP read from the new character object after a reload | live A10 |
| A real Ascetic at that bonfire gave the same, permanently | HP read after the burn | live B56 step 7 |
| 2,530 and 5,313 are `stat_hp` and `stat_hp_ng1` of `EnemyParam` 309600 | the row read from the regulation | data, 18 Nov |
| The Last Giant's generator (row 857) names bonfire 10655 | `GeneratorParam_m10_10` read | data, 18 Nov |
| The six later columns follow x1.2 ... x1.9 of the first step in 810 of 976 rows; the first step is x1.2 to x2.1 | all rows read; the 63 boss rows counted | data, 18 Nov |
| The byte is `(cycles << 1) \| lit` | writer, readers and loader all split it so | code, see [Bonfire lit state](bonfire-lit-state.md) |
| `1:130381(8)` gives enemy damage x1.5 | three hit pairs from one boss | live B56 step 6 |
| Covenant 0 to 8 to 0 from an event group | covenant read from memory | live B56 step 6, B57 step 4 |
| FromSoftware's event scripts never use `130381`; talk scripts do 48 times | every original script read | data, 18 Nov |
| `1:130317` is used once, on generator 803, with 95 down to 50 | the m10_14 script read | data, 18 Nov |
| `1:130317` has its own case in the exe | the handler's byte map and jump table | code, see [Script command reference](../modding-scripts/script-command-reference.md) |
| `1:130317` was never called by us | no test | untested |
| `1000:1` is (int32 stat, float32 seconds, float32 multiplier) | 899 events in 11 files decoded | data, 18 Nov |
| No `SpEffectParam` among the params; 11 `SpEffect*.emevd` | the regulation's contents | data; [Editing params](../modding-scripts/editing-params.md) |
| A duration-0 effect acts once from a script | HP x0.9 once, then normal | live B56 step 4 |
| `1:130303` stops damage and reactions; a load resets it | HP log flat; damage after the warp | live B56 step 5 |
| Restore, Repair and Hard mode work as options | all steps passed | live B57 |
| No effect uses index 9 of `1000:0` or index 4 of `1000:19` | all 120 and 14 uses scanned | data, 18 Nov |

## Tools

| Tool | What it does for this topic |
|---|---|
| `tools/arena.py` | `bonfire <id> [byte]` reads a bonfire's level byte, or writes it when a byte is given. The byte is raw, not a tier. |
| `tools/options.py` | The Restore, Repair and Hard mode toggles: the on/off list, the covenant calls (`SETTINGS`), the group that fires the effects at each arrival. |
| `tools/probe56.py` | The throwaway B56 probe group: one command per step flag (`restore_a`, `restore_b`, `repair`, `mask_on`, `inv_on`, `champ_on` and the "off" steps). It is off in builds. |
| `tools/b56.py` | Fires a probe step from outside the game and prints HP, max HP, cap, covenant, Estus and durability before and after; `boss` reads the nearest boss's HP. |
| `tools/ce_table.py` | Builds the cheat table, including the "Can't die (practice)" HP-floor entry. |

There is no EMEVD tool in `tools/`. The reads on this page used a short parser from a sister project. The layout it
relies on: `EVD\0` and a flag byte at +5 (0xFF means 64-bit words); 16 header words from +0x10 (a count or size and an
offset for the event, instruction, layer, parameter, link, argument and string tables); an event record of five words
plus 8 bytes; an instruction record of a bank int32, an id int32, an argument length and an argument offset. A writer has
to keep every count and offset consistent [data, 18 Nov; the write side is an inference].

## Open questions

- **Cycles 2 to 7.** Write 3, 5, 7 and 15 to one lit bonfire, read the boss's HP each time and check it against the
  `EnemyParam` columns. Burn a real Ascetic and read the byte afterwards (3 is expected). [untested]
- **What else a cycle changes.** Damage, defence and souls on the same boss; whether a fight's NG+ rows come with it
  ([NG+ near boss arenas](ng-plus-differences.md)). [untested]
- **An `EnemyParam` edit as a lever.** Raise `stat_hp` and the `_ng` columns on one boss row, then check that the live HP
  follows. Try the damage multiplier fields too. [untested]
- **The Company of Champions on more than one boss.** Do ordinary enemies, spells and the DLC bosses take x1.5? Read HP at
  level 1 with and without the covenant. What happens to the rank of the covenant it replaces? [untested]
- **Where x1.5 lives.** Not in `VowParam`. The exe, or an effect we did not link to the covenant, is the place to look.
  [untested]
- **`1:130317`.** Call it from a probe group: 50 on a boss, 100, 0, and a value above 100. Does a boss with a phase change
  then play its later phase? Does it need the generator to be in the loaded map? [untested]
- **A timed effect.** Does an effect with a real duration, fired by a script, survive a map load? Needed for any
  handicap. [untested]
- **A changed EMEVD in the plain regulation.** Does the game read it at all? [untested]
- **No writer needed?** An existing effect with a real duration might serve: the enemy effect 91290030 (max HP x0.7 for 60 s)
  and 31290001 (a spell effect, stat 7 x0.7 for 60 s). Fire one on the player and re-fire before it ends. Both also hold
  five stats (52 to 56) at x1.2 and `100040` entries, which we did not read. Whether `130334` accepts an enemy or spell
  effect id for the player is not known. [untested]
- **Online play.** All of this was offline. [untested]

## Related pages

- [Bonfire lit state](bonfire-lit-state.md): the level byte, its two halves and how to write a tier.
- [NG+ near boss arenas](ng-plus-differences.md): the intensity total, the NG+ spawn rows and which bonfire governs a boss.
- [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md): the restore, repair and human effects, the timings and the cheat-table HP floor.
- [Script command reference](../modding-scripts/script-command-reference.md): `130381`, `130317`, `130303`, `130311` to `130314` and the exe handler chain.
- [Bringing a boss back](respawning-bosses.md): kill counts, `SpawnLimit` and the Bonfire Ascetic.
- [Adding dialogue options](../modding-menus/adding-dialogue-options.md): the on/off list that switches the options.
- [Editing params](../modding-scripts/editing-params.md): the regulation's contents, the plain DCX, and what is untested about EMEVD files.
- [Vendor shops](../modding-menus/vendor-shops.md): selling the Ascetic and other items.
- [Memory map for PC](../tools/memory-map-pc.md): the player and bonfire fields for the memory-write routes.
