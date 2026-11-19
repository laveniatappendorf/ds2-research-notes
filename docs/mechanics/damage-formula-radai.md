# Radai's "Dark Souls II Damage Formula" guide and the game data (2026-11-01)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Source: Radai's guide *Dark Souls II Damage Formula* (9 pages, a Word export; the PDF is in the parent DS2 folder).
Text copy for searching: `damage-formula-radai.txt` (same folder). It describes PC Scholar at 60 FPS: the PvP values,
Avelyn's 0.865 and the DLC enemies only exist there. Re-run the checks with `tools\damage_formula_check.py`.

**In short: the guide matches the data.** Every number in it that can be read from the params or the SpEffect scripts
matches exactly, and it is the formula the rest of these notes build on. The data adds some detail on top: a few rules
have exceptions in particular rows, three values read differently in our data (open until a live test), and the
timing and PvP parts can only be confirmed live.

## The formula in our field names

Damage = (PhysDmg + ElemDmg) x Dm x Cm x hMod x distance cut, with
PhysDmg = (hMod x pMod x (pAR + atkAR) - DEF) / (pCut x hMod) x Rm and ElemDmg = (eAR + bAR + atkAR) x (1 - ElemRES / 100).
In PvE (pMod = pCut = 6) this works out to (hMod x AR - DEF / 6) x Dm x ...: the same model the community calculator
uses and that [Mundane at SL50](mundane-sl50-v100.md) used (max(11.5, AR - def/6) x MV).

