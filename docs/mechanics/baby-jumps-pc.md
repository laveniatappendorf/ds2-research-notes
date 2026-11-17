# Baby jumps (PC Scholar 1.0.3.0)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

First written 2026-11-01 (code); rewritten 2026-11-17 after live tests on flat ground and at a Dragon Shrine ledge.

Speedrunners call a jump that falls well short of the usual distance a "baby jump". The usual explanations are
pressing jump too close to the edge, and noise in the game's speed calculation. Both turn out to describe something
real, but they are **two different mechanisms**:

1. **A low-speed jump.** The jump's forward speed is fixed from your running speed on the frame the press registers.
   A slow reading gives a slow, short jump. This is the classic baby jump, and the one the community no-baby-jump
   mods address.
2. **A late jump.** The push that launches you fires 11 frames after the press, even if you have already run off the
   ledge by then. Launched in mid-fall, the arc is low and falls short, although the forward speed is normal.

There is still no "distance to the edge" term anywhere in the jump code. The edge matters through timing.

## Summary

| | Low-speed jump | Late jump |
|---|---|---|
| What is short | forward speed (6 m/s instead of up to 8.5) | height of the arc (starts lower, weaker lift) |
| Caused by | a speed reading below about 6.02 on the press frame, 5.9 or less for the shortest (easing off sprint, jumping within about 0.3 s of starting to sprint, uphill slopes, noise) | pressing so close to the lip that the 8-frame wind-up carries you off it |
| Status | confirmed live (eased off sprint); sprint start from live speed readings; slope and noise causes from the code / community | confirmed live at a Dragon Shrine ledge |
| Fixed by the no-baby-jump mods? | the noise part, yes | no: the forward speed is already full |

## How a jump plays out, frame by frame (live)

Seen identically in all 20 logged jumps. Frames on this page are the game's own frames at 60 fps (59.1 measured, one
log record per frame), not the 30 fps frames often used for i-frames: halve the numbers for that convention.

| Frame | What happens |
|---|---|
| 0 | The press registers. The jump test passes and the forward push is fixed from the speed reading on this frame. |
| 1-8 | Wind-up, still on the ground. Speed drops to 5.1-5.2 m/s and you travel about 0.73 m. |
| 9 | You leave the ground with a small hop, about 0.85 m from where the press registered. |
| 11 | **The push fires**, about 1.0-1.15 m from the press. It *sets* horizontal speed to the push and *adds* 3.45 m/s to your current vertical speed. |
| 12 on | Horizontal speed is capped at 8.5 m/s, then slows by about 1 m/s per second; gravity takes about 13.8 m/s^2. |

Two consequences:

- The take-off you see is not the press. A character who looks clear of the ledge at launch may have pressed 0.85-1 m
  earlier, still on the ledge.
- The push fires on frame 11 whether or not you are still on the ground.
- Sprinting after the press changes nothing. The wind-up runs at 5.1-5.2 m/s in every jump, including one where sprint
  had already been released (reading 5.46), and the push then replaces your horizontal speed. You can't accelerate
  into a better jump after pressing.

## 1. The push speed (code, confirmed live)

FUN_140381db0 (PlayerGameParamCalculator slot 76, vtable 0x1410e4ec8):

```
reading = | horizontal velocity . facing |            (see section 2)
t       = 0 if reading <= 5.9, 1 if reading >= 6.15, else (reading - 5.9) / 0.25
push    = M x (6 + 5 t)          (ChrMoveParam row 100: dash_speed 5.9, dash_max_speed 6.15,
                                  jump_length_min 6.0 +0x12c, jump_length_max 11.0 +0x130)
upward  = jump_height 3.45 (+0x120)
```

- jump_length_min / max are **speeds in m/s**, not distances. Live, M = 1 (slot 77, FUN_140381cb0, modifier kind 0x18,
  not identified).
