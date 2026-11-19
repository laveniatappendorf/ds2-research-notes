# SL1, +0 weapons, no rings, no armour: best damage (2026-11-15)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author): best damage output for an SL1 character with no weapon upgrades, rings or armour.

Data: PC Scholar weapons through the community Damage Calculator v2.44 formulas ([engine](infusion-vs-uninfused-sl120.md) from
`tools\infusion_sl120`, scripts in `tools\sl1_naked`). [Vanilla PC 1.12 weapon data is identical](../builds/damage-calculator-vanilla.md). PS3 v1.00 not checked.
Nothing tested in game.

Stats: SL1 = Deprived, all nine stats 6. At stat 6 every scaling stat already gives a bonus of 52-63
(STR / DEX 54, magic 52, lightning 54, fire / dark 63), so scaling still counts at SL1.
Two-handing halves the STR requirement (community rule, as the calculator does), so 2h reaches STR req 13.

## Weapons you can wield (1h needs STR 6 / DEX 6; 2h STR <= 13, DEX <= 6)

| | weapon | AR |
|---|---|---|
| best plain | Puzzling Stone Sword (2h; STR 7 / DEX 6) | 174 |
| | Mace (2h) | 166 |
| | Hand Axe (2h) | 162 |
| best one-handed | Broken Straight Sword | 96 |
| best infused | Fire / Dark Hand Axe (2h) | 216 (105 phys + 111 element) |
| | Lightning Hand Axe | 213 |
| | Fire / Dark Puzzling Stone Sword | 204 |
| | Fire / Dark Mace | 202 |
| best with a resin | Puzzling Stone Sword + Charcoal / Dark Pine Resin | 210 (calculator's resin formula, unverified) |

Raw, Enchanted and Mundane all lose at SL1: they trade away the stat-6 scaling (Mundane bonus at lowest stat 6 = 80).

Vanquisher's Seal (calculator "fist weapon", 254 AR, no requirements) is the Company of Champions ring
(ItemParam 40760000, "greatly increases attack power of bare hands"), so it is excluded.

## Per hit (calculator PvE model: physical (AR - DEF/6) x MV x type, element AR x (1 - res %) x MV)

| weapon | 2h R2 MV | Hollow Soldier (317 / 15-30 %) | Pursuer FoFG (514, lightning 10 %) | Lost Sinner (363) | 2h jump (MV 1.512) |
|---|---|---|---|---|---|
| Lightning Hand Axe | 1.44 slash | 184 | 166 | 173 | 193 / 175 / 182 |
| Fire Hand Axe | 1.44 | 186 | 137 | 143 | 196 / 144 / 151 |
| Mace (plain) | 1.44 strike | 163 | 139 | 152 | 171 / 146 / 159 |
| Puzzling Stone Sword (plain) | 1.2 slash | 145 | 99 | 136 | 183 / 124 / 172 |

Puzzling Stone Sword's 2h R1 is a thrust (MV 1.176): 192 on Hollow Soldiers (thrust x1.35).
Elemental splits win at SL1 because physical defence is subtracted (DEF/6, 53-86 here) while elements are cut by a
percentage. **Best all-round: Lightning Hand Axe, two-handed.**

## Under requirement

[PlayerLackOfStatsParam row 0](armour-requirement-penalties.md) (attack down 30-80 %; also slower wind-up / recovery, +30-50 % stamina). How the weapon
shortfall feeds it is not traced. With penalty = 30 % + 50 % x worst fraction missing (the model used for the
[Large Club](sl1-large-club-vs-old-knight-hammer.md#ps3-v100)), the best is Smelter Hammer at ~161 AR, below the Hand Axe and slower. Only a flat 30 % would change the
answer (Fire Smelter Hammer ~395): one in-game look at the AR shown at SL1 would settle it.

### Live check (PAUSED 2026-11-15 at the author's request, nothing measured yet)

Tool: `tools\lackcheck.py` (info / read / sweep / give / run). Done so far, on the all-99 test character in
Huntsman's Copse (m10_15): Old Knight Hammer, Smelter Hammer, Uchigatana and Handmaid's Ladle given to the bag;
a DEX sweep 99 -> 1 left the attack block group A ([[PlayerCtrl+0x230]]+0xF08+0x1C) at 153 throughout, so it is
NOT recomputed on a stat change (probably filled per swing; unconfirmed). DEX put back to 99.

Resume:
1. Start the recorder (background), then the author equips the Old Knight Hammer in right-hand slot 1, one-handed:
   `uv run --no-project --python 3.12 python -I -X utf8 tools\lackcheck.py run STR 99,30,29,27,24,20,15,10,6 2 60 okh-str 2730000`
   (waits for the weapon, three beeps; one R1 after each high beep; two low beeps = done; CSV in findings\lack-live\).
2. If group A doesn't refill on an air swing, or doesn't drop below 250 when short: measure HP drops on an enemy
   instead (Arena.chrs_of / set_hp in DS2 Boss Arena\tools\arenalib.py; damage = (AR - DEF/6) x MV).
3. Then Uchigatana DEX 16 -> 6 (STR 99) for the DEX side, Smelter Hammer at all 6 one- and two-handed for the SL1
   case, and OKH two-handed at STR 15 / 14 to confirm the halved requirement.

## Off-hand stat item (not a ring or armour)

Vessel Shield in the left hand: STR / DEX 10, other stats 7. Best: Fire / Dark Craftsman's Hammer 2h 265 AR, plain
Craftsman's Hammer 209. Assumes the shield's bonus still counts while two-handing (not checked).

## Throwables (calculator ConsumablesDamage sheet at stat 6)

Black Firebomb 120 + 0.75 x fire bonus = 167 fire AR; the calculator says it hits twice (contact + explosion).
Firebomb 141, Lightning Urn 135, Throwing Knife 180 + 0.26 x DEX bonus = 194 physical.
