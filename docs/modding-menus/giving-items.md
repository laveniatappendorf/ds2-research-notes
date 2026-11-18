# Giving the player items (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.
> Test ids such as B20d point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags are explained in the Evidence section.

## Summary

- **From a script, give an item lot, not an item.** `1:131402(lot, 1)` takes a row id of `ItemLotParam2_Other`. In the
  game's own scripts the second argument is 1 in all 74 uses. The game shows its normal "item received" pop-up, one per
  call. For an item with no suitable lot, re-item a lot that nothing refers to: change `item_lot_0` and `amount_lot_0`
  and leave the rest of the row alone. [live B20d, B20e, B24, B26, B43, B47; data]
- **The lot's amount counts in full, even above the item's `max_held_count`.** A Torch lot of 99 gave "Torch x99" in one
  pop-up; so did 99 Human Effigies. [live B20e, B20d]
- **The Torch is not a bag item.** It is pooled burn time: about 5 minutes per Torch, 99 of them took the item menu's
  remaining time from 10 minutes to 8 h 30 min. The bag never holds a Torch after a pick-up. [live B20d, B20e]
- **The main trap: a give that the game refuses is silent, and the game's item-dialog library never times out.** A key
  item you already hold shows no pop-up, so a script that waits for the pop-up hangs. The mod writes its own two states
  per lot with a 3 s fallback. [live B20d, B47; data]
- **From outside the game (a trainer), call the game's own pick-up code** instead of writing bag bytes. The bag has
  **two** lists: 3,840 slots for ordinary items and a second list of 256 at bag +0xF0F4 for key items and gestures. Our
  first bag reader followed the community cheat table's layout, which documents the first list; so a held Ashen Mist
  Heart looked absent, and the game refused a second one. [live 4 Nov item-giver check]
- Vanilla NPCs accept items and souls given by either route: levelling, upgrading and a boss-soul trade all worked on
  granted goods. [live A9]

### Two routes

| | Script route | Memory route |
|---|---|---|
| Who runs it | A talk or event script, inside the game | A trainer (a separate program) on a remote thread in the game process |
| What it needs | A lot row that holds your item | The item id, quantity, upgrade, infusion |
| What the player sees | The normal item pop-up | The same pop-up (the trainer calls the pop-up code too) |
| Used by the mod for | The once-per-save arena setup, later top-ups | Items a fight needs at fight start (the Ashen Mist Heart for the Giant Lord), building a test kit |
| Needs a param edit | Only if no existing lot fits (a loose `ItemLotParam2_Other`) | No |

#### Commands and functions

Names are the developer labels in the scripts' `.edd` files (read with `tools/edd.py`). Commands (`1:` bank) do things;
functions (`f`) read state. The two are separate namespaces. Counts are over the 28 event and 25 talk scripts of the
unmodified game. [data, recount 17 Nov]

| Id | Developer name | Gloss | Use |
|---|---|---|---|
| `1:131402(lot, 1)` | 【アイテム】アイテム追加 | item: add item | Gives the lot. 74 uses (18 event, 56 talk); second argument 1 in all 74. 9 uses name a lot literally, 65 take the lot from the calling machine |
| `f130412()` | 【FE】アイテム取得ウィンドウ表示判定 | FE: is the item-get window shown | Non-zero while the pop-up is up. The game waits for `!= 0` (it opened), then `!= 1` (it closed) |
| `f131402(item, 1)` | 【アイテム】アイテムを取得できるか | item: can the item be obtained | A pre-check. 7 uses, all in event scripts, always with a literal or passed **item** id and count 1. We never use it |
| `1:130494(lot, 0, 190 or -1)` | 【FE】アイテム取得失敗ウィンドウを表示 | FE: show the item-get-failed window | 30 uses (once with 40 in the middle slot). Shown by the game's own "cannot take it" branch. The two callers we traced pass a lot id. We never use it |
| `f130413()` | 【FE】アイテム取得失敗ウィンドウ表示判定 | FE: is the item-get-failed window shown | Waited on the same way as `f130412` |
| `1:131401(item, count)` | 【アイテム】アイテム消費 | item: consume item | 46 uses. The reverse of giving. We never used it |
| `f131401(item, 1, 1, 0)` | 【アイテム】アイテム所持数取得 | item: get number held | Count in the bag. The Things Betwixt respec checks for a Soul Vessel with it |
| `1:132241()` | 【その他】セーブ実行 | execute save | Saves at once. The setup calls it after the last item |
| `1:132211(n)` | 【その他】ソウル値加算 | add to the soul value | Souls. Vanilla only uses it with a negative number (paying at a grave: 23 uses). Giving souls with a positive value is [untested] |

