# Adding new dialogue options: the N-choice list window (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.
> Test ids such as B36c point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags: `[live B36c]` seen in the game,
> `[code]` read in the exe (Ghidra `FUN_14xxxxxxx`, image base 0x140000000), `[data]` read in the game files,
> `[inference]` reasoned, not proven, `[untested]` an idea only.

## Summary

- **An NPC's own menu cannot take a new entry.** Its entries are the game's fixed functions, switched on by bits in
  `NpcEventParam`, each with one fixed text in `npcmenu.fmg` [data; live B31]. The route that works: hang a question off
  the NPC's **Talk** entry, then open the game's **N-choice list window**, `1:130481`, over rows you add to
  `PluralSelectMenuParam`. [live B36c, B57]
- **A row is 20 bytes**: a label id (a `pluralselect.fmg` string), a soul cost, **the event group to start when the row
  is picked**, a hide flag and a mode byte. A pick starts that group (its `EventParam_<map>` row copied from Gilligan's
  ladder row 111095); the group sets flags the talk script waits for. [live B36c, B57]
- **The trap: the window never appeared, and nothing said why.** Opened from Talk while the NPC's menu is still open
  behind the conversation, the list is refused in the exe (front-end busy word +0x3B8 holds bit 0x8) and the refusal is
  silent. The cure is four commands just before the list: `1:130482; 1:130442; 1:130462; 1:130464`, then `1:130481`.
  It took three builds: B36 FAIL, B36b FAIL, B36c PASS. [live; code]
- **Limits:** 12 rows per list [code], so long lists are split into pages (the mod uses pages of 10 or fewer). PC ignores
  the title and key-guide arguments and always uses npcmenu 80000 and 89901 [code; live B37]. The confirm window after a
  pick always shows a soul cost (npcmenu 89101) [code; live B51]. A row shows only while its flag is OFF [code; live B36c].
- **Once it opens it is cheap to reuse.** A list reopens as often as you like in one conversation, and the same row
  group runs again on a second pick, so one list can be a toggle menu. [live B36c, B57]

### 1. Why the NPC's own menu cannot take new entries

