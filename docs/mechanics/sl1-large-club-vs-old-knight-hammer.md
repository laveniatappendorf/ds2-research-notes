# SL1: Large Club vs Old Knight Hammer (2026-11-01)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Question (the author): at SL1 with all available gear, which does more damage, the Large Club or the Old Knight Hammer, and
does the gear needed to wield one over the other change the answer?

Short answer:

- **Scholar, or vanilla 1.08+ with the DLCs:** the Old Knight Hammer, by about 25 % per hit.
  - Both weapons need exactly the same gear (Strength Ring + Vessel Shield), so the gear doesn't separate them.
  - The hammer was [buffed to 500 base in 1.08](../builds/balance-vanilla-vs-scholar.md#3-weapons) and takes Raw well, having no scaling to lose.
- **PS3 v1.00 to 1.03 (no DLC):** neither weapon is usable. No gear raises Strength in those builds, and both miss
  their requirement by a long way.

Static data only (params and effect scripts); nothing checked in game.

## The two weapons (params; identical in every build except where noted)

| | Large Club | Old Knight Hammer |
|---|---|---|
| Requirements | STR 26 / DEX 4 | STR 30 / DEX 6 |
| Two-handed STR needed (half, rounded down) | 13 | 15 |
| Physical +0 / +10 | 180 / 360 | 200 / 400 in v1.00-1.03 and PC up to 2014-06-02; **250 / 500 from PS3 1.08 / PC 1.12 and Scholar** |
| STR scaling (+10) | 1.2 x the STR bonus (Raw 0.3, Mundane 0.6) | none |
| Weight | 12 | 18 |
| Durability | 70 | 30 |
| Stamina per swing (WeaponParam) | 351 | 322 |
| Poise damage (menu / PvE) | 60 / 6 | 60 / 6; **65 / 6.5 in PC Scholar only** |
| Stamina / posture damage | x1.0 | x1.0375 |
| Moveset | same attack rows (WeaponAttackMotionParam 10050000 ...) | same rows: identical animations, speeds and motion values |

Since the movesets are identical, the per-hit ratio is the DPS ratio.

## Two-handing rule (community, not checked in code)

Two-handing in DS2 halves the Strength requirement and adds nothing to scaling. Sources:

- Fextralife's Strength page: effective Strength doubles for requirements only.
- darksouls2.wikidot.com/strength: "halves the Strength level required".
- Community Damage Calculator v2.44: "Two Hand Only" when STR >= ROUNDDOWN(req x 0.5); the STR bonus uses the
  unmodified STR.

**This contradicts the x1.5 rule used in [Mundane at SL50](mundane-sl50-v100.md) and `scratchpad dps3.py`** (requirements and scaling
both x1.5). Those two-handed figures need redoing.

## Strength gear

- **PS3 v1.00:** `SpEffectArmor/Ring/Weapon.emevd` (big-endian, 32-bit; [stat change = command 1000:0](../modding-world/difficulty-levers.md), byte 0 = stat,
  byte 8 = amount) has **no STR, DEX or ADP source at all**, only INT / FTH hoods, Velstadt's Helm (END / VIT) and the
  Rings of Knowledge / Prayer. Flynn's Ring doesn't exist yet either. Ring of Blades +20 / +35 / +50 does.
- **PS3 1.10 and Scholar:** the DLC items add Strength. Base STR 6 at SL1 (Deprived) can reach:
  - Strength Ring 41010000: +5 (Brume Tower);
  - Vessel Shield 11320000: +4 STR, +4 DEX, +1 to VGR / END / ADP / INT / FTH (Frozen Eleum Loyce);
  - Crown of the Sunken King 21650100: +1 to END / STR / DEX / INT / FTH, -1 to VGR / VIT;
  - all three together: STR 16.
- **Which combinations clear the requirements:**
  - Strength Ring + Vessel Shield = STR 15, which clears both (13 and 15).
  - Ring + Crown = 12, which clears neither; so does Vessel Shield + Crown (11).
  - The Crown never unlocks anything.
  - One-handed (26 / 30) is out of reach for both.

**Gear for the Large Club = gear for the Old Knight Hammer.** Both leave three ring slots free.

Assumptions:

- The Vessel Shield's bonus still counts while two-handing, and while you're under its own requirement (STR 20 /
  DEX 17). Neither is checked.
- Flynn's Ring 41100000 (Shulva) adds +50 while maximum equip load is <= 60, falling to +10 at 80 and +1 at 100
  (args 60 / 80 / 100, 50 / 10 / 1; same as the calculator). Maximum load at VIT 6 is 47.5, or 57 with Royal Soldier's
  Ring+2, so Flynn's stays at +50 either way.

## AR at SL1, two-handed, +10 (STR 15; Flynn's +50 and Ring of Blades+2 +50 on top)

| Weapon (build) | plain | Raw | Mundane (lowest stat 6) | best + rings | per hit, MV 1.2, def 0 / 300 / 514 |
|---|---|---|---|---|---|
| Large Club (any) | **442** | 434 | 307 | 542 | 650 / 590 / 547 |
| Old Knight Hammer, 1.08+ / Scholar | 500 | **575** | 336 | 675 | 810 / 750 / 707 (**+25-29 %**) |
| Old Knight Hammer, v1.00-1.03 base (for reference) | 400 | **460** | 286 | 560 | 672 / 612 / 569 (+3-4 %) |

Notes:

- Raw doesn't help the Large Club: it loses more scaling than it gains in base.
- Adding the Crown (STR 16) gives the Large Club +2 AR, and drops VIT to 5 (lowest stat 5, Mundane bonus 80).
- Red Tearstone Ring (x1.2 at low HP) and resins hit both equally.

## Other differences

- **Equip load.** With the Vessel Shield (5 on PC, 7 on PS3) and three rings, a naked character sits at 44 % with the
  Large Club and 56 % with the hammer. Room for armour before 70 %:
  - Large Club: 12.6 (18.4 with Royal Soldier's Ring+2);
  - Old Knight Hammer: 6.6 (12.4).
- **Durability.** The hammer breaks 2.3x sooner (30 vs 70). The free fourth ring is the choice between Royal
  Soldier's, Bracing Knuckle and Stone Ring.
- **Stamina.** The hammer costs about 8 % less per swing (322 vs 351), which matters with END 7.
- **Poise.** The hammer does slightly more poise and stamina damage (PC Scholar 65 vs 60 poise; x1.0375 stamina /
  posture everywhere).

## PS3 v1.00

STR 6 against 13 / 15 two-handed, so both take the weapon group of PlayerLackOfStatsParam ([stat requirement penalties](armour-requirement-penalties.md)):

| Penalty | Range |
|---|---|
| Attack | 30-80 % less |
| Wind-up | 10-40 % slower |
| Recovery | 10-30 % slower |
| Stamina per attack | 30-50 % more |
| Swing wobble / stagger | blended in |

How the weapon shortfall is computed isn't traced (see [SL1 naked damage](sl1-naked-damage.md#under-requirement)).

- **Before the penalty** (Ring of Blades+2 only): Raw Large Club 430 + 50, Raw Old Knight Hammer 460 + 50.
- **If the penalty scales with the fraction of Strength missing:** the club loses about 57 % and the hammer about
  60 %, which leaves them roughly even.
- **Verdict:** neither is a sensible SL1 weapon in v1.00.

Scripts (scratchpad): `sl1club.py` (params, every build), `sl1moves.py` (movesets), `sl1ar.py` (AR / load),
`spfx_v100.py` (big-endian effect-script reader).

## Fourth ring: Simpleton's vs Royal Soldier's+2, and roll i-frames (added 2026-11-01)

Rules, from [roll i-frames and Agility](agility-iframes.md) (code-traced in v1.00; the Agility columns are unchanged through PC Scholar):

- **Roll i-frames** = max(4, 3 + floor(14 x r)) at 30 fps, where r = PhysicalStats[floor((3 ADP + ATN) / 4)]
  roll column. PC Scholar's code gives the same window in seconds: 0.1 + 0.467 r.
- **What changes them:**
  - Agility only. Equip load has no input; the heavy roll animation has the same windows and only rolls shorter and
    slower.
  - Unmet armour requirements cost -30 to -50 % of r. None of the pieces below has a requirement.
  - Rolling short of stamina costs up to -50 % of r.
- **Royal Soldier's Ring therefore never adds i-frames.** It only raises the load at which the heavy roll starts
  (70 % of 47.5 = 33.25, or of 57 = 39.9). Flynn's stays at +50 either way.
- **ADP sources at SL1:**
  - Vessel Shield +1 and Simpleton's Ring +5 (DLC, all builds);
  - in **PC Scholar 1.0.3.0 only**: Mad Warrior Mask +2 (5.0, poise 9) and Peasant Attire / Long Gloves / Trousers
    +2 each (3.8 / 1.8 / 2.6, INT -1 each).
  - The PS3 Scholar disc and PS3 1.10 have no effect on those pieces. A later PS3 Scholar patch wasn't checked.
  - ATN stays 6: no gear raises it.

Weights: rings Strength 1.5 + Flynn's 0.2 + Ring of Blades+2 2.0, plus Simpleton's 1.5 or Royal Soldier's+2 0.8;
Vessel Shield 5 (PC) / 7 (PS3). Script: scratchpad `sl1roll.py`.

### PC Scholar

| Weapon + 4th ring | Armour | ADP (row) | Roll i-frames | Load |
|---|---|---|---|---|
| Large Club + Simpleton's | none | 12 (10) | 9 (0.300 s) | 47 % |
| | Mask, Attire, Gloves | 18 (15) | **10 (0.333 s)** | 69 % |
| | all four pieces | 20 (16) | **11 (0.367 s)** | 74.5 % (heavy roll) |
| Large Club + Royal Soldier's+2 | all four pieces | 15 (12) | 10 | 61 % |
| Old Knight Hammer + Simpleton's | none | 12 (10) | 9 | 59 % |
| | Gloves + Trousers (or the Mask alone, 69.9 %) | 14 (12) | **10** | 68.6 % |
| | all four pieces | 20 (16) | **11** | 87 % (heavy roll) |
| Old Knight Hammer + Royal Soldier's+2 | Mask, Attire, Trousers | 13 (11) | 9 | 68 % |
| | all four pieces | 15 (12) | 10 | 71.4 % (heavy roll) |

Notes:

- **Simpleton's beats Royal Soldier's+2 for both weapons.** It gives the same or one more i-frame in every
  comparison.
- **The weapons tie on i-frames.** Both reach 10 frames with a normal roll, or 11 with a heavy roll. The hammer's
  extra 6 weight only costs which ADP pieces fit under 70 %, never a frame.
- **11 frames with a normal roll** is possible only for the Large Club, by swapping Ring of Blades+2 for Royal
  Soldier's+2 (Simpleton's kept, all four pieces, 60 %). That costs 50 AR (542 -> 492; the hammer's lead becomes about
  37 %). The hammer can't get there: it sits at 70.5 % even then.

### PS3 Scholar disc / vanilla 1.10 with the DLCs

ADP tops out at 12 (Vessel Shield + Simpleton's): row 10, 9 frames.

- **With Simpleton's:** both weapons are under 70 % with no armour (Large Club 51 %, hammer 64 %). There's room for
  9 more weight with the club and 3 with the hammer before the heavy roll.
- **With Royal Soldier's+2 instead:** ADP 7, row 6, 8 frames.

## Stamina: R2 swings per bar (added 2026-11-01)

How the cost is built (params; identical in v1.00, PS3 1.10 and PC Scholar):

- **Attack cost** = WeaponParam stamina_consumption x the WeaponStaminaCostParam factor for that attack. Both weapons
  use cost row 10, so only the base differs: 351 for the club, 322 for the hammer.
- **Factors:** two-handed R2 1.75, then 1.812 for the follow-up R2; two-handed R1 1.187, then 1.312.
- **Roll** = ChrCommonParam base action 300 x 1.0 (a chained roll x1.25; a backstep x0.7). Regen is 519 per second
  (ChrParam).
- **Pool** = PhysicalStats stat_stamina: +20 per END point up to 20, then +10. SL1 END 6 gives 920, or 940 with the
  Vessel Shield's +1. The params use tenths of the stat screen's number, so the tables below divide by 10.

### Costs (stat-screen units)

| Attack | Large Club | Old Knight Hammer |
|---|---|---|
| 2h R2 | 61.4 | 56.4 |
| follow-up 2h R2 | 63.6 | 58.3 |
| 2h R1, then the follow-up R1 | 41.7, 46.1 | 38.2, 42.3 |

### What fits in one bar (94 at END 7)

| | Large Club | Old Knight Hammer |
|---|---|---|
| Full-stamina R2s | **1** (32.6 left) | **1** (37.7 left) |
| A second R2 | 31.0 short (49 % of its cost) | 20.6 short (35 %) |
| R2 then a roll | yes, 2.6 left | yes, 7.7 left |
| Two R1s | yes, 6.3 left | yes, 13.5 left |
| Two full R2s would need | 125 (END 25) | 114.7 (END 18) |

**Two full R2s are out of reach at SL1.** The most END gear can add:

- Vessel Shield +1;
- a +1 helm (Crown, Velstadt's or Cale's), which costs the Mad Warrior Mask's ADP;
- Flower Skirt +1;
- total END 9 (98).
- The Ring of the Embedded may add +7 at stat 6. That reading of its 1000:19 bytes (6, 60, 7, 3), interpolating
  +7 at 6 down to +3 at 60, is unverified. Even then END 16 (112) falls short.

**The second R2 still starts.** DS2 lets an attack start with any stamina left. PlayerLackOfStatsParam's stamina terms
would then apply:

- up to 10 % less damage;
- up to 30 % slower wind-up;
- up to 20 % slower recovery;
- swing wobble.

If they scale with the shortfall the way the roll's does (code-traced only for evasion), the club's second R2 loses
about 4.9 % damage and gets 14.6 % slower to wind up; the hammer's loses 3.5 % and gets 10.6 % slower. Either way the
bar is then empty, so there's no roll.

**Damage per stamina** (first R2, motion value 1.512, defence 300, AR with Flynn's and Ring of Blades+2):

| Weapon | First R2 hit | Per stamina point |
|---|---|---|
| Large Club | 744 | 12.1 |
| Old Knight Hammer | 945 | 16.8 (**+38 %**) |