#### The first argument is a lot

`1:131402`'s first argument is a row id of `ItemLotParam2_Other`. An early plan had passed the item id; the project's
read of the scripts caught it before any test. [data]

- The 9 literal uses name 8 distinct lots (60015000 twice); every one is a row of `ItemLotParam2_Other`, and each gives
  one named item (Forgotten Key, three crowns, Soul of Nadalia, Ring of the Dead, Soul of Alsanna), except 60001000, which
  gives a Forgotten Key and a Soul of a Giant. [data]
- Item ids make poor lot ids: item 50910000 (Ashen Mist Heart) and 50920000 (Soul of a Giant) are not rows of any lot
  param. Item 50800000 (Bastille Key) is a row of `ItemLotParam2_Chr`, an all-empty enemy-drop row, and not of
  `ItemLotParam2_Other`. [data]
- A lot id can also be an item id by chance. Lot 60020000 gives the Crown of the Sunken King; item 60020000 is the
  Radiant Lifegem. Passing an item id would give the wrong thing without any error. [data; not run]
- The game has three lot params: `ItemLotParam2_Other` (1,432 rows), `ItemLotParam2_Chr` (749, enemy drops) and
  `ItemLotParam2_SvrEvent` (32). Every literal use in the scripts hits `_Other`. Whether the command can read the other
  two is [untested].

Lot fields (paramdef names), 124-byte rows [data]:

| Field | What we saw |
|---|---|
| `item_lot_0` .. `item_lot_9` (s32) | the items; 0, -1 and 10 mark an empty slot in the rows we read |
| `amount_lot_0` .. `amount_lot_9` (u8) | the count of each; the amount of lot 60019000 is 6 Smelter Wedges |
| `chance_lot_0` .. `chance_lot_9` (f32) | 100.0 on the Ancient Dragon's and the Aged Feather's lots, 1.0 on the other lots we use. Both gave the item every time. Read as a weight among the slots, not a percentage [inference] |
| `lots_dropped_num` (u8) | 1 on 1,427 of 1,432 rows; 2, 3 and 7 on the other five. The Forgotten Key lot has 2 and holds two items. Probably how many slots are picked [inference] |
| `infinite_lot_n` (u8) | 1 on the used slots of every lot we use, including the unused rows we re-itemed. Meaning untested |
| `reinforcement_lot_n`, `infusion_lot_n` | upgrade and infusion of a weapon in the lot. Not used |

**Use lots with one item.** 1,118 of the 1,432 `_Other` rows have exactly one item. The project's build refuses a lot
that gives anything but the one wanted item, or one whose amount does not divide the count asked for. [code,
`arenasetup.awards`] A lot with several slots and `lots_dropped_num` 1 is probably a weighted random pick [inference];
we only used single-slot lots.

#### Finding or making a lot

1. Look for an existing single-item lot of the item (the Aged Feather has lot 1701000, the Smelter Wedge lot 60019000,
   six at a time). Several lots hold one Human Effigy (10250, 10260, 10270) and one holds six (1705000). [data]
2. If none fits, re-item an **unused** lot. The unmodified game has 18 rows, 99995000-99995008 and 99996000-99996008,
   that nothing refers to: re-scanned on 17 Nov over all 538 scripts, the 205 regulation params, 28 MSB files and the
   exe, with no hit other than the rows' own ids (the first scan was on 6 Nov). [data] The project's row writer refuses
   this param, so adding rows was not tried; a field edit on an existing row works [live B20e]. Whether the refusal was the
   reason for re-iteming is not recorded [inference]. See [Editing params](../modding-scripts/editing-params.md).
