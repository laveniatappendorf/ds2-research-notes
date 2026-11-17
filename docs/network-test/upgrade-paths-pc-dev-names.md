# Upgrade paths, the cut materials and Break: what the PC builds add (2026-11-09)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author asked whether the PC builds (vanilla PC and PC Scholar) tell us more about the upgrade paths, the two cut
upgrade materials and the dissolving stone than the PS3 releases did. Background: `unused-material-icons.md`,
`cut-infusion-crystals.md`, `break-element.md`.

Checked: the six vanilla PC builds (31 March 2014 pre-release, 1.01, 1.02, the 2 June 2014 regulation update, 1.03,
1.12) and PC Scholar. Tools and raw output: `..\..\DS2 Vanilla PC\tools\dev_names\` (`devnames.py` dumps every
developer row name, `devnames-2014-03-31.txt`; `wrp2.py` sorts the weapons by upgrade path; `brk.py` / `brk2.py` read
the Break rows; `allicons.py` hunts hidden icons, `hidden-icons.json`).

## Short answer

- **Nothing new about the cut items themselves.** None of the PC builds has a row, name, text, item lot or extra
  icon for 古代の銀鋼石 (61010000), 龍の骨片 (61020000), the two crystals (61040000 / 61050000) or the dissolving
  stone (61120000). Their icons were already in PS3 v1.00. The only PC text near them is Yellow Sea Troches, which PS3
  has too.
- **What PC adds is the developers' row names.** The pre-release PC params (31 March 2014 to 1.02) keep the
  developers' Japanese row names; PS3 and later PC builds have none. Those names explain three things the PS3 data
  holds without labels:
  1. the three upgrade paths and their separate infusion-cost tables;
  2. the working names of Twinkling Titanite, Petrified Dragon Bone and every infusion stone;
  3. two Break leftovers that survive, unnamed, in every build.

## 1. The three upgrade paths

Every weapon points at an upgrade-cost row (ReinforceCostParam), an infusion permission row (CustomAttrSpecParam) and
an infusion cost row (CustomAttrCostParam). The developer names sort them into three paths:

| path (developer name) | upgrade material | weapons | how the cost rows are named |
|---|---|---|---|
| 通常 "normal" | Titanite Shard -> Large Shard -> Chunk -> Slab | 194 | by tier and weapon class: 最下位 / 下位 / 中位 / 上位 / 最上位 (lowest to highest), × 49 weapon, shield and armour classes |
| 光石 "light stone" | Twinkling Titanite (光る楔石, "shining titanite") ×1-5 | 42 | one row per weapon, named after its first owner, mostly NPCs and enemies: 王国軍師 (royal strategist), デュラハン, ファントムメイジ, スペル四天王 ("the four spell kings") |
| ドラゴンウェポン "dragon weapon" | Petrified Dragon Bone ×1-5 | 43, plus 6 that can't be infused | one row per weapon, mostly boss weapons; their item names carry a "DS：" tag (DS：蜘蛛の女王, DS：熔鉄デーモン, DS：マンスコーピオン) |

- **Infusion was planned per path.** CustomAttrCostParam has one row per path: 100000 属性強化-通常, 100100
  属性強化-ドラゴンウェポン, 100200 属性強化-光石 (plus 0 "no change" and 900000 "test"). CustomAttrSpecParam likewise
  has 通常 / ドラゴンウェポン / 光石 × each weapon class and armour, plus 全派生許可 "all infusions allowed" and
  通常強化のみ "normal upgrades only".
- **But the three cost rows are identical** in every build, from the pre-release onwards: one stone each, 1,000 souls
  for the Palestone and 2,000 for the rest. By 31 March 2014 the separate costs had been flattened.
- PS3 v1.00 has the same structure, unnamed (194 / 43 / 42 weapons); PC 1.12 and PC Scholar too, with a few weapons
  moved by later patches.

**What this means for the cut materials** (inference, not data). Each special path was its own track with its own
infusion cost. The Network Test's pairs fit the two special tracks: 純粋な黒鋼石 "pure black titanite" (Twinkling
Titanite's working name, below) with 古代の銀鋼石 "ancient silver titanite" on the 光石 path, and 龍の骨片 "dragon bone
fragment" with the 龍の化石 "dragon fossil" on the dragon path. "DS" is most likely "demon soul": the boss-soul
weapons, the Network Test's "demon equipment" (デーモン装備) that both dragon bones reinforce. By the pre-release each
path had one material, ×1-5. The per-path infusion costs would have been the natural place to charge a catalyst (the
five-coloured or rainbow crystal), but no row says so.

## 2. Working names

ItemParam's developer names read "retail name（working name）":

| id | Network Test | PC working name | retail Japanese | retail English |
|---|---|---|---|---|
| 61000000 | 純粋な黒鋼石 | 純粋な黒鋼石 "pure black titanite" | 光る楔石 | Twinkling Titanite |
| 61030000 | 龍の化石 | 古龍の化石 "ancient dragon fossil" | 竜の骨の化石 | Petrified Dragon Bone |
| 61060000 | 星光石 | 星光石 "starlight stone" | 蒼光石 | Faintstone |
| 61070000 | 月光石 | 月光石 "moonlight stone" | 雷光石 | Boltstone |
| 61080000 | 火竜石 | 火竜石 | 火竜石 | Firedrake Stone |
| 61090000 | 暗月石 | 暗月石 "dark moon stone" | 暗夜石 | Darknight Stone |
| 61100000 | 毒塊石 | 毒塊石 | 毒塊石 | Poison Stone |
| 61110000 | 血塊石 | 血塊石 | 血塊石 | Bleed Stone |
| 61120000 | 溶解石 | (no row) | - | - |
| 61130000 | - | 粗製石 "crude stone" | 粗剛石 | Raw Stone |
| 61140000 | - | 魔力石 | 魔力石 | Magic Stone |
| 61150000 | - | 深淵石 "abyss stone" | 無明の古石 | Old Mundane Stone |
| 61160000 | - | ノーマル戻す石 "stone that returns to normal" | 白石 | Palestone |

- The seven Network Test stones kept their Network Test names as working names. The four stones retail added
  (Raw, Magic, Mundane, Palestone) have working names of their own. Old Mundane Stone was the "abyss stone".
- Twinkling Titanite's working name is the Network Test's "pure black titanite". That confirms the black titanite /
  Twinkling link, which the Network Test showed only by position. The Petrified Dragon Bone was briefly the "ancient
  dragon fossil", a name in neither the Network Test nor retail.
- The infusion order in CustomAttrCostParam matches the stones: 1 Palestone, 2 Firedrake, 3 Faintstone, 4 Boltstone,
  5 Darknight, 6 Poison, 7 Bleed, 8 Raw, 9 Magic, 10 Old Mundane.

## 3. Break leftovers the names identify

Both sit in the Network Test and every retail build, but only the PC names say what they are:

- **A Break weapon buff glow.** ChrPhantomParam lists the weapon-buff glows in the Network Test's nine-type order:
  18 physical, 19 magic, 20 lightning, 21 fire, 22 dark, 23 poison, 24 bleed, **25 武器エンチャント_武器破壊強化
  "weapon enchant: weapon break"**, 26 curse. The shield glows repeat it at 50-58, with **57 "shield break"**. Rows 25
  and 57 copy the curse rows' blue (64 / 128 / 160), so they look like placeholders. The Network Test has rows 18-26
  (no shield rows). Row 25 fits the Network Test's Break resin 龍の紅液, which "temporarily gives the right-hand weapon
  a break effect".
- **Break as a third status effect.** PlayerDamageParam's status rows: 9010 毒 poison (15 damage per tick), 9020 出血
  bleed (200 at once), **9030 武器破壊 "weapon break"**, pointing at DamageCtrlParam **99987030
  "generic – status ailment – weapon break"**, with 99987040 curse after it. So Break was built as a build-up status
  between bleed and curse, the same order as the Network Test menus (poison, bleed, Break, curse). Retail's damage
  types still have a gap there: 5 poison, 6 bleed, 8 curse. Row 9030 is in the Network Test (older layout, not
  decoded) and in every retail build, where it carries no damage values.
- Also named, but parts of retail's working equipment-damage system: the Destructive Greatarrow (大矢_武器破壊), the
  Corrosive Urn (武器破壊瓶), generic equipment-break damage (PlayerDamageParam 6010 / 7010 -> 99959010), and
  weapon-breaking throws by the Dark Wraith and the Duelist.

## Side find: vanilla PC hides 345 unused icons

The same hidden-file search as for the rings (`unused-ring-icons.md`), widened to every icon number (IC_00 / IC_10 +
00000000-99999999), found **345 icon files in every vanilla PC build that no PS3 build or PC Scholar has, and nothing
uses**. Surveyed the same day: `hidden-pc-icons.md`. In the item range they are **numbered placeholder cards**, not
art: Humanity ×3, 解呪石, 黒松明, 告罪符 and all 26 Network Test boss souls ("Item 139-164"), so the souls still have
no art. None is an upgrade material. (A first count said 22 souls and four shield icons; the four were souls whose
path hash also fits a shield name.)

## Not established

- Whether retail's code ever uses ChrPhantomParam 25 / 57 or PlayerDamageParam 9030.
- The PC executables weren't searched for strings about Break or the cut materials.
- What "DS" stands for: "demon soul" is a reading, not a label.
