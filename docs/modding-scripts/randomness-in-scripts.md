# Randomness in event scripts: the dice, the game's percentage roll, and drawing without repeats (PC Scholar 1.0.3.0, 2026-11-18)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 18 November 2026 from the Boss Arena project's tests.
> Test ids such as B52 point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags: `[live B52]` seen in the
> game, `[code]` read in the exe, `[data]` read in the game files, `[inference]` reasoned, not proven, `[untested]` an idea only.

## Summary

- **The dice are one command and one function.** `1:1(slot, min, max)` rolls a whole number into a random slot and
  `f1(slot)` reads it. `f1` works **inside a condition**: `f1(0) == k` chose fights ten times in four runs, with both ends of
  `1:1(0, 1, 39)` drawn. Roll in one state, read in the next. [live B52]
- **The game's own percentage roll is `1:140531`.** FromSoftware rolls `1:1(0, 0, 99)`, registers `1:140531(0, 0, N, 5)` and
  tests `f140001(0)`: a hit when the roll is `<= N`, so a chance of N + 1 %. The Forlorn's invasion draw is N = 14 (15 %).
  Six other groups in the event scripts use it. We never ran it. [data]; reading of the arguments [inference]
- **Several different values, no repeats:** one "taken" flag per value, tested in the same condition as the roll:
  `(f1(0) == k) && (f6(taken_k) == 0)`. Eight rolls, then a scan for the next free value, so the draw always ends. The rolls
  worked live; the scan never ran, because `f1` was always readable. [live B52]; scan [inference]
- **The traps:** a roll that matches no condition leaves the group idle, so end with a catch-all; one result cannot show the
  roll is being read if your fallback can give the same result; "taken" flags are global and keep their value, so clear them
  at the start of every sequence.
- **Never tried:** the percentage roll, a same-state read in an event group, the slot rules (how many, shared or not), the
  random-wait forms, a full 39-pick sequence, any count of how even the dice are.

### What is proven, what is read from the data, what was never tried

| Proven in the game | Read from the game's files | Never tried |
|---|---|---|
| `1:1(0, 1, 39)` rolls; `f1(0) == k` is true in a state's condition; both ends occur; ten picks over four runs and two game launches, none repeated within a run [live B52] | The Forlorn's two draw groups, the six other users of `1:140531`, the debug flag 109990, the enemy AI's and Majula's reads of `f1`, argument shapes and counts [data] | `1:140531` and `f140001` on our own roll; setting flag 109990; reading `f1` in the same state as the roll in an event group; `5:1`; `f4`, `1:140511` as random waits; the scan fallback; a full run |

### 1. The two commands

| Command | Developer name | What we know | Proof | Original use |
|---|---|---|---|---|
| `1:1(slot, min, max)` | 【乱数】乱数生成, "generate a random number" | Rolls a whole number from `min` to `max` into random slot `slot`. Both ends came up with `(0, 1, 39)`: fight 1 and fight 39. Event scripts use slots 0 and 1, the enemy AI 0 to 3. The `max` can be a script argument (the Forlorn passes its spot count) | [live B52]; slots and shapes [data] | E 37 · T 64 |
| `5:1(condition, slot, min, max)` | the conditional form (condition first) | Majula's armour-shop placement rolls 1 to 5, or 6 to 12, depending on three flags. Never tried | [data] | E 2 |
| `f1(slot)` | 乱数値の取得, "get the random value" | The value last rolled into that slot. In a condition: works. As a command argument: the Forlorn does it (`1:130121(var, f1(1))` stores the roll in an area variable); we did not | [live B52] condition; [data] argument | E 68 · T 318 |

These counts agree with [Event and talk script commands](script-command-reference.md). The enemy AI scripts (`ai*.esd`) are the
dice's heavy users: about 10,600 `1:1`, 914 `5:1` and 39,300 `f1`, 37,100 of them in conditions. [data, walked 18 Nov 2026]
So the engine reads the dice in conditions all the time, and so does Majula's own event script (26 conditions). When the
picker was built the project knew only the Forlorn's recipe, which reads `f1` as a command argument, so the condition form
was a live test (B52). It passed. The Majula reads were found afterwards, in the survey for this page. [data]

