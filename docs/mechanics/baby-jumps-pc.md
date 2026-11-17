# Baby jumps (PC Scholar 1.0.3.0), 2026-11-01

!!! success "Status: Confirmed in code"
    Read from the game's code or logic (disassembly or data), but not run live. See [status labels](../status-labels.md).

Question: speedrunners say a "baby jump" (a jump of roughly 1/2 to 2/3 the usual length) comes from pressing jump too
close to the edge of a ledge. Is that in the code?

Short answer: **there's no edge or ledge term anywhere in the jump code.** Jump length depends on one live input: the
character's forward speed on the frame the jump registers. An edge can only matter if something at the ledge lowers
that reading. That part isn't settled statically (see "Open").

Static only (headless Ghidra on DS2SPC + capstone on the exe file). Nothing run live.

## 1. The rule (FUN_140381db0 = PlayerGameParamCalculator slot 76, vtable 0x1410e4ec8)

```
speed = | horizontal velocity . facing |          (FUN_14031eb90, see 2)
lo, hi = ChrMoveParam dash_speed 5.9, dash_max_speed 6.15
         (locked on, probably: lock_on_run_speed 5.0, lock_on_dash_max_speed 6.5)
t = 0 if speed <= lo, 1 if speed >= hi, else (speed - lo) / (hi - lo)
forward push = M x (jump_length_min + (jump_length_max - jump_length_min) x t)
             = M x (6 + 5 t)                        (ChrMoveParam row 100: +0x12c 6.0, +0x130 11.0)
upward       = jump_height 3.45 (+0x120) x 1 (or another constant when slot 5 is true)
```

- So 6 = the baby jump and 11 = the full jump. Any speed at or below 5.9 gets the floor of 6. The ramp is only 0.25
  m/s wide, so a dip of 0.25 m/s turns a full jump into a baby jump.
- The lock-on switch is `[[chr+0x3a0]]->vfunc+0x20` returning non-null. It's probably the lock-on target, but
  that isn't confirmed.
- M = slot 77 (FUN_140381cb0, modifier kind 0x18), which isn't identified.
- A TimeAct override exists: when `[chr+0xb8]+0x4b8` bit 5 is set, min / max / height / jump-speed gate come from
  `[chr+0xb8]` +0x3a8 / +0x3ac / +0x3a4 / +0x3b8 instead of the params.
- These figures match B3LYP's 2017 write-up (5.9 / 6.15 / 6 / 11, found by experiment). The write-up is in the repo
  github.com/pseudostripy/bbj_mod_sotfs.

## 2. Which speed (FUN_14031eb90)

- It takes the facing vector (chr vfunc +0x240) and zeroes Y and W with the mask {ffffffff, 0, ffffffff, 0}
  @0x141137170. Then it projects the velocity `[X+0x30]` onto that facing. X comes from getter 0x140312ad0, which is
  obfuscated and starts with `[chr+0x100]`.
- The result is the forward horizontal component, as an absolute value.
- It's a physics / proxy velocity. B3LYP measured it as noisy, worse under CPU load and on slopes.
- The community no-baby-jump mods only replace this number. bbj_mod_sotfs hooks exe+0x381e1c (the `addss` just
  before the `sqrtps` at 0x140381e24) and loads `[[[BaseA]+0xD0]+0xF8]+0xF0]+0x3798` instead, a clean speed. The mods
  don't touch any edge logic.

## 3. When it's sampled: on the jump request, in the pad input code

- ChrPadManipulator::Update (FUN_140367750, vtable slot 5 @0x1410e3f40) calls FUN_140367ea0 and then
  FUN_140367e20.
- FUN_140367ea0 raises request flag 0xd (bank `[this+0x18]+2`, setter FUN_140366020) when FUN_14038e810 is true and
  stamina > 0.
- FUN_140367e20 then computes the vector into `this+0x70`:
  - flag 0xd gives the normal jump, FUN_14031e590 (min/max/height as in 1).
  - flag 0xc gives FUN_14031f7d0, using ChrMoveParam +0x134 / +0x138, which are 0 / 0 (no forward push).
- So the speed is read on the frame(s) the request is raised, not at a later take-off event.

FUN_14038e810 is the jump request test. All three of these must be true:
- Input bit 6 of `[input+0x128]` is set.
- FUN_14038eb10: two speeds are both >= PlayerCommonParam `min_speed_for_jump` 4.4 x TimeAct speed rate
  (`[chr+0xb8]+0x11c`):
  - the 3D magnitude of `[chr+0xe0]->vfunc+0x1c8`
  - the 3D magnitude of the physics velocity `[X+0x30]`
- FUN_1403144c0(chr) passes. This goes `[chr+0x100]+0x10` -> FUN_14036da20, a support / ground-style check that
  isn't named.

Below 4.4 there's no jump at all. Between 4.4 and 5.9 you always get the minimum jump.

## Reading the speedrunner claim

- **Not supported as written.** No input in the length calculation depends on distance to a ledge.
- **What fits the code:**
  - The jump is decided by the speed at the moment the request registers.
  - Anything that slows that reading below 6.15 shortens the jump, and at 5.9 or below you get the 1/2-2/3 minimum.
- **Known ways to get there:**
  - Noise in the physics reading (B3LYP: about 3 % of random jumps at low CPU load, more at high load).
  - Slopes.
  - Easing off sprint. For B (circle) jumps, the brief release of B before the tap lets the speed fall from dash
    toward run/walk. The longer the gap, the shorter the jump, and below 4.4 the jump doesn't happen. This is a real
    "pressed too late" mechanism, but it's about button timing, not the edge.
- A ledge could still matter indirectly if the physics body's forward velocity dips as the capsule reaches the lip.
  The no-baby-jump mods only replace the speed reading, so if they also remove ledge baby jumps, that's consistent
  with an edge effect, if there is one, running through this reading.

## Open

- Live check: sample the reading (`[X+0x30]` along facing, or the result at ChrPadManipulator+0x70) every frame while
  sprinting at a ledge, then compare against flat ground.
- Name X (getter 0x140312ad0), the slot-77 modifier kind 0x18, the lock-on switch, and FUN_14036da20.
- Who consumes ChrPadManipulator+0x70, and whether the request flag stays up for more than one frame.

Rules note: the speedrun.com DS2 forum post "No Babyjump Mod is allowed." (about 2019) reports a Speedsouls vote
19 / 37 **to allow** the mod for DS2 and SotFS leaderboard runs. Current category rules weren't checked.

Tools (scratchpad, not kept): pedis.py (disasm / AOB scan), vt.py (vtable + RTTI name for an address),
callers.py / callslot.py (direct and vtable call sites).