3. Change only `item_lot_0` and `amount_lot_0`. The rest of the row (the weights, the `infinite` byte, the dropped
   count) already works. The mod's edits, as loose-param field edits:

| Lot | Was (unmodified game) | Now | Why |
|---|---|---|---|
| 99995008 | Human Effigy x1 | Human Effigy x99 | 99 Effigies in one pop-up [live B20d] |
| 99995007 | Divine Blessing x1 | Torch x99 | one pop-up, about 8 h of burn time [live B20e] |
| 99995006 | Elizabeth Mushroom x1 | Lenigrast's Key x1 | no lot in the game gives the key [live B24] |
| 99995005 | Old Radiant Lifegem x3 | Soul of a Great Hero x1 | 50,000 souls to start buying [live B43] |

Loose params are explained on [Editing params](../modding-scripts/editing-params.md). Lenigrast's Key is given because the shops no longer
sell keys ([Vendor stock](vendor-shops.md)).

#### What the game's own item-dialog library does

Majula's talk script (`talk_m10_04`) has the library `【Lib】アイテム取得ダイアログ：会話中` ("item-get dialog, in
conversation"), group 2147483623. It sets five flags (`1:130101(arg5 .. arg1, 1)`), runs `1:131402(arg0, 1)`, waits
`f130412() != 0` and then `f130412() != 1`, and returns. **It has no timeout.** If the give is refused there is no
pop-up and the library waits for ever. Three other groups of the same script call it. [data]

