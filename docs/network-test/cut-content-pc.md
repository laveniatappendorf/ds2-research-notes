# Cut content in the PC Scholar: what survives from PS3 Scholar (2026-10-24)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author, stage 1 of `..\ASSESSMENT.md`): does the cut and unused content we found on PS3 (Network Test
leftovers, vanilla's unused models, PS3 Scholar's DLC leftovers) survive in the PC Scholar (Steam 1.0.3.0)?

Tools (`..\tools`, run with `uv run --no-project --python 3.12 python -I -X utf8 <script>`; nothing extra needed):
`cut_pc.py` (models: map pieces, objects, equipment, characters, layout; `cut_pc.tsv`, `cut_pc_layout.tsv`,
output `cut_pc.txt`), `cut_pc_text.py` (items, text, AI scripts; `cut_pc_ai.tsv`, output `cut_pc_text.txt`). Shared
readers made PC-aware for this: `..\..\DS2 Vanilla\tools\devsigns\msb2.py` (PC MSBs) and `hidden.py`, and
`..\..\DS2 Scholar\tools\cut_survival.py` (`Build` for `pcsotfs`); PS3 results re-checked identical (section 9).

**Method.** PC and PS3 files differ in byte order, so models are compared by geometry: **same** = same bounding box and
the same mesh, material, vertex-buffer, vertex and face counts; **edited** = same bounding box, other counts (a few
vertices; the PC re-export touched some models); **different** = another bounding box (none found). **Placed** = a part
of some map's MSB uses the model.

## Short answer

**The PC build was packaged from references.** PS3 Scholar shipped everything vanilla had, used or not. The PC keeps a
file only if something points at it, and dropped the rest:
- **objects** only if a map's MSB model list names them (354 of the 428 never-placed objects are gone);
- **characters** only if an enemy uses them (c2020, c6031 and c7015 are gone);
- **AI scripts** only if a param names them (the 51 nothing named are gone, plus 9 others);
- **armour models** only for reachable slot and sex variants (209 files are gone).

**What survives is whatever lives inside a map's own model binder:** every unused map piece, byte-for-byte the same
geometry in 862 of 882 cases. That covers:
- all 13 developer signs;
- the early whole Lost Bastille with its NPC labels and the crab tower;
- the earlier Dark Chasm and the Iron Keep district;
- the DLC's Brume Tower spares.

All 72 unused equipment models, the hidden map parts, the three Copse PvP arenas and every text leftover survive too.

The checks also corrected an older claim: **c1400 and c7210 (Llewellyn's model) are placed by generators** in vanilla
and both Scholars; the truly unused characters are c2020, c6031 and c7015 (section 3).

## 1. Map pieces: all survive

| PS3 unused list | on PC | geometry | placed on PC? |
|---|---|---|---|
| vanilla's 874 ([unused map pieces](../unused/unused-map-pieces.md)) | all 874 | 854 same, 20 edited | none |
| PS3 Scholar's 8 DLC pieces ([cut content in PS3 Scholar](cut-content-scholar.md#5-new-in-scholar-the-dlcs-own-leftovers) section 5) | all 8 | same | none |

- **Developer signs**: all 13 kept (12 base-game sign models + Eleum Loyce `m9000`), 11 same, Lost Bastille `m8000`
  and Copse `m0800` edited.
- **Early whole Lost Bastille `m10_16 m0000`** (NPC labels, crab tower): same geometry, same 17 materials and textures.
- **Earlier Dark Chasm `m40_03 m0000`**: edited. **Brume Tower** `m0054` lava plane, `m0920` tower top, `m0990` /
  `m0991` ruined towers: same. Eleum Loyce `m0007` blockout, Memory of the King `m0000` empty model: same.
- Per map the edits are few (m10_23 Copse 5, m20_11 4, the rest 0-2).

## 2. Objects: most of the never-placed ones are gone

| PS3 unused list | gone on PC | kept (same / edited) | placed on PC |
|---|---|---|---|
| vanilla's 355 unused objects (`unused-models\objects.tsv`) | **324** | 31 (11 / 20) | none |
| PS3 Scholar's 73 never-placed DLC objects | **30** | 43 (28 / 15) | none |
| PS3 Scholar's other 95 never-placed objects (vanilla's "spawned some other way") | **62** | 33 | none |

