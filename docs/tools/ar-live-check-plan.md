# Live check: attack-rating formula and the infusion split (plan, 2026-10-26; claws + dark buffs added 2026-11-03)

!!! warning "Status: Plan, not run"
    A test or idea written down. Nothing here has been tried. See [status labels](../status-labels.md).

Follow-up to `..\..\DS2 Vanilla\findings\scaling-letters-and-infusion.md`. The static work could not trace the AR
code (it sits behind the PC exe's anti-tamper stubs), so this reads the weapon screen at controlled stats and scores
competing formulas. Predictions: `tools\ar_predict.py` (`spawns`, `predict [set]`, `check obs.csv`).

## Questions

1. Is the stat bonus flat (`coefficient x bonus table`, the same for every weapon) or proportional to base damage?
2. On an infused weapon, is the physical bonus cut by the physical share (the YouTube comment's model) or not?
   Same question for the element bonus and the element share.
3. Which stat drives fire and dark (INT, FTH, their average, or the lower of the two)? The param points fire at INT
   and dark at FTH; the community Damage Calculator v2.44 uses the average for fire and the lower one for dark.
4. (Optional) Does the in-combat attack record match the screen?
5. Does innate bleed take part of the infusion split? The calculator keeps physical at 1 - share / n (n = elements
   after infusion, innate bleed counted), so Dark Manikin Claws (bleed) keep 75 % physical and Dark Malformed Claws
   (no bleed) 50 %. Bandit's Knife and Flamberge (both innate bleed) test the same rule on fire.
6. How much does Dark Weapon (bFlat 15, bMod 0.3) add, and Dark Pine Resin (40, 0.2)? Radai's guide and the
   calculator: bMod x (bFlat + dark scaling x Dark bonus + dark base x base mult), plus share x bFlat when the
   weapon is dark-infused. Open: is the base term there on an uninfused weapon, and is the matching bonus real?
7. Which weapons refuse which buffs? ItemParam `item_speffect_application_type` 2 (Claws, Flamberge, Manslayer,
   Black Knight weapons ...) reads as "spells only, no resins"; the calculator also bars resins on element- and
   status-infused weapons but lets spells onto any infusion.

Questions 3, 5 and 6 decide the claw + Dark Weapon build (`<local path>`):
if dark follows Faith alone, INT can stay at Dark Weapon's 16.

## Design

- **Same scaling, different base.** Dagger 57, Uchigatana 115 and Falchion 120 all have 15 % STR / 40 % DEX at +0.
  A flat bonus is identical across the three; a proportional one doubles.
- **Different splits.** Fire infusion on the Bandit's Knife (20 % fire, base x1.0), Dagger (50 %, x1.4), Blue Dagger
  (55 %, x1.4) and Flamberge (60 %, x1.2), so a share factor can't hide behind one number.
- **Stat sets** (all above every test weapon's requirements, one-handed, no rings or buffs):

| set | STR | DEX | INT | FTH | read |
|---|---|---|---|---|---|
| S1 | 20 | 20 | 20 | 20 | all 21 weapons, then the buff readings |
| S2 | 20 | 40 | 20 | 20 | the three twins, Fire Dagger / Knife / Blue Dagger, Manikin + Dark Manikin, Dark Malformed |
| S3 | 40 | 20 | 20 | 20 | the same nine |
| S4 | 20 | 20 | 40 | 20 | Fire, Magic, Lightning, Dark Dagger, Dark Manikin +10, Dark Malformed; Dark Weapon on Manikin +10 and Dark Manikin +10 |
| S5 | 20 | 20 | 20 | 40 | the same |

  Bonus table values used: 20 -> 80 and 40 -> 140 for STR / DEX; fire 117 / 155, dark 117 / 170 (dark 30 = 162).

Key predictions (S1, whole numbers on screen; a hypothesis "fits" within 1):

| weapon | physical flat | proportional | flat x share | element flat | x share |
|---|---|---|---|---|---|
| Dagger +0 | 101 | 82 | | | |
| Uchigatana +0 | 159 | 166 | | | |
| Falchion +0 | 164 | 173 | | | |
| Fire Dagger +0 | 73 | 53 | 56 | 101 | 71 |
| Fire Bandit's Knife +0 | 70 | 61 | 66 | 62 | 23 |
| Fire Flamberge +0 | 90 | 78 | 72 | 106 | 83 |
| Dagger +10 / Fire Dagger +10 | 163 / 117 | 170 / 110 | — / 99 | — / 151 | — / 116 |

S4 / S5 then separate fire and dark keying (Fire Dagger fire: INT-keyed 121 / 101, FTH-keyed 101 / 121, average 117 /
117, lower-of-two 101 / 101, flat version). Every key gives a different S4 / S5 pair, so the two sets settle it.
The calculator's physical rule ("calc") differs from "share" only on weapons with innate bleed: Fire Bandit's Knife
75 (flat 70, share 66), Fire Flamberge 126 (flat 90, share 72).

### Claws and dark buffs (added 2026-11-03)

Claw attack, S1 (all +0 unless marked; "calc" physical and "share" element = the calculator):

| weapon | physical flat | share | calc | dark flat | dark share |
|---|---|---|---|---|---|
| Claws / Manikin / Malformed (uninfused) | 121 / 131 / 117 | | same | | |
| Dark Claws | 72 | 55 | 83 | 79 | 59 |
| Dark Manikin Claws | 87 | 66 | 99 | 102 | 74 |
| Manikin Claws +10 / Dark +10 | 210 / 135 | — / 113 | 210 / 169 | — / 156 | — / 123 |
| Dark Malformed Claws | 85 | 65 | 65 | 107 | 76 |

Dark keying on Dark Manikin Claws +10 (dark, share), S4 / S5: INT 138 / 123, FTH 123 / 138, average 135 / 135,
lower 123 / 123. Dark Malformed and the Dark Dagger give a second and third read of the same split.

Buff readings = how much the dark number rises after one cast. `full` = the formula in question 6; `nomatch` without
the matching bonus; `nobase` without the base term:

| set | weapon | Dark Weapon full | nomatch | nobase |
|---|---|---|---|---|
| S1 | Manikin Claws +0 | 39.3 | | 16.8 |
| S1 | Manikin Claws +10 | 63.5 | | 18.5 |
| S1 | Dark Manikin Claws +10 | 85.7 | 78.2 | 31.7 |
| S1 | Dark Malformed Claws +0 | 57.7 | 50.2 | 30.4 |
| S1 | Claws +0 (spell-only flag) | 39.9 | | 16.8 |
| S1 | Fire Dagger +0 (spell on another element) | 46.9 | | 22.9 |
| S4 / S5 | Dark Manikin Claws +10, keyed INT / FTH / average / lower | 94.6 / 85.7 / 93.2 / 85.7 and 85.7 / 94.6 / 93.2 / 85.7 | | |

Dark Pine Resin at S1: Manikin Claws +10 47.4, Malformed Claws +0 29.2; Claws +0 and Dark Manikin Claws +10 should be
refused. `ar_predict.py predict S1 S4 S5` prints all of it.

## Spawn list (community table, "Spawn Item": id, reinforcement, infusion code)

| label | item id | upgrade | infusion |
|---|---|---|---|
| Dagger | 1000000 | +0 | 0 |
| Fire / Magic / Lightning / Dark Dagger | 1000000 | +0 | 1 / 2 / 3 / 4 |
| Uchigatana | 1700000 | +0 | 0 |
| Falchion | 1600000 | +0 | 0 |
| Bandit's Knife, Fire Bandit's Knife | 1010000 | +0 | 0, 1 |
| Fire Blue Dagger | 1140000 | +0 | 1 |
| Fire Flamberge | 1810000 | +0 | 1 |
| Dagger +10, Fire Dagger +10 | 1000000 | +10 | 0, 1 |
| Claws, Dark Claws | 3410000 | +0 | 0, 4 |
| Manikin Claws, Dark Manikin Claws | 3430000 | +0 and +10 | 0, 4 |
| Malformed Claws, Dark Malformed Claws | 3420000 | +0 | 0, 4 |
| Dark Weapon (spell), Witchtree Branch (staff), Dark Pine Resin x5 | 34060000, 3820000, 60270000 | | |

Infusion codes 0 none, 1 fire, 2 magic, 3 lightning, 4 dark ... 8 enchanted, 9 mundane (the table's own list).

## Setup

- Steam set to Offline (pcmem refuses writes otherwise). Game windowed or borderless, test save `fotfg003` (any save
  is fine: saves are disposable). Stand at a bonfire. Take off rings and left-hand weapon; no buffs.
- Cheat Engine with `Dark Sous II 0 SOTFS community advanced.CT` (its Spawn Item script). **Risk:** no community table
  claims exe 1.0.3.0; if Spawn Item fails, the fallback is DS2S-META (pseudostripy fork 0.8, supports this exe), which
  would be a download (asked first).
- Stats: `pcmem.py setstats STR=20 DEX=20 INT=20 FTH=20` and so on (logged writes; soul level follows).
- Buff part only: also `ATN=30` (more casts; attunement doesn't touch attack), attune Dark Weapon at the bonfire,
  Witchtree Branch in left hand 1 (INT 16; every set has INT 20+, FTH 20+ covers the spell's FTH 14).

## Procedure (about 45-60 minutes, plus about 30 for the buffs)

1. Spawn the 21 weapons and the three buff items; check each name on screen (e.g. "Fire Dagger") and that the upgrade level is right.
2. **Checkpoint: does the screen follow a memory stat write?** Note the Dagger's attack, run setstats S1, reopen the
   menu. If the value doesn't move: rest at the bonfire, then try a warp; note which one refreshes it. If nothing
   does, fall back to levelling up for real (sets then run in rising order, and S2 / S3 / S4 / S5 need care).
3. For each set S1-S5: setstats, refresh as found in step 2, open Equipment > right weapon 1, scroll over the
   weapons listed for that set, capture the detail panel for each (all damage types, and base + bonus if the panel
   splits them): `powershell -File tools\pcshot.ps1 S1_fire_dagger` -> `findings\ar-live\S1_fire_dagger_<time>.png`.
4. Write the readings to `findings\ar-live-check.csv` (`set,weapon,physical,element,dw,resin`, labels as in
   `ar_predict.py spawns`) and run `ar_predict.py check`. Any miss is printed against the flat prediction.
4b. **Buffs (S1, then S4 and S5; about 30 minutes).** First a checkpoint: does the Equipment screen show a buff at
   all? Put Manikin Claws +10 in right hand 1, note the dark number, cast Dark Weapon, open the menu at once (the game
   doesn't pause and the buff lasts 90 s). If the number doesn't move, read `pcmem.py arblock` group A / B dark
   before and after the cast instead (that reader is untested live). Then for each weapon in the buff table: equip,
   read dark, cast, read dark again, capture both with pcshot (`S1_dw_manikin10_before` / `_after`). Swapping weapons
   may drop the buff, so one cast per weapon; resting at the bonfire refills casts and clears buffs. Record the rise
   in the `dw` column. Resin: use Dark Pine Resin from the item menu on the four resin weapons, record the rise in
   `resin`, or `x` and a screenshot of the message if the game refuses.
5. Optional: equip the Fire Dagger at S1 and run `pcmem.py arblock 30` while swinging at the air: it prints the stats,
   the equipped hand items (with their 0x14-byte slot records, which may hold upgrade / infusion) and the community
   table's "Attack Rating" block `[[PlayerCtrl+0x230]]+0xF08` (group A +0x1C.. and group B +0x50.., physical, magic,
   lightning, fire, dark), then one line per change. Unrounded numbers if either group turns out to be the AR.
6. Put the stats back if wanted (`setstats STR=26 DEX=4 ...`); the spawned weapons can stay.

## Capturing the screen

In order of preference: `tools\pcshot.ps1 <label>` (built 2026-10-26: the shared
`..\..\DS2 Network Test\tools\screenshot.ps1 -Process DarkSoulsII`, PrintWindow with a screen-copy fallback; tested on
other windows and on its error paths, **not yet on the DirectX 11 game window**: check it on the first screen, and
keep the game windowed or borderless), Steam's F12 screenshots (saved under
`C:\Program Files (x86)\Steam\userdata\<account>\760\remote\335300\screenshots`; the folder doesn't exist yet), or
reading the numbers out.

## Prep (done 2026-10-26)

- `tools\pcshot.ps1` + the `-Process` switch on the shared screenshot.ps1 (default still rpcs3; its black-frame test
  now samples a 5 x 5 grid, and tiny windows no longer throw).
- `pcmem.py arblock [s]` (read-only, PlayerCtrl RTTI-checked; tested on a fake memory image, not live). Pointers from
  the community tables: hand slots in ChrAsmCtrl `[PlayerCtrl+0x378]` (L1 +0x9C, R1 +0xB0, L2 +0xC4, R2 +0xD8, L3
  +0xEC, R3 +0x100: records of 0x14 bytes, left and right alternating), rings +0x4E8..+0x4F4.

## How the outcome reads

- Twins equal + flat fits everywhere: the bonus is flat; the comment's "not multiplied by base damage" holds.
- Infused physical fits "flat x share": the comment's split model holds (101.3 % on paper = 45.6 % in effect on the
  Twin-headed Greatbow). Fits "flat": the letters are honest and infusion keeps all the physical bonus.
- Neither: base splitting or rounding works differently; the misses and the attack record (step 5) are the lead.
- Dark key (S4 / S5 elements and buffs): "lower" = the calculator and the build advice stand (INT and FTH 30 / 30).
  "FTH" = INT can stay at 16 (14 levels saved). "average" = INT and FTH trade freely. "INT" = FTH can stay at 14.
- `phys:calc` beats `share` on the bleed weapons: innate bleed takes part of the split, as the calculator says.
- Buffs fit `full`: Radai's formula holds and Dark infusion is worth what the build table says. `nobase` fits:
  buffs ignore the weapon's base damage, so upgrade level matters less and the infused / uninfused gap shrinks.
  `nomatch` fits only on the dark-infused weapons: the matching bonus isn't real (about 7 attack less).
- Resin refused on Claws but not Manikin Claws: application type 2 = "no resins" is confirmed.