The game's event scripts guard against the same case. In the memory exit of `event_m20_10`, one machine tests
`f131402(item, 1) != 0` before the give, and a separate machine, `【実行】巨人の死体から帰還_アイテム取得不可` ("execute:
return from the giant's corpse, item cannot be obtained"), shows `1:130494(lot, 0, -1)` and waits on `f130413`. The same
shape (check, then give or show the failed window) is in 5 event scripts. [data]

#### The mod's own give states

The mod does not call the library. For each lot it writes two states (`give_chain` in `tools/choices.py`):

```
give_k   entry: 1:131402(lot, 1)
         -> got_k   if f130412() != 0          (the pop-up opened)
         -> next    if f3() > 3                 (3 s, no pop-up: refused or already held)
got_k    -> next    if f130412() != 1           (the pop-up closed)
```

`f3()` is the time in the current state ("get state elapsed time"). `next` is the next lot's give state, or the final
state. Items arrive about one per second: Flame Butterfly 8 to 13 to 18, Soul of a Giant 6 to 11. [live B20d]

The final state sets the setup flag and the edition flags, then `1:132241()` and an OK window. **The flags are set last**,
so quitting in the middle gives the items again next time. A repeat is harmless: stacks stop at their limit and a held
key item is refused. [code; findings, 5 Nov]

#### What a refused give looks like

- **A key item already held** (Ashen Mist Heart, Bastille Key): no pop-up, the count stays 1, the 3 s fallback carries on.
  [live B20d]
- **An Aged Feather already held** (max 1): the same. [live B47: "Not the feather, I already had it"]
- Whether a refused `1:131402` also shows the failed window (`1:130494`) is not recorded in any test. No test note
  mentions one. [untested]
- Whether the script command refuses by the same rule as the trainer's check (below) is [inference]; both held items
  were refused.

#### The Torch is burn time

- B20d's first setup gave Torch x1 (the lot was a Forest treasure lot). The pop-up showed ("only 1"), but a memory read
  afterwards found no Torch (60420000) anywhere in the bag or in the key list. The Torch the trainer had given on
  5 Nov was gone too. [live B20d]
- The item's description says to find new torches to extend the period. `ItemParam` has `max_held_count` 1, category 8
  (goods), item type id 1300, no weapon link. [data; live B20e]
- B20e set lot 99995007 to Torch x99. The pop-up read "Torch x99" and the remaining torch time went from **10 min to
  8 h 30 min**, about 5 minutes per Torch, with no cap met. So the lot's amount counts in full above `max_held_count`, and
  the Torch is pooled time, not a held item. [live B20e]
- Why it matters: the Lost Sinner's sconces need a lit Torch. A butterfly and a Torch lit them. [live B15]
- Burn time is a number you can see in the item menu. We do not know its cap or the exact seconds per Torch [untested].

#### What the arena setup gives

Once per save, after two YES/NO questions in the Emerald Herald's Talk ([Asking the player a question](yes-no-windows.md)).
The list comes from `tools/arenasetup.py`: a base list plus the `setup_items` of three fights. 15 lot calls, so up to 15
pop-ups. [code, run 17 Nov]

| Item (id) | Count | Lot | Where the lot comes from | Why the setup gives it |
|---|---|---|---|---|
| Ashen Mist Heart (50910000) | 1 | 1787000 | the Ancient Dragon's own lot | the Giant Lord's memory |
| Human Effigy (60151000) | 99 | 99995008 | re-itemed, amount 99 | a design choice: back to human by item, not by script on every load |
| Torch (60420000) | 99 | 99995007 | re-itemed | the Lost Sinner's sconces, and long burn time |
| Lenigrast's Key (50870000) | 1 | 99995006 | re-itemed | the blacksmith from the start |
| Smelter Wedge (53200000) | 6 | 60019000 | the game's own lot | the Fume Knight's four idols, two spare [live B26] |
| Aged Feather (60355000) | 1 | 1701000 | the game's single-item lot | a reusable warp to the last bonfire rested at (edition 2) |
| Soul of a Great Hero (60720000) | 1 | 99995005 | re-itemed | 50,000 souls (edition 2) |
| Bastille Key (50800000) | 1 | 10166180 | a Lost Bastille treasure lot | the Lost Sinner |
| Flame Butterfly (60430000) | 10 | 50367130 twice | a Brume Tower treasure lot, 5 each | the Lost Sinner |
| Soul of a Giant (50920000) | 5 | 60004000 five times | the memories' own lot, 1 each | Vendrick |

A treasure lot given by script does not touch the map pick-up [inference, never checked]. Five Souls of a Giant take five
pop-ups because the lot gives one; an own lot with amount 5 would give one pop-up. We left it.

The setup does not set the Ancient Dragon's "heart given" flag (103140), so he may still offer a second Heart, which the
game refuses. [data; findings, 5 Nov; not seen]

#### Setup editions: topping up an older save

A later release can add items without asking the questions again. Each **edition** is a flag plus the items it added
(`arenasetup.EDITIONS`, `EDITION_ITEMS`).

- Edition 1 is the setup flag 104828. Edition 2 is flag 104831 and holds the Aged Feather and the Soul of a Great Hero.
  [code]
- Talk gate: setup flag **off** -> the two questions and the full setup, which sets every edition flag. Setup flag **on**,
  an edition flag off -> give that edition's lots, set its flag, `1:132241()`, show an OK window ("Boss Arena updated"),
  then the menu. One edition per Talk. [code, `menu_talk(topups=)`]
- A save set up before edition 2 got the two items, the window and her menu. A feather already held showed no pop-up and
  the fallback went on. [live B47, the clean copy, 9 Nov]
- Rules the tools enforce: every edition item must also be a base item with the same count (so the full setup and
  "old setup plus top-up" end the same), and the flags are frozen in the project's id register: add one, never move or
  reuse one. [code, `arenasetup.topups`, `tools/ids.py`] Flags: [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md).
- A tool, `tools/resetsetup.py`, clears the setup flag once so the questions are asked again on the same save. [code]

### The memory route (a trainer)

The route is for a program outside the game that wants an item in the bag, with the normal pop-up. It reuses the game's
own pick-up routine instead of writing bag bytes.

The game's pick-up routine is Ghidra `FUN_140199cc0` (image base 0x140000000, RVA +0x199CC0). The steps, from the
4 Nov Ghidra reading [code, as recorded in the notes; the project's offline test checks the call order in the built
stub]:

1. `bag = [[GameManagerImp + 0xA8] + 0x10]`. Stop if any step is null. `GameManagerImp` is the global in
   [Memory map](../tools/memory-map-pc.md).
2. `can_give(bag, list, n)` at RVA +0x1ABA90. It checks room, stack limit and validity. If false, stop: **refused**.
3. `give(bag, list, n)` at RVA +0x1AC3D0. If false, stop.
4. The pop-up: `popup_prep(stack, list, n, 1)` at RVA +0x5D950, then `popup(display, stack)` at RVA +0x501080, with
   `display = [GameManagerImp + 0x22E0]`. The stack is a scratch area the game sizes at 72 dwords; the stub gives it 0x120
   bytes.

In Ghidra form (image base 0x140000000) the four calls are at 0x1401ABA90, 0x1401AC3D0, 0x14005D950 and 0x140501080.
The community cheat table's item-give script gave us the shape and the entry layout, which saved a lot of work. The game's
own routine added two details: the room check before the give, and the pop-up's first argument (the object at
`GameManagerImp + 0x22E0`). [code]

