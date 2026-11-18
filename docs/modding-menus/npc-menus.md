# NPC menus: option bits, Talk, first meetings, respec (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.
> Test ids such as B31 point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags: `[live B31]` seen in the game,
> `[code]` read in the exe, `[data]` read in the game files, `[inference]` reasoned, not proven, `[untested]` an idea only.

## Summary

- **An NPC's menu is one `NpcEventParam` row, and its entries are 23 fixed game functions, 22 of them switched by
  bits.** Three byte fields, `Unk00`, `Unk01` and `Unk03` (offsets +0, +1, +3), carry one bit per entry. Each entry
  has one fixed string in `npcmenu.fmg` (ids 10101-10123), shared by every NPC. You can switch entries on and off.
  You cannot add one [data; live B31, B32, B42]. Why a new entry needs a different window:
  [Adding dialogue options](adding-dialogue-options.md).
- **Ten entries need no script** (Buy, Repair, Reinforce, Sell, Trade, Covenants, Infuse, Give equipment, Reinforce
  Pyromancy Flame, Incense): set the bit and the window does the work. Infuse on any smith is one bit, `Unk03` 8
  [live B31]. **Twelve entries hand a code back to the talk script**, which needs a branch for it: Talk comes back as
  `B9 == 2`, Move path as `B9 == 9`, Reallocate as `B9 == 11` [live B31, B42].
- **Talk is almost certainly `Unk03` value 1** [inference from data: all 56 script-opened rows that have it handle
  Talk, and the four that lack it never do; never tested by clearing it]. The build did not rely on that. It made
  every vendor's Talk inert in the script instead [live B43].
- **"More items" is the game's "Move path" entry, renamed.** Its text is used by one NPC only (Licia), so the project
  renamed the string in every language, took it off Licia and gave it to the six paged vendors. It turns the shop page
  [live B42, B43, B49].
- **First meetings: the trap is the hostile flags.** The flags the first reading took for "met" flags are the
  vendors' *hostile* flags. Setting them turned every vendor hostile at once [live B42 run 1]. What really gates a first
  meeting is a set of dialogue "heard" flags (block 20, 200000-209999). The mod sets all 93 and clears the 17 hostile
  ones at every Majula load.
- **Respec: use the menu's own Reallocate entry.** The two commands failed at the end of a Talk question chain (the
  animation played, no window) and passed when run from the menu's Reallocate result, with a 0.1 s wait added
  [live B30, B31].

### The NPC window and its row

A talk script opens an NPC menu through a shared library, 【Lib】メニュー開始：汎用 ("Lib: start menu, generic";
group 2147483634 in `talk_m10_04`, another id in some other talk scripts):

```
6:2147483634(0, 220, <NpcEventParam row>, x)      ->  1:130461(x, 0, 220, row)
```

`1:130461` is 【FE】NPCウィンドウを開く ("open the NPC window"). The library waits, reads `f130452()`
(【FE】NPCウィンドウの結果取得, "get the NPC window's result") and returns a small number, which the calling state
tests as `B9`. All 72 library calls in the 19 talk scripts that have the library use `0` and `220`, with two
exceptions: `x` is 5 in Shalquoir's call, and the `220` is -1 in the three calls for the Victor's Stone and the Altar
of Sunlight [data]. What `x` and `220` do is not known. The calling state is a menu state: it opens the window and
branches on `B9`.

The row (`NpcEventParam`, 48 bytes, 73 rows). Shop fields are on [Vendor stock](vendor-shops.md).

