# Vendor stock: the 64-row cap, flag pages, free items and Trade lists (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- A shop is two params. An `NpcEventParam` menu row says "this NPC sells" (buy bit `Unk00` = 1) and gives an id range
  (`shopLineupStartRow` .. `shopLineupEndRow`). The `ShopLineupParam` rows inside that range are the stock. You can add
  rows by cloning an existing one, and you can give an NPC who had no shop a new one (Saulden, B28).
- **The shop window shows at most 64 items.** Four lists of 111, 229, 435 and 102 rows were all cut at 64 [live B28].
  The cut is on the list, not per tab [live B28, B29]. Rows that a flag keeps hidden do not count [live B30, inference].
- **To sell more than 64 items from one NPC, page the stock with flags**: one range, and each row carries an
  `enable_flag` / `disable_flag` that says which page it is on. A talk-script state flips the flags and reopens the
  same menu row [live B30]. One `NpcEventParam` row per page does not work: every page showed the same stock [live B29].
- **Free stock**: `price_rate` 0, `quantity` 255, `material_id` 0. The trap: **at price 0 the quantity picker
  disappears** (one purchase buys one item) [live B30]. Stackable items get a price of about 1 soul instead [live B31].
- **A Trade list** is a second list on the same menu row: `Unk00` bit 128 plus `shopLineupParamId` .. `Unk15`. With
  `material_id` 0 and `price_rate` 0 it is free. Any vendor can have one [live B32].
- Free levels, upgrades and infusions are wildcard edits that set the soul-cost fields to 0 on every row.

### The two params

Offsets are from the project's param reader. The names are its names; fields called `Unk` are still unexplained.

**`ShopLineupParam`**: 36-byte rows, 692 in the game. A shop reads every row whose id lies in its menu row's range.

| Field | Offset | Notes |
|---|---|---|
| `item_id` | +0 | the item (ItemParam id) |
| `Unk00` | +4 | meaning unknown. Values in the game: 10 (593 rows), 500 (85), 50, 6000, 100, 1000. **Not** the item's base price (equal on 9 of 691 rows) [data] |
| `enable_flag` | +8 | -1 = always; else the row shows only while this flag is ON [data, live B30] |
| `disable_flag` | +12 | -1 = never; else the row is hidden while this flag is ON [data, live B30] |
| `material_id` | +16 | an item the purchase uses up. Non-zero only on the Straid / Ornifex Trade rows (the boss soul) [data] |
| `duplicate_item_id` | +20 | FromSoftware sets it to the item's own id on 6 rows. Meaning untested; our rows use 0 [data] |
| `Unk01` | +24 | 0 on all but four rows |
| `price_rate` | +28 (f32) | multiplier on the item's `base_price` (ItemParam) [inference from the mod's price maths, live B31] |
| `quantity` | +32 | stock; 255 is what FromSoftware's always-in-stock rows use [data] |

**`NpcEventParam`**: 48-byte rows, 73 in the game. One row is one menu an NPC can open; an NPC may have several (the
talk script picks which row to open by state).

| Field | Offset | Notes |
|---|---|---|
| `Unk00` | +0 (u8) | menu options, as a bit mask. 1 = Buy, 128 = Trade. The other bits are on [NPC menus](npc-menus.md) |
| `shopLineupStartRow`, `shopLineupEndRow` | +4, +8 | the Buy range, both ends included |
| `shopLineupParamId`, `Unk15` | +20, +44 | the Trade range, start and end, both included [data] |
| `npcMenu_NameID` | +28 (u16) | the NPC; also the window title in `npcmenu.fmg` |

FromSoftware's own ranges are whole blocks: each NPC owns ids `<npcMenu id> * 10000` upward (Melentia, menu id 7540:
75400000-75409999). The largest Buy range of a real vendor holds 70 rows (Melentia) and the next 66 (Maughlin) [data].
Both gate many of their rows with flags. One debug row, 99999999, lists 501 rows (75409000-78309211) but nothing
opens it [data]; it is not evidence that a shop can show 501 items (B28).

### What the 64 cap counts

- **A shop shows only the first 64 rows of its range** [live B28]. Lenigrast had 229 rows in one range and stopped
  inside the weapons: swords were there, hammers and the materials at the end were not. Gilligan (111 rows),
  Maughlin (435) and Melentia (102) were all cut at 64 as well. The tester counted the window as 7 rows of 9 plus 1.
  The rows that went missing were the last ones in row-id order, so the cut follows that order [inference from B28].
