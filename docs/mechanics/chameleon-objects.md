# Chameleon: which objects you turn into (2026-11-14)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

Short answer: yes. The list is **AreaParam** (regulation), one row per map, row id = map id
(m10_10_00_00 -> 10100000; the code converts it in FUN_1403bf980). Each row names up to 5 object models
(PC Scholar) or 3 (vanilla PS3 / PC, and PS3 Scholar, which keeps the 3-slot table), each with a weight. Checked in PC Scholar 1.0.3.0 code (Ghidra, headless copy of DS2SPC)
and in the params of PC Scholar, vanilla PC 1.12 and PS3 v1.00.

## How it works (PC Scholar code)

- Spell 31280000 runs SpEffectSpell.emevd event 31280010 = command 1000:12, dev name キャラステータス擬態抽選
  "character status: mimicry lottery" (class SpEffectActionImpl_ChrStatus_ChooseChameleon). Its arguments are the
  events 31280001-31280005 (vanilla: 31280001-31280003).
- The lottery itself is in the exe (FUN_1401c65f0), not the script: it reads the current map's AreaParam row and draws a
  slot by weight. Then it runs the event in that slot's position, which is 1000:2 (0, slot, 5187), dev name
  キャラステータス擬態 "character status: mimicry" (SpEffectActionImpl_ChrStatus_Chameleon).
- The disguise is not a model swap on the player. Each map holds a stock of placed copies of every listed object
  (MapChameleonManager / MapChameleonAreaData, FUN_1401c57c0): the game collects copies of the model that carry a
  particular object type (value 0x22), and a slot is dropped for that map unless it finds exactly 6. Casting
  borrows a free copy and moves it onto the player each frame (MapChameleonActiveObjCtrl, FUN_1401c4aa0), turned and
  shifted by the row's per-slot rotation and offset. At most 6 players per area can be disguised at once.
  If the drawn slot has no free copy, nothing happens.
- Loose Scholar MSBs back this up: every listed model has at least 6 placements in its map, and several
  (o10_9300, o25_9300, o32_9300, o11_9300, the o00_2000/2010/2100 sets in the small linking maps) have exactly 6.
- The consumable Silver Talisman (60370000, 銀のタリスマン) uses the same system (see "Silver Talisman" below). It is
  NOT cut (corrected 15 Nov 2026; an earlier note here said it was).
- There are NPC versions too (ChooseNpcChameleon / NpcChameleon, dev names キャラステータスNPC擬態(抽選)): the
  Silver Talisman's NPC event (below) is the one user found; Maldron the Assassin carries it.

## Silver Talisman (15 Nov 2026)

The author asked whether the Silver Talisman works like Chameleon. **Yes, the same mechanism.** PC Scholar regulation,
decoded with a 64-bit EVD reader (header: event count +0x10, events +0x18 (0x30 each: id, instruction count, offset),
instructions +0x28 (0x20 each: bank, id, arg length, arg offset), arguments at +0x78):

| | Chameleon (spell 31280000) | Silver Talisman (item 60370000) |
|---|---|---|
| lottery | SpEffectSpell 31280010 = 1000:12(31280001-05) | SpEffectActiveItem 60370000 = 1000:12(60370001-05) |
| each slot | 100700:2(); 1000:2(0, slot, 5187) | 100700:2(); 1000:2(0, slot, 878) |

So the same five-slot AreaParam draw, chances, ‡ floor rule and shared stock of 6 copies per form (a talisman user
and a Chameleon caster take copies from the same pool). The third 1000:2 argument is a visual effect: f0005187.ffx
(Chameleon) and f0000878.ffx (talisman), both in sfx\sfx9999.ffxbnd, two different effects (22,735 / 22,927 bytes
unzipped). ItemParam 60370000: speffect_id 60370000, sfx_on_use_3 877 (f0000877.ffx), animation_speed 0.7, item type
200 / usage 1700, max held 99, price 1300 / sells 130. Chameleon's own SpEffect 31280000 adds 100040:1(5032, 249, 0)
and two 100170:1 calls (32080050 / 32230050, Bountiful Sunlight / Sacred Oath ids) that 19 spell effects share:
spell boilerplate, nothing Chameleon-specific.

Obtainable in every build (PS3 v1.00, PS3 Scholar, PC vanilla 1.12, PC Scholar): ItemLotParam2_Other 10185100
(No-man's Wharf) and 10256350 (the Gutter), plus 50355100 / 50366820 / 50375730 (the three DLC maps, not in PS3
v1.00); ItemLotParam2_Chr 10620000, 14600000, 14601000, 14602000; ShopLineupParam 78300601.

NPC use: SpEffectActiveItem 60375000 = 1000:20(60375001) and 60375001 = 100700:2(); 1000:21(0, 0, 878), the NPC
lottery with a single entry, slot 0. NpcPlayerStatusParam 8710 / 8711 (item_1 = 60370000, also Corrosive Urns, Rebel's
Greatshield) are EnemyParam 871000 / 871100 = Maldron the Assassin, the NPC invader in Brume Tower (generator 731) and
Frozen Eleum Loyce (generator 5320); dev event name 闇霊02：姑息戦士 "dark spirit 02: sneaky warrior". Reading:
Maldron's talisman always takes slot 1 of the area's row, i.e. the ash figure in Brume Tower and the plain barrel in
Eleum Loyce, whatever the floor under him. Code (checked in the council review, 15 Nov): the NPC action
FUN_140218870 (SpEffectActionImpl_ChrStatus_NpcChameleon, vtable 0x1410cc118) calls FUN_1401c6030(manager, chr,
slot), whose gate FUN_1401c6720() reads the floor under the LOCAL PLAYER (no character argument), so the NPC disguise
needs the player on a value 2 or 3 floor and ignores the NPC's own floor. Not checked in game.

