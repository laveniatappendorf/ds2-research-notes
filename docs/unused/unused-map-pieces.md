# Unused map pieces in retail v1.00

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-10-17, follow-on from `developer-signs.md` and `hidden-map-parts.md`. Retail BLUS41045 v1.00, static work plus
three in-game tests (the Undead Crypt statue, and an Iron Keep hall and a Huntsman's gatehouse for missing textures, under
"Can they be seen in game?"). Renders and tables: `findings\unused-map-pieces\`. Tools: `tools\devsigns\unused_*.py`,
`labels.py`.

## What this covers

874 map-piece models sit in retail's map archives (`model\map\<map>.mapbhd/.mapbdt`) without any layout part using them:
749 aren't in their map's MSB model table at all, 125 are listed there but no part places them (shadow `_s` models left
out). Every one was rendered (top, front and oblique views, its footprint over the map's placed geometry, and an overlay
with the closest model) and looked at on per-map contact sheets. The notable ones were re-rendered larger.

## Headlines

- **Two earlier whole maps.** Lost Bastille `m0000` is a sprawling walled castle on a cliff plateau, 114k triangles,
  sitting where the retail Bastille stands (43% of it lies on the placed geometry), and it carries **3D-text labels for
  characters**: 防具鍛冶屋 "armour smith", 追跡する戦士 "pursuing warrior", 妹を探す兄 "brother searching for his
  sister" and three copies of 少女 ハンシ 仕込み "girl / hanshi / set-up". Dark Chasm `m0000` is a bigger version of the
  Dark Chasm cave with long extra tunnels, textured `m10_03_*`, which turns out to be the Dark Chasm's own working map
  number.
- **Cut areas in map coordinates.** Iron Keep has a whole district of 26 pieces beside the keep, hardly touching the
  retail map: gothic halls with spires, a 142 m ring wall, a 90 m sculpted mass, walkways, and 590 m chains. There are
  smaller ones in Huntsman's Copse (a gatehouse and plaza), Harvest Valley (an older mine network, a domed building off
  the map), the Undead Crypt (older corridors), Shrine of Amana (root-wrapped towers and cliffs off the map) and the Dark
  Chasm (a 455 m cliff wall with a stilted building).
- **Older versions of what's there.** Harvest Valley's earlier terrain tiling (up to 99% over today's ground), Doors of
  Pharros house clusters (96-97% over the placed ones), two revisions of a Things Betwixt plateau, and 25 Undead Crypt
  vault / arch pieces that line up with the crypt's round central hall.
- **Five more models with 3D text**, beyond the 12 sign models and the 4 Dark Chasm prop labels already written up: the
  Bastille labels above, ボス部屋への昇降機 "lift to the boss room" (Undead Crypt), 崩落 "collapse" (the Gutter),
  篝火 "bonfire" (Grave of Saints) and 素材屋 "materials shop" (Harvest Valley).
- **Unplaced set dressing and figures:** four ships in No-man's Wharf, a dog in Majula, a T-posed human figure in the
  Lost Bastille, a seated mannequin in Amana, two colossal statues (86 m and 97 m) shared by the Gutter and the Grave of
  Saints, a dragon skull, arm and bones in Aldia's Keep, a dragon ribcage in the Dark Chasm, tent sets, furniture,
  windmill gearing, Drangleic dial and turntable mechanisms, a domed rotunda, thrones, statues.
- **Network test:** 107 of Huntsman's Copse's 110 unused pieces were already unused in the network test (103 of them
  byte-identical, 4 with identical geometry). Besides the signs, the test placed two that retail dropped: `m0031` (a
  waterfall) and `m0278` (bony arm-like growths, textures `doragon_arm`).
- **The developer signs annotate the retail layouts.** In all 12 sign maps the sign boards sit on placed geometry,
  except one Harvest Valley sign that sits on an unused blockout building (and two Drangleic boards just over 4 m away).

## How it was done

1. **Survey** (`unused_survey.py`, DS2_GAME picks the build). Per map: the MSB's map-piece parts and their transforms,
   and for every model in the map archive its size, materials, texture names, an exact-geometry key (rounded vertex
   set), the flat-glyph count from `detect.py`, and a surface sample on a 1 m grid.
2. **Map origin.** Baked map pieces are modelled in map coordinates and placed at one shared position. The origin is
   the most common position of the *unrotated, unscaled* map-piece parts. In every map with unused pieces but one, that
   is also the most common position of all parts. The Undead Crypt is the exception: 8 rotated copies of `m0203` and
   `m0049` sit at its central hall (-1021.2, -124.7, -179.5), but the four unrotated pieces are at (-1115, -157, -196). That is the
   origin `developer-signs.md` used, and the right one: the crypt signs sit on placed geometry there.
3. **Where a piece would sit.** A piece whose own origin is far outside its bounding box ("far", or "near" and over
   15 m) is modelled in map coordinates: its centre plus the origin gives the map position (MSB coordinates, as in the
   sign tables). A piece with its origin inside its box is a prop modelled at its own origin, to be placed anywhere
   (position unknown), unless it is big and lines up with the map at the origin (the two whole maps).
4. **Overlap.** The share of the piece's 1 m surface cells that lie within one cell of the map's placed geometry, plus
   the placed models those cells belong to and how much of each the piece covers back. For props: a model-space
   likeness score (shared cells at 1/40 of the bounding diagonal) against every model of similar size in the map.
   Placed parts are transformed with the DSMapStudio convention (scale, then X, Z, Y rotations). The renders of
   transformed parts look right (walls and floors join up), but the convention isn't otherwise proven.
5. **Renders** (`unused_render.py`, `unused_sheets.py`, `unused_closeup.py`): flat-shaded orthographic views coloured
   by material, top-down context (x right, z down; which way is north hasn't been established).
6. **Text** (`labels.py`): flat pieces of 10 or more triangles under 1 m seed clusters, and each cluster is drawn
   face-on. Run on the unused models with 3 or more such pieces.
7. **Network test**: the same survey on NPUB90979's m10_23 (`survey_nt`), compared model by model.

### How closely each piece was looked at

- All 874 tiles were looked at on the contact sheets (`unused-map-pieces\sheets\`, 7 pieces a page, 134 pages, views
  about 200 px). Most got a glance: enough to say "rock", "tree", "wall segment" or "vault bay", not a study.
- About 35 groups were re-rendered larger (the images below) and looked at properly.
- Only geometry, material names and texture *names* were checked. No texture was viewed, so colour, condition and
  anything painted on are unknown. Identifications are by eye from untextured, flat-shaded renders, by an AI assistant, and the
  translations are the AI assistant's: check anything you want to rely on.
- `pieces.tsv` gives an automatic category for every piece (from frame, overlap and likeness), with hand notes for the
  273 described here or in the viewing notes. The categories are a rough sort, not a judgement.

## Per-map summary

Counts: **n** unused (not in MSB / listed); **over** / **part** / **off** = map-coordinate pieces with 75%+ / 25-75% /
under 25% of their surface on placed geometry; **prop** = own-frame models with nothing similar placed; **var** = close
variants of another model; **same** = identical geometry to a model used somewhere; **sky** = domes, seas, backdrops over
1 km; **hall** = Undead Crypt pieces that line up at the central hall; **blk** = flagged as blockout (placeholder texture
names, or under 200 triangles and over 8 m). Overview maps: `unused-map-pieces\overview\<map>.png`.

| Map | Area | n (not / listed) | over | part | off | prop | var | same | sky | other | blk | What's there |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| m10_02 | Things Betwixt | 48 (41 / 7) | 0 | 5 | 5 | 25 | 9 | 1 | 1 | 2 empty | 6 | older terrain, a rock spire with a doorway, rocks and logs, a furniture set |
| m10_04 | Majula | 102 (97 / 5) | 5 | 3 | 21 | 61 | 11 | 1 | 0 | | 9 | cross-plan hall, rotunda, rock-tunnel kit, shafts, tents, a dog, trees, rocks, wells |
| m10_10 | Forest of Fallen Giants | 1 (0 / 1) | 0 | 1 | 0 | 0 | 0 | 0 | 0 | | 0 | a duplicate of a placed tree-card scatter |
| m10_15 | Aldia's Keep | 24 (18 / 6) | 0 | 1 | 7 | 12 | 3 | 0 | 1 | | 2 | dragon skull, arm and bones, impaled soldiers, weapon rack, towers |
| m10_16 | Lost Bastille | 150 (143 / 7) | 5 | 13 | 14 | 86 | 26 | 3 | 1 | 1 empty, signs | 41 | **an earlier whole Bastille with labels**, wall / vault / stair / mine kits, blockouts, T-posed figure, moon sky |
| m10_17 | Harvest Valley / Earthen Peak | 63 (49 / 14) | 8 | 16 | 21 | 13 | 1 | 2 | 1 | signs | 17 | older terrain, an older mine network, gears, a domed building off the map, blockout buildings |
| m10_18 | No-man's Wharf | 23 (14 / 9) | 0 | 1 | 4 | 10 | 4 | 3 | 0 | signs | 3 | four ships, harbour houses, cliffs the Bastille places instead |
| m10_19 | Iron Keep | 54 (36 / 18) | 3 | 5 | 27 | 13 | 3 | 2 | 0 | signs | 5 | **a cut district of gothic buildings**, older room pieces, a caryatid |
| m10_23 | Huntsman's Copse / Undead Purgatory | 110 (106 / 4) | 5 | 16 | 16 | 54 | 15 | 0 | 2 | 1 empty, signs | 12 | gatehouse plaza, Purgatory alternates, tents, graves, trees; two pieces the NT placed |
| m10_25 | The Gutter / Black Gulch | 17 (15 / 2) | 0 | 0 | 8 | 6 | 0 | 2 | 0 | signs | 0 | colossal statues, a river and lock system, a pillar field, a tank tower |
| m10_27 | Dragon Aerie | 5 (0 / 5) | 0 | 0 | 0 | 4 | 0 | 1 | 0 | | 0 | columns, a pedestal, rocks |
| m10_30 | small map (name not checked) | 1 (1 / 0) | 0 | 0 | 0 | 1 | 0 | 0 | 0 | | 0 | a pedestal |
| m10_31 | Heide's Tower of Flame | 5 (2 / 3) | 0 | 1 | 2 | 1 | 0 | 0 | 0 | signs | 3 | an older plaza floor, a 95 m pole, a "prison" distant-view card |
| m10_32 | Shaded Woods | 1 (1 / 0) | 0 | 0 | 0 | 1 | 0 | 0 | 0 | | 0 | a dead tree |
| m10_33 | Doors of Pharros | 6 (4 / 2) | 3 | 0 | 1 | 2 | 0 | 0 | 0 | | 1 | older house clusters, a blockout box |
| m10_34 | Grave of Saints | 25 (21 / 4) | 1 | 1 | 10 | 8 | 1 | 3 | 0 | signs | 2 | mostly the Gutter's unused pieces, two big complexes, hanging corpses |
| m20_10 | the giant memories (name not checked) | 7 (2 / 5) | 0 | 1 | 3 | 1 | 0 | 2 | 0 | | 0 | a rock arch, floor slabs, Forest copies |
| m20_11 | Shrine of Amana | 32 (4 / 28) | 0 | 1 | 10 | 17 | 3 | 0 | 0 | signs | 4 | root-wrapped towers, cliffs off the map, marker scatters, a mannequin |
| m20_21 | Drangleic Castle | 23 (22 / 1) | 0 | 0 | 5 | 13 | 2 | 0 | 2 | signs | 4 | rotunda, knight statue, dials and a turntable, a long colonnade bridge, a throne |
| m20_24 | Undead Crypt | 70 (70 / 0) | 0 | 1 | 22 | 17 | 2 | 0 | 2 | 25 hall, signs | 4 | older corridors, central-hall segments, huge round structures, a statue |
| m20_26 | (name not checked) | 3 (1 / 2) | 0 | 0 | 0 | 2 | 0 | 1 | 0 | | 0 | rocks, a claw-like root spire |
| m40_03 | Dark Chasm of Old | 104 (102 / 2) | 0 | 1 | 4 | 77 | 11 | 4 | 2 | signs, 4 labelled props | 5 | **an earlier, larger Dark Chasm**, the cave kit it was built from, statues, tents, a dragon ribcage |
| | **Total** | **874 (749 / 125)** | 30 | 67 | 180 | 424 | 91 | 25 | 12 | 4 empty, 25 hall, 16 known | 118 | |

Brightstone Cove Tseldora (m10_14) and m10_29 have none. Of the two whole-map pieces, Lost Bastille `m0000` counts under
"part" (43%) and Dark Chasm `m0000` under "prop" (only 13% of it lies on placed geometry; see below).

## Notable pieces

Coordinates are map (MSB) coordinates of the piece's bounding-box centre at the map origin, unless marked "own frame".
Network test m10_23 coordinates = retail + (-336, 11, 363).

### Lost Bastille (m10_16): an earlier Bastille with character labels

*[image not published: Lost Bastille m0000]*

`m0000` (not in the MSB) is a whole castle: two polygonal walled enclosures with round towers on a cliff plateau,
a dense keep, a big flat ground plane, 114,447 triangles. Textures are `m0000_wall/wall03/wall04/wood/metal`,
`m0000_water_kari` (*kari* = temporary), `m10_16_ground00/01`, `m10_16_cliff00` and a Drangleic door `m10_21_door00`.
At the map origin it lies over much of the retail Bastille's footprint (centre (-140, 1, 658); 43% of its surface
within 1 m of placed geometry). So it's an earlier, much wider layout of the same site. The retail Bastille is a tall, compact keep
(middle panel).

It carries six 3D-text labels in the same style as the developer signs, but inside the map model itself. Map
coordinates are the label centres:

*[image not published: labels]*

| Label | Literal | Map coords | Thoughts |
|---|---|---|---|
| 防具鍛冶屋 | armour smith | -81.8, 1.3, 590.6 | retail's Lost Bastille has a blacksmith, Steady Hand McDuff |
| 追跡する戦士 | warrior who pursues | -184.0, -10.6, 627.0 | the Pursuer is 追跡者 in Japanese; this looks like its working description |
| 妹を探す兄 | older brother searching for his younger sister | -179.1, 3.1, 536.4 | the reverse of Lucatiel, who searches for her brother |
| 少女 ハンシ 仕込み (3 copies) | girl / hanshi / set-up | -153.6, -5.8, 616.6; -89.3, 10.3, 588.5; -113.0, 30.5, 603.2 | unclear. ハンシ could be 半死 "half-dead" and 仕込み "planted, rigged", i.e. a staged dying girl? A guess |

There's a gap after ハンシ where a glyph may be missing. The first copy has two flat blocks under it. The same model
also holds about 20 upright flat cut-outs, 0.9 m wide and 1 m tall (see "Blockouts and markers").

The rest of the Bastille's 150 pieces are mostly the parts it's built from:
- castle-wall segments and variants (`m0100`-`m0166`), round tower rings (`m0190`-`m0196`), wall panels with doors and
  windows (`m0211`-`m0226`), rib-vault bays (`m0300`-`m0355`), stair towers and landings (`m0373`-`m0422`), roofs
  (`m0460`-`m0470`), timber scaffolding (`m0490`-`m0539`), and a wood-shored mine-tunnel kit (`m0630`-`m0644`). These
  are variants of placed pieces with the same numbers (likeness 60-98%).
- blockouts: plain boxes (`m0610`, `m0615`), a 430-tri walkway outline (`m0290`), a 77 m corridor outline (`m0590`),
  checker panels (`m1201`), slabs (`m1400`), a box room with a door hole (`m0951`); 41 of the 150 are flagged.
- tableware (`m0700`-`m0706`), a seated skeleton (`m0730`), a 7 m bell (`m0755`), a spinning wheel (`m0920`), an iron
  birdcage (`m2020`).
- an unused moon sky: a 3.9 km dome `m0990` and a 230 m moon with a halo `m0991`.
- `m1000` is an empty model.
- `m9999`: **a 1.7 m T-posed hooded, armoured human figure**, textured `m10_16_ground01`. It looks like a scale
  reference.

*[image not published: m9999]* *[image not published: moon]*

#### The crab tower inside `m0000` (2026-10-22)

The author spotted a creature that looks like a crustacean guarding a gateway in the early Bastille. It is part of `m0000`:
a 4,600-triangle round tower with a creature at its foot, all on its own material `表面` ("surface", texture
`dummy128`). No other map model in retail or the network test uses that material name, so it was probably modelled
separately and dropped into the layout.

- Tower: 9.8 m across, 15 m tall, battlements, an outside stair and a ladder doorway, on the curtain wall at model
  (-132, -3..12, 81) = map (-210, 1..16, 643). An arched gateway runs through its base on the +z side.
- Creature: 12.6 x 7.1 x 11.3 m, crouched in that gateway, with four tall segmented legs on each side, a small head
  and long whip-like feelers sweeping forward.

*[image not published: crab tower]*

What else the files say:
- No 3D-text label (the nearest is 追跡する戦士, 30 m away). In retail the spot is open sea past the keep: no enemy
  spawn within 58 m, no named region within 72 m (遠方の海2 "distant sea 2").
- **The network test names a crab boss.** NT and JP NT `ItemName` 60733000 = 宿り大蟹のソウル, "soul of the dwelling
  giant crab", in the boss-soul block 60730000-60755000, between 蜘蛛の女王 "spider queen" and 幼生ナーガ "larval
  naga". It has no English name, and its short description is 仮テキスト "placeholder text". 宿り is the 宿 of
  ヤドカリ (宿借り, hermit crab, "borrowed lodging"), so a giant hermit crab, which fits a creature wrapped round a
  tower like a shell. That's a reading: nothing in the data ties the soul to this model. Retail has no 6073xxxx items.
- Every file of all three builds was searched (as for Agdayne, UTF-16 / Shift-JIS / UTF-8 / ASCII) for 蟹, カニ,
  ヤドカリ, ザリガニ, 甲殻, 門番, crab and similar. Apart from the soul, the only crustacean text is retail's two Aldia
  weapons, whose descriptions say their origin is unknown: Malformed Shell 2700000 ("a fragment of a giant shell", JP
  "seems to be part of a crustacean's body", a pickup in Aldia's Keep, lot 10155020) and Malformed Claws 3420000
  ("a crustacean pincer", dropped by enemy 123002 / 123022 = model c1230, a 2.2 m beaked humanoid, 2 spawns in
  No-man's Wharf). Retail has no crab enemy.
- The Flexile Sentry's descriptions call it 異形の番人, "contorted sentry", which ships the prison's overflow out to
  sea. It's the only other warden near the Bastille's story, and it isn't a crab.

With `show-m10_16` applied the creature sits in the free-camera gallery at about (352..365, 70..77, 556..567).

**No character model for it (searched 2026-10-22).** Every character model in the three builds was surveyed (retail
166, each network test 18) for size, skeleton bone names and spawns (`GeneratorRegistParam` enemy id // 100 = model).
- Retail models no map spawns: only c0001 (player), c1400 (a 5 m winged woman), c2020 (a small white c2010 variant),
  c2250 (empty), c7015 and c7210 (human NPC bodies). The network test adds only c1560 and c8100 to retail's set, and its
  unspawned models are c0001, c3270 (a dragon) and c7310 (a human). None is a crab.
- The many-legged models all spawn in retail and the AI script labels name them: c2050 毒角虫 "poison horn beetle"
  and c2200 巨大毒角虫・酸 "giant acid horn beetle" (the domed back that looked like a shell is a beetle carapace),
  c5030 マンスコーピオン♀ (Najka) and c5020 マンスコーピオン（男） "male man-scorpion", c6030 the spider queen, c1070 /
  c2030 spiders, c2130 石守 "stone keeper". Object models with leg bones are a 7 m armoured giant (`o10_9200`), a
  moth, birds and a flight of wyverns.
- A second search of every file for 宿り / ヤドリ / ヤドカ / 巨蟹 / 大カニ (and yadokari / hermit) finds only the soul
  name, plus an unrelated bonfire-ascetic message in retail.

So the crab boss never got further than its soul's name and the stand-in mesh in the early Bastille.

### Dark Chasm of Old (m40_03): an earlier, larger cave

*[image not published: Dark Chasm m0000]*

Nearly all of the Dark Chasm's 104 unused pieces use textures named `m10_03_*` (wall, ground, wood, cloth, brick, door,
rock, plant, spider, ray, smoke). Retail has no map m10_03, but the Dark Chasm's *placed* pieces use `m10_03`
textures too. So m10_03 is the Dark Chasm's working map number, in the same way Amana's pieces use `m10_11` (m20_11),
Drangleic's `m10_21` (m20_21) and the Undead Crypt's `m10_24` (m20_24).

`m0000` (150k triangles, 343 x 102 x 198 m) is a whole cave level. Drawn in the same frame as the placed map, its central
block lines up with the retail central chamber (only 13% of its surface within 1 m, so the detail differs), and it
carries long winding tunnels and a large platform that retail doesn't have. Around it are the pieces it seems to be
built from: tunnel and cave segments (`m0010`-`m0390`, several overlapping each other 50-64%), an 817 m cave mass
`m0230`, rocks and stalagmites, wooden posts, ladders and frames, stone steps and arches, brick cylinders.

*[image not published: props]*

Also: a **dragon ribcage and spine** `m0023` (`doragon_mainbone` / `rib` textures), a robed statue `m0910`, a 1.8 m
armoured knight figure `m1300`, two cloth tents `m0920`/`m0921`, a 64 m tree with a root fan `m0002`, a brazier
`m1010`, candles, an iron fire basket, cloth strips, and `m1200`, a 12-triangle board textured **`wordmap_d`** (a
world-map board? the texture wasn't viewed). `m4000` (listed) is a 176-triangle Dummy box enclosing the whole Dark
Chasm (a region or lighting volume). The sky dome `m0990` is the same geometry as Drangleic's unused `m9000` and the
Undead Crypt's `m0990`.

`m1500` is a 455 m cliff wall with a stilted wooden building at (-914, 58, 430), textured **`m10_26_wall01`,
`m10_26_floor00`**. By the pattern above, m10_26 might be m20_26's working number, but that's unconfirmed.

*[image not published: m1500]*

### Iron Keep (m10_19): a cut district

*[image not published: Iron Keep district]*

26 pieces in map coordinates form a district beside the keep, between about (-400..-600, 160..215, 700..1020). Hardly
any of it touches the placed map (0-10% of each piece's surface, 25% for the vault room `m0016`):
- gothic churches and halls with spires and flying buttresses: `m0010` (109 m), `m0019`, `m0020` (110 m tall, three
  spires), `m0036`, `m0037` (94 m long), `m0044`, `m0046` (115 m tall); a timber-framed hall `m0043`; block buildings
  `m0038`, `m0041`, `m0042`.
- `m0049`, a **142 m circular wall ring** with an entrance ramp, at (-596, 157, 786).
- `m0047`, a 90 m tall rough sculpted mass with a pole, inside the ring at (-578, 188, 792): a giant statue or figure
  blockout?
- walkways and galleries (`m0009`, `m0015`, `m0018`, `m0045`), a groin-vault room (`m0016`), winding paths 37-230 m
  long (`m0030`, `m0040`, `m0048`), an 85 m arcade (`m0052`), and a far group round a 50 m column (`m0029`, `m0031`,
  `m0032`).
- `m0000`, a 101-triangle house blockout at (-413, 184, 842) that matches `m0036`'s shape (82%): the blockout and the
  modelled building both survive.
- `m0053`: 590 m of chains running from a 30 m cylinder anchor out away from the keep (bounding-box centre
  (-201, 254, 658)).

*[image not published: buildings]*

Textures are the keep's own (`m0000_*`, `m0002`, outerwall, roof, ornament), so this is Iron Keep content, not a
borrowed set. The rest: older room and corridor pieces over the placed keep (`m0024`, `m0026`, `m0034`,
`m0039`, 23-90%), own-frame corridor clusters (`m0011`-`m0017`), timber roof trusses, a plank walkway, and
`m0750`, **a 1 m caryatid carrying a dish on her head** (`BD_o80_8000`).

*[image not published: caryatid]*

### Majula (m10_04)

Majula's origin is the world origin (0, 0, 0), so "where would it sit" can't separate props from map-coordinate
pieces there.

*[image not published: hall]*

- `m0970`-`m0973`: a **61 m cross-plan stone hall** with tall arched windows, buttresses and a door, in four pieces that
  share one frame (a core and three wall sets). At the origin it would stand in the middle of Majula (11% overlap), so
  it may be a prop modelled at its own origin instead.
- `m0780` / `m0785`: a ribbed-dome rotunda, and the same rotunda with a 60 m sloping corridor to a gatehouse.
- a **modular rock-tunnel kit**: straights, a slope, an L-bend, a wide curve, smaller tubes, chambers with pillars and
  a 96 m junction (`m0710`-`m0716`, `m0720`, `m0750`, `m0760`, `m0770`).
- shafts: `m1420` (47 m with a spiral ramp), `m1512` (98 m deep, round), and `m1400`, a rock basin with a carved
  round disc in it.
- a **cloth tent and awning set** (`m0400`-`m0450`), wells with winch frames (`m1520`-`m1522`), a robed statue (`m1530`),
  barrel, crate and books (`m1100`-`m1210`), a bonfire with its sword as a map piece (`m9999`).
- **`m9000`: a dog**, 0.9 m, textures `m10_04_dog_001` (with `eyeballsSG`).
- terrain: older cliff pieces near the centre (`m0003`, `m0004`, `m0010`), a 57 m ravine path with plants at
  (-204, -31, -483) beyond the placed map (`m1612`, listed), 124 m shoreline strips along the coast (`m1800`, listed,
  100% on placed geometry), a 798 m spread of distant cliff fragments (`m0717`), an 800 m tall light-ray card cross.

*[image not published: rotunda]*
*[image not published: tunnels]*
*[image not published: tents]*
*[image not published: shafts]*
*[image not published: dog]*

### Harvest Valley / Earthen Peak (m10_17)

*[image not published: terrain]*

- `m0004`-`m0018`: **older terrain and room pieces** of the valley in map coordinates, many lying on today's ground
  (`m0005` 99%, `m0017` 91%, `m0016` 88%, `m0004` 76%): an earlier tiling of the same terrain. Among them a ravine
  room with a timber stair (`m0009`), a hollow with a lattice gate (`m0012`) and a round 34 m pit with seven ring holes
  in its floor (`m0015`, at about (-570, 36, 425)).
- `m0019`-`m0024` (three listed): an **older mine-tunnel network** (`ground_mine_00`), 20-60 m worm-like tubes with
  shafts, under the valley at about (-600, 46, 434); also `m0026` to `m0028`. `m0028` carries the label **素材屋
  "materials shop"** at (-507.1, 48.0, 474.6).
- **windmill / gear machinery**: 10-11 m gears (`m0060`, `m0067`), a paddle wheel (`m0072`), 35 m geared shafts
  (`m0500`) and gear scatters (`m0066`, `m0074`). Probably Earthen Peak windmill workings.
- `m0100`: an **80 m round domed building** with a conical spire, a domed annex and a pipe, at (-779, 48, 470), off the
  edge of the placed map (textures `m10_17_ground00`, `m0000_metal`, `m0000_water_kari`, `m99_91_st`). `m0030`: a
  95 x 100 m building-complex roof at (-697, 46, 470), also off the map.
- blockout buildings textured `test_0006` round the windmill base (`m0056`, `m0057`, `m0064`, `m0065`, `m0071`), and
  `m0068`, a block building sitting exactly on the placed `m0069` (100%). **Sign 045** in `developer-signs.md` ("fall
  down this hole and there's a platform below...", at (-541.8, 72.2, 503.7)) sits on the blockout `m0065`, not on
  anything placed.
- also: a 104 m spiral of brick stair blocks round the tower (`m0077`, 89% on placed geometry: an older stair),
  54 m chains (`m0076`), a cross-shaped arcade (`m0213`, 73%), three 56 m trumpet-flower pipes (`m0062`), a hanging
  poison bucket (`m0600`), effect planes.

*[image not published: mines]*
*[image not published: gears]*
*[image not published: domed building]*

### Huntsman's Copse / Undead Purgatory (m10_23) and the network test

The network test has the same map (one origin shift aside), and it already had 107 of these 110 pieces unused:
103 byte-identical, 4 with identical geometry. Retail's four extra names (`m0417`, `m0418`, `m0600`, `m0601`) are all
placed in retail. The test placed three models that retail keeps only in its model table:
- `m0800`, the developer signs (already known);
- `m0031`, a waterfall (`滝`), at retail (-403, 17, 216);
- `m0278`, three 7 m bony, arm-like growths in a round clearing at retail (-415, 41, 255), textured `doragon_arm`,
  29,748 triangles. Possibly a dragon-arm grave or decoration, removed after the test.

*[image not published: m0278]*
*[image not published: gate]*

Retail's unused pieces otherwise: `m0044` + `m0045`, a 55 x 82 m ground slab and a **48 m triple-arched gatehouse**
beside Undead Purgatory at about (-257, 50-61, -160), off the placed geometry; a 32 m rope-plank bridge (`m0027`);
a 122 m oval floor slab with a ramp (`m0068`); Undead Purgatory alternates (vault bays, gates, galleries, ruined
copies: `m0062`-`m0077`, `m0249`-`m0273`); cloth tents using Majula's textures (`m0200`-`m0207`); gravestones, an
iron memorial grave, a pegged stone tablet; a big bird's nest; a king oyster mushroom (`c2270` textures); hanging
cages and a censer; long tree-card strips for distant tree lines; a 102 m two-triangle distant-view card (`enkei`);
a 5 km sea plane; `m0209`, an empty model.

### The Gutter (m10_25) and Grave of Saints (m10_34)

*[image not published: statues]*

Twelve pieces exist in both maps with identical geometry, unused in both, all with `m10_25` textures:
- `m0100`: an **86 m statue of a woman** with spread arms; `m0101`: a **97 m statue of a crowned, robed king**
  (`king_01`-`07`, `BD_c7020` textures). Seen in game 2026-10-21 (showcase patch): the king is **Vendrick before hollowing**, the woman
  looks like the **(false) Nashandra** of Drangleic Castle.
- `m0040`: a 351 m field of about 60 block pillars (`enkei` textures: a distant view); `m0020`: a 120 m round tank or
  tower with a chimney building; `m0102`: a 136 m cylinder tower; `m0103`: a 288 m aqueduct; `m0104`: a 544 m
  roofed hall shell; `m0105`: a 282 m river channel with a round building; and sewer pieces (`m0011`), a brick arch, a
  debris pile and a junk pile (`m0200`-`m0202`). Both maps also have a `m0203`, a corpse on a table.

The Gutter alone has `m0010`, a 452 m river and lock system with a 220 m pillar (`dobuRiver`, sewage river), carrying
the label **崩落 "collapse"**. Grave of Saints alone has `m0000`, a 317 x 270 x 413 m complex next to the map, and
`m0030`, a 377 m sprawling complex with the label **篝火 "bonfire"** (both `m10_25` textures). `developer-signs.md`
already noted that the Gutter's signs don't match the retail Gutter; these unused Gutter-textured complexes may be
part of that earlier plan.

*[image not published: pillars and river]*

### Undead Crypt (m20_24)

The crypt has two frames (see "How it was done"). 25 unused vault bays, arches, gates and corridor pieces line up with
the placed geometry only when put at the central hall's centre (-1021.2, -124.7, -179.5): 50-100% of their surface.
They're **radial segments of the round central hall**, like the placed `m0203`, and alternates for it. Another 22 pieces
modelled away from their own origin (`m0451`, `m0610`-`m0700`, `m0630` a three-way junction, `m0670`) are corridors
and rooms that fit neither frame well (0-35% at either): an older layout, or more hall pieces. Which frame they belong
to isn't clear, so their positions in `pieces.tsv` (at the baked origin) are uncertain.

*[image not published: towers]*

The big own-frame structures are textured `m10_24_*` (the crypt's working number): `m0010`, a 339 m round tower with
radial corridors and stairs; `m1200`, a 353 m drum with radial buttresses and a slot (137k triangles); `m0020`, a domed
tower with a spoked roof; `m1400`, a 202 m stacked structure carrying the label **ボス部屋への昇降機 "lift to the boss
room"** at its own origin (the first kana is missing its voicing mark). Also a 190 m blockout cylinder (`m0016`), a
976 x 1137 m terrain bowl with a hole in the middle (`m0800`), a funnel pit, and `m1020`, **a 5.9 m statue of a woman
with an arm raised**.

*[image not published: statue]*

### Drangleic Castle (m20_21)

*[image not published: props]*

A **72 m domed rotunda** with a ribbed dome and arcades (`m4030`, own frame); an **armoured knight statue with halberd
and shield** using the `c1500` character's textures (`m4000`); **11 m dials** with handles (`m4002`, `m4005`,
texture `Switch_00`) and an **18 m geared turntable** (`m4004`), castle puzzle machinery of some kind; an ornate throne
with lions (`m5900`); three framed paintings; a portcullis gateway; a chandelier; and `m5000`, a 486 m sloping colonnade
bridge with a gatehouse, with `m4500`, a 139 m walkway with round pads (`m10_21` textures).

*[image not published: colonnade]*

### Aldia's Keep (m10_15)

*[image not published: dragon]*

A **dragon skull with a horn** (`m0020`, 10 m, `Dragon_Head_n`), a 20 m dragon arm (`m0021`, `doragon_arm`, like
Huntsman's `m0278`), giant bones and vertebrae lying in the area (`m0006`, `m0024`, `m0025`, at about (-520..-535,
64-72, -250)), soldier corpses impaled on stakes (`m0034`/`m0035`), a weapon rack with sword, axe, spear and shield
(`m0076`), a two-storey building with cages and chains, a round tower with a bridge, gallows-like scaffolds and a
5.5 km forest backdrop bowl (`m10_27_forest00`, `ryu_enkei`).

### No-man's Wharf (m10_18)

*[image not published: ships]*

**Four ships**, all own-frame and unplaced: an 18 m galleon with sails and rigging (`m0300`), a 7 m sailing boat
(`m0301`), a 50 m tall ship (`m0302`) and a 25 m longship with oars (`m0303`). That's alongside the hidden three-master
`m0078` in `hidden-map-parts.md`. Also two versions of one harbour house (`m0093`/`m0094`), a broken hull, an 8 m
Dummy-textured cube (`m0400`, the floor square the PC mod swaps in), and `m0212` / `m0901` / `m0902`. Those three are
cliffs and rock pillars that lie 95-100% on geometry the **Lost Bastille's** layout places (`m0901` = Bastille
`m3901`): the Wharf-side cliffs are drawn by the Bastille map instead.

### Things Betwixt (m10_02)

*[image not published: furniture]*
*[image not published: spire]*

A furniture set (a table with bones and carpets, a bed, a chair, a **rocking chair**, a stool and a bench,
`m0054`-`m0060`); `m0049`, a 120 m rock spire with a doorway at its foot; older terrain (`m0001`, 108 m with a ruined
tower, at (-368, 13, -225); `m0002`/`m0003`, two revisions of one plateau lying ~60% on today's ground; `m0006`,
`m0007`); a 600-triangle scatter textured `m10_02_dummy`; two empty models (`m0016`, `m0017`).

### Shrine of Amana (m20_11)

*[image not published: mannequin]*

28 of the 32 are listed in the MSB. They include root-wrapped tower ruins (`m0201` at (-779, -3, -292), `m0204` at
(-1070, 4, -430)), a 57 m pillar tower and a 26 m round tower, cliffs and a 97 m root mass beyond the map edge
(`m0120`, `m0124`, `m0125`, `m0704`), a 688 m light dome, scatters of placeholder markers textured `m10_11_dummy04`
(`m0000`, `m0006`), upright boards over 450 m (`m0001`), island ruins that are older versions of placed ones (`m0080`,
`m0081`), and `m0060`, **a 0.5 m seated mannequin**.

### Smaller maps

- **Heide's Tower (m10_31):** `m1043`, a flat 123 m distant-view card (32 triangles) textured **`enkei_prison`** (a
  far view of a prison, presumably the Lost Bastille) at (-385, 109, 922); `m0900`, a 95 m pole on a rocky mound with a bulbous head at
  (-73, 34, 111); an older 60 m plaza floor (`m0020`, 73%); a candelabra.
- **Doors of Pharros (m10_33):** three house clusters lying 96-97% on the placed `m0451`-`m0453` (older versions), a
  house facade, and a 300 m, 83-triangle blockout box.
- **The giant memories (m20_10):** a 39 m rock arch, floor slabs, a vault, and two Forest of Fallen Giants pieces
  (`m0118` identical to the Forest's placed one).
- **Forest of Fallen Giants (m10_10):** one listed tree-card scatter, a duplicate of the placed `m0312`.
- **Dragon Aerie (m10_27):** columns, a pedestal, rocks. **m10_30:** a pedestal. **Shaded Woods (m10_32):** a dead tree.
  **m20_26:** a rock identical to the placed `m0023`, a claw-like root spire, rocks.

*[image not published: m0900]* *[image not published: m1043]*

## Text and labels

`detect.py`'s flat-glyph test, tightened to flat pieces of 10 or more triangles, puts the 12 developer-sign models at
the top (270-2,700 such pieces each; only Iron Keep `m0020`'s window tracery, 760, ranks among them). `labels.py`
drew face-on the clusters of the other 92 unused models with 3 or more such pieces.
Besides the four known Dark Chasm prop labels (`m1460`, `m1480`, `m1481` came up; `m1470` has too few pieces), it found:

| Map | Model | Text | Where |
|---|---|---|---|
| m10_16 | `m0000` | 少女 ハンシ 仕込み (x3), 防具鍛冶屋, 追跡する戦士, 妹を探す兄 | see the Bastille table |
| m20_24 | `m1400` | ボス部屋への昇降機 "lift to the boss room" | the model's own origin |
| m10_25 | `m0010` | 崩落 "collapse" | model (-195.9, 28.7, 29.5), map about (-7.9, -122.3, -10.5) |
| m10_34 | `m0030` | 篝火 "bonfire" | model (-59.6, -30.4, 24.1), own frame |
| m10_17 | `m0028` | 素材屋 "materials shop" | (-507.1, 48.0, 474.6) |

Each is a single board or label inside an otherwise ordinary model, easy to miss in a render of the whole model. The
other clusters were ornament: window tracery, rosettes, wheel spokes, rigging, chain links. Models with fewer than 3
such glyph pieces weren't drawn, so a very short label (one or two simple characters) could still be missed.

## Patterns

- **Working map numbers in texture names:** `m10_03` = Dark Chasm (m40_03), `m10_11` = Amana (m20_11), `m10_21` =
  Drangleic (m20_21), `m10_24` = Undead Crypt (m20_24; only its unused pieces use it), and `m10_26` once in the Dark
  Chasm. Shared test-map textures `m99_91` and `m99_94` turn up in a few pieces in several maps, placed and unplaced.
- **Prefixed placeholder textures** (`Dummy_K`, `test_0006`, `*_kari`, `*_dummy*`) appear on 68 unused pieces, but
  also on many placed ones, so they're no sign of "unused" by themselves.
- **Shared pieces:** 25 unused pieces have exact geometry that is placed somewhere (often another map: Wharf cages =
  Aldia's Keep's, the Dark Chasm twisted pillar `m2501` is listed in the Bastille, Iron Keep and the Gutter). The Gutter
  and Grave of Saints share twelve unused pieces.
- **Kits:** most "prop" pieces are members of modular kits next to the placed members (walls, vaults, stairs, tunnels,
  trees, rocks, grass cards). The unused members are spare variants, often numbered between the placed ones.
- **Characters as map pieces:** the dog, the T-posed figure, the mannequin, the knight figures and several statues use
  character or object textures (`c1500`, `c2270`, `c7020`, `o80_*`), i.e. they were built as static map geometry.
- **Four empty models** (no triangles): Things Betwixt `m0016` / `m0017`, Bastille `m1000`, Huntsman's `m0209`.

### Blockouts and markers

*[image not published: cut-outs]*

118 pieces are flagged as blockout (placeholder texture names, or under 200 triangles while over 8 m). The clearest are
plain boxes and outlines in the Bastille, the `test_0006` buildings in Harvest Valley, the 82%-matching house blockout
in Iron Keep, the Amana marker scatters and the Dark Chasm volume box.

A recurring oddity: upright flat cut-outs about 0.9 m wide and 1 m tall, standing on floors, with a rounded top and a
small side handle, some with a box outline and two small diamonds. There are about 20 in the early Bastille, 25 in
Undead Crypt `m1400`, and more in Iron Keep `m0004`, `m0024`, `m0034`, `m0041` and Grave of Saints `m0000`. They look
like stand-in markers for some object, chest-front or gravestone outlines perhaps. Not identified.

## The developer signs sit on the retail layouts

With every unused piece and the placed geometry in one frame, each developer-sign board was tested for what lies
within 4 m of it. In all 12 maps the boards sit on placed geometry: Lost Bastille 56/56 boards, Harvest Valley 66/67,
Wharf 31/31, Iron Keep 30/30, Huntsman's 55/55, Gutter 17/17, Heide's 20/20, Grave of Saints 12/12, Amana 54/54,
Drangleic 46/48 (the other two 4-5 m off), Undead Crypt 11/11, Dark Chasm 13/13. The exception is Harvest Valley sign
045, which sits on the unused blockout `m0065`. So the signs annotate (roughly) the shipped layouts, not the cut
material here. The Undead Crypt result needs the (-1115, -157, -196) origin. At the hall centre the crypt signs float
10-58 m from anything placed.

## Can they be seen in game?

- **Retail, normal play: none.** No MSB part places them, and the cheat table's "Draw the whole map" can't help: it
  only switches on the draw groups of parts that exist.
- **Network test: two can be seen now.** The test's m10_23 places `m0031` (the waterfall) in 26 draw groups shown by 25
  floor collisions, and `m0278` (the bony growths) in 5 groups shown by 5 floors. Both have their textures in the
  test's archive. Look near NT (-739, 28, 579) and (-751, 52, 618), the bounding-box centres. The signs `m0800` need
  draw-all (group 96), as already known.
- **With a layout edit: any of the 874, in principle.** That is what the developer-signs mod does on PC: it points an
  MSB part's model entry at the unused model. The 125 listed models only need a part; the 749 others also need a model
  entry, or a renamed one as in the mod. Give the part any draw group and use draw-all, or the floors' groups. On
  PS3 / RPCS3 the MSB has to be changed inside the disc image. **Done once: the Undead Crypt statue `m1020` renders,
  textured and lit, at the Undead Ditch bonfire**, and two pieces with missing textures blink (below).
- **Live, without editing files:** conceivable only for the 125 listed models, and only if the map loader loads every
  model in the MSB table. An existing part's model would then have to be retargeted in memory. Unverified, and it
  would need reverse engineering of the part / model structures.

What an edited-in piece would look like depends mostly on its textures (`unused_textures.py`):

| | pieces | all textures in own map's archives | all but its own baked lightmap | lightmap present (of those that use one) |
|---|---|---|---|---|
| placed | 1,905 | 1,771 (93%) | 1,833 (96%) | 1,522 / 1,586 (96%) |
| unused | 874 | 221 (25%) | 428 (49%) | 200 / 600 (33%) |

So most unused pieces have lost textures from the shipped archives, and two thirds of their lightmaps (`m####_gi_00`)
were never baked or were dropped. **In game, a missing texture blinks**: whatever is missing, a diffuse texture or a
lightmap, is drawn as flat white for half a second, then black for half a second, steadily, with no crash or hang (tried
below). A missing lightmap darkens or lightens the whole material, so a piece without one blinks completely; a missing
diffuse texture blinks just its own material, shaded by the lightmap. So the 221 complete pieces look right, and the
other 653 can be shown but blink, wholly or in places. By piece:
- complete: the Undead Crypt's big structures `m0010`, `m1200`, `m1400` and the statue `m1020` (65 of the crypt's 70
  pieces), Drangleic's rotunda `m4030`, the Majula tunnel tubes `m0710`-`m0716` (with lightmaps; the chamber pieces
  `m0720`-`m0770` lack a rock and a brick texture), Aldia's weapon rack.