- Locked on, the thresholds are probably lock_on_run_speed 5.0 / lock_on_dash_max_speed 6.5 (the switch,
  `[[chr+0x3a0]]->vfunc+0x20`, is probably the lock-on target).
- A TimeAct override exists: with `[chr+0xb8]+0x4b8` bit 5 set, min / max / height / speed gate come from
  `[chr+0xb8]` +0x3a8 / +0x3ac / +0x3a4 / +0x3b8.
- These thresholds match B3LYP's 2017 experiments (5.9 / 6.15 / 6 / 11), written up in
  github.com/pseudostripy/bbj_mod_sotfs.

**The 8.5 m/s cap changes what "full" means.** The player's movement controller limits horizontal speed to 8.5 m/s
(section 6). The push is clamped on the next frame, so every push from 8.5 to 11 flies the same:

- Live: pushes of 9.00 and 9.71 on the same drop landed 5.95 and 5.96 m away. A downhill push of 11.0 also flew at
  8.48 m/s.
- So **any reading of about 6.02 or more gives a full jump**, not 6.15.
- A reading of 5.9 or less gives the minimum push, 6 m/s: about 70 % of a normal jump's flight speed.

## 2. The speed reading

- FUN_14031eb90 projects the velocity at P+0x30 onto the character's facing (Y zeroed). P = `[[PlayerCtrl+0x100]+0x10]`,
  a `ChrPhysicalProxy`; the obfuscated getter at 0x140312ad0 returns exactly this object.
- It's a physics velocity, not the movement controller's clean speed. Live, at full sprint on flat ground in a straight
  line (402 frames), it sat at **6.046-6.092** (median 6.081), swinging with the run cycle. The clean speed
  (`[[PlayerCtrl+0xF8]+0xF0]+0x3798`) stayed at 6.15.
- That clears the 6.02 full-jump threshold by only 0.03-0.07 m/s. No steady-sprint frame in our logs fell below it, so
  noise alone never shortened a jump here. B3LYP measured about 3 % random baby jumps at low CPU load and more at high
  load; load was not varied in our tests.
- The community no-baby-jump mods (bbj_mod_sotfs hooks exe+0x381e1c and loads the clean speed instead) replace only
  this number. They make the reading 6.15 at full sprint, so noise can't shorten a jump.

**Starting a sprint (live readings).** Going from a run (4.28 m/s) into a sprint, the reading reaches about 5.86 on
the first frame, so the character looks fully sprinting straight away. The last few hundredths of a m/s take much
longer:

| Frames after the sprint starts | Reading | Push it gives (section 1) |
|---|---|---|
| 1-2 | 5.86-5.88 | 6.0 m/s, the minimum |
| 3-11 | 5.90-5.98 | 6.0-7.6 m/s |
| 12-17 | 5.98-6.00 | 7.6-8.0 m/s |
| 18 on | 6.02 or more | full (8.5 m/s after the cap) |

- Of 27 sprint starts from a run in the logs, 26 took 15-20 frames to reach 6.02 (20 of them exactly 18, about
  0.3 s). One took 4.
- So a jump pressed in the first 0.3 s of a sprint is a low-speed jump, even though it looks like top speed on
  screen. No jump was pressed during a sprint start in our tests: the shorter push follows from the formula in
  section 1, not from a logged jump.
- The no-baby-jump mods barely change this. The clean speed they read starts at 5.91 and climbs a steady 0.0074 m/s
  per frame (0.44 m/s^2): 6.02 after 15 frames, 6.15 after 32. With the mod, the short-jump window is about
  15 frames instead of 18.

**Easing off sprint** lowers both readings: sprint decelerates at 0.15 m/s per frame (dash_deceleration 9.0). Live, a
press 4 frames after letting go read 5.46 and gave the minimum push. From the last full-speed frame to the press, the
readings were 6.08, 6.00, 5.79, 5.62 and 5.46: one frame of slowing still gave about 8.0 m/s, and the second already reached the
minimum. The mods don't change this, because the clean speed falls too. Below 4.4 (PlayerCommonParam
min_speed_for_jump, checked on two speed measures) there is no jump at all.

