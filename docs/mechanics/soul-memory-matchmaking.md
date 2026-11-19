# Soul memory matchmaking: what is in the game, what is on the server (2026-11-02)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

**In short:** the game client only *counts* soul memory and *sends* it. The tier table and the "tiers down / tiers up"
range for each multiplayer item are not in any client build we have, in params or in code. So the belief is right:
the bucketing and range check run on the server. Everything published about tiers and ranges was worked out by players
from what they saw online, and the open-server project (ds3os) copied those community numbers into its own server.

## What the client holds

- **The counter.** The counter sits in PlayerParam (SOTFS PC +0xF4 / +0xFC, from the CE work in
  `DS2NT-automation-plan.md`). It shows on the status screen as text 909050 "Soul Memory". That text was added in
  retail: the NT status screen has no Soul Memory line ([Agility timeline](agility-timeline.md)).
- **What it sends.** Every sign, invasion, arena and "good area" request carries a protobuf `MatchingParameter`.
  Field list from ds3os `Protobuf/DarkSouls2/DS2_Frpg2RequestMessage.proto` (worked out from traffic; DS2 uses
  protobuf-lite, so the game has no field names):

  | # | field | note |
  |---|---|---|
  | 1 | calibration_version | regulation version, 20200 on SOTFS |
  | 2 | soul_level | sent, but no known check uses it |
  | 3 | clear_count | NG cycle |
  | 4 | unknown_4 | changes with clear_count (77, 154, 231, 693) |
  | 6 | covenant | |
  | 7, 9, 10 | unknown | NAT type / region guesses |
  | 8 | disable_cross_region_play | |
  | 11 | name_engraved_ring | the god chosen, 0 = no ring |
  | 12 | soul_memory | the raw u32 count, not a tier |

  The type name `Frpg2RequestMessage.MatchingParameter` is also in the PS3 v1.00 and NT EBOOTs. The NT's own field
  list is not checked: it would mean finding the protobuf-lite serializer.
- **The params.** None of these holds a soul memory threshold:
  - `NetworkParam` is one row of timers and rates.
  - `NetworkAreaParam` is per-area times (30 / 20 / 30) plus a bitmask that is probably "allowed multiplayer types".
  - `ChrMultiplayParam` is co-op enemy scaling (100 / 90 / 80 / 67 / 60 / 45 % by phantom count; the NT has none).
  - `VowParam` holds the covenant rank thresholds.
- **One text trace of a server rule.** Up to 1.09, `bofire.fmg` 15110 warned before NG+: "only players on journey 2
  to Drangleic and beyond will be eligible for matchmaking". From 1.10 it only says "Begin your second journey?".
  The client just sends `clear_count`. Splitting NG from NG+ was a server decision, and the warning was dropped at
  1.10 (`text_1.09_to_1.10.txt`). Whether the split itself ended at 1.10 is not checked.

## The scan (tools\sm_tier_scan.py)

The scan looked for the tier boundaries, in both the "end" form (1,749,999) and the "start" form (1,750,000), in
these ways:
- as int32, float32 and float64 values, in both byte orders, as single values;
- as runs of 4 or more boundaries at strides of 4 to 24 bytes;
- on PS3, as constants built in code with `lis` + `addi`/`ori`.

As a check that the scan can see such constants, it also looked for the 999,999,999 soul cap. The cap was found in
every build:

| build | soul cap found | tier values found |
|---|---|---|
| PC Scholar exe | 44 | 0 |
| PC vanilla 1.03 exe | 37 | 0 |
| PS3 v1.00 EBOOT | 28 (in code) | 0 |
| PS3 Scholar EBOOT | 30 (in code) | 0 |
| NT EBOOT | 10 (in code) | 0 |

## What the community and ds3os say (server side, so it cannot be checked in the files)

- **Tiers.** The wiki.gg table has 45 tiers: 10k steps to 50k, then 20k, 30k, 50k, 100k, 250k and bigger steps, with
  the last two being 45M - 359,999,999 and 360M - 999,999,999. ds3os's `k_defaultTiers` has 44: it has no 360M split.
  The tier rule is `tier = count of boundaries < SM`.
- **Ranges (down / up; with the Name-Engraved Ring, same god).**

  | item | range | with the ring |
  |---|---|---|
  | Small White Sign Soapstone | -4 / +2 | -7 / +5 |
  | White Sign Soapstone | -3 / +1 | -6 / +4 |
  | Red Sign Soapstone | -5 / +2 | no change |
  | Dragon Eye | -5 / +5 | |
  | Cracked Red Eye Orb | -0 / +4 | |
  | Cracked Blue Eye Orb | -3 / +3 | |
  | Bell Keeper | -1 / +3 | |
  | Rat | -1 / +3 | |
  | Guardian's Seal | -7 / +6 | |

  ds3os uses the wiki values except where it says "Guessed":
  - Rat: ds3os -5 / +2, wiki -1 / +3;
  - Blue Sentinel: ds3os -4 / +2 (-7 / +5 with the ring), wiki -7 / +6;
  - Mirror Knight and arena: guessed;
  - "right matching area": ±5.

  ds3os widens the range whenever the host wears the ring. It does not check that both players chose the same god.
- **No soul level or weapon level matching in DS2.** `soul_level` is sent but not used. ds3os's old SL / weapon-level
  check is in `#if 0`.

## Open

- The NT `MatchingParameter` fields: did the network test send soul memory at all?
- 44 vs 45 tiers (the 360M split). This could only be settled on live servers (PC SOTFS), with two characters
  either side of 360M.
