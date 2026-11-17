# Mundane at SL50 before Shrine of Winter, PS3 v1.00 (2026-11-01)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Question (the author): was Mundane "busted" at launch, and do fast weapons like (dual) daggers get very high DPS from its
flat bonus? Scope: SL50, weapons obtainable before Shrine of Winter, BLUS41045 v1.00 data. Static only: nothing was
tested in game.

## Verdict

At SL50 Mundane is **not** busted. The best Mundane option (two-handed Dagger R1 chain) roughly ties an uninfused
Dagger on a DEX build, beats normal builds only on the weakest daggers (Thief Dagger, Bandit's Knife, Manikin Knife,
Broken Thief Sword), and does about half to two-thirds of the DPS of ordinary SL50 builds with early halberds,
katanas or axes. The legend fits higher levels better: the bonus keys off the *lowest* stat, so it grows with SL
while a DEX build caps at DEX 99 (Dagger: Mundane = DEX build at SL50, +8 % at SL100, +14 % at SL150, +20 % at
SL200, with every stat balanced). v1.00 also gave every weapon type the full bonus; later patches cut some types.

## Mundane in v1.00

- Bonus = `PhysicalStatsPerLevelStatValuesParam.stat_mundane_scaling` at the lowest of the nine stats: 1 -> 50,
  10 -> 110, 11 -> 116, 20 -> 170, 30 -> 200, 40 -> 210, 99 -> 250. The table is identical in every build checked
  (v1.00, patches 1.01-1.10, PC vanilla 2014-2022, PC Scholar). **Code-confirmed in v1.00** (2026-11-01): the stat
  builder `FUN_008609f0` loops over the attribute array (0x861208-0x8612cc: start 99, keep the smaller, skip the
  10th / 11th entries), looks up that row and stores column +0x8c to out +0x2c. The array is the builder's `attrs`
  argument, the effective attributes (base + equipment bonus, clamped 1-99, agility-iframes.md), so gear stat
  boosts count toward the lowest stat. Where the weapon side multiplies it (and whether v1.00 has any type factor)
  was not traced.
- No item or menu text in any build says which stat Mundane uses: the Old Mundane Stone only says "power of the
  mundane" (Japanese 無明, "unenlightenment"). The highest -> lowest text change in patch 1.03 is the **Dark** bonus
  help (iconHelp 909190), and that was a text fix: v1.00 code already used the lower of INT / FTH
  (`DS2 Vanilla Patches\findings\1.01-to-1.03.md`).
- Mundane infusion row (WeaponStatsAffectParam stats id + 9): base x0.5, STR / DEX scaling halved.
- `WeaponTypeParam.mundane_scaling_mult` is 0 on every type in v1.00-1.03 and first has values in 1.08 (100 on most,
  47-80 on a few, 0 on bows / crossbows in PC Scholar). Mundane obviously worked at launch, so v1.00 code cannot have
  multiplied by that field: **every weapon got the full bonus** (inferred; the v1.00 code was not traced). Community
  reports: a PC "1.06" update cut Mundane Avelyn 30 % and Mundane Santier's Spear 21.5 % (gameranx; Santier's
  confirmed below), Calibrations 1.11 cut Mundane on several daggers (PCInvasion).
- Old Mundane Stone (61150000) in v1.00: no ground pickups; Stone Trader Chloanne sells 3 (ShopLineupParam 76200617,
  behind flag 100975); drops from Gyrm Warriors (Doors of Pharros, lots 13500000-13505000), Undead Steelworkers
  (Harvest Valley, 14900000), Primal Knights (Drangleic Castle, post-Shrine) and 1271400. Infusion by McDuff.

## SL50 stats

Every class has stat total = SL + 53, so SL50 = 103 points: all nine stats 11 with 4 spare (lowest stat 11 ->
+116). The spare points cover requirements (two-handing counts STR x1.5) or go to DEX. **Caveat (added 2026-11-01, later):** the community rule is that two-handing halves the STR requirement and adds no STR scaling (`sl1-large-club-vs-old-knight-hammer.md`), so the two-handed figures below (requirements and STR scaling at x1.5) need redoing.

