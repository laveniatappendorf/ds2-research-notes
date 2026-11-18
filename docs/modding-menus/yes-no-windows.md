# Asking the player a question: YES/NO windows (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.
> Test ids such as B20d point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags are explained in the Evidence section.

## Summary

- **A YES/NO question works when an NPC's Talk script asks it.** Call the talk script's own YES/NO library (Majula: state
  machine group 2147483635, a wrapper round `1:130441`), or call `1:130441(text, 0, -1, 0, 0, 0)` yourself. Wait for
  `f130422() != 0` (window open), then read `f130421()`: 1 YES, 2 NO, 3 cancelled. [live B16, B20d, B23, B41 run 3, B43; data]
- **The same question from an event script, asked on its own, does not work.** The window opens, then closes by itself
  with no input. Three builds failed (B20, B20b, B20c) and the Talk route passed at the first try (B20d). All 32 YES/NO
  windows in the game's own event scripts sit in a state machine that first makes a key-guide area or checks an object's
  "examine" [data]; we never tried that route [untested].
- **The failure looks like a timing bug** (the window flashes and is gone), so the first two fixes changed waits, not the
  cause. If your event-script window flashes, the wait is probably not the problem.
- **Texts** go in `mapevent.fmg`. A text of **more than 127 characters** (line breaks included) crashes the game as its
  window opens; a 131-character question did (B41) [live B41; code]. Keep a margin: the mod uses at most 124 characters,
  3 lines and 759 px a line in the language's own font (the width is a display precaution, not the crash limit).
  Details: [Text length limits](text-length-limits.md).
- **After YES** the same Talk can give items (`1:131402`, one lot per state, with your own 3 s fallback because the game's
  item-dialog library has no timeout), set flags, save (`1:132241`) and confirm with a YES-only window. [live B20d]
- **Changes load without a game restart.** A talk script is re-read at the next load of its map [live B20d, same game
  process]; the project's notes of 6 Nov say the same for event scripts [live, from a note]. New texts are read at game
  start, so restart after changing `mapevent.fmg` [live B12 for another text file; inference for this one].

### The two windows, their commands and their functions

Commands (`1:` bank) do things; functions (`f`) read state. The two are separate namespaces: the same number can be a
command and an unrelated function (see Pitfalls). Names are the developer labels in the scripts' `.edd` files.

| Id | Developer name | Gloss | Use |
|---|---|---|---|
| `1:130441(text, a1, a2, kind, value, 0)` | 【FE】YES／NO選択ウィンドウの表示 | show YES/NO choice window | Opens the question. `text` = a `mapevent.fmg` id |
| `1:130442()` | 【FE】YES／NO選択ウィンドウを閉じる | close YES/NO window | Closes it from script. No vanilla script uses it (0 of 53) [data]; we use it before opening an N-choice list (B36c) |
| `1:130443(text, 0, 0, a3, 0, 0, 0)` | 【FE】YESのみウィンドウの表示 | show YES-only window | A message with one button: "OK" windows |
| `1:130444()` | 【FE】YESのみウィンドウを閉じる | close YES-only window | Not used by vanilla or by us |
| `1:130445`, `1:130446` | …（オブジェクトインスタンスID指定） | the same two, with an object instance id | 30 and 17 uses in event scripts. Untested here |
| `f130421()` | 【FE】YES／NO選択ウィンドウの結果取得 | get YES/NO result | 1 YES, 2 NO, 3 cancelled (B button or closed) |
| `f130422()` | 【FE】YES／NO選択ウィンドウ表示判定 | is the YES/NO window shown | 1 while open |
| `f130431()` | 【FE】YESのみウィンドウ表示判定 | is the YES-only window shown | 1 while open |
| `f130432()` | 【FE】イベントメッセージウィンドウ表示判定 | is an event message window shown | Probed in B20c: 0 at the time (no area-name banner) |
| `f130412()` | 【FE】アイテム取得ウィンドウ表示判定 | is the item-get pop-up shown | Used to wait for item pop-ups |
| `1:130408(a0, a1)` / `1:130407()` | 【FE】キーガイド領域の作成 / を削除 | create / delete key-guide area | The "press the action button here" prompt. `a0` is 34 in all 38 event-script uses and 9 in all 84 talk-script uses; `a1` is mostly 600 or 610 [data]. Meaning of the arguments unknown |
| `f6(flag)`, `f3()` | フラグの取得 / ステート経過時間の取得 | get flag / get time in this state | Ordinary script reads |
| `1:130101(flag, value)` | 【フラグ】フラグを設定 | set flag | Works from Talk scripts for global flags [live B16, B20d] |
| `1:132241()` | 【その他】セーブ実行 | execute save | 22 uses in vanilla event scripts, 46 in talk scripts [data] |