- nearly complete: Huntsman's gatehouse `m0045` (20/22, with lightmap), the Iron Keep district (about two thirds of
  each building's textures, no lightmaps), the Gutter's woman statue (3/4).
- mostly missing: the early Bastille `m0000` (2 of 16, its ground and cliff textures are gone), the Dark Chasm cave
  `m0000` (14 of 82), the Gutter's king statue (0/27), the Majula dog (0/3), the T-posed figure (0/1).

Some missing textures survive in other maps' archives (581 of the 874 pieces find all their textures *somewhere*), but
a map only has its own textures, and those of any neighbour that's loaded, to hand.

They also have no collision as far as is known (unused collision pieces weren't checked), so they would be for
looking at with the free camera or the Ghost, not for walking on.

### Tried: the Undead Crypt statue in RPCS3 (2026-10-17)

*[image not published: in game]*

**Result:** the unused 5.9 m statue `m20_24` `m1020` stands 3 m west of the Undead Ditch bonfire. In the capture its
textures look right (stone, hair, the robe's folds from its normal maps), it takes the bonfire's warm light on the side facing it
like the room around it, and it faces the bonfire as intended. The map loads normally with no hang or crash, and the
game was running at 60 fps. It has no collision. Captured from the RPCS3 window while the author played (above, with the
player for scale).

**Where the game reads a layout from.** `map:/` files come through `gamedata:/`, which the executable binds to
`title:/GameData.bhd` + `.bdt` on the disc (strings `title:/GameData.bhd`, `gamedata:/`, `map:/map/`). The HDD
install cache is a separate mount (`install:/Install/InstallData.bdt`, mounted as `install_binder:/`) and holds no
`map\` files at all: models, timeact, ezstate, morpheme and sound only. So an MSB edit has to go into the disc's
GameData. Whether a loose file or an InstallData entry would override GameData wasn't tested, and isn't needed.

**The archive format allows a same-size edit.** GameData.bhd is BHF4 format `0x1c` (size, 64-bit offset, name) with
no hashes and no checksums. The BDT entries are packed back to back. Each MSB is an EDGE DCX: 64 KB chunks, each raw
deflate, at 16-byte aligned offsets listed in an `EgdT` table, with 2-14 bytes of slack after each. So a changed chunk
can be recompressed in place as long as it fits its slot. Then the entry, the index and everything after them stay
unchanged. The RPCS3 ISO is decrypted and holds GameData.bdt contiguously, so the entry can be written into the ISO
file directly. A full copy of the ISO as a backup wasn't possible (7.5 GB free on C:, the ISO is 7.9 GB), and
extracting the disc to a folder wasn't either.

**The edit**, as the PC developer-signs mod does it, and in the same slot the mod uses in this map:
- model entry 48 `m1045` -> `m1020` (same length, so nothing moves);
- part `m1045_0000` -> `m1020_0000`, moved from (-978.0, -137.5, -28.8) to (-1056.0, -126.9375, -182.25), with
  rotation 32.0° -> -90° about Y;
- draw groups (+0x30) and the second group mask (+0x50) both set to groups 0-95, as the mod does, so it draws from
  anywhere in the map.

42 bytes of the 221,888-byte MSB change. Chunk 0 recompresses with zlib -9 (11,894 -> 11,888 bytes). Chunk 1
(8,358 bytes, slot 8,368) doesn't fit with zlib: the new floats and masks cost about 25 bytes, and zlib is only 5-7
bytes smaller than Sony's EDGE compressor. Zopfli fits it in 8,179 bytes, and the game's EDGE decompressor took the
zopfli stream without trouble. No zlib setting reproduces Sony's chunks byte for byte (all levels, memory levels,
windows and strategies tried on chunk 0). The side effect: `m1045` (one
flat "Statue_Plane" card, 230 x 163 m, groups 8-9, with a lightmap) is gone from the crypt while the patch is in.
That's the piece the PC mod also gives up.

**Choosing the spot.** Bonfire objects are `o00_0100` parts: one per bonfire in every map. Their entity ids
(20240650 / 20240651) match `MapObjectInstanceParam` rows naming the bonfire: 24650 Undead Ditch = `o00_0100_0000` at
(-1053.0, -126.9, -182.3), and 24655 Undead Crypt Entrance = `o00_0100_0001`. Heights sampled from the placed map
pieces put Undead Ditch in a room about 8 x 5 m with a 7 m ceiling, and the floor 3 m west within a few cm of the
bonfire's base. The statue's origin is at its feet. Its front is local -z: the face sits ahead of the hair and its
vertex normals point -z. With the survey's rotation convention, a Y rotation of -90° turns it to face +x, the
bonfire. That's what the game shows, so the convention holds for rotations about Y at least.

**Files touched:**
- the disc image `C:\Games\PS3 Emulator\rpcs3-v0.0.39-18764-aeaa62a2_win64_msvc\games\Dark Souls II (USA)
  (En,Fr,Es).iso`, 35,264 bytes at ISO byte `0x15b81576c` (GameData.bdt +`0xd93c9f6c`, entry
  `map\m20_24_00_00\m20_24_00_00.msb`). SHA-1 of the entry: original `b6a31c86...`, patched `a5ec3882...`. Nothing
  else in the ISO changed, and nothing in GameData.bhd, InstallData or RPCS3's folders.
- backups: the original entry in `iso-patches\m20_24_00_00.msb.dcx.original` (`DS2 Vanilla\GameData.bdt` is also an
  untouched copy), every write logged in `iso-patches\log.jsonl`, and the save (`NPUB31358-GAME_000`) copied to
  `saves\20261017_183355_before_crypt_statue` first. The save wasn't changed by the patch.

**Undone the same evening** (18:54, at the author's request): the ISO's Crypt entry is original again, with the same SHA-1
as `DS2 Vanilla\GameData.bdt` and the backup. To put the statue back, stop the game and run (from `tools\devsigns`)
`uv run --no-project --python 3.12 --with zopfli python -I -X utf8 msbpatch.py apply`. `undo` takes it out again: it
writes the original bytes back, checks them against the pristine copy, and reports "original".

**The tool, `tools\devsigns\msbpatch.py`:** `status | plan | apply | undo [preset]`. `plan` is a dry run: it shows the
edits and checks the chunks fit, and `--out` writes the patched MSB for inspection. `apply` refuses to write while
RPCS3's window shows the game running, or if the ISO holds anything other than the original or this patch. New
presets go in `PRESETS`: a map, a single-use model slot to rename (same-length names), the part, a position,
rotation and group masks. A chunk that won't fit even with zopfli is refused, and nothing is written. The tool was
tested end to end (apply, status, undo, byte-identical afterwards) on a sparse copy of the ISO holding only the
directory, index and Crypt sectors, before the real write.

**Not done:** none of the other 873 pieces (but see the next section for missing textures and lightmaps). An edit
that needs a new model entry, rather than a renamed one, would change the MSB's size, so it would need room in GameData
(append and fix the index). Not tried.

### Tried: missing textures and lightmaps (2026-10-17)

*[image not published: Iron Keep hall, the two phases]*

*[image not published: Huntsman's gatehouse, the two phases (brightened)]*

**Result: a missing texture is drawn as a blinking fallback, flat white for 0.5 s, then black for 0.5 s.** It's the
same for a missing diffuse texture and a missing lightmap. Nothing crashes or hangs, both maps load normally, and the
rest of the scene is unaffected. Two pieces were edited in, with the same method as the Crypt statue:

| | Iron Keep `m10_19` `m0043` | Huntsman's Copse `m10_23` `m0045` |
|---|---|---|
| piece | 49 x 25 x 44 m timber-framed hall (cut district) | 48 x 27 x 16 m triple-arched gatehouse |
| lightmap (`m####_gi_00`) | missing on all 9 materials | present |
| diffuse missing | `m0000_wood_d` (roof, frame: 21% of the surface), `tree_piller_00_d` (+ its normal map; walls: 37%) | `outerwall_04_d` (+ normal `innerwall_06_n`) on material #1447: 20% |
| where | over the lava 12-60 m east of the Threshold Bridge bonfire, floor level with the bridge | over the chasm 13 m east of the bridge north of the Bridge Approach bonfire |
| seen | **the whole hall blinks.** White phase: the wooden frame, roof and timber panels are flat, unshaded white, while the brick walls and stone floor (textures present) show their texture, washed out. Black phase: everything is black, with the brick and floor just visible | **only material #1447 blinks**: the inner faces behind the central arch. White phase: light grey carrying the lightmap's soft shading. Black phase: dark. The rest is textured and lit correctly by its own baked lightmap |

Reading the two together: the fallback stands in for each missing texture separately. A lightmap multiplies every
material, so a missing one makes the whole piece blink, even where the diffuse textures are there. A missing diffuse
texture blinks only its own material, and the lightmap's shading still shows on it in the white phase. In the hall the
wood lacks both, so it is pure white or pure black.

Timing: measured from bursts of window captures at three sampling rates (every 0.10, 0.13 and 0.17 s). Every one gives
half-periods of 0.50-0.52 s, so the blink is a real 1 Hz square wave, not an artefact of the sampling. That's about 15
frames a phase at 30 fps. The author saw it by eye as flashing white and black. An emulator bug is unlikely to produce such a
clean, steady 1 Hz, so it is probably the engine's own missing-texture marker. The executable names per-slot fallback
textures `t_dummy_d`, `t_dummy_s`, `t_dummy_n` and `t_dummy_0` (UTF-16 strings at `0x17ef339`-`0x17ef381`, next to
`dummy128` and `g_GITexture`). No archive ships them (only a few character FLVERs mention them), so they are presumably
made in code. Real PS3 hardware wasn't tried. **Traced 2026-10-22** (`missing-texture-blink.md`): the engine's
placeholder is a flip-book of its built-in white and black 1x1 textures with a 0.5 s period; an RPCS3 patch
(`rpcs3-patches\imported_patch.yml`) or `tools\vwhite.py on` keeps it white.

What this means for showing them: the 221 complete pieces look right. The 207 that lack only their lightmap blink as a
whole: in the white phase they show their full textures, flat-lit. The other 446 lack diffuse textures too, and those
parts show no texture detail at all. Any of them can be looked at, and a capture timed to the white phase shows the
textures that do exist. Untested: a missing normal or specular map on its own (no piece here had only that missing).

**Presets** (`msbpatch.py`; both undone at 19:34 the same evening, so the ISO is fully original again, and `apply <preset>` puts either back):
- `ironkeep-hall`: model entry 46 `m0252` -> `m0043` (slot: a 6 x 9 x 8 m prop 240 m from the bonfire, which is gone
  while the patch is in), part position (-582.0, 149.0, 318.0), rotation 0, groups 0-95. Chunk 0 recompresses with
  zopfli (11,980 -> 10,409 bytes, room 11,984). Entry SHA-1 original `cbaabfb8...`, patched `a7c17236...`.
- `copse-gatehouse`: model entry 121 `m0601` -> `m0045` (slot: a 10 x 12 x 12 m rock with plants, groups 17-18, 357 m
  from the bonfire), part position (-117.0, -30.0, 161.75), rotation unchanged (-0, 0, -0), groups 0-95. Chunks 0 and 3
  recompress with zlib -9. Entry SHA-1 original `29c979e1...`, patched `59afec2e...`.
- Choosing them: the bonfire parts (`o00_0100_000n`) carry their entity id in the part's last word (Iron Keep
  10190650-10190665, Huntsman's 10230650-10230665). `MapObjectInstanceParam_<map>` row = entity id, its first field =
  the `BonfireName.fmg` id. Iron Keep: `_0000` Ironhearth Hall, `_0001` Threshold Bridge (at the map origin), `_0002`
  Eygil's Idol, `_0003` Belfry Sol Approach. Huntsman's: `_0001` Undead Refuge, `_0002` Bridge Approach, `_0003` Undead
  Lockaway, `_0004` Undead Purgatory (in a walled colonnade off the Chariot's round arena, so a poor viewpoint). Spots
  were checked with top-down height maps and rough perspective renders of the placed pieces.

**Files touched:** the ISO's `m10_19_00_00.msb` entry (29,840 bytes at ISO byte `0x12884c079`) and `m10_23_00_00.msb`
entry (88,448 bytes at `0x119a346b4`), each written and read back. The originals are in `iso-patches\` (with the log),
and the save was backed up to `saves\20261017_190157_before_ironkeep_hall` first.

**To undo**, stop the game, then from `tools\devsigns`:
`uv run --no-project --python 3.12 --with zopfli python -I -X utf8 msbpatch.py undo ironkeep-hall`, and the same with
`copse-gatehouse`. Each reports "original restored", and `status <preset>` then says "original".

## Caveats

- "Unused" means no MSB part of that map uses the model. Nothing else that could place map geometry (events, other
  maps' MSBs) was checked, except that the Wharf cliffs turned out to be placed by the Lost Bastille's MSB under other
  names.
- Overlap is a 1 m surface-cell test, so a piece can score high just by lying on terrain (small props at Majula's
  origin, scatters). Read it together with the pictures.
- Placed parts use an assumed rotation order (above), and map names for m10_30, m20_10 and m20_26 weren't checked.
- The cut-out "markers", the Iron Keep sculpted mass and the Bastille's 少女 ハンシ 仕込み label are not identified;
  the readings offered are guesses.

## Files

- `unused-map-pieces\pieces.tsv`: all 874 pieces (map, model, MSB status, automatic category, size, triangles, map
  centre or "own frame", overlap, best placed match, most alike model, same-geometry matches, flags incl. text, working
  numbers and blockout hints, materials, hand note). `notes.tsv` holds the hand notes.
- `unused-map-pieces\sheets\<map>_pNN.png`: the contact sheets, 7 pieces a page in model order. Each row: top, front,
  oblique (coloured by material), the piece in red over the placed map at the origin (zoomed and whole map), and an
  overlay (red = this piece, blue = the closest model).
- `unused-map-pieces\overview\<map>.png`: placed geometry with every map-coordinate unused piece outlined and named.
- `unused-map-pieces\*.png`: the larger renders above, `labels.png` (all lettering found), `cutouts.png`;
  `ingame_m20_24_m1020.jpg` (the in-game test); `ingame_m10_19_m0043.jpg` and `ingame_m10_23_m0045.jpg` (the
  missing-texture tests: white and black phase side by side, the second brightened x2.5).
- Tools (`tools\devsigns`): `unused_survey.py <cache dir> [map]` (writes `survey_<DS2_GAME>.pkl`), `unused_render.py
  <pkl> <tiles dir> [map]`, `unused_sheets.py <tiles> <out> [per page] [map]`, `unused_closeup.py <pkl> <map> <out.png>
  <model>[+<model>...] ... [--context] [--placed]`, `unused_table.py <pkl> <out.tsv> [notes.tsv]`, `labels.py <flv>
  <out.png>`, `unused_glyphs.py <dir of models from unplaced.py>`, `unused_signsit.py <pkl>` (the sign check above),
  `unused_textures.py <pkl>` (the texture check), `msbpatch.py status|plan|apply|undo [preset]` (the in-game test's ISO
  patch, run `--with zopfli`). Run with `uv run --no-project --python 3.12 --with numpy --with scipy --with pillow python -I -X utf8
  <script>` and `DS2_GAME=vanilla` (or `nt`). The survey takes about a minute for all maps, the tiles 35 s.
