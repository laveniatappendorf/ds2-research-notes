# Best miracle catalyst for high Faith (PS3 v1.00, BLUS41045)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-10-31. The author asked which catalyst is best for miracles on a high-Faith character in v1.00.

## Method

Spell buff = the chime's lightning attack rating, using the community damage calculator's formula (v2.44, CatalystAR
sheet): base(level) + coefficient(level) x lightning bonus(Faith), at max upgrade, uninfused. Values from the v1.00
regulation (WeaponParam, WeaponReinforceParam, WeaponStatsAffectParam, PhysicalStatsPerLevelStatValuesParam; the
lightning bonus is 140 / 155 / 162 / 200 at Faith 40 / 50 / 60 / 99). In that calculator a lightning miracle's damage
is (spell buff + the spell's flat value) x the spell's multiplier, so the ranking holds for every lightning miracle.
Cast speed = WeaponTypeParam `cast_speed`, which the calculator multiplies into the character's cast speed (higher =
faster). Neither formula is traced in game code, and nothing was tested live. Healing and buff miracles were not
looked at. Tool: `tools\chime_buff.py [build] [faith ...]`.

## v1.00

| Chime | Max | FTH req | Buff at FTH 40 / 50 / 60 / 99 | Cast speed |
|---|---|---|---|---|
| Dragon Chime | +5 | 50 | 394 / 409 / 417 / 456 | 0.55 |
| Priest's Chime | +10 | 18 | 330 / 345 / 352 / 390 | 0.8 |
| Witchtree Bellvine | +10 | 15 | 326 / 340 / 346 / 380 | 1.0 |
| Chime of Want | +5 | 40 (INT 15) | 326 / 338 / 344 / 376 | 0.7 |
| Archdrake Chime | +10 | 22 | 308 / 318 / 323 / 350 | 0.8 |
| Disc Chime | +10 | 18 | 292 / 304 / 310 / 340 | 1.0 (casts on strong attack only) |
| Cleric's Sacred Chime | +10 | 10 | 285 / 296 / 302 / 330 | 0.6 |
| Protective Chime | +10 | 25 | 297 / 305 / 309 / 330 | 0.6 |
| Idol's Chime | +5 | 25 | 266 / 271 / 273 / 286 | 0.95 |
| Sacred Chime Hammer | +5 | 12 (STR 40, INT 12) | 203 / 209 / 211 / 226 | n/a |

Caitha's Chime is hex-only ("Cannot be used to cast miracles"; type flag is_cast_miracles 0). Chime of Screams
(4120000, cast speed 2.5) is not in v1.00.

Dragon Chime has the highest buff at every Faith from its 50 requirement up (about 17-19 % over Priest's Chime) and the
slowest cast speed. Witchtree Bellvine is 2-3 % behind Priest's Chime and casts fastest (1.0 vs 0.8).

## v1.00 quirk: +5 chimes stop halfway up their curve

In v1.00 the +5 chimes (Dragon, Want, Idol's, Sacred Chime Hammer) share an 11-step scaling curve with the +10 ones,
so at +5 they only reach the curve's midpoint (Dragon Chime 1.028 instead of 1.050). By 1.10 their curves reach the
top at +5. It costs Dragon Chime 4 points of buff at Faith 99; the ranking is unchanged. PC Scholar matches 1.10.