### 2. The Forlorn's draw: the game's own recipe

Each Forlorn invasion area has two small library groups, copied into each map under different compiler numbers (in
`event_m10_16` they are groups 2147483512 and 2147483511). [data]

| Group (developer label) | Its steps |
|---|---|
| Hit test: 【Lib】【DC】【条件】放浪者_ランダム抽選, "library, Forlorn, condition: random draw" | State 1 entry: `1:1(0, 0, 99)`. State 2 entry: `1:140531(0, 0, chance, 5)` and `1:140301(0, 109990, 1)`. Its conditions: `f140001(0)` goes to "hit", `1` goes to "miss" |
| Spot pick: 【Lib】【DC】【実行】放浪者_ランダム抽選_アタリ, "library, Forlorn, execute: random draw, hit" | Sets the draw-done flag; entry `1:1(1, 1, n)` rolls into **slot 1**, the spot count `n` as the maximum; the next state runs `1:130121(var, f1(1))`, which keeps the roll in an area variable. The spawn presets later compare that variable with their own spot number |

The roll is made in one state and read in the next. `1:140531` is 【グループ条件：イベント】乱数比較, "group condition
(event): random comparison". A **group condition** is registered in a state's entry, and its result is read with `f140001(slot)`,
【グループ条件】グループ条件を満たす, "the group condition is met", the same function the mod uses for "is the player in this
map". `1:140301` is 【グループ条件：フラグ】イベントフラグ比較, "group condition (flag): compare an event flag". [data]

**What the four arguments of `1:140531` look like.** All 23 uses in the 28 original event scripts, 17-18 Nov. [data]

| Argument | Values seen | Reading |
|---|---|---|
| 1 | 0, 1, 2 | The group-condition slot that gets the result; `f140001(slot)` reads it [inference] |
| 2 | always 0 | Probably the dice slot to compare: every roll that feeds one of these goes into slot 0 [inference] |
| 3 | 19 or 0 written in the call, or a script argument (14 for most Forlorn areas, 39 for the Forest) | The value compared with the roll |
| 4 | 5 in 20 uses, 0 in three | The comparison kind. 5 reads as "<=": the scripts that split a roll into bands end on "<= 99", the top of the roll. 0 is probably "==" [inference] |

**Who uses it.**

| Use | Shape | Reading |
|---|---|---|
| Forlorn draw, 14 maps | roll `(0, 0, 99)`, compare `(0, 0, N, 5)` | N = 14 is 15 %. The Forest uses 39 (40 %), and only after the King's Ring. Huntsman's Copse has no roll [data] |
| An invader check labelled for the Bell Keeper covenant, `m10_19` | roll `(0, 0, 99)`, compare `(0, 0, 19, 5)` | 20 % [inference] |
| Three groups in `m10_14`, `m10_33`, `m20_10` (a piglet, a dwarf statue, a ballista) | roll `(0, 0, 999)`, compare `(0, 0, 0, 0)` | Probably "roll equals 0": one in a thousand [inference] |
| Two groups in `m50_35`, 乱数生成から2体抽選 and 乱数生成から3体抽選, "a lottery among 2 (or 3) from a random roll" | one roll, then two or three `1:140531` into slots 0, 1 (and 2) with rising thresholds | Bands. The conditions are tried in order, so the first band that holds wins [inference] |

**The debug flag.** The hit-test group also registers `1:140301(0, 109990, 1)` into the same slot. The `.edd` labels that
step "DEBUG: flag 109990 ON makes a sure hit". All 14 draw groups do it, and no script, param or exe reference sets the
flag. [data] If two conditions in one slot are an OR, setting 109990 forces every Forlorn draw to hit. [inference] We have
not set it; that would be the quickest way to test the invasions. [untested]

### 3. Where and when to read the roll

- **In a condition of the next state.** This is what the Forlorn does and what the picker below does. [data; live B52]
- **After a timer.** Majula's cat group rolls `1:1(0, 0, 50)` in a state's entry with a one-second timer
  (`1:140511(0, 1, 3, 1)`), then reads `f1(0) > 0 && f1(0) < 10`, and so on, in bands. So a value survives a state change of
  at least a second. [data]
