# Unused models in retail (2026-10-22)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

How many of retail v1.00's model files does nothing use? About **1,305 of 5,079 (26%)**, nearly all of them map pieces
and objects.

| Kind | Models | Unused | How "unused" was decided |
|---|---|---|---|
| Map pieces | 2,863 | **874** | no MSB part uses the model (749 not even listed in a map's model table); see [unused map pieces](unused-map-pieces.md) |
| Objects | 1,261 | **355** | no MSB part places it and no other file names it; 95 more are never placed but can still appear (below) |
| Equipment | 789 | **72** | no `WeaponParam` / `ArmorParam` row names the model |
| Characters | 166 | **4** | no map's `GeneratorRegistParam` spawns it and no other file names it |

Counts are distinct models; equipment ships male / female / left-hand files per model (2,290 files for the 789).

## Characters (4)

Spawns: `GeneratorRegistParam` enemy id // 100 = model (123002 -> c1230), over all 24 maps. 160 of the 166 models
spawn. Of the six that don't, two are in use anyway: c0001 is the player, and c2250 (no triangles, 20 bones) is named
in the executable, so it's probably an invisible helper. The four unused ones:

| Model | What | Other files that name it |
|---|---|---|
| c1400 | 5.2 m winged woman | its own sound bank `frpg2_c1400` only |
| c2020 | a white variant of c2010 (0.8 m) | sound banks only |
| c7015 | a human body (`BD_c7010_F` textures); **the child Emerald Herald**, `EnemyParam` 701500, 1.21 m ([child Emerald Herald (c7015)](../network-test/child-herald-c7015.md), 2026-10-26) | the c2260 animation file and the c7010 sound bank |
| c7210 | [Llewellyn](../network-test/llewellyn-references.md), the cut "royal strategist" NPC ([armour, rings and spells: Network Test vs retail](../network-test/armour-rings-spells-nt-vs-retail.md)) | nothing |

**Correction (2026-10-24, PC Scholar stage 1, [cut content in PC Scholar](../network-test/cut-content-pc.md)):** enemy id // 100 is not
always the model; EnemyParam's `chr_id` is. Counted that way (and through GeneratorParam -> GeneratorRegistParam, so
only enemies a generator places), **c1400 and c7210 are placed**: c1400 = enemy 776000-776002 (AI ai140000
"ベルカの鴉人", Velka's crow-person), 4 generators in Brightstone Cove (m10_14) and the Shrine of Amana (m10_32);
c7210 = enemy 721100 (AI ai721100 "下位：軍師の霊", lower: the strategist's spirit), 1 generator in Drangleic Castle
(m20_21). c0001 (human body, 97 generators of NPCs and phantoms) and c2250 (5) are placed too. **The models no
enemy uses are c2020, c6031 and c7015** (c6031 was missed: enemy 603100 uses c6030). Whether the c1400 / c7210
generators ever fire in play (event-gated?) is not checked.

The many-legged models are all spawned enemies; their AI scripts name them (`ezstate\aiNNNNNN.esd` top labels
`aiNNNNNN_<name>`, e.g. ai205000_毒角虫 "poison horn beetle"). No model in any build is the network test's crab boss
([unused map pieces](unused-map-pieces.md), crab tower).

## Objects (355, plus 95 placed some other way)

811 of 1,261 object models are placed by MSB parts (type 1, 6,914 parts). For the other 450, every file was searched
for their names (`oNN_NNNN`, ASCII and UTF-16):
- 293 are named nowhere else, and 62 more only in leftovers: an MSB model table entry with no part, the level editor's
  `prefabeditor` files, or map lighting data. **355 unused.**
- 95 are named by something that can spawn them: animation events (`timeact`), another object (break pieces), a
  character model, a cutscene (`model\scn`) or the executable.

`MapObjectParam` is no help: it has rows for 1,248 objects, including 437 of the unplaced ones. Not checked: an object
spawned by its number alone (say from a bullet row); number matching across params gave only chance hits. So 355 is an
upper bound.

By map prefix, the unused objects are mostly common objects `o00` (152) and Forest of Fallen Giants / Memories `o10`
(49), then `o14` Brightstone Cove, `o03` Dark Chasm, `o21` Drangleic Castle, `o15` Aldia's Keep, `o17` Harvest Valley,
`o24` Undead Crypt (14-23 each). The prefix is the map whose MSBs place that series (`o04` Majula, `o10` m10_10 and
m20_10, `o03` m40_03, `o11` m20_11, the rest their own m10 / m20 number).

### What the 355 are (contact sheets, 2026-10-22)

All 355 are drawn in `unused-models\objects_01.png` to `objects_12.png` (30 a sheet, one oblique view, labelled with
name, size, triangles and first texture); details per object in `unused-models\objects.tsv`. Most are ordinary set
dressing: rope and ladder kits in every length (4.6-30 m), doors, gates, furniture, pots, barrels, candlesticks,
flags, rocks, 62 m light shafts for the Undead Crypt, Harvest Valley gearing and windmill sails, and 15 empty models.
The rest:

**Labelled stand-ins (22).** 3D text in the developer-sign style, read face-on with `labels.py` and
`textviews2.py` (renders in `unused-models\labels_*.png`; ground labels read from below):

| Object | Label | Literal |
|---|---|---|
| o00_0317 | Aさん宅の鍵扉 | locked door of A-san's house ("A-san" = a placeholder name) |
| o00_0318 | 王の指輪の扉 | King's Ring door |
| o00_0321, 0322, 0324, 0325, 0326 | 壊れない (both faces) | won't break (doors) |
| o00_1011 | 虫鍵の灯火台 | Pharros-lockstone lamp stand (虫鍵 = the Lockstone's working name, `developer-signs.md`) |
| o00_1960 | 点火できる小型燈火台 | small lamp stand that can be lit |
| o00_3002 | だらだら壺 | dribbling pot |
| o00_3007 | 油壺 | oil jar |
| o00_3010 | 飛竜の卵 | wyvern egg (an egg-shaped model) |
| o00_3020 | 壊れる床 | breakable floor |
| o00_9115 | レバー起動跳ね橋 | lever-operated drawbridge |
| o00_9116 | 仮ハンドル | temporary handle (a wheel) |
| o00_9117 | 鐘守りの鐘 | the Bell Keepers' bell |
| o00_9118 | 墓からの敵起こす鐘 | bell that raises the enemies from the graves |
| o00_9300 | 壊れた王の器 | broken king's vessel (a root mass) |
| o00_9500 | 船 | ship (a boat with a cross mast; a triangle marks the bow) |
| o10_1011 | 連射型テスト | rapid-fire type test (a ballista) |
| o10_1012 | 火矢テスト | fire-arrow test (a ballista) |
| o32_0470 | 王の扉 | King's door |

**Review markers (11).** 1.5 m cubes, the label on all four sides, plus one slab: o00_9800 暗いエリア "dark area",
9803 要検討 "needs discussion", 9804 要確認 "needs checking", 9805 調整中 "being adjusted", 9806 ここギミック
"mechanism here", 9807 ここ明るい "bright here", 9808 ここ広く "make this wider", 9809 ここ狭く "make this
narrower", 9810 ここ天井調整 "adjust the ceiling here", 9811 間が長い "this stretch is too long", 9812
イベント発生用仮モデル "temporary model for triggering events"; 9801 is an unlabelled 2.4 m slab.

**Characters retail doesn't have** (`unused-models\closeups_*.png`; network test rows via `cnumtrace.py`):

| Object | What | Textures | Other traces |
|---|---|---|---|
| o10_9200 | 10 m hulking armoured giant, as stone | `BossStone`, `c6090` | NT: a boss (`BossParam` 609000 / 609010, `EnemyParam`, behaviour rows); retail: two stray attack / logic rows. The NT soul list has "Kanossof the rock giant", but nothing links them |
| o00_9111 | 42 m emaciated giant, face turned up | `BD_c6245`, `SH_c6245` | NT `EnemyParam` + behaviour rows for c6245 |
| o10_9230 | 40 m rocky mass with a toothed mouth between two humps | `c6240_01-03` | NT `EnemyParam` + behaviour rows for c6240 |
| o21_9200 | 1.7 m woman in a long dress with hanging sleeves, braided hair; full face / eye / hair materials like an NPC | `c7030_F` | NT `EnemyParam`; retail still ships `sound\frpg2_c7030.bnd` |
| o23_9050 | the Executioner's Chariot as a static model, with two horses | `c6180` | retail's chariot is c6191 |
| o23_0000 | 3.7 m mushroom-capped pillar | `c4140` | none |

No item lot in the network test holds any of [its 26 boss souls](../network-test/nt-bosses-not-in-retail.md#2-the-27-nt-boss-souls-itemname-60730000-60755000-and-50140000), so none of these can be tied to a soul.

Named 2026-10-28 by the PC pre-release's row names ([community cut-content sheet](../network-test/community-cut-sheet-crossref.md)). `o10_9230` = MapObjectParam
109230 【抵抗の砦】巨人王 "Giant King" (enemy rows 624000 巨人王 ※没 "cut" / 624500 巨人王). `o10_9200` fits 609010
巨人族親衛隊長（石化後） "after petrification". `o21_9200` = 219200 【王城】王妃NPC仮モデル "queen NPC temporary model".
`o23_9050` = 239050 【拷問場】シルバーチャリオット（仮）.

**Other figures.** o00_0242-0255 (13) and o10_9190: unplaced ragdoll corpses (skinned, 25-67 bones, enemy face and
armour textures; their placed siblings o00_0240 / 0247 / 0256-0258 appear up to 371 times), so they show in their
bind T-pose. o00_4904: a 13 m giant crawling on all fours, with a tail and a face (placeholder texture only).
o00_4907: a static copy of the Covetous Demon (same shape as c5000); o00_4905 / 4908: horns textured c6020 / c5000.
o10_6010: a posed knight with sword and shield (NPC c7240's textures). o00_9112: a robed figure statue on a pedestal
with a sword. o24_2000: a dancing-figure statue. Also a rat, a bird, two fish, moths and an apple.

**Scenery.** o00_9110: a 145 m castle tower with walls (materials 兵舎の壁 "barracks wall", 城外壁A "castle outer wall
A"), a backdrop. o10_1100: a 41 m oared galley with sail and flag, textured `kakiwari` (a painted stage backdrop).
o00_1020: an alternative bonfire, the coiled sword in a heap of bones with decals. o00_1000: a 12 m blockout tree.

**[Firelink Throne](../network-test/network-test-music.md).** Retail's `MapName` 10280000 = "Firelink Throne" (火継ぎの玉座), but there are [no m10_28 map files](../builds/v1.00-only-content.md)
at all; only four o28 objects survive (o28_9000, an animated lamp stand, plus three unplaced coal-tar pots and a lamp,
o28_9001-9003).

Texture names with short prefixes (m12_ to m19_) are texture-library names used across many object series, not map
numbers: m15_ textures sit on placed objects of four series. Only the m18 set (`m18_Plinth_of_fire`, `m18_N_tree`) is used by
unplaced objects alone: 10 of them, 8 among the 355, including the ship and the root mass.

## Equipment (72)

Weapon model = `WeaponParam` +0x04 (shields when `WeaponTypeParam` +0x36 is 26-28, `SD_` files; else `WP_`); armour
model = `ArmorParam` +0x0a u16, part +0x05 (1 head `HD_`, 2 body `BD_`, 3 arms `AM_`, 4 legs `LG_`).

| Kind | Models | Unused | Which |
|---|---|---|---|
| Weapons | 249 | 7 | 2510 ([the tongs](nt-only-equipment-models.md), [weapons: Network Test vs retail](../network-test/weapons-nt-vs-retail.md)), 7000, 9900-9904 |
| Shields | 79 | 6 | 1095 (a cut wooden shield, Melinda the Butcher's: retail's `NPCGenerateParam` still equips 11095000, which has no row; [weapons: Network Test vs retail](../network-test/weapons-nt-vs-retail.md)), 9900-9904 |
| Head | 119 | 15 | 2481, 3190, [7521](../network-test/cut-content-scholar.md), 99xx (12) |
| Body | 125 | 18 | 2481, 3190, 9040, 99xx (15) |
| Arms | 107 | 12 | 2481, 3190, 99xx (10) |
| Legs | 110 | 14 | 2481, 3190, 99xx (12) |

The 99xx series (9900-9918, 9950) makes up 59 of the 72; going by the numbers they're probably test models, but none
was rendered. 2481 and 3190 are [whole unused armour sets](unused-armour-icons.md). The 14 accessory models (`AS_`) aren't counted: no param
was found that names them.

## Tools

`DS2 Vanilla\tools\unused_models\` (scratch-grade, run with `uv run --no-project --python 3.12 --with numpy --with
pillow --with scipy python -I -X utf8`; they write their pickles next to themselves or in the current folder):
`chrsurvey.py <game> <out.pkl>` (every character FLVER: size, triangles, bone names, materials), `spawned.py <game>`
(models spawned per map), `chrsheet.py <game> <out.png> <model>...` (contact sheet; objects too), `chrrefscan.py <game>
<cNNNN>...` (files naming a character), `objuse.py <game>` (placed vs unplaced objects), `objrefscan.py <game>` (every
file's object names), `objlegs.py <game>` (objects with leg / claw bones), `partsuse.py` (equipment), `objsheets.py <game> <names.txt>
<out dir>` (object contact sheets + tsv), `objflv.py` (extract object FLVERs), `textviews2.py <out.png> <flv>...`
(lettering from four sides; `MAXE=<m>` keeps only pieces smaller than that), `closeups.py`, `montage.py`,
`flipsheet.py`, `cnumtrace.py <game> <NNNN>...` (files and param rows for a character number), `bossparam.py`,
`soullots.py`, `texprefix.py`, `objprefix.py`.
