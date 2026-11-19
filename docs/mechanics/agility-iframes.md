# Retail roll i-frames: how derived Agility becomes invincibility (2026-10-16)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Question (from the Network Test's i-frame work, `..\..\DS2 Network Test\Ghidra Project\export\win\s18\findings.md`
sections 2-3b): the demo's roll i-frames are `3 + 14 x ratio` frames with the ratio taken from
PhysicalStatsPerLevelStatValuesParam column +0x64. Retail's community table (6 -> 17 frames over AGL 85-120) fits the
same window with a ratio of 3/14 -> 1.0. Where does retail's ratio come from? Static work only (Ghidra DS2V headless,
the shared `tools`, params); the live evidence is the community's measured table.

**Answer.** Retail kept the demo's mechanism (same TAE windows, same 110300 handler, same "value x (1 - penalty)"
builder) and changed three things:

1. **The row.** [Agility is no longer an attribute](agility-timeline.md). The derived-stat builder picks the PhysicalStats row
   `level = (3 x ADP + ATN) / 4` (integer, rounded down; any result outside 1-99 uses row 1) from the *effective*
   attributes (base + equipment bonus, each clamped to 1-99). This is exactly the wiki's "investment".
2. **The columns.** The demo's three Agility percentages (+0x64 / +0x68 / +0x6c) moved to +0x68 / +0x6c / +0x70
   (one resistance column was inserted before them) and were retuned to 20 -> 100 / 25 -> 100 / 25 -> 100. The demo used
   the first for both rolls and backsteps and never read the second; retail gives each evasion its own column:
   **backstep = +0x68, roll = +0x6c, landing roll = +0x70.**
3. **The penalty.** [PlayerLackOfStatsParam row 18](armour-requirement-penalties.md) grew from (20, 50) to (30, 50, 0, 50): the armour-requirement term is
   now 30-50 % (demo 20-50 %), and a new **insufficient-stamina** term takes off up to 50 % more.

```
retail, 30 fps:
  level = floor((3 x ADP_eff + ATN_eff) / 4)                      ADP_eff = clamp(ADP + bonus, 1, 99), same for ATN
  base  = PhysicalStatsPerLevelStatValuesParam[level].col / 100   col: backstep +0x68, roll +0x6c, landing roll +0x70
  A     = lack  > 0 ? 30 + 20 x lack  : 0                         lack = armour stat-requirement shortfall, 0..1
  B     = short > 0 ?  0 + 50 x short : 0                         short = 1 if stamina <= 0,
                                                                          (cost - stamina) / cost if stamina < cost, else 0
  ratio = base x (1 - clamp((A + B) / 100, 0, 1))
  roll i-frames     = max(4, 3 + floor(14 x ratio))   (animation 120020/120030: 110100 to 0.133 s + 110300 0.1-0.567 s)
  backstep i-frames = floor(11 x ratio), from frame 5  (110011/110031: 110300 0.133-0.5 s)
  landing i-frames  = floor(13 x ratio), from frame 1  (140020: 110300 0-0.433 s; no stamina cost, so no B term)
  menu AGL          = MenuStatsParam[level] +0x32 (u16)
```

With armour requirements met and enough stamina this reproduces the community spreadsheet (linked from the wiki)
**for all 99 investment rows and all four columns it gives: menu AGL, roll, backstep and landing roll, 396 values, 0
mismatches** (`tools\agility_iframes.py check`). The near-integer rows (14 x ratio = 9.002 at row 19, 11.004 at 33,
11.998 at 52, 12.99 at 71) all land on the side the floor predicts. Other columns don't fit: the roll with +0x68 gives 20
mismatches, the backstep with +0x6c gives 13.

## 1. Curves: demo vs retail (armour requirements met, enough stamina)

Demo (NPUB90979): level = the Agility attribute. Roll and backstep ratio = +0x64 / 100, landing = +0x6c / 100 (the
two columns are equal).

| demo Agility | ratio | roll frames | backstep | landing roll |
|---:|---:|---:|---:|---:|
| 1 | 0.150 | 5 (0.167 s) | 1 | 1 |
| 8 | 0.167 | 5 | 1 | 2 |
| 16 | 0.183 | 5 | 2 | 2 |
| 31 | 0.215 | 6 | 2 | 2 |
| 39 | 0.232 | 6 | 2 | 3 |
| 58 | 0.273 | 6 | 3 | 3 |
| 65 | 0.287 | 7 | 3 | 3 |
| 75 | 0.309 | 7 | 3 | 4 |
| 89 | 0.358 | 8 | 3 | 4 |
| 91 | 0.366 | 8 | 4 | 4 |
| 96 | 0.387 | 8 | 4 | 5 |
| 99 | 0.400 | 8 (0.267 s) | 4 | 5 |

