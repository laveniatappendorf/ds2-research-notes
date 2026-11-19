# Community damage calculator v2.44: Scholar vs vanilla PC 1.12

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-10-28. The author asked whether the community "Damage Calculator v2.44 - Dark Souls II" (Google Sheets, written for
Scholar) would differ on vanilla PC's current patch, and for a vanilla copy if so.
Copy: `<local path> Calculator v2.44 - Dark Souls II - Vanilla PC 1.12.xlsx` (new "Vanilla Notes"
tab inside). Scripts: `..\tools\damage_calc\`.

## Vanilla 1.12 regulation

Installed build 1.12 (exe 1.0.12.0): `enc_regulation.bnd.dcx` copied to `..\1.12\`, decrypted with
`pccrypto.py regulation` (BND4 version **01001500**; PC Scholar 02000200), unpacked with `bndx.py x` into
`..\1.12\regulation\` (221 files). Vanilla has 4 params Scholar lacks (ItemLotParam, ItemLotParam2, ItemLotTicketParam,
ItemLotWeightParam) and lacks 10 of Scholar's (camera params, [ChunkPhaseParam, MapObjectPlayGoParam](../mechanics/chunk-phase-playgo.md),
PlayerWeaponReinforceParam, 3 GeneralLocationEventParam). AreaParam, ChrMultiplayParam, MapObjectParam and
NetworkParam have smaller rows than Scholar's (the DS2S defs don't fit them).

## Vanilla 1.12 vs PC Scholar (Calibrations 2.02), regulation-wide

`regdiff.py` / `fielddiff.py` (params, field by field), `emevdcmp.py` (SpEffect*.emevd, parsed: vanilla is 32-bit
EMEVD, Scholar 64-bit, so the bytes always differ).

- **No changed rows** in WeaponParam, WeaponReinforceParam, WeaponStatsAffectParam, WeaponAttackMotionParam,
  WeaponActionCategoryParam, ArmorParam, ArmorReinforceParam, ArrowParam, PlayerDamageParam, DamageCtrlParam,
  PlayerStatusParam, PlayerLevelUpSoulsParam, CustomAttrSpecParam. Scholar only adds rows: 17 weapons (the Forlorn
  Greatsword 1997000 and Scythe 3080000; the rest are unnamed NPC weapons 886000-910000), 5 armour rows (Forlorn set
  16940100-103, NPC 2300).
- SpellParam: the 7 changed rows are NPC spell copies (7xxxxxxx) that Scholar points at its own bullets and
  EnemyDamageParam rows (19xxxxxxx). Player spells are identical. RingParam: two repair costs. ItemParam: sort /
  NG-behaviour fields only.
- EMEVD: Ring, Weapon, Armor, AbnormalState, Condition, PassiveItem identical. Spell / ActiveItem: Chameleon and Silver
  Talisman get [5 forms in Scholar (3 in vanilla)](../mechanics/chameleon-objects.md). WideUse: 3 Scholar-only events (170000000, 170001000, 170100000), no
  param references them. Enemy: 2 Scholar-only events + command 100090:2 -> 100090:9 with the same arguments.
- So the calculator's weapon, moveset, armour, ring, spell, buff, stat and level data is the same in vanilla.
  **Only the enemy table differs.** The exe's damage code was not compared.

## Enemies sheet

The calculator's hidden Enemies sheet is Evan's comprehensive PC Scholar export (the same data as
`<local path> of Dark Souls 2 SOTFS - Comprehensive Enemy Defenses V2.xlsx`). Every stat column of all 644 rows reproduces exactly from PC Scholar EnemyParam /
EnemyCommonParam (`enemycheck.py sotfs`). Spawn Count = generators per EnemyParam id for 584 rows; 60 rows are hand
counts ("All Variants" family sums, NPCs with several generators counted once, the Belfry Gargoyles as 5, etc.).
Spawn Count feeds no formula; it is only an anchor for an OFFSET into the HP columns.

Vanilla 1.12 (`enemydata.py vanilla`: 2,094 generators in 28 maps, [the PS3 Scholar count; PC Scholar has 2,458](placements-balance-pc.md)):

- 124 rows removed: 120 ids not in vanilla's EnemyParam (Scholar's new placements: Forlorn, Sellsword Luet, Bashful
  Ray, Heide Knights in Heide's Tower, Falconers in Tseldora / Shaded Woods, Pursuer in Things Betwixt, ...), 4 ids
  vanilla has but never places.
- 30 kept rows have different stats: Drangleic Castle Ruin Sentinel 2070 -> 3260 HP, Black Dragon Knights 1570 ->
  2290, all 7 Mimics about 23 % less HP in vanilla, Vengarl summon 13180 -> 6850, Drummond / Benhart summons lower,
  Varangian Sailors and the NG+ Giant Undead Boar have 0 poise in vanilla (22 in Scholar), Grave Wardens poison
  resistance 70 (Scholar 10), Undead Steelworker 1.0 physical effectiveness (Scholar 0.6), Flame Lizard thrust
  1.0 (Scholar 1.5), Lion Clan bleed resistance 15 (Scholar 0).
- 11 rows added. The method: match generators by (map, generator id). 98 of the 121 placed-but-unlisted vanilla ids sit
  in slots that hold the same id in Scholar too, so the sheet's author left them out on purpose (duplicate NPCs, stat
  twins). The rest are new: Skeleton Lords minions 157003 / 157022 (Scholar's 157008 / 157025), Forest Guardians
  838000 + 838100 (Scholar 8382xx), [Prowling Magus at Amana 137022](shrine-of-amana-pc.md#3-the-other-enemies), Giant Basilisk in Shaded Woods 222001, Heide
  Knights 301001 (Forest of Fallen Giants) / 301003 (Shaded Woods), Flexile Sentry 303301 (Shaded Woods), NPC phantoms
  851000 (Brigand set, Earthen Peak, no name text) and 853300 / 853400 (Drangleic Castle, Guthry's name text 8530).
  Stat twins of listed rows were not added. **Not checked in game**: whether events enable these generators.
- 9 location labels rewritten where vanilla places the id elsewhere (nearest labelled generator on the same map).

## Forlorn gear

The six Forlorn items stay in the lists (positional lookups keep working) with "(SotFS only)" appended to their
English shared strings, which renames them in Weapons, Armors, Usable Weapons, SL1Gear, MovesRedirect and
TranslateWidget together.

## How the copy is built

`build_vanilla.py <in.xlsx> <out.xlsx>` edits the xlsx XML directly (an openpyxl round trip would drop Google's cached
spill values and risk validations). Changed parts: sheet19 (Enemies), sharedStrings, styles (+1 bold font / xf for the
notes), workbook (+ "Vanilla Notes" tab, `localSheetId` 18 -> 19, `fullCalcOnLoad`), rels, content types.
`verify.py`: Enemies rows vs vanilla params (531 / 531), formulas in every sheet unchanged, other cell changes = the
Forlorn names only. Not recalculated here (no spreadsheet program on this PC). Google Sheets recalculates on import.
