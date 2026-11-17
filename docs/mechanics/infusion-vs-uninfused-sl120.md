# Infused or uninfused at SL120? (PC Scholar, 2026-11-04)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question: at level 120, which weapons deal their most damage infused? Infused weapons take spell buffs only; uninfused
take a spell **or** a resin. Community claim: about 80 % are better infused.

## Method

- Engine = the Damage Calculator v2.44 MeleeAR formulas, read from the calculator's hidden `Weapons` sheet (one row per
  weapon x infusion, with legal-infusion and resin / spell flags). Checked against `tools\ar_predict.py` and the claws
  note: same numbers (Dagger 101, Fire Dagger 56.4 + 70.6, Dark Manikin 397 / uninfused 279, Dark Malformed 347 ...).
- Scope: the calculator's 207 melee weapons (shields, catalysts, bows, crossbows left out).
- AR = physical + magic + lightning + fire + dark. Poison / bleed build-up is not damage, so Poison and Bleed
  infusions only compete on their physical part. No rings, no Sacred Oath, no enemy defence, one-handed AR
  (2h only halves the STR requirement, as the calculator does).
- Every weapon at its max upgrade (+10 on all infusions in Scholar data).
- SL120 from a Deprived start = 114 free points over 6 in each stat. R points are reserved for VGR / END / VIT / ATN /
  ADP (headline R = 40), the rest is placed on STR / DEX / INT / FTH to maximise that weapon's AR (exact search).
- Weapon requirements, spell requirements (Magic Weapon INT 10, Great 18, Crystal 28, Sunlight Blade FTH 36,
  Dark Weapon 16 / 14, Resonant 15 / 24) are enforced.
- Mundane: bonus uses the lowest of all nine stats, so all nine are levelled together (about 18 each at R = 0).
- Buffs tried: Magic / Great Magic / Crystal Magic Weapon, Sunlight Blade, Flame Weapon, Dark Weapon, Resonant Weapon
  (100 % durability); resins Aromatic Ooze, Gold / Charcoal / Dark Pine. Uninfused row may use any legal spell or resin,
  infused rows any spell (the calculator's resin flag is N on every infused row).

## Result

| set-up | infused best | uninfused best |
|---|---|---|
| all buffs, R = 40 (headline) | 191 / 196 (97.4 %) | 5 |
| all buffs, R = 0 | 191 / 196 (97.4 %) | 5 |
| all buffs, R = 70 | 187 / 193 (96.9 %), 3 weapons unreachable | 6 |
| no Resonant Weapon, R = 40 | 191 / 196 | 5 |
| no buffs at all, R = 40 | 184 / 196 (93.9 %) | 12 |

- Median infused advantage with buffs (R = 40): +25 % (p10 +7 %, p90 +35 %).
- Uninfused wins (R = 40, with buffs): Chaos Rapier (+7.5 % uninfused, Charcoal Pine Resin), Sanctum Mace (+6 %),
  Caestus (+3 %), Bound Hand Axe (+2 %), Ivory King Ultra Greatsword (+2 %, no buff). Crescent Axe and Sorcerer's
  Twinblade are within 1.2 % the other way.
- Winning infusion among infused-best weapons: Dark 135, Magic 31, Fire 10, Lightning 8, Raw 5, Mundane 2.
- Non-infusable boss weapons (melee list): **11** - Mytha's Bent Blade, Umbral Dagger, Vanquisher's Seal, Key to the
  Embedded, Chariot Lance, Scythe of Want, Dragonslayer Spear, Ivory Straight Sword, Aged Smelter Sword, Smelter Sword,
  Notched Whip. (Excluded from the percentages.)

Per category, nearly all are 100 % infused-best (Axe 7/8, Fist 1/2, Greathammer 16/17, Thrusting 8/9, UGS 11/12).

## Caveats

- Much of the gap is the calculator's buff model: every spell / resin adds `bMod x (bFlat + scaling + the weapon's
  reinforce-table base for that element)` even on a weapon with no such element, and a matching infusion adds a
  share bonus on top. Whether the base term is real on an uninfused weapon is the open question 6 in
  `ar-live-check-plan.md`; the live check has not been run. If it is smaller than modelled, uninfused loses more.
- Dark infusion wins 135 mainly through Resonant Weapon / Dark Weapon, but removing Resonant changes nothing at the
  headline (still 191).
- 97 % (93.9 % with no buffs) is well above the community ~80 %. The claim probably uses lower investment, other stat
  splits or a different weapon pool (shields, bows, boss weapons counted as not-infused would give 191 / 207 = 92 %).
- Data only, nothing tested in game.

Files: `infusion-vs-uninfused-sl120-R40.tsv` (every weapon, best set-up each side), scripts `tools\infusion_sl120\`.

## Addendum 2026-11-04: against enemy resistances

Enemies sheet of the calculator (Evan's export): 644 PvE rows, 2283 spawns. Per hit, as the calculator's WeaponDamage sheet:
physical = max(11.5, (phys AR - DEF / 6) x slash/thrust/strike mean), element = AR x (1 - element defence %). Averages
(spawn-weighted): flat DEF 490, defence Magic 26.4 %, Lightning 29.6 %, Fire 30.6 %, Dark 31.4 % (unweighted 28.8 / 29.8 /
31.9 / 31.9). So Dark does ~7 % less per AR point than Magic. Same R = 40 set-up, configs re-picked for the enemy mix.

| | raw AR | vs enemies (spawn-weighted) |
|---|---|---|
| infused best vs uninfused best, mean / median | +24.9 % / +25.9 % | +13.8 % / +11.1 % |
| infused ahead | 191 / 196 | 174 / 196 |
| Dark vs best of Magic / Lightning / Fire, mean | -0.3 % (Dark ahead 133) | -3.6 % (Dark ahead 6) |

- Dark's 135 wins were razor-thin: its lead over the best other element is about 0.3 % in AR, so the ~7 % resistance gap
  flips it. Under the enemy mix the best infusion is Magic 146, Raw 8, Fire 7, Lightning 6, Dark 5, Mundane 2.
- Infusion still beats uninfused: the flat DEF / 6 comes off the physical part only, which hurts uninfused weapons more.
- 22 weapons flip to uninfused (worst Caestus -24 %, Bound Hand Axe -13 %, Chaos Rapier -12 %, Puzzling Stone Sword -11 %).
- Unweighted enemies: Dark ahead on 8 / 196, mean -2.4 %. Per-enemy variation (Dark-weak enemies exist) and bosses not split out.
Scripts: tools\infusion_sl120\ enemies.py, run2.py, an2.py, an3.py.
