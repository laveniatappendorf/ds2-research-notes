# Weapons: Network Test vs retail v1.00, and what was cut (2026-10-16)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question: which Network Test weapons are missing from retail, given that display names changed between the builds?
Static work only: `regulation\WeaponParam / ItemParam / WeaponTypeParam` of both builds, both builds' ItemName FMGs,
both archive indexes. Tool: `tools\weapon_compare.py` (summary, `all` for every row, `new` for retail additions).
The Japanese network test (01.00-01.04) has a byte-identical WeaponParam, so everything here holds for it too.

## How weapons are matched without names

Names are no use as a key: the demo only names the 27 weapons it hands out (its ItemName.fmg has 146 strings, and
the second English / Japanese copies in GameData and the other languages hold nothing more). So:

1. **Param id.** Ids are stable between the builds: every id present in both uses the same model in both (bar three
   test / fist rows), so an id in both builds is the same weapon whatever each build calls it.
2. **Model.** WeaponParam +0x04 names the model file (`parts\weapon\WP_<m>_M.bnd`; shields `parts\shield\SD_<m>_M.bnd`,
   their own number space, so model 1220 is both the Longsword and the Silver Eagle Kite Shield). The demo's own test
   rows (8xxx / 9xxx, including the 27 handed-out weapons 8400-8426) are matched to the retail weapon on the same model.
3. **Stats.** A row with neither is compared with the retail-only weapons of the same kind (weight, requirements),
   to catch a weapon that came back under a new id and model.

Retail evidence is then checked for every leftover: retail text under the id, the model file in retail's archives,
and any retail param still pointing at the id.

Field notes: WeaponParam is 176 bytes in the demo and 184 in retail but shares the first 0x40 bytes: +0x04 model,
+0x10 WeaponTypeParam id, +0x18 four u16 stat requirements (str, dex, int, fth), +0x20 f32 weight. WeaponTypeParam
+0x36 indexes weapontype.fmg in both. ItemParam: the WeaponParam id is at +0x18 in the demo, +0x14 in retail.

## Result

The demo has 437 WeaponParam rows: 306 real weapons (ids of 1,000,000 and up), 127 test rows and 4 empty-hand rows.

- **300 of the 306 real weapons are in retail under the same id and model** (267 named retail weapons, 33 unnamed
  placeholder / system rows). Only 58 distinct models are actually in the demo's GameData; the rest of the
  demo's weapons are params without a model file, but they are the same entries as retail's.
- **All test rows but two map onto retail rows** (by model; 8182 / 8183 by id). The two exceptions (8220, 8230) sit
  on the dummy models 9903 / 9904, which retail still ships.
- **6 real weapons are not in retail** (below). None of them came back under another id: no retail row uses their
  models, and where retail added a weapon of the same kind it differs in model, weight and requirements.
- I found no demo weapon that retail kept in its tables but stopped handing out: every named retail weapon that the
  demo also has turns up in a retail item lot, shop, starting-class or NPC param, except Santier's Spear, which I
  did not chase further (probably given by a map script).

### Cut between the Network Test and retail

| demo id | kind | weight, req (str/dex/int/fth) | what retail kept | label in the Scholar cheat table |
|---|---|---|---|---|
| 3510000 | Fists | 0.7, 7/7/0/0 | **Name and description**: "Shadow Claws" / 影の鉤爪, claws used by those who lurk in the shadows, a blade on each finger, wielded by knights who gave up their code for coin. No WeaponParam row, no item, no model | Shadow Claws |
| 3511000 | Fists | 0.7, 7/7/0/0 | nothing; identical stats to 3510000, so a variant of the same claws | - |
| 2450000 | Hammer | 3.6, 18/0/0/0 | nothing (no model in either build) | Floral Club (cut) |
| 2510000 | Hammer | 3.6, 9/0/0/0 | **model WP_2510_M.bnd** (0.72 m long and thin, which fits tongs), no row, no text | "Blackmith's Tongs (cut)" |
| 11095000 | Small Shield | 1.0, 5/8/0/0 | **model SD_1095_M.bnd** (0.48 m, Iron Parma-sized), and **retail params still equip it**: NPCGenerateParam 814000, 814009, 833000 (an NPC with Butcher's Knives; the demo's NPCGenerateParam 813000 has the same loadout) and the debug start loadout DbgPlayerStatusParam 300101000, all pointing at a WeaponParam row that no longer exists. (DbgPlayerStatusParam is a table of developer test characters in the starting-class format, which the retail EBOOT never names; see `armour-rings-spells-nt-vs-retail.md`. The demo's own PlayerStatusParam also has a row 300101000, but it doesn't hold this shield.) Most likely an NPC's shield | - |
| 7000000 | Bare fists | 0, no req | model WP_7000_M.bnd (0.81 m, five meshes and a cloth simulation); retail has a sibling row 7010000 on model 7010. Probably an equipped prop or hand rather than a player weapon | - |