Command and function ids come from `tools/edd.py`; event and talk scripts share one command table. [data]

### The arguments

`1:130441(text, a1, a2, kind, value, 0)`. The talk form and the event form differ in one slot.

| Form | Where | Shape | Uses in vanilla [data] |
|---|---|---|---|
| Talk, plain | talk scripts | `(text, 0, -1, 0, 0, 0)` | 13 (the libraries are among them); 2 more with `a1` = 15 and 1 with `a1` = 50 |
| Talk, with a number or item name | talk scripts | `kind` 1 or 2, `value` = the number or the item id | 13 (10 of kind 2, 3 of kind 1) |
| Event, with a number | event scripts | `(text, 0, 190, 1, f132262(..), f132252(..))` | 20 (the grave's soul offer) |
| Event, with an item name | event scripts | `(text, 3 or 4, 220, 2, 60537000, 0)` | 10 |
| Event, with an item name from the caller | event scripts | `(text, 0, 190, 2, arg3, 0)` | 2 |
| Event, plain (`kind` 0) | none | | **0**: the form the mod tried (`(text, 0, 190, 0, 0, 0)`) has no vanilla precedent |

- `kind` 0 shows the text as it is. `kind` 1 fills a `%d` from `value`; `kind` 2 fills a `%s` with the name of item
  `value`. [data: texts with `%d` / `%s` exist; inference: the exact mapping]. We never used kind 1 or 2 ourselves.
- What `a1` (0, 3, 4, 10, 15, 50 occur) and `a2` (-1 in talk scripts, 190 or 220 in event scripts) mean is unknown.
- YES-only: `(text, 0, 0, -1, 0, 0, 0)` in talk scripts (20 uses). In event scripts `(text, 0, 0, 190, 0, 0, 0)` (9 uses),
  `(text, 3, 15, 190 or 220, ...)` (30), `(text, 0, 0, 190, 2, item, 0)` (3) and one with `a1` = 10. [data]
- The event-script form of the YES-only window did open and show its text in the failed builds (B20b screenshot). [live B20b]

### FromSoftware's wait pattern

After `1:130441` the script must not read the result at once. The pattern in the game's own scripts:

1. State A runs `1:130441(...)`.
2. State A moves on only when `f130422() != 0`: the window has opened.
3. The next state branches on `f130421()`: 1 YES, 2 NO, 3 cancelled.

Counted over the 28 original event scripts: 32 uses of `1:130441`; 30 follow this pattern. The two exceptions are the
same prompt in `event_m10_23` and `event_m10_31`, in a machine its developers named for a duel request waiting on the
key guide (決闘リクエスト_開始_キーガイド待機). They test `f130422() != 1` straight after the show and then
`f130421() == 1`; they work in the game, so `!= 1` is not always "closed too early". [data, recount 17 Nov]

The B20 build copied one of those two. The log blamed the pattern ("true before the window is up"); the later builds
used the proper wait and failed the same way, so that diagnosis was not the cause (see Pitfalls). The proper wait is
still the right one: B20c found `f130421()` already non-zero before our window opened (the `stale` flag), so never read
the result before the window has reported open. [live B20c]

### Why an event script cannot ask on its own

What we saw in Majula, in the event script `event_m10_04`, with a new group asking 4 s, then 10 s, after the load:

- The window opens (`f130422() == 1`), then reports closed at once, with a result that is neither YES nor NO. The
  player sees a flash, or the text for a moment, and no input has been given. The debug flags set `opened` and
  `cancel`, never `yes` or `no`. [live B20c]
- Not the bonfire's "Rest" prompt: the same away from the bonfire (B20b). Not the "Majula" area-name banner: a 10 s
  delay put it past the banner and `f130432()` read 0 (B20c). The flag for "result left from an earlier window" was
  set before the window opened, but the window was cancelled anyway. [live B20b, B20c]
- Nothing in the vanilla scripts closes a YES/NO window: `1:130442` has no use in any of the 53 event and talk scripts.
  [data]

What the vanilla scripts do differently: every one of the 32 event-script YES/NO uses is in a state machine that also
creates a key-guide area (`1:130408`) or checks an object's "examine" (command 140205, 【グループ条件：OBJ】OBJ調べる判定,
"group condition: object examine check").
Counted: 20 have both, 10 only the key-guide area, 2 only the examine check. [data, recount 17 Nov] So the working
rule we use is: **the game cancels a script-opened YES/NO window unless the player's action-button press leads to it.**
This is [inference] from the correlation. Three limits on it:

- We only saw it in Majula.
- Our call used a form no vanilla script uses (`kind` 0 with 190 in the event slot), so the argument form may matter too.
- We did not find the code in the exe that cancels the window.

### The route that works: the NPC's Talk

A talk script runs inside a conversation the player started with the action button, and the game's own talk scripts call
`1:130441` from there with `a2 = -1`. Talk scripts also create a key-guide area themselves (`1:130408(9, ...)`, 84 uses),
so a Talk-route question follows an action press as well [inference, fits the event-script rule above]. Majula's talk
script (`talk_m10_04`) has these shared libraries (state machines that other states call with command bank 6):

| Group | Developer name | Gloss | What it does [data] |
|---|---|---|---|
| 2147483635 | 【Lib】選択ダイアログ | choice dialog | Shows `1:130441(arg0, 0, -1, 0, 0, 0)`, waits `f130422() != 0`, returns **0 YES, 1 NO, 2 cancelled** |
| 2147483628 | 【Lib】OKダイアログ | OK dialog | Shows `1:130443(arg0, 0, 0, -1, 0, 0, 0)`, waits for it to open, then to close |
| 2147483623 | 【Lib】アイテム取得ダイアログ：会話中 | item-get dialog, in conversation | Sets five flags, `1:131402(arg0, 1)`, waits for the pop-up to open, then close. **No timeout** |
| 2147483634 | 【Lib】メニュー開始：汎用 | menu start, general | Opens the NPC window (`1:130461`) and maps its result to a return value |

Other talk scripts carry their own copy with a different group id (2147483636, -37, -39, -40 in some). Find the group by
its name or by `1:130441` in its first state. The library returns 2 for both a real cancel (`f130421() == 3`) and a window
that closes with no result (`f130422() != 1`). Its conditions run in the order NO, cancel, closed, YES. [data]

The caller reads the result as `B9` (esd.py's name for a called machine's return value) and knows the call has
returned when `B9 != BA`. [data, tools/esd.py]

**Where the Herald's question hangs.** Her NPC menu is the machine `龍の巫女：NPCメニュー` (group 2147483574). The menu's
entries are fixed game functions; "Talk" returns 2 (library value `B9 == 2`) and is handled by script. The mod points
that branch at its own chain of YES/NO states; "NO to every question" falls through to her normal Talk line, so
nothing of hers is lost. [data; live B16] Why her menu cannot take new entries: [Adding dialogue options](adding-dialogue-options.md).

The arena setup, as built (`choices.menu_talk(setup=...)`):

```
Talk
 gate     setup flag 104828 on?  -> yes: the normal Talk (or a top-up, below)
 q1       1:130465();  library YES/NO (5803)       YES -> q2     NO / cancel -> said_no
 q2       library YES/NO (5804)                     YES -> items  NO / cancel -> said_no
 items    for each lot:  1:131402(lot, 1)  ->  wait f130412() != 0, then != 1   (or 3 s: next lot)
 done     flags on (104828 + the rest), 1:132241() save, OK window (5805) -> her menu
 said_no  OK window (5806) -> her menu         (asked again at the next Talk)
```

