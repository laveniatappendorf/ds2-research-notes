# Which bosses count as hollow (PS3 v1.00, BLUS41045)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-10-31. The author asked which bosses the game counts as hollow in PS3 1.00.

## The flag

EnemyParam `is_hollow` (u8 at row +0x131; community name "Is Hollow", SmithBox defs). 133 of 618 v1.00 rows have it.

What reads it: DamageCtrlParam `is_only_affect_hollows` is set on 3 of 2,817 rows, and two of them belong to the
two player tools whose text says they only hurt hollows:

| DamageCtrl | Used by | Item |
|---|---|---|
| 99986022 | PlayerDamageParam 32170011 (+ EnemyDamageParam 192170010) | Soul Appease 32170000: "Inflicts heavy damage on mindless Hollows, but has no effect on anything else" |
| 99993030 | PlayerDamageParam 60620011 | Holy Water Urn 60620000: "scalds the skin of Undead and Hollows" |
| 7041100 | PlayerDamageParam 3660000-3660002 | not identified |

So "counted as hollow" = takes damage from Soul Appease and the Holy Water Urn. Static reading only: the game code
that pairs the two flags was not traced, and nothing was tested live.

The flag is wider than the name: besides hollow humans it is set on Gyrm, Varangian Sailors, Heide Knights,
Ironclad Soldiers, rats (Mongrel, Corpse, Rat King), dogs (Stray, Hunting), boars and Mimics.

## Bosses (placed in v1.00)

Hollow:
- The Rotten 326000
- Royal Rat Vanguard 226100 (its Corpse Rats 226001 too)
- Royal Rat Authority 628000
- Vendrick 514600
- Prowling Magus & Congregation: the congregation only (Hollow Priests 106001, Hollow Prisoners 127002, Undead
  Supplicants 147004); the Prowling Magus 137021 is not
- Executioner's Chariot: the Black Hollow Mages 118001 only; the Chariot 619110 and its skeletons are not

Not hollow: Last Giant, Pursuer (all rows), Dragonriders (Heide + castle), Old Dragonslayer, Flexile Sentry and
its Suspicious Shadows, Ruin Sentinels, Belfry Gargoyles, Lost Sinner, Skeleton Lords (Lords, Bonewheels,
skeletons), Covetous Demon, Mytha, Smelter Demon, Old Iron King, Scorpioness Najka, Freja and her spiders, Demon of
Song, Looking Glass Knight, Velstadt, Throne Watcher, Throne Defender, Nashandra, Darklurker, Guardian Dragon, Giant
Lord, Ancient Dragon. DLC bosses are not in the v1.00 regulation.

Fights and rows: the community SOTFS defence sheet's boss rows, grouped and matched to v1.00 generators as in
`DS2 Scholar PC\tools\bosses_pc.py`. Script: `tools\hollow_bosses.py` (`all` = every EnemyParam row). Later
patches not compared.
