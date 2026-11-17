# Japanese network test (NPJB90625) vs Western network test (NPUB90979), first pass (2026-10-16)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Static comparison only. US = the Western test as installed (APP_VER 01.00, `..\..\DS2 Network Test`); JP = this
folder (base 01.00 + updates 01.02 / 01.03 / 01.04). Dates are the binder stamps (YY, month letter with B = January,
day, hour letter, minute; see `..\..\DS2 Vanilla\findings\agility-timeline.md`).

## 1. Order of the builds

| build | data packed | notes |
|---|---|---|
| JP 01.00 | 23 Aug 2013 23:20 (GameData, regulation 23:21) | |
| US 01.00 | 17 Sep 2013 16:09 (GameData, regulation 16:10) | |
| JP 01.02 | regulation 2 Oct 2013 17:07; menu 02 and model c1330 3 Oct 2013 | loose files override JP 01.00's GameData |
| JP 01.03, 01.04 | no data, EBOOT only | server protocol 0x00020103 / 0x00020104; not datable from the files |

It is one line of development: **JP 01.00 -> US 01.00 -> JP 01.02**.
- JP 01.00 vs US: 226 of 231 params byte-identical. US has, on top: ArmorParam (314 rows: requirement words +0x2c/+0x30
  changed, 3 new rows), ChrNetworkPhantomSoulRateParam (4 rows changed, 5 new), ItemParam (7 rows: id field +0x0),
  NetworkParam (1 row), `SpEffectRing.emevd`.