The trainer runs this as a few bytes of code on a remote thread (`tools/arenalib.py: give_stub`). It writes one result
byte: 0 refused (a null pointer, `can_give` false or `give` false), 1 given with no pop-up, 2 given with the pop-up.
It will not run while the player is dead or loading. It gives 1 to 8 items per call: 8 is the size of the entry area
in the stub's page, not a game limit; the 4 Nov notes say the game takes up to 32 [code; the 32 is from the notes].

#### The give entry

The list is 16-byte entries. **This is not the bag slot's layout** (below): the id is at +4 here and at +0 there.

| Offset | Size | Field |
|---|---|---|
| +0x00 | u32 | 0 |
| +0x04 | u32 | item id |
| +0x08 | f32 | durability (weapons, armour, rings); 0 for goods |
| +0x0C | u16 | quantity |
| +0x0E | u8 | upgrade level |
| +0x0F | u8 | infusion |

`item_entry` fills it from the params (`tools/arenalib.py`). The item's type is `ItemParam` +0x4F (the category: 0
weapon, 1 shield, 2 to 5 armour, 6 ammo, 7 ring, 8 goods, 9 spell, 10 gesture) and its stack limit is the u16 at +0x4A
(`max_held_count`). [code; data: 263 / 74 / 144 / 111 / 103 / 103 / 18 / 122 / 211 / 106 / 20 rows in categories 0 to 10]

| Type | What the entry gets |
|---|---|
| Weapon, shield (0, 1) | quantity 1; full durability from `WeaponParam` +0x28; upgrade clamped to the top of its `WeaponReinforceParam` row (+0x48), so special weapons stop at +5; infusion only if the weapon's `CustomAttrSpecParam` row allows it, else 0 |
| Armour (2 to 5) | quantity 1; durability from `ArmorParam` +0x38; upgrade clamped to the `ArmorReinforceParam` row (+0x60) |
| Ring (7) | quantity 1; durability from `RingParam` +0x4 |
| Ammo, goods (6, 8) | quantity clamped to 1 .. `max_held_count` |
| Spell (9) | quantity 1 |
| Gesture (10), unknown id | refused by `item_entry` before anything is called |

[code, `tools/arenalib.py`; offline tests, e.g. Dagger +15 -> +10, Human Effigy 150 -> 99]

#### The bag

Checked live on 4 Nov: a read of the bag gave 29 items, then 41 once the second list was read, with names right.

| Where | What |
|---|---|
| `[[GameManagerImp + 0xA8] + 0x10]` | the bag object |
| bag +0xF0 | list 1: **3,840 slots x 16 bytes**. Weapons, armour, rings, ammo, goods, spells |
| bag +0xF0F4 | list 2: **256 slots x 16 bytes**. Key items and gestures |

The first list ends at +0xF0F0, so the second begins 4 bytes later. What is in those 4 bytes is not known.

A slot:

