# Earthen Peak aggro: vanilla PC 1.12 vs PC Scholar (2026-11-02)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author's question: enemies in Earthen Peak seem to aggro from much further away in Scholar than in vanilla PC (latest
patch). Has the leash or aggro range changed?

Builds: vanilla PC 1.12 (Steam install, per-map params via `DS2 Vanilla PC\tools\pcindex.py`, regulation in
`DS2 Vanilla PC\1.12\regulation`) and PC Scholar 1.0.3.0 (`gamedata_for("pcsotfs")`, `DS2 Scholar PC\regulation`).
Earthen Peak shares map m10_17_00_00 with Harvest Valley; "Earthen Peak" below = generators with z >= 150 (the beetle
pool and up), bosses and NPCs left out: 62 enemies in 1.12, 66 in Scholar. Param layouts are the same size in both
builds (LogicComParam 128, GeneratorParam 160, GeneratorRegistParam 16 bytes), so both decode with the PC Scholar defs.
Scripts were ad hoc (session scratchpad), not kept in tools.

## 1. Sight and notice radius: unchanged

The perception row in effect (LogicComParam, by the spawn's LogicParamID; the DefaultLogicParam chain gives the same
row) is **identical, every field**, for every enemy type in Earthen Peak in both builds:

| enemy | sight lit (`Unk04`) | sight dark (`sight_distance`) | all-round (`detect_distance`) | cone |
|---|---|---|---|---|
| Manikins 124xxx | 20 m | 13 m | 1.3 m | 60° × 45° |
| Desert Sorceress 316001 | 30 m | 13 m | 1.1 m | 60° × 45° |
| Grave Warden 312xxx | 20 m | 13 m | 1.1 m | 60° × 45° |
| Poison Horn Beetle 205000 | 20 m | 13 m | 1.3 m | 90° × 45° |
| Undead Steelworker 149010 | 30 m | 13 m | 1.3 m | 60° × 45° |
| Hollow Prisoner 127003 | 20 m | 13 m | 1.0 m | 60° × 45° |
| Pot prisoners 127030 (logic 127090) | 50 m | 50 m | 50 m | 60° × 45° |

Game-wide, only 16 of 333 shared LogicComParam rows differ between 1.12 and Scholar; none is an Earthen Peak enemy.
Only the maidens change a field whose meaning is known:
- 105010, 105030 Amana Shrine Maidens: sight lit + dark 40 -> 30 m, `Unk68` (online sync radius) 0 -> 1.
- 305010, 305011 Smelter Demon (two rows), 318010 Pursuer, 506110 Darklurker, 600010 Ancient Dragon, 619110
  Executioner's Chariot, 627010 Nashandra, 628010 Royal Rat Authority: `Unk36` 3 -> 4 (unknown; same edit on all eight
  bosses).
- 132010, 132030, 132040, 132100 Skeletons: `Unk28` 0.5 -> 0.35, `Unk2C` 0.4 (0.2 on 132100) -> 0.5 (unknown).
- 113020 Varangian Sailor (Scimitars): `Unk24` 3 -> 9 (unknown).
- 900010 (no name in the community sheet): `Unk43`, `Unk66`, `Unk67` 0 -> 1 (unknown).
Names are row id - 10 in the community enemy sheet (the logic row sits 10 above its enemy row), so treat them as likely.

## 2. Leash: unchanged

GeneratorParam `LeashDistance` is 50 m on almost every Earthen Peak spawn in both builds (1.12: 55 x 50, 3 x 40,
4 x 15; Scholar: 59 x 50, 5 x 40, 1 x 15, 1 x 20). No systematic change.

## 3. What Scholar did change

- **Placement**: 62 -> 66 enemies, a different mix (Manikins 15 -> 21, now 9 Claws + 6 Knife + 3 Sabre + 2 Bow + 1
  124070; Steelworkers 9 -> 4; Beetles 17 -> 12; new Undead Supplicants x3; Desert Sorceresses moved, one now stands
  with the bow Manikins).
