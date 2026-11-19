# Cut content in PS3 Scholar: what survives from the Network Test and vanilla (2026-10-23)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author): which cut maps, items and other leftovers (from the Network Test and from vanilla v1.00) survive in
the PS3 Scholar disc?

Tools (this folder's `tools`, `uv run --no-project --python 3.12 python -I -X utf8 tools\<script>`):
`cut_survival.py` (every model on vanilla's unused lists: still on the disc? byte-identical? now used? plus models
unused only in Scholar; `cut_survival.tsv`, `cut_new_unused.tsv`), `cut_items.py` (cut item ids across NT / vanilla /
Scholar), `cut_text.py` (named items without rows, deleted text files, map numbers, Agdayne leftovers). Vanilla tools
reused with `DS2_GAME=scholar`: `devsigns\hidden.py`, `unused_survey.py` + `unused_render.py`, `labels.py`,
`signs2.py`, `unused_models\objsheets.py`. Pictures and dumps: `cut-content\`.

## Short answer

**Almost everything survives, byte for byte.** Scholar keeps every unused model vanilla had (874 map pieces incl.
the early whole Bastille, the crab tower, the earlier Dark Chasm, the Iron Keep district and all 12 developer-sign
models; 355 objects; 4 characters; the unused equipment), every hidden map part (the Wharf ship, blockouts, the three
Copse PvP arenas) and the cut-item text leftovers. **What Scholar removed** is text: vanilla's stale talk files for map
numbers that have no map, which held **two pieces of cut dialogue from the Network Test era** (section 4), plus the
demo spell ids in the debug loadouts and the unobtainable "Barrel" armour row. **One cut slot came back**: the Network
Test's [spare Lucatiel's Mask row](armour-rings-spells-nt-vs-retail.md#armour) and its unused model `HD_7521` are now a real item, the Mirrah Hat. **New leftovers**:
the DLC maps have their own unused pieces, a 3D developer sign in Eleum Loyce, and 73 never-placed DLC objects, many of
them "仮" (provisional) stand-ins.

## 1. Models

| vanilla's unused list | in Scholar | still unused? |
|---|---|---|
| 874 map pieces ([unused map pieces](../unused/unused-map-pieces.md)) | **all 874 byte-identical** | all unused |
| of them, the 12 developer-sign models (m10_16 / 17 / 18 / 34, m20_11 / 21 / 24 `m8000`, m10_19 `m0900`, m10_23 / 25 `m0800`, m10_31 `m6000`, m40_03 `m1600`) | identical | unused (never placed) |
| early whole Lost Bastille `m10_16 m0000` with the NPC labels and the crab tower; earlier Dark Chasm `m40_03 m0000`; Iron Keep's 26-piece cut district | identical | unused |
| 355 objects (`unused-models\objects.tsv`) | 321 identical, **34 rebuilt** (o19 Iron Keep, o24 Undead Crypt, o25 Gutter, o32 Shaded Woods: DlcData copies) | all unused |
| characters c1400 (5.2 m winged woman), c2020, c7015, c7210 (Llewellyn) | identical | not spawned (Scholar's 39 new characters all spawn). **Corrected 2026-10-24**: by EnemyParam `chr_id`, c1400 (4 generators, m10_14 / m10_32) and c7210 (1, m20_21) are placed; the unused models are c2020, c6031, c7015 ([unused models](../unused/unused-models.md)) |
| weapons [WP_2510 (tongs), 7000](../unused/nt-only-equipment-models.md), 9900-9904; shields SD_1095 (Melinda's wooden shield), 9900-9904 | identical | unused |
| armour models [2481, 3190](../unused/unused-armour-icons.md), 9040, 99xx | present | unused |
| **HD_7521** (the cut second Lucatiel's Mask's model) | present | **now used**: Scholar's ArmorParam 17521100 = item 27521100 **Mirrah Hat** (ミラの帽子, "a hat from the land of Mirrah ... used in a ceremonial capacity") |
| BD_1450 (vanilla's ArmorParam 11450101, the "Barrel" body armour with a name but no item row) | present | **now unused**: Scholar deleted the armour row; the name "Barrel" (21450101) is still in ItemName |

Notes: the `_s` companion pieces (vanilla 84, Scholar 91) are never placed in either build and are left out, as in
the vanilla survey. Scholar's 95 other never-placed vanilla objects are the ones vanilla's note already lists as
spawned some other way.

## 2. Maps and map parts

- **[Hidden parts](../unused/hidden-map-parts.md)**: `hidden.py` gives the same list on every base map in both builds (Wharf three-master `m0078`, the
  group-127 blockouts, the 9.3 km sky dome, Harvest Valley's early terrain ...). DLC maps hide nothing, except one
  Brume Tower object with no draw group (`o36_9045_0037`).
- **Copse PvP arenas** (m10_23 groups 70 / 71 / 72): identical, 114 / 242 / 259 parts.
- **What Scholar added to the base maps** (part diff, nothing removed): the three DLC doorways (Gutter / Black Gulch:
  `o00_1080` + four `o35_9058`; Iron Keep: map piece `m0503` + `o00_1080` + four `o36_9058`; Shaded Woods: `o37_9000`
  + four `o37_9001`), objects `o00_9900`-`9903` on six of [Aldia's seven maps](../builds/1.08-to-1.10.md#3-109-110-data-built-26-nov-2014-with-scholar-two-layers) (presumably his appearances), two objects
  in Drangleic Castle, his seventh map (`o21_9409` / `9410`), and small moves (Undead Crypt bonfire, a Gutter object, four Dragon Aerie pieces nudged
  1-5 m). Many other parts changed bytes without moving.
- **Map numbers without a map**: vanilla's archives named m10_21, m20_04, m90_00 (talk files, 11 languages) and m99_97
  (Japanese talk); **Scholar has none of them**. On the disc, the map sound banks numbered 1021 and 1028 (m10_28 = the
  "Firelink Throne" that only `MapName` names) survive in both retail discs. Of vanilla's 91 disc sound files, 88 are
  in Scholar at the same size (sizes compared, not contents); the 3 `*_stationary.fev` event files differ. Scholar's new banks are the DLC's (5035-5038) and seven more, `sm6014 /
  6016 / 6019 / 6025 / 6027 / 7021 / 7024`: one per Aldia map (m10_14, 16, 19, 25, 27, m20_21, 24, shifted by 50),
  presumably his scenes' audio (not checked).

## 3. Items

| leftover | NT | vanilla | Scholar |
|---|---|---|---|
| [Shadow Claws 3510000](weapons-nt-vs-retail.md) (cut weapon) | row, no text | text only (name + description) | text only, unchanged |
| other NT-only weapons 3511000, 2450000, 2510000, 11095000, 7000000 | rows | none | none |
| NT-only armour 12231100, 17830102 / 103 | rows | none | none |
| NT armour 17521100 (second Lucatiel's Mask) | row | EquipInterfereParam row + model only | **revived as the Mirrah Hat** |
| shield 11095000 equipped by NPCGenerateParam | 813000 | 814000, 814009, 833000 (no weapon row) | the same three, still dangling |
| NT rings 40170000-40200000 (starstone), 40240000, 40250000, 40270000, 40380000, 40490000 | rows (仮・ names) | none | none |
| Ring of Thorns name | NT ring 40250000 | reused for 40530000 | same |
| 29 grey per-class placeholder weapons 6300000-6550000, 12000000-12020000 | 29 rows | 29 rows | **29 rows**, still unnamed |
| [NT boss souls 60730000-60755000](nt-bosses-not-in-retail.md#2-the-27-nt-boss-souls-itemname-60730000-60755000-and-50140000) (26, e.g. 宿り大蟹 crab, 王妃デュナシャンドラ) | names | none | none |
| [ten demo spell ids in the debug loadouts](armour-rings-spells-nt-vs-retail.md#spells) (DbgPlayerStatusParam) | - | 31111000, 31122000 ... (10) | **removed** (276 rows, no demo ids) |
| named items with no ItemParam row | - | 96 (Hunter's Hat, Singer's Dress x2, Rosabeth's Dress, Illusory Rings of the Vengeful / Guilty, Weapon / Armor Smithbox, a second Estus Flask, Yellow Sea Troches, Pungent Ooze, Rhoy's Stone (+ of Knowledge), Gold / Fake Dead / Illusory Talisman, Wooden / Magic / Dark Greatarrow, "Proper bow" ...) | **the same 96**, none revived, plus 9 `no text` placeholders |

## 4. Text Scholar deleted: two cut dialogues from the Network Test era

Vanilla v1.00 ships talk files for three map numbers that have no map. They are byte-identical to the Japanese network
test's, Japanese text only (the English files are empty), and **PS3 Scholar removed all three** (and the m99_97 file).
(Dated 2026-10-23: the removal is in **patch 1.03**, data built 1 Apr 2014; 1.01 / 1.02 still ship them. See
[cut talk timeline](../builds/cut-talk-timeline.md).)
Full text: `cut-content\cut_talk_m20_04_m10_21_m90_00.txt`; **line-by-line translation of all 128 lines:
`cut-content\cut-dialogue-translation.md`** (with retail's official English beside the castle lines it kept).
Translations below are mine.

**m10_21: the Emerald Herald's castle dialogue, network test version.** m10_21 is the castle's old map number
(texture prefixes, `sm1021` bank). All 29 lines equal the network test's own `m20_21` talk word for word. Retail's
`m20_21` revised three of them and dropped nine:
- 70004410-70004460: "My name is Shanalotte (シャナロット). A name that dragon gave me, who was born without one. I am a
  dragon's child, made by people. Those who tried to go beyond the causality once ordained made me. But their scheme
  failed. I was a failure." (kept in retail)
- **70004480 / 70004510: "If you go on, Shandra (シャンドラ) will set upon you." / "Please calm Shandra."** Retail:
  デュナシャンドラ (Dunashandra, English Nashandra). The test's boss soul list already says 王妃デュナシャンドラ, so the
  rename happened during the test period.
- 70004490: "to take that king's seat and become **the one who inherits the fire** (火を継ぐ者)". Retail drops "the fire":
  "become the one who inherits" (継ぐ者).
- 70009000-70009600 (dropped in retail): "Put an end to your... to my journey", "What...?", "If that is your will...",
  and placeholder dashes.

**m20_04: a cut questline: a little girl and the man who keeps her.** 98 lines, two speakers (talk ids 70100xxx and
70400xxx), in a "village" that has no map in any build:
- **[The girl](child-herald-c7015.md)** (70100xxx): "I'm so tired... I walked and walked... I don't want to walk any more", "Ah, big brother!? /
  big sister!?" (both forms, for the player's sex). She has travelled with many people: "when the person I travel with
  disappears, a new one always comes and I travel again", and is puzzled that you don't have to travel with her
  "because that man from before is gone". She loves pretty things and asks for **"Estintia's feather"**
  (エスティンティアの羽: "whoever picks it up, a time of happiness descends on them"); the Japanese test's ItemName has
  it as item **50220000**. Given it, she promises to remember you "for ever and ever". She decides to stay "in this
  village for good". Rescued, she gives you a secret **incantation**: "Wandering souls gathered in the hearts of
  people... root souls riding the flow of the body... with the ancient, undying power dwelling in my blood as source,
  lend strength to the one who saved me." Being dragged: "Don't pull me!", "I'm not a tool!", "Stop!".
- **[The man](sparkling-girl-dyna-tillo.md#is-she-related-to-the-cut-girl-and-dealer-m20_04)** (70400xxx): a haggling dealer who calls her his **商売道具, "the tools of my trade"**: "I'm handing her
  over to you specially, you understand... (heh heh)", a run of haggling lines ("I'll knock a little off, but I have
  a living to make"), "depending on your 'sincerity', I might set her free", and fight lines ("What are you doing to
  my merchandise!", "This time I'll kill you!").
- Retail's 3D labels in the early whole Bastille ([unused map pieces](../unused/unused-map-pieces.md): 妹を探す兄 "brother looking for his younger
  sister", 少女 "girl") may belong to the same idea; nothing links them directly.

**m90_00**: one line, カットシーンテスト文章です ("this is cutscene test text"). Also removed: `bofireName.fmg` (one entry,
"Test" / テスト) and `regulation.fmg` (one entry, "The letters are worn beyond recognition.").

**Agdayne / Darklurker leftovers** ([Agdayne and the Darklurker](agdayne-darklurker.md)): still there in Scholar: the AI labels "Agdayne's Stand"
(ai506100 / 506110), the wormholes (ai506200 / 506210), Grandahl "pilgrim of the abyss" (ai725000 / 725010), and
`event_m40_03` still spells アガトゥラン 4 times beside アガドゥラン 24 times. The scripts were rebuilt but keep these names.

## 5. New in Scholar: the DLC's own leftovers

- **Unused map pieces** (renders in `cut-content\dlc-unused-pieces\`): Brume Tower `m0054` (an 8 km lava plane),
  `m0920` (a 36 m round tower top with rings, 82 % over a placed piece: an earlier revision), `m0990` / `m0991`
  (ruined hollow towers, 139 m and 162 m tall, 28 m across, 160-180 m north-west of the map; put into the game by
  the presets in `showcase-brume-tower.md`, 2026-10-23, not yet applied); Eleum Loyce `m0007` (a 100 m
  blockout box with a `test_11_d` texture), `m0501` (a 2-triangle ice plane), **`m9000` = a developer sign**; Memory of
  the King `m0000` (an empty model). Shulva has none.
- **Eleum Loyce developer sign** (`m9000`, 4 signs, `cut-content\dlc-sign\signs\`; translations mine):
  ここに敵がいると、シルエットになって見栄えが良い "If an enemy stands here, it makes a silhouette and looks good";
  ここが開通すると、ショートカットできる "Once this opens up, you can take a shortcut"; 宝死体 "treasure corpse" (x2). The DLC
  team marked up its maps the same way as the base game's.
- **73 never-placed new objects** (sheets in `cut-content\new-unused-objects\`; unlike vanilla's list, not yet searched
  for other spawners, so an upper bound): many **仮 ("provisional") stand-ins**, among them five copies of the DLC-doorway
  gadget (`o19_1080`, `o35_1080` / `1081`, `o36_1060`, `o38_1060`; the placed one is `o00_1080`) and 仮-lettered
  pillars / signs (`o35_0720`, `o35_0730`, `o36_0600`, `o36_0610`, `o37_0600`); a text marker box (`o36_1160`, "アイ...
  用すると ... 止", partly legible); test-textured blockouts (`o35_1200`-`1204`, `o37_1060`); and real props: a 20 m
  winged dragon-like figure (`o35_8000`, Shulva), a crouching figure in ice (`o37_1070`, Eleum Loyce), a skeleton
  (`o35_1090`), Brume Tower cages and chains (`o36_9001`-`9050`), three weapons (`o36_2400`-`2402`), a shield with a
  `c5040` texture (`o37_9050`). Three are empty models.
- **Text**: Eleum Loyce has two `（No Text）` area names (50370002 / 50370003) between Grand Cathedral and Frigid
  Outskirts; 9 new `no text` item names.

## 6. Not checked

- Whether the 73 new objects are spawned by events or other files (vanilla's objects needed that search).
- Item placements of NG+ (the "+1" lot variants are compared in [item placements: vanilla vs PS3 Scholar](../builds/item-placements-vanilla-vs-scholar.md)).
- Sound bank contents (`sm60xx` / `sm70xx` assumed to be Aldia's).
- (Checked since: the Western network test's three talk files are byte-identical to the JP test's and vanilla's, so
  none of the cut lines is test-only; only the Estintia's feather item name 50220000 is. `cut-dialogue-translation.md`.)

## Related pages

- [Cut content in PC Scholar](cut-content-pc.md): the same survey carried on to PC Scholar, which was packed from references: most never-placed objects, the three unused character models and 51 unreferenced AI scripts are gone.