- JP 01.02 has the US ArmorParam, ChrNetworkPhantomSoulRateParam and `SpEffectRing.emevd` byte for byte; ItemParam and
  NetworkParam changed again (ItemParam row 0's id field is back to -1 as in JP 01.00); **and more**: ChrEquipLoadParam (3 rows), ChrMoveParam
  (row 100), ItemParam (123 rows changed, **16 new rows: the retail Aurous, Alva and Black Witch armour sets and the
  Ring of Thorns**, with no names in the test's text), ItemTypeParam (11), WeaponAttackMotionParam (77), WeaponTypeParam
  (1), NetworkParam; and 15 new param names that exist in retail (DemoParam, EventFlagListParam, NpcPlayerStatusParam,
  EnemyDefaultLogicParam, EquipCullingTargetParam, EnemyBindParam_m9000/9003/9201, MapTargetDirParam for 7 maps) as
  **empty (0-byte) entries**.
- JP 01.02's menu 17 is byte-identical to the US one (17 Sep); its menu 02 (the in-game menu, `l02_01_In-Game.flo`)
  and model c1330 are newer (3 Oct).
- Balance moving towards retail in JP 01.02: ChrEquipLoadParam row 17 = 0 / 0 / 0 / 0 / 70 (US 12.5 / 17.5 / 25 /
  70, vanilla = JP 01.02), row 9 25 / 35 / 50 / 90 (US 40 / 56 / 80 / 90, vanilla 25 / 35 / 70 / 90), row 8 halfway.

## 2. What is the same

- **Text:** 420 of 430 menu FMGs byte-identical (all languages, including the attribute help 909004-909049 with
  Resistance, Agility, Evasion, Action speed, Trap disabling). The other 10 are TitleMenu, one entry each:
  "Lock-on(Press right stick)" (US) vs "(press right stick)" (JP 01.02).
- **The stat system:** identical in JP 01.00, US 01.00 and JP 01.02: PhysicalStatsPerLevelStatValuesParam 300 x 136,
  RelatePhysicalStatToLevelStatParam 36 entries, EventCommonParam trap weights 25 / 35 ..., EventCommonParamFloat 22
  rows with the 3.0 / 4.5 trap fuse.
- **Executables:** every JP version (01.00, 01.02, 01.03, 01.04) keeps the stats-upload names `avert_invincible_time`,
  `action_speed`, `trap_fail_probability` and `taikyuryoku` (Resistance); none has `tekioryoku` (Adaptability) or
  retail's `physical_toughness` / `stone_resist`. JP 01.02 / 01.04 are the same size as the US EBOOT.elf (28,090,320
  bytes) but laid out differently (TOC 0x1a97350 vs 0x1a97300), so they are separate builds.

## 3. What it adds to the Adaptability timeline

The main game's content was still flowing into the test on 2-3 October 2013 (retail armour sets, retail balance values,
retail param names), and that update still carries the Network Test stat system everywhere: params, text and code.
That is consistent with the change coming after early October, but it cannot prove it: the test's code reads the stat
tables in the old layout, so the team could not have taken a changed table into the test even if the main game had one
(retail's per-level table rows are 148 bytes, not 136). The 15 retail params arriving as empty entries show the same
pattern: names merged, data not. The window stays 17 Sep 2013 - 14 Jan 2014 for certain; "after about 2 Oct" is the
likely reading. Nothing found here can be dated later than 3 Oct 2013.

## 4. Second pass: world data, animations, player movement (2026-10-16 afternoon)

Every GameData entry hashed in both builds (JP = 01.00 archive with 01.02's loose files on top; scratch scripts, not
kept): **3,424 of 3,444 US files identical, none missing**. The 20 that differ are all 01.02 loose files: menu 02,
the ten TitleMenu FMGs, model c1330 (.bnd only), the player animation bundle `c0001.anibnd.dcx` and six
`c0001_c<enemy>.extanibnd`. So maps, enemy placement, events and every TAE (including `c000100_pl.tae`: i-frames,
attack windows) are the same. JP adds 175 entries: 96 also loose in the US install (sfx, sound params, fonts, shaders),
2 different (server lists), 77 new: 46 player `extanibnd` for enemies outside the test (retail content again), debug
fonts (`dbgfont12`, `dbgfont14h`), debug / GUI shaders (`shader\debug`, `guishader`, `flverdebugshader.bnd`) and
`debug\network` server lists.
- `c0001.anibnd`: 867 -> 882 files, 778 identical; mostly attack animations (some fist / rapier / jump / plunge variants
  dropped, new spear and straight sword 0330-0336 series), a few stagger / death animations.
- **ChrMoveParam row 100 (the player)**, US -> JP 01.02 (retail): out-of-bounds kill time +0x150 **60 -> 7 s** (7);
  the angular speeds (multiples of pi rad/s) mostly drop by 40-60 %: +0x2c 1080 -> 640 deg/s (720), +0x30 1260 -> 540
  (1080), +0x54 1440 -> 640 (520), +0x58 720 -> 440 (640), +0x78 1440 -> 900 (1600), +0x9c 1440 -> 900 (1080), +0xa0
  720 -> 421 (540); +0x27c / +0x28c / +0x29c / +0x2ac / +0x2bc (20 / 50 / 40 / 30 / 30) -> 0 as in retail. Which turn
  (walk, attack, lock-on) each field is has not been tested.
- Against retail: WeaponAttackMotionParam's 125 changed values (fields +0x8 / +0xc per attack, mostly raised) are 76
  retail, 18 back to US, 31 neither; ItemTypeParam +0xc (1.0, row 400 3.0 -> 0.5 for all 11 item types) and
  WeaponTypeParam row 480 are retail; ChrEquipLoadParam 5 of 9 changed values retail.

## 5. Not done yet

- Code comparison in Ghidra (`Ghidra Project\DS2JPNT`, 01.04): e.g. whether the chest trap code (`MapObjTreasureBox
  Component`) and the Agility fuse are unchanged, and what 01.03 / 01.04 fixed (code-only updates).
- The JP 01.00 EBOOT (`versions\01.00\EBOOT.elf`, 66 KB smaller) is not in Ghidra; import it into the same project if
  the 01.00 -> 01.02 code changes matter.
- The in-game menu layouts (`.flo`) are not decoded, so the status screen layout was not compared.