The demo has an item for each of the six, but no name or model file for any of them, so nothing in the demo itself
says what they are. The Scholar table's "Floral Club" and "Blacksmith's Tongs" names are community labels; retail
v1.00 has no text for either.

### Shadow Claws in the Network Test

The demo has Shadow Claws as data only: stats, an upgrade table and a moveset, but no name, no model and no way to
get it.

- **WeaponParam 3510000**: model 3510, WeaponTypeParam 30 (Fists, the Caestus's type), requirements str 7 / dex 7,
  weight 0.7, durability 150. +0x08 / +0x0c point at its own WeaponReinforceParam and WeaponActionCategoryParam rows
  (3510). Apart from those ids and the weapon type, the row is byte-identical to Malformed Claws (3420000), Manikin
  Claws (3430000) and Work Hook (3440000), which use type 20 (Claw). So it is a copy of the claw template, not a
  separately balanced weapon.
- **WeaponReinforceParam 3510** (upgrade table): identical to 3420, 3430 and 3440 (and test rows 9010 / 9011).
- **WeaponActionCategoryParam 3510** (moveset): identical to the claw rows 3410, 3420, 3430, 3440 and the claw
  placeholder 6440, so it would attack like the claws.
- **ItemParam 3510000**: an item pointing at the weapon, with the same values as the demo's other weapon items.
- **3511000** is the same set again on model 3511 (WeaponParam, ItemParam, WeaponReinforceParam 3511 and
  WeaponActionCategoryParam 3511, each identical to its 3510 twin).
- **Missing:** no name or description in any language; no `WP_3510` / `WP_3511` model (the demo ships only 58
  weapon and shield models, and none of the claws or the Caestus are among them); no item lot, shop, NPC or enemy loadout points
  at either id. The Japanese test has the same rows. **Correction 2026-11-09:** no PS3 build has a `WP_3510` /
  `WP_3511` model, but every vanilla PC build does, as unnamed files (`model\parts\weapon\WP_3510_M.bnd` and
  `WP_3511_M.bnd`, found by path hash; `nt-only-equipment-models.md`). Rendered: 3510 is five loose finger blades
  and a thumb spike whose material is named after enemy c1230 (異形の影, the "shadow" the claws' text names); 3511 is a
  full armoured claw gauntlet. In the same render set, `WP_7000` turns out to be a peacock-feather fan, a held prop.

The rest of the "Shadow" kit is in the demo too, also unnamed and without models: the Shadow Dagger (WeaponParam
1050000) and the Shadow armour set (ArmorParam 12230100-12230103, items 22230100-22230103). Retail kept the dagger and
the armour (named, with models: `WP_1050`, `BD_2230`). For the claws it dropped every row (weapon, item, upgrade table,
moveset) but kept the English and Japanese name and description. That suggests the claws got as far as having text
written before they were cut. The text can't be dated, because the demo has text for almost no weapons.

(PhysicsTextureParam0 also has rows 3510 / 3511 in both builds, but it is a dense table of ids 0-4564, so that is
a coincidence.) Untested: the demo lets you equip a weapon row that has no model and attacks with it invisibly (see
the Bandit's Knife in `..\..\DS2 Network Test\Ghidra Project\export\win\s16\findings.md`), so 3510000 could be
equipped live to see the moveset.

### Can the cut weapons be used?

**In the demo, yes, but not as finished weapons** (from the data; none of the six has been tried live yet). All six
still have a WeaponParam row, an item, an upgrade row and a moveset row, so the cheat table (type the id into a hand
slot) or `tools\weapon_swap.py` can equip them, and "Make permanent" / `give` accepts them because each has an item.
None has a model file in the demo, so each would be invisible in the hand, like the Bandit's Knife.

| demo id | moveset row is identical to | so it would |
|---|---|---|
| 3510000, 3511000 | the claws (3410-3440) | attack as claws |
| 2450000, 2510000 | the generic hammers (2400-2440: Club, Mace ...) | attack as a hammer |
| 11095000 | the small shields (11000, 11005, the demo's Target / Small Leather Shield rows) | block as a small shield |
| 7000000 | bare fists (3400, 6430) | punch |

An untested idea for making one visible: point the loaded WeaponParam row's +0x04 at a model the demo ships (for
3510000, the grey placeholder claw 6440).

**In retail, no**, not without a mod: none of the six has a WeaponParam or ItemParam row. What survives is the
Shadow Claws' text and the models `WP_2510`, `SD_1095` and `WP_7000`, which a mod that adds the rows back could
use. Retail's NPCGenerateParam 814000 / 814009 / 833000 still give 11095000 to an NPC dressed as a prisoner
(Prisoner's Hood and Tatters, Butcher's Knives). With no row behind the id the shield can almost certainly not
appear, and I have not checked whether that NPC spawns anywhere.

**Shield 11095000, update 2026-11-09.**

- **Who carries it:** the pre-release PC params' developer names say who. 814000 / 814009 are
  マルチ用NPC：肉断ち（女） "multiplayer NPC: butcher (female)", white and black phantom. That is **Melinda the
  Butcher**: her summon sign at Dragon Aerie (814000) and her invasion in The Gutter (814009), both spawned in retail
  (`..\..\DS2 Scholar PC\findings\npc-invaders-pc.md`).
- **Where it sits in her loadout:** it is her right-hand slot 1, with a Butcher's Knife in right slot 2 and both left
  slots. With no WeaponParam row, that slot can't show the shield; what she holds there in play wasn't checked.
- **A third row:** 833000 is アマナの祭壇：人食い番長 ("Shrine of Amana: cannibal boss"), the same loadout, with a
  generator (3100) in the Shrine of Amana map. It isn't one of the spirit spawns in the NPC-invader table, and
  whether it ever spawns wasn't checked.
- **The Network Test** gave the same loadout to its NPCGenerateParam 813000; retail's 813000 is Felicia the Brave.
- **The model,** `SD_1095_M` (render: `unused-shield-1095.png`, tool `tools\unused_models\shieldview.py`): a round
  shield of wooden planks, 0.47 m across, one face plain and the other braced with an H-shaped batten, much like a
  barrel lid. That suits a butcher dressed in prisoner's rags.
- **Its icon** `IC_0011095000` is in every retail build (the community wiki's "Unknown Wooden Shield").
- **No name anywhere.** No build has text for it. Our earlier label "Shadow shield" (`unused-models.md` and the
  Scholar notes) had no source and has been dropped.

### Renamed (the 27 weapons the demo names)

Matched by model to the retail weapon. 19 kept their names; 8 changed in English and 6 in Japanese:

| demo id | model | demo name | retail name | demo JP | retail JP |
|---|---|---|---|---|---|
| 8409 | 4060 | Dragonknight's Bell | Archdrake Chime | 竜騎士団の聖鈴 | 古竜院の聖鈴 |
| 8410 | 4080 | Bell of the Idol | Idol's Chime | 偶像の聖鈴 | (same) |
| 8416 | 1250 (shield) | Dragonknight's Shield | Archdrake Shield | 竜騎士団の盾 | 古竜院の盾 |
| 8418 | 1620 (shield) | Imperial Shield | Pate's Shield | 帝国兵の盾 | ペイトの盾 |
| 8419 | 4110 | Disc Bell | Disc Chime | 円の聖鈴 | (same) |
| 8420 | 5270 | Pursuer's Greatsword | Pursuer's Ultra Greatsword | 呪縛者の大剣 | 呪縛者の特大剣 |
| 8425 | 1910 | Knight's Greatsword | Mirrah Greatsword | 騎士団の大剣 | 正統騎士団の大剣 |
| 8426 | 1770 | Inquisitor's Blade | Berserker Blade | 探索者の刀剣 | 狂戦士の刀剣 |

The bell-to-chime changes are English only (聖鈴 stays). The Japanese changes are real renames: the
"Dragonknight order" became the Archdrake order (古竜院), the "imperial soldier's" shield became Pate's, the
"knight order's" greatsword became the *orthodox* knight order's (Mirrah), and the "seeker's" blade became the
berserker's.

Weapon kinds were renamed and re-sorted as well: Sacred Bell -> Sacred Chime, Twin Blades -> Twinblade,
Great Arrow -> Greatarrow. The scythes (3000000-3070000) and the Silverblack Sickle (2100000) moved from Halberd /
Axe to Reaper, the Heide lances and Grand / Chariot Lance (2900000-2940000) from Spear to Lance, and the Drakekeeper's
Warpick (3350000) from Halberd to Great Hammer. The demo already had Lance and Reaper weapon types, but only its test
rows used them (9070 on the Grand Lance model, 9110 / 9111 on the Great Scythe model), and its weapontype.fmg has no
names for them.

### Placeholder weapons (both builds)

Rows 6300000-6550000 and 12000000-12020000 are generic stand-ins, one per weapon class, with no item and no name.
They are not the same as the demo's unnamed 8xxx / 9xxx test rows, which use real weapons' models and textures (the
cheat table now labels those "(test row, <weapon> model)").

- **Own model, no colour.** Each placeholder has its own mesh sized for its class, but its binder holds only a
  normal (bump) map. Real weapons ship colour (`_d`), normal (`_n`) and specular (`_s`) textures in both builds. The
  placeholders' materials point at the generic `dummy128.tga` and `spe_040_s.tga` instead, so in game they show
  untextured grey.
- **Coverage.** Every class with real demo weapons has a placeholder except Fists (type 30: Caestus, Shadow Claws).
  There is one for Reaper, a class the demo uses for no real weapon (its scythes are Halberds; retail moved them).
  6440000 is the only claw with a model in the demo, and its moveset row equals the real claws'.
- **Retail v1.00 still ships all 29**, still without an item. The textures are byte-identical. Every `.acb` file
  and 8 meshes differ, and the whip, bow and greatbow gained a `.weppwv` file. The demo's
  binders are stamped 19 Aug 2013, retail's 15-19 Jan 2014.

| id | class | length | | id | class | length |
|---|---|---|---|---|---|---|
| 6300000 | Dagger | 0.51 m | | 6430000 | Bare fists | empty model, no geometry |
| 6310000 | Thrusting Sword | 1.06 m | | 6440000 | Claw | 0.58 m |
| 6320000 | Curved Sword | 1.04 m | | 6450000 | Torch | 0.76 m |
| 6330000 | Katana | 1.40 m | | 6460000 | Binoculars | 0.26 m |
| 6340000 | Straight Sword | 1.12 m | | 6470000 | Whip | 2.24 m, crashes the demo when equipped |
| 6350000 | Greatsword | 1.39 m | | 6480000 | Staff | 1.28 m |
| 6360000 | Hammer | 0.94 m | | 6490000 | Sacred Chime | 0.45 m |
| 6370000 | Great Hammer | 1.92 m | | 6500000 | Bow | 1.14 m |
| 6380000 | Axe | 1.16 m | | 6510000 | Greatbow | 2.69 m |
| 6390000 | Greataxe | 1.43 m | | 6520000 | Crossbow | 0.79 m |
| 6400000 | Spear | 2.31 m | | 6530000 | Curved Greatsword | 1.82 m |
| 6410000 | Reaper | 2.45 m | | 6540000 | Ultra Greatsword | 2.21 m |
| 6420000 | Halberd | 2.74 m | | 6550000 | Twinblade | 2.66 m |
| 12000000 | Small Shield | 0.49 m | | 12010000 | Shield | 0.80 m |
| 12020000 | Greatshield | 0.98 m | | | | |

Lengths are the largest side of the model's bounding box (FLVER header). The whip, bow and greatbow binders also hold
Havok animation files; the real Short Bow (`WP_4200`) has the same `a40_24` pair, so they animate the weapon's own
moving parts (bowstring, whip lash). In-game behaviour is in `..\..\DS2 Network Test\Ghidra Project\export\win\s16\
findings.md` (the placeholders swing with their class's moveset at zero stamina cost).

### New in retail

97 retail rows have no demo row. 73 are NPC rows (7xx000 / 8xx000), nearly all on models the demo already had, and
7 are unnamed fist rows (3403000-3408000, 7010000). The other 17 are new player weapons: Monastery Scimitar,
Key to the Embedded, Dragonslayer Spear and Bat Staff (new models), Old Mirrah Greatsword (1911000, on the Mirrah
Greatsword's model), Pyromancy Flame and Dark Pyromancy Flame (on the bare-fist model 3400), and a themed set of five
weapons with matching shields: Black Flamestone Dagger / Parma, Yellow Quartz Longsword / Shield, Bound Hand Axe /
Wooden Shield, Homunculus Mace / Wooden Shield, Transgressor's Staff / Leather Shield.

## Limits

- "Cut" means in the demo's data but not in retail v1.00. The Scholar edition and the DLC were not checked.
- For the 267 shared named weapons the demo has no text of its own, so any rename among them can't be seen; only
  that the id and model are the same.
- The model descriptions are bounding boxes and mesh counts read from the FLVER headers. The models were not
  rendered.