- With separate sprint and jump inputs held together (back paddles, keyboard and mouse), sprint stays held up to the
  press and none of this applies.
- If sprint has to be let go to press jump, the reading falls at this rate from the moment sprint ends. Whether a
  quick release and re-press of a shared sprint / jump button ends the sprint at once wasn't tested: the logger doesn't
  record buttons. Thumb-only jumps aren't usually short in practice, which suggests the game keeps the sprint going
  briefly, but that's a guess.

## 3. When a press counts: the ground test

The jump request test FUN_14038e810 needs:

- the jump input (bit 6 of `[input+0x128]`)
- both speed measures >= 4.4 x the TimeAct speed rate
- stamina above 0
- **"supported"**: P+0xC4, via FUN_1403144c0 -> FUN_14036da20

P+0xC4 is set after each physics step (FUN_14036dc50) from the support query FUN_14036f0f0. You count as supported if
either of these holds:

- **The body's own contact.** The character body had a contact this step, and the contact point is within 0.25 m of
  the root, measured horizontally. There is no angle check on this path.
- **A sphere probe.** A sphere of radius 0.25 m is cast 0.25 m down from 0.25 m above the root. It needs a contact at
  most 0.1 m below the root, on a surface at most 45 deg from flat.

The 0.25 m comes from half of ChrCommonParam hit_capsule_radius (0.5 m, read live at P+0x20).

- So the root can be **about 0.25 m past the lip and a press still counts.** While it rolls over the edge, the game
  keeps it glued to the lip and lowers it by up to 7 cm.
- **The jump test reads the support flag one frame old.** Live, presses registered 0.26 m and 0.32 m past the lip, on
  frames whose post-step support read "off". The three force-support counters (P+0x100 / +0x104 / +0x108) were zero
  in every frame.
- **Why the read is one frame old: frame order.** Each frame runs character logic, then the Havok step, then the
  physics results are read back.
  - The support flag has one writer: the post-step character update (PlayerCtrl vtable slot 16, FUN_140314e90 ->
    FUN_14036dc50).
  - The jump test runs in the input handler, earlier in the frame (ChrPadManipulator::Update, driven by
    PlayerOperator `[PlayerCtrl+0xE8]`).
  - So it sees the previous step's answer. Its exact place in the first stage is behind an obfuscated thunk and wasn't
    followed.
- This is the same root as the hitbox lag on the [phantom hitboxes](phantom-hitboxes-pc.md) page (physics results
  reach game logic a frame later), but on different bodies.
  - Ground support uses the character body and probe, not the 11-part hit model.
  - For the player, the read is in step with the screen: the image on screen at the press is the previous step's
    result, which is the state the test uses (display latency not measured).

## 4. Slopes

**Sprint slope rule (code only).** FUN_140336780 reads ChrMoveParam +0x270..+0x2BC: five groups of (start angle, end
angle, A, B). The two groups probably used for sprinting (+0x290, +0x2B0) are 10-40 deg with A = 10. On a slope in
that range, it sets TimeAct rate slot 3 to `1 - 10 % x t`, where t = (slope - 10) / 30. Larger slope changes restart a
smoothstep blend instead of applying at once.

- Only one direction counts. The sign test reads as uphill (not confirmed).
- If this rate scales sprint speed, a steep enough uphill run-up would push the reading below 6.02 and shorten the
  jump. Not tested live: the rate stayed at 1.000 in all 26,604 logged frames.

**Downhill (live).** A flight of stairs near the Dragon Shrine ledge is a smooth 29.8 deg ramp in the collision.
Sprinting down it raised the reading to 6.25 (push 11.0), and the slope rate stayed at 1.0.

## 5. Late jumps at a ledge: the Dragon Shrine example (live)