- **In the same state, in a condition.** The enemy AI does this about 5,400 times and 22 talk-script states do it. No event
  group we read rolls and reads one slot in one state (Majula's armour-shop pick rolls slot 1 and reads slot 0; see the
  pitfalls), and we never tried it. [data]; in an event group [untested]
- **Not in the group's first state.** Commands in a group's first state did not take (B9; see
  [Event script groups](event-script-groups.md)). Our roll states are never the first.
- **Conditions are tried in order and the first true one is taken**, so "`f1(0) == k` and the flag is clear" lists work as a
  draw with a filter. [live B52]; the rule: see [Event script groups](event-script-groups.md)

### 4. Other random things in the name table (none tried)

| Command | Developer name | What we know | Proof | Original use |
|---|---|---|---|---|
| `f4(min, max)` | ステート経過時間用の乱数値の取得, "get a random value for the state's elapsed time" | Talk scripts wait on it: `f3() > f4(0.1, 3)` is a random wait of 0.1 to 3 s | [data] | T 36 |
| `1:140511(slot, min, comparison, max)` | 【グループ条件：イベント】ステート経過時間比較, "group condition (event): compare the state's elapsed time" | The same `(min, max)` pair. Mostly `min == max`, a plain timer. The Forlorn's spawn wait passes a range (0 to 5 s, the Bastille 0 to 10), a random wait [inference] | [data] | E 300 |
| `1:131705(...)` | 【ジェネレータ】初期開始位置を指定ポイントからランダムで選択して上書き, "pick the start position at random from the given points" | The Forlorn's random spawn point | [data] | E 14 |
| `1:131713(...)` | 【ジェネレータ】指定ジェネレータ内のキャラをランダム強制生成, "force-spawn a random character in the generator" | Not traced | [data] | E 2 |
| `f140002(slot)` | 【グループ条件】ホスト調停した結果がグループ条件を満たす, "the host-arbitrated result meets the group condition" | The host-decided twin of `f140001`. Every random compare read above uses `f140001`, the local one | [data] | E 77 |

### A. One of n, equal chance (done live)

1. In a state that is not the first, run `1:1(0, 1, n)` and add a condition `1` that goes to a "test" state.
2. In the test state put one condition per outcome, in order: `f1(0) == 1` to the first, `f1(0) == 2` to the second, and so on.
3. Make the last condition `1` and send it to a fixed outcome. **Do not send it back to the roll**: if `f1` ever stops
   matching, a re-roll loop never ends.
4. Test with at least two results, and pick a test where your fallback gives a different answer from a good roll (below).

### B. A percentage chance, the game's way (never run by us)

1. State 1, not the first: `1:1(0, 0, 99)`; condition `1` to state 2.
2. State 2: entry `1:140531(0, 0, N, 5)`; conditions `f140001(0)` to "hit" and `1` to "miss". The chance is N + 1 %.
3. To force a hit while testing, register your own flag into the same slot as the Forlorn does
   (`1:140301(0, flag, 1)`). That relies on the OR reading above. [untested]

### C. Several different values with no repeats (worked example: the shuffled boss rush)

This is the only multi-draw we built. The rush draws one of 39 fights after each win; the picker is a group
(`picker_spec` in `tools/bossrush.py`) that runs once per pick. [live B52]

1. **One flag per value.** Reserve a "taken" flag for each (39 here) in a free run; see
   [Event flags and safe ids](event-flags-and-free-ids.md). They are global flags and keep their value after a sequence ends, so
   **clear all of them when a new sequence starts**.
2. **Trigger.** The group waits on a "pick now" flag (cleared as the first thing it does) and on the player being in the map
   (`1:140402` and `f140001`), because a map that merely streams in runs its scripts too. [live B39c, B52]
3. **Stop when everything is taken.** One condition tests every taken flag and ends the sequence before any roll. Without it
   the scan below could loop.