## AreaParam layout

| | Scholar (108-byte rows) | Vanilla (80-byte rows) |
|---|---|---|
| +0x28 | u8 flag: 1 = keep slot 1 out of the normal draw | same |
| +0x29.. | u8 weight per slot (5) | u8 weight per slot (3) |
| object models | +0x30..+0x40, s32 x5 | +0x2C..+0x34, s32 x3 |
| rotation (degrees) | +0x44..+0x54 | +0x38..+0x40 |
| offset (probably height, not confirmed) | +0x58..+0x68 | +0x44..+0x4C |

Weights add up to 10 in almost every row, so read them as "out of 10". The vanilla column comes from lining the values
up with Scholar (the same rotations and offsets sit on the same objects); the vanilla exe was not decompiled.
SmithBox's vanilla AREA_PARAM def is wrong here: it reads mimicry_map_object_1-5 from +0x30, which is one field late
and runs into the rotations. Its Scholar def is right for the models; its Unk17-26 are the rotations and offsets.
The Network Test has no Chameleon: its AreaParam rows are 40 bytes and stop before these fields.

‡ = the +0x28 flag is set, so slot 1 is left out of the normal draw and only comes up on certain floors (next section).

## What switches the ‡ mode: the floor you stand on (traced 14 Nov 2026)

FUN_1401c6720 (disassembly read) takes the collision the local player is standing on (World = GameManagerImp
(0x1416148F0) +0x38; World +0x8 -> +0x20) and, if it is a map collision (+0xA2 == 2), returns byte +0x40 of that
collision's MSB type data (part entry +0x30 -> its +0x70 type-data pointer; the part's own +0x08 type is 3, as in the
file). Other code reads neighbouring fields of the same record: player light +0x1C and camera SFX +0x18 (FUN_1403c8a00),
play area +0x20 (FUN_14046fb80), hit group +0x3C for ESD checks (FUN_14046fc00). SmithBox calls the byte `UnkT40`.

| floor value | what Chameleon does (FUN_1401c65f0, from the disassembly) |
|---|---|
| 2 | normal weighted draw; a ‡ slot 1 is left out |
| 3 | area flag set: always slot 1 (the ‡ object). Flag not set: nothing happens |
| anything else, or not standing on a map collision | nothing happens (FUN_1401c5be0 / FUN_1401c6030 need 2 or 3) |

Values over every map collision in the 28 PS3 Scholar MSBs: 2 on 1216, 0 on 239 (all of them the special h99_* pieces;
**PC Scholar's own MSBs from the Steam archives have 0 on 244**: +2 Lost Bastille, +1 Gutter, +1 Heide's, +1 Brume
Tower, with values 2 and 3 identical; council review 15 Nov, review\2026-11-15\evidence\E7),
3 on 93, all in the seven maps with a ‡ row. PS3 v1.00 has the same value-3 floors in its four maps (no DLC), and the
loose PC Scholar MSBs agree.