**The rule is exact**: every kept never-placed object (74 + 33, and the 7 new ones below) is named in some PC map's MSB
model list (an MSB can list a model it never places), and none of the 416 gone ones is. Nine were listed on PS3 and dropped from the PC lists, then from the
archive (`o00_0324`, `o00_2520`, `o15_1040`, `o15_1050`, `o15_9012`, `o32_0470`, `o32_1030`, `o32_7002`, `o32_7006`).

Kept and still unused: `o00_0259`, `0475`, `0901`, `0912`, `1020`, `1063`, `2460`, `2620`, `2991`, `2994`, `4905`, `9016`, `9020`;
`o02_8511`, `o03_9060`, `o10_1012`, `o10_6010`, `o10_9150`, `o14_0371`, `o15_2010`, `o15_9000` / `9002` / `9030`, `o16_1141`,
`o17_3000`, `o19_1160`, `o23_9001` / `9015`, `o25_2062`, `o27_1010`, `o31_9000`; of the DLC ones the doorway gadgets `o19_1080`,
`o35_1080` / `1081`, `o38_1060`, the 仮 pillars `o35_0720` / `0730`, the blockouts `o35_1200`-`1204`, the 20 m winged
figure `o35_8000`, the skeleton `o35_1090`, Brume weapons `o36_2400`-`2402`. Gone among the DLC ones: the 仮 signs
`o36_0600` / `0610`, `o37_0600`, the text marker box `o36_1160`, the ice figure `o37_1070`, the shield `o37_9050`, most
Brume cages and chains. Full list: `cut_pc.tsv`.