- **Linked aggro groups** (GeneratorParam `AggroGroup`; SmithBox: entities in one group "become aggressive at the same
  trigger"): 1.12 links only beetles, pot prisoners and one Manikin pair. Scholar adds groups 38 (bow Manikin + Manikin
  124070 + Desert Sorceress), 41 (two Knife Manikins), 58 (Grave Warden + 2 Supplicants), 109 (two Claw Manikins).
  When one of these sees you, the rest come too, so a pack wakes from further away than any single member's sight.
- **Region links**: more Earthen Peak spawns carry map-region ids (`LocationEventID1` set on 7 -> 22). The logic
  scripts query regions (f7310 / f7020 / f7091 / f7092; meaning inferred, not confirmed), e.g. Manikin logic ai124010:
  think mode 100 + `f7310(0, 0)` -> straight to battle mode 15. Scholar put Knife Manikin 4100 in mode 100 with new
  region 50032000.
- **New AI modes** (GeneratorParam `AIThinkID` = mode group in the behaviour script): Scholar added Manikin 106 / 107 /
  110 / 111, Desert Sorceress 100, Grave Warden 410, Supplicant 110, and uses them here (Claw Manikins 111 x4 /
  106 x2, Sorceresses 100 x2, Grave Warden 410). These groups start when logic mode 15 (battle) is set or hold a pose
  (action 0) until noticed: they change how an enemy behaves once hostile, not its sight range.
- **Manikin give-up timer**: ai124010's return-home routine: >= 10 m from home and timer 0 >= **30 s** (1.12,
  hard-coded) -> **40 s** (Scholar, passed in as `sub(50, 0, 40)`; it was `sub(50, 0, 20)` with the 20 unused). Manikins
  chase about 10 s longer before going back. Logic scripts of Sorceress 316010, Grave Warden 312010, Beetle 205010,
  Steelworker 149010, Artificial Undead 139010: identical. Prisoner 127010 and Supplicant 147010 gained a new
  `f5900(7) == 66` path straight into battle (logic slot 7 = 66; who sets it not traced).

## 4. Light vs dark: same in both builds (checked 2026-11-02)

Question: does vanilla PC count Earthen Peak enemies as "in the dark" (13 m sight) where Scholar counts them lit (20 m)?
**No.** Code and data agree in both builds.

**Sight test.** Scholar FUN_14043BEF0, vanilla 1.12 FUN_00849660 (new Ghidra project `DS2 Vanilla PC\Ghidra Project`
DS2VPC, found by `scripts\SightHunt.java`: the only function that uses a 27.0 constant together with LogicComParam
+0x4 / +0x8 / +0xc / +0x32; on DS2SPC it finds exactly FUN_14043BEF0). Same logic line for line: closer than
`detect_distance` (+0xc) = noticed; range = `Unk04` (+0x4), or `sight_distance` (+0x8) when `Unk32` = 0 and the
enemy is unlit / `Unk32` = 1 and lit; an unlit enemy with range < 27 m sees a target holding a lit torch out to 27 m
(both constants 27.0); then the cone test.

**Lit flag** (`GXLightCollector`, per character; Scholar FUN_140148950 / 1401489E0, vanilla FUN_005D9B30). Each frame:
1. forced lit if a bright-location region set it this frame (Scholar FUN_1401EB230: a `MapGeneralBrightLocation` whose
   linked object reports lit);
2. else lit if the character's floor collision has a `MapHitBrightnessComponent` whose dark byte is 0 (Scholar
   +0x3c, vanilla +0x20);
3. else, once a second, lit if a character holding a lit torch (flag at +0x6d4, radius by torch level) is in range
   with a clear line (Scholar FUN_1401E6EB0).

**Where the dark byte comes from.** `MapHitBrightnessComponent` keeps a pointer to a `HitGroupParam_<map>` row (row id =
the collision part's hit group, MSB collision part int at entry end - 28, confirmed: every Earthen Peak value is a row
id). Dark byte = row byte 0 by default, byte 1 when switched (Scholar FUN_1401C8510, vanilla FUN_00644800). The switch
is event command `1:132151(hit group, state)` (Scholar dispatcher case at 0x140468116: command id 0x2040F + index 40).