| Offset | Size | Field |
|---|---|---|
| +0x00 | u32 | item id (0 = empty) |
| +0x04 | u32 | 0 (as the reader's notes give it) |
| +0x08 | u16 or f32 | **quantity (u16) for ammo, goods and spells; durability (f32) for weapons, armour and rings** |
| +0x0C | | upgrade level (a Large Club +3 read as 3; the field's width was not checked) |

Weapons, armour and rings take one slot each. [live 4 Nov: Estoc 45.0 and Faraam 85.0 read as durability]

- **The first bag reader summed durability as quantity.** At +0x08 a slot holds the f32 durability for equipment, so
  summing it as a u16 gave huge numbers. The reader now counts one per slot for types 0 to 5 and 7. [live 4 Nov]
- **The second list was missing.** The Ashen Mist Heart was refused as "not in the bag" (13:42), but it was there: a
  memory search for 50910000 found it at bag +0xF1A4. That is slot 11 of the second list (0xF1A4 - 0xF0F4 = 0xB0 = 11
  slots), after the 8 starting gestures, the Soldier Key, the Eye of the Priestess and Lenigrast's Key. [live 4 Nov]
- **Which list holds what.** The four keys in the second list all have `ItemParam` item type id 9000 ("key"), and
  the gestures have 3000 and category 10; the 27 rows with type id 9000 include the Heart, the Bastille Key and
  Lenigrast's Key. So list membership probably follows the item type id [inference, from `ItemParam`; the exe was not
  read]. The Torch, Effigy and Aged Feather are type id 1300, 500 and 250: ordinary goods.
- The game's room and stack check refused a second Heart (max 1). Which of `can_give` and `give` said no is not logged;
  the stub's result byte says only "refused". [live 4 Nov; inference for the cause]
- Every named weapon, armour piece and ring fits at once: 333 + 435 + 122 = 890 slots of the 3,840. The Bottomless Box
  was not checked. [data, 5 Nov count; untested]

#### Live check of the trainer's give (4 Nov, 13:40 to 13:54)

| Time | What | Result |
|---|---|---|
| 13:40 | Read the bag, nothing written | 29 items, names right. Equipment quantities wrong: the +0x08 fix above |
| 13:41 | `give 60151000` (a Human Effigy) | "given (pop-up shown)": Effigies 2 to 3, and the pop-up was seen |
| 13:42 | Giant Lord start: give the Ashen Mist Heart | "the game refused it"; no pop-up. The corpse still took the player in, because the Heart was held |
| 13:52 | Same, after the second list was added to the reader | "already in the bag", no give called |
| 13:53 | Arrival at the saved spot | the fight started. The earlier no-aggro run was a skipped start region, not the item |

The trainer's `GIVE_ITEMS` gives the Heart at fight start only if the bag reader does not find it. A bag it cannot read
is given to anyway, and the game's own check refuses a duplicate. [code, `arenactl.give_needed`]

**A weapon given by the trainer still needs its stats.** Items alone do not let a level 1 character use a weapon with
stat requirements; that needs a separate stat write. [4 Nov session note; not a test of its own]

### NPCs accept granted goods and souls

A9 (28 Oct) checked that the vanilla NPC paths take items and souls that were given from outside. [live A9]

| NPC | Given | Result |
|---|---|---|
| Emerald Herald | 20,000,000 souls, given by the community cheat table's souls entry | Vigor 1 to 10 for 421,241 souls |
| Lenigrast | titanite, given by the cheat table's item give | an upgrade paid 550 souls |
| Straid of Olaphis | a boss soul given from outside; Intelligence and Faith raised first at the Herald | the boss-soul trade went through, 1,500 souls |

The NPCs' own stat gates still applied: Straid needed his minimum Intelligence and Faith first. So the arena can hand out
souls and materials freely without breaking a vendor.

### Give an item from a talk script

1. **Find or make a lot.** Look for a single-item lot of your item in `ItemLotParam2_Other`; else re-item an unused one
   (`99995000`-`99995008` or `99996000`-`99996008` in the unmodified game; check again that nothing refers to your pick).
   Edit only `item_lot_0` and `amount_lot_0`.
2. **Read the lot back.** One item, the amount you want, and the amount divides your total. `tools/arenasetup.py:
   awards` does this against the params plus your loose edits and stops the build otherwise.
3. **Install the loose param** ([Editing params](../modding-scripts/editing-params.md)).
4. **Add the give states** to the NPC's talk script, as in the sketch above: per lot, a give state (`1:131402(lot, 1)`,
   leave on `f130412() != 0` or `f3() > 3`; the mod lists the pop-up exit first) and a state that waits for
   `f130412() != 1`. Writing states: [Writing ESD files](../modding-scripts/writing-esd-files.md).