## Method

- AR (community calculator v2.44 model): max physical x infusion base mult + STR coef x STR bonus + DEX coef x DEX
  bonus (+ Mundane bonus); innate elements x base mult added flat. Tool: scratchpad `mundane50.py`, `dps2.py`.
- Damage per hit vs NPCs: max(11.5, AR - defence / 6) x motion value (calculator's NPC formula; defence in physical
  points, Pursuer 514, Heide Knights ~400-540, Smelter Demon 664 per the calculator's Enemies sheet).
- Swing time from the player TAE (`timeact\c000100_pl.tae`): first hit window (event 2200) / startup_speed +
  stretch to the follow-up window (event 111500 whose 5th parameter word is 1) / recovery_speed, averaged over the
  category's light_1 and light_2 rows (WeaponActionCategoryParam -> WeaponAttackMotionParam). **Assumption**: that
  111500 variant is the next-attack window (the other variant sits between the two hits of the dual-dagger
  powerstance attack). Treat swing times as +-30 %. One hit per swing.
- Normal builds: uninfused, best STR / DEX split with STR + DEX = 61 (others 6: "all-in") or 41 (VGR / END / VIT /
  ADP 11 like the Mundane build: "balanced").
- Availability: v1.00 ground pickups (`DS2 Scholar\findings\item_placements.tsv`), ShopLineupParam (NPC, boss-soul
  trades) and ItemLotParam2_Chr drops mapped to enemy families per map (`enemy_counts_families_v110_vs_pc.tsv`).
  Excluded as post-Shrine: Avelyn (Undead Crypt), Lucerne / Yellow Quartz Longsword (Wellager), Drakekeeper weapons
  (Dragon Aerie), Giant Warrior Club (memories), Curved Dragon Greatsword (Ancient Dragon soul).

## Results (DPS = R1 chain, vs defence 0 / 300 / 514)

| weapon (grip) | source | Mundane AR | Mundane DPS | all-in AR | all-in DPS | balanced AR | balanced DPS |
|---|---|---:|---|---:|---|---:|---|
| Dagger (2h) | Things Betwixt | 194 | 784 / 582 / 438 | 195 (DEX 50) | 789 / 587 / 443 | 178 | 719 / 517 / 373 |
| Manikin Knife (2h) | Manikins, Harvest Valley | 184 | 744 / 542 / 398 | 162 | 657 / 455 / 310 | 150 | 604 / 402 / 258 |
| Broken Thief Sword (2h) | Heide's Tower | 186 | 751 / 549 / 405 | 181 | 731 / 529 / 385 | 170 | 688 / 486 / 342 |
| Bandit's Knife (2h) | Huntsman's Copse | 178 | 618 / 444 / 321 | 162 | 561 / 388 / 264 | 151 | 525 / 352 / 228 |
| Varangian Sword (2h) | Varangian Sailors, Wharf | 267 | 658 / 535 / 447 | 356 | 879 / 755 / 667 | 342 | 842 / 718 / 630 |
| Heide Lance (2h) | Old Knights, Heide's Tower | 304 | 639 / 534 / 459 | 428 | 899 / 794 / 719 | 412 | 866 / 761 / 686 |
| Broadsword (2h) | Lenigrast | 260 | 642 / 518 / 430 | 350 | 863 / 740 / 652 | 334 | 823 / 700 / 612 |
| Halberd (2h) | Forest of Fallen Giants | (STR 20) | n/a | 460 | 1028 / 916 / 837 | 444 | 991 / 879 / 800 |
| Uchigatana (2h) | McDuff | (DEX 16) | n/a | 322 | 859 / 726 / 630 | 298 | 794 / 661 / 565 |
| Blacksteel Katana (2h) | Alonne Knights, Iron Keep | (DEX 25) | n/a | 392 | 1044 / 911 / 816 | 326 | 871 / 738 / 643 |
| Black Knight Halberd (2h) | Shaded Woods | (17 / 26) | n/a | 567 | 1267 / 1155 / 1076 | 530 | 1184 / 1073 / 993 |
| Mastodon Halberd (2h) | Hollow Primal Knights, Pharros (drop) | (26 / 15) | n/a | 571 | 1275 / 1163 / 1084 | 539 | 1205 / 1093 / 1014 |

(n/a = the weapon needs more STR / DEX than an all-11 build can reach with 4 spare points; these are mostly
Mundane-infusable too.)

- Two-handed light attacks have higher motion values (Dagger 1.0 / 0.925 one-handed, 1.2 / 1.11 two-handed) and
  in this TAE reading the two-handed dagger chain is also faster (0.29 s vs 0.34 s).
- Dual Dagger powerstance (L1 10007200 / 10007300): two hits (MV 0.7 / 0.65) every ~0.53 s: about 460 / 340 / 260
  DPS with Mundane, below the two-handed R1 chain.
- Flat defence works against the "fast weapon" idea: each hit loses defence / 6 (Pursuer: -86 per hit), a third
  to half of a 190-AR dagger hit but a fifth of a 450-AR halberd hit.
- Flat scaling is not Mundane's own: DS2's STR / DEX bonus is also flat per hit (scaling-letters-and-infusion.md).
  What is special is that Mundane's bonus ignores the weapon's own coefficients, so weapons with low base damage and
  little scaling (daggers, thief weapons) gain most.

## Santier's Spear (added 2026-11-01)

- v1.00: 133 / 200 physical (+0 / +5, Twinkling Titanite), STR 20 / DEX 22, **no stat scaling at all**
  (WeaponStatsAffectParam 1000000: STR / DEX coefficients 0 in every slot). So the uninfused spear is 200 for any build,
  Raw 230 (x1.15), Mundane 100 (x0.5) + the bonus. Pickup in Doors of Pharros (lot 10335040), pre-Shrine. Row 3251000
  (same name, model 3251, shorter hitbox) is in no lot: unused duplicate.
- The requirements eat the stat budget: two-handed (STR 14 x1.5 = 21) + DEX 22 leaves 67 points for the other seven
  stats at SL50, so the lowest is 9 (+104): **Mundane 204 vs plain 200 vs Raw 230**. One-handed: lowest 8, Mundane
  198. Mundane passes Raw from about SL81 (lowest 14, 234); SL100 246, SL150 276, SL200 294, cap 350 (all stats 99).
- SL50 two-handed R1 DPS (0.543 s, MV 1.213, same TAE method) vs defence 0 / 300 / 514: Mundane 456 / 344 / 264, plain
  447 / 335 / 255, Raw 514 / 402 / 322. Below the Mundane Dagger (582 at 300) and far below the early halberds.
- Patch history: PC regulation 1.06 (2014-06-02, `DS2 Vanilla PC\2014-06-02`) moves it to WeaponStatsAffectParam
  1006030: Mundane base 0.5 -> 0.15 (Mundane = 30 + bonus), Raw 1.15 -> 1.1, base still 200. (100 + b -> 30 + b) is
  -21.5 % at b = 225 (lowest stat 50), which matches the gameranx figure. PS3 1.08 / PC 1.04 (July 2014) and later:
  base 200 -> 350, so Mundane = 52.5 + bonus, never better than plain (max 302.5 vs 350). PS3 1.03 still has the
  v1.00 values; PS3 1.04-1.07 are not obtainable, so the PS3 date of the cut is unknown.
- Scratchpad `santier_ar.py`. Gear that raises DEX (Cale's Helm +2, Work Hook +5 / ADP -3) frees at most a few points
  (lowest 10, about 210) and only if gear bonuses count toward requirements (not checked).

## SL120: Santier's Spear vs daggers vs Avelyn (added 2026-11-01)

Any weapon in v1.00 (no pre-Shrine limit). Scratchpad `dps3.py` (any SL: Mundane lowest stat = highest the requirements
allow; normal builds "all-in" = other seven stats 6, "balanced" = VGR / END / VIT / ADP at the Mundane build's lowest
stat, ATN / INT / FTH 6), `dps3show.py`, `avelyn120.py`. SL120 = 173 points, Mundane lowest stat 19 (+164), 18 (+158)
when requirements bite. Same model and swing-time caveats as above; bleed / poison not counted.

| weapon (2h R1) | Mundane AR | Mundane DPS (def 0 / 300 / 514) | best normal AR (all-in) | all-in DPS | balanced DPS |
|---|---:|---|---:|---|---|
| Royal Dirk | 264 | 1069 / 866 / 722 | 256 | 1035 / 833 / 689 | 960 / 758 / 613 |
| Shadow Dagger | 259 | 1046 / 843 / 699 | 255 | 1030 / 828 / 684 | 961 / 759 / 615 |
| Black Flamestone Dagger | 253 | 1023 / 820 / 676 | 243 | 983 / 781 / 637 | 900 / 698 / 554 |
| Dagger | 247 | 1000 / 798 / 653 | 228 | 921 / 719 / 575 | 834 / 632 / 488 |
| Broken Thief Sword | 240 | 970 / 768 / 623 | 185 | 748 / 546 / 402 | same |
| Manikin Knife | 237 | 957 / 755 / 611 | 187 | 757 / 554 / 410 | 699 / 497 / 353 |
| Thief Dagger | 224 | 777 / 604 / 481 | 135 | 468 / 295 / 171 | same |
| Santier's Spear | 258 | 577 / 465 / 385 | 200 (Raw 230) | 447 / 335 / 256 (Raw ~402 at 300) | same |
| Black Knight Halberd | 428 | 957 / 845 / 765 | 617 | 1378 / 1266 / 1187 | 1318 / 1206 / 1126 |
| Blacksteel Katana | 318 | 848 / 715 / 620 | 450 | 1201 / 1067 / 972 | 1110 / 977 / 882 |

- At SL120 Mundane daggers pass DEX builds (Dagger +11 % at defence 300 vs DEX 99, +26 % vs the balanced build) and
  Royal Dirk (2h) is the best Mundane weapon of all, but the best normal two-handed builds still do ~1.2-1.5x
  (Black Knight Halberd 1266 / 1206).
- Santier's Spear stays far behind the daggers: same AR as a Mundane dagger, half the swing rate.
- **Avelyn (corrected after the bolt trace below):** per bolt = (0.9 x (crossbow AR + the bolt's flat damage) -
  defence / 6) x 0.98, three bolts per shot, each carrying the crossbow's full AR. Mundane Avelyn +10 at SL120
  (STR 25 / DEX 15, lowest 18): AR 85 + 158 = 243 (plain 170, Raw 196, no scaling). Heavy Bolts, all three hitting:
  **775 / 628 / 523 per shot** (plain 582 / 435 / 330, Raw 650 / 503 / 398); Lightning Bolts against 0 % lightning
  resistance 934 / 787 / 682. Mundane Heavy Crossbow 286 / 237 / 202 per bolt. One Avelyn shot = about 2.8 Mundane
  Dagger hits at defence 300. Rough cycle 2.9 s (shot 0.4 s + reload 2.5 s animation; other crossbows 0.4 + 1.83 s),
  so ~215 DPS with Heavy Bolts at defence 300: burst and range, far below the daggers' DPS. Avelyn and the Black
  Flamestone Dagger are post-Shrine (Undead Crypt / Wellager). `avelyn120.py`.
- History: the reduced bolt rows are the **powerstance** (L1 / L2) ones, not Avelyn's: 0.686 in v1.00 / 1.01 (-30 %),
  0.882 from PS3 1.02 and on every PC build (-10 %), 0.247 vs 0.392 after the rework (-37 %, as Radai says). PC
  regulation 1.06 (2 Jun 2014) then cut Avelyn's Mundane base 0.5 -> 0.15 (WeaponStatsAffectParam 1006030, like
  Santier's), its hand multiplier 0.9 -> 0.865, and reworked every bolt (flat 20 / 50 / 110 -> 200 / 280 / 200 + 200
  physical, Dm 0.98 -> 0.392), so the crossbow's AR, and with it Mundane, counts 40 % as much. PS3 got the rework
  somewhere in 1.04-1.08. So Mundane Avelyn was at full strength from launch (v1.00 included) until that patch.
  gameranx's 834 -> 576 (-31 %) was measured across it.

## Bolt damage in v1.00 (traced 2026-11-01)

- **The arrow attack shown in the menu is not used for hits.** ArrowParam's damage block (+0x46 physical, +0x48
  magic, +0x4a lightning, +0x4c fire, +0x4e dark) is read only by the item-info code: `FUN_0053eee4`, called from the
  inventory-record builder at 0x535d08 (switch on item category, case 9 = ammo), plus a network record copier. The
  ArrowParam row getter is `FUN_007fff4c` (table [[App+0xc]+0x290]+0x8c); its callers are that menu code, network
  code and the equipment slots (ammo row pointer at E+0x220 + 12 x slot). This matches Radai's "dummy values" on PC.
- **Hits use the bolt's PlayerDamageParam row** (ArrowParam damage_id_0..15): flat damage_0 with its element in
  damage_type_0 (Wood 20, Heavy 50 physical; Magic / Lightning / Fire / Dark 110 of their element), Dm 0.98; ids 12-15
  point at the reduced rows (0.686).
- **Avelyn uses the normal rows.** Its shot rows (WeaponAttackMotionParam 20011400 one-handed, 20011900 two-handed)
  are attack_type 10 like the Light Crossbow's (20006800 / 20007300). Only the powerstance L1 rows (20012700, 20008100)
  are attack_type 11, so the reduced group belongs to powerstance. Check against gameranx (PC 1.05 -> 1.06, Mundane
  Avelyn + Lightning Bolts, 192 per bolt after the patch): on the reduced rows (0.247) a bolt could deal at most ~144
  even with a 250 Mundane bonus, zero defence and zero resistance. On the normal rows (0.392) the formula can reach 192.
- **Formula:** Radai's ranged formula (data-checked on PC Scholar, `DS2 Scholar PC\findings\damage-formula-radai.md`):
  physical = (h x (pAR + atkAR) - DEF / 6) x Dm, elemental atkAR = h x atkAR x (1 - res) x Dm, with h 0.9 for every v1.00
  crossbow. The combining code itself was not found in the v1.00 EBOOT (the DS2V Ghidra project is read-only and
  misses many functions), so "crossbow AR added to every bolt" rests on that guide plus the gameranx check.
- **Reload:** the Avelyn's shot animation is 42210011 (12 frames). Its reload is 42210021 (75 frames, 2.5 s), against
  42210020 (55 frames, 1.83 s) for the other crossbows. The crossbows' startup / recovery speeds (Avelyn 0.9 / 0.7,
  Light Crossbow 0.9 / 0.3) can't be reload playback rates, because the Light Crossbow would then take over 6 s.
  The real cycle needs a live timing.
- Side find: the item param selector `FUN_00543320` maps id 3251000 to 3250000, so the unused duplicate Santier's
  Spear row resolves to the real one.

## Open

- The bolt damage combining code in v1.00 (formula taken from Radai, data-checked; physical_damage_floor units
  unknown) and a live timing of the crossbow reload. Live test planned in `avelyn-live-test-plan.md` (2026-11-01).
- Code check of the weapon-side Mundane multiplication in v1.00 (no type multiplier assumed). The lowest-stat
  lookup itself is confirmed (above).
- Which TAE event really opens the next attack; an in-game timing of one dagger and one halberd chain would
  calibrate the DPS column.
- Drop rates, NG+ and PvP (players: defence / 10, x10/12, absorption) were not modelled.