Every question state also has a fourth condition, `f130452() == 19` (the NPC window was cancelled), which ends the
conversation as her normal Leave/abort state does. [data; the walk-away case itself is untested]

**Items from Talk.** `1:131402(lot, 1)` takes an item lot, not an item (details: [Giving items](giving-items.md)). The
game's item-dialog library has no timeout, so a lot that gives no pop-up would hang the conversation. The mod writes its
own two states per lot: give, then move on when `f130412() != 0`, or after `f3() > 3` (3 s) if no pop-up appears. A key
item already held shows no pop-up, and the 3 s fallback carried the setup on with the item count unchanged. [live B20d]
Items arrive about one per second: Flame Butterfly 8 to 13 to 18, Soul of a Giant 6 to 11 (B20d watcher).

**Save.** The last state sets the flags and runs `1:132241()`, as vanilla talk and event scripts do after changing a
covenant. The setup flag and the items survived a reload (B20d), but nothing isolated the save command, so "it forces a
save" is [inference].

**Gating by flag.** The question is asked only while the setup flag (a free global flag, block 10) is off. NO leaves it
off, so Talk asks again. Later setup "editions" (new items added in an update) use a second flag and give the missing
items on Talk with no question, then show an OK window ("top-up"). [live B47: a save set up before the edition got two
items, the OK window and then her menu; a feather already held showed no pop-up and the 3 s fallback went on]

### Texts

- Question and OK texts live in `mapevent.fmg`. Only an id inside an existing range can be filled, so use empty slots
  that no script names. The setup uses 5803-5806 and 5839; the fight-choice questions use 5811-5828 and 5831-5836, with
  gaps. [data]
- English only unless you fill the other languages: an unfilled language shows an empty window [inference, from how the
  menu behaved; never tested]. The mod fills all 11.
- The text in the mod now: question 1 is three lines (712, 638 and 667 px, 119 characters), question 2 is two lines
  (604 and 737 px, 82 characters). [data, `tools/i18n.py`]

## How to do it

This is a question asked from an NPC's Talk, the route that works.

1. **Pick the host.** A talk script whose NPC menu machine handles Talk. In the Herald's script that is the branch
   `B9 == 2` of her menu state, which normally goes to her Talk line. Add your first question state and point that
   branch at it.
2. **Find the libraries.** Dump the talk script (`tools/esd.py dump`) and note the group ids of the choice dialog
   (the machine that calls `1:130441` with `a2 = -1`) and the OK dialog. In Majula: 2147483635 and 2147483628.