**New on PC, never placed** (7 files PS3 doesn't have): `o25_2035`, a 4.6 m kokeshi doll (こけし material, Gutter
prefix); `o99_0004` / `0005` / `0010`, ~3.5 m props built from the c3300 / c3310 / c3330 enemy bodies;
`o99_0036` (7.2 m, [c6710 + kokeshi textures](../builds/early-pc-builds.md#dlc-enemies-in-the-spring-2014-builds)); `o99_0037` (10.5 m, c5050); `o99_0040`, an untextured 3.9 m box
(material "02 - Default"). The other six PC-only objects are placed.

## 3. Characters, and a correction

The PS3 notes found unspawned characters by **enemy id // 100**, which is not always the model: EnemyParam's
`chr_id` is (enemy 603100 uses c6030, not c6031). Counted by `chr_id` through GeneratorParam -> GeneratorRegistParam:

| model | vanilla / PS3 Scholar | PC |
|---|---|---|
| c0001 (human body: NPCs and phantoms) | 97 / 150 generators | kept, 284 generators |
| c1400 (5.2 m winged figure) | **placed**: enemy 776000-776002, AI label `ai140000_ベルカの鴉人` ("Velka's crow-person"), 4 generators in Brightstone Cove (m10_14) and the Shrine of Amana (m10_32) | kept (edited), same 4 generators; the script `ai140000` is not in the PC archive (see section 5 on such gaps) |
| c2250 (no triangles, 20 bones) | 5 / 9 generators | kept, 9 generators |
| c7210 ([Llewellyn's model](llewellyn-references.md#cut-and-leftover)) | **placed**: enemy 721100, AI `ai721100_下位：軍師の霊` ("lower: the strategist's spirit"), 1 generator in Drangleic Castle (m20_21) | kept (edited), same generator |
| **c2020** (white c2010), **c6031**, **[c7015](child-herald-c7015.md)** (human body) | no enemy uses them | **gone** |

So the PC removed exactly the three models no enemy uses; all 203 PC character models are used. Whether the c1400 and
c7210 generators fire in play (event-gated?) is not checked. The vanilla and PS3 notes now carry this correction
([unused models](../unused/unused-models.md), [cut content in PS3 Scholar](cut-content-scholar.md)).

Also gone: 206 per-character camera binders (`.cambnd`) and the texture binders of c0001, c2250, c5062 (the empty
"wormhole" model), c6930 and c7050 / 7051 / 7053 / 7056 / 7058. These are characters dressed in equipment parts or
with an empty model, so the PS3 texture binders were never read.

## 4. Equipment

- **All 72 equipment models on the PS3 list survive**: the 71 unused ones stay unused (WP_2510 tongs, WP_7000,
  WP_9900-9904, SD_1095 (Melinda's wooden shield), SD_9900-9904, armour 2481, 3190, 9040, 99xx), and HD_7521 is still the Mirrah Hat.
- **209 PS3 files gone are unreachable duplicates.** Female variants of armour that isn't gendered (Drakeblood,
  Rampart Golem, Alonne, Loyce, Ivory King …), and arm, body and leg models for head-only items (the three crowns,
  Old Bell Helm, Pharros Mask, Mirrah Hat …: e.g. `AM_1630_*` for the Crown of the Old Iron King), five
  `SD_xxxx_M_A` shield variants, and the unused face model `FG_7010`.
- New on PC: the Forlorn armour (`am/bd/hd/lg_6940`; ItemParam 26940100-103) and weapons (`wp_1997` Greatsword of the
  Forlorn 1997000, `wp_3080` Scythe of the Forlorn 3080000), and male `*_1620_m` variants.

## 5. AI scripts: 51 unreferenced ones are gone

PS3 Scholar ships 528 AI scripts (`ezstate\aiNNNNNN.esd`); **51 are referenced by nothing** (no EnemyParam
`behavior_param_id`, no GeneratorRegistParam logic id). The PC ships 485, **all referenced**, and has none of the 51.
Their top labels read like cut enemies and behaviours (translations mine; full list `cut_pc_ai.tsv`):
- **cut enemies**: [リッチ "Lich" (ai507000 / 507010), レイス "Wraith" (ai508000-508030)](nt-bosses-not-in-retail.md#1-nt-only-boss-slots-what-the-nt-holds-and-what-retail-kept), 生の騎士 / 死の騎士 "Knight of
  Life" / "Knight of Death" (ai335000 / 336000), ストーカー戦士ウイックリー "stalker warrior Wickley" (ai755000),
  奴隷獣（溶鉄壺有り） "slave beast (with molten-iron pot)" (ai201000 / 201010), ネズミ王 "Rat King" (ai226500).
- **characters**: 巫女（子供） "shrine maiden (child)" (ai701000 / 701010), 巫女（死骸） "shrine maiden (corpse)"
  (ai702000), **王妃（死骸） "the queen (corpse)"** (ai702010), [キラキラ娘 "sparkling girl"](sparkling-girl-dyna-tillo.md) (ai786000 / 786010; enemy
  786000 uses the invisible c2250 and a generator places it in Things Betwixt), [ミラーナイトの下僕（狩猟の森）](mirror-names.md#3-the-only-other-in-the-mirror-name-swordsman-in-the-mirror) "the Mirror
  Knight's servant (Huntsman's Copse)" (ai730020).
- **behaviours**: hollow soldiers **playing dead** (死体擬態, ai102053, 103050 / 103051), as a **treasure corpse** (宝死体,
  ai102054 / 102092) or a **treasure-chest trap** (宝箱罠, ai102090); a parasite spider that starts at once (ai107030 /
  203040), a sound-reacting undead citizen, lizard man, wraith and phantom mage; warping skeletons and phantom mages; a
  poison dragon that takes off on the route (ai681020); a heavy dwarf "GDD test" and "GDD high-ground cannon" (ai135090 /
  135091); a lower soldier "waiting to attack (for testing)" (ai103055).

The other 9 PS3-only scripts (`ai102030`, `102040`, `103030`, `103040`, `119000`, `140000`, `203000`, `337000`, `666040`)
are named by EnemyParam or generator rows on PS3, and the PC's params still name them, yet the PC ships no such file.
That is not unusual: a behaviour or logic id with no script of the same number occurs in both builds (PS3 40 + 2 ids,
PC 48 + 10), so the game must fall back somehow (not traced). "Referenced" here therefore means "some param names
the number", not "the game loads it"; what holds without exception is that **the PC ships no script nothing names**.

## 6. Maps and layout

- **[Hidden parts](../unused/hidden-map-parts.md)** (`hidden.py`, now PC-aware): the same lists on every map as PS3 Scholar (Wharf ship, blockouts, sky
  dome …). The PC adds 12 parts with no draw group in the Lost Bastille (six `o00_2445` and six `o16_2001` in a row
  near (-60, -11, 576)); Brume Tower's `o36_9045_0037` now has a group.
- **Copse PvP arenas** (m10_23 groups 70 / 71 / 72): all 114 / 242 / 259 PS3 parts are on PC at the same positions;
  29 new PC objects also carry the three groups.
- **No longer placed on PC** (still in the archive): map pieces `m10_18 m0011`, `m10_31 m1001`, `m20_21 m2553`, and
  object `o00_0802` (a 14 m rope).
- The PC maps have 929 more parts than PS3 Scholar (26,624 vs 25,695): [the reworked placements](../builds/placements-balance-pc.md).

## 7. Items and text

Identical to PS3 Scholar (English; the PC has no Japanese menu text, so the Japanese-only leftovers can't be checked):
- the Shadow Claws name and description without a weapon row;
- the "Barrel" name without its armour row;
- the Mirrah Hat;
- shield 11095000 still dangling in NPCGenerateParam 814000 / 814009 / 833000;
- the 29 grey placeholder weapons (unnamed);
- **the same 96 named items with no ItemParam row**, none revived;
- 9 "no text" names.

BonfireName and MapName are identical. The PC adds 7 ItemParam rows (the Forlorn gear), 2 debug loadouts (278), and two
English files: `dconlymessage.fmg` (38: voice chat …) and `win32onlymessage.fmg` (293: PC controls). It drops 14 empty
bloodmes / talk stubs.

AI labels of [the Agdayne leftovers](agdayne-darklurker.md) are unchanged: Agdayne's Stand `ai506100` / `506110`, the wormholes `ai506200` / `506210`, Grandahl
`ai725000` / `725010`. Editor data (`.edd`): PS3 584 files, PC 53. The effect scripts moved from `EventMakerEx\` into the
regulation (11 of PS3's 13; no `SpEffectEnumeratedCondition` / `SpEffectMain`), which also carries the PC's only
Japanese text (`regulationJapanese.fmg`).

## 8. What this means for showing cut content on PC

- **Map pieces are all there**, so the PS3 showcase idea carries over: after unpacking, an edited MSB placing an unused
  piece is a loose-file change (no same-size limit; the PS3 5.6 MB model limit should not apply). Same candidates: the
  early Bastille / crab tower, the Iron Keep district, the developer signs, Brume Tower's spares.
- **Most unused objects can't be shown**: 354 are not in the archives. The 74 kept ones can.
- c2020 / c6031 / c7015 can't be spawned on PC (no model); the other cut-looking characters can.

## 9. Checks and not checked

- PS3 results unchanged by the shared-reader edits: old vs new `msb2.parse` + `hidden.parts` identical on every MSB of
  nt, jpnt, vanilla, v1.01, v1.03, v1.10 and PS3 Scholar (130 MSBs); PS3 Scholar `cut_survival.py` re-run gives
  byte-identical `cut_survival.tsv` / `cut_new_unused.tsv`; the 676-check stage 0 snapshot covered the NT tools.
- Not checked: whether the c1400 / c7210 generators fire in play; what the 12 new ungrouped Bastille parts are for;
  edited models' actual differences (counts only); textures (only names); sound banks.