5. **Set your flag last**, then `1:132241()` if the result must be on disk at once, then an OK window.
6. **Gate it.** Ask or give only while your flag is off. For a later release, add an **edition** flag and the lots it adds
   (rules under "Setup editions").
7. **Test the held-item case.** Give an item the character already holds (a key item) and check that the fallback goes
   on and no window stays open.
8. **Test as a fresh character and as an old save**, then quit and reload and talk again. [live B23, B47]

Giving from an event script of your own is [untested]: the game does it 18 times, but our event-script setup never got as
far as the items, because its question was cancelled (B20 to B20c, see [Asking the player a question](yes-no-windows.md)).

### Give an item from a trainer

1. Check the game build and that the player is alive and not loading.
2. Build the 16-byte entries (table above), up to 8 per call. Clamp quantity, upgrade and infusion to what the item allows.
3. Write the stub and run it on a remote thread; read the result byte (0 / 1 / 2).
4. If you must know the bag's contents first, read **both** lists.
5. Treat 0 as "already held, at its stack limit, no room, or invalid", not as an error in your call.

## Pitfalls and what failed

| Test | What | Result | Lesson |
|---|---|---|---|
| 5 Nov plan | `1:131402(item, 1)` as an item id | Caught offline by reading the 74 uses: the first argument is a lot; no `_Other` row has the item ids used | The argument is a lot id. A lot and an item can share an id by chance (60020000) |
| 4 Nov | Trainer give of a Heart that was held | Refused, no pop-up; the reader had missed the key list | Read both lists; the game refuses a second key item |
| 4 Nov | First bag reader | Equipment counted as huge quantities | +0x08 is durability for weapons, armour, rings |
| B20d | Torch x1 by lot 10106080 | Pop-up shown, no Torch in the bag or key list | The Torch is burn time (resolved in B20e) |
| B20d, B47 | Key item or Aged Feather already held | No pop-up; fallback carried on | Always write the fallback; the game's item library never times out |
| B20e | Torch lot x99 | One pop-up, 99 counted, 10 min to 8 h 30 min | Amount above `max_held_count` counts in full |

The passes that followed (B24, B26, B43, B47) are in the Evidence table.

Other things to know:

- **The Torch cannot be counted in the bag.** A test that waits for a Torch to appear will fail. Watch the burn time.
- **A lot that gives one unit costs a pop-up per unit.** Five Souls of a Giant are five pop-ups and about 5 s.
- **Our fallback is a timer, not a check.** `f131402(item, 1)` is the game's own check [data]; we have not tried it
  [untested].
- **Stack limits.** `max_held_count` is 99 for 160 of the 211 goods and 1 for the other 51 (keys, the Torch, the Aged
  Feather). The trainer clamps its quantity to the limit. From a script, a lot above the limit is known only for the
  Torch (limit 1, 99 counted as burn time, B20e). For an ordinary stackable it is [untested]. [data, 17 Nov]
- **Changes to a talk script load at the next load of its map, without a game restart.** [live B20d, same process]

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| `1:131402` takes a lot id; 74 uses, second argument 1 in all; 9 literal uses are 8 `_Other` lots | data recount | 53 event and talk scripts, 17 Nov |
| The setup's lots give their item and amount; one pop-up per lot | live | B20d (11 pop-ups), B20e |
| Amount above `max_held_count` counts in full | live | B20e (Torch x99), B20d (Effigy x99) |
| The Torch is burn time, about 5 min each, no bag slot | live | B20d, B20e |
| A held key item gives no pop-up and the 3 s fallback goes on | live | B20d, B47 |
| The game's item dialog library has no timeout | data | `talk_m10_04`, group 2147483623 |
| The game pre-checks with `f131402(item, 1)` and shows `1:130494` on failure | data | 7 and 30 uses, `event_m20_10` |
| Lots 99995000-99996008 are referenced by nothing | data | 538 scripts, 205 params, 28 MSBs, exe; 17 Nov |
| Smelter Wedges x6 by lot 60019000 | live | B26 |
| Lenigrast's Key by lot 99995006 | live | B24 |
| Soul of a Great Hero by lot 99995005 | live | B43 |
| Setup top-up by edition flag 104831 | live | B47 |
| A fresh character gets the whole setup | live | B23 |
| The pick-up routine, can-give and pop-up calls | code | Ghidra reading, 4 Nov; stub order checked offline |
| The trainer's give works with the pop-up | live | 4 Nov, 13:41 |
| Bag: list 1 at +0xF0 (3,840 x 16), list 2 at +0xF0F4 (256 x 16) | live | 4 Nov |
| A second Ashen Mist Heart is refused | live | 4 Nov, 13:42 |
| Key item list membership follows `item_type_id` 9000 | inference | `ItemParam`, 17 Nov |
| NPCs accept granted souls and goods | live | A9 |
| Giving souls by `1:132211(+n)` | untested | vanilla uses only negative values |

