# Forest Spirits: Eye of the Priestess, transparency, damage, drops (PC Scholar 1.0.3.0, data only)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-11-14. Four community claims checked against the data. No live test.

## The two kinds

| | Dark Chasm of Old (m40_03) | Frozen Eleum Loyce (m50_37) |
|---|---|---|
| EnemyParam | 129000 (generators 3100, 3210), 129010 (3000, 3200) | 129200 (generators 5210-5212, by the Lower Garrison bonfire 37665) |
| Model | c1290 (several weapons) | c1292 = the c1290 body and textures with only WP_3820 (Witchtree Branch); same materials (C[Ibl][DSB] + _Alp_1bit) |
| chr_phantom_param_id | 17 = ghost draw (also Wall Ghosts, Doors of Pharros Rat King); fade alpha 1.0 -> 0.1 | 0 = drawn like an ordinary enemy |
| chr_fade_in_on_spawn | 0 | 0.8 s |
| Death lot | 12900000 (129000) = Witchtree Branch 3820000, 1 %; 12901000 (129010) = Witchtree Bellvine 4020000, 1 % | 12900000 = Witchtree Branch, 1 % |
| HP / phys defence | 560 / 635 | 560 / 635 |
| Slash / thrust / strike absorption | 1.0 | 1.0 |
| Bullet absorption phys / mag / lit / fire / dark | 1.0 all | 0.8 / 0.475 / 0.425 / 0.56 / 0.575 |
| base_damage_mult | 1.0 | 0.75 (most likely their own attack damage; the defence sheet doesn't use it) |
| Posture / souls | 4400 / 600 | 21978 / 1500 |
| Unknowns that differ | Unk01 0, Unk04 20, DrawGroup 255 | Unk01 5, Unk04 110, DrawGroup 12 |

## Claims

1. **Chasm spirits more visible with the Eye of the Priestess: not supported.** The Eye works through flag 537000012
   (set by picking it up, event group 17000; holding the item alone does nothing, Boss Arena A8 live test,
   2026-10-28). Only m50_37's transparency groups read it: 18000-18210 "ザコの透明管理 / 透明化" for the Wall
   Watchmen (generators 2620 / 2621 / 2700, SpEffects 98830010 / 98830000) and 26000 / 26010 "トラの透明化" for
   Aava (907, SpEffects 96790010 / 96790000 / 96790020). The Chasm script (85 groups) never reads the flag and has no
   transparency groups. The spirits' AI (ai129000 / ai129010) applies no visibility effect. Their see-through look is
   phantom row 17, which nothing switches.
2. **Eleum spirits more transparent: not supported by data; data points the other way.** No phantom draw, opaque
   model, and their generators are not in the m50_37 event script. Open: what Unk01 / Unk04 / DrawGroup do, and how the
   frozen area's lighting and fog make them look. Needs a look in game.
3. **Physical attacks weaker against the most transparent version: no for melee.** Same defence and absorption on
   both; per-part damage rates switched off on both (EnemyCommonParam +0x254 = 0; see Najka write-up); same 0.2 grab
   cut (+0x26C); TAE c129000 / c129010 hold only event types 2200, 2300, 110000, 120200, 280100, 290002 (no 111000
   part-rate events, no SpEffect ids). Projectiles do less to the **Eleum** kind (physical x0.8, spells about half).
4. **Eleum spirits drop the Witchtree Branch: yes**, 1 % (lot 12900000, the same lot as Chasm 129000). The other
   Chasm kind (129010) drops the Witchtree Bellvine at 1 %.

Placements are the same in PS3 Scholar (enemy_placements_pc.tsv). Scripts: scratchpad only (gen.py, ph.py,
taescan.py, mtd.py); event dumps with tools\evdump_pc.py.