4. **Eight rolls.** Roll state `i`: entry `1:1(0, 1, 39)`, condition `1` to test state `i`. Test state `i`: for each value
   `k`, in order, the condition `(f1(0) == k) && (f6(taken_k) == 0)` goes to "take k". In test states 1 to 7 a last
   condition `1` goes to the next roll state.
5. **The scan.** The eighth test state ends differently: after the 39 "take" conditions, `f1(0) == k` goes to scan state `k`
   and a last `1` goes to scan state 1. Scan state `k`: `f6(taken_k) == 0` goes to "take k"; otherwise `1` goes to scan
   state `k + 1`, wrapping from 39 to 1. It always ends, because step 3 guarantees a free value.
6. **Take k:** entry sets `taken_k` to 1, then does the work (here, a warp).

The whole group is about 100 states: 16 roll and test states, 39 "take", 39 scan, and 5 for the trigger, the in-map wait,
the check and the end. It holds about 450 conditions, and the eighth test state has 79. (Counted from `picker_spec`.)

**Why this shape.** A loop that rolls until it hits a free value is shorter, but if `f1` could not be read in a condition it
would never end. The bounded version makes that failure harmless: the unreadable value matches nothing, all eight tests miss,
and the scan starts at value 1, so the run goes in order instead of sticking. [inference]; an offline walk of the built
state table with fake rolls ends on a free value in 400 of 400 random cases and gives the in-order result when `f1` reads 0
[inference, not the game]

**How much the scan matters.** Eight rolls all land on taken values with probability (d/39)^8, where d is how many are taken:
d = 10: 0.002 %; 20: 0.5 %; 30: 12 %; 35: 42 %; 38: 81 %. So only the last few picks lean on the scan, and the scan favours the
free value that follows a long run of taken ones. [inference, arithmetic]