Retail v1.00 (BLUS41045): level = floor((3 ADP + ATN) / 4) = the wiki's investment. Rows where the roll gains a frame:

| row (investment) | menu AGL | backstep ratio (+0x68) | roll ratio (+0x6c) | roll frames | backstep | landing roll |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 85 | 0.200 | 0.250 | 6 (0.2 s) | 2 | 3 |
| 2 | 85 | 0.248 | 0.295 | 7 | 2 | 3 |
| 5 | 85 | 0.320 | 0.362 | 8 | 3 | 4 |
| 8 | 88 | 0.392 | 0.430 | 9 | 4 | 5 |
| 12 | 92 | 0.480 | 0.512 | 10 | 5 | 6 |
| 16 | 96 | 0.560 | 0.587 | 11 | 6 | 7 |
| 19 | 99 | 0.620 | 0.643 | 12 (0.4 s) | 6 | 8 |
| 25 | 105 | 0.700 | 0.718 | 13 | 7 | 9 |
| 33 | 110 | 0.772 | 0.786 | 14 | 8 | 10 |
| 53 | 113 | 0.852 | 0.861 | 15 | 9 | 11 |
| 72 | 116 | 0.928 | 0.932 | 16 | 10 | 12 |
| 99 | 120 | 1.000 | 1.000 | 17 (0.567 s) | 11 | 13 |

`tools\agility_iframes.py steps` prints every row where any count changes (`table` prints all 99). In short: rolls go
from 5-8 frames (demo) to 6-17 (retail), backsteps from 1-4 to 2-11, landing rolls from 1-5 to 3-13. Retail's weakest
roll (row 1, 0.25) is weaker than the demo's best (0.40) as a ratio, but retail reaches the full 0.567 s window at
row 99.