| Field | Offset | Notes |
|---|---|---|
| `Unk00`, `Unk01`, `Unk03` | +0, +1, +3 (u8) | the entry bit masks (table below) |
| `Unk02` | +2 (u8) | a covenant id on the 11 rows that open a covenant (Saulden 4, Victor's Stone 8, ...) [data] |
| `displayItemTextId` | +16 | an item id, set on 7 rows: the six with an Offer or Give bit and a second row of the Give NPC (Human Effigy there). Probably fills the `%s` [inference] |
| `Unk06` | +24 | the gesture item (53xxxxxx) a "Learn gesture" entry teaches: set on 13 of the 14 rows with that bit [data] |
| `npcMenu_NameID` | +28 (u16) | the NPC; also the window title in `npcmenu.fmg` |
| `Unk08`, `Unk09` | +31, +32 | set only on 14 rows (`Unk08` 1, 8 or 100; `Unk09` 200-227, one per NPC), all of them the game's own shop NPCs. Other buy rows leave them 0 and their shops work [data; live B28]. Meaning unknown |
| `Unk13`, `Unk14` | +36, +40 | first and last `PluralSelectMenuParam` row of the ladder prices; Gilligan's four rows only (10, 12) [data] |

### The entries

One bit per entry. "Code" is what `f130452()` returns; "`B9`" is what library 2147483634 turns it into. Every
code-and-`B9` pair below was read from the library's state table in `talk_m10_04` [data]; the "who" column is from
the 60 menu states with a literal row id [data, checked 17 Nov].

| Entry (npcmenu id) | Field, bit | Code | `B9` | Script branch? |
|---|---|---|---|---|
| Buy item (10102) | `Unk00` 1 | none | none | no, the window does it |
| Reallocate points (10101) | `Unk00` 2 | 1 | 11 | yes: Strowen (Things Betwixt); the Herald since B31 |
| Repair (10103) | `Unk00` 4 | none | none | no |
| Reinforce (10104) | `Unk00` 8 | none | none | no |
| Enter covenant (10105) | `Unk00` 16 | 5 | 6 | yes (Saulden, Grandahl, ...) |
| Offer %s (10106) | `Unk00` 32 | 6 | 4 | yes (Rat King, Victor's Stone, altars) |
| Sell (10107) | `Unk00` 64 | none | none | no |
| Trade (10108) | `Unk00` 128 | none | none | no [live B32] |
| Ask for pardon (10109) | `Unk01` 1 | 9 | 7 | yes: one NPC (see below) |
| Abandon covenant (10110) | `Unk01` 2 | 10 | 3 | yes (Shalquoir) |
| Learn gesture (10111) | `Unk01` 4 | 11 | 10 | yes (14 rows) |
| Set up ladder (10112) | `Unk01` 8 | 12 | 5 | yes (Gilligan; opens the N-choice list) |
| Move path (10113), now "More items" | `Unk01` 16 | 13 | 9 | yes (Licia; the mod's page switch) |
| Give %s (10114) | `Unk01` 32 | 14 | 8 | yes (Grandahl) |
| Covenants (10115) | `Unk01` 64 | none | none | probably no: its one row also has bit 2 [inference] |
| Talk (10116) | `Unk03` 1 | 16 | 2 | yes, every NPC |
| Leave (10117) | no bit | 17 (18 also) | 1 | yes, every NPC |
| Upgrade Estus Flask (10118) | `Unk03` 2 | 20 | 12 | yes (the Herald) |
| Level up (10119) | `Unk03` 4 | 21 | 13 | yes (the Herald) |
| Infuse (10120) | `Unk03` 8 | none | none | no [live B31] |
| Give equipment (10121) | `Unk03` 16 | none | none | no [data] |
| Reinforce Pyromancy Flame (10122) | `Unk03` 32 | none | none | no [data] |
| Incense (10123) | `Unk03` 64 | none | none | no [data] |
| (window cancelled with the B button) | none | 19 | 0 | yes: every menu state ends the conversation |

Counts: 8 + 7 bits in `Unk00` and `Unk01`, 7 in `Unk03`, plus Leave with no bit = 23 strings, the whole run
10101-10123. The ids are not in bit order (Reallocate, 10101, is bit 2 of `Unk00`).

**How the table was checked.** Every talk script was scanned for menu states that open a row by literal id (60 states
in 19 scripts). For each bit, the `B9` codes handled by every state whose row has the bit, and by no state whose row
lacks it, were listed. The pairs `Unk00` 2 / 11, 16 / 6, 32 / 4 and `Unk01` 1 / 7, 2 / 3, 4 / 10, 8 / 5, 16 / 9,
32 / 8 all came out one to one. `Unk03` 2 and 4 sit on one row (the Herald), so the scan cannot tell them apart; the
library's code table does (20 and 21). "No branch" means no script in the game handles a code for that bit. Live: Buy,
Trade, Infuse and Reallocate each did what the table says (B28, B31, B32), and the Move path and Talk branches work as
tabulated (B42, B43).

**Earlier notes disagree on one bit.** A session write-up listed `Unk03` value 1 as "Leave" and said Talk is not a
bit. The data above disagree: every menu state handles Leave (`B9 == 1`), including the four whose row has `Unk03` 0,
so Leave cannot be that bit. See [Talk](#talk).

### Talk

- **Where it comes from.** `Unk03` value 1 is set on 60 of the 72 real rows (the 73rd is a debug row). The 12 rows
  with `Unk03` 0 are the Victor's Stone and the Altar of Sunlight (3 rows: objects, nobody to talk to), a second
  variant row of one NPC (77101000 against 77100000), and 8 rows of four gesture-teaching NPCs that no script opens by
  a literal row id [data]. Of the 60 menu states that open a row by literal id, the 56 whose row has `Unk03` value 1
  all handle `B9 == 2`. The four whose row has `Unk03` 0 (77101000, 78601000, 78602000, 78602001) never do [data].
  That makes value 1 of `Unk03` the Talk bit [inference; never tested by clearing it]. If it holds, **clearing
  `Unk03` value 1 removes Talk from a menu with no script edit** [untested].
- **What the project did instead (B43).** Its first reading was that Talk is not a menu bit: Talk's text (10116)
  follows Covenants (10115), which would make it `Unk01` 128, and no vendor row has that bit, yet every vendor showed
  Talk. So the build patched the script: in every vendor menu state the Talk branch (`B9 == 2`) goes back into the same
  state, which reopens the menu with no dialogue and no quest step (`shops.talk_quiet_patch`). Talk is still shown
  and does nothing [live B43: "Talk does nothing (the menu stays)"]. The Herald's Talk is her arena menu, untouched.
- **Why Talk was made inert.** A design choice: the arena has no use for the vendors' chat, quest steps or gifts.
  Talk had also been the shop page switch's first trigger (B30 to B32) until B42 moved that to "More items".
- **One string for every NPC.** npcmenu 10116 is "Talk" for every NPC, the Herald included. Renaming it renames hers
  too [data]. This is why "More items" is built from another entry (below).

### "More items": renaming Move path

"Move path" (10113, `Unk01` 16, code 13, `B9` 9) is on exactly one vanilla row, Licia's 76900000 [data]. The build:

1. renames npcmenu 10113 to "More items" in every language folder of the installed copy [data];
2. clears the bit from Licia (`Unk01` 16 to 0), who is not paged;
3. sets `Unk01` 16 on the menu rows of the six paged vendors (Lenigrast, Gilligan, Maughlin, Saulden, Melentia,
   Carhillion);
4. adds `B9 == 9` to each of their menu states, going to the page switch (`shops.talk_page_patch`, see
   [Vendor stock](vendor-shops.md)); Talk stays as it was.

Passed: the entry shows, turns the page, Talk is inert [live B43, B49].

### The rows the mod changes

Original row values are from the game's own params; "now" is the installed build [data, read 17 Nov]. Shop fields
(`Unk00` bit 1, the ranges, bit 128) are in [Vendor stock](vendor-shops.md).

| NPC (row) | Field | Original | Now | Why |
|---|---|---|---|---|
| Lenigrast (76400000) | `Unk03` | 1 | 9 | + Infuse [live B31] |
| Lenigrast | `Unk01` | 0 | 16 | + More items |
| Herald (70000000) | `Unk00` | 0 | 2 | + Reallocate [live B31]; her `Unk03` stays 7 (Talk, Estus, Level up) |
| Saulden (74100000 / 1) | `Unk00` | 16 | 1 | covenant off, buy on |
| Saulden | `Unk01` | 4 / 0 | 16 | gesture off, More items on |
| Gilligan (70401000-3) | `Unk01` | 12 / 8 / 4 / 0 | 16 | ladder and gesture off, More items on |
| Shalquoir (77700000) | `Unk01` | 66 | 0 | Abandon covenant and Covenants off |
| Licia (76900000) | `Unk01` | 16 | 0 | Move path off |
| Rosabeth (76300000) | `Unk03` | 49 | 33 | Give equipment off; Reinforce Pyromancy Flame stays |
| Maughlin, Melentia, Carhillion | `Unk01` | 0 | 16 | + More items |

Left alone: Chloanne, Licia's second row, Rosabeth's second row (`Unk03` 33), Carhillion's flame bit (`Unk03` 33).
Saulden's vanilla row has no buy bit (`Unk00` 16); the early write-up's "17" was the value after the first shop build.

### The Herald's menu

Her row 70000000 gives Talk, Reallocate, Upgrade Estus Flask, Level up and Leave. The menu state is group
2147483574 (龍の巫女：NPCメニュー, "Emerald Herald: NPC menu"). After the project's edits, `B9`:

| `B9` | Goes to |
|---|---|
| 2 (Talk) | the arena chain: setup, options, boss rush and fight lists, ending in her usual Talk line |
| 11 (Reallocate) | the respec states, below |
| 12 (Estus), 13 (Level up) | her own groups, as in the game |
| 1 (Leave), 0 (cancelled) | end |

**Level up.** Her Level up group (龍の巫女：メニュー：レベルアップ, group 2147483533) runs `1:132301(1)`
(【キャラアクション】プレイヤーにアクションリクエスト, "request an action for the player") and then
`1:130455(0, 220, 1)` (【FE】ステータス再設定ウィンドウを開く, "open the status re-set window"). The soul cost comes from
`PlayerLevelUpSoulsParam`, which the mod zeroes: free levels worked at the Herald [live B28]. Details of the param
edit: [Vendor stock](vendor-shops.md).

**Respec (Reallocate).** Only one NPC has it in the game: Strowen in Things Betwixt (menu row 70500000, the one row
with `Unk00` 2). Her talk script (`talk_m10_02`, group 退役の火防女１：ステータスリセット, "retired fire keeper 1:
status reset") does this when the menu returns `B9 == 11`:

| Step | Does |
|---|---|
| wait 0.1 s | `f3() > 0.1` (time in state) |
| a Soul Vessel check | asks a YES/NO with the item, tests the count with `f131401` (【アイテム】アイテム所持数取得, "get number of items held") for item 50960000, and plays a "no Vessel" line if it fails |
| animation | `1:132301(9)` |
| screen | next state: `1:130455(0, 220, 0)` |
| wait | `f130434() != 0` (the window is up), then `f130434() != 1` (it has closed); `f130453() != 0` means the player did something |
| end | `1:132303()` (【キャラアクション】プレイヤーにアクション終了リクエスト, "end the player's action request"), `1:130464()`, a flag of her own, back to the menu |

The Herald's copy keeps every step except the Vessel question, which is left out because the screen itself uses the
Vessel (npcmenu 119303, the screen's confirm question, says the reallocation consumes an item, `%s`) [data]. Whether the screen opens
with no Vessel in the bag was not tried [untested]. The two screens differ in their arguments: respec is
`132301(9)` and `130455(0, 220, 0)`, level up is `132301(1)` and `130455(0, 220, 1)` [data]. Strowen's own group also
sets a flag of hers at the end (102020128); the Herald's copy does not.

### First meetings

**What a first meeting is in the script.** Each vendor has a first-conversation group (names end in 初回会話, "first
conversation": 防具屋：強気：初回会話 for Maughlin, 奇跡の人：初回会話 for Licia, and so on). The vendor's main group
chooses between that group and the usual greeting. Both play spoken lines through 【Lib】会話：汎用 ("talk, generic",
group 2147483647), which takes `(line, flag, ...)`: it plays the line and then sets the given "heard" flag
[data]. So a first meeting is gated by a chain of **heard flags**, all in block 20 (200000-209999):

- Saulden's intro is five lines, each skipped once its flag is on: 203100 (line 74100000), then 203112 (74100050),
  203101 (74100100), 203102 (74100200), 203103 (74100300, the Blue Sentinels offer last). His main group
  (青教戦士：会話, "Blue Sentinel: talk", 2147483593) tests 203103: on = the plain greeting, off = the intro [data].
- Counting every line the ten vendors' Majula dialogue plays gives 93 heard flags (Gilligan 6, Saulden 16, Melentia 3,
  Chloanne 6, Carhillion 12, Licia 4, Lenigrast 10, Maughlin 8, Shalquoir 23, Rosabeth 5). In the original scripts they
  appear only in `talk_m10_04`; no other event or talk script and no GeneratorParam file holds one [data, re-checked
  17 Nov; enemy AI scripts not scanned].

**What the mod does.** Majula's load group 990012 (`event_m10_04`, runs only once the setup flag 104828 is on) sets all
93 on every load, plus the vendors' move flags and Melentia's flag (next sections). The aim is the state of a player
who has heard everything: no intros, no story chat, only the short lines. A side effect: a dialogue gift (Saulden's, after 203117) no longer comes;
every item is in the shops anyway (`tools/arenasetup.py`, `TALK_FLAGS`). The first meetings being skipped was **not
logged line by line**: B42's retest steps ("Saulden: no Blue Sentinels intro") have no recorded result, but B43 and the
clean-install test B49 passed with the flags in place [live B43, B49; the intro skip itself untested].

### Hostile flags: the mistake that looked like a first meeting

The first try set the flags in the table below as "met" flags. The reasoning was a table of `2147483643(flag a, line,
0, flag b)` calls, read as "first speech". It was wrong. After the setup, **every vendor turned hostile** [live B42 run
1: "after talking to the herald, every NPC vendor became hostile"].

| Library (group) | Developer name | What it does [data] |
|---|---|---|
| 2147483643 | 【Lib】初回敵対化 | "first-time hostility": sets flag `a`, flag `b` and 103999, plays a hostile line, saves |
| 2147483631 | 【Lib】初回敵対化_（誓約解除） | the same, and drops you from the covenant (`6:2147483618`, 【Lib】誓約解除：上書き). Saulden's |
| 2147483638 | 【Lib】イベント：分岐 | "event: branch": returns 1 (the hostile path) when its first argument, flag `b`, is on and its second, a death flag, is off; else 0 |

Every setter of the 17 flags below, in every non-AI script of the game, is a call of one of the first two libraries;
no other script writes them. The readers are `2147483638` calls, a Majula event library that counts migrant
survivors, and one NPC-phantom library in a DLC map (it reads Carhillion's `a`) [data, 17 Nov]. The developer name
was in the script's own debug file all along: reading the library names first would have avoided the mistake.

| Vendor | `a` (every place) | `b` in Majula | `b` at home | Note |
|---|---|---|---|---|
| Gilligan | 103550 | 103552 | 103551 | `b` Majula = the Majula generator's `HostileEventID1` |
| Melentia | 103710 | 103712 | 103711 | |
| Chloanne | 103760 | 103762 | 103761 | |
| Carhillion | 103800 | 103802 | 103801 | |
| Licia | 103820 | 103822 | 103821 | |
| Rosabeth | 103770 | 103772 | 103771 | |
| Saulden | 103640 | | | one flag |
| Lenigrast, Maughlin, Shalquoir | 103780, 103750, 103870 | | | one flag each |

`b` for the Majula-born vendors is their generator's `HostileEventID1` too. Home generators (Forest, and so on) hold
`b` at home as their own `HostileEventID1`; the Majula generator carries that same flag in one of its two
`PreviousHostileEventID` slots (the first for five vendors, the second for Gilligan) [data].
The mod's fix: group 990012 sets 102 flags and **clears the 17 `a` and `b` flags** (Melentia's `b` at home, 103711,
is among them) on every load, so a vendor cannot stay hostile in an arena save. It should also mend a save the mistake
reached [built and installed 8 Nov; no logged retest]. The home `b` flags other than 103711 are not touched [data].
The list is `arenasetup.CLEAR_FLAGS`; an offline test checks that no hostile flag is in the set list.

**A pardon resets them.** Every one of the 17 flags, and 103999, is a row in `EventFlagListParam` rows 3500-3999 (flags
103500-103999). The one NPC with "Ask for pardon" (the pardoner, row 78400000, group 免罪の教戒師：NPCメニュー,
"absolving chaplain: NPC menu") reaches `1:130131(3500, 3999, 0)` (【フラグ】パラメータに設定されている指定範囲のフラグを
まとめて設定, "set a flag range from the param, together") in his pardon groups. That is the game's own "make everyone
friendly again" [data; not run by the mod; untested].

### Vendors standing in Majula

Six vendors live elsewhere in the game and move to Majula when a flag is on. The flag is the **`AppearanceEventID` of
their Majula generator** in `GeneratorParam_m10_04`. The setup sets them, group 990012 sets them again at every load
(`arenasetup.MOVE_FLAGS`). The generators are read when the map loads, so **reload Majula after the setup**: the flags
are set on the first load and the vendors stand there on the next [inference; every test reloaded: B28, B30, B42, and
the vendors appeared].

| Vendor | Majula generator row | `AppearanceEventID` (the move flag) | `PreviousHostileEventID1 / 2` | `HostileEventID1` |
|---|---|---|---|---|
| Gilligan | 45 | 102165 | 100956 / 103551 | 103552 |
| Chloanne | 110 | 102625 | 103761 / 0 | 103762 |
| Rosabeth | 115 | 102650 (+ stage flags 102652, 102653) | 103771 / 111355 | 103772 |
| Carhillion | 135 | 102722 | 103801 / 111395 | 103802 |
| Licia | 150 | 102765 | 103821 / 104320 | 103822 |
| Melentia | 260 | 102521 | 103711 / 100971 | 103712 |

(Values 1 to 8 in `AppearanceEventID` are not flags: they read as the minimum NG+ intensity of the linked bonfire
[data; the rule is an inference]. See [NG+ near boss arenas](../modding-world/ng-plus-differences.md) and
[GeneratorParam fields](../modding-world/generatorparam-fields.md).)

All six move flags are in the block of global flags (block 10) [data]. The home generator of each vendor uses the same
flag (Melentia's Forest row has 102521 too), so one flag stands for "this vendor has moved". How the game keeps the
vendor out of the home map after that was not looked at, but the data gives a candidate: the home row carries the same
flag with condition byte 8, and the Majula row carries it with 7. [data] If 7 means "present while the flag is on" and
8 "present while it is off", the one flag moves the vendor out of the home map and into Majula. [inference] See
[GeneratorParam fields](../modding-world/generatorparam-fields.md) for the bytes.

**Melentia needed one more flag.** On a fresh character after the setup, every vendor but Melentia stood in Majula
[live B41 run 3, 8 Nov]. Her row names two more flags, 103711 and 100971. 103711 is her hostile flag at home (it is a
hostile flag, see above, not a "met in the Forest" flag as the first reading had it). **100971 is the Last Giant's
beaten flag**, also read by Majula's bonfire routine for that boss and by one other NPC's move group. The build sets
only 100971 (`arenasetup.MELENTIA_FLAGS`). With it she was in Majula on a new character [live B43 step 3, B49]. This is
a before / after comparison, not a test that isolated the flag [inference]. The other five vendors' rows also carry
flags in the `PreviousHostile` slots (Gilligan's 100956 looks like another boss flag [inference]) and they appeared
with both off, so the fields are not a simple gate. The field names are no guide: Maughlin's three rows use that
slot for three place flags (below).

**The three who need nothing.**

- **Maughlin**: all four of his generators (rows 105-108) are Majula's; there is no home map for him. Majula's group
  111334 (OBJ：防具屋：状態遷移, "armourer: state change") puts him at one of three spots by an NPC-info value
  (`f132251(7610, 5)` and `(7610, 4)`, 【その他】NPC情報汎用値取得, "get general value from NPC info"), setting one
  of 102610 / 102611 / 102612, which his rows 105 / 107 / 108 name in `PreviousHostileEventID1` [data; live B49: he
  stood in Majula and sold].
- **Lenigrast** is behind a locked door in Majula; the setup gives his key, so nothing else is needed [live B49].
- **Saulden and Shalquoir** have Majula rows with no move flag: Saulden's `AppearanceEventID` is his own death flag
  (row 100: 104140), Shalquoir's rows use flags 104010186 and up, one per row. Both were in Majula on a fresh
  character [live B41 run 3, B49; data].

### A. Switch on a fixed entry (no script)

1. Find the NPC's row (`npcMenu_NameID` = the NPC; an NPC often has several rows, one per script state). Read the
   talk script to see which row each menu state opens (`6:<menu library>(0, 220, <row>, x)`).
2. Set the bit: `Unk03` |= 8 for Infuse, `Unk00` |= 128 plus a range for Trade, `Unk00` |= 1 plus a range for Buy
   (ranges: [Vendor stock](vendor-shops.md)). Edit every row that state can open.
3. Install the changed `NpcEventParam` as a loose param ([Editing params](../modding-scripts/editing-params.md)), with the game closed (the
   project always did; a restart is assumed to be needed [inference]).
4. Check: open the NPC; the entry shows; use it. Infuse worked on a smith with no script change [live B31].

### B. Switch on an entry the script must handle (Reallocate as the example)

1. Set the bit on the row (`Unk00` |= 2).
2. In the menu state of the talk script, add a branch for the new code: `B9 == 11` goes to a new state. For the
   code and `B9` of any entry, use the table above.
3. Build the states as the game does (its own copy is `talk_m10_02` group 2147483619): wait 0.1 s; `1:132301(9)`; in
   the next state `1:130455(0, 220, 0)`; wait for `f130434() != 0`, then `f130434() != 1`; then `1:132303()` and
   `1:130464()` and go back to the menu state. The script writer: [Writing ESD files](../modding-scripts/writing-esd-files.md).
4. Enter it from the **menu's own result**, not from a question you ask after Talk (see B30 below).
5. Check with a Soul Vessel in the bag: the screen opens, you can reallocate, the menu comes back [live B31].

### C. Take an entry off

Clear its bit. If the entry has a script branch, leave the branch: nothing reaches it. Done for covenants, gestures,
ladder, Move path and Give equipment [live B49: vendors work, installed menus read back; the missing entries were not
checked one by one].

### D. Reuse a spare entry for your own purpose

Pick an entry no NPC you keep still uses, rename its string in `npcmenu.fmg` for every language (the string is global),
clear the bit from the NPC that had it, set it on the NPCs that should show it, and add the `B9` branch for its code.
The project's "More items" is this recipe on Move path.

### E. Silence Talk

Either point the Talk branch (`B9 == 2`) of each menu state back at the state itself (works [live B43]), or clear
`Unk03` bit 1 on the row (should work [inference]; untested; try it on one row first).

### F. Skip a vendor's first meeting

1. Find the vendor's first-conversation group and the main group's gate (a `f6(<flag>)` test).
2. List every `(line, flag)` the vendor's dialogue plays through 2147483647 / 2147483646 / 2147483645; those flags are
   the heard flags.
3. Set them all with `1:130101(flag, 1)` in a load group.
4. **Do not set anything that a 初回敵対化 call sets.** If a vendor is already hostile, clear the `a` and `b` flags.
5. Test on a character that has not met the vendor, and watch for hostility first.

### G. Stand a vendor in a map

Find the vendor's generator in that map's `GeneratorParam`; its `AppearanceEventID` is the flag; set it (global flags:
[Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md)) and reload the map. Check the row's other flags and clear any
hostile flag.

## Pitfalls and what failed

| Test | What happened | Lesson |
|---|---|---|
| B30 | The Herald's respec as the last question of her Talk chain: the character played the animation, the reallocation window never opened | The commands were right; the entry route was not (next row) [live B30] |
| B31 | Reallocate as a menu entry (`Unk00` bit 2), reached through `B9 == 11`: the screen opened | The retired B30 build ran `132301(9)` and `130455(0, 220, 0)` in the same two states as the B31 build, entered straight from a YES [data]. B31 added a 0.1 s wait and entered from the menu result [live B31]. Which of the two mattered was not isolated. The Talk route also left her menu open behind the question, the state that silently refuses an N-choice list opened from Talk ([Adding dialogue options](adding-dialogue-options.md)); the same cause is a candidate [inference] |
| B30 | Lenigrast could not infuse | Infuse is a separate menu entry in the game: the infusing smith's row (76430000) has `Unk03` 9 where Lenigrast's had 1. `Unk03` 1 to 9 fixed it [live B31] |
| B42 run 1 | Every vendor hostile after the Herald's setup | The "met" flags were hostile flags (above) [live B42] |
| B42 build | "More items" and "options off" steps undid Saulden's buy bit | `tools/shops.py` decoded each row from the original param (`P`) but wrote a copy (`T`): later steps must decode the copy |
| B41 run 3 | Melentia missing on a fresh character, every other vendor present | Her row also names 100971; setting it fixed it (before / after, not isolated) |
| early write-up | Said `Unk03` value 1 is Leave and that Talk has no bit; listed Saulden's row as 17 | Re-read from the original params and scripts (above); 17 was the value after the first shop build |
| B43 | Passed: Talk inert, More items turns pages | Talk was silenced in the script, not in the data |

More notes:

- **A label is global.** Renaming 10113 changed Licia's vanilla Move path too; harmless once her bit is off. Renaming
  10116 (Talk) would rename the Herald's Talk, which opens the arena menu.
- **Every NPC has several rows.** Gilligan has four, Saulden two. Edit every row a menu state can open.
- **Read the edited copy at every step** when several edits touch one row (the B42 build bug above).

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| The entries are fixed functions chosen by `Unk00`, `Unk01`, `Unk03` bits | data: all 73 rows and every script's branches; live: bits added Reallocate, Infuse, Trade | B31, B32 |
| `Unk00` 2 / 16 / 32 and `Unk01` 1 / 2 / 4 / 8 / 16 / 32 map to codes (and `B9`) as tabulated | data: 60 menu states over 19 talk scripts; the library's code table | read 17 Nov |
| Buy, Repair, Reinforce, Sell, Trade, Covenants, Infuse, Give equipment, Pyromancy Flame, Incense need no script branch | data: no script handles a code for them; live for Infuse, Trade, Buy | B28, B31, B32 |
| `Unk03` 8 is Infuse | live | B31 |
| `Unk00` 2 is Reallocate; `B9 == 11` | live | B31 |
| A Talk-chain call of `132301(9)` + `130455` plays the animation only; the menu route opens the screen | live (the same commands) | B30, B31 |
| `Unk03` 1 is Talk | data: 56 of 56 states with it handle `B9 == 2`; 0 of 4 without; untested | read 17 Nov |
| Talk can be made inert in the script | live | B43 |
| npcmenu 10116 is the Herald's Talk too | data (one string) | `npcmenu.fmg` |
| Move path is used by one vanilla row (Licia) | data | `NpcEventParam` |
| "More items" shows, turns pages, in a clean install | live | B43, B49 |
| 2147483643 / 2147483631 are the first-time-hostility libraries | data: developer names; only setters of the 17 flags | B42, read 17 Nov |
| Setting those flags turned every vendor hostile | live | B42 run 1 |
| The 93 heard flags appear only in `talk_m10_04`; not in generators | data | read 17 Nov |
| Saulden's five-line intro and its gate | data | B42, `talk_m10_04` |
| The 17 flags and 103999 are rows 3500-3999 of `EventFlagListParam`; a pardon resets that range | data | `EventFlagListParam`, `talk_m10_14` |
| Group 990012 sets 102 flags and clears 17 | read back from the installed script | B42 |
| The six move flags are the Majula generators' `AppearanceEventID` | data; live that vendors appear after a reload | B28, B30 |
| Melentia needs 100971 | live before / after; not isolated | B41 run 3, B43, B49 |
| Maughlin needs no flag | data: his generators and group 111334; live: present | B49 |
| First meetings skipped | built and installed; vendors friendly in clean install; the skip itself not logged | B42, B43, B49 |
| A Herald level-up is free; her level-up group uses `130455(0, 220, 1)` | live (free); data (the call) | B28 |

## Tools

| Tool | For this topic |
|---|---|
| `tools/shops.py` | `MENU_BITS` (Infuse, Reallocate), `MENU_CLEAR` (the options taken off), the "More items" constants (`MORE_*`), `talk_page_patch`, `talk_quiet_patch`, `npc_events` (writes the rows; reads the edited copy) |
| `tools/arenasetup.py` | `MOVE_FLAGS`, `HOSTILE_FLAGS` / `CLEAR_FLAGS`, `MELENTIA_FLAGS`, `TALK_FLAGS` (the 93), `NPC_FLAGS`, group 990012's spec (`npc_spec`) |
| `tools/choices.py` | `menu_talk(respec_menu=True)`: the Herald's Reallocate states |
| `tools/paramrows.py` | `set_fields` for the row edits |
| `esd.py dump`, `edd.py` | read a talk script and the developer names of its libraries and commands (the project's readers, not in `tools/`) |
| `tools/offline_tests.py` | checks that no hostile flag is set, the 93 heard flags are all set and the options read back |

## Open questions

- **Does clearing `Unk03` bit 1 remove Talk?** One row, one look. If it does, B43's script patch is not needed.
- **What does a menu do when its row offers a script-handled entry the script has no branch for?** The mod always added
  the bit and the branch together. [untested]
- **What do `x` (5 for Shalquoir) and the `220` / -1 in the `130461` call do?** Unknown.
- **Is it the 0.1 s wait or the menu route that made Reallocate work?** Both changed at once between B30 and B31.
- **Does the respec screen open with no Soul Vessel, and is a Vessel used up?** The text says it consumes one.
  [untested]
- **Does Melentia appear without 100971?** Isolate with a fresh character, 102521 on, 100971 off.
- **Do the first-meeting intros really stay quiet?** The retest steps were not logged. Watch Saulden on a new character.
- **What do `Unk08` and `Unk09` do?** They sit on the 14 rows with their own shop screen; other buy rows work at 0.
- **What are the other readers of the `a` flags?** One DLC NPC-phantom library reads Carhillion's. Does an NPC with a
  hostile `a` flag stop appearing as a summon? [untested]
- **Which entry does `Unk01` 64 (Covenants) open, and does it need a branch?** Only Shalquoir's row has it.

## Related pages

- [Vendor stock: the 64-row cap, flag pages, free items and Trade lists](vendor-shops.md): the shop fields, page switch and costs.
- [Adding new dialogue options: the N-choice list window](adding-dialogue-options.md): why a menu cannot take a new entry, and the route that works.
- [Asking the player a question: YES/NO windows](yes-no-windows.md): the question libraries the Herald's chain uses.
- [Event and talk script commands](../modding-scripts/script-command-reference.md): `130461`-`130465`, `130455`, `132301`, `130131`.
- [Writing ESD files](../modding-scripts/writing-esd-files.md): building the menu-state branches.
- [Editing params](../modding-scripts/editing-params.md): loose params, and why `NpcEventParam` edits apply.
- [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md): global flags, blocks 10 and 20, the pardon range.
- [GeneratorParam fields](../modding-world/generatorparam-fields.md): `AppearanceEventID`, `HostileEventID1`, and the flags above.
- [Giving the player items](giving-items.md): the setup the Herald's Talk runs.
- [Game text and languages](game-text-and-languages.md): the `npcmenu.fmg` strings and the rename in every language.