The ledge: you sprint along a walkway and jump a 6 m gap to a parallel walkway, which carries an arch underneath.

| Part | Collision |
|---|---|
| Near walkway | flat to the lip (0.0 deg, -0.3 deg at most), no kerb |
| Gap | 6.0 m; floor 8.7 m below |
| Far walkway | 3 m wide, 1.35 m lower than the near one, a coping ridge on each edge; underside 2.6 m below the near walkway, with the arch opening under it |

The body's front reaches the far walkway when the root is about 5.4 m past the lip (capsule radius 0.5). What decides
the jump is the root's height there:

- Above about 1.2 m below the near walkway (the top of the far walkway's coping ridge), the capsule rides onto the
  walkway.
- Below that, it hits the face, slides down and drops through the arch.
- In the log, the lowest landing was at -1.17 m and the highest miss at -1.26 m.

13 logged jumps (speed readings 6.05-6.14, pushes 9.0-10.8, so no low-speed jump). Heights are at 5.4 m past the lip:

| Press, relative to the lip | Result |
|---|---|
| 2.9, 2.7 m before | too early: the full arc is already 2.2-2.4 m down by 5.2 m out; miss |
| 1.56 m before | just too early: -1.26 m; miss |
| 1.45, 0.74, 0.67 m before | land (-1.17, -0.67, -0.69 m) |
| 0.25, 0.24 m before | land (-0.84 m): ran off the lip on frames 4-7, short fall, upward speed +2.8 m/s after the push |
| 0.07 m before | miss by about 0.1 m (-1.33 m): ran off on frame 3, upward +1.85 m/s |
| 0.08-0.32 m past | miss (-1.37 to -1.69 m): launched mid-fall, upward +0.9 to +1.6 m/s |

- A press near the lip runs off it during the wind-up and falls for up to 11 frames, sinking 0.2-0.36 m.
- The frame-11 push then adds 3.45 m/s to a fall of 1.1-2.5 m/s, so the arc peaks around the near walkway's height
  instead of 0.4 m above it.
- On screen, this is a small drop just before the jump, and the launch happens 1.0-1.4 m past the lip.
- **The window at this ledge is a press 0.25-1.45 m before the lip**: about 1.2 m, or 0.2 s at sprint speed.
- The ledge is unforgiving because the target sits only 1.35 m lower (its coping 1.2 m lower) and 6 m out, so the lower
  arc of a late jump has no margin. The press-to-launch delay turns a press "at the edge" into a launch past it.

## 6. The flight speed cap (code, confirmed live)

- 8.5 appears in the exe only as immediates. The one in character code is in **PlayerMovementCtrl**'s constructor,
  FUN_140385b40. It sets hard-coded defaults, with no param behind them:

| Field | Value | Role |
|---|---|---|
| +0x158 | 8.5 | default horizontal speed limit |
| +0x15C | 3.5 | limit in one state |
| +0x160 | 2.0 | limit in another state |
| +0x164 | 20.0 | limit in a third state |
| +0x154 | 10.0 | how fast the limit slides to its target (per s) |

- Its update, FUN_140385fe0 (0x140386c8a-0x140386dff), does this each frame:
  1. Picks the target limit from state-flag bytes.
  2. Moves the current limit (+0x150) toward it.
  3. If the horizontal velocity is above the limit, scales it down to the limit and keeps the vertical part.
- On the ground the limit never matters (sprint tops out at 6.15). In the air it clamps any push above 8.5.
- Live: PlayerMovementCtrl = `[[PlayerCtrl+0xE0]+0x148]` (`PlayerActionCtrl` +0x148), holding exactly these values.
- Which states pick 3.5 / 2.0 / 20 isn't identified.

## 7. Frame rate (not tested above 60 fps)

All the frame counts above are at 60 fps. What should happen with a frame rate unlocker, from what the values are:

- **Set in m/s or metres, so probably unchanged:** the push speeds (6-11 m/s), the full-jump reading (about 6.02), the
  8.5 m/s cap, the 4.4 m/s minimum and the 0.25 m support margin past the lip. The sprint slow-down of 0.15 m/s per
  frame is dash_deceleration 9.0 m/s^2 at 1/60 s, so at 120 fps it would be 0.075 per frame, the same per second.
- **One frame by nature, so should shrink:** the one-frame-old ground check (section 3). At 120 fps it would last half
  as long, and the extra distance past the lip it allows would roughly halve (about 0.1 m to 0.05 m at sprint speed).
- **Unknown:**
  - Whether the 11-frame push delay is counted in frames or in animation time (about 0.18 s). Animation time would
    make it about 22 frames at 120 fps, with the same timing and distance. The push wasn't found among the jump
    animation's timed events in c000100_pl.tae, so this isn't settled.
  - The noise in the speed reading, which comes from the physics step.
  - Whether the game scales its step to the frame time. In our logs gravity took exactly 0.229 m/s off the vertical
    speed every frame, which can't tell the two apart at a steady 60 fps.
- A 120 fps run of the flat-ground test (full-sprint jumps, an eased-off jump, a jump just after starting to sprint)
  would settle all three with the same logger.

## The community explanations, mechanism by mechanism

| Explanation | What the code and tests show |
|---|---|
| "Pressing jump too close to the edge" | Right about the timing, though there's no edge term. The launch happens 11 frames after the press, so a press near the edge launches past it, mid-fall, with a low arc (section 5). |
| "Baby jumps are random, from a noisy speed calculation" (B3LYP) | The reading is noisy, and the margin is small: 0.03-0.07 m/s above the true full-jump threshold of about 6.02. It didn't trigger in our steady-sprint frames; behaviour under CPU load wasn't re-tested. |
| "Releasing sprint before the jump" | Confirmed: 0.15 m/s lost per frame. One frame of slowing still gave about 8 m/s; two reached the minimum push. |
| "It baby jumped even though I was at top speed" | Two hidden cases. In the first 0.3 s of a sprint the character looks at full speed but the reading is still below 6.02 (section 2). And at a steady sprint, noise could push it under (not seen in our logs). Sprinting on after the press can't help either way. |
| The no-baby-jump mod | Removes the noise. It doesn't help a late jump or a deliberate ease-off, and only shortens the sprint-start window from about 18 frames to 15. |

## Method

- **Code:** headless Ghidra on the 1.0.3.0 exe, plus capstone. Constants read from the exe.
- **Live:** Steam Offline, a test character, `tools\ledgelog_pc.py` (read-only, one record per game frame). It logs
  the support flag, the speed reading, the slope rate, the jump vector (ChrPadManipulator +0x70) and the root position.
- **Placement:** jumps placed against the map collision with `tools\ledge_jumps_pc.py`. `tools\hkx_collision_pc.py`
  reads the Havok 2014 collision. World = collision + (-712, 12, -434) for m10_27, the collision parts' map position.
- **Logs:** findings/ledgelog-2026-11-17 (8,869 frames of flat-ground jumps, 17,735 frames at the ledge). The sprint
  starts in section 2 are the 27 run-to-sprint starts in the same logs.

## Open

- Which states use the 3.5 / 2.0 / 20 m/s limits.
- The slope rule live: does an uphill run-up cut sprint speed, and by how much?
- The noisy reading under CPU load.
- M (slot 77), and the exact call site of the input handler inside the first part of the frame.
- Frame rate: frames or animation time for the push delay, and the support lag and reading noise at 120 fps
  (section 7).
- Whether a quick release and re-press of a shared sprint / jump button costs speed (needs button logging).
- A jump pressed during a sprint start, to confirm the short push live.

Rules note: the speedrun.com DS2 forum post "No Babyjump Mod is allowed." (about 2019) reports a Speedsouls vote
19 / 37 **to allow** the mod for DS2 and SotFS leaderboard runs. Current category rules weren't checked.
