# Balance params: vanilla v1.00 vs PS3 Scholar (2026-10-23)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author): the PS3 Scholar is the base game plus DLC plus the patches; what did the patches change in weapons,
armour, damage and spells? Static comparison of the two regulation files (vanilla `..\DS2 Vanilla\regulation`, 252
params, v1.00 disc; Scholar `regulation`, 221 params, disc of 26 Nov 2014).

Tools (this folder's `tools`, run with `uv run --no-project --python 3.12 python -I -X utf8 tools\<script>`):
`balance_compare.py` (every param, field by field; writes `balance_changes.tsv` = 58,827 changed fields and
`balance_params.tsv` = one line per param), `balance_fields.py <Param>` (per field: up / down / typical ratio),
`balance_tables.py` (weapon attack and armour tables), `balance_spells.py` (per spell), `speffect_compare.py` (effect
scripts), `paramalign.py` (old layout -> new). Shared: `..\..\DS2 Network Test\tools\paramdef.py` (decodes any DS2 PS3
param with the community Paramdex defs).

**Re-checked 2026-10-24 (PS3 reading fixed).** Setting PS3 Scholar rows beside the PC Scholar's showed two errors in
how `paramdef.py` read PS3 params:
- PS3 bitfields are packed from the high bit;
- the community defs give some fields the wrong width (a u8 typed u32, a float typed as four u8 …).

Both are fixed: per-field readings are measured by `..\..\DS2 Network Test\tools\ps3fields.py`, 131 fields in 72 param
types. `balance_changes.tsv` / `balance_params.tsv` are regenerated.
- **Same 58,827 changed fields in the same rows and params.** What changed is presentation:
  - the names of the flag bits that change in PlayerDamageParam / EnemyDamageParam (`target_*`);
  - the values shown for mis-sized fields, e.g. WeaponAttackMotionParam `attack_type` 117440512 -> 7,
    DamageCtrlParam `block_interaction` 512 -> 2, MapObjectParam `Unk10` 64 -> 2.
- **No figure quoted in this note was affected** (tracking, poise, motion values, armour, spells, PvP and NG+ numbers
  come from correctly read fields).

## 0. Summary

- **116 of the 220 shared params differ: 15,830 changed rows and 8,255 new ones** (most new rows are DLC). The
  patches retuned almost every combat table.
- **The disc is at patch 1.10 (Calibrations 1.13)**: it has the 1.10 changes (Covenant of Champions damage, Awestone
  drops, Human Effigy rewards, covenant devotion, Agape Ring), and its regulation binder's version field reads
  `01001300`. It does not have the later Calibration 1.15 changes (section 1).
- **Weapons**: 79 weapons hit harder (median +13 %, mostly early or weak ones: Mail Breaker x2.1, Red Iron Twinblade
  x1.9, Santier's Spear x1.75, Thief Dagger x1.4), 7 hit less (Chaos Blade, Shotel, Butcher's Knife ...); PvP poise
  damage up on 324 weapons (median x1.5); a new hyper-armour field on 97 weapons; attack tracking cut hard
  (mid-swing tracking 1.0 -> 0.3 on 514 attacks); poise damage of 1,041 attack rows up (median x1.8); counter-hit
  bonus 1.0 -> 0.91 on 334 attacks; every weapon type gets a Mundane scaling factor (0 -> 100).
- **Armour**: the heavy sets gain poise (+12-15 %: Havel's Armor 47 -> 51) but lose defence (max slash -5 to -10 %)
  and elemental absorption (-15-20 %); the Black Dragon set is rebuilt (weight / 3, no poise, no requirements,
  fire absorption x2.5).
- **Spells**: 88 of 106 changed: most cast faster and cost less stamina; cast counts reshuffled (Lightning Spear 10
  -> 3 uses at the first tier, but damage x1.2; Great Soul Arrow 10 -> 15; Cast Light / Unveil / Perseverance 4 -> 8;
  Emit Force 8 -> 4); several rebuilt mechanically (homing soul arrows, heals, Soul Greatsword).
- **System**: new PvP multipliers (backstab x0.8, riposte x0.9 vs players), Company of Champions enemy damage 1.35 ->
  1.5, resistances from stats roughly doubled, enemy hits wear equipment 20 % less, NG+ scaling reworked.
- **Unchanged and worth knowing**: the PhysicalStats columns behind roll / backstep i-frames (agility-iframes.md)
  are identical, so the retail i-frame findings hold for Scholar.

## 1. Which patch level is on the disc?

Patch notes (Fextralife's patch page and press coverage, see Sources) against the data:

| patch | note | on the Scholar disc? |
|---|---|---|
| 1.04 (Apr 2014) | Soul Geyser nerf (half damage, slower, more stamina) | consistent: Soul Geyser startup 1.0 -> 0.8, stamina 2.5 -> 3.0 |
| 1.07 / Cal. 1.10 (Aug 2014) | Ring of Steel Protection defence raised, offline only | **yes**: an extra 0.9 / 0.875 factor added to its effect script |
| 1.07 | defence lowered: Gyrm Helm, Smelter Demon Helm, Velstadt's Helm, Black Witch Robe | **yes**, all four (section 4) |
| 1.07 | spell parry time up for Cleric's Parma, King's Mirror ... | yes: parry_frames_duration changed on 72 weapons, Cleric's Parma among them |
| 1.07 | Soul Spear casting time raised | yes: Soul Spear cast_speed 1.0 -> 0.9, startup 0.5 -> 0.75 s |
| 1.08 (Jul 2014, with DLC 1) | many weapons more attack, spells gain / lose uses, less stamina | yes (section 3, 5); a player report of Lightning Spear "11 -> 3" uses matches 10-11 -> 3 here |
| 1.10 / **Cal. 1.13** (Feb 2015) | Covenant of Champions: enemies deal more damage | **yes**: DamageAdjustParam enemy_damage_mult_coc 1.35 -> 1.5 |
| 1.10 | Awestones dropped more often | **yes**: lot 6000000 Awestone 3 % -> 12.5 % (item-placements note) |
| 1.10 | Human Effigy rewards for invader / covenant kills | **yes**: ResultEventParam rows 1100 / 1200 / 2400 + covenant lots |
| 1.10 | Blue Sentinels devotion +1, Brotherhood of Blood +2 | **yes**: ResultEventParam 1205 flag 0x04020001 -> 0x04020101, 2510 0x04020100 -> 0x04020200 (the third byte = devotion) |
| 1.10 | Agape Ring; Aldia (the Scholar) | **yes**: item 42000000 Agape Ring; Aldia (section 3 of the README) |
| Cal. 1.15 (1 Apr 2015) | attack up: Thief Dagger, Hand Axe, Battle Axe, Bandit Axe, Lion Greataxe, Gyrm Great Hammer | **partly**: the first four are higher, Lion Greataxe and Gyrm Great Hammer are not |
| Cal. 1.15 | Sorcerer's Twinblade magic scaling, Astrologist set lighter | **no**: neither changed |

So the disc carries everything through 1.10 / Calibrations 1.13 and not 1.15 (released with the new-gen Scholar).
The four 1.15-listed weapons that are already stronger probably got their buff earlier (1.08 raised many weapons).
The binder version `01001300` fits "1.13" (vanilla's own reads `00010000`). A title update could bring the disc to
1.14 / 1.15 in game; none is installed in RPCS3.

## 2. Method and limits

- Field names: the PC vanilla DS2 Paramdex defs (SmithBox, `C:\Games\SmithBox\Assets\PARAM\DS2\Defs`) fit **every
  Scholar param** (220 of 221; ItemLotParam2 is empty), big-endian. They do **not** fit 48 of vanilla's: the patches
  grew some layouts. `paramalign.py` maps vanilla's rows onto the newer layout by the data (column agreement over all
  shared rows); every grown param turned out to be the old layout plus new fields at the end or one inserted word:
  WeaponParam 184 -> 188 (+ hyper_armor_poise_mult), EnemyParam / ChrParam 356 -> 360 (+ speffect_id), Enemy /
  ChrCommonParam 888 -> 908 (+ five bullet_absorption values), DamageAdjustParam 344 -> 396 (+ 14 values),
  PlayerCommonParam 356 -> 440 (+ 21 values incl. three PvP critical multipliers), GrabParam, GroundMaterialParam,
  FootEffectSfxParam, DamageEffectSeParam / MaterialParam, KeyConfigParam, CameraSubjectiveParam,
  DualWieldingPermissionParam. 30 vanilla params Scholar dropped are named by neither executable (dead data).
- Names are the community's; many are `UnkNN`, and some named ones are guesses (e.g. DamageAdjustParam's new
  "*_absorption_mult_pvp" fields hold 6, 10, 12, which do not look like multipliers). Treat names as labels.
- Rows are named from ItemName (weapons, spells, rings by id; armour through ItemParam armor_id; upgrade and scaling
  rows through the weapons that use them; weapon types through weapontype.fmg; enemies from their AI script labels).
- Vanilla's 30 dead params and the 4 % of fields that are bitfields: bit order inside a byte follows SoulsFormats
  (lowest bit first); a flag could be mislabelled, the values are right.

## 3. Weapons

### Attack power (WeaponReinforceParam, base and +max physical; elemental columns move with it)

86 upgrade lines changed: 79 up (median +13 %), 7 down. All of them, base -> Scholar (max at full upgrade):

- **x1.5 and more**: Mail Breaker 35 -> 75 (70 -> 150), Key to the Embedded 115 -> 220, Red Iron Twinblade 90 -> 170,
  Santier's Spear 133 -> 233, Roaring Halberd 72 -> 124, Chaos Rapier 46 -> 73, Silverblack Spear 50 -> 75.
- **x1.2-1.4**: Shield Crossbow, Heide Spear, Thief Dagger, Dragonslayer Spear, Black Scorpion Stinger, Twinblade,
  Notched Whip, Pate's Spear, Heavy / Light Crossbow, Bandit's Knife, Black Flamestone Dagger, Witchtree Branch,
  Old Knight Hammer, Great Machete, Sorcerer's Staff, Dragonrider Twinblade, Bandit Greataxe, Scythe, Curved
  Twinblade, Foot Soldier Sword, Aldia Hammer.
- **x1.02-1.19**: Melu Scimitar, Claws, Spotted Whip, Malformed Claws, Blue Flame, Bluemoon Greatsword, Crescent Axe,
  Bandit Axe, Partizan, Lost Sinner's Sword, Winged Spear, Manikin Sabre, Archdrake Mace, Giant Warrior Club,
  Darkdrift, Royal Greatsword, Red Rust Scimitar, Stone Soldier Spear, Pike, Old Knight Pike, Homunculus Mace,
  Blacksmith's Hammer, Spear, Drakekeeper's Great Hammer, Hand Axe, Warped Sword, Battle Axe, Spider's Silk, Gyrm
  Greataxe, Lucerne, Full Moon Sickle, Barbed Club, Silverblack Sickle, Malformed Skull, Smelter Sword, Blue Knight's
  Halberd, Old Knight Halberd, Broken Thief Sword, Black Dragon Greataxe, Drakekeeper's Warpick, Scimitar,
  Uchigatana, Rapier, Drakewing Ultra Greatsword, Great Scythe, Gargoyle Bident, Black Dragon Warpick, Zweihander,
  Espada Ropera.
- **Down**: Drangleic Sword 186 -> 180, Morning Star 115 -> 110, Chaos Blade 106 -> 100 (max 265 -> 250), Butcher's
  Knife 120 -> 112, Channeler's Trident 130 -> 120, Shotel 110 -> 100, Mace of the Insolent 55 -> 50.

Also in WeaponReinforceParam: 21 / 19 lines got higher poison / bleed infusion factors (e.g. Manikin Sabre 25 -> 50),
40 lines point at a different scaling row, shield stability changed on 10 lines.

### Scaling (WeaponStatsAffectParam)

340 scaling rows changed, almost all the **fire, lightning and dark scaling** of the infusion rows, a few per cent up
per upgrade level (median +2 % at +1, +3 % at +2, e.g. 0.539 -> 0.551). Black Knight Halberd's Strength scaling
changed. 514 new rows (DLC weapons).

### Per weapon (WeaponParam)

| field | rows | change |
|---|---|---|
| poise_damage_pvp | 324 | all up, median x1.5 (dagger 2 -> 3.5, 7 -> 8.75 ...) |
| hyper_armor_poise_mult (new) | 97 of 456 | 0.3 (36), 0.5 (28), 0.8 (26), 0.6 (7): poise damage taken while in hyper armour |
| parry_frames_duration | 72 | 48 longer, 24 shorter (0.675 -> 0.72 s, 0.63 -> 0.54 s ...) |
| recovery_animation_weight | 33 | all lower (median x0.56) |
| hitbox_length | 26 | mostly shorter (daggers 1.0 -> 0.7) |
| damage_mult | 8 | Silverblack Sickle / Dragonrider Bow 0.85, Scythe of Want 0.8, Light Crossbow and two others 1.25 |
| max_durability | 7 | twinblades 60 -> 120-175, Defender / Watcher Greatsword 60 -> 50 |
| stamina_consumption / damage | 10 | mostly up (x1.1, one bow down) |
| weight | 3 | Defender Greatsword 8 -> 12, Butcher's Knife 22 -> 16, Old Knight Halberd 16 -> 15 |
| weapon_type_id | 6 | Bandit's Knife, Shadow Dagger, Manikin Knife, Spitfire Spear ... moved to new weapon types (new movesets) |
| poise_damage_pve | 6 | twinblades 1.0 -> 2.5 |
| status_effect_amount | 13 | staves etc. 0 -> 1 |

### Weapon types (WeaponTypeParam)

`mundane_scaling_mult` 0 -> 100 on all 103 types (2 x 64, 2 x 75, 1 x 47): Mundane infusion scaling now set per type.
Bows / greatbows / crossbows: two unnamed factors 1.0 -> 0.7 / 0.5 / 0.15. Powerstance category changed on 11 types
(Ultra / Curved Greatsword 110 -> 100, Great Hammer 100 -> 120).

### Attacks (WeaponAttackMotionParam, PlayerDamageParam, DamageCtrlParam)

- **Tracking**: tracking_start down on 857 attacks (1.0 -> 0.7 on 521), **tracking_middle 1.0 -> 0.3 on 514**,
  tracking_end 0.2 -> 0.1 on 220: swings turn much less after they start. Startup / recovery speeds retuned both ways
  (887 / 658 rows).
- **Motion values**: damage_mult changed on 555 player attack rows (316 up, 239 down, median x1.11); knockback on
  511; stamina damage on 435; **counter_damage_mult 1.0 -> 0.91 on 303 attacks and 0.8 on 31** (less counter-hit
  bonus); hitbox radius up on 156.
- **Poise damage**: DamageCtrlParam poise_damage up on 1,041 rows (median x1.8: 52.5 -> 94.5, 37.5 -> 67.5,
  15 -> 27), down on 174; stagger_mode 0 -> 1 on 207; posture_damage down on 176; tip-hit distances up on 166.

## 4. Armour and rings

- **Poise up, defence down on the heavy sets** (ArmorParam poise 65 up / 4 down; ArmorReinforceParam defences 73 of
  79 down, elemental absorption about -15 to -20 %): Havel's, Steel, Gyrm Warrior, Ironclad, Old Ironclad, Smelter
  Demon, Alonne Knight, Vengarl's, Rusted Mastodon, Mastodon, Old Knight, Drakekeeper, Velstadt's, Looking Glass,
  Imperious, Dragonrider. E.g. Havel's Armor poise 47 -> 51, max slash 491 -> 453, fire 4.5 -> 4.1; Gyrm Warrior
  Greathelm poise 16 -> 24 and heavier (7.9 -> 9.6). Ironclad / Old Ironclad Armor weight 16.8 -> 29.8.
- **Black Dragon set rebuilt**: weight 6.5 / 14.5 / 5.6 / 8.4 -> 2.2 / 4.8 / 1.8 / 2.7, poise -> 0, Strength and
  Dexterity requirement 14 -> 0, slash defence about / 2.5, fire absorption x2.5.
- **Fixes**: Cale's Leather Armor and Cale's Shoes swap their values (vanilla had them the wrong way round: armour 3.0
  weight / 77 slash, shoes 6.4 / 164); Monastery set: headcloth / shirt / gloves up, skirt down.
- Gyrm Helm (max slash 179 -> 164), Smelter Demon Helm (213 -> 178), Velstadt's Helm (179 -> 172) and Black Witch
  Robe (141 -> 116) defence lowered: exactly patch 1.07's four pieces.
- **Rings**: RingParam unchanged (23 new rows: DLC and the Agape Ring). The effects live in `SpEffectRing.emevd` (13
  changed, 23 new): Chloranthy Ring 1.035 / 1.055 / 1.085 -> **1.15 / 1.2 / 1.25**; Ring of Steel Protection gets an
  extra 0.9 / 0.875 factor (the 1.07 "offline only" buff); Stone Ring's packed argument 10 -> 30; Clear Bluestone
  Ring+2 0.45 -> 0.6; Redeye Ring 10 -> 2; Gower's Ring gains two factors (0.8, 0.15); Abyss Seal 1.2 -> 1.075;
  Old Sun Ring and White Ring changed. The instruction set is not decoded, so these are raw numbers.

## 5. Spells

88 of 106 spells changed in SpellParam, 61 in their damage row (PlayerDamageParam, same id as the spell), 39 in their
bullet (BulletParam, same id); 103 of 123 vanilla spell effect events changed and 71 were added (`SpEffectSpell.emevd`).
Patterns (full list: `tools\balance_spells.py`):

- **Faster casting**: cast_speed up on 110 spells (1.0 -> 1.3 on 41, up to 2.0 for Shockwave, 1.8 for Force,
  1.6 Homing Soulmass), down on 44 (Soul Spear 0.9, Crystal Soul Spear 0.85, Soul Greatsword 0.85, the Lightning /
  Sunlight Spears 0.9, Great Heal 0.9).
- **Cheaper**: stamina cost down on 61 spells, up on 26 (median x0.67; Soul Spear 2.0 -> 1.25, Homing Soulmass 2.5
  -> 1.5, Soul Geyser up 2.5 -> 3.0, Emit Force 1.25 -> 2.0).
- **Uses per attunement tier** (casts_tier_1 .. 10): Lightning Spear 10-20 -> 3-15, Great Lightning Spear 8-15 ->
  2-10, Sunlight Spear 3-5 -> 2-4, Emit Force 8-15 -> 4-10, Soul Appease 4-8 -> 2-5 (down); Great Soul Arrow
  10-18 -> 15-19, Soul Shower 4-8 -> 5-12, Soul Bolt 2-4 -> 3-6, Heavenly Thunder 4-8 -> 5-12, Cast Light,
  Caressing Prayer, Unveil, Perseverance 4-8 -> 8-15 (up). Crystal Soul Spear now takes 2 slots, Unveil 1.
- **Damage**: the spears hit harder per cast (Lightning Spear damage_mult 1.25 -> 1.5, floor 85 -> 100; Great
  Lightning Spear 1.45 -> 1.75; Sunlight Spear 1.75 -> 2.5; Soul Spear 1.25 -> 1.8; Crystal Soul Spear 1.5 -> 2.0)
  while their stamina damage fell; Emit Force 1.16 -> 1.0. Patch 1.07 said Soul Spear's and Crystal Soul Spear's
  "strength" was lowered, yet the v1.00 -> disc change in damage_mult is up: either a later patch (1.08) raised them
  again, or damage_mult is only part of a spell's damage (damage_0 also changed, 120 -> 220 / 220 -> 300). Only a
  live test would settle it.
- **Rebuilt**: Homing / Heavy Homing Soul Arrow (the damage moved off the main bullet), Soul Greatsword (child
  bullet), Shockwave (child bullet on hit), the heals and Magic Barrier / Sacred Oath / Perseverance (apply their
  effect through a special effect instead of a child bullet), Blinding Bolt (faster, second bullet).

## 6. Damage, PvP and New Game+

- **PvP** (PlayerCommonParam, new fields): pvp_backstab_damage_mult **0.8**, pvp_riposte_damage_mult **0.9**,
  pvp_guardbreak_damage_mult 1.0 (patch 1.07: "balanced fatal blows ... in online multiplayer").
- **DamageAdjustParam**: enemy_damage_mult_coc **1.35 -> 1.5** (Company of Champions); seven unnamed factors 1.25 ->
  1.15; 14 new values (named "*_mult_pvp" by the community but holding 6 / 10 / 12, so unclear).
- **Resistance from stats** (PhysicalStatsPerLevelStatValuesParam, 99 levels): the magic / fire / lightning / dark /
  poison / bleed / petrify / curse columns roughly double (level 1: 5 -> 10; 17.7 -> 28.5 mid-range; median x1.84).
  MenuStatsParam's displayed defences follow (86 up, 9 down). The i-frame and other columns are unchanged.
- **Equip load** (ChrEquipLoadParam): two rows retuned: row 2 breakpoints 30 / 70 / 100 -> 21 / 49 / 70 % and row 25
  31.5 / 45 -> 28 / 40 %; which characters use rows 2 and 25 is not traced.
- **NG+** (ChrRoundDamageParam, 57 rows): stamina damage per cycle up everywhere (NG+1 120 -> 125-200 ... NG+7 180 ->
  200-600); damage per cycle reshaped: 13 rows lower from NG+2 on (170 -> 160, 200 -> 175, 250 -> 200, 300 -> 215,
  350 -> 230), others higher. EnemyParam moves 279 enemies onto other NG+ rows (1 -> 21, 9 -> 29 ...).

## 7. Enemies

- **Equipment wear**: EnemyDamageParam durability_damage_mult down on 4,921 enemy attacks (5 -> 4 on 4,592, 10 -> 8
  on 300): enemy hits wear your gear 20 % less. damage_0 up on 375 enemy attacks (median +9 %), down on 83.
- **Online**: EnemyParam coop_time_reduction x0.67 on 567 enemies (90 -> 60, 18 -> 12, 180 -> 120).
- **HP** (18 rows; names = AI script labels): The Lost Sinner (忘れられた罪人) 3900 -> 3560; undead citizens
  (アンデッド市民, 9 rows) about 280 -> 90; 異形の影（生者） 560 -> 330; 呪術師の罪人 770 -> 310; and NPCs doubled:
  呪術屋 "pyromancy dealer" 1280 -> 2550, 奇跡の人 "miracle person" 2440 -> 4580, 上位武器屋 "upper weapon dealer"
  1840 -> 3670.
- **Projectiles**: EnemyCommonParam's new bullet_absorption fields (225 rows, one per enemy type): 51 rows take
  x0.8 physical, **x0.475 magic** and x0.56 fire from projectiles; 34 more x0.85-0.9; 140 x1.0.
- EnemyParam's new speffect_id: set on 23 of 834 enemy rows (e.g. 96660000 on 6).

## 8. Not covered / open

- The effect scripts' instructions are only laid out, not decoded: ring / spell / weapon buff magnitudes above are
  raw arguments. 9 of 11 scripts changed (Spell 103 events, Condition 32, Ring 13, ActiveItem 10, Weapon 9).
- Which PlayerDamageParam / DamageCtrlParam rows belong to which weapon's moveset (needs the TAE / action links).
- Item effects: Rusted Coin, Hush and Chameleon (1.10) did not show in params; probably in the effect scripts.
- The 1.14 / 1.15 title updates are not on the disc.

## Sources

- [Fextralife DS2 wiki: Patches](https://darksouls2.wiki.fextralife.com/Patches)
- [Game Informer: Dark Souls II 1.08 patch brings buffs, nerfs and bug fixes](https://www.gameinformer.com/b/news/archive/2014/07/24/dark-souls-ii-1-08-patch-brings-buffs-nerfs-and-bug-fixes.aspx)
- [GameBanshee: Dark Souls II Patch v1.10 released](https://gamebanshee.com/k3uy8)
- [IBTimes: Scholar of the First Sin patch detailed](https://www.ibtimes.co.in/dark-souls-2-scholar-first-sin-patch-detailed-release-date-full-patch-notes-620593)