- **Tabs do not each get 64.** The shop window groups items under tabs. In Lenigrast's first build the materials
  were the last rows of the range (214-229) and never appeared, which a per-tab limit would not do [inference from
  B28 + B29]. A later page of about 61 rows used two tabs (items and weapons) and showed [live B29].
- **Rows hidden by a flag do not count.** The mod's vendors keep every item in one range (Maughlin: 435 rows) and
  show 64 or fewer at a time by flags. Maughlin's seven pages, the last starting at the 375th row of the range, all
  worked [live B30]. So the game filters by flag first and then cuts at 64 [inference from B28 + B30].
- **Nothing about the limit is read from the exe yet.** The number is the observed one. FromSoftware's own Trade
  lists hold 55, 60 and 60 rows [data], which fits a 64 cap but proves nothing.
- The mod keeps every Buy page at 64 rows or fewer, and also holds each Trade list to 64 by build rule
  (`tools/shops.py`, `PAGE = 64`). The Trade list's own cap is untested. Two pages in the final build hold exactly
  64 rows (Maughlin page 1, Melentia page 1) [data]; no test counted their last row [untested].

### Flag pages

FromSoftware varies stock with `enable_flag` / `disable_flag`. Melentia's Lifegem rows are a pair: one with
`disable_flag` 203850 and `quantity` 10, one with `enable_flag` 203850 and `quantity` 255. Chloanne's Titanite Shard
rows use the same pair (flag 100952). Melentia's Brightbug is a chain of six rows, each enabled by one flag and
disabled by the next, with `price_rate` 1, 2, 3, 4, 6, 8 [data]. The mod uses the same mechanism as a page switch.

A vendor with `n` pages gets one range and a block of flags, `G` and `F1` .. `F(n-1)`:

| Page | `enable_flag` of its rows | `disable_flag` of its rows | Flags set to show it |
|---|---|---|---|
| 1 | -1 | `G` | all of `G`, `F1` .. off |
| `k` + 1 (k = 1 .. n-1) | `Fk` | -1 | `G` on, `Fk` on, the other `F` off |

All flags off therefore means page 1, so a fresh save needs no set-up. The flags are global event flags in a free
range (104600-104699, ten per vendor: `G` = 104600 + 10 × vendor index, `Fk` = `G` + k; no script, param, MSB or exe
reference was found for the range; see [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md)). Lenigrast has `G` =
104600, `F1` = 104601, `F2` = 104602 [data].

The page switch is a few states in the Majula talk script [data, read from the installed script]. For Lenigrast:

| State | Does |
|---|---|
| menu state | opens the library `6:sub(0, 220, 76400000, 0)` (developer name 【Lib】メニュー開始：汎用, "Lib: start menu, generic"), which calls `1:130461` (【FE】NPCウィンドウを開く, "open the NPC window") with the menu row. Its result `B9` picks the branch |
| `B9 == 9` ("More items") | goes to the **switch** state |
| switch | tests `f6(104602)` (フラグの取得, "get flag"): ON means page 3, so go to page 1. Else `f6(104601)` ON means page 2, so go to page 3. Else page 1, so go to page 2 |
| a go-state | `1:130101(flag, value)` (【フラグ】フラグを設定, "set flag") three times, to the pattern in the table above |
| a name state | `1:130465()` (【FE】NPCウィンドウで「何かを実行したか」をクリア, "NPC window: clear 'did something'"), then an OK window with the page's text (`6:sub(<text id>)`, library 【Lib】OKダイアログ, "OK dialogue") |
| after the window | `1:130464()` (【FE】NPCウィンドウで「何を選択したか」をクリア, "NPC window: clear 'what was chosen'"), then back to the menu state, which opens the **same menu row** again |