The menu you get when you talk to an NPC is the game's NPC window, opened with `1:130461`
(【FE】NPCウィンドウを開く, "open the NPC window") and one `NpcEventParam` row number. The row decides which entries show:
`Unk00`, `Unk01` and `Unk03` are bit masks with one bit per fixed function (Buy, Reallocate, Repair, Enter covenant,
Talk, Level up and so on). Each function has one fixed string in `npcmenu.fmg` (ids 10101-10123) and returns one fixed
code from `f130452()`. [data; inference from all 73 `NpcEventParam` rows and every talk script's handlers]

Two live results show the limit. Setting `Unk00` bit 2 on the Herald's row added "Reallocate", and `Unk03` 1 -> 9 on a
smith's row added "Infuse" [live B31]. Both are existing functions with existing text. No bit means "new entry" and
the script cannot set an entry's text, so a talk script can only react to the codes the window offers. More in
[NPC menus](npc-menus.md).

A top-level entry of our own would mean repurposing one fixed entry (a spare bit, a regulation edit and a global rename
of its text, which every other NPC with that bit would show too) [untested; design only]. The N-choice list needed no
such trade. So the choices go behind the **Talk** entry, which every NPC has (code 16; the shared menu library returns 2
for it) [data].

### 2. The window, its commands and its functions

Commands (`1:` bank) do things, functions (`f`) read state; they are separate namespaces (`f130452` is not
`1:130452`). Names are the developer names in the map's `.edd` file.

| Id | Developer name | Meaning here |
|---|---|---|
| `1:130481(first, last, 0, title, key guide)` | 【FE】N択ウィンドウの表示 "show the N-choice window" | Opens a list over `PluralSelectMenuParam` ids first..last. The third argument is 0 in every call, Gilligan's and ours (meaning unknown). Title and key-guide ids are ignored on PC. |
| `1:130482()` | 【FE】N択ウィンドウを閉じる "close the N-choice window" | Acts only while the talk's window tracker is in state 5 (a list). |
| `f130423()` | 【FE】N択ウィンドウの結果取得 "get the N-choice window's result" | 1 after a pick, 3 = cancelled (Back). It does not say which row. |
| `f130433()` | 【FE】N択ウィンドウ表示判定 "is the N-choice window shown?" | Non-zero while a list is up. |
| `1:130442()` | 【FE】YES／NO選択ウィンドウを閉じる "close the YES/NO window" | Acts only in tracker state 1. |
| `1:130462()` | 【FE】NPCウィンドウを閉じる "close the NPC window" | Acts only in tracker state 6 (the NPC menu). |
| `1:130464()` | 【FE】NPCウィンドウで「何を選択したか」をクリア "clear what was chosen in the NPC window" | Clears the choice. Gilligan's own script does it before his list. |
| `f130452()` | 【FE】NPCウィンドウの結果取得 "get the NPC window's result" | 16 = Talk was chosen; 19 = the menu was cancelled; **23 = a list has just closed** (pick or Back). |
| `f130422()` | 【FE】YES／NO選択ウィンドウ表示判定 "is the YES/NO window shown?" | Used to wait until a question has gone. |
| `f130031(group)` | 【システム】イベント終了判定 "has the event ended?" | Gilligan's script waits on his row's group with it. We use flags instead. |
| `1:130101(flag, value)` | 【フラグ】フラグを設定 "set a flag" | How a row's group tells the talk script what was picked. |
| `f6(flag)`, `f3()` | フラグの取得 "get a flag"; ステート経過時間の取得 "time in this state" | The conditions the talk states use. |

Gilligan's ladder prices are the game's only user of this window [data: 538 scripts scanned, B51]. The vanilla scripts
open it once, from his "Set up ladder" menu entry (group 2147483582 in `talk_m10_04`). His flow: clear a "done" flag, a
spoken line, `1:130481(10, 12, 0, 80000, 89901)`; cancelled = `f130423() == 3`; a pick starts the row's event group,
which sets a flag; the talk script closes the window with `1:130482` and speaks again. [data]

### 3. One row of `PluralSelectMenuParam`

20 bytes, five 4-byte fields [data]. The offsets are the ones the exe reads [code: `FUN_1400c4830`, `FUN_1400c5110`].

| Offset | Field | What the window does with it | Gilligan's row 10 | The mod's rows |
|---|---|---|---|---|
| +0x00 | `ladder_unlock_index` | **The label**: the id of the string in `pluralselect.fmg` [live B36c] | 10 (his rows use 10 / 20 / 30) | the row's own id |
| +0x04 | `soul_cost` | Shown in the confirm window and compared with the player's souls [code; live B51] | 500 | 0 |
| +0x08 | `esd_event_id` | **The event group started on a pick** [live B36c] | 111095 | 990200 and up |
| +0x0C | `global_event_flag` | The row's hide flag [code; live B36c] | 102170 | one flag per row |
| +0x10 | `Unk04` | 1 = shown while the flag is OFF; 0 = shown while it is ON [code]. All our rows use 1 | 1 | 1 |

Gilligan's three rows have ids 10, 11 and 12 (costs 500 / 3500 / 12000, groups 111095-111097, flags 102170-102172).
`ladder_unlock_index` is a poor name: it is simply the text id of the label.

A flag of -1, a flag of 0 and a never-set flag all show their row; a flag that is ON hides it. [live B36c: the four
test rows "flag -1", "flag 0", "never set" and "costs 1 soul" showed and ran their groups, "flag set on" did not
show]. The `Unk04` = 0 branch (show while ON) is in the code but no row of ours used it [untested].

### 4. What one list does, step by step

1. **Open.** The talk command goes through the script command handler (jump table at 0x14046db80, 45 entries, a byte
   map after it, index = command id - 130307; the same table serves event scripts; 130481 lands at 0x1404634cb). The case reads the arguments, builds the list's
   parameter block with `FUN_1400c47c0(first, last)` and calls `FUN_140198ed0`. That calls the front-end open function
   `FUN_1404fe810` and sets the talk's window tracker to **state 5**. [code]
2. **Collect rows.** `FUN_1400c4830` walks every id from `first` to `last`. A missing id is skipped [code; untested
   live]. A row whose flag test fails is skipped (see `Unk04`). The rest go into a fixed 12-slot vector. Rows are
   collected once, **when the list opens**: the trace logs 5, 10, 9 and 4 rows at the four kinds of opening in B36c.
3. **Show.** Labels from `pluralselect.fmg`, the title from npcmenu 80000, the key-guide strip from npcmenu 89901,
   whatever the script passed (`FUN_1400c47c0` fetches those two ids itself). [code]
4. **Pick.** The window asks for confirmation with its own YES/NO (npcmenu 89101, `FUN_1400c5110`). NO returns to the
   list. YES closes the window and **starts the row's event group** [live B36c, B51].
5. **Result.** `f130423()` reads 1 and `f130452()` reads 23 [live B36c]. Back (the cancel button) closes the list with
   `f130423()` = 3 [live B36c]. The window **never sets the row's flag**: hiding a row after a pick is the group's job
   [live B36c: the "costs 1 soul" test row's flag stayed 0 after its pick].

The window is not tied to the NPC that opened it. The Herald's Talk opened our rows, and Gilligan's talk opens his; the
code checks no `NpcEventParam` row [code, B36b notes; live B36c]. The event group must exist in the **current map's**
event script, because that is where the window starts it: all our lists open in Majula, so all their groups live in
`event_m10_04_00_00` [data].

### 5. Why a list opened from Talk is refused (the exe side)

All of this is on the front-end (FE) manager, `[GameManagerImp+0x22E0]`, which every FE window goes through.

| FE offset | What it holds | How we read it |
|---|---|---|
| +0x32C | mode: 0 YES/NO object, 2 NPC menu / list operator, 3 talk, 5 none | [code, B36b notes; `FUN_1404fe810` compares it with 2] |
| +0x110 | the NPC-menu operator object (`FeOperatorNpcMenu`); a list is one of its modes | [code] |
| +0x128 | the NPC window's result, i.e. `f130452()` | [code, live trace: 16 / 19 / 23 match] |
| **+0x3B8** | **busy bits** | [code] |

The talk side has its own tracker at `[[GameManagerImp+0x70]+0x50]`, with the state at +0x54: 0 none, 1 YES/NO,
5 list, 6 NPC menu (others exist). Byte +0x5B means "an NPC menu is underneath"; `1:130461` sets it. [code, B36b notes;
the states and +0x5B as the closes use them re-read 17 Nov 2026]

**The refusal.** `FUN_1404fe810` (at 0x1404fe8df) does `test eax, 0x100001e` on +0x3B8 and returns 0 without opening if
any of those bits is set. Otherwise it sets `0x1000009` and creates the list window. [code]

**Who sets the bit.** The NPC menu's own open function (`FUN_1404fe650`; the chain is `1:130461` ->
`FUN_140199020` -> `FUN_1404fe650`) refuses on `0x1000016` and sets `0x1000009`, which leaves **bit 0x8**
("the NPC-menu operator is open") set. After you choose Talk, her menu does not close: it stays alive behind the
conversation, so bit 0x8 stays set all through the talk. [code, B36b notes; the two masks re-read 17 Nov 2026]

**Why the questions worked and the list did not.** The talk's YES/NO question showed up with 0x8 already set and left
the word unchanged: the trace shows no change of bits between her menu opening and the question closing. [live B36c
trace] The list open checks the word; the question does not. So every YES/NO worked while the list never did.

**The refusal is silent.** The caller of the open function, `FUN_140198ed0`, does not test its return value. It sets the
tracker to state 5 whatever happened. A refused list therefore leaves the talk script waiting for a window that does not
exist, with the tracker in "a list is open". That is the first build's symptom: YES to the question, no window, the
conversation stuck, quit to the title to escape (B36). [code, read 17 Nov 2026; live B36]

**What clears bit 0x8.** The tracker setter `FUN_140199e40`, when it steps *into state 0*, clears the "underneath" byte
and, if the operator is open, calls `FUN_1404fecd0`, which does `and [FE+0x3B8], ~0x8`. A mode switch of the FE (a
spoken line) is the other way: every route to Gilligan's list in his script passes a spoken line first [data], which is
why his own list always opens. [code for the first; the spoken line as a reset is from the B36b analysis, not re-traced]

**The four closes.** Each command checks the tracker state and does nothing in any other state, so they are safe to
run in a row, before every list. [code: `FUN_1401995a0`, `FUN_140199510`, `FUN_1401995d0`]

| Command | Acts when the tracker is in | Moves it to |
|---|---|---|
| `1:130482` close list | 5 | 6 if an NPC menu is underneath (+0x5B), else 0 |
| `1:130442` close YES/NO | 1 | 6 if an NPC menu is underneath, else 0 |
| `1:130462` close NPC window | 6 | 0 (which clears bit 0x8) |
| `1:130464` clear what was chosen | any | no state change |

Run in that order, whatever is up ends in state 0 with bit 0x8 clear, and `1:130481` is accepted. [live B36c]

**A trace of one conversation** (`tools/fltrace.py`, read-only, B36c; times in seconds from the Herald's menu):

| t | What was logged |
|---|---|
| 0.0 | Menu up: bits 0x8, mode 2, result 16 (Talk), tracker 1 (the list question is up) |
| 1.6 | Tracker 6: the question answered YES, her menu still underneath |
| 3.5 | **List up**: bits 0x1000008 then 0x8, operator in list mode, 5 rows, tracker 5, "list open" flag on |
| 8.9 | A category picked: bits 0x0, result 23, window gone |
| 9.4 | The page opens, 10 rows, 0.5 s after the last list closed |
| 13.9 | A fight picked: "picked" flag on, window gone, result 23 |
| 19.0 | FE mode 5, nothing open; the map's flags read as unavailable: the warp has started |

### 6. Texts

| Text | Where | Notes |
|---|---|---|
| Row labels | `pluralselect.fmg`, id = the row's `ladder_unlock_index` | The file has one range (10-30) as shipped, with strings at 10, 20 and 30 only: ids 11-19 and 21-29 sit inside it and are free, so `set` can fill them [data]; never run live [untested]. A new id outside the range must open a new range: `tools/fmgwrite.py` `add_ids` does it; `set` refuses an id outside every range. [data] |
| List title | `npcmenu.fmg` **80000**, always | Shown for every list. The original English is "Pay how much?"; the mod rewrote it to "Select an option". [live B37] |
| Key-guide strip | `npcmenu.fmg` 89901, always | Gilligan's. [code] |
| Confirm after a pick | `npcmenu.fmg` **89101**, always | The original shows "Souls needed: %d" with the row's cost. The mod rewrote it to "Confirm selection?" (no number). [live B51] |
| The question that leads to the list | `mapevent.fmg`, a free id | Same as any YES/NO text; see [YES/NO windows](yes-no-windows.md). |

Passing your own title id does nothing on PC. The mod added ids 80001 and 80002 and passed them; the lists were still
titled by 80000. [live B37] The only text lookup in the exe that asks for 80000 is in the list window
(0x1400c47fc), and all 1,089 calls of the text getter (0x14003d870) were checked, so renaming 80000 renames Gilligan's
ladder window and the mod's lists, nothing else. [code; live B37] The same holds for 89101: the only code that asks for
it is `FUN_1400c5110` (at 0x1400c5181), which always shows it and compares the player's souls with the row's cost even
at 0. [code; live B51] **If your mod keeps Gilligan's ladder, you cannot give it and your lists different titles or
confirm text.** The Boss Arena removed his "Set up ladder" entry, so nothing shows the old wording.

Label width: the widest list row shown without trouble was 571 px of the front-end font (`FeFont_Big`, English) [live
B36c]. Nothing wider was tried. For the YES/NO question text the crash limit is known: more than 127 characters
crashes the window; the widths are only a display precaution. See [Text length limits](text-length-limits.md).

## How to do it

This is the recipe the Boss Arena uses for a list in the Herald's Talk. The ids are the mod's; yours will differ (see
[Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md)).

1. **Plan the lists.** One list shows at most 12 rows. Split anything longer into pages behind a category list, as the
   mod does: 5 categories, then pages of 10 / 10 / 9 / 4 / 9 for 42 fights. The cap counts rows that *pass the flag test
   when the list opens*, not the id range [code; untested].
2. **Give every row an id and a group id.** The rows of one list must be a consecutive id range (`first`..`last`).
   The mod: categories 9000-9004, fights 9100-9141, options 9300-9308 (the label ids are the same numbers); event groups
   990200-990204, 990300-990341, 990500-990508.
3. **Add the rows to `PluralSelectMenuParam`** (a loose param file; see [Editing params](../modding-scripts/editing-params.md)). Copy
   Gilligan's row 10 and set the five fields: `ladder_unlock_index` = the row id, `soul_cost` = 0, `esd_event_id` = the
   row's group, `global_event_flag` = the row's hide flag (or -1 for a row that is never hidden [live B36c]), `Unk04` = 1.
   The mod builds all rows with `tools/paramrows.py` (`set_fields`) inside `tools/fightlist.py` `plural_rows`.
   `[live B36c]`
4. **Add the labels to `pluralselect.fmg`** for every language you ship, with `fmgwrite.add_ids`. A language without
   them is untested [untested]. Add the question to `mapevent.fmg`.
5. **Add one `EventParam_<map>` row per row group**, copied from Gilligan's row 111095 with `event_id` changed. Here that
   is `eventparam_m10_04_00_00.param`. The copied row is: `flag_id` 10044000, `Unk0B` 3, `Unk0C` 1, `Unk0D` 5, the rest 0.
   A load-time group uses a different kind (1 / 0 / 6, flag 0). Whether a menu group needs the 3 / 1 / 5 kind is not known
   [untested]; copying is what was tested. [data; live B36c]
6. **Write one event group per row in the NPC's map event script.** Each has the same four states (the first state may
   hold no commands: [Event script groups](../modding-scripts/event-script-groups.md)):

   | State | Commands | Moves on |
   |---|---|---|
   | `s0` | none | to `s1` always |
   | `s1` | none | to `go` if `f6(LIST_OPEN) != 0`, else to `end` |
   | `go` | the row's work, then `1:130101(PICKED, 1)` | nowhere |
   | `end` | none | nowhere |

   The guard on `LIST_OPEN` makes a group that is started any other way do nothing. A fight row's `go` does more:
   it sets its own hide flag to 0 and `PICKED` to 1, waits 2 s (`f3() > 2`) so the talk can close its windows, then sets
   the respawn and warps ([Warping by script](../modding-world/warping-by-script.md), [The respawn point](../modding-world/respawn-point.md)).
7. **Write the talk states** (below).
8. **Add a load group** for the map that clears `LIST_OPEN`, `PICKED` and every row and category flag at each load, so
   no flag left on by an interrupted conversation marks a row. (The mod's group 990210 does this.)
9. **Install.** Rules params (`PluralSelectMenuParam` is one) and FMGs are read at game start: install them with the
   game closed and restart [inference for the params]. Per-map params, such as the `EventParam_<map>` row of the pick
   group, and the talk and event scripts are re-read at the next load of the map, with no restart [live B13, B54 for
   per-map params; B20d, same game process, for scripts]. (The B36 install
   note told the player to restart for scripts too; that was caution, and B20d's earlier result is the one to trust.)
   Then test every list, its Back button, a second visit, and a pick of each row.

### The talk states for one list

Build these inside the NPC's menu machine, after the Talk entry (the Herald's is group 2147483574). In the mod, NO to
every question falls through to her usual Talk line, so nothing of hers is lost.

| State | Does | Leaves for |
|---|---|---|
| `ask` | `1:130465()` (clear "did something"), then the YES/NO library with the question | YES -> `wait`; NO -> the next question or her normal Talk; Back -> her menu; walked away (`f130452() == 19`) -> abort |
| `wait` | nothing | `open` when `(f3() > 0.5) && (f130422() == 0) && (f130433() == 0)`, or after `f3() > 2` |
| `open` | `LIST_OPEN` = 1, `PICKED` = 0; then **`1:130482; 1:130442; 1:130462; 1:130464; 1:130481(first, last, 0, title, key guide)`** | cancelled (`f130423() == 3`) -> `cancel`; `PICKED` on -> `closing`; walked away -> `walk`; list never shown (`f3() > 6 && f130433() == 0`) -> `fail` |
| `cancel` | `LIST_OPEN` = 0 | her menu, once the window has gone (same wait as `wait`) |
| `closing` | `1:130482` | the next state, once the window has gone |
| `walk` | `1:130482`, `LIST_OPEN` = 0 | abort (her menu closes) |
| `fail` | `1:130482`, `LIST_OPEN` = 0, a "did not open" flag on, an OK window with the text | back |

The `fail` and `walk` exits are insurance. They stayed unused in the passing build (the "did not open" flag never
went on in B36c). They came from B36b, where a list that never opened would otherwise have locked the conversation:
with them B36b ended without a lock-up. [live B36b, B36c]

The wait before `open` is 0.5 s of quiet, 2 s at most. In B36c the list came up about 2 s after the answer. Whether the
wait is needed once the four closes are in place was not tested separately [inference: it is harmless].

### Pattern A: a cascade (categories, then pages)

The categories list is one list of 5 rows. A category row's group does only `1:130101(CAT_FLAG_k, 1)` (and sets its own
hide flag to 0). The talk script, in `open`, also watches `f6(CAT_FLAG_k) != 0` for each k. It then:

1. goes to `go_k`: `1:130482` (close the categories), waits for the window to be gone;
2. goes to `opened_k`: clears `PICKED` and the category flag, runs the four closes, opens that category's page with its
   own `first`..`last`;
3. a page's cancel goes back to the categories: wait for the window to go, then `open` again;
4. a page's pick (`PICKED` on) goes to `closing` then `leave` (abort: her menu closes), and the fight group warps 2 s later.

A category can be picked twice in one conversation: the same group simply runs again. [live B57; B36c's own
"same category twice" step is not visible in its trace]

### Pattern B: a toggle list (on / off settings)

Because a row hides while its flag is ON, a toggle needs **two rows per setting** and **two flags**: a setting flag and a
complement kept opposite to it.

| Row | `global_event_flag` | Shown when | Its group |
|---|---|---|---|
| `<setting>: OFF` | the setting flag | the setting is OFF | sets the setting ON, the complement OFF, `PICKED` = 1 |
| `<setting>: ON` | the complement flag | the setting is ON | sets the setting OFF, the complement ON, `PICKED` = 1 |

After a pick, the talk script closes the list and **opens it again** (`closing` -> `reopen` -> `open`), because rows are
collected only at open: the new state shows only in the new list. Cancel goes back to her menu. A load group in the
map derives the complements from the saved settings at every load. [live B57 step 3: Restore picked three times in
one visit, ON, OFF, ON, OFF, the list reopening with the new state each time; the group runs again every time]

The mod shipped this with three settings (6 rows, 9300-9305; B57). A later change adds a three-way setting as one row per
mode, where a pick moves to the next mode (rows 9306-9308): built, [untested] (B59).
`Unk04` = 0 rows would remove the complement flags (show while ON) [untested].

## Pitfalls and what failed

1. **B36, first build, FAIL (7 Nov).** YES to the list question: no window, the conversation stuck, nothing worked, not
   even the Herald; quit to the title to get out. The idea at the time: the list opened in the same moment the YES/NO was
   closing. Wrong. Its cause is the exe-side refusal above.
2. **B36b, FAIL on all three tests.** Every list now waited for the question to be gone, had a walk-away exit and a 6 s
   timeout, and two probe questions were added: P1 opened *Gilligan's own rows* from her Talk, P2 opened the fight list
   with Gilligan's title. All three failed, but nothing locked up. Gilligan's own question, asked of Gilligan, still
   opened his list. So the cause was not our rows, not our texts, not the timing: it was opening from her Talk at all.
   The probes are what made that readable. [live B36b]
3. **The exe analysis** (headless Ghidra plus a disassembler) found bit 0x8 and the silent refusal, section 5.
4. **B36c, PASS (7 Nov).** The four closes before `1:130481`. Five conversations in the trace, each: Herald's menu, Talk,
   list question, list window; rows collected 5 (categories), 10 / 10 / 10 / 10 (pages), 9 (DLC page), 4 of 5 (test
   list: the "set on" row hidden, as the code said); the test list opened five times in a row (four picks, then Back) and
   the categories came back after it; four picks set `PICKED` and warped. [live B36c; fltrace log]
5. **A list of 30 or 39 fights.** The first build had "All 39 fights" and "Base game 30" lists. The 12-row cap
   (below) forced the split into pages before any of them ran. [code; B36b]
6. **The 13th row.** In `FUN_1400c4830` the row count is tested after each flag test; the 13th row jumps to an assertion
   routine (`FUN_140832cb0`, message "out of memory.", from a fixed-capacity vector)
   and the byte after the call is `int3`. Treat it as a crash. [code, read 17 Nov 2026; never run live]
7. **Titles that do not show.** The mod passed its own title ids (80001 and 80002). The code ignores them, so every list
   would carry npcmenu 80000, Gilligan's "Pay how much?"; once 80000 was rewritten, both lists showed the new text
   (B37). The only way to change the title is to change 80000, for every list. [code; live B37]
8. **A confirm window that says "Souls needed: 0".** The window's own confirm shows the cost even at 0; no field hides it.
   The mod rewrote npcmenu 89101 in all 11 languages (B51). [code; live B51]
9. **The result does not say which row.** `f130423()` is 1 for every pick. Use one flag per row, set by the row's group.
   [live B36c: only the `== 1` test matched across different test rows]
10. **The window does not set the row's flag.** If a row should vanish after a pick (as Gilligan's ladders do once built),
    the row's group must set the flag. [live B36c]
