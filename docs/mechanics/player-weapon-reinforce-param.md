# PlayerWeaponReinforceParam (PC Scholar only)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Written 2026-11-19. The author asked whether `PlayerWeaponReinforceParam`, which is unused in Scholar, had been in use in
the Network Test. Builds checked: the US and JP Network Tests, PS3 v1.00 (US / Asia / Japan), Xbox 360, PS3 patches
1.01-1.03 and 1.08-1.10, PC vanilla 2014-03-31 / 2014-04-24 / 2014-06-02 / 1.02 / 1.03 / 1.12, PS3 Scholar and PC
Scholar 1.0.3.0. Params read with the shared `paramdef.py` reader.

## Short answer

- **It is not in the Network Test, in any form.** No such param file, and no param of its internal type
  (`EQUIP_REINFORCE_PARAM`) under any other name, in either Network Test build.
- **It exists in PC Scholar and nowhere else** — not in any PS3 or Xbox 360 build, not in any PC vanilla build, and
  not in PS3 Scholar, which is the old-generation codebase. It is one of ten params the 2015 current-generation
  rebuild added, so it is newer than the Network Test, not a leftover from it.
- **It looks unused in PC Scholar too.** `DarkSoulsII.exe` carries a plain UTF-16 table of param names
  (`WeaponReinforceParam`, `ArmorReinforceParam`, `ReinforceCostParam` and the rest are all in it). The string
  `PlayerWeaponReinforceParam` appears nowhere in the exe, in ASCII or in UTF-16 either byte order.
- Its three rows hold data, not placeholder junk, but no field in it has a published name and its purpose is not
  established.

## 1. Where it is, and is not

| Build | Present |
|---|---|
| Network Test (US) | no |
| Network Test (JP, NPJB90625) | no |
| PS3 v1.00 (BLUS41045), Asia (BLAS50640), Japan (BLJM61113) | no |
| Xbox 360 (465307E4) | no |
| PS3 patches 1.01, 1.02, 1.03, 1.08, 1.09, 1.10 | no |
| PC vanilla 2014-03-31, 2014-04-24, 2014-06-02, 1.02, 1.03, 1.12 | no |
| PS3 Scholar (BLUS31528) | no |
| **PC Scholar 1.0.3.0** | **yes** |

Two further checks on the Network Test, in case it shipped under another name:

- No param in the NT regulation declares the type `EQUIP_REINFORCE_PARAM` in its header. The type appears in exactly
  one file in any build, this one.
- Scanning `GameData.bdt` (986,680,276 bytes) for `PlayerWeaponReinforce` and `EQUIP_REINFORCE_PARAM` in ASCII and in
  UTF-16, both byte orders, returns nothing. Params live in the regulation, so this only confirms the obvious, but it
  rules out a stray copy in the archive.

## 2. It arrives with the current-generation rebuild

PC vanilla 1.12 has 199 params, PS3 Scholar 199, PC Scholar 205. PC Scholar adds ten and drops four:

| Added in PC Scholar | Dropped |
|---|---|
| `ChunkPhaseParam`, `MapObjectPlayGoParam` | `ItemLotParam` |
| `CameraAreaParam`, `CameraBattleParam`, `CameraControlParam`, `CameraElasticParam` | `ItemLotParam2` |
| `GeneralLocationEventParam_m20_10_00_00`, `_m99_80_50_04`, `_m99_97_00_00` | `ItemLotTicketParam` |
| `PlayerWeaponReinforceParam` | `ItemLotWeightParam` |

The first two are the install-in-chunks system ([ChunkPhaseParam and the PlayGo doors](chunk-phase-playgo.md)) and are
the only two of the ten whose names the exe does name.

## 3. The exe never names it

Eleven of PC Scholar's 205 regulation params have names that appear nowhere in `DarkSoulsII.exe`:

`CameraAreaParam`, `CameraBattleParam`, `CameraControlParam`, `CameraElasticParam`, `DbgPlayerStatusParam`,
`EnemyDefaultLogicParam`, `GeneralLocationEventParam_m20_10_00_00`, `GeneralLocationEventParam_m99_80_50_04`,
`GeneralLocationEventParam_m99_97_00_00`, `PhysicsTextureParam0`, `PlayerWeaponReinforceParam`.

That is a debug param, a default-logic param, three event params for cut or dev maps (m20_10 is the Giant memory,
m99 is dev space), a texture param and the four Camera params the rebuild added — company that fits dead data.

**This test does not transfer to the PS3 builds.** 44 of the Network Test's regulation params are likewise unnamed in
`EBOOT.elf`, including ones that are certainly live (`PlayerAttackParam`, `StaminaParam`, `ChrAttackParam`), so the
PS3 executables must reach some params another way. The Network Test conclusion above rests on the param not
existing, not on the exe.

The exe does carry the RTTI names `EquipmentReinforceLv` and `FeGroupNpcMenuReinforce`; neither is tied to this param
by anything found so far.

## 4. What is in it

860-byte file, little-endian, type `EQUIP_REINFORCE_PARAM`, three rows (ids 0, 1 and 2, no row names), 240 bytes per
row. SmithBox and DSMapStudio give the type 60 unnamed fields, 41 `s32` and 19 `f32`, named `Unk00` ... `UnkEC` after
their offsets only; the split below is read from the bytes, not from the definition.

Row 0 is all zeros. Rows 1 and 2:

| Offset | Read as | Row 1 | Row 2 |
|---|---|---|---|
| 0x00-0x30 | 13 × s32 | 100, 110, 121, 133, 146, 160, 176, 193, 212, 233, 256, 281, 309 | 50, 55, 60, 66, 72, 79, 86, 94, 103, 113, 124, 136, 149 |
| 0x34 | f32 | 100.0 | 100.0 |
| 0x38 | f32 | 0.0 | 0.0 |
| 0x3C-0x58 | 8 × f32 | 20.0 × 8 | 20.0, 20.0, 20.0, 20.0, 20.0, 20.0, 40.0, 30.0 |
| 0x5C | f32 | 0.0 | 0.0 |
| 0x60-0x7C | 8 × f32 | 20.0 × 8 | 20.0, 20.0, 20.0, 20.0, 20.0, 20.0, 40.0, 20.0 |
| 0x80-0xEC | 27 × u32 | 0 | 0 |

Both integer runs are the same curve at two magnitudes: start at 100 (row 1) or 50 (row 2) and repeat
`v = int(v * 1.1)` twelve times. That reproduces all 26 values exactly, truncation included.

The row is not `WEAPON_REINFORCE_PARAM` (244 bytes) with a field removed: read at the weapon layout's offsets the
leading integers are not floats of any sensible size.

## 5. Not established

- What any field means, what the 13 integers are indexed by, and what distinguishes rows 1 and 2.
- Whether the two eight-float runs are eight fields each or nine with a zero at one end.
- Whether anything in the game or in another param selects row 0, 1 or 2.
- Whether the PS4 or Xbox One Scholar builds use it. Neither is on disk; only PC Scholar was checked.

## 6. Adjacent: the Network Test's own weapon reinforcement

The Network Test does all weapon reinforcement in one table, and a wider one than any retail build:

| Build | `WeaponReinforceParam` rows | Row size |
|---|---|---|
| Network Test (US and JP) | 438 | 268 |
| PS3 v1.00 | 332 | 244 |
| PC vanilla 1.12, PS3 Scholar | 372 | 244 |
| PC Scholar | 374 | 244 |

24 bytes per row more than retail, six fields' worth, and about 100 rows more than v1.00. Not examined here.
