# Parryable but never riposted: enemies with an empty riposte zone (PC Scholar 1.0.3.0, 2026-11-23)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The riposte check tests the player's position against the riposte zone of the enemy being riposted (EnemyCommonParam `riposte_hitbox_angle_xy` / `_angle_z` / `_height` / `_radius`, +0x110 to +0x11C). With a zero radius no position passes. Why, and the live test on Sir Alonne: [Parries and ripostes](parry-and-riposte-pc.md).

This list applies that rule to the PC Scholar regulation file. Read it with these limits:

- **A zone is needed but not enough.** The enemy's parried stagger must also raise the riposte window (the Old Dragonslayer's doesn't).
- **"Parryable" means the data has at least one parryable hitbox for the row** (DamageCtrlParam byte +0x3 = 0 or 2, in its EnemyBehaviorParam groups or combos). Some of those hits may be rare or unused in play.
- Only Sir Alonne was tested live. The rows are EnemyParam rows, so one enemy can appear several times (per area, NG+, phantom).

**535 parryable EnemyParam rows: 398 with a riposte zone, 137 with an empty one.** The 137 are listed below. Names are filled in where known.

| EnemyParam | EnemyCommonParam | Name |
|---|---|---|
| 3180 | 318000 |  |
| 3181 | 318000 |  |
| 3320 | 332000 |  |
| 6200 | 101000 |  |
| 6220 | 300000 |  |
| 6240 | 101000 |  |
| 6260 | 300000 |  |
| 6280 | 101000 |  |
| 6300 | 300000 |  |
| 101000 | 101000 | Kobold - Things Betwixt |
| 101001 | 101000 | Kobold - Shaded Woods / Aldia's Keep |
| 107000 | 107000 | Parasitized Undead |
| 107001 | 107000 | Parasitized Undead |
| 121020 | 121000 | Giant (Pyromancer) |
| 121030 | 121000 | Giant (Unarmed) |
| 125004 | 125000 | Rupturing Hollow (Red Core) - Majula Pit |
| 125009 | 125000 | Rupturing Hollow (Red Core, Black Phantom) - Sinner's Rise (NG+) |
| 125010 | 125010 | Rupturing Hollow (Red Core) - Lost Bastille |
| 125011 | 125000 | Rupturing Hollow (Black Core) - Lost Bastille |
| 125012 | 125000 | Rupturing Hollow (Black Core) - Sinner's Rise |
| 125013 | 125000 | Rupturing Hollow (Black Core) - The Gutter |
| 125014 | 125000 | Rupturing Hollow (Brown Core) - Dragon Aerie |
| 125015 | 125000 | Rupturing Hollow (Yellow Core) - Lost Bastille |
| 125016 | 125000 | Rupturing Hollow (Red Core, Black Phantom) - The Gutter (NG+) |
| 141000 | 141000 | Undead Aberration - Aldia's Keep |
| 141002 | 141000 | Undead Aberration - Sinner's Rise (NG+) |
| 154000 | 154000 | Skeleton Lord (Halberd) (Boss - Skeleton Lords) |
| 154001 | 154000 |  |
| 154010 | 154000 | Skeleton Lord (Reaper) (Boss - Skeleton Lords) |
| 154011 | 154000 |  |
| 154020 | 154000 | Skeleton Lord (Staff) (Boss - Skeleton Lords) |
| 154021 | 154000 |  |
| 155000 | 155000 | Amana Aberration |
| 155001 | 155000 |  |
| 201100 | 201100 | Small Boar |
| 201101 | 201100 | Small Boar |
| 202100 | 202000 | Undead Boar - Brightstone Cove Tseldora |
| 202101 | 202000 |  |
| 202102 | 202000 | Undead Boar - Majula |
| 203000 | 203000 | Parasite Spider |
| 203001 | 203000 | Parasite Spider |
| 203002 | 203000 | Parasite Spider (Boss - The Duke's Dear Freja) |
| 203009 | 203000 | Parasite Spider (Black Phantom) - (NG+) |
| 205000 | 205000 | Poison Horn Beetle - Earthen Peak |
| 205001 | 205000 | Poison Horn Beetle - The Gutter |
| 205100 | 205000 |  |
| 205102 | 205000 | Acid Horn Beetle |
| 209000 | 209000 | Hunting Dog - Aldia's Keep |
| 209002 | 209000 | Hunting Dog - The Gutter |
| 210000 | 210000 | Basilisk - Things Betwixt / Old Akelarre / Shaded Woods |
| 210001 | 210000 |  |
| 210002 | 210000 | Basilisk - Aldia's Keep |
| 210003 | 210000 |  |
| 210004 | 210000 | Basilisk - Unseen Path to Heide |
| 210005 | 210000 | Basilisk - Shrine of Amana |
| 210010 | 210000 | Basilisk - Brightstone Cove Tseldora |
| 214001 | 214000 | Giant Undead Boar - Brightstone Cove Tseldora (NG+) |
| 214002 | 214000 |  |
| 214003 | 214000 | Giant Undead Boar - Majula |
| 216000 | 216000 | Wall Ghost - Undead Crypt |
| 216005 | 216000 | Wall Ghost - Drangleic Castle |
| 220000 | 220000 |  |
| 220001 | 220000 | Giant Acid Horn Beetle - Aldia's Keep |
| 220002 | 220000 | Giant Acid Horn Beetle - Shrine of Amana |
| 300000 | 300000 | Ogre - Things Betwixt / Forest of Fallen Giants |
| 300001 | 300000 | Ogre - Shrine of Amana |
| 300002 | 300000 | Ogre - Aldia's Keep |
| 311000 | 311000 | Mounted Overseer |
| 314000 | 314000 | Hollow Primal Knight - Doors of Pharros |
| 314002 | 314000 | Hollow Primal Knight - Unseen Path to Heide |
| 314003 | 314000 |  |
| 318000 | 318000 | The Pursuer - Drangleic Castle |
| 318010 | 318000 | The Pursuer - Forest of Fallen Giants (Boss) |
| 318011 | 318000 | The Pursuer - Drangleic Castle |
| 318020 | 318000 | The Pursuer - Iron Keep |
| 318030 | 318000 | The Pursuer - Lost Bastille |
| 318040 | 318000 | The Pursuer - Things Betwixt |
| 321000 | 321000 | Mimic (Wooden Chest) - Aldia's Keep |
| 321001 | 321000 |  |
| 321002 | 321000 |  |
| 321003 | 321000 | Mimic (Wooden Chest) - Earthen Peak |
| 321004 | 321000 | Mimic (Wooden Chest) - The Gutter |
| 321005 | 321000 | Mimic (Wooden Chest) - Drangleic Castle |
| 321010 | 321000 | Mimic (Metal Chest) - Aldia's Keep |
| 321011 | 321000 | Mimic (Metal Chest) - Iron Keep |
| 321012 | 321000 |  |
| 321014 | 321010 | Mimic (Metal Chest) - Frozen Eleum Loyce |
| 321015 | 321010 | Mimic (Metal Chest) - Frozen Eleum Loyce |
| 321016 | 321000 | Mimic (Metal Chest) - Majula (NG+) |
| 321017 | 321000 | Mimic (Metal Chest) - Brightstone Cove Tseldora |
| 324000 | 324000 | Belfry Gargoyle (Boss) |
| 324010 | 324000 | Belfry Gargoyle - Drangleic Castle |
| 324011 | 324000 | Belfry Gargoyle - Aldia's Keep |
| 325000 | 325000 | Ruin Sentinel (Boss) |
| 325010 | 325000 | Ruin Sentinel - Drangleic Castle |
| 332000 | 332000 | Throne Defender (Boss) |
| 333000 | 333000 | Velstadt, the Royal Aegis (Boss) |
| 334000 | 334000 | Throne Watcher (Boss) |
| 501000 | 501000 | Mytha, the Baneful Queen (Boss) |
| 504000 | 504000 | Looking Glass Knight (Boss) |
| 601000 | 601000 | Flame Lizard - Forest of Fallen Giants |
| 601001 | 601001 | Flame Lizard - Memory of the Old Iron King |
| 611010 | 611010 | Dragonrider (Bow) (Boss) - Drangleic Castle |
| 611500 | 611000 | Dragonrider (Halberd) (Boss) - Drangleic Castle |
| 611510 | 611001 | Dragonrider (Halberd) (Boss) - Heide’s Tower of Flame |
| 611520 | 611001 | Dragonrider (Halberd) - Undead Crypt |
| 611530 | 611001 | Dragonrider (Halberd) - Shrine of Amana |
| 625000 | 625000 | Old Dragonslayer (Boss) |
| 626000 | 626000 | The Lost Sinner (Boss) |
| 654000 | 654000 | Ashen Crawler |
| 654001 | 654000 |  |
| 661000 | 661000 | Frozen Reindeer |
| 661001 | 661000 | Frozen Reindeer |
| 670000 | 670000 | Sanctum Priestess |
| 680000 | 680000 | Sir Alonne (Boss) |
| 682000 | 682000 | Elana, the Squalid Queen (Boss) |
| 686000 | 686000 | Small Undead Boar (Boss - Elana, the Squalid Queen) |
| 687000 | 687000 | Velstadt, the Royal Aegis (Boss - Elana, the Squalid Queen) |
| 688000 | 688000 | 下位：巫女を護る白の騎士 |
| 688010 | 688000 | enemy 688010 |
| 688020 | 688000 | enemy 688020 |
| 725000 | 725000 | Darkdiver Grandahl - Shaded Woods |
| 725001 | 725000 | Darkdiver Grandahl - Shaded Woods |
| 725002 | 725000 | Darkdiver Grandahl - Black Gulch |
| 725003 | 725000 | Darkdiver Grandahl - Drangleic Castle |
| 753000 | 753000 | Bell Keeper (NPC) - Belfry Luna |
| 753001 | 753000 | Bell Keeper (NPC) - Belfry Luna |
| 753002 | 753000 | Bell Keeper (NPC) - Belfry Sol |
| 753003 | 753000 | Bell Keeper - Belfry Luna |
| 753004 | 753000 | Bell Keeper - Belfry Sol |
| 753008 | 753000 | Bell Keeper (Black Phantom) - Belfry Sol |
| 753009 | 753000 | Bell Keeper (Black Phantom) - Belfry Luna |
| 754000 | 754000 | Merchant Hag Melentia |
| 754001 | 754000 | Merchant Hag Melentia |
| 776000 | 776000 | Weaponsmith Ornifex - Brightstone Cove Tseldora |
| 776001 | 776000 | Weaponsmith Ornifex - Brightstone Cove Tseldora |
| 776002 | 776000 | Weaponsmith Ornifex - Shaded Woods |