11. **A stale list.** Rows are fixed when the list opens, so a flag changed while a list is up does not change that
    list [code]. Reopen it to show the new state. [live B57]
12. **A pick can run its group twice.** The notes expected "a group cannot be restarted in one visit"; B57 showed it can.
    [live B57]
13. **Other languages.** B37 renamed the English title only; other languages kept "Pay how much?" until the i18n build
    (B51 says the title was in all 11 languages by then). A label missing from a language's `pluralselect.fmg`: not tested.

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| NPC menu entries are fixed functions chosen by `NpcEventParam` bits; a new entry cannot be added from script | data (73 rows, all talk handlers); adding a bit added only an existing entry | B31; `majula-menu` findings |
| A list opens from the Herald's Talk over new rows | live: five conversations traced, picks warped | B36c |
| Rows = every id first..last, flag-tested, 12 slots, missing ids skipped | code (`FUN_1400c4830`); counts 5 / 10 / 9 / 4 in the trace | B36b notes, re-read 17 Nov; B36c |
| A 13th row asserts | code only: assertion, then `int3` | B36b notes, re-read 17 Nov; never run |
| Title and key-guide arguments are ignored | code (`FUN_1400c47c0` fixed ids 80000 / 89901); live: title ids 80001 / 80002 never showed | B36b, B37 |
| `Unk04` = 1 hides a row while its flag is ON | code; live: the "set on" test row never showed | B36c |
| The window does not set a row's flag | live: the "costs 1 soul" row's flag stayed 0 | B36c |
| `f130423()` = 1 pick, 3 cancel; `f130452()` = 23 when a list closes | live, flags and trace | B36c |
| A pick starts the row's event group, kind 3 / 1 / 5 EventParam row copied from 111095 | data and live | B36c, B57 |
| The same group runs again on a second pick in one visit | live | B57 step 3 |
| A list reopens many times in one conversation | live trace: five openings in a row | B36c |
| The list open is refused while FE +0x3B8 has any of 0x100001E | code (0x1404fe8df) | B36b notes, re-read 17 Nov |
| The refusal is silent; the caller still sets tracker state 5 | code (`FUN_140198ed0` ignores the return) | read 17 Nov |
| The Herald's NPC menu sets and keeps bit 0x8 through Talk | code (`FUN_1404fe650`, mask 0x1000016 / sets 0x1000009); trace shows 0x8 during the question | B36b notes; B36c trace |
| The talk YES/NO leaves the busy bits alone | live trace: bits constant during the question | B36c trace |
| Tracker step into state 0 clears bit 0x8 (`FUN_140199e40` -> `FUN_1404fecd0`) | code | B36b notes, re-read 17 Nov |
| `1:130482` / `130442` / `130462` act only in tracker states 5 / 1 / 6 | code (`FUN_1401995a0`, `FUN_140199510`, `FUN_1401995d0`) | B36b notes, re-read 17 Nov |
| The four closes make the list open | live: B36c passes with them | B36c |
| The B36 / B36b waits alone were not enough | live: B36b failed with the wait and the probes | B36b |
| Renaming npcmenu 80000 renames only the list window's title | code: one lookup at 0x1400c47fc, 1,089 calls checked; live | B37 |
| npcmenu 89101 is always shown and compared with the row cost | code (0x1400c5181); live: reworded text showed | B51 |
| Gilligan's ladder list is the window's only other user | data: 538 scripts scanned | B51 |
| A 571 px row label showed fine | live | B36c (`tools/i18n.py` `NAME_PX`) |
| Rules params / FMGs need a restart; scripts and per-map params are re-read at the next map load | live for scripts (B20d), per-map params (B13, B54) and a bonfire-name FMG (B12); rules params by inference | B20d, B13, B54, B12, B36 |
| The three-way "After a win" row cycles | built only | B59 [untested] |

