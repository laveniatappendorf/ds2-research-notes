# Placements and balance: PS3 Scholar vs PC Scholar (2026-10-24)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author, stage 2 of `..\ASSESSMENT.md`): how do the PC Scholar's enemy and item placements and its balance
(Calibrations 2.02) differ from PS3 Scholar's (vanilla placements, Calibrations 1.13)?

Tools (`..\tools`):
- `placements_pc.py` (enemies + items). Run it with `uv run --no-project --python 3.12 --with openpyxl python -I -X utf8`;
  openpyxl is only for enemy names from the community sheet.
- `balance_pc.py` (every regulation param field by field; `show <Param>`, `fields <Param>`).

Outputs: `placements_pc.txt`, `enemy_placements_pc.tsv` (2,762 generators of both builds), `item_placements_pc.tsv`
(1,010 placements), `balance_pc.txt`, `balance_changes_pc.tsv` (4,758 changed fields), `balance_params_pc.tsv`.
(Balance numbers updated 2026-10-24 after the alignment fix in section 5: first published as 1,617 rows / 5,056 fields.)

**Enemy names** come from the community "Comprehensive Enemy Defenses" sheet (PC SOTFS, keyed by EnemyParam id),
including its location tags. A tag says where the sheet's author met that param id, so e.g. "Parasite Spider (Boss -
The Duke's Dear Freja)" may also be a plain spider elsewhere. Ids the sheet lacks fall back to the AI script label or,
for human NPCs, their gear. Translations of labels are mine.

## Short answer

- **Enemies were reworked heavily.**
  - Generators: 2,094 on PS3, 2,458 on PC. By generator id, 1,433 are unchanged, **668 added, 304 removed, 219 moved,
    138 with another enemy**.
  - The DLC maps are almost untouched (+6), apart from new NPC invaders.
  - 129 enemy kinds (by sheet name) appear only on PC, e.g. the Forlorn (one variant per map), more NPC invaders and
    summons, Heide Knights in Heide's Tower, Insolent Knights in the Undead Crypt.
- **Items were reworked far less.**
  - 858 of PS3's 924 placements are unchanged; 86 pickups are new, 37 have new contents, 19 moved (0.3-38 m), one
    was relocated 132 m, 5 changed in several ways, 3 are gone.
  - The PC adds consumables (Human Effigy 37 → 48, Smooth & Silky Stone 6 → 12, Fragrant Branch of Yore 5 → 11,
    Petrified Something 3 → 10) and puts some gear in the world: the Aurous, Priestess and Wanderer sets, Flamberge,
    Whip, Washing Pole …
  - Enemy drop lists gained 136 rows (the new enemies), and several armour-set drop chances went up (3 % → 10-20 %).
- **Balance moved only a little from 1.13 to 2.02.**
  - 59 of the 195 shared params differ: **1,591 changed rows**, 1,710 new (the new content), 77 gone.
  - Main changes: weapon durability, some attack power and poise damage up; a few armour sets get more poise or
    absorption; 250 enemies take less co-op time; Mimics have more HP; the Black Dragon Knights have less HP but there
    are many more of them.
- **Decoding lesson.** PS3 rows only compare with PC rows after two platform fixes: PS3 bitfields are packed from the
  high bit, and the community definitions mis-size many fields. Without the fixes, over 130,000 fields looked changed;
  with them, 4,758 are (section 4). The same fix now applies to every PS3 tool (section 4).

## 1. Enemies

Data per map: GeneratorParam (`GeneratorRegistParam` field) → GeneratorRegistParam (+0 = EnemyParam id) and
GeneratorLocation (same row id; +0 = position), matched by generator id. Columns below: same / moved > 1 m / another
enemy / added / removed.