| guide name | ours (SmithBox DS2S defs) |
|---|---|
| DamageParam.damageRate (Dm) | PlayerDamageParam.damage_mult |
| DamageParam.damage01-03 (atkAR) | damage_0..2, types damage_type_0..2 (0 physical, 1 magic, 2 lightning, 3 fire, 4 dark, 6 poison, 7 bleed, 8 = Break, see [Break element](../network-test/break-element.md)) |
| DamageParam.damageLower | physical_damage_floor |
| Counter Strength / 100 (Cm) | WeaponTypeParam.counter_damage; the per-attack factor is DamageParam.counter_damage_mult (1.0, 0.91, 0.8) |
| WeaponTypeParam.right/leftDamageScale (hMod) | right_hand_damage_mult / left_hand_damage_mult |
| DamageCtrlParam.damageDecrementHpMaxRate | see conflict 1 (+0xac vs +0xb0) |
| DamageAdjustParam.attributeAdjustNormalMul01 / Div01 | physical_attack_rating_mult_pve +0x114 / physical_absorption_mult_pve +0x14c |
| pcAttributeNormalMul / Div | physical_attack_rating_mult_pvp +0x158 / physical_absorption_mult_pvp +0x15c |
| ChrParam.armorDefenseValue, armorMagicDefenseValue ... | EnemyParam.defence_physical, defence_magic ... |
| ChrCommonParam.receive*DamageRate (Rm) | absorption_slash / thrust / strike |
| ChrCommonParam.grabDamageRate* (Crm) | critical_absorption_slash / thrust / strike |
| ChrParam.bulletDamageRate* (pBm, eBm) | EnemyCommonParam.bullet_absorption_* (the common param, not ChrParam; the guide's list also leaves out lightning) |
| WeaponStatsAffect.magicX, baseValueScale | WeaponStatsAffectParam.level_X_magic_scaling, base_damage_mult |

## Confirmed (PC Scholar regulation unless stated)

| claim | data |
|---|---|
| PvE pMod 6, pCut 6; PvP 10 / 12; PvP eMod = eCut = 6 per element | exact (DamageAdjustParam) |
| hMod right 1 / left 0.9; Majestic Greatsword 1 / 1 | 97 weapon types 1 / 0.9; Majestic 1 / 1 |
| bows, greatbows, crossbows 0.9 / 0.9; Avelyn 0.865; catalysts 1 / 1 | exact (15 bow types, Avelyn, every staff / chime / pyromancy flame) |
| Cm = counter strength / 100, plus a per-attack factor | counter_damage 1.0-1.6; counter_damage_mult on each attack |
| players: Rm 1, Crm 0.8 | ChrCommonParam: absorption 1 / 1 / 1, critical 0.8 x 4 |
| ranged absorption: base enemies 1, DLC mostly 0.8 | placed base enemies 569 / 666 at 1.0; DLC enemies 137 / 159 at 0.8 |
| ammunition table (atkAR, Dm) | every value exact: arrows x1.176 (Wood 0, Iron 30, Magic / Fire / Dark 110, Lightning 90, Iron Greatarrow 50, Lightning 150, Fire 100), bolts x0.392 (200, Heavy 280) |
| bolts in powerstance: Dm -37 % | second row set at x0.247 = 0.63 x 0.392 |
| the arrow AR in the stats screen is a dummy | ArrowParam's shown values (Wood 50, Iron Greatarrow 200 ...) are not the damage rows' values (not traced in code) |
| [pre-patch Lightning Spear](../builds/balance-vanilla-vs-scholar.md#5-spells) 32140000 atkAR 200 x1.25, AOE 32140010 atkAR 0 x0.25 | exact in PS3 v1.00 and PC 2014-03-31 to 2014-06-02; from PC 1.12 / Scholar atkAR 0, x1.5 / x0.45 |
| different hits in one swing use different rows (Pate's Spear 2h run 10020210 / 10020211) | x0.426 / x0.710 |
| buff table (bFlat, bMod) | all exact in SpEffect*.emevd (100090:2 = flat, 100080:1 = multiplier %): Magic Weapon 30 / 0.15, Great Magic Weapon 30 / 0.2, Crystal Magic Weapon 15 / 0.3, Sunlight Blade 15 / 0.3, Flame Weapon 15 / 0.2, Dark Weapon 15 / 0.3, Resonant Weapon 0.35 / 0.3 / 0.15 / 0.1 / 0.1 (60 / 60 / 30 / 20 / 10 s), Defender Greatsword 70 / 0.5, Watcher Greatsword 10 / 0.7, Possessed Armor Sword 50 / 0 (fire, 25 s), elemental resins 40 / 0.2 |
| element scaling 0.4 uninfused, 0.6 infused | the commonest values at +10 (60 / 96 and 34 / 95); the rule is infused = uninfused x 1.5 (+0: 0.35 -> 0.525; +5: 0.378 -> 0.566), as in [scaling letters and infusion](scaling-letters-and-infusion.md) |
| Red Tearstone Ring x1.2 | one x1.2 effect (SpEffectRing 1000:1 index 27, Blue Tearstone index 28 x0.65); which terms it multiplies is in code |
| "tip hits" depend on distance from the attacker | matches [distance damage cut](distance-damage-cut-pc.md) (attacker origin to target origin, code-traced) |

## Where the data adds detail

1. **Distance cut size.** The guide gives a cut of up to 70 % for most attacks. On player attacks the HP ramp field
   (+0xac, traced in code in [distance damage cut](distance-damage-cut-pc.md)) is -30 on 1,708 rows, -50 on 157, -20 on 153 and -70 on 75.
   -70 is the usual value of +0xb0 / +0xb4 / +0xb8 (1,249 rows), the ramp that code applies to three other hit values
   (posture, poise and one unidentified). So the 70 % may come from the poise / posture cut, with the HP cut mostly
   30 %. The planned live test (two enemies, one past the band) would settle it.
2. **Elemental bolts and the Destructive Greatarrow have a second damage slot.** Magic / Lightning / Fire / Dark Bolts
   carry 200 of their element and 200 physical; the guide gives the elemental 200. The Destructive Greatarrow carries
   [100 Break (type 8)](../network-test/broken-status.md#3-break-damage-damage-type-8-in-retail) and 100 physical; the guide gives the 100 physical. How the game adds the two slots is untested.
3. **Exceptions to the general rules.**
   - [Hand multipliers](community-claims-2026-11-02.md#2-left-hand-attacks-cost-more-stamina-and-deal-less-damage-true-for-nearly-every-weapon): the general 1 / 0.9 has more exceptions than the Majestic Greatsword. 30 weapon types are
     1 / 1 (catalysts, torch, binoculars), 15 are 0.9 / 0.9 (bows) and Orma's Greatshield is right 0.9 / left 1.
   - Enemy Crm 1: true of 522 of 666 placed base enemies; 110 are 0.9 and 34 are 0.6-0.8.
   - Bosses 0.85 ranged: true of the main bodies in `bosses_pc.tsv` (Smelter, Last Giant, Giant Lord, Pursuer,
     Rotten, Throne Watcher, Vendrick, Ancient Dragon, Nashandra, Flexile Sentry); Royal Rat Authority is 0.9 physical,
     and adds such as Freja's spiders and the Skeleton Lords' skeletons are 1.0. [That table](bosses-pc.md) only lists fights with a
     balance change, so this is not a full boss list.

## Version notes (outside the guide's scope)

- PS3 v1.00 (2026-11-01, [Mundane at SL50](mundane-sl50-v100.md)): the menu arrow AR is a dummy there too (ArrowParam's
  damage block is read only by the item-info code `FUN_0053eee4`). v1.00 bolts: flat 20 Wood / 50 Heavy / 110 elemental,
  Dm 0.98, powerstance rows 0.686 (-30 %; [0.882 from PS3 1.02](../builds/1.01-to-1.03.md#101-102-a-balance-patch-calibrations-102-103)). The rework to 200 / 280 / 200 + 200 and 0.392 / 0.247
  shipped in [PC regulation 1.06](../builds/early-pc-builds.md#102-103) (2 Jun 2014), with Avelyn's hand multiplier 0.9 -> 0.865.
- The PvP fields (pMod 10, pCut 12, element 6) only exist in the 396-byte DamageAdjustParam of PC 1.12, PS3 Scholar
  and PC Scholar. PS3 v1.00 and PC up to 2014-06-02 have a 344-byte row that ends before them, so the PvP formula
  doesn't apply to launch builds as written. What launch PvP used is unknown.
- In the same param, the [Company of Champions enemy damage went 1.35 -> 1.5](../builds/balance-vanilla-vs-scholar.md#6-damage-pvp-and-new-game) and nine unnamed values at +0xcc-+0xec went
  1.15 / 1.25 -> 1.05 / 1.15.

## Not checkable from the data

- [Critical tick counts at 60 FPS](community-claims-2026-11-02.md#6-ripostes-deal-more-damage-at-60-fps-than-at-30-fps-plausible-data-fits-the-explanation) (TAE damage events plus frame timing): needs a live test.
- Power Stance swing multiplier 0.9, the infusion match bonus (MatchBns, "Bandit Knife only 3"), and child bullets
  taking atkAR from their parent (the data fits: Lightning Spear AOE rows carry 0).
- The PvP elemental formula (divide by 1000, players' innate 10 %), and the PvP critical cuts (backstab 0.8, guard
  break 0.9, "older versions have none"): no such values in DamageAdjustParam; they may live in code or other params.
  (PlayerCommonParam has PvP backstab and riposte multipliers: see
  [balance: vanilla vs PS3 Scholar](../builds/balance-vanilla-vs-scholar.md#6-damage-pvp-and-new-game).)
- physical_damage_floor units: values are 70 (1,742 rows), 0, 40, 25 ...; the calculator uses a flat 11.5.
- Our open attack-rating question is a separate one (the guide works from the menu AR): [attack-rating live check plan](../tools/ar-live-check-plan.md) still
  applies.