The new flags take effect on the reopen, so the shop reads its flags each time the menu opens [live B30]. This is the
difference from B29: the same menu row with different flags changes the stock; a different menu row with a different
range did not. The page lives in the save (global flags), so the next conversation starts on the page last shown
[inference; B30's pass covered switching, not a second conversation].

The option that turns the page is the game's own "Move path" option, renamed "More items". Talk was the first trigger
(B30) and was replaced on 8 Nov (B42, B43). See [NPC menus](npc-menus.md) for the bit and the rename.

**Page names.** Each switch ends in an OK window saying what is now on the page ("Now showing: straight swords,
reapers, ..."; for armour and rings "<first item> to <last item>"). 18 texts, one per page of the six paged vendors,
in `mapevent.fmg` slots checked free: 5301-5309, 1214-1219, 5807-5809 [live B32]. Slots in the 54xx range were avoided
out of caution, because event scripts use 5400 / 5410 / 5420; a later check found the game reads those as plain ids,
not as bases with an offset (see [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md)). Pages are cut at group edges (weapon class, armour set,
ring family), most-needed first, so a page name stays short. Window width limits are on
[Text that crashes the game](text-length-limits.md); other languages are on
[Game text and languages](game-text-and-languages.md).

### Prices, stock and the quantity picker

- **Free** = `price_rate` 0, `quantity` 255, `material_id` 0. 1,015 of the mod's 1,132 added rows have
  `price_rate` 0, and all 1,132 have `quantity` 255 [data].
- **The quantity picker needs a price.** At `price_rate` 0 Chloanne's titanite bought one at a time, with no
  up / right control, although her row matched FromSoftware's byte for byte except for the price [live B30, B31].
  The fix is a nominal price on the 117 items with `max_held_count` > 1 and a base price: `price_rate` = 1.5 /
  `base_price`, about 1 soul each. Weapons, armour, rings and spells stay at 0 [live B31]. Why price 0 removes the picker
  is not known [inference: the picker's maximum comes from souls ÷ price].
- **Quantity 255** is FromSoftware's "always in stock" [data]. Buying 10 or more in one purchase worked [live B31].
  We never tried to exhaust it [untested].

### Trade lists

A Trade list is a second list on a menu row, opened by the "Trade" option. It is separate from the Buy range.
FromSoftware uses it for Straid and Ornifex, who trade a boss soul for an item. Their menu rows carry `Unk00` 129
(Straid: Buy + Trade) and 141 (Ornifex: also Repair and Reinforce), a Buy range of their own, and the Trade range in
`shopLineupParamId` .. `Unk15` (76801000-76801999 for Straid) [data]. Their 175 Trade rows have `material_id` = a boss
soul, `price_rate` 1.0 (0.0 on Ornifex's second list, 77602000-77602999), and every one carries an enable or disable
flag [data]. So the game reads flags on Trade rows too [inference]. Ornifex has two menu rows that share the Buy range
but point at different Trade ranges, the free one on the second [data].

The mod gives Lenigrast one, to hand out the 56 weapon-type items those two trade (43 melee weapons, 7 shields,
4 bows, 2 chimes):

| Item | Value |
|---|---|
| Menu row 76400000 `Unk00` | 141 (13 in the game, plus Trade bit 128) |
| `shopLineupParamId`, `Unk15` | 76409000, 76409055 |
| Trade rows | 76409000-76409055: `material_id` 0, `price_rate` 0, `quantity` 255, both flags -1 |

The Trade entry lists the boss weapons and gives them away with no soul. **Bit 128 alone was enough**; Straid's `Unk08`
(8) was not copied [live B32]. Any Majula vendor can have a second list [live B32].

### Free levels, upgrades and infusions

These are plain field edits, set to 0 on every row of the param. The project's `tools/looseparams.py` writes them as
`Param:<row | *>:<field | glob>=value`.

| Edit | Rows | Values set | Effect |
|---|---|---|---|
| `PlayerLevelUpSoulsParam:*:soul_level_cost=0` | 852 | 852 | free level-ups at the Herald [live B28] |
| `ReinforceCostParam:*:reinforce_soul_cost_*=0` | 585 | 5,850 (10 per row) | upgrades cost 0 souls [live B3, B30] |
| `CustomAttrCostParam:*:infusion_*_cost=0` | 5 | 50 | infusions cost 0 souls [live B31] |

The game's params had 4,025, 40 and 851 non-zero values in those three groups; the installed ones have none [data].
The **materials stay**: an upgrade still needs its titanite and an infusion its stone. The mod sells those
(Chloanne, 16 items; the stackable ones at about 1 soul). In the first version Lenigrast sold them, but his 229-row list
was cut at 64 (B28), so they moved to a vendor of their own.

`PlayerLevelUpSoulsParam` is one of the params whose rows have no names at all (every row's name offset is 0), a layout
`tools/paramrows.py` refuses to rebuild, so rows were not added; field edits are enough here. See
[Editing params](../modding-scripts/editing-params.md).

### The mod's final layout

Every vendor has one Buy range starting at `<menu id> * 10000 + 5000` (Trade rows start at `+ 9000`), cut into pages of
64 or fewer. Counts and pages are from the installed params [data]; page sizes also match `tools/shops.py plan`. Keys,
quest items, boss souls and gestures are not sold: a design choice, not a limit.

| Vendor | Menu id | Buy range | Items | Pages | `G` |
|---|---|---|---|---|---|
| Lenigrast | 7640 | 76405000-76405169 | 170 melee weapons | 3 (57 / 60 / 53) | 104600 |
| Chloanne | 7620 | 76205000-76205015 | 16 upgrade materials | 1 | |
| Gilligan | 7040 | 70405000-70405099 | 100 shields, bows, ammo, Binoculars | 2 (53 / 47) | 104620 |
| Maughlin | 7610 | 76105000-76105434 | 435 armour pieces | 7 (64 / 62 / 61 / 62 / 63 / 62 / 61) | 104630 |
| Saulden | 7410 | 74105000-74105121 | 122 rings | 2 (62 / 60) | 104640 |
| Melentia | 7540 | 75405000-75405084 | 85 consumables, Soul Vessels | 2 (64 / 21) | 104650 |
| Shalquoir | 7770 | 77705000-77705016 | 17 multiplayer and covenant items | 1 | |
| Carhillion | 7660 | 76605000-76605067 | 68 sorceries, hexes, staves | 2 (54 / 14) | 104670 |
| Licia | 7690 | 76905000-76905036 | 37 miracles, chimes | 1 | |
| Rosabeth | 7630 | 76305000-76305025 | 26 pyromancies, flames | 1 | |

Lenigrast also has the Trade list above. In all, the mod added 1,132 rows: ShopLineupParam grows from 692 to 1,824.
Every added row has `quantity` 255, and FromSoftware's 692 rows are untouched [data]. Why the layout is this one
(everything for sale, price 0) is a design choice for a sandbox arena, not a game rule.

Chloanne and Shalquoir were added after B29: the first because the materials were lost past the cap, the second for the
multiplayer items. The vendors' presence in Majula and their menus are on [NPC menus](npc-menus.md).

## How to do it

This is a recipe for a free shop of any size on one vendor. `tools/shops.py` is a worked implementation.

1. **Pick the menu row(s).** Find the `NpcEventParam` rows whose `npcMenu_NameID` is the vendor. Read the talk script
   to see which row it opens. Every row of the same vendor should get the same range.
2. **Pick a clear id range.** The mod uses the vendor block `+ 5000`. Check that no row of that block is at or above the
   start (the tool stops if one is).
3. **Clone rows.** Copy an existing row that is always on, quantity 255, no material, no duplicate id (any one will do),
   then set `item_id`, `Unk00` (copy from an existing row of the same item; else the most common value for its
   category), `enable_flag`, `disable_flag`, `price_rate` and `quantity`. Rows must be sorted by id in the file.
   The file layout and adding rows are on [Editing params](../modding-scripts/editing-params.md). Check first that the param rebuilds
   byte-identically with no change (`tools/paramrows.py check`). `ShopLineupParam`, `NpcEventParam`,
   `ReinforceCostParam` and `CustomAttrCostParam` do (168 of the 205 params do); 37 whose rows have no names (name
   offset 0), such as `ItemParam` and `PlayerLevelUpSoulsParam`, are refused.
4. **Point the menu rows at the range.** `shopLineupStartRow` = first id, `shopLineupEndRow` = last id, and set
   `Unk00` |= 1. For a vendor with no shop, setting the bit and the range is enough (Saulden, B28).
5. **Count.** More than 64 visible rows means more than one page: go to step 6. Otherwise stop.
6. **Assign pages.** Cut the list into pages of at most 64. Page 1: `enable_flag` -1, `disable_flag` `G`. Page `k` + 1:
   `enable_flag` `Fk`, `disable_flag` -1. Take `G` and `F1` .. `F(n-1)` from a free global flag range.
7. **Write the switch** in the vendor's menu state of the talk script, as in the table in "Flag pages". The state
   that tests the flags must go from "page k" to "page k + 1" and wrap at the end. Always reopen the **same** menu row.
   See [Writing ESD files](../modding-scripts/writing-esd-files.md) for the script writer.
8. **Name the pages** with an OK window per page, with the text in a free text slot.
9. **Price the stackables.** For every item with `max_held_count` > 1 and `base_price` > 0, set `price_rate` to
   1.5 / `base_price`. Everything else stays at 0. Read `price_rate` as raw bytes when you check it: the project's
   param reader rounds floats to 4 places, so a small rate such as 1.5 / 7000 (0.000214) would read back as 0.0002.
10. **Add a Trade list (optional).** On the menu row: `Unk00` |= 128, `shopLineupParamId` = the first Trade row id,
    `Unk15` = the last. Add the Trade rows with `material_id` 0, `price_rate` 0, flags -1.
11. **Free costs (optional).** Add the three wildcard edits above.
12. **Read it all back** from the rebuilt params before installing: every item once, each page ≤ 64, each page's flags,
    the menu ranges, and (for Trade) the rows. `tools/shops.py check` does this.

## Pitfalls and what failed

| Test | What happened | Why / lesson |
|---|---|---|
| B28 | A 229-row Lenigrast list, and Gilligan, Maughlin and Melentia, were cut at 64 items. The materials at the end of Lenigrast's list were missing, so no upgrades or respec could be tried | The cap (see above). The 501-row debug row is no evidence of a bigger list |
| B29 | Pages as extra `NpcEventParam` rows (menu row id + 100 per page, each with its own range, switched from Talk) showed the **same items** on every page | Opening another row mid-conversation did not change the stock. Cause unknown. FromSoftware gives all of an NPC's rows the same Buy range and varies stock with flags |
| B30 | Flag pages passed. But Chloanne's items bought one at a time, with no quantity picker | Price 0 removes the picker. Fixed in B31 |
| B30 | Respec failed; Lenigrast could not infuse | Menu-option bits, not shop stock. See [NPC menus](npc-menus.md) |
| B31 | Fixed: stackables at ~1 soul show the picker; infuse and respec pass | `price_rate` = 1.5 / `base_price` |
| B32 | Passed first time: Trade list free, page windows named | `Unk00` bit 128 was enough; no `material_id` needed |
| B42 build (8 Nov) | The new "More items" and "options off" steps undid Saulden's buy bit | A build bug in `tools/shops.py`: it decoded each row from the original param (`P`) but wrote a copy (`T`). Later steps must read the copy, or they overwrite earlier edits |

More notes:

- **Do not size a page by the 64 alone.** Cut it at group edges, or the page name cannot say what is on it.
- **Page 1 must be the "all flags off" page.** In a fresh save every flag is off, so any other choice would open an
  empty shop.
- **Moving the range hides FromSoftware's own stock.** The mod's ranges start at `+ 5000` and so leave out the
  vendor's original rows (and their quest-flag staging, such as the Lifegem and Brightbug rows). That is intended
  here; a mod that wants the original stock plus its own must keep the range wide and count the visible rows.
- **Editing FromSoftware's own rows no longer shows in a moved shop.** `tools/looseparams.py` still carries a B3c edit
  to Melentia's Human Effigy row (75400607: price 0, quantity 255). Her range now starts at 75405000, so that row is
  outside it [data].

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| Edited `ShopLineupParam` rows take effect (price 0) | live | B3 (with the exe patch), B3b (loose params, no patch) |
| Quantity 255 and price 0 on a FromSoftware row work | memory read; shop not visited | B3c |
| A shop shows at most 64 items; the 4 lists cut at 64 | live, counted in the window | B28 |
| The cap is not per tab | live (materials tab absent); page of 61 over two tabs shown | B28, B29 |
| A new `NpcEventParam` buy bit plus range gives a new shop (Saulden) | live | B28 |
| One menu row per page shows the same stock on every page | live (fail) | B29 |
| Flag pages (one range; enable / disable flags; Talk flips and reopens) work on every vendor | live | B30 |
| Hidden (flag-off) rows do not count towards 64 | inferred from B28 + B30 | B28, B30 |
| Price 0 removes the quantity picker; ~1 soul restores it | live | B30, B31 |
| Chloanne, Shalquoir and Lenigrast's flags and ranges as listed | installed params read back | data, `tools/shops.py` |
| Trade list = `Unk00` bit 128 + `shopLineupParamId` .. `Unk15`; free with `material_id` 0 and price 0 | live; Straid / Ornifex rows read | B32, data |
| Straid's and Ornifex's Trade rows: `material_id` is a boss soul, 175 rows | data | `ShopLineupParam`, `NpcEventParam` |
| Free levels (852), upgrades (5,850), infusions (50) read back as 0 | data; live: level-up, upgrade and infusion at 0 souls | B3, B28, B30, B31 |
| FromSoftware stages stock with flag pairs and chains (Lifegem, Titanite Shard, Brightbug) | data | `ShopLineupParam` |
| The four shop params rebuild byte-identically | `tools/paramrows.py check` over the 205 params | tools |
| The installed Lenigrast switch: `B9 == 9`, the `f6` tests, the flag writes, the OK window, reopen | read from the installed talk script | data, `esd.py dump` |
| The page-name texts and slots | released text list; `tools/shops.py` | B32 |

## Tools

| Tool | For this topic |
|---|---|
| `tools/shops.py` | the shop generator. `plan` prints the pages per vendor; `check` builds both params from the originals and reads them back (pages ≤ 64, flags, menu ranges, Trade rows). Holds `VENDORS`, `MENU_ROWS`, `PAGE = 64`, `FLAG_BASE`, the price rule, the Trade lists, the page texts and the talk-script patches. `tools/offline_tests.py` also checks it (each item once, pages ≤ 64, FromSoftware's 692 rows untouched) |
| `tools/paramrows.py` | `check <folder>` rebuilds each param unchanged and compares bytes; `add` / `delete` for a row. Layout: header 0x40, 24-byte row entries sorted by id, data packed in id order |
| `tools/looseparams.py` | applies field edits (`ARENA_EDITS`, wildcards) and calls the generators in `REBUILD_FROM`; installs the loose params. See [Editing params](../modding-scripts/editing-params.md) |
| `esd.py` (`dump`), `edd.py` | read the Majula talk script and the developer names of its commands |

## Open questions

- **Where does 64 come from?** Not looked for in the exe. Does the Trade list share the cap?
- **Direct test of "hidden rows do not count".** A range of 80 rows where 70 are visible at once would confirm that
  the 65th visible row is the one that goes. B28 (all visible) and B30 (pages) together imply it.
- **Why did B29's other-row pages show the same stock?** FromSoftware's Ornifex has two menu rows whose Trade
  ranges differ (77601000 and 77602000), so at least the Trade range is read per row. Is the Buy range cached per NPC
  or per conversation? Untested.
- **Can a Trade list be paged by flags?** Its rows have flags in FromSoftware's data, so probably. Untested.
- **A price the picker likes that rounds to 0.** The picker needs a non-zero price; is `price_rate` tiny but non-zero
  enough? Untested.
- **Exhausting 255.** Does the stock fall at all? Does a weapon at quantity 255 stack in the bag (every weapon takes
  its own slot, a design estimate)?
- **What `Unk00` (ShopLineupParam), `Unk01` and `duplicate_item_id` do.**
- **The last row of a 64-row page** (Maughlin page 1, Melentia page 1): check by eye.

## Related pages

- [Editing params: the encrypted regulation vs loose params](../modding-scripts/editing-params.md): how the edited params load, and the row-adding layout.
- [NPC menus](npc-menus.md): option bits, "More items", Talk, first meetings, vendors' presence in Majula, the respec.
- [Event flags and free ids](../modding-scripts/event-flags-and-free-ids.md): how the 104600-104699 range was found and checked.
- [Writing ESD files](../modding-scripts/writing-esd-files.md): the tool behind the page-switch states.
- [Event and talk script commands](../modding-scripts/script-command-reference.md): the commands used here (`130101`, `130461`, `130464`, `130465`).
- [Giving the player items](giving-items.md): the other route, item lots, for what a shop should not sell.
- [Text that crashes the game](text-length-limits.md) and [Game text and languages](game-text-and-languages.md): the page-name windows.
- [Asking the player a question](yes-no-windows.md) and [Adding dialogue options](adding-dialogue-options.md): the N-choice list, a different window from the shop.