3. **Add the texts.** Pick free ids in `mapevent.fmg` that no script uses, one per question and per confirmation. Keep
   each text under 128 characters in total (the crash limit; the mod keeps to 124), and to 3 lines of 759 px for looks
   (measure in the language's font: [Text length limits](text-length-limits.md)).
4. **Add a question state.** Entry: the library call (bank 6, group id, text id). Four conditions, in this order:
   `B9 == 0` (YES) -> your YES branch, `B9 == 1` (NO) -> your NO branch, `B9 == 2` (cancel) -> back to the menu, and
   `f130452() == 19` -> the menu's abort state. Do not read `f130421()` yourself; the library does.
5. **Add the YES states.** Set flags with `1:130101(flag, 1)`. For items, one state per lot: `1:131402(lot, 1)`, then
   `f130412() != 0` (or `f3() > 3`) to the next state, and a state that waits `f130412() != 1`. End with `1:132241()`
   if the result must be on disk at once, then an OK window (library 2147483628) and its return condition `B9 != BA`.
6. **Return to the menu.** End the chain with a state that runs `1:130464()` (clear "what was chosen") and goes back
   to the menu's first state, so her window reopens. A conversation that should end goes to the menu's abort state.
7. **Install.** Put the talk script (`ezstate/talk_<map>.esd`) and `mapevent.fmg` (`menu/Text/<language>/`) where your
   install reads loose files, and load the NPC's map again. No restart is needed for the script; restart if the texts
   changed.
8. **Test as the player does.** Ask, answer NO, answer YES then NO, answer YES twice, quit and reload, ask again. Check
   that the window stays up and all lines are readable, and that nothing stays on screen after a NO.

If you must ask from an event script, copy the game's own shape and test it first: a key-guide area, the action press, then
`1:130441` in kind 1 or 2 form, with the `f130422() != 0` wait. We have not shown that this works from a script of our
own [untested].

## Pitfalls and what failed

| Test | What was built | Result | Lesson |
|---|---|---|---|
| B16 (5 Nov) | The Herald's Talk leads into six YES/NO questions through library 2147483635 and sets choice flags | **PASS**, all six | The Talk route works from the first build |
| B20 (6 Nov) | Event group in Majula asks 4 s after load; wait `f130422() != 1` (copied from one vanilla use) | **FAIL**: the question flashed and was gone, twice; flag stayed 0 | Hypothesis in the log: `!= 1` is true before the window is up. Not the cause |
| B20b (6 Nov) | As B20 with the proper wait (open, then result), plus 11 item pop-ups | **FAIL**: gone at once, more than one text box seen, the bonfire's "Rest" prompt on screen | Same failure away from the bonfire. A screenshot showed the question (NO highlighted) and the "Nothing changed" window together with the area banner |
| B20c (6 Nov) | Question 10 s after load, wait to open, wait to close, read the result; seven debug flags | **FAIL**: `stale opened cancel` | The game itself closes it with no input. The result flag records "anything else", so we do not know whether it held 3 or 0 |
| B20d (6 Nov) | The setup moved into the Herald's Talk | **PASS**; flag set, items given, kept after reload | Scripts re-read at the next map load, no restart |
| B41 runs 1-2 (8 Nov) | Question 2 reworded, lines of 771 and 791 px, 131 characters | **Crash** on YES to question 1, in both installs tried | See below |
| B41 run 3, B43 | Reworded to the window rule | **PASS** | Measure widths before shipping a text |

Other things that went wrong or nearly did:

- **Other windows on screen at the same time.** In the event-script builds the question appeared together with the
  bonfire's "Rest" prompt (B20b), and in one screenshot with the area-name banner and the "Nothing changed" window,
  none readable. These were symptoms, not the cause: a 10 s delay and standing away from the bonfire changed nothing.
  A Talk question never meets them, because the conversation owns the screen. [live B20b, B20c, B20d]
- **A crash from a long text.** A question of 131 characters (with lines of 771 and 791 px) made the game throw an
  unhandled `std::length_error` ("string too long") as the window was built, then abort (0xc0000409). The cause is the
  character count: the string helper the window uses refuses more than 127 characters, and it never looks at the
  width. The first reading of the test blamed the widths, because the text was over on every measure at once.
  [live B41; code: the CRT abort at RVA +0xc39b1c, the length check in the wstring helper at RVA +0xefa30, reached
  from RVA +0x198326; addresses read from the crash dump and confirmed in the exe]. More: [Text length limits](text-length-limits.md).
- **Namespaces.** Function 130441 is "character-making window shown" and command 130441 is the YES/NO window; function
  130461 is "player inside their key-guide area" and command 130461 opens the NPC window; function 132241 is the
  new-game-plus cycle count and command 132241 saves. Read the `.edd` name for the right table. [data]
- **The game's item dialog can hang.** Library 2147483623 has no timeout. Use your own give states. [data; live B20d]
- **Opening an N-choice list from Talk.** The list is refused while the NPC's menu window is still open behind the
  conversation (busy bit 0x8); the YES/NO ignores those busy bits, which is why every question worked while the list
  did not. Closing the YES/NO as well is a cheap precaution, never shown to matter on its own. Close the windows first
  (`1:130482(); 1:130442(); 1:130462(); 1:130464()`). [live B36c; code, B36b notes: Ghidra `FUN_1401987f0`,
  image base 0x140000000]. See [Adding dialogue options](adding-dialogue-options.md).

## Evidence

Tags: `[live Bxx]` seen in the game; `[code]` read in the exe; `[data]` read in the game files; `[inference]` reasoned;
`[untested]` an idea only.

| Claim | How we know | Test or source |
|---|---|---|
| A Talk-route YES/NO chain works (six questions, flags set) | live | B16 |
| The setup question, items, flag, "Arena ready" work from Talk | live | B20d, B23 (fresh character), B41 run 3, B43 |
| An event-script YES/NO from a Majula group opens then closes itself, three times | live | B20, B20b, B20c |
| Result already non-zero before our window opened | live | B20c (`stale`) |
| Not the bonfire prompt, not the area banner | live | B20b (away from the bonfire), B20c (10 s delay, `f130432` 0) |
| 32 `1:130441` uses in the 28 event scripts, 30 with the `!= 0` wait | data (recount) | scripts, 17 Nov |
| Every one of those 32 sits with a key-guide area or an examine check | data (recount) | scripts, 17 Nov |
| No vanilla script closes a YES/NO window | data | 53 event and talk scripts |
| The choice library returns 0 / 1 / 2 and waits for open first | data | `talk_m10_04` group 2147483635 |
| The item library has no timeout | data | group 2147483623 |
| A held key item gives no pop-up; the 3 s fallback carries on | live | B20d |
| Scripts reload at the next map load, no restart | live | B20d |
| Texts are read at game start | live (another file) | B12 |
| A 131-character text crashed the window build; the limit is 127 characters, not a width | live, code | B41; exe check 17 Nov |
| The YES/NO window ignores the front-end busy bits | code, live | B36b notes, B36c |
| Save command `1:132241` forces a save | inference | vanilla uses; B20d is consistent |
| Cancel cause = no action-button press | inference | correlation only |
| Walking away mid-question ends the conversation cleanly | untested | by design, no live test |

## Tools

All in the project's `tools/` folder.

| Tool | Use for this topic |
|---|---|
| `tools/choices.py` | `menu_talk()` builds the Herald's chain: the `yesno()` helper (library call plus four conditions), `give_chain()` (give, wait, fallback), the gate, top-ups |
| `tools/arenasetup.py` | The setup's texts, lots, `ITEM_WAIT` (3 s), edition flags; `texts()` refuses a text that does not fit |
| `tools/i18n.py` | `px()` and `fits()`: measure a text in the language's font against the window rule |
| `tools/esd.py`, `tools/edd.py` | Read scripts and their developer names |
| `tools/esdwrite.py` | Write the new states (the writer re-writes a script byte for byte before it is trusted) |
| `tools/resetsetup.py` | Clear the setup flag once so the question is asked again on the same save |
| `tools/testwatch.py` | Logs flags and item counts during a test; B20c's debug flags used it |
| `tools/offline_tests.py` | Checks the built menu, including that the menu without the setup is byte-identical to the tested one |

## Open questions

- Does a vanilla-shaped event-script question (kind 1 or 2, with a key-guide area and a real action press) work from a
  script we add? [untested]
- Is the cancel tied to the argument form (`kind` 0 with 190 in the event slot) rather than to the missing press?
  We only tried that one form. [untested]
- Does the cancel happen outside Majula? We only tested `event_m10_04`. [untested]
- What do `a1` and `a2` do (0, 3, 4, 10, 15, 50; 190, 220)? [untested]
- Is a Talk-route question ended cleanly by walking away? The chain has the abort condition; no live test. [untested]
- Is `1:132241` needed for the setup to persist, or would the game's own autosave do? [untested]
- What do `1:130445` and `1:130446` (the object-instance forms) do outside the game's own scripts? [untested]
- A live check of the crash limit: 127 characters should open, 128 should crash [code says so; untested live]. See
  [Text length limits](text-length-limits.md).

## Related pages

- [Adding dialogue options: the N-choice list window](adding-dialogue-options.md): the list that follows a question, and why the window refused to open
- [Text length limits](text-length-limits.md): the crash from wide lines, widths per language, missing glyphs
- [Giving the player items](giving-items.md): lots, key items, the Torch
- [NPC menus](npc-menus.md): option bits, Talk, why a menu cannot take new entries
- [Event script groups](../modding-scripts/event-script-groups.md): adding a group and its `EventParam` row
- [Script command reference](../modding-scripts/script-command-reference.md): other commands we proved
- [Writing ESD files](../modding-scripts/writing-esd-files.md): building new states
- [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md): choosing the setup flag
- [Adding and translating game text](game-text-and-languages.md): `mapevent.fmg` slots and languages
