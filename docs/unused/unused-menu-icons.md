# "Unknown" menu icons from a community page, matched to the Network Test (2026-10-26)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

**Update 2026-11-16 (`wiki-cut-icons.md`):** all 15 are now found. They are exact cells of an older draft of the
shared icon sheet, the Network Test's loose `menu\tex\02\Icon_tex.tpf` (US and JP), including #7 at (4,2) and #12 at
(8,2). The matches below stand as near copies. Also corrected there: vanilla PC (every build) does have
`menu\04.febnd` with the level-up sheet; PS3 retail and PC Scholar don't.

The author pasted [a table of 15 "unknown" DS2 menu icons](../network-test/wiki-unused-content-crossref.md) from a website (screenshot, icons ~39 px). Question: can they be
matched to the Network Test's icons at that resolution? **Yes**: the game's cells are 26 px, so the website copies
are about 1.5x the original; template matching (OpenCV, normalised correlation, grey, scales 20-44 px) scored the true
matches 0.81-0.93, and every pairing below was checked by eye (`unused-menu-icons-compare.png`). Textures decoded with
`..\..\DS2 Network Test\tools\ps3tpf.py` (PS3 TPF: DXT1 / DXT5, and format 9 = swizzled A8R8G8B8); PC TPFs wrap plain
DDS files.

| # | website description | Network Test | retail (v1.00, PS3 Scholar, PC) |
|---|---|---|---|
| 1 | unknown (♂ ♀ / alchemy-like symbols) | shop icon sheet `Icon_05_Shop` (row 1, col 0) | same sheet, present |
| 2 | "Shop screens, unused NPC trading" (scales) | `Icon_06_Blacksmith` (0,0), also `Icon_05_Shop` | present |
| 3 | unknown (wing) | **level-up sheet** `Icon_04_Priestess` cell (1,4) | absent |
| 4 | "cancelled Trap evasion skill" (spread-eagle figure) | level-up sheet (3,1) | absent |
| 5 | unknown (star) | level-up sheet (2,2) | absent |
| 6 | "alternate ring slot" (ring) | **covenant sheet** `Icon_10_Covenant` (0,0) | absent |
| 7 | "alternate covenant rank" (medal) | **not found** | not found |
| 8 | unknown (crossed blades with hook) | level-up sheet (0,3) | absent |
| 9 | unknown (wing with blade) | level-up sheet (1,3) | absent |
| 10 | unknown (armour struck by a blade) | level-up sheet (2,8) | absent |
| 11 | unknown (anchor-like) | level-up sheet (3,3) | absent |
| 12 | unknown (star over a torso) | **not found** (the sheets' starbursts are a different design) | not found |
| 13 | unknown (loop around a star) | level-up sheet (3,4) | absent |
| 14 | unknown (feather and figure) | level-up sheet (1,6) | absent |
| 15 | unknown (stone, "x?") | `Icon_05_Shop` (2,6) | present |

Cells are (row, column) of 26 px from the top left.

## The two Network-Test-only sheets

- `menu\04.febnd.dcx` = the Emerald Herald / level-up menu (`l04_01_Priestess.flo` + `Icon_04_Priestess.tpf`,
  256 x 128 DXT5, 35 icons in 4 x 9 cells + a partial fifth row): `unused-menu-icons-nt-levelup-sheet.png`.
  **Retail has no `04.febnd` at all** (v1.00, PS3 Scholar, PC), so the whole sheet is gone.
- `menu\10.febnd.dcx` = covenant menu: `Icon_10_Covenant.tpf` (64 x 64, format 9) with four icons: ring, crossed
  blades, three figures, chalice (`unused-menu-icons-nt-covenant-sheet.png`). Retail's `10.febnd` has no textures.
- Searched in retail and not found: every menu binder (`menu\*.febnd`), loose `menu\tex` files, the language
  texture binders (banners only) and the item icons. So the website's copies of #3-#14 most likely come from the
  Network Test (or another pre-release build), not from retail files.

## Layout decode (2026-10-26, same day): the sheet is orphaned in the Network Test too

Partial `.flo` reader: `..\..\DS2 Network Test\tools\flo.py` (header, texture table, sprite records; the node tree,
keyframes and text nodes are recognised but not decoded). Findings, over all 46 US Network Test layouts, the JP
Network Test's and v1.00's:

- **No layout references `Icon_04_Priestess`**, in either Network Test: it is not in any layout's texture table,
  and the EBOOTs name no texture at all (textures are reached only through layout tables). The sheet is packed into
  `04.febnd` and `18.febnd` but nothing can display it. The level-up layout (`l04_01_Priestess.flo`) draws its icons
  from the shared `Icon_tex` sheet (25 sprite records) and its frame from `04_Priestess`.
- `Icon_10_Covenant` (#6 ring) and `Icon_05_Shop` are listed in their layouts' texture tables but have no fixed
  sprite: if used, code picks the cell at run time.
- On the level-up sheet, 22 cells are the same art as an `Icon_tex` cell (match 0.75-0.95); **13 are unique art**
  found nowhere else: the nine website icons plus (0,5) a framed starburst, (1,2) a "?", (1,5) a star, (3,2) a star
  in a circle. The unique cells don't line up slot-for-slot with `Icon_tex`, so the sheet isn't an earlier draft of
  it in the same order.
- Labels are not in layouts: the stat names exist only as the first line of each `iconHelp.fmg` entry, attached by
  code in the order of the EBOOT tables. Icons are fixed per layout.
- Network Test vs v1.00 `Icon_tex`: 23 of 81 cells were redrawn (mostly green covenant / status icons and armour
  icons); layouts use 77 cells in the Network Test, 75 in v1.00; only (5,0) (scales with a cross, bonfire and
  level-up layouts) and (8,2) (in-game menu) are used by the Network Test alone.

**So the website icons can't be paired with stats from the data**: they were already unhooked in the Network
Test, with no layout, table or code reference. Pairings like "#4 = trap evasion" can only come from the pictures.
The Network Test stats retail cut, which the unique art could have been drawn for: Resistance, Agility, Evasion,
Action speed, Trap disabling, Break ATK, Break resist.

## What the level-up icons stand for: not established

The Network Test's executable has the help-text lists per screen (`iconHelp.fmg` ids in display order at VA
0x159b49c, 0x159c828, 0x159d0f4, 0x159d7c4). They include [the stats retail cut](../mechanics/agility-timeline.md#3-vanilla-is-the-nt-system-edited-in-place): Resistance 909012, Agility 909013,
Evasion 909037, Action speed 909038, Trap disabling 909039. **[Break ATK (909027) and Break resist (909035)](../network-test/break-element.md#network-test) have text
but appear in none of the lists**, so the Network Test no longer displays them. The menu layouts (`.flo`) hold
neither sprite names nor help-text ids in plain form, and the text binding is in code, so the icon-to-stat pairing
was not recovered. Ways to settle it: read the Network Test's status / level-up screens live in RPCS3, or decode the
`.flo` layout format (element records with texture cells).