## Tools

| Tool | What it does for this topic |
|---|---|
| `tools/fightlist.py` | The cascade: rows, labels, groups, EventParam rows, talk states, texts. `fightlist.py` alone prints every row with its group and flag. `MAX_ROWS = 12` and `check()` refuse a list over the cap. |
| `tools/options.py` | The toggle list (Pattern B): paired rows, complement flags, the load group, the talk states. |
| `tools/choices.py` | `menu_talk()` threads the lists into the Herald's menu machine; `spec_group()` builds an event group from a list of states. |
| `tools/fightlist_check.py` | Checks a build: texts, groups, EventParam rows, the talk windows. `--game` checks the installed copy, read-only. |
| `tools/paramrows.py` | Reads a param, copies a row, `set_fields`. |
| `tools/fmgwrite.py` | `add_ids` adds new ids (and ranges) to an FMG. |
| `tools/looseparams.py` | Rebuilds the loose params (`REBUILD`): `PluralSelectMenuParam.param` is rebuilt by the fight list's module. |
| `tools/fltrace.py` | **Read-only live trace** of the FE: polls every ~10 ms and logs each change of the busy bits (+0x3B8), FE mode (+0x32C), operator (scene, mode, rows collected), NPC window result (+0x128), talk tracker state (+0x54) and a chosen set of flags. `fltrace.py [minutes] [--flags a,b,...] [--no-scan]`. The log is `tools/fltrace-<date>.log`. |