**Earthen Peak data** (m10_17_00_00, both builds):
- HitGroupParam: 12 rows, bytes 0 / 1 identical in both: row 1 = dark by default (01 00), all others lit by default
  (00 01). Byte 2 is 12 in vanilla, 18 in Scholar on every row; it belongs to the row's other user,
  `MapHitMaterialSlotComponent`, not the brightness code.
- Collision parts: 83 in both, type data identical (3 differ only in string padding); hit groups 0 x70, 20, 21, 30 x2,
  40 x3, 50-100 x1. **None in group 1.**
- `1:132151` appears only in event_m10_18 (No-Man's Wharf) and event_m10_23, identically in both builds; never in
  event_m10_17.

So Earthen Peak floors are lit in both builds, and enemies there use `Unk04` (20 m, Sorceress / Steelworker 30 m) in
both. The aggro difference the author sees is not a light-rule difference; section 3 (linked groups, region triggers, new
modes, placement) remains the explanation from the data. Not tested live.

Still open: function meanings in the ESDs (no names; inferred from use).

## 5. Linked aggro groups game-wide (2026-11-02)

Regular enemies only (EnemyParamID < 500000 and a logic script; no bosses, NPCs, phantoms, so DLC maps, whose enemies
mostly have 6xxxxx ids, are undercounted, and the DLC is unchanged in Scholar anyway). "Linked" = sharing a non-zero
`AggroGroup` with at least one other regular enemy on the same map.

Game-wide: 500 of 1,239 enemies linked (40 %, 157 groups) in vanilla 1.12 -> 623 of 1,460 (43 %, 198 groups) in
Scholar. About a quarter more linked enemies, but mostly because there are more enemies; the share barely moves.
Groups stay small and tight in both (median spread mostly 5-15 m).

| area (map) | linked share 1.12 -> Scholar | groups |
|---|---|---|
| Iron Keep / Belfry Sol (m10_19) | 5 % -> 20 % | 1 -> 5 |
| The Gutter / Black Gulch (m10_25) | 28 % -> 49 % | 9 -> 13 |
| Shaded Woods / Shrine of Winter (m10_32) | 22 % -> 41 % | 3 -> 7 |
| Harvest Valley / Earthen Peak (m10_17) | 28 % -> 39 % | 8 -> 14 |
| The Lost Bastille / Sinners' Rise (m10_16) | 30 % -> 42 % | 11 -> 18 |
| Brightstone Cove Tseldora (m10_14) | 38 % -> 47 % | 6 -> 16 |
| Shrine of Amana (m20_11) | 10 % -> 18 % | 2 -> 8 |
| Memories of Vammar / Orro (m20_10) | 68 % -> 78 % | 10 -> 13 |
| Dragon Aerie / Dragon Shrine (m10_27) | 42 % -> 4 % | 6 -> 1 |
| Heide's Tower of Flame (m10_30) | 40 % -> 0 % | 1 -> 0 |
| Heide's / Cathedral of Blue (m10_31) | 38 % -> 22 % | 2 -> 1 |
| Drangleic Castle / King's Passage (m20_21) | 39 % -> 26 % | 13 -> 7 |
| Forest of Fallen Giants (m10_10) | 63 % -> 52 % | 25 -> 26 |
| No-man's Wharf (m10_18) | 40 % -> 32 % | 7 -> 10 |
| Aldia's Keep (m10_15) | 31 % -> 20 % | 1 -> 1 |

Unchanged: Majula (m10_29), Doors of Pharros, Undead Crypt (one regrouping), Dark Chasm, DLC. Huntsman's Copse,
Things Betwixt, Majula (m10_04), Grave of Saints: small changes. `AggroGroup` meaning (enemies in one group turn
hostile together) is the SmithBox community description, not checked in code.