| map | PS3 | PC | same / moved / enemy / added / removed | biggest kind changes (PS3 → PC) |
|---|---|---|---|---|
| m10_02 Things Betwixt | 46 | 54 | 41 / 1 / 3 / 9 / 1 | Basilisks 0 → 6, a Pursuer, wanderers behind the statue |
| m10_04 Majula | 52 | 56 | 42 / 0 / 10 / 4 / 0 | basement skeletons regrouped by NG+ cycle, Undead Boars 0 → 2 + a giant one |
| m10_10 Forest of Fallen Giants | 124 | 149 | 68 / 17 / 12 / 52 / 27 | Skeletons 0 → 6, Hollow Infantry up, Old Ironclad Soldiers moved to Soldier's Rest, Royal Swordsmen 0 → 3 |
| m10_14 Brightstone Cove Tseldora | 131 | 185 | 21 / 0 / 21 / 143 / 89 | **Parasite Spiders 44 → 85**, peasants 10 → 17, Hollow Falconers new; most of the map's generators were redone |
| m10_15 Aldia's Keep | 37 | 60 | 35 / 1 / 0 / 24 / 1 | Mimics, Forlorn, Dragon Acolytes 5 → 7, a Belfry Gargoyle |
| m10_16 Lost Bastille / Sinners' Rise | 130 | 149 | 90 / 21 / 4 / 34 / 15 | **The Pursuer 0 → 4** (the multi-Pursuer fights), Belfry Gargoyles 6 → 10 (the 4 extra are a scripted fly-by, see enemy-counts-v110-vs-pc.md), jailers 8 → 4 |
| m10_17 Harvest Valley / Earthen Peak | 106 | 122 | 43 / 22 / 20 / 37 / 21 | Steelworkers 11 → 2 (+4 on Earthen Peak), Undead Supplicants 0 → 8, beetles 17 → 12 |
| m10_18 No-man's Wharf | 58 | 80 | 33 / 17 / 5 / 25 / 3 | dogs 2 → 7, sailors 12 → 16, Hollow Infantry 2 → 6 |
| m10_19 Iron Keep / Belfry Sol | 64 | 76 | 34 / 16 / 9 / 17 / 5 | **Alonne Knights (katana) 11 → 21** |
| m10_23 Huntsman's Copse / Undead Purgatory | 105 | 140 | 67 / 12 / 12 / 49 / 14 | skeletons (Skeleton Lords' params) 0 → 15, prisoners 3 → 11, moths 5 → 11 |
| m10_25 The Gutter / Black Gulch | 104 | 105 | 39 / 18 / 13 / 35 / 34 | Captive Undead 37 → 19 (unarmed), Corrosive Ants new |
| m10_27 Dragon Aerie / Dragon Shrine | 40 | 76 | 20 / 10 / 2 / 44 / 8 | **Black Dragon Knights 3 → 20, Rupturing Hollows 4 → 25**, Drakekeepers fewer |
| m10_29 (MapName: Majula) | 16 | 16 | 15 / 0 / 1 / 0 / 0 | a Grotesque became a Gyrm |
| m10_30 Heide's Tower (Unseen Path) | 14 | 17 | 10 / 0 / 0 / 7 / 4 | Old Knights 4 → 0, a Primal Knight, a Syan Knight |
| m10_31 Heide's Tower of Flame | 24 | 30 | 9 / 10 / 3 / 8 / 2 | **Heide Knights 0 → 6**, a Guardian Dragon |
| m10_32 Shaded Woods | 58 | 75 | 23 / 9 / 21 / 22 / 5 | Forest Guardians 0 → 9, Grotesques fewer, crossbow NPCs 838000 / 838100 → 838300 |
| m10_33 Doors of Pharros | 36 | 37 | 34 / 1 / 0 / 2 / 1 | |
| m10_34 Grave of Saints | 46 | 45 | 41 / 2 / 0 / 2 / 3 | |
| m20_10 Memory of Vammar / Orro / Jeigh | 44 | 61 | 31 / 4 / 2 / 24 / 7 | Ironclad Soldiers 0 → 7, Royal Soldiers up |
| m20_11 Shrine of Amana | 102 | 131 | 58 / 30 / 0 / 43 / 14 | Aberrations, maidens and clerics up, Basilisks and Pyromancers new |
| m20_21 Drangleic Castle / King's Passage | 100 | 104 | 50 / 17 / 0 / 37 / 33 | Alonne Captains (bow) 9 → 1, Stone Soldiers 19 → 10, more Mimics / Old Knights |
| m20_24 Undead Crypt | 69 | 96 | 41 / 11 / 0 / 44 / 17 | **Insolent Knights 0 → 12**, unequipped human NPCs 0 → 13 (696000) |
| m20_26, m40_03, m50_35-38 | 588 | 594 | 588 / 0 / 0 / 6 / 0 | new NPC invaders (Forlorn, O'Harrah, Luet), Velstadt in the Memory of the King |

**New on PC**:
- **The Forlorn**: human invaders 694000-695008 in the Forlorn set, one per map;
- more named NPC invaders and summons (Bradley of the Old Guard, Sellsword Luet, Manhunter O'Harrah, Fencer Sharron,
  Felicia the Brave, Masterless Glencour, Ruined Aflis …);
- Bashful Rays in six maps, Belfry Gargoyles outside the belfries, the Executioner's Chariot in Drangleic Castle;
- 696000, an unequipped human with no AI script of its own number, 33 generators (13 in the Undead Crypt).

**Only on PS3**: crossbow NPCs 838000 / 838100 (replaced by 838300), an unequipped NPC 851000, `上位：ネクロマンサー
（イモータル）` ("necromancer (immortal)") and `彷徨える騎士` ("wandering knight").

**Check**: the community sheet's spawn counts (PC SOTFS) match the PC generator counts for **584 of 644** enemies, so
the PC reading is right; the PS3 side of this comparison uses the same code.

## 2. Items

Chain as on PS3 (`..\..\DS2 Scholar\findings\item-placements-vanilla-vs-scholar.md`): MSB object part entity id
(PS3 +0x9c, PC +0xa0) → `MapObjectInstanceParam_<map>` +0x24 → `ItemLotParam2_Other`.

| | count |
|---|---|
| placements PS3 / PC | 924 / 1,007 (base maps 652 on PS3) |
| unchanged (position, container, lot, contents) | 858 |
| new pickups | 86 (9 of them with an empty lot: Brightstone Cove 5, the Gutter 4) |
| new contents in the same place | 37 (+ 5 with a new lot or container too) |
| moved | 19 (0.3-38 m; Great Heal in the Grave of Saints 38 m) + 1 relocated (a Lifegem in the Forest, 132 m) |
| gone | 3 (Human Effigy x5 chest in Brightstone Cove, a Twinkling Titanite chest in the Gutter, Fire Greatarrows in Drangleic Castle) |

**In the world only on PC**:
- armour sets: Helm / Armor / Gauntlets / Leggings of Aurous (the Gutter, replacing the Ring of the Evil Eye); the
  Priestess set (Brightstone Cove, replacing an Estus Flask Shard); the Wanderer set (Lost Bastille);
- weapons and shields: Flamberge (Copse, replacing Poison Moss), Whip, Washing Pole, Foot Soldier Shield, Cleric's
  Parma (Heide, replacing the Heide Knight Iron Mask);
- the three DLC keys Dragon Talon, Heavy Iron Key and Frozen Flower (Majula, Forest, Drangleic Castle).

**In the world only on PS3**: both Rings of Life Protection, Staff of Wisdom, Watchdragon Parma, Heide Knight Iron Mask,
Ring of the Evil Eye, and the Aldia Key, which moved to a lot no map object places (60050000).

The 12 Estus Flask Shards stay at 12 but several moved: PS3 shard spots in the Forest, Brightstone Cove, Harvest
Valley, the Wharf and Shaded Woods now hold other items, and new shards appear in Things Betwixt, the Forest, the
Gutter, Heide and the Copse. Full list: `item_placements_pc.tsv`.

**Lots no map object places**: PS3 434, PC 464. 4 changed:
- the Second Dragon Ring lot gains a Black Scorpion Stinger;
- lot 1721000 (Royal Dirk + Espada Ropera) gains Llewellyn's armour;
- Smooth & Silky Stone → Petrified Something;
- Green Blossom x3 → Ring of Binding.

28 are new: 25 "Rubbish" placeholder lots 11000-11240, two Twinkling Titanite lots and the Aldia Key's.

**Enemy drops** (`ItemLotParam2_Chr`): 615 → 749 rows (136 new, 2 gone, 24 changed):
- armour-set chances up (Aurous 3 → 10 %, Monastery 3 / 5 → 20 %, Royal Swordsman 3 → 10 %, Priestess pieces 3 → 10 %);
- Black Dragon weapons down (3 → 0.5 %); a weapon-gamble lot (20300000 / 20300001: Mail Breaker, Parrying Dagger,
  Zweihander …) 0.5 → 0.2 %;
- the Lion Warrior and Ruin sets drop at 2 or 4 % instead of 1 %;
- Bell Keepers drop their bow and shield (5 %).

## 3. Balance: Calibrations 1.13 → 2.02

`balance_pc.py` (both builds decoded with the PC definitions; section 4).

| | |
|---|---|
| shared params | 195; **59 differ**, 136 identical (PS3-only: the four old item-lot params; PC-only: 10, e.g. camera params, `PlayerWeaponReinforceParam`, `MapObjectPlayGoParam`) |
| rows | 1,591 changed, 1,710 new (Forlorn, new NPCs and enemy variants, damage rows of new attacks), 77 gone (63 of them `LogicComParam`) |
| grown params (layout) | AreaParam 80 → 108 B (more mimicry-object slots), ChrMultiplayParam 24 → 36, MapObjectParam 28 → 32, NetworkParam 588 → 616 |

Highlights (`balance_pc.py fields <Param>` gives the detail):
- **Weapons** (WeaponParam 80 rows changed, 17 new):
  - durability up on 28 weapons (median x1.43: Royal Dirk 60 → 90, Espada Ropera 45 → 80); Broken Thief Sword 60 → 30;
  - poise damage up on 16 (PvE x1.17, PvP x1.27); counter damage up on 14 (Blacksteel Katana 130 → 150);
  - lower requirements on five (Retainer's Short Sword STR 11 → 5, Old Knight Greatsword 25 → 17; Gargoyle Bident
    15 → 22 is the one raise);
  - attack power on 19 reinforce rows, median +8 % (Hand Axe 120 → 135, Battle Axe 125 → 140, Drakeblood Greatsword
    150 → 165, Thief Dagger 72 → 79; Old Mirrah Greatsword 125 → 110).
- **Armour**:
  - Old Knight set poise +11-22 % (armour 45 → 50); Astrologist's set x0.8 weight;
  - absorption raised: Bell Keeper lightning x2, Syan's fire x1.5, Rusted Mastodon dark x1.5, Judgment poison x2,
    Singer's Dress.
- **Enemies** (EnemyParam 303 rows changed, 144 new):
  - `coop_time_reduction` down for 250 enemies (median x0.75: 12 → 10, 60 → 48);
  - Mimics +30 % HP (1,160 → 1,500); Ruin Sentinel in Drangleic Castle 3,260 → 2,070; Black Dragon Knights 2,290 →
    1,570; Body of Vengarl 1,340 → 2,340 and his summon 6,850 → 13,180;
  - Varangian Sailors and a Giant Undead Boar gain poise (0 → 22);
  - souls: The Pursuer 3,000 → 6,000 / 7,000, Corrosive Ant Queen 300 → 1,800, Bowman Guthry 2,806 → 7,483, Prowling
    Magus 0 → 2,500, Navlaan 2,000 → 5,334;
  - Aldia (693000) HP 10 → 999,999.
- **Rings**: Ring of Soul / Life Protection repair cost 6,000 / 3,000 → 14,000.
- **Spells**: SpellParam unchanged for the player. The enemy-cast versions (7xxxxxxx) of Soul Greatsword, Soul Vortex,
  Soul Flash, Focus Souls, Emit Force, Scraps of Life and Resonant Soul get their own bullet and damage rows.
- **Items**: the Peasant set gets its own special effect per piece (-1 → 22480100-103); 11 spells get an
  `item_additional_type`; ng_behavior_type of the three DLC keys 15 → 11.
- **AI**: two logic rows see less far (`LogicComParam` 105010 / 105030: sight 40 → 30 m); skeleton logics (132xxx)
  retimed; 63 logic rows gone.
- **PvP / system**: riposte, backstab and guard-break animations re-pointed for four weapon action categories; the
  co-op / network tables changed in layout, hardly in content (ChrMultiplayParam grew 24 → 36 B with all 28 rows'
  old values kept; NetworkParam grew, one row with a few changed values).

## 4. Decoding PS3 rows against PC rows

Both builds are decoded with the PC (SmithBox DS2S) definitions, so field names match. Two platform differences
appeared once PS3 and PC rows were set side by side:

1. **PS3 bitfields are packed from the high bit of their unit; PC from the low bit.** `paramdef.decode` assumes the
   low bit for both.
2. **The definitions often declare the wrong width**: a u8 upgrade cap typed u32, a u16 level typed s32, a float typed
   as four u8, 8 or 32 flag bits typed as a number, a 1-bit flag widened with padding. On little-endian PC these still
   read correctly; on big-endian PS3 they do not (an upgrade cap of 10 reads 167,772,160).

`balance_pc.py` gives each field the conversion under which the PS3 row equals the PC row in the most shared rows:
- a k-byte swap;
- per-byte bit reversal;
- the 2- / 4-byte scalar the field is part of;
- high-bit packing or mirrored packing, for bitfields.

Ties keep the plain reading, and the chosen conversions are listed per param in `balance_params_pc.tsv`. That took
over 130,000 apparent field changes (112 params) down to 4,758 (59 params), with the large remainders checked by hand
(co-op time, physics textures, light intensities).

**Fixed for every PS3 tool (2026-10-24).** The shared `paramdef.py` now reads big-endian rows with high-bit bitfields
and the per-field readings measured by `..\..\DS2 Network Test\tools\ps3fields.py` (131 fields in 72 param types, kept
in `ps3fields.json`; these use the PS3 def set, while this note's own conversions use the PC defs). Where a field's
bytes are identical on both platforms, the PS3 value now reads the way the PC and SmithBox read those bytes.

Re-check of the PS3 work:
- the vanilla-vs-Scholar and patch-to-patch tables were regenerated: the same changed rows and the same counts, with
  different flag-bit names and readable values for mis-sized fields;
- no figure quoted in a PS3 note was affected (`..\..\DS2 Scholar\findings\balance-vanilla-vs-scholar.md`,
  `..\..\DS2 Vanilla Patches\README.md`);
- the enemy-defence sheet's fields (HP, defences, absorptions, poise, `is_hollow`) are plain fields and read the same
  on both platforms.

## 5. Checks and not checked

- **Correction (2026-10-24)**: the stage 2 edit making `paramalign.py` read each param in its own byte order did not
  apply at the time. The four grown params were aligned with PC words read as big-endian, so the earlier "PS3 results
  identical" check was trivially true. Applied with the PS3 fix; the PC balance numbers above were updated (1,617 →
  1,591 changed rows, 5,056 → 4,758 fields: ChrMultiplayParam's 26 "changed" rows were an alignment artefact).
- PC enemy loader: 584 of 644 community-sheet spawn counts match.
- Not checked:
  - NG+ rules: per-generator spawn limits are in the TSV, not analysed;
  - what the 9 empty new pickups are for (event-filled?);
  - why 60 sheet rows differ (sheet made from an older PC version?);
  - whether the 696000 humans are statues or corpses;
  - the effect scripts (SpEffect EMEVD, now in the regulation);
  - the PC patch history (all of this is 1.0.3.0 / Calibrations 2.02).