## Tools

All in the project's `tools/` folder.

| Tool | Use for this topic |
|---|---|
| `tools/arenasetup.py` | the setup: `LOTS` (item to lot), `BASE_ITEMS`, `EDITIONS`, `EDITION_ITEMS`, `ITEM_WAIT` (3 s), `awards()` (checks every lot against the params, stops on a missing lot), `topups()` |
| `tools/choices.py` | `menu_talk(setup=, topups=)` builds the give states (`give_chain`), the gate and the edition top-ups |
| `tools/looseparams.py` | `ARENA_EDITS` holds the four lot edits and writes the loose params |
| `tools/arenalib.py` | `give_stub`, `item_entry`, `Arena.give_items` (the memory route) and `Arena.inventory` (reads both bag lists) |
| `tools/arenactl.py` | `give <item>[:qty[:upgrade[:infusion]]]`, 8 per call; `GIVE_ITEMS` (items a fight needs, given at fight start if missing) |
| `tools/resetsetup.py` | clears the setup flag once so the questions run again on the same save |
| `tools/ids.py`, `ids_v1.json` | the id register that freezes the setup and edition flags |
| `tools/testwatch.py` | logs flags and item counts during a test (B20d's counts) |
| `tools/offline_tests.py` | checks the lots, the stub's call order, `item_entry` clamps and the built give states |
| `tools/esd.py`, `tools/edd.py` | read the scripts and their developer names |

## Open questions

- Does a refused `1:131402` show the failed window (`1:130494`) on its own? No test note records one. [untested]
- What does `f131402(item, 1)` return for a held key item? Could it replace the 3 s fallback? [untested]
- What does the second argument of `1:131402` do? It is 1 in all 74 vanilla uses. [untested]
- A lot with several items: one pop-up listing all, or one each? How do `chance_lot_n` and `lots_dropped_num` pick? [untested]
- Can the command read `ItemLotParam2_Chr` or `_SvrEvent` rows? [untested]
- What is `infinite_lot_n`? It is 1 on every used slot we saw. [untested]
- What is in the 4 bytes between the two bag lists? [untested]
- Does `item_type_id` 9000 decide which list an item goes to? The exe was not read for it. [untested]
- What happens at 3,840 items (a full list)? [untested]
- A lot above an ordinary stackable's limit (say 150 Effigies): clamped, refused or given? [untested]
- Is the Torch's burn time capped, and how many seconds is one Torch? [untested]
- Does a memory-given item survive a quit before the next save? [untested]
- Can `1:132211` give souls with a positive number? [untested]
- Does a script from an event group of our own give items (the game's event scripts do it 18 times)? [untested]

## Related pages

- [Asking the player a question: YES/NO windows](yes-no-windows.md): the questions the setup is behind; the wait pattern for pop-ups
- [Vendor stock](vendor-shops.md): the other route for items; why keys and the Aged Feather are not sold
- [Memory map for PC 1.0.3.0](../tools/memory-map-pc.md): `GameManagerImp` and the rest of the trainer offsets (the bag layout stays here)
- [Editing params](../modding-scripts/editing-params.md): loose `ItemLotParam2_Other` rows
- [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md): the setup and edition flags
- [Writing ESD files](../modding-scripts/writing-esd-files.md): building the give states
- [Event and talk script commands](../modding-scripts/script-command-reference.md): other commands we proved
- [NPC menus](npc-menus.md): Talk, and the menus the setup hangs off
- [Live-testing notes](../modding-scripts/live-testing-notes.md): watching counts and flags during a test
