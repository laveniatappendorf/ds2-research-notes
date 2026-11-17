# Distance damage cut ("tip hit") and multi-target damage, PC Scholar (2026-11-01)

!!! success "Status: Confirmed in code"
    Read from the game's code or logic (disassembly or data), but not run live. See [status labels](../status-labels.md).

Question: when a swing passes through a group, the second enemy seems to take less damage. Is there a multi-target
stat?

Answer: no per-target rule was found. The drop comes from a distance cut between attacker and target, set per attack
in DamageCtrlParam. Modders call it "tip hit", but the code never looks at which part of the weapon connected.

## The fields (DamageCtrlParam, 192-byte rows)

| Offset | SmithBox name | Typical weapon value |
|---|---|---|
| +0xa4 | tip_hit_min_distance | 1.25-3.9 m, grows with reach |
| +0xa8 | tip_hit_max_distance | 1.5-4.2 m |
| +0xac | tip_hit_damage_mult | -30 (spears -20) |
| +0xb0 | Unk37 | -70 (some -50 / -30 / 0) |
| +0xb4 | tip_hit_posture_damage_mult | -70 |
| +0xb8 | tip_hit_poise_damage_mult | -70 |

1,803 of 1,893 player weapon hitbox rows use a ctrl row with a nonzero cut.

## Code (DarkSoulsII.exe 1.0.3.0, DS2SPC Ghidra project)

- Lookup chain: damage tables = DamageParamContainer at [[0x1416148f0] + 0x10] + 0xb40 (slot k at +0x10 * (k + 1),
  names from FUN_14048ba80). The DamageCtrlParam getter is FUN_140170be0(ctrl id), which goes through
  thunk_FUN_141ca22b0 into the protected (anti-tamper) code. The main damage-row getters are protected the same way,
  which is why plain offset searches missed the code.
- Hit driver: FUN_1401345c0 = ChrDamageActionCtrl vtable (0x1410bf178) slot 5. It runs once per character hit and
  builds a fresh {damage row, ctrl row} record, then calls the slots below. PlayerDamageActionCtrl (0x1410c9ce8) and
  EnemyDamageActionCtrl (0x1410c2888) share slots 10 and 32; the enemy slot 14 (FUN_14017aa50) calls the shared one.
- Slot 10, FUN_140136ab0 (hit record): d = straight 3D distance between the attacker's origin and the target's
  origin. The attacker is the owner found by FUN_140139a80, which walks bullet to owner; its world transform comes
  from FUN_14040d770, and the target position from vfunc +0x148.
  - If d > [target + 0x40] + 0x35c (a per-target threshold, not identified), then
    lo = hit[+0xdc] * min and hi = hit[+0xe0] * max (two per-attack scales, not identified);
    t = 0 when d <= lo, (d - lo) / (hi - lo) between them, and 1 when d >= hi.
  - Otherwise t = 0. t is stored at record +0x78.
- Slot 32, FUN_140138d50 (HP damage, stored as hit result +0x34):
  damage = (sum of type terms) * (1 + ctrl[+0xac] * t * 0.01) + one unscaled term. The 0.01 constant is
  DAT_1410acb08. ctrl[+0x13] (is_instant_kill) makes the damage equal to the target's HP.
- Slot 14, FUN_140137aa0: three more hit values get v * (1 + field * t * 0.01) with +0xb0, +0xb4 and +0xb8 (outputs
  +0x38 / +0x3c / +0x40 via vfuncs +0x110 / +0x118 / +0x120; not checked which is poise or posture).

So a straight sword (1.4-2.0 m, -30 %) does full damage up to 1.4 m between the two characters, -15 % at 1.7 m and
-30 % from 2.0 m. Poise and posture fall the same way, by up to 70 %.

## Not covered

- No counter of targets already hit appears in slots 5 / 10 / 13 / 14 / 32. Several multipliers in slot 13 come from
  vfuncs (attacker / target pairs) that were not traced, and the protected getters were not read. So this is "not
  found", not "proven absent".
- Not seen live. A test: two enemies at the same distance should take the same damage. Moving one enemy past the
  band should cost 30 %.
- The hitbox-section multipliers (PlayerDamageParam damage_id_followup chains: polearm shaft x0.4625, UGS root,
  whip sections) are a separate, per-hitbox mechanism and were not traced in code.