**What the live runs showed** (8 and 9 Nov, two game launches; fight numbers are the arena's list order). [live B52]

| Run | Picks, in order |
|---|---|
| 1 | 1 The Last Giant, 32 Sinh |
| 2 | 15 Najka, 12 Mytha, 27 Ancient Dragon |
| 3 | 9 Skeleton Lords, 6 Ruin Sentinels, 4 Old Dragonslayer, 39 Lud and Zallen |
| 4 | 16 Royal Rat Authority |

Each run ended with a death. The first picks of the four runs (1, 15, 9, 16) came from the roll made at Majula; the others from
rolls made in the arrival maps. A run with shuffling off went in order, as before. B57 (16 Nov) ran another shuffled
rush through four fights, but its watcher did not log the picks.

## Pitfalls and what failed

Nothing about the dice failed in the live runs. These are the traps we met or designed against.

- **A single result can prove nothing.** The first pick of run 1 was fight 1, which is also what the in-order fallback gives.
  It could have been the fallback or a one-in-39 roll. The second pick (fight 32) settled it. Choose a test where a working
  roll and your fallback differ, and look at two results. [live B52]
- **"Taken" flags outlive the run.** They are global, and the watcher printed the last run's picks every time it reattached.
  Clear them at the start. [live B52] One log line (9 Nov, 10:00) showed a second flag on and straight off in the same second
  as a map change, with the run flag off, so the picker cannot have run; we read it as the watcher reading mid-load. [live B52];
  reading [inference]
- **A neighbouring map runs its scripts too.** The picker waits for the player to be in its own map, and a neighbour that
  streamed in during one fight (`m10_32` with `m10_33`) did not trigger its picker. [live B52; see B39c]
- **A roll can match no condition.** Majula's cat group rolls 0 to 50 and tests bands 1-9, 10-19 up to 40-49, with no
  catch-all. A roll of 0 or 50 matches none. The likeliest result is an idle group, "no cat this time", but we did not test
  it. [data]; effect [inference]
- **Roll and read the same slot.** Majula's armour-shop pick rolls `1:1(1, 0, 19)` and then reads `f1(0)`. Either that is a slip
  that happens to work, or slots are not what we think. Until the slot rules are known, read the slot you rolled. [data]
- **Two groups can use slot 0.** The picker rolls slot 0 in arrival maps that also run the Forlorn draw, which uses slot 0
  too. We saw no clash, but never looked for one. [untested]

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| `1:1(0, 1, 39)` rolls and both ends occur | Fight 1 on 8 Nov (run 1) and fight 39 on 9 Nov (run 3) | [live B52] |
| `f1(0) == k` is true in a condition | Ten picks in four runs, out of list order; the first picks 1, 15, 9, 16 over two launches; the in-order fallback would give 1, 2, 3 | [live B52] |
| A roll made in a Majula group and in the arrival maps both work | First picks at Majula; the later ones in the arrival maps | [live B52] |
| A neighbouring map did not trigger its picker | `m10_32` + `m10_33` loaded during one fight, no extra pick | [live B52] |
| The Forlorn's two draw groups and their arguments | A dump of `event_m10_16` and the `.edd` labels, matched in the 14 maps | [data] |
| 23 uses of `1:140531`, in 20 distinct groups of the 28 event scripts | A walk over the 28 scripts, 18 Nov | [data] |
| 14 draw groups register flag 109990; nothing sets it | The same walk; the project's Forlorn survey and the flag scan | [data] |
| `1:1` E 39, T 64; `f1` E 68, T 318 | The walk again, matching the command table page | [data] |
| `f1` is read in 26 event conditions, all in Majula's script, and in 16 event command arguments | The same walk | [data] |
| `1:140531` gives N + 1 % with a 0-99 roll | Reading of the argument shapes; the project's Forlorn survey gives 15 % and 40 % | [inference]; never run [untested] |
| Eight rolls then a scan always ends; an unreadable `f1` gives an in-order run | An offline walk of the built state table with fake rolls | [inference, not the game] |
| No sign of a fixed seed per launch | First picks differ between the two launches (1, 15 and 9, 16). Four values say nothing about how even the dice are | [live B52], weak |

## Tools

- `tools/bossrush.py`: `picker_spec()` builds the no-repeat picker; `ROLLS` and `SLOT` are its two settings.
- `tools/shufflewatch.py`: read-only log of the maps, the run flags and each pick in order, so a run's order can be read back.
- `tools/offline_tests.py`: walks the built picker with fake rolls (400 random starts) before any game test.
- `edd.py`, `esd.py` (the project's earlier research tools, not in `tools/`): `esd.py dump` prints a script's groups and
  commands, `edd.py` prints the developer names and labels. Print the file with UTF-8 output (`python -X utf8`), because the
  labels are Japanese.

## Open questions

- **Slots.** How many random slots exist? Are they per script, per group, or global? Does one group's roll overwrite another's
  between a roll and its read?
- **A same-state read in an event group** (roll in an entry, read in that state's conditions), and `f1` in the same entry
  as the roll. The AI scripts do both. [untested]
- **The limits of `1:1`.** `max` is inclusive at 39; the 0-99 case is assumed. What happens with `min > max`, negative values or a
  huge range? [untested]
- **`1:140531` on our own roll.** Does the compare give N + 1 %? Which comparison numbers exist? A probe group and a read-only
  flag watcher would answer both; flag 109990 would show whether the Forlorn's OR reading is right. [untested]
- **Online play.** Each machine seems to roll for itself, since the draws use `f140001` and not `f140002`. Whether any roll is
  shared in multiplayer is unknown.
- **The long run.** A full 39-pick sequence, the 79-condition state, and how strongly the late picks lean on the scan: none was
  reached live. [untested]
- **A tidier draw.** Roll once in `1..left` and walk to the k-th free value. It needs a counter, and area variables were never
  tried. [untested]
- **The random waits** (`f4`, `1:140511`, `1:131705`, `1:131713`) from the table above. [untested]

## Related pages

- [Event and talk script commands](script-command-reference.md): the command rows for `1:1`, `f1` and `1:140531`
- [Event script groups](event-script-groups.md): rows, the empty first state, "tried in order", waiting for the map
- [Event flags and safe ids](event-flags-and-free-ids.md): the two kinds of flag and a free run for "taken" flags
- [Writing ESD files](writing-esd-files.md): how states and conditions are written, and what a group costs
- [Warping the player by script](../modding-world/warping-by-script.md): the warp each pick ends with
- [Detecting a player death](detecting-death-in-scripts.md): how a death ends a run of picks