Maps: `chameleon-objects\floors\chameleon_floors_<map>.png` (top-down, value-3 floors in orange over the other
floors, bonfires tagged with the value of the floor right under them). "On" below = the highest collision triangle
under the bonfire (exact test; its height matches the bonfire's, e.g. Cardinal Tower bonfire y 1.61, floor 1.65).

| map | value-3 floors | where | bonfires ON a value-3 floor | ‡ object |
|---|---|---|---|---|
| Forest of Fallen Giants | 1 (h06_3600) | a tall L-shaped structure (floors -26 to 9 m) right beside the Cardinal Tower bonfire, probably the tower itself; no enemy spawns on it | none (Cardinal Tower's own floor is 2) | red barrel |
| The Gutter & Black Gulch | 12 | all of Black Gulch (every floor with its area banner, MapNameID 102510) | Black Gulch Mouth, Hidden Chamber | yellow-green stone figure |
| Shaded Woods | 1 (h03_1000) | a large area above and left of Shaded Ruins on the map, plus a long path running down the map from it (-1 to 111 m) | none (Shaded Ruins is 2, at the patch's edge) | orange flower bush |
| Shrine of Amana | 1 (h03_2800) | a ~100 x 60 m area around Crumbled Ruins (-54 to 42 m) | none (Crumbled Ruins is 2, inside the patch's outline) | Elizabeth Mushroom |
| Shulva | 7 | Cave of the Dead, below Priestess' Chamber on the map (banner 503540; 4 more Cave floors are 2) | none | dark stone figure |
| Brume Tower | 10 | several levels of the central tower (-81 to 200 m) and the round room below and right of it on the map; top-down they cover most of the tower, but the levels are stacked | Lowermost Floor only | ash-covered figure |
| Frozen Eleum Loyce | 61 (44 others are 2) | buildings and passages inside the walls: the Grand Cathedral, the Inner Wall passage, rooms around Lower Garrison and Abandoned Dwelling; the streets, the Frigid Outskirts and the Outer Wall / Expulsion Chamber floors are 2 | Abandoned Dwelling, Lower Garrison, Grand Cathedral, Inner Wall | plain wooden barrel |

So the ‡ object is the area's "this spot" disguise: in Black Gulch you always become its idol-like figure, inside
Eleum Loyce's buildings a plain barrel, while the normal draw (ice, snowy crate, pillar) belongs to the other floors.

Huntsman's Copse has no value-3 floor in any build, so Scholar's ‡ slot-1 object o23_2011 (the bone throne) can never
come up; Scholar's effective Copse draw is o00_1901 (1), o00_2100 (5), o23_9300 (2), o23_9003 (1). In vanilla the Copse
row had no flag, and o23_9300 (birdcage) was a normal slot 1.

**The bone throne is the Skeleton Lords' side throne (15 Nov 2026; the author recognised it).** In the Skeleton Lords' arena
each of the three bosses sits on a bone throne as you enter. m10_23 MSB (PS3 v1.00, PS3 Scholar and PC Scholar alike):
o23_2010_0000 at (-416.8, 37.3, 255.6) yaw -20 (middle), o23_2011_0000 at (-404.8, 37.3, 258.4) yaw 25 and
o23_2011_0001 at (-425.9, 37.4, 252.5) yaw -50 (sides), ~24 m past the boss fog (o00_0501_0000 at (-406.9, 36.9,
232.3); the Boss Arena table's boss_dist from that fog is 24.3 m). PC Scholar adds the six Chameleon stock copies
o23_2011_0600-0605, lined up at x -172 about 310 m away. So the Chameleon form is the side throne's own model file:
identical to the two side thrones. The middle throne o23_2010 is a different model (PS3 Scholar files, FLVER read by
chameleon_render.load):

| model | size (x y z) | triangles | vertices | material / texture |
|---|---|---|---|---|
| o23_2010 middle throne | 3.32 x 7.24 x 2.02 m | 11,240 | 7,954 | Material #102 / doragon_arm_d |
| o23_2011 side throne = Chameleon form | 1.76 x 6.11 x 1.67 m | 9,036 | 6,363 | Material #102 / doragon_arm_d |

Both are tall-backed bone chairs; the middle one is wider with long bones flaring out at the top of the back. Each
binder also holds a `_1` FLVER + `_1_h.hkx`: the same throne pre-cut into pieces, the pattern breakable pots use
(o00_2100 has o00_2100_1.flv / _1_h.hkx too), so the thrones are breakable objects (not checked in game). Side-by-side
render: `chameleon-objects\renders\throne_compare_o23_2010_o23_2011.png`. Scholar added the side throne as the Copse's
‡ form (the thrones stand in the arena in v1.00 too).

## Can every form appear? (stock check, 14 Nov 2026)

The stock copies are marked in MapObjectInstanceParam_<map>: the row of the part's entity id has byte +0x18 = 0x22
(the check in FUN_1401c68e0 / FUN_1403ba6a0), map_obj_instance_type_id 1607 and default_state 0; ordinary copies of
the same models have 0x0A there or no instance row. `tools\chameleon_stock.py` counted marked copies per form, as
FUN_1401c57c0 does:

- PC Scholar (loose maps / params, `.orig` where the Boss Arena edited them): every one of the 105 forms in all 28 areas
  has exactly 6 marked copies, and every form has a weight above 0 or is a ‡ object with value-3 floors. So every form
  can appear on PC Scholar except o23_2011 (Huntsman's Copse bone throne): its 6 copies are there, but its slot is
  flagged and the Copse has no value-3 floor. It looks set up and then never given a floor.
- PS3 Scholar: AreaParam has vanilla's 80-byte rows and the lottery event has 3 arguments, so only the three original
  forms per area exist; its maps stock exactly 4 copies of each of those, and none of the forms PC Scholar added.
  The extra forms (nest, skeleton, desk, bone throne...) are PC Scholar only (PS4 / Xbox One not checked).
- Pool size: 4 copies on PS3 (e.g. o10_9300_0000-0003, o36_2350_0500-0503, lined up ~2 m apart), 6 on PC, matching
  4 players on PS3 and 6 on PC Scholar. The PC exe wants exactly 6; the PS3 exe's count was not checked.

Location method: `tools\chameleon_maps.py` reads the PS3 Scholar collision meshes (`model\map\hNN_NN_00_00.hkxbhd`;
each float4 vertex array is followed by its indices16 triangles, four u16 per triangle with a 0 pad), places them at
the part position (no collision part is rotated), draws the upward-facing triangles top-down and tests the floor under
each bonfire. Bonfire names: bonfire part entity id -> MapObjectInstanceParam row -> BonfireName.fmg.

Model ids are map objects: 21050 = o02_1050, 2010 = o00_2010. PS3 v1.00 rows are identical to PC 1.12; the four DLC
rows are on PC 1.12 only of what was checked.

## Vanilla PS3 vs vanilla PC (15 Nov 2026)

The author asked whether PS3 and PC vanilla share the tables. **Yes, exactly.** AreaParam's Chameleon fields (flag,
3 weights, 3 models, rotations, offsets; 80-byte rows) are identical in every build checked: PS3 v1.00, 1.01, 1.02,
1.03, 1.08, 1.09, 1.10, the JP disc, PC 2014-03-31 pre-release, 1.01, 1.02, the 2014-06-02 regulation update, 1.03
and 1.12, and PS3 Scholar too. The only change over time: the four DLC rows (50350000-50380000) appear in PS3 1.08 and
are the same in PS3 1.10 and PC 1.12. Stock copies (MapObjectInstanceParam +0x18 == 0x22): 4 of every form in PS3
v1.00, PS3 1.10 and PC 1.12 (78 forms, all exactly 4; so at most 4 disguised per area there, against PC Scholar's 6).
Chameleon floor values per map: identical too (PS3 1.10 and PC 1.12 both 2 x 1216, 0 x 239, 3 x 93, map by map; PS3
v1.00 the same minus the DLC maps). The vanilla PC MSBs use the PS3 layout in little-endian (msb2 pc=False, shift 0).
So PC Scholar (and probably PS4 / Xbox One Scholar, not checked) is the only version with a different list.
Scripts (scratchpad, not kept): AreaParam compare across all regulation folders; stock + floor count via gamedata
(PS3) and `DS2 Vanilla PC\tools\pcindex.py` read (PC 1.12 Steam install). Vanilla PC exe's own copy count not checked.

## Renders

All 73 models rendered with textures from PS3 Scholar (BLUS31528) on 14 Nov 2026: `chameleon-objects\renders\<model>.png`
(front, side, back, three-quarter; label = the model's material names, often the developers' Japanese labels such as
樽 "barrel", 木箱 "wooden box", 卵 "egg"), per-area pages `chameleon-objects\areas_*.png`, descriptions in
`chameleon-objects\names.tsv`. Tool: `DS2 Scholar PC\tools\chameleon_render.py` (textures from the object's binder,
then the map texture archive its paths name, then any map archive, then the equipment binders). Only o36_9350 (Brume
Tower Possessed Armor corpse) is missing textures: its three armour textures (am_hd_6560 / bd_6560_1 / bd_6560_2) were not
found, so those parts render grey. Nothing else names these objects: SmithBox's object aliases are blank.

The ‡ slot-1 objects are the odd ones out: a red barrel (Forest), a 6 m bone throne (the Skeleton Lords' side throne model) (Huntsman's Copse,
Scholar only), a yellow-green stone figure (Gutter), an orange flower bush (Shaded Woods), a 3.7 m Elizabeth Mushroom
(Shrine of Amana), a dark stone figure (Shulva), an ash-covered figure (Brume Tower) and a plain wooden barrel (Eleum
Loyce). Several stone figures share one mesh (0.51 x 1.44 x 0.47 m): o04_2020, o25_2020, o25_9300, o33_2000 and
o35_2010, whose texture is named kokesi (kokeshi doll).

Dark Souls throwbacks (17 Nov): the author found a Reddit post linking the Shaded Woods flower bush to the orange ground
flowers of the Royal Wood (Artorias of the Abyss, the past of Darkroot Garden). Side by side with a Royal Wood
screenshot the flowers match in colour and cup shape; the DS2 model is a 1.65 m clump on tall stems where the Royal
Wood ones are low ground cover. Its mesh / material are named Yflower / Yflower_A (yellow flower), nothing more. With
the Elizabeth Mushroom (Oolacile Sanctuary, next to the Royal Wood) that makes two location forms pointing at the same
corner of the DS1 DLC. Both models are placed only as the Chameleon stock across the 28 Chameleon maps: o32_9300 six
times in m10_32 (_0000-_0003 + _0600-_0601, the 06xx pair being Scholar's extra copies), o11_9300 six times in m20_11.
Not compared with DS1's own files (no DS1 install here). Whether it is a lore or a thematic link is not in the data.
Added to the Google Doc's Other observations on 17 Nov as "Possible Dark Souls throwback", worded (the author) as fair
speculation with no hard evidence, crediting "a Reddit post".

| Row | Map | Area | Scholar (PC 1.0.3.0): model, what it is (weight) | Vanilla (v1.00 to PC 1.12): model, what it is (weight) |
|---|---|---|---|---|
| 10020000 | m10_02 | Things Betwixt | o02_1050 stone grave tablet (3)<br>o00_1901 mossy stone pedestal (3)<br>o02_8510 wooden armchair (2)<br>o00_1030 skeleton lying on the ground (1)<br>o02_9020 large bird's nest, 3.3 m across (1) | o02_1050 stone grave tablet (4)<br>o00_1901 mossy stone pedestal (4)<br>o02_8510 wooden armchair (2) |
| 10040000 | m10_04 | Majula | o04_2020 small weathered stone figure (2)<br>o00_2010 wooden crate (3)<br>o00_2000 wooden barrel (3)<br>o00_2560 wooden desk (2) | o04_2020 small weathered stone figure (2)<br>o00_2010 wooden crate (4)<br>o00_2000 wooden barrel (4) |
| 10100000 | m10_10 | Forest of Fallen Giants | o10_9300 red barrel (0) ‡<br>o00_2000 wooden barrel (3)<br>o00_2010 wooden crate (3)<br>o00_2700 propped wooden boards with an arrow (2)<br>o00_1900 iron brazier on a tripod (2) | o10_9300 red barrel (0) ‡<br>o00_2000 wooden barrel (5)<br>o00_2010 wooden crate (5) |
| 10140000 | m10_14 | Brightstone Cove Tseldora | o00_2010 wooden crate (5)<br>o14_2120 large clay jar (2)<br>o14_1160 statue of a robed woman (1) [turned 180°]<br>o14_1900 iron basin on a stand (1) | o14_1160 statue of a robed woman (1) [turned 180°]<br>o00_2010 wooden crate (6)<br>o14_2120 large clay jar (3) |
| 10150000 | m10_15 | Aldia's Keep | o15_2020 round wooden table (2)<br>o00_2740 ornate urn on a plinth (3)<br>o15_2005 large framed painting, 3.5 m tall (1) [turned 270°, offset 1.8]<br>o15_2050 weapon rack (2)<br>o15_9003 iron candle stand (2) | o15_2020 round wooden table (4)<br>o00_2740 ornate urn on a plinth (5)<br>o15_2005 large framed painting, 3.5 m tall (1) [turned 270°, offset 1.8] |
| 10160000 | m10_16 | The Lost Bastille & Belfry Luna | o00_2242 barrel on a wooden cradle (4)<br>o16_1303 doll-shaped stone figure (1)<br>o16_1910 tall iron cage stand (3)<br>o00_2740 ornate urn on a plinth (2) | o00_2242 barrel on a wooden cradle (5)<br>o16_1303 doll-shaped stone figure (1)<br>o16_1910 tall iron cage stand (4) |
| 10170000 | m10_17 | Harvest Valley & Earthen Peak | o00_2999 tall clay jar of poison water (3)<br>o00_2100 large clay pot (3)<br>o00_2010 wooden crate (2)<br>o00_1030 skeleton lying on the ground (1)<br>o00_3006 wide cracked pot of poison water (1) | o00_2999 tall clay jar of poison water (4)<br>o00_2100 large clay pot (3)<br>o00_2010 wooden crate (3) |
| 10180000 | m10_18 | No-man's Wharf | o00_2090 wooden cabinet (2)<br>o18_2040 tall clay jar of poison water (1)<br>o00_2000 wooden barrel (2)<br>o18_1900 bundle of sharpened stakes (2)<br>o00_3001 tall pot with a rope-tied lid (3) | o00_2090 wooden cabinet (3)<br>o18_2040 tall clay jar of poison water (1)<br>o00_2000 wooden barrel (6) |
| 10190000 | m10_19 | Iron Keep & Belfry Sol | o19_9300 horned stone tablet (2)<br>o19_1070 dark iron urn on a stand (4)<br>o00_2100 large clay pot (3)<br>o19_1015 wooden ballista (1) | o19_9300 horned stone tablet (2)<br>o19_1070 dark iron urn on a stand (5)<br>o00_2100 large clay pot (3) |
| 10210000 | m10_21 | Drangleic Castle (alt) | none | none |
| 10230000 | m10_23 | Huntsman's Copse & Undead Purgatory | o23_2011 6 m bone throne (the Skeleton Lords' side throne model) (1) ‡<br>o00_1901 mossy stone pedestal (1)<br>o00_2100 large clay pot (5)<br>o23_9300 iron birdcage (2)<br>o23_9003 lidded clay urn (1) | o23_9300 iron birdcage (3)<br>o00_1901 mossy stone pedestal (1)<br>o00_2100 large clay pot (6) |
| 10250000 | m10_25 | The Gutter & Black Gulch | o25_9300 yellow-green stone figure (7) ‡<br>o25_2020 small stone figure (1)<br>o25_3004 dark egg-shaped pot (1)<br>o25_1900 broken stone statue (1)<br>o25_2120 large clay jar (1) | o25_9300 yellow-green stone figure (9) ‡<br>o25_2020 small stone figure (1)<br>o25_3004 dark egg-shaped pot (1) |
| 10270000 | m10_27 | Dragon Aerie & Dragon Shrine | o27_2000 dragon egg (6)<br>o00_2100 large clay pot (3)<br>o00_2010 wooden crate (1) | o27_2000 dragon egg (6)<br>o00_2100 large clay pot (3)<br>o00_2010 wooden crate (1) |
| 10290000 | m10_29 | Majula <-> Shaded Woods | o00_2000 wooden barrel (4)<br>o00_2100 large clay pot (4)<br>o00_2010 wooden crate (2) | o00_2000 wooden barrel (4)<br>o00_2100 large clay pot (4)<br>o00_2010 wooden crate (2) |
| 10300000 | m10_30 | Heide's Tower <-> No-man's Wharf | o00_2100 large clay pot (4)<br>o00_2000 wooden barrel (4)<br>o00_2010 wooden crate (2) | o00_2100 large clay pot (4)<br>o00_2000 wooden barrel (4)<br>o00_2010 wooden crate (2) |
| 10310000 | m10_31 | Heide's Tower of Flame | o00_2010 wooden crate (1)<br>o00_2000 wooden barrel (1)<br>o00_2100 large clay pot (8) | o00_2010 wooden crate (1)<br>o00_2000 wooden barrel (1)<br>o00_2100 large clay pot (8) |
| 10320000 | m10_32 | Shaded Woods & Shrine of Winter | o32_9300 orange flower bush (0) ‡<br>o32_9003 stone figure with an iron ring (3)<br>o32_2000 lidded clay urn (6)<br>o32_1020 open-topped barrel (1) | o32_9300 orange flower bush (0) ‡<br>o32_9003 stone figure with an iron ring (3)<br>o32_2000 lidded clay urn (7) |
| 10330000 | m10_33 | Doors of Pharros | o33_9402 headless armoured torso (6)<br>o33_1900 square stone pillar (3)<br>o33_2000 small stone figure (1) | o33_9402 headless armoured torso (6)<br>o33_1900 square stone pillar (3)<br>o33_2000 small stone figure (1) |
| 10340000 | m10_34 | Grave of Saints | o34_9004 giant rat (5)<br>o00_1900 iron brazier on a tripod (1)<br>o34_9001 rat statue beside a plinth (4) [turned 90°] | o34_9004 giant rat (5)<br>o00_1900 iron brazier on a tripod (1)<br>o34_9001 rat statue beside a plinth (4) [turned 90°] |
| 20100000 | m20_10 | Memory of Vammar, Orro, and Jeigh | o10_1010 wooden ballista (7)<br>o00_3001 tall pot with a rope-tied lid (3) | o10_1010 wooden ballista (10) |
| 20110000 | m20_11 | Shrine of Amana | o11_9300 Elizabeth Mushroom, 3.7 m tall (0) ‡<br>o11_1900 broken stone pillar (5)<br>o11_2100 lidded clay urn (5) | o11_9300 Elizabeth Mushroom, 3.7 m tall (0) ‡<br>o11_1900 broken stone pillar (5)<br>o11_2100 lidded clay urn (5) |
| 20210000 | m20_21 | Drangleic Castle & Throne of Want | o21_9051 oval mirror (1) [offset 1.5]<br>o00_2010 wooden crate (4)<br>o00_2100 large clay pot (3)<br>o21_9080 statue of a knight with a spear (1)<br>o00_2315 armour on a wooden stand (1) | o21_9051 oval mirror (1) [offset 1.5]<br>o00_2010 wooden crate (6)<br>o00_2100 large clay pot (3) |
| 20240000 | m20_24 | Undead Crypt | o00_3001 tall pot with a rope-tied lid (3)<br>o24_9000 boulder (3)<br>o24_9002 dark grave slab (2)<br>o24_1110 dark stone statue on a plinth (1) | o00_3001 tall pot with a rope-tied lid (4)<br>o24_9000 boulder (4)<br>o24_9002 dark grave slab (2) |
| 20260000 | m20_26 | Dragon Memories | o00_2010 wooden crate (10) | o00_2010 wooden crate (10) |
| 20620000 | m20_62 | (E3 demo copy) | none | none |
| 20630000 | m20_63 | (E3 demo copy) | none | none |
| 40030000 | m40_03 | Dark Chasm of Old | o00_1901 mossy stone pedestal (4)<br>o00_2100 large clay pot (3)<br>o00_2000 wooden barrel (3) | o00_1901 mossy stone pedestal (4)<br>o00_2100 large clay pot (3)<br>o00_2000 wooden barrel (3) |
| 50350000 | m50_35 | Shulva, Sanctum City | o35_2010 dark stone figure (0) ‡<br>o35_0740 spine pillar (6) [offset 0.1]<br>o35_1000 fleshy egg cluster (2)<br>o35_2100 open treasure chest (1)<br>o35_3006 wide cracked pot (1) | o35_2010 dark stone figure (0) ‡<br>o35_0740 spine pillar (7) [offset 0.1]<br>o35_1000 fleshy egg cluster (3) |
| 50360000 | m50_36 | Brume Tower | o36_2350 ash-covered hooded figure (0) ‡ [turned 330°]<br>o36_2244 barrel (4) [turned 270°]<br>o36_2100 wooden scaffold (4)<br>o36_9350 Possessed Armor corpse (with two weapons) (2) | o36_2350 ash-covered hooded figure (0) ‡ [turned 330°]<br>o36_2244 barrel (5) [turned 270°]<br>o36_2100 wooden scaffold (5) |
| 50370000 | m50_37 | Frozen Eleum Loyce | o00_2000 wooden barrel (0) ‡<br>o37_1044 cluster of ice (3)<br>o37_2010 snow-covered crate (5)<br>o37_9007 broken stone pillar (2) | o00_2000 wooden barrel (0) ‡<br>o37_1044 cluster of ice (3)<br>o37_2010 snow-covered crate (7) |
| 50380000 | m50_38 | Memory of the King | o00_3001 tall pot with a rope-tied lid (10) | o00_3001 tall pot with a rope-tied lid (10) |

## Scholar changes

Scholar keeps every vanilla form (Tseldora and Huntsman's Copse reorder theirs) and adds one or two in 18 of the
28 areas that have forms, so each original form comes up less often. Huntsman's Copse also gains a held-back slot 1 (o23_2011). Memory of Vammar,
Orro and Jeigh goes from one form to two.

## Cross-check against the wiki.gg Chameleon page (14 Nov 2026)

Source: darksouls2.wiki.gg/wiki/Chameleon, oldid 27222 (last edited 28 Oct 2026), saved as PDF in Downloads. The page
lists 4-5 forms in several areas (Things Betwixt, Majula, Harvest Valley), so it describes Scholar on PC / PS4 / Xbox One,
not PS3 (3 forms per area). Every form the wiki names matches one of ours; the matches below are by render. The wiki
names o00_3001 "Ceramic Barrel" (Undead Crypt, Memory of the King), o00_1030 "Bone Pile", o00_1901 / o00_1900 / the
xx_1900 pieces "Sconce", o16_1303 (2 m stone pot with a grille on one side) "Barred Ceramic Pot", o36_2100 "Cheap
Wooden Shelf", o36_2244 "Stack of Barrels" (it is one barrel).

Forms the wiki does not list (besides the nest and the bone throne):

| area | missing form | weight | why it is easy to miss |
|---|---|---|---|
| Forest of Fallen Giants | o10_9300 red barrel ‡ | - | only on the one value-3 piece beside Cardinal Tower |
| Shaded Woods | o32_9300 orange flower bush ‡ | - | only on the one value-3 piece above / left of Shaded Ruins |
| Frozen Eleum Loyce | o00_2000 plain wooden barrel ‡ | - | only inside the buildings; the wiki's Grand Cathedral / Old Chaos row is blank, and both are value-3 floors throughout (5 and 1 pieces), so it would always be this barrel |
| No-man's Wharf | probably o00_3001 tall pot with a rope-tied lid | 3 | 5 forms, wiki lists 4; it names o00_3001 "Ceramic Barrel" elsewhere, so "Oil Sconce" was guessed to be the 2 m jar o18_2040. SUPERSEDED 15 Nov: the author saw the jar live and it is plainly a jar, so the doc counts both the jar and o00_3001 as new (18 entries) and leaves "Oil Sconce" open |
| Memory of Vammar, Orro, Jeigh | o00_3001 tall pot | 3 | wiki lists only the ballista |
| Aldia's Keep | o15_2050 weapon rack, o15_9003 iron candle stand | 2 each | |
| Brume Tower | o36_9350 Possessed Armor corpse (with two weapons) | 2 | |
| Drangleic Castle | o21_9080 knight statue with a spear, o00_2315 armour on a wooden stand | 1 each | |
| Undead Crypt | o24_1110 dark stone statue on a plinth | 1 | |
| Shulva | o35_2100 open treasure chest, o35_3006 wide cracked pot | 1 each | |
| Dark Chasm of Old | the whole row: o00_1901 mossy stone pedestal (4), o00_2100 large clay pot (3), o00_2000 wooden barrel (3) | | wiki row blank; its 26 non-special floors are value 2, so the normal draw works there |

Ten models appear nowhere on the page: red barrel, flower bush, weapon rack, candle stand, Possessed Armor corpse, knight
statue, armour stand, Crypt plinth statue, treasure chest, Shulva wide pot (plus the nest and the bone throne). The
normal-draw misses are all Scholar additions at weight 1-3 of 10; the page does have many other Scholar additions
(desk, barricade, Forest brazier, skeleton, the stake sconce, Tseldora basin, Bastille urn, Harvest Valley small pot,
Iron Keep ballista, Copse urn, Shaded Woods open barrel, Gutter statue and jar), so it reads as a Scholar list built
from too few casts per area. The three ‡ misses need one particular floor.

Other differences:
- "Shaded Woods (Old Akelarre)" is map m10_29 (the Old Akelarre bonfire, 29650, is in it): barrel 4, clay pot 4,
  crate 2. The wiki's fourth form there, "Cursed Ceramic Pot", is not in that row; it is the commonest Shaded Woods
  (m10_32) form, o32_2000 (6 of 10), so it was probably cast over the map boundary.
- The wiki ties some forms to places ("Giant Wooden Barrel (Shaded Ruins)", "Ornate Sconce (Chapel Threshold / Lower
  Brightstone Cove)") and gives Brume Tower's sub-areas one form each. Only ‡ forms depend on the spot; on value-2
  floors the whole row is drawn anywhere in the map. Iron Passage (17 pieces) and Memory of the Old Iron King (8) are
  all value 2, so each gives barrel / shelf / corpse, while the tower itself (10 value-3 pieces) gives the ash figure.
- Frigid Outskirts (blank on the wiki): 9 value-2 pieces, so the Eleum Loyce normal draw (ice, snowy crate, pillar).
- Area names per floor piece: MSB collision type data +0x28 MapNameID (e.g. 503610 -> MapName.fmg 50360001 Iron
  Passage), read from the PS3 Scholar MSBs.

## Forcing a form in a running game (PC, 14 Nov 2026)

`tools\chameleon_force.py` (on top of pcmem.py: Steam offline only, every write logged): the draw reads the area's
AreaParam row live on every cast, so writing weight 10 on one slot and 0 on the rest (flag 0) makes every cast on an
ordinary floor give that object; `cycle` moves the 10 on after each cast (it watches the stock entries' in-use bytes),
so repeated casts show every stocked form in turn, and `restore` (also on Ctrl+C) puts the bytes back. Pointer chain:
[[BaseA] + 0x38] + 0x208 = MapChameleonManager, +0x58 area-data pointers (count +0x78), area data +0x10 = AreaParam row,
+0x18 stock entries (count +0xE8). With the flag cleared, `force 1` in Huntsman's Copse should show the otherwise
unreachable bone throne, whose stock exists. NOT YET RUN against the live game.

## Live test (15 Nov 2026, Steam install, no mods)

Clean Steam install (exe and regulation SHA-1 = the originals), Steam offline, all-99 test character, Chameleon given
with arenactl give 31280000.
- `status` read the live tables right first time: Forest flag 1, weights 0/3/3/2/2; Things Betwixt flag 0, weights
  3/3/2/1/1; every form stocked 6/6, as the data says.
- **Bird's nest PASS**: Things Betwixt forced to slot 5, the author cast and became the nest; the tool saw a nest stock copy
  (o02_9020) go into use. So the nest works in game; its absence from community lists is rarity (1 in 10) or not being
  recognised, not a fault. The full Things Betwixt cycle then ran (6 casts: nest, grave tablet, pedestal, armchair,
  skeleton, nest), each cast taking the forced form's copy; the row restored itself when the author warped away.
- Aldia's Keep (warped to Foregarden 15650): loads with m10_32 Shaded Woods beside it, so `--map m10_15` is needed.
- **Full recording run (15 Nov, 06:40-08:35, the author recorded every stop in OBS for the wiki;
  `chameleon-objects\recording-route.md`)**: 102 of the 105 forms seen live, each cast's borrowed stock copy logged
  by the tool. Not cast: only the Heide's to No-man's Wharf link row (m10_30, no bonfire; same three models as Heide's).
  - Normal forms: forced one per cast with `cycle` at one bonfire per area (23 areas); every cast took the forced form.
  - ‡ forms, NOT forced (read-only `watch`, game weights): Grand Cathedral (h02_2100, value 3) 3 x plain barrel;
    Lowermost Floor (h06_6000) 3 x ash figure; Cave of the Dead (h08_8500) 2 x dark stone figure; Black Gulch Mouth
    (h04_2100) yellow-green figure; Amana (h03_2800, a lower level of the ruins than the Crumbled Ruins bonfire's
    h03_2400) Elizabeth Mushroom; Shaded Woods (h03_1000, west of Shaded Ruins) flower bush; Forest (h06_3600, by Cardinal
    Tower) red barrel. So the floor rule traced from code holds live: on a value-3 floor every cast gives slot 1
    although its weight is 0. Bone throne (Copse): seen only forced (flag cleared, slot 1 weighted), as expected.
  - No-bonfire maps, walked in (Ashen Mist Heart): Memory of the King (from the Undead Crypt, Undead Ditch) tall lidded
    pot; a giant's memory m20_10 (from the Forest) ballista + tall lidded pot; Dragon Memories (from Tseldora, Chapel
    Threshold side) crate. Which script enters each: event_m20_24 -> 50380000, event_m10_10 -> 20100000, event_m10_14
    -> 20260000; the Dark Chasm portals are in event_m10_25 / m10_32 / m20_21.
  - Dark Chasm (m40_03): reached the Boss Arena way: covenant 9 (Pilgrims of Dark), global flags 102326 = 1 (portal
    open) and 102329 = 0 (not used), then a warp to Under Castle Drangleic (21665) so the castle reads them on load;
    the castle portal then took the author in. Pedestal, clay pot, barrel all cast (floor h03_1800, value 2). The character
    is left in Pilgrims of Dark.
  - Cast Light (31270000) cast before Chameleon: the light stays while disguised, but the glowing orb above the head
    disappears (the author). Guess, not checked: the orb is an effect on the player's model, which the disguise hides,
    while the light itself is not tied to the model.
  - Tools added during the run: `chameleon_force.py floor [once]` (the collision under the player and its value, read
    as FUN_1401c6720 does: X = [[World+8]+0x20], part entry [X+0x30], name at its +0, type data at +0x70, value
    +0x40; matched the expected pieces every time) and `watch` (read-only cast log with the floor, survives loads
    without `--map`). A map reload (warp to another bonfire of the same map) stops a cycle; it restores and needs a
    restart.
  - Old Akelarre bonfire: only m10_29 + Majula loaded, not Shaded Woods, so the wiki's extra "Cursed Ceramic Pot"
    there is not from casting at the bonfire (open).
- No-man's Wharf: the author saw the 2 m clay jar o18_2040 live and says it is plainly a jar, not the wiki's "Oil Sconce".
  So the wiki's "Oil Sconce" is still unidentified (stake stand o18_1900 = its "Sconce"?); OPEN, look at it again
  with the recording.
- Lesson: the AreaParam rows sit in the regulation, loaded once per session, so a forced row stays forced after its map
  unloads. `restore` now checks the row at its saved address (model ids) and writes it back; the Forest row was
  restored that way and read back as 0/3/3/2/2.

## Wiki write-up (15 Nov 2026)

Google Doc "Chameleon transform objects: proposed wiki update"
(link not published), for the author to discuss with the
wiki contributors: method, table with the wiki's names and the chance of each form (weight / total), errors to remove,
how location (‡) forms work, the seven floor maps, the bone throne as unused content. Built by
`tools\chameleon_wiki_doc.py` (local copy `chameleon-objects\Chameleon wiki update.docx`); maps from
`tools\chameleon_maps.py --wiki` in `chameleon-objects\wiki-maps\`. The author: the wiki's existing names were right wherever
we agreed on the object, so the doc uses them. 15 Nov later: bone throne section rewritten, names updated (Possessed
Armor Corpse, Elizabeth Mushroom), and a section "Original Dark Souls II and PS3 Scholar" added after the main table
(vanilla table with chances, wiki names, merged location cells; inserted through the Docs API, builder updated to
match). Then a "Silver Talisman" section before "Other observations" (same mechanism; Maldron reading, unverified).

## Open

- The wiki's No-man's Wharf "Oil Sconce": not the clay jar o18_2040 (the author, live); check the recording.
- The wiki's Old Akelarre "Cursed Ceramic Pot" (see the run notes).
- Cast on an h99_* floor or on a lift (expect nothing): not tried.
- Renders vs the recordings (rotation / offset): compare when the author's footage is cut.
- Vanilla layout confirmed from data only, not from the vanilla exe (DS2VPC project exists).
