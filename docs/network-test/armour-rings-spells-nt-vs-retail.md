# Armour, rings and spells: Network Test vs retail v1.00, and what was cut (2026-10-16)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question: which Network Test armour pieces, rings and spells are missing from retail, given that display names
changed between the builds? This is the follow-on to [weapons: Network Test vs retail](weapons-nt-vs-retail.md), done the same way. It is static work
only: `regulation\ArmorParam / RingParam / SpellParam / BulletParam / ItemParam`, both builds' `SpEffectRing.emevd`,
the ItemName / DetailedExplanation FMGs and both archive indexes.
Tool: `tools\equip_compare.py` (`armour [all|new]`, `rings`, `spells`, `--md`; with no argument, one line per table).
[The Japanese network test's final regulation](jp-network-test-vs-us.md#1-order-of-the-builds) has the same ArmorParam, RingParam, SpellParam and SpEffectRing as the US
demo. Its 01.00 ArmorParam lacks three rows (13081100 / 101 / 103), and its 01.04 ItemParam has 16 more armour items,
so everything below holds for it too.

## Summary

| | Network Test | same id in retail | matched another way | not in retail | new in retail |
|---|---|---|---|---|---|
| armour | 412 real pieces + 77 test rows | **408** (same part and model) | all 77 test rows, by model | **4** | none named (9 unnamed rows) |
| rings | 51 rings | **43** (24 of them under another name) | - | **8** (+ 1 name with no row) | 18 rings + 40 "+1 / +2" variants |
| spells | 69 spells | **0**: every spell was renumbered | 17 by name, 3 by description, 1 by bullet | **48** with no retail counterpart found | 74 |

- **Armour hardly changed.** Retail keeps every demo armour id but four, and adds no new named armour: all 381
  named retail pieces are already in the demo's ArmorParam, on the same model. Retail renamed five sets
  (16 pieces) and made most armour lighter.
- **Rings mostly kept their ids**, but most of the demo's ring names are provisional (仮・) and differ from retail's.
  About half of those still describe the same ring. For about a third, the id seems to have gone to a different ring.
  Eight rings are gone, among them four "starstone" rings, the Rainbow Tearstone and the Rusted Iron Ring. The demo
  has working effects for only four rings. One of them, Ring of War, has exactly the effect of retail's Ring of
  Steel Protection under the same id, so that one is a proven rename.
- **Spells were reworked.** Every spell id changed, along with casts, requirements, catalyst animations, most
  bullets and all the spell effects. Only 21 of the demo's 69 spells can be tied to a retail spell. Of the other 48,
  four are named demo spells with real descriptions (Fire Serpent, Swollen Fireball, Dark Barrier, Dark Burden). The
  remaining 44 are provisional names whose description is the placeholder 仮テキスト.

## How they are matched without names

1. **Param id.** Armour and ring ids are stable between the builds. For armour, every id in both builds has the same
   part and model in both, apart from two system rows. **Spell ids are not stable.** The demo numbers spells
   block, 1, number, variant, 000 (blocks 31-35, variant 1 or 2: 31111000, 31112000, 31121000 ... 31192000);
   retail numbers them block, two-digit number, 0000 (31010000, 31020000 ...). No demo spell id is a retail
   SpellParam row.
2. **Model (armour).** ArmorParam +0x05 part and +0x0a model name the file
   `parts\head|body|arm|leg\HD|BD|AM|LG_<m>_M.bnd`. The demo's test rows 91xxxxxx / 92xxxxxx (items 89xxxxxxx,
   the only armour it names) match the retail piece on the same part and model.
   **Names (rings, spells).** Rings and spells have no model. The demo names almost all of them in Japanese, mostly
   with a provisional 仮・ prefix, so they are matched on the Japanese name without the prefix, or on the English
   name. For rings, the effect script under the id (`SpEffectRing.emevd`; a ring's effect id is its own id) is
   compared too.
3. **Stats / description.** For armour this step found nothing, because no retail-only piece has a name. For spells,
   the demo's own description is compared with retail's. Only its 17 named spells have one; the other 52 say
   仮テキスト. The test is the first three lines (what the spell is and does), ratio >= 0.55 and 0.15 ahead of the
   next best. That gives War = Sacred Oath, Flame Lash = Fire Whip and Bitter Mist = Dark Fog, whose effect lines
   are word for word the same. It also keeps out Dark Barrier, which shares only its lore paragraph with Dark Hail.
4. **Bullets (spells).** BulletParam rows can't be compared by id, because they follow the renumbered spell ids, nor
   by SFX id, because the spell effects were rebuilt (see "Bullets" under Spells). So the spells' bullet rows are
   compared by content: every value that is not an SFX id, a bullet id or a flag byte. A value counts when it occurs
   in the bullets of at most two player spells in each build. A match needs three such values shared, the same school,
   and only one unmatched retail spell that passes. On the 20 pairs already matched, this passes Soul Shower alone,
   and no other pair of the 69 × 95 shares more than two values. It adds one match: 浮遊する火球 = Lingering Flame.

Then each leftover gets the retail evidence: text under the id, the model file in retail's archives, retail params
still holding the id, retail text anywhere that uses its name, and for rings an event in retail's effect script.

Field notes (demo / retail):
- ItemParam (88 / 84 bytes): retail drops the demo's +0x04 word, so every later field moves up 4. Armour id
  +0x1c / +0x18, ring id +0x24 / +0x20, spell id +0x28 / +0x24, item kind byte +0x55 / +0x4f (2-5 head / body /
  arms / legs, 7 ring, 9 spell). Armour item id = armour id + 10,000,000 in both.
- ArmorParam (80 / 84 bytes): the same up to +0x38. Fields: +0x05 part, +0x0a u16 model, +0x34 f32 weight,
  +0x38 f32 durability. Retail inserts a u32 at +0x3c (it scales with the piece: 2000 for Havel's Armor, 46 for the
  Prisoner's Hood), so poise is +0x3c in the demo and +0x40 in retail. Weight, durability and poise are read from
  their ranges (Havel's Armor 19.5 / 255 / 47, Prisoner's Hood 0.8 / 25 / 0), not checked in game.
- RingParam (8 / 16 bytes): +0x00 f32 weight (0.2 on every demo row). Retail adds an f32 at +0x04 (20-340,
  probably durability) and a u32 at +0x08.
- SpellParam (320 / 264 bytes):
  - school: u32 at +0x04 (demo) / +0x00 (retail); 0 sorcery, 1 miracle, 2 pyromancy, 3 hex, and in retail 4 for
    the 35xxxxxx hexes;
  - INT / FTH requirements: u16 at +0x14 / +0x08;
  - eight cast entries: 0x1c bytes each from +0x20 in the demo, 0x14 bytes each from +0x1c in retail. Each entry
    starts with two cast animation ids, and the demo's also hold bullet ids;
  - from +0x134 / +0xf0: one byte, then ten casts-per-attunement-level bytes. The byte is 1 on every demo spell and
    1-4 in retail, so it is probably attunement slots.
  - Bullet ids follow the spell ids (demo 31111000 + n, retail 31010000), so BulletParam can't be compared by id.
    Of the 64 BulletParam ids both builds share, the seven in the spell range (31151000, 31181000, 31191000,
    31211000, 31251000, 32261000, 33171000) each sit 1000 above a retail spell, so they are retail sub-ids. Three of
    them happen to equal demo spell ids.
  - Where the bullets are named: each demo cast entry names one at entry +0x08 (spell id + 0-7, one per entry, all
    eight alike). Retail's row holds the spell id at +0x0c / +0x10 and the main bullet at +0x100 / +0x104 (the spell
    id, or id + 1000 for the weapon buffs and Soul Vortex). A retail spell's other bullets are numbered from its id
    (31130000, 31130001, 31130010).
- BulletParam (292 / 320 bytes): the same up to +0xa8. Retail inserts an f32 at +0xac, so the demo's +0xac onwards
  sits 4 bytes later, and it appends 24 bytes (+0x124 is an f32, 10 on most retail spell bullets). This was worked out
  on the 64 rows both builds share; for example, the demo's +0xc8 (1000) is retail's +0xcc on all 64. Positions on
  retail's grid:
  - SFX ids: +0x00, +0xb4, +0xbc and +0xc0 (the demo's +0x00 / +0xb0 / +0xb8 / +0xbc);
  - child bullets: three pairs at +0xd4-+0xe8;
  - packed bytes: +0x04, +0x2c, +0x44, +0xb0, +0xb8 and +0xec;
  - +0x48 is probably the lifetime in seconds (Soul Arrow 1.25), read from the ranges, not checked in game.

## Armour

The demo's ArmorParam has 492 rows: 412 real pieces (ids 11xxxxxx-17xxxxxx), 77 test rows and 3 system rows.

- **408 of the 412 real pieces are in retail under the same id, part and model**; 381 of them are named there.
  The demo ships model files for only 59 of the 412, the same pattern as weapons: the params list the full game,
  the archive has a demo's worth of models.
- **All 77 test rows map onto retail pieces** by part and model: the 45 named pieces below and 32 unnamed test rows
  (91060100-91320103, 92000000-92000503) on the Elite Knight (1060), Faraam (1320), Archdrake (2310), Black Hood
  (7680) and Hexer's (7700) models.
- **Retail added no named armour.** Its 9 extra rows are unnamed: a dummy body 9800, [a test set 991800-991803](broken-status.md#5-other-developer-names-31-march-2014-pc) on
  the Knight model 1050, and four model-0 rows 11002100-11002103. The two system rows 10 and 14 changed from model 0
  to 1050.
- **Armour got lighter.** 385 of the 408 shared pieces changed weight; retail's weight is a median 0.80 of the demo's.
  207 demo pieces have poise, against 185 in retail; the demo's Lucatiel's Mask and Nahr Alma Hood, for example,
  have 4.8 and retail's have 0.

### Cut between the Network Test and retail

| demo id | part, model | weight / poise | what the demo has | what retail kept |
|---|---|---|---|---|
| 12231100 | head, 2231 | 3.64 / 0 | item 22231100, plus items 22231101-103 for a body, arms and legs that have no rows; stats identical to the Shadow Mask (12230100, model 2230) | nothing: no row, [no model](../unused/nt-only-equipment-models.md) (HD_2231 is in neither build), no text |
| 17521100 | head, 7521 | 3.06 / 4.8 | no item; stats identical to Lucatiel's Mask (17520100, model 7520) | **the model** (`HD_7521_M/F.bnd` and their `_L` versions, 304 KB against 309 KB for HD_7520) and **a new EquipInterfereParam row 17521100**, a copy of Lucatiel's Mask's. The demo's EquipInterfereParam has no such row, so retail added it after the demo while dropping the armour row |
| 17830102 | arms, 7830 | 3.6 / 6.5 | item 27830102; the arms of the Despatcher's set (retail: Nahr Alma) | nothing: retail's set is hood and robes only (17830100 / 101); no AM_7830 model in either build |
| 17830103 | legs, 7830 | 4.48 / 8.1 | item 27830103; the legs of the same set | nothing; no LG_7830 model |

The demo's test copy of that set agrees: items 892002202 / 892002203 point at test rows 92002202 / 92002203, which
don't exist. Their provisional names are 仮・黒ファントム屋・腕 / 足 ("black phantom outfit, arms / legs").

12231100 is the armour counterpart of [the Shadow Claws' twin 3511000](weapons-nt-vs-retail.md#shadow-claws-in-the-network-test): a "1" variant on the next model number, in
the same Shadow kit. [17521100 looks like a second version of Lucatiel's Mask](cut-content-scholar.md#1-models), perhaps the NPC's own; nothing in
retail's params equips it.

### Renamed (the 45 pieces the demo names)

The demo names armour only through its test items (892001000-892002201). Matched by model, 29 kept their names.
16 changed in English and 13 in Japanese, set by set:

| model | demo set (EN / JP) | retail set (EN / JP) | retail ids |
|---|---|---|---|
| 7700 | Chaos Hood / Robes / Gloves / Boots, 混沌の | **Hexer's** Hood / Robes / Gloves / Boots, 闇術の | 17700100-103 |
| 2310 | Dragonknight Helm / Robes / Gloves / Boots, 竜騎士団の | **Archdrake** Helm / Robes / Gloves / Boots, 古竜院の | 12310100-103 |
| 2180 | Hollow Mage Hood / Robes, 亡者魔導士の | **Black** Hollow Mage Hood / Robe, 亡者**魔術師**の | 12180100-101 |
| 7240 | Defender Helm / Mail / Gauntlet / Leggings | **Drangleic** Helm / Mail / Gauntlets / Leggings (Japanese unchanged: ガーディアン) | 17240100-103 |
| 7830 | Despatcher's Hood / Robes, 闇送りの | **Nahr Alma** Hood / Robes, ドロマの | 17830100-101 |
| 7680 | Black Gloves, 黒衣の**長**手袋 | Black Gloves, 黒衣の手袋 (Japanese only) | 17680102 |

Unchanged: Faraam, Black Hood set, Hunter's Hat / Leather, Knight, Rogue, both Prisoner's sets, Llewellyn (body,
arms, legs).

- Dragonknight became Archdrake (竜騎士団 to 古竜院) here as it did for the Dragonknight's Bell and Shield.
  "Chaos" became 闇術, the word the demo uses for its hex school.
- **Nahr Alma started out as the war god.** The demo's Despatcher's Hood text reads "Hood worn by tiny Gren.
  Blessed with the protection of the war god Nahr Alma" (戦神ドロマ). Retail renamed the set after the god and made
  him the "god of blood" (血の神ドロマ). The demo's Crest of Blood likewise says its wearer "belongs to the war god".
  Retail's Japanese DetailedExplanation still has the demo's texts for test items 892002200-892002203 word for word
  (the hood, the robe, and the 仮テキスト placeholders for the arms and legs), but under ids 900008400-900008403.
  Its 900008405-900008426 hold the demo's test-weapon texts 900008400-900008421, so the whole block is five ids out
  in Japanese; the English is not.
- **Straid's homeland was オストロ (Ostro) in the demo's Japanese** (2026-11-17). The demo's Black Hood set texts
  (test items 892001300-892001303) say Straid was the old king's sworn friend in 古き国オストロ ("the old land of
  Ostro"), and its Dispelling Ring and Ring of Resistance texts say 古の亡国オストロ ("the lost land of Ostro"). The
  demo's English says Olaphis in all six. Retail's Japanese mostly uses オラフィス: npcMenu 7680 オラフィスのストレイド,
  Straid's spells, a staff (3890000), and its leftover copy of the demo's set text (now under 892001401-892001500),
  which is the demo text with each オストロ replaced by オラフィス and nothing else changed. Three retail strings kept
  オストロ: the Dispelling Ring and Ring of Resistance descriptions (rewritten for retail, the name left as it was) and
  Straid's line 76801120 ("ここの様を見るに、オストロは…", English "Long enough for Olaphis to rise, fall, and fade
  away, evidently"). PS3 Scholar's Japanese still has all three, plus the new Dispelling Ring+1, so every retail
  build gives the land two Japanese names.
- **Missing pieces, named before they existed.** 84 demo items point at armour ids its ArmorParam doesn't have:
  the missing parts of 32 sets, for example 12180102 / 103 for the arms and legs of the Hollow Mage set. Retail has
  none of these ids as rows and no such orphan items at all, so the sets were finished as they were (hood and robe
  only) and the items removed. The demo names five of them, all test items:

| demo item | part | demo JP | set | retail |
|---|---|---|---|---|
| 892001802 / 803 | arms / legs | 囚人の篭手 / 囚人の腰巻き | second Prisoner's set (2271) | no such pieces |
| 892001902 / 903 | arms / legs | 仮・ダークウィザード・腕 / 足 ("dark wizard, arms / legs") | Hollow Mage (2180) | no such pieces |
| 892002100 | head | [仮・王国軍師・頭 ("royal strategist, head")](llewellyn-references.md#cut-and-leftover) | Llewellyn (7210) | no head (the demo's 17210100 item also has no row) |
| 892002202 / 203 | arms / legs | 仮・黒ファントム屋・腕 / 足 | Despatcher's / Nahr Alma (7830) | cut, see above |

## Rings

The demo has 51 RingParam rows (plus row 0). Its Japanese ItemName names 48 of them, plus three ids that have no
row (40470000-40490000); most of the names have the 仮・ prefix. The English names cover 9 rings. The demo's
descriptions are real for those 9 and 仮テキスト for the rest. Its effect script has events for 8 rings, and as
the twentieth NT session found, four of those are no-ops.

### Cut between the Network Test and retail

| demo id | demo JP (reading) | what retail kept |
|---|---|---|
| 40170000 | 仮・魔法星石の指輪 (Magic Starstone Ring) | nothing |
| 40180000 | 仮・炎星石の指輪 (Flame Starstone Ring) | nothing |
| 40190000 | 仮・雷星石の指輪 (Lightning Starstone Ring) | nothing |
| 40200000 | 仮・闇星石の指輪 (Dark Starstone Ring) | nothing |
| 40240000 | 仮・竜爪の指輪 (Dragon Claw Ring) | nothing |
| 40250000 | 仮・棘の指輪 (Ring of Thorns) | the name: retail's Ring of Thorns 棘の指輪 is a new ring at **40530000** (with +1 / +2) |
| 40270000 | 仮・虹色の涙石の指輪 (Rainbow Tearstone Ring) | nothing; the red and blue tearstone rings stayed |
| 40380000 | 仮・錆びた鉄の指輪 (Rusted Iron Ring, a Dark Souls name) | nothing |
| 40490000 | 仮・大鷲印の指輪 (Great Eagle Crest Ring), name only, no row in either build | nothing |

"Nothing" means: no text under the id in any retail FMG, no event in retail's ring script, no retail param holding
the id, and no retail text using the name (nor, for the longer names, the name without の指輪). The four starstone rings sit
next to the four quartz rings (晶石, same four elements), which retail kept. Retail instead gave the quartz rings +1
and +2 versions, so [the starstones may have been a second tier](../unused/unused-ring-icons.md#what-the-starstones-might-have-done-the-code-2026-10-26) that the upgrade system replaced. That is a guess:
the demo has no text or effect for them.

Icons ([unused ring icons](../unused/unused-ring-icons.md)): no PS3 build and not PC Scholar has an icon file under a cut ring's id, but **every
vanilla PC build does** (found 2026-11-09; the files have no name in any dictionary, so the earlier checks missed
them). All four starstones, the Rainbow Tearstone, Dragon Claw, the demo's Ring of Thorns, Rusted Iron and Great
Eagle Crest have one. They are the source of the community wiki's "unknown" star and rainbow ring icons. Four of them
are pixel-identical to retail icons: Flame Starstone = Old Sun Ring, Dragon Claw = Ring of Giants, the demo's Ring of
Thorns = Old Leo Ring, Rusted Iron = Stone Ring ([hidden PC icons](../unused/hidden-pc-icons.md)). The Network Test's own Ring of War icon became the DLC
Ivory Warrior Ring's.

### Same id, same ring (17)

Royal Soldier's Ring, the Spell / Flame / Thunder / Dark Quartz Rings, Dispelling Ring, Ring of Resistance, Red and
Blue Tearstone Rings, Ring of Giants, Ring of Soul Protection, Ring of Restoration, Binding Ring (English name now
Ring of Binding), Silvercat Ring, Guardian's Seal and Crest of Blood keep the same Japanese name. **Ring of War**
(戦の指輪, 40050000) became the **Ring of Steel Protection** (鉄の加護の指輪), and the effect proves it: the demo's
script ends `90[11](0, 50)`, retail's `100090[04](0, 50)`, the same type-0 +50 bonus under a renumbered command
(type 0 = physical defence, as the twenty-first NT session found). The demo's text already says it "increases
physical defense".

The other comparable effects changed in strength, not kind:
- Dispelling Ring: demo +15 to types 1-4, retail +6.
- Ring of Resistance: demo +20 to types 5-8, [retail +12 to types 5-9](broken-status.md#4-break-resistance-left-in-the-retail-scripts). The demo's text lists poison, bleeding,
  **weapon breakage** and curse; retail's lists poison, bleeding, **petrification** and curses.
- Royal Soldier's Ring, Red Tearstone, Ring of Restoration and Binding Ring are no-ops in the demo's script and
  real effects in retail.

Three demo rows have no name in the demo: 40510000, 40700000 and 40710000. Retail has the King's Ring, Crest of the
Rat and Bell Keeper's Seal under those ids.

### Same id, different name (25)

The demo's provisional names have no effects or text behind them, so the only test is whether the name fits what
the retail ring under that id does. English readings of the provisional names are the seventeenth NT session's
(`RING_EN` in `..\..\DS2 Network Test\tools\weapons.py`).

| id | demo name (reading) | retail ring and what it does | reading |
|---|---|---|---|
| 40100000 | 解毒の指甲 (Antidote Knuckle Ring) | Poisonbite Ring: poison resistance (script type 5) | same ring |
| 40110000 | 止血の指甲 (Staunching Knuckle Ring) | Bloodbite Ring: bleed resistance (type 6) | same ring |
| 40130000 | 阻呪の指甲 (Curse-ward Knuckle Ring) | Cursebite Ring: curse resistance (type 8) | same ring |
| 40120000 | 防護の指甲 (Protective Knuckle Ring) | [Bracing Knuckle Ring 封壊の指甲](break-element.md#update-2026-11-16-the-break-resistance-knuckle-ring): slows equipment wear | same family (指甲 knuckle rings) |
| 40210000 | ストレイドの知恵の指輪 (Straid's Wisdom Ring) | Ring of Knowledge: INT, sold by Straid (below) | same ring |
| 40220000 | ローネスの祈りの指輪 (Rhoness's Prayer Ring) | Ring of Prayer: FTH, sold by Licia of Lindelt (below) | same ring (retail credits Ivory Rhones with the Soul Protection rings) |
| 40160000 | 王の剣の指輪 (King's Sword Ring) | Ring of Blades: physical attack | probably the same |
| 40310000 | 身代りの指輪 (Ring of the Scapegoat) | Ring of Life Protection: no loss on death | same ring |
| 40320000 | 緩やかな刻の指輪 (Ring of Slow Time) | Lingering Dragoncrest Ring: longer spell effects | same ring |
| 40330000 | 速唱の指輪 (Quick-chant Ring) | Clear Bluestone Ring: faster casting | same ring |
| 40340000 | 偉大なる祠祭の指冠 (Great Priest's Band) | Northern Ritual Band 北の祠祭の指冠: more spell uses | same family |
| 40350000 | 祠祭の指冠 (Priest's Band) | Southern Ritual Band 南の祠祭の指冠: more attunement slots | same family |
| 40450000 | 絆の指輪 (Ring of Bonds) | Name-engraved Ring: connects players who chose the same god | probably the same |
| 40040000 | 眠れる龍の指輪 (Sleeping Dragon Ring) | First Dragon Ring: HP, stamina, load | unclear: retail's sleeping-dragon ring is the Slumbering Dragoncrest (静かに眠る竜印, 40460000) |
| 40360000 | 金腕の指輪 (Golden Arm Ring) | Covetous Gold Serpent Ring: item discovery | unclear |
| 40430000 | 魔眼の指輪 (Evil Eye Ring) | Redeye Ring 赤眼: draws enemies | unclear: retail's "evil eye" (邪眼) is the Ring of the Evil Eye at 40390000 |
| 40010000 | 守護者の指輪 (Guardian's Ring) | Life Ring: max HP | different ring |
| 40020000 | 名誉の指輪 (Ring of Honour) | Chloranthy Ring: stamina recovery | different ring |
| 40230000 | 大蜘蛛の指輪 (Great Spider Ring) | Stone Ring: poise damage | different ring |
| 40370000 | 純真の指輪 (Ring of Innocence) | Covetous Silver Serpent Ring: more souls | different ring |
| 40390000 | 黒き精の指輪 (Black Spirit Ring) | Ring of the Evil Eye: HP per kill | different ring |
| 40440000 | 四つ葉の指輪 (Four-leaf Ring) | Gower's Ring of Protection: less damage from behind | different ring |
| 40460000 | 仇の指輪 (Ring of Vengeance) | Slumbering Dragoncrest Ring: silent | different ring |
| 40470000 | 手作りの指輪 (Handmade Ring), name only | Hawk Ring: arrow range | different ring |
| 40480000 | 盾印の指輪 (Shield Crest Ring), name only | Old Sun Ring: stores damage, then explodes | different ring |

So 13 of the 25 are the same ring renamed, 3 are unclear, and 9 ids went to a ring that has nothing in common with
the planned one. Those 9 planned rings (Guardian's, Honour, Great Spider, Innocence, Black Spirit, Four-leaf,
Vengeance, Handmade, Shield Crest) are best counted as cut too, although their ids survive. Some names moved rather
than vanished: Ring of Thorns (40250000 to 40530000), and possibly the sleeping dragon and the evil eye. The names
alone can't settle the unclear ones.

**The two stat rings kept their sellers (2026-11-17).** Retail v1.00 sells the Ring of Knowledge (the demo's
Straid's Wisdom Ring) at Straid of Olaphis himself: ShopLineupParam 76800400, vendor 7680, always in stock. The Ring
of Prayer (the demo's Rhoness's Prayer Ring) is sold by Licia of Lindelt, the miracle vendor (76900400, vendor 7690),
and retail's Ring of Soul Protection text calls Ivory Rhones (白のローネス) a great sage of Lindelt
(リンデルトの大賢者). So each ring is sold by the person its demo name pointed to, or by a cleric of that person's
land. Only the names and texts changed: both retail descriptions now credit the High Priest of Volgen instead. The
demo has no shop rows for either ring (its ShopLineupParam stocks vendors 7540 and 7830 only), but its npcMenu
already has both vendor slots: 7680 スペルマニア ("spell maniac", the label [Network Test AI labels](nt-ai-labels-retail-enemies.md) ties to
Straid) and 7690 奇跡の人 ("the miracle person"). That the rings were planned for these two is a reading of the
names; nothing in the demo's data links them. Prompted by a Japanese blog post of 17 January 2014 (ebiflynageruyo
on livedoor, via the Wayback Machine) that lists the demo's text; its 49 ring names match the demo's ItemName
40010000-40490000 exactly.

### New in retail

18 rings with no demo row: Ash Knuckle Ring (40135000), Old Leo Ring (40295000), Hawk Ring, Old Sun Ring, Ring of
the Dead, Ring of Thorns, Delicate String, White Ring, Ring of Whispers, the Illusory Rings of a Conqueror / the
Exalted / the Vengeful / the Guilty (the last two have no item in v1.00), and the covenant seals Blue Seal, Abyss
Seal, Vanquisher's Seal, Sun Seal and Ancient Dragon Seal. Retail also has 40 upgraded variants: "+1" / "+2" for 19
rings, and the Second / Third Dragon Rings (40040001 / 40040002).

## Spells

### The renumbering

Every spell id changed; see "How they are matched". The demo has five blocks of ids: 31 sorcery (16 spells),
32 miracle (16), 33 pyromancy (16), 34 hex (16) and 35, five hexes stored with the miracle school number.
Retail has 31 sorcery (29), 32 miracle (26), 33 pyromancy (21), 34 hexes (10, school 3) and 35 a second hex group
(9, school 4), plus 98 rows 71xxxxxx-75xxxxxx that repeat the same spells for NPCs.

**Retail still uses the old ids in one place:** the debug start loadouts. These are ready-made test characters.
- **The table.** `regulation\DbgPlayerStatusParam.param` has the same row type (PLAYER_STATUS_PARAM) and size
  (296 bytes) as `PlayerStatusParam`, the table that defines the starting classes. Each row sets what a character
  begins with: level and stats, weapons, armour, items, spells and gestures. Retail's class table has 18 rows, the
  debug table 224 (groups 800, 900, 1000, 1100, 1200, 3000 and 300101000-300245000), and none of them is a class
  a player can pick.
- [**Rows 900-949 (41 rows)**](nt-bosses-not-in-retail.md#boss-weapons-on-the-debug-list-dbgplayerstatusparam) look like maxed-out test characters. Row 900, for example, has:
  - stats at 99;
  - consumables: Lifegems, Estus Flask, herbs, Green Blossom, Witching Urn;
  - weapons: Spider Fang, Falchion, Claymore;
  - the Black Witch Hat;
  - seven gestures;
  - the same ten spells as the other rows, by their **demo** ids: Soul Arrow 31111000, Soul Spear 31122000,
    Soul Shower 31142000, Soul Greatsword 31162000, Replenishment 32132000, War 32141000, Lightning Spear
    32172000, Flame Lash 33121000, Swollen Fireball 33141000 and Dark Orb 34121000.

  None of those ten ids exists in retail's SpellParam.
- **Retail may never load it.** A string search of the retail EBOOT finds "PlayerStatusParam" (UTF-16, at
  0x17fe880) but no "DbgPlayerStatusParam", in plain text or UTF-16. So the table may be developer data left in
  the regulation file. How the game picks which params to load was not traced, so this is not certain.
- **The demo has no DbgPlayerStatusParam.** 29 of the debug table's row ids (3000 and 300101000-300128000) are
  also rows of the demo's own PlayerStatusParam, which has 289 rows. Rows 900-949 are not among them, and the demo's
  rows hold none of the ten spell ids.

So the 900-949 loadouts were written while spells still had the demo's numbering, and nobody updated them when the
spells were renumbered. Whether they were written before or after the demo was built can't be told: the demo may
simply not have shipped the debug table.

### Catalysts changed

The cast animations show which catalyst a school was built for:
- **Sorceries** use the staff animations (38110xxx) in both builds.
- **Demo pyromancies use the same staff animations**; retail pyromancies have their own (38111xxx). The demo has
  no pyromancy flame (it is new in retail; see [the weapon note](weapons-nt-vs-retail.md#new-in-retail)), so in the demo pyromancies were cast with a staff.
- **Demo hexes use the miracle (chime) animations** (39110xxx), all 21 of them. The demo's chime texts call the
  bells catalysts for "miracles and 闇術 (hexes)". Retail splits hexes: the 34xxxxxx ones use the staff animations,
  and the 35xxxxxx ones start with a chime animation.
- The demo's hexer **Galio** (EN) is retail's **Gilleah**; the Japanese is ギリア in both.
- **How spells are cast changed too** (2026-11-17, live): every demo spell has eight cast setups with their own
  projectile and damage row (two-handed and heavy casts hit harder, up to ×2.1), two-handed staves and chimes get a
  bow-style aiming mode, and catalysts have power-stance categories (though they cannot power stance in the demo).
  Retail flattened all of it. See [Network Test cast setups](nt-casting-setups.md).

### Matched (21)

| demo id | demo name | retail id | retail name | demo INT/FTH, casts | retail INT/FTH, casts |
|---|---|---|---|---|---|
| 31111000 | Soul Arrow | 31010000 | Soul Arrow | 10/0, 20 | 10/0, 30 |
| 31112000 | 仮・落下制御 | 31240000 | Fall Control | 0/0, 4 | 15/0, 10 |
| 31121000 | 仮・追尾するソウルの矢 | 31050000 | Homing Soul Arrow | 0/0, 10 | 25/0, 10 |
| 31122000 | Soul Spear | 31090000 | Soul Spear | 12/0, 10 | 40/0, 2 |
| 31142000 | Soul Shower | 31130000 | Soul Shower | 18/0, 4 | 22/0, 4 |
| 31162000 | Soul Greatsword | 31140000 | Soul Greatsword | 20/0, 4 | 27/0, 3 |
| 31181000 | 仮・渦巻くソウルの塊 | 31150000 | Soul Vortex | 0/0, 2 | 37/0, 4 |
| 31192000 | 仮・ソウルの奔流 | 31170000 | Soul Geyser | 0/0, 1 | 64/0, 1 |
| 32111000 | Force | 32100000 | Force | 0/10, 10 | 0/12, 10 |
| 32121000 | Heal | 32010000 | Heal | 0/10, 4 | 0/12, 3 |
| 32132000 | Replenishment | 32060000 | Replenishment | 0/18, 4 | 0/16, 2 |
| 32141000 | **War** 戦争 | 32230000 | **Sacred Oath** 固い誓い (by description; same cast animations too) | 0/13, 2 | 0/25, 3 |
| 32172000 | Lightning Spear | 32140000 | Lightning Spear | 0/20, 2 | 0/22, 10 |
| 33111000 | 仮・浮遊する火球 "floating fireball" | 33140000 | **Lingering Flame** 漂う火球 (by bullet; see "Bullets") | 0/0, 4 | 0/0, 4 |
| 33121000 | **Flame Lash** 炎の波 (by description) | 33100000 | **Fire Whip** 薙ぎ払う炎 | 15/14, 4 | 0/0, 4 |
| 33152000 | Great Combustion | 33090000 | Great Combustion | 20/18, 4 | 0/0, 6 |
| 33172000 | 仮・混沌の嵐 | 33070000 | Chaos Storm | 0/0, 2 | 0/0, 2 |
| 33191000 | 仮・混沌の大火球 | 33040000 | Great Chaos Fireball | 0/0, 1 | 0/0, 2 |
| 34112000 | 仮・死者の活性 | 34050000 | Dead Again | 0/0, 10 | 22/20, 10 |
| 34121000 | Dark Orb | 34010000 | Dark Orb | 10/16, 4 | 12/10, 20 |
| 34152000 | **Bitter Mist** 冷たい霧 (by description) | 34030000 | **Dark Fog** 闇の霧 | 13/15, 4 | 25/14, 4 |

The 52 provisional demo spells have no requirement (0/0). The demo's named pyromancies needed INT and FTH; the
retail pyromancies above need nothing.

### Bullets (BulletParam compared by content)

Bullet ids follow the renumbered spell ids, so the bullets were compared by what is in them. Four findings:

- **SFX ids can't be carried across.** The demo's player-spell effects are f0001300-f00016xx in
  `sfx\sfx9999.ffxbnd`, built from a folder called `OldSfx`. Retail's are f0005000 onwards in `sfx9999.ffxbnd.dcx`
  on the disc (`PS3_GAME\USRDIR\sfx`, read from the disc image, not copied here), built from `Sfx`. Both builds
  use the same DLsE effect format; retail packs each file with zlib. 270 effect files are byte-identical in the two
  builds, all under the same id, but none of them is a spell effect: no demo spell effect id exists in retail, and
  no retail one in the demo. So the spell effects were rebuilt and renumbered, and an SFX id means nothing across
  the builds.
- **Most bullet values changed.** Retail retuned and restructured most spell bullets:
  - the demo's Soul Arrow and Soul Spear fire an empty launcher bullet whose child (id + 100,000) is the
    projectile; retail's are a single bullet;
  - 50 of the 52 retail spell bullets that set +0x104 have the same values from there to +0x124, where the demo
    has several different sets;
  - 19 of the 20 pairs matched by name or description share no rare bullet value at all.

  Two looser scratch tests (not kept in the tool) did no better. Overlap of all values put the true retail spell
  first, or tied first, for 4 of the 20. A distance on the motion values (speeds, lifetimes, angles, counts) did it
  for 3, with a median rank of 34 among the 95 retail spells.
- **Bullets describe a kind of spell more than a spell.** Both builds give several spells the same bullets. In the
  demo:
  - eight miracles share one bullet pair (Heal, Replenishment, War and five provisional ones);
  - the sorcery and miracle reversals, 鎮魂の響き and 断呪の加護 share another;
  - Dark Barrier and two provisional hexes share a third;
  - Dark Burden and three 念 spells share a fourth.

  Retail has 19 such groups, for example Heal, Med Heal, Great Heal Excerpt, Great Heal, Bountiful Sunlight and
  Caressing Prayer. Within a group only the SFX ids tell the spells apart.
- **What survives is the odd rare value.** The test counts only values found in the bullets of at most two player
  spells per build. The true Soul Shower pair shares nine of them: its spread, its sub-bullet angles and its timings
  (+0x14, +0x38-+0x40, +0x50, +0x5c, +0x60, +0x98). Apart from the match below, which shares four, no other pair of
  the 69 × 95 shares more than two. Moving the cut-off to three or four spells per build changes nothing: the same
  two pairs stand out (Soul Shower then shares 12 or 13 values). At one spell per build only Soul Shower passes,
  with six.

**One new match: 仮・浮遊する火球 (33111000, "floating fireball") = Lingering Flame (漂う火球, "drifting fireball",
33140000).**
- **The demo's bullet** lives 20 s, starts at 2 (+0x4c, probably speed) with -1 at +0x54, and has +0xf0 / +0xf4 /
  +0xf8 = 2, 2π, 2π.
- **Retail splits it in two.** A launcher starts at 3 with -3 at +0x54. Its child bullet 33140001 lives 20 s and has
  the same 2, 2π, 2π.
- **That block is unique.** +0xf0 = 2.0 occurs nowhere else in either build's BulletParam: only in the demo spell's
  eight entry bullets (33111000-007) and in retail's 33140001.
- Both spells are pyromancies, and the name was already read as "likely" Lingering Flame. That reading is now a
  bullet match.

That is the only match. The bullets give nothing for the other 48 spells. For the name readings below they point
neither way:
- 乱射するソウルの槍 shares one rare value with Soul Spear Barrage (+0x24 0.0349, a 2° angle);
- five other demo spells share one rare value each with some retail spell;
- all of that is below the bar. Since 19 of the 20 known pairs share none, the lack of bullet evidence counts
  against no reading.

### Not in retail (48)

**Named demo spells with a real description, and no retail counterpart:**
- **Fire Serpent** (火蛇, 33122000): "an elementary pyromancy. Launches a fireball that chases foes", a homing
  fireball. None of retail's 21 pyromancy descriptions mentions homing or chasing.
- **Swollen Fireball** (膨らむ火弾, 33141000): "one of the most ancient pyromancies... hurls an explosive fireball".
  Retail's fireballs open differently. Possibly the forerunner of Fireball / Fire Orb, but nothing ties them.
- **Dark Barrier** (闇の防護, 34111000): an elementary hex, a barrier of darkness that repels pyromancy flames. Its
  lore paragraph (hexes began as sorcery, now forbidden) went to retail's Dark Hail.
- **Dark Burden** (重たい闇, 35111000): "doubles the gravitational force" around the caster, at the cost of the
  caster's maximum HP.

**Provisional (仮・) spells.** These have placeholder text, so only the name speaks. Where a name strongly suggests a
retail spell, it is given as a *possible* counterpart. That is a reading of the name, not a match; the bullets
neither confirm nor rule out any of them (see "Bullets"). 浮遊する火球, read here before as "likely" Lingering Flame,
is now matched by its bullet.

| school | demo spells (reading) | possible retail counterpart |
|---|---|---|
| sorcery | 偽魂 (false soul), 拡散するソウルの矢 (spreading soul arrows), 追尾するソウルの槍 (homing soul spear), 輝くソウルの光束 (shining soul beam), 魔術の反転 (sorcery reversal) | none |
| sorcery | 乱射するソウルの槍 (rapid-fire soul spears) | Soul Spear Barrage 乱れるソウルの槍 (likely) |
| sorcery | 衝撃波 (shockwave) | Shockwave 衝撃 (likely) |
| sorcery | ソウルの光束 (soul beam) | Soul Bolt ソウルの閃光 (possible) |
| miracle | 小さな大地の恵み (small blessing of the earth), 解毒 (cure poison), 清らかな加護 (pure protection), 奇蹟の反転 (miracle reversal), 断呪の加護 (curse-breaking protection), 三女神の護り (protection of the three goddesses), 大いなる聖女の癒し (great healing of the saint), 放電 (discharge), 聖女の祈り (the saint's prayer) | none |
| miracle | 鎮魂の響き (echo of the requiem) | Soul Appease 鎮魂 (likely) |
| miracle | 大いなる神の癒し (great healing of the gods) | Great Heal 大回復 (possible) |
| pyromancy | 炎陣 (flame circle), 呪術の反転 (pyromancy reversal), 炎の飛礫 (flame pebbles), 炎上 / 大炎上 (blaze / great blaze), 地竜の怒り (wrath of the earth dragon) | none |
| pyromancy | 火柱 / 連なる火柱 (fire pillar / chained fire pillars), 炎の大波 (great wave of flame) | Firestorm / Fire Tempest (possible) |
| hex | 毒の沼 (poison swamp), 闇術の反転 (hex reversal), 呪詛 (curse), 大いなる死者の活性 (great Dead Again), 冷たい炎の守り (guard of cold flame), 魂の吸収 (soul absorption), 大怨念 (great grudge), 死霊の召喚 (summon the dead) | none |
| hex | 逃れられぬ怨念 (inescapable grudge) | Affinity 追う者たち (possible) |
| hex | 封術 (sealing art) | Profound Still 深い沈黙 (possible) |
| hex | 闇の刃 (blade of darkness) | Dark Weapon 闇の武器 (possible) |
| hex | 冷たい炎の嵐 (storm of cold flame) | Darkstorm 闇の嵐, "a vortex of dark flame" (possible) |
| hex (35) | 再動の念, 硬化の念, 活性の念, 衝撃の念 (thoughts of re-motion, hardening, activation, impact) | none |

Even if every "likely" and "possible" reading is right, 36 demo spells have no retail counterpart (32 provisional,
4 named). Notable groups:
- **A "reversal" spell for every school** (魔術 / 奇蹟 / 呪術 / 闇術の反転).
- **A small 念 ("thought") group** of four spells in the 35 block, next to Dark Burden.
- Healing and protection miracles tied to **a saint (聖女) and three goddesses**.
- Hexes built on **cold flame (冷たい炎)** and **grudges (怨念)**.

Apart from Swollen Fireball, which the retail debug loadouts still list, none of the 48 left a trace in retail (but see [hidden PC icons](../unused/hidden-pc-icons.md#artwork)):
- no text under the old ids;
- no retail param holding the ids. The only hit, PlayerDamageParam 33171000, is retail's own sub-id of Flame
  Weapon (33170000);
- no retail text using the names. The hits are ordinary words (衝撃波 "shockwave" in Force's text, 解毒 in Poison
  Moss's, 呪詛 in a mask's) and 聖女の祈り, which turns up only in the demo's own chime text that retail carries,
  shifted, under 900008415.

### New in retail (74)

All 74 retail spells not matched above, among them: the Great / Heavy / Crystal soul arrow family, Crystal Soul
Spear, the weapon buffs (Magic / Great Magic / Crystal Magic / Flame / Dark / Resonant Weapon, Sunlight Blade), the
utility sorceries (Yearn, Hush, Hidden Weapon, Repair, Cast Light, Chameleon, Unleash Magic), the heal ladder (Med
Heal, Great Heal Excerpt, Great Heal, Soothing / Bountiful Sunlight, Resplendent Life, Caressing Prayer), Wrath of
the Gods, Emit Force, Heavenly Thunder, Great Lightning Spear, Sunlight Spear, Blinding Bolt, the Magic Barriers,
Homeward, Guidance, Unveil, Perseverance, most pyromancies (Fireball, Fire Orb, Great Fireball, Firestorm, Fire
Tempest, Combustion, Poison / Toxic Mist, Acid Surge, Flame Swathe, Forbidden Sun, Flash Sweat,
Iron Flesh, Warmth, Immolation) and most hexes (Dark Hail, Affinity, Whisper of Despair, Repel, Twisted Barricade,
Numbness, and the whole 35 block: Scraps of Life, Darkstorm, Resonant Soul / Great Resonant Soul, Climax, Resonant
Flesh, Lifedrain Patch, Profound Still). Some of these may be the provisional spells above under their final
names; the "possible" readings name the candidates. `equip_compare.py spells` lists them.

## Can the cut armour and rings be used? (2026-10-16)

From the data only; none of these has been tried live yet. Weapons are covered in [weapons: Network Test vs retail](weapons-nt-vs-retail.md).

**In the demo:** every cut piece and ring still has its param row, so the cheat table (Hands and armour / ring
slots, or `tools\weapon_swap.py`) can equip it.

| demo id | equip | keep after a load | what to expect |
|---|---|---|---|
| 12231100 (Shadow head variant), 17830102 / 17830103 (Despatcher's arms / legs) | yes | yes (each has an item, so "Make permanent" / `give` accepts it) | no model file in the demo. Armour without a model has never been equipped, so what it shows, and whether it is safe, is unknown: back up the save first |
| 17521100 (second Lucatiel's Mask) | yes | no: no item, so a live swap lasts until the next load | the same unknown; `HD_7521` exists only in retail |
| the 8 cut rings (40170000-40200000, 40240000, 40250000, 40270000, 40380000) | yes, they are in the ring dropdown | yes | 0.2 weight and nothing else: the demo's SpEffectRing.emevd gives effects to 8 rings, none of them these |
| 40490000 (Great Eagle Crest Ring) | no | - | only the Japanese name exists; there is no RingParam row |

**In retail, no**, not without a mod: none of these has an ArmorParam / RingParam or ItemParam row. Leftovers:
the Lucatiel's Mask variant's model (`HD_7521`) and its EquipInterfereParam row 17521100, so only that ArmorParam row
would be needed to bring it back. Retail kept the Ring of Thorns name, but for a new ring at 40530000.

## Limits

- "Cut" means in the demo's data but not in retail v1.00. [The Scholar edition](cut-content-scholar.md#3-items) and the DLC were not checked.
- Rings and spells have no model, so the provisional names carry most of the weight. Where the demo has neither
  text nor an effect script (most rings, 44 spells), "same ring", "different ring" and "possible counterpart" are
  readings of the names, marked as such.
- The demo's ring effects were compared by their script arguments, not decoded further than the twentieth /
  twenty-first NT sessions did (type 0 = physical defence). Retail's quartz and bite rings, whose descriptions say
  what they do, put magic, lightning, fire and dark at types 1-4 and poison, bleeding and curse at 5, 6 and 8.
- The cast animation ids were compared, but the animations themselves were not looked at. Spell bullets were
  compared by content (see "Bullets"); their field meanings (lifetime, speed) are read from value ranges, not
  checked in game. SFX ids could not be compared, because the spell effects were rebuilt. Whether the old and new
  effects look alike was not checked: that would need the DLsE effect files parsed, or both games run.
- The two armour variants (12231100, 17521100) could be equipped live in the demo, but neither has a model there
  (17521100's exists only in retail), so they would be invisible.

## Related pages

- [Unused armour icons](../unused/unused-armour-icons.md): one of the Network Test's orphan armour items, set 2481 (items 22481100-22481103), has a complete model set in retail: the unused hooded sister of the Peasant set.
