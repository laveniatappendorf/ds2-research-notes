# Scaling letter grades, DEX's own scale and elemental infusion (2026-10-26)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Prompted by a YouTube comment on DS2's weak scaling / strong infusion (the author asked: does it hold in the Network Test
and in v1.00, and is the reason given for DEX's different grading supported?). Static data from every build, plus
the letter-grade code in the PC exe and the Network Test EBOOT. Tool: `tools\scaling_compare.py`
(`thresholds | weapons | infusion | bonus`).

## The comment's claims, checked

| claim | Network Test | v1.00 | PC SOTFS |
|---|---|---|---|
| STR letter cut-offs are shared by Magic / Lightning / Fire / Dark, DEX has its own | **no**: all 5 rows identical | yes | yes |
| Dagger +10 = 45 % DEX = A | n/a | yes (exactly on the A cut-off) | yes |
| ... and would be D on STR's scale | | **no: C** (C = 35-55 %) | **no: C** |
| Twin-headed Greatbow +5 = 135 % STR, the highest | n/a | not in v1.00 (DLC); top = Large Club 120 % | yes |
| elemental infusion shows it as 101.3 % = S | n/a (stones planned, no working infusion) | infusion rules identical | yes (1.0125 >= 1.00) |
| ... with 55 % of the damage moved to the element | n/a | | yes (infusion_mult 55) |
| ... so effective STR scaling = 101.3 % x 45 % | | | arithmetic 45.6 % (comment says 45.8); the model itself **not verified in code** |
| DS2 scaling bonus isn't multiplied by base damage | | consistent with the data (one flat stat-bonus table for every weapon), formula not traced in code | |

## Letter cut-offs (FeWeaponThreshold)

v1.00 = 1.10 = PS3 Scholar = PC (never changed after v1.00). Row names are the developer names (SmithBox DS2S
Developer Row Names).

| row | name | S | A | B | C | D |
|---|---|---|---|---|---|---|
| 1 | 筋力 STR | 100 % | 80 | 55 | 35 | 20 |
| 2 | 技量 DEX | 60 | 45 | 35 | 25 | 15 |
| 3-6 | 魔法 magic, 炎 fire, 雷（信仰）lightning, 闇 dark | 100 | 80 | 55 | 35 | 20 |
| 7 | 魔力 | 50 | 40 | 30 | 20 | 15 |
| 8 | 耐久力 | 100 | 52 | 35 | 25 | 21 |