Penalty examples (retail): one armour piece 1 point short of a requirement of 13 (lack 1/13, as on the eighteenth
session's demo character) gives A = 31.5 %, so ratio x 0.685 (demo: x 0.777). Rolling with stamina at half the roll's
cost gives B = 25 %, so at row 99 the roll drops from 17 to 13 frames. If A + B reaches 100 % the ratio is 0: the roll
keeps only its fixed 110100 part (4 frames, 0.133 s) and a backstep has none.

## 2. Code path, demo and retail side by side

`p` below = `[chr+0x368]+4` (demo) / `[chr+0x378]+4` (retail, `FUN_005b261c` returns its argument + 4). The
eighteenth session's "pparam+0x98" is the demo's p+0x94 (index 0); the retail values are at pparam +0xac / +0xb0 /
+0xb4 for indices 0 / 1 / 2 (useful for [a later live cheat-table entry](../tools/cheat-table-port.md#2-per-entry)).

| step | demo NPUB90979 | retail BLUS41045 v1.00 |
|---|---|---|
| TAE event dispatcher | `FUN_005bdba0`, 110300 case, raise at 0x5bdfd8 | `FUN_007aeec8` (OPD 0x1cd4560), 110300 compare at 0x7af18c (`xoris r0,r9,0xfffe; cmpwi cr7,r0,-0x5124`), raise at 0x7af244 |
| state object S | `[chr+0x84]` | `[chr+0x94]` |
| ratio | S+0x90 | S+0xa4 |
| i-frame counters raised | S+0x3dc, S+0x3ec | S+0x5a0, S+0x5b4 |
| handler test | progress (t - max(start, 0)) / (end - max(start, 0)) < ratio | identical |
| roll start (flags 7 / 6 / 8) | `FUN_005c13a0`, store 0x5c14fc (live-verified in session 18), vslot +0x6c -> index 0 | `FUN_00864d30` (store 0x864e8c) and `FUN_007b2864` (0x7b2990), vslot +0x6c -> **index 1**, stamina cost from `FUN_0083a738` |
| backstep start (flags 10 / 11) | `FUN_005c15c8`, store 0x5c1678, vslot +0x6c -> index 0 | `FUN_00864fa0` (0x8650a8) and `FUN_007b2748` (0x7b2808), vslot +0x70 -> **index 0**, cost from `FUN_0083a898` |
| landing roll (by elimination and the fit) | `FUN_005cab14`, store 0x5caba0, vslot +0x98 -> index 2 | `FUN_007beb84`, store 0x7bec18, vslot +0x9c -> index 2, cost 0 |
| evasion-stat object | `[chr+0x2f4]`, vtable 0x19b3b50 | `[chr+0x2e4]`, vtable 0x1c2abe0 (one slot inserted before +0x6c) |
| index thunks | +0x68 -> 1 (0x6412ac, no callers found), +0x6c -> 0 (0x6412f0), +0x98 -> 2 (0x641378) | +0x6c -> 1 (0x849178), +0x70 -> 0 (0x8491bc), +0x9c -> 2 (0x849244, passes 0.0 as the cost) |
| ratio builder | vslot +0x108 `FUN_006421c8`: value(i) x vslot +0x38(lack, row 0x12) | vslot +0x124 `FUN_00849990`: value(i) x (1 - penalty), via vslot +0x3c (0x7a301c = 1 - vslot +0x34) |
| penalty | (20 + 30 x lack) / 100 if lack > 0 (PlayerLackOfStatsParam row 18 = 20, 50) | vslot +0x34 `FUN_0084ab7c`: {row 0x12, lack (`FUN_007a4b28` mode 2, equipment +0x214), short (`FUN_007a3420` -> `FUN_0083a428`, stamina at chr+0x140)}; row 18 = 30, 50, 0, 50; x 0.01, clamped 0..1 |
| value getter | `FUN_0065738c`: p + 0x94 + 4i | `FUN_00860408`: p + 0xa8 + 4i |
| where the values come from | derived-stat builder (prologue 0x659828), 0x659e2c-0x659ec8: RelatePhysicalStat bytes 26-28 (= attribute 10, the spendable Agility) pick the row, columns +0x64 / +0x68 / +0x6c x 0.01 -> p+0x94 / +0x98 / +0x9c | `FUN_008609f0(out = p+0x2c, attrs = p+0x16)` from `FUN_008626b8`; Agility block 0x861c4c-0x861d68 computes the row inline from ADP (+0x10) and ATN (+0x6), no table lookup; +0x64 / +0x68 / +0x6c / +0x70 x 0.01 (constant 0x3c23d70a) -> out +0x78 / +0x7c / +0x80 / +0x84 = p +0xa4 / **+0xa8 / +0xac / +0xb0** |
| effective attributes | (not traced) | `FUN_008626b8`: p+0x00..0x14 + an 11-byte bonus array (`FUN_00490fa0`, bytes +0x2fb..+0x305 of an equipment-side object), each clamped to 1..99 (`FUN_005b5f60`), stored at p+0x16 |
| menu AGL | (not checked) | `FUN_00863080`: MenuStatsParam[(3 x ADP + ATN) / 4] +0x32, read at 0x864074 |

So the retail roll reads the column the demo left unused (index 1), the backstep keeps index 0, and both now pass their
stamina cost into the builder.

## 3. The wiki (darksouls2.wiki.gg, Adaptability#Agility) against the game

| wiki | game | verdict |
|---|---|---|
| Agility rises with ADP and ATN, ADP three times as fast | row = (3 ADP + ATN) / 4 (builder and menu alike) | agree |
| "Investment" = ADP x 0.75 + ATN x 0.25, rounded down | the same integer: `(2a + a + b) >> 2` with `srawi/addze` (truncation; equal to floor for these positive values) | agree, exactly |
| Investment tiers describe the true i-frames; the menu value can hide them; 110 covers 7 tiers | the investment **is** the param row index; every Agility effect reads that row. The menu number is just another column (MenuStatsParam +0x32) that stays flat over 7 rows above 110. There are no hidden decimals in the game | agree on the effect; the "(a)-(g)" sub-values are simply rows that share a menu number |
| AGL calculated below 85 is shown as 85; SL -44 = 85(a) | MenuStatsParam rows 1-5 all hold 85; nothing is computed or clamped. Attributes all 1 give row (3 + 1) / 4 = 1 | agree on what you see; the mechanism is a table, not a floor |
| Bandit (ADP 3, ATN 2) = tier 2, 7 frames; Warrior / Cleric = tier 5, 8 frames | (9 + 2) / 4 = 2 -> 7; row 5 -> 8 | agree |
| Breakpoints 92 / 96 / 99 (and 88, 105) always match the menu | rows 8 / 12 / 16 / 19 / 25 have menu values 88 / 92 / 96 / 99 / 105 (menu = row + 80 for rows 6-29) | agree |
| 96 AGL = 11 frames; 110 = 13 or 14; backstep at 112 = 8 or 9 | row 16 -> 11; rows 30-32 -> 13, 33-36 -> 14; rows 44 -> 8, 45-50 -> 9 | agree |
| Hard cap 120 with ADP and ATN at 99 | row 99: menu 120, ratio 1.00 (the whole 110300 window) | agree |
| Agility, not equip load, sets i-frames; equip load only changes distance | the ratio has no equip-load input; the heavy roll 120030 has the same windows as 120020 | agree (true in the demo too) |
| Rolling with insufficient stamina reduces i-frames | new in retail: up to -50 % of the ratio, scaled by (cost - stamina) / cost | agree, and the code gives the size |
| (not on the wiki) armour stat requirements | -30 to -50 % of the ratio when an armour piece's STR / DEX / INT / FTH requirement isn't met (demo -20 to -50 %) | game-only detail |
| Item use multiplier changes only between 90 and 100 AGL | column +0x78: 0.975 / 0.97 up to row 10, rises from row 11 (AGL 91), 1.2 from row 20 (AGL 100) | agree (param only, use not traced) |
| Weapon swap 120 % at 93 or less, 145 % at max | column +0x74: 1.2 through row 13 (AGL 93), 1.45 at row 99 | agree (param only) |
| Guard raise 100 % at 89 and under, 150 % at max | column +0x7c: 1.0 through row 9 (AGL 89), 1.5 at row 99 | agree (param only) |
| Breakpoints described "at 60 frames per second" | the chart's counts are 30 fps game frames: 6 frames = 0.2 s at 85(a), 17 frames = the 0.567 s window | wording differs; the numbers are 30 fps frames |
| Equipment that boosts Adaptability (Simpleton's Ring etc.) | the row is computed from effective attributes (base + an 11-value bonus array, clamped 1-99) | consistent; the bonus source wasn't traced to the ring (see [SL1: Large Club vs Old Knight Hammer](sl1-large-club-vs-old-knight-hammer.md#fourth-ring-simpletons-vs-royal-soldiers2-and-roll-i-frames-added-2026-11-01)) |
| 70 % equip-load backsteps: same counts, starting at frame 10 instead of 5 | the count fits the same ratio; the 5-frame delay isn't in 110011 / 110031 (both 0.133-0.5 s). 110051 / 110071 (0.333-0.667 s) would give one frame fewer and start at frame 11 | open |

## 4. Limits and open points

- Static only. The live proof is the community's measured table (396 values); the demo's roll path was verified live in
  session 18 and the retail functions are its structural twins (same flags, same follow-on code). Which state number
  the flags 7 / 6 / 8 and 10 / 11 mean was not decoded; the roll / backstep split rests on that twin and on the fits.
- The armour `lack` value (retail equipment +0x214, demo +0x21c) and chr+0x140 = current stamina are read by analogy
  with the demo and by use (the stamina function only makes sense as stamina vs cost); neither was re-traced in retail.
- Each state has two writers in retail (`FUN_00864d30` / `FUN_007b2864`, `FUN_00864fa0` / `FUN_007b2748`); their
  difference (lock-on, another controller, network copy?) wasn't looked at.
- The frame rule `3 + floor(14 r)` is the observed count; it matches the handler if a frame's i-frame test uses the
  frame's end time. At r = 1 the 17th frame lies exactly on the window end, so float timing decides it. The data agree.
- The 70 % backstep delay (section 3, last row) is unexplained.
- Scholar (BLUS31528) not checked (see [balance: vanilla vs PS3 Scholar](../builds/balance-vanilla-vs-scholar.md)).

## 5. Files

- `findings\agility-iframes-community.tsv`: the community's raw table (investment, menu AGL, roll, backstep, landing
  roll), copied from the "DS2 Raw iFrames" tab of the spreadsheet linked from the wiki page (the "Fromsoft Framedata
  Charts" sheet), 2026-10-16.
- `tools\agility_iframes.py check | table | steps`: both curves from the params and the check above.
- `timeact\c000100_pl.tae`: the retail player TAE (`DS2_GAME=vanilla gamedata.py x c000100_pl.tae timeact`). Its
  i-frame windows are the demo's: 120020 / 120021 / 120030 / 120031 (110100 -0.167-0.133 + 110300 0.1-0.567), 110011 /
  110031 (110300 0.133-0.5), 110051 / 110071 (0.333-0.667), 140020 (0-0.433).
- Shared tools used: `tae.py`, `gamedata.py`, `elfread.py`, `immscan.py`, `ppcdis.py`, `tocrefs.py`; Ghidra
  `FuncAt.java` / `DefineAndDecomp.java` on DS2V and DS2NT_mac (read-only).

## Related pages

- [Port: player character](../tools/port-character.md): carries these retail offsets (state object, i-frame counter, roll ratio, pparam) into the full Network Test to retail character table.