The talk and event scripts are read and written with the project's ESD tools (see [Writing ESD files](../modding-scripts/writing-esd-files.md)).

## Open questions

- **Does a non-zero `soul_cost` take souls?** All the mod's rows are 0. A "costs 1 soul" test row ran, with no reported
  problem, but nobody measured the player's souls. [untested]
- **Does the pick group's EventParam kind matter** (3 / 1 / 5 against 1 / 0 / 6)? We always copy 111095's. [untested]
- **Can a list open from another NPC in another map?** The groups would have to be in that map's event script. [untested]
- **Walking away with a list up.** The exit tests `f130452() == 19`; 19 was seen live when her menu itself was
  cancelled, never with a list open. The `walk` and `fail` exits have not fired in a live run. [untested]
- **The third argument** of `1:130481` (0 in every call). The talk code reads it as a number; the list window does not
  seem to use it. [untested]
- **A list with no question before it**, straight from Talk. The four closes should make it work. [untested]
- **What the player sees.** B36c asked what the window shows for a cost of 0 (a price column?) and whether a 10-row
  page scrolls; the answers were not written down (the test passed). Both are unrecorded.
- **Is `1:130464` needed**, or is the fix the first three closes alone? It was kept because Gilligan's script does it. [untested]
- **Is the wait before `open` needed** with the four closes in place? [untested]
- **`Unk04` = 0** (a row shown only while its flag is ON). The code does it. [untested]
- **A language missing the label ids.** [untested]
- **Gilligan's route** (a spoken line before the list) as an alternative to the closes. Never needed. [untested]

## Related pages

- [Asking the player a question: YES/NO windows](yes-no-windows.md): the question that leads to the list, and its timing.
- [NPC menus: option bits, Talk, first meetings, respec](npc-menus.md): the fixed entries and how to switch them on.
- [Event script groups](../modding-scripts/event-script-groups.md): the row groups, their EventParam rows and the "no commands in the first state" rule.
- [Warping by script](../modding-world/warping-by-script.md): what the fight list's groups do after a pick.
- [The respawn point](../modding-world/respawn-point.md): the respawn the list sets before it warps.
- [Event and talk script commands](../modding-scripts/script-command-reference.md): every command and function named here.
- [Editing params](../modding-scripts/editing-params.md): loose params and adding rows.
- [Adding and translating game text](game-text-and-languages.md) and [Text length limits](text-length-limits.md): the FMGs, the title and confirm text.
- [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md): choosing safe flag and group ids.
- [Writing ESD files](../modding-scripts/writing-esd-files.md): building the talk states and groups.
- [Restoring HP, Estus, spells and durability by script](restoring-hp-and-estus.md) and [Difficulty levers](../modding-world/difficulty-levers.md): what the options list's rows switch.