PC code: `FUN_1400404d0(value, row)` (row via `FUN_1404ffc60` = menu-param holder `[DAT_1416148f0+0x22e0]+0x290`,
loaded from `param:/FeWeaponThreshold.param` in `FUN_140502a90`): returns the first grade whose cut-off the value
reaches (`>=`); above 0 but under D = 5 (E); 0 or no row = 6 (no grade). Callers: the weapon screen
`FUN_140037720` maps menu fields 0x29 STR -> row 1, 0x2a DEX -> 2, 0x2b magic -> 3 (**row 7 when the weapon's
infusion type is 8**, Enchanted in CustomAttrSpecParam's flag order), 0x2c -> 5, 0x2d -> 4, 0x2e -> 6. Row 8 is not
used there. A second caller `FUN_14034fd40` turns the grade into a multiplier from a 6-float table (+0x13c..+0x150 of
an unidentified param row): not followed.

Network Test (US and JP identical): 5 rows, all S 200 %, A 160, B 120, C 80, D 40. Code `0x2c3b18` (same loop,
row from `[[..]+0x21d0]` via `0x8bc998`): **no E**: a value above 0 but under D returns 6, the same as 0. Callers:
the menu routine around `0x2c06dc` (switch at `0x2c017c`, index = field - 0x1f): fields 0x37-0x3a -> rows 1-4, field
0x1f -> row 5; values from the getter `0x2bbc38`, weapon record `[info+0]` +0x50, +0x54, +0x58, +0x5c and +0x38. What
fills those was not traced (the record is built at runtime, not a param row). Loader: `0x7f8980` /
`title:/param/FeWeaponThreshold.param` (TOC slot 0x1a22ed4).

## Infusion

Network Test: **no working infusion machinery**: no CustomAttrSpecParam / CustomAttrCostParam, and
WeaponStatsAffectParam has 16 shared curves (ids 0-12, 1010-1050, 2010-2060; 12 x 9 floats, no per-weapon or
per-infusion rows). Only the 27 handed-out weapons (8400-8426) have their own reinforce rows; 417 rows use the
placeholder curve 1010. **But infusion was already planned** (corrected 2026-10-26, same day):
- the stones are ItemParam rows with Japanese names and short descriptions (no English): 星光石 Faintstone 61060000
  "gives equipment the 魔力 (magic) attribute", 月光石 Boltstone lightning, 火竜石 Firedrake fire, 暗月石 Darknight
  dark, 毒塊石 poison, 血塊石 bleed, plus two cut ones: **溶解石 61120000 "gives equipment the 破壊 (destruction)
  attribute"**, and two catalyst crystals, 五色の結晶 61040000 and 虹色の原結晶 61050000 ("a stone that acts as the
  catalyst for giving equipment an attribute"). Retail drops the destruction stone and the crystals and adds Raw,
  Magic (= Enchanted), Old Mundane and Palestone;
- resins: Gold / Charcoal / Dark / Rotten Pine Resin as in retail, 龍の黄水 60290000 (bleed; retail Bleeding Serum),
  and a cut 龍の紅液 60300000 "temporarily gives the right-hand weapon a destruction effect";
- WeaponReinforceParam already has the per-damage-type block (physical, magic, lightning, fire, dark ...; staves
  55 magic / 45 fire, bells 55-60 lightning / 40-45 dark, Dragonknight's Shield 80 physical / 20 lightning) and,
  where retail keeps the infusion shares, **two 9-wide blocks** (30 / 60 on most weapons, 25 / 50 staves, 20 / 40
  bells, 10 / 20 the Dragonknight's Shield, 35 / 70 Golden Wing Shield): uniform across elements, the second always
  double the first. Possibly the element share at two strengths (two catalyst crystals?): a guess, not traced.
- The NT ships no blacksmith menu text (no npcMenu.fmg), so the missing infusion menu proves nothing.
- The destruction stone belongs to a whole cut element, **Break**: see `break-element.md`.
- The two catalyst crystals: see `cut-infusion-crystals.md` (rainbow crystal = Bell Keeper invasion reward; its icon
  survives in retail as the Magic Stone's picture).

Retail has **no unused infusion types** (checked 2026-10-26 in v1.00, PS3 Scholar and PC): CustomAttrSpecParam uses
exactly 10 bits (none + 9; every bit set on 61-93 of 94 rows, no higher bit anywhere), CustomAttrCostParam has exactly
10 slots (Palestone + the nine stones, all real items), the infused-name tables (common.fmg 20059202-20059210,
npcMenu.fmg 59202-59211) hold the nine plus "None", and the stone id range 61060000-61160000 has only the one gap at
61120000 (the Network Test's Break stone). The two Network Test catalyst crystals (61040000 / 61050000) left nothing.
Only placeholder reinforce rows carry petrify / curse damage types (row 4; Binoculars' row 6100 = 100 % petrify with
zero damage).

v1.00 and PC: WeaponStatsAffectParam row = stats id + infusion slot. Elemental slots 1-4 multiply STR and DEX
scaling by **0.75** on every weapon (v1.00 648/648; PC 742/752, the rest rounding plus 2 zeros), element scaling
x1.5, base damage x1.4 for most (v1.00 556 of 757). WeaponReinforceParam `infusion_mult_<element>` = the element's
share: 50 % for most weapons, 55 % for some (Twin-headed Greatbow), 30 % / 20 % for a few. Same structure in v1.00 and
PC; ~54 weapons' infused rows were rebalanced by PC, and PS3 Scholar nudged infused fire / lightning / dark scaling up
a few per cent (balance-vanilla-vs-scholar.md).

## Why DEX has its own scale

Stat-bonus tables (PhysicalStatsPerLevelStatValues): **STR and DEX are identical** (1: 50, 20: 80, 40: 140, 50: 155,
99: 200), so 1 % of DEX scaling is worth exactly 1 % of STR scaling; the lower DEX cut-offs are display only, and a
DEX "A" is worth less than a STR "A". Fire and dark rise faster early.

Weapons (uninfused, max upgrade, one row per name; melee = id blocks 10-36, 50-53, 55):

| | v1.00 | PC |
|---|---|---|
| STR median / DEX median (weapons that scale) | 0.50 / 0.25 | 0.50 / 0.25 |
| DEX letters on DEX's scale | S5 A31 B18 C42 D25 E23 | S17 A40 B22 C37 D28 E25 |
| DEX letters on STR's scale | S2 A0 B11 C41 D52 E38 | S3 A4 B30 C42 D48 E42 |
| STR letters | S3 A21 B33 C52 D22 E31 | S10 A37 B32 C49 D25 E35 |
| DEX-leaning melee: median max physical / weight / main scaling | 210 / 4.5 / 0.43 (n 59) | 220 / 4.5 / 0.48 (n 73) |
| STR-leaning melee | 255 / 8.0 / 0.55 (n 104) | 280 / 9.0 / 0.55 (n 115) |
| (STR + DEX) x 140 / max physical, DEX- vs STR-leaning | 0.39 vs 0.40 | 0.44 vs 0.41 |

Supports the comment: DEX weapons are lighter, hit for less and carry less scaling, and scaling looks tuned to base
damage (the bonus at 40/40 is ~40 % of max base physical for both groups). Without its own scale DEX would show
almost no S or A (v1.00: two S, both 130 % weapons, no A). Nuance: the DEX scale (60 % of STR's at S) is more
generous than the DEX-vs-STR weapon gap (main scaling 0.43 vs 0.55 = 78 %); it tracks the spread of all DEX
coefficients (median half of STR's, including the small DEX values on STR weapons), so the letters even out the
distribution rather than the value. v1.00 had two 130 % DEX weapons (Ricard's Rapier, Blacksteel Katana), cut to
100 % by PC.

## Not established

- The attack-rating formula (flat bonus = coefficient x table? does the infusion split scale the physical bonus?):
  the PC code around it is behind the exe's anti-tamper stubs (jump-obfuscated second .text); PS3 v1.00 code is
  clean but was not searched. Candidate next step: a live read of the weapon screen (AR + letters) at known stats.
- What the Network Test's five graded values are, so whether NT scaling was "weak".
