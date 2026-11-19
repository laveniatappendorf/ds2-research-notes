# Aava's leap: one move, stretched to fit the distance (PC Scholar 1.0.3.0, 2026-11-10)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

The author asked whether Aava's leap-and-strike, which sometimes lands on you however early you turn and sprint, is
several moves or one move with a variable distance. Read from the files only (no live test). Aava and Lud & Zallen
share the behaviour row 679000, AI scripts ai679000 / ai679010 and the network c6790, so all of this applies to them too.

## Short answer

Both, in a way. There are **three leaping attacks** (AI attack ids 14, 15 and 16). Each one is a single move whose
length is **blended between a short and a long version of the leap by your distance from her**. The game also **keeps
re-measuring that distance during the wind-up and most of the flight**, so running straight away only makes the leap longer.

## 1. The blend (c6790.nmb)

Each leap's first animation is a BlendN node with two children, the plain clip and an `_L` clip (same length, same
TAE id). The blend weight comes from the control parameter `Target_Distance`:

| attack | clips | fully short at | fully long at | clip length |
|---|---|---|---|---|
| 14 | a65_14_1010 / _L | 9 m | 20 m | 2.50 s |
| 15 | a65_15_1010 / _L | 9 m | 25 m | 2.50 s |
| 16 (leaping grab) | a65_16_1010 / _L | 9 m | 25 m | 2.83 s |
| 10, 11 (lunge into claw combo) | a65_10/11_1010 / _L | 5 m | 7 m | 3.33 s |
| 19 | a65_19_1010 / _L | 5 m | 10 m | 2.67 s |

Between the two points the mix is linear. Beyond the far point the long leap is used whole and the leap cannot
stretch further.

Attack 16 is the grab: its state machine moves on to a65_16_1020 (two grab hits, TAE 2201 at 1.50 s and 3.73 s)
only on the `CatchAttack` request, which the update code sends when the catch connects. Attacks 14 and 15 have the
same timing as each other: one landing hit at 0.73-1.10 s (14) / 0.73-1.13 s (15).

## 2. Distance is re-read while the leap plays (code + network events)

- `FUN_1403a4a60` names the enemy control parameters; index 0x12 = `ControlParameters|Target_Distance`.
- `FUN_1403a4e70` (called from the network driver's update `FUN_1403a4950`) writes index 0x12 =
  sqrt(dx² + dy² + dz²) between the enemy (`[chr+0xB8]+0x30`) and its target, **only while the counter
  `[chr+0xB8]+0x504` is above 0**. Otherwise the parameter keeps its last value.
- That counter is raised and lowered by morpheme duration event **125** (`FUN_140368e70`, case 0x7D: +1 on start,
  -1 on end; case 0x7E / +0x508 does the same for Target_X / Target_Y).
- In c6790 the event sits on a track named **TargetDistanceControl**, from the start of each blended clip:

| attack | event 125 window (fraction of clip) | in seconds | landing hit opens |
|---|---|---|---|
| 14 | 0.000-0.400 | 0-1.00 s | 0.73 s |
| 15 | 0.000-0.360 | 0-0.90 s | 0.73 s |
| 16 | 0.000-0.235 | 0-0.67 s | 0.73 s |
| 10 / 11 | 0.000-0.350 | 0-1.17 s | 1.20 s (10) |
| 19 | 0.000-0.125 | 0-0.33 s | 0.80 s |

So for 14 and 15 the distance is re-measured through the take-off and the whole flight, into the landing hit;
for 16 it stops just before the hit. It is the live distance between her and you, so the blend works as a rough
homing: if you run away it moves towards the long leap, and as she closes in it moves back towards the short one.
Code reading only: the per-frame call is inferred from the driver's update method, not watched live.

**Correction (13 Nov): the times in sections 1-3 are clip times.** The game plays the take-off slower than the clip,
so in real time the measuring lasts longer: about 1.37 s (14), 1.27 s (15) and 0.83 s (16). See section 9.

## 3. Direction (TAE type 280100, most likely turn speed in degrees per second)

| attack | before take-off | snap | in the air | landing |
|---|---|---|---|---|
| 14 | 60 (0-0.40 s) | 360 (0.40-0.50 s) | 90 (0.50-0.83 s) | 180 (0.83-1.00 s), then 0 |
| 15 | 60 | 360 (0.40-0.50 s) | 90 (0.50-0.83 s) | 180 (0.83-1.00 s), then 0 |
| 16 | 60 (0-0.63 s) | 360 (0.63-0.73 s) | 90 (0.73-1.07 s) | 180 (1.07-1.20 s), then 0 |

Distance is corrected for almost the whole leap, but direction only turns at about 90°/s once she is airborne.
Running straight away is the one escape the leap is built to handle. Moving across her path after the take-off snap
is what her turning limits.

## 4. When the AI uses them (ai679000.esd)

The attack choice (`G2147483642`) rolls 1-100 by distance band. Roughly (a blocked choice re-rolls):

| your distance | leap 14 | leap 15 | leap grab 16 | all leaps |
|---|---|---|---|---|
| 8-15 m | 25 % | 15 % | 20 % | ~60 % |
| 15-19 m | 25 % | 15 % | 20 % | ~60 % |
| 19-25 m | 10 % | 20 % | 20 % | ~50 % |
| over 25 m | 5 % | 10 % | 10 % | ~25 % (attack 22, a shot, takes 65 %) |

Each leap routine (`G2147483629` / `G2147483621` / `G2147483627`): if you are within 20 m she just turns to face you
(within 90° for 14 and 16, 30° for 15) and leaps; beyond 20 m she first closes to 15 m. If you come within 9.5 m while
she lines up, the leap is dropped. So a leap almost always starts with you 9.5-20 m away, inside the blend range.

## 5. The hop back, and where you are safe from a leap (added later the same day)

**The hop back** is the step routine `G2147483640(65, 100, 5, 0, 7, 1)`: command 1:5221 with directions 180 / 150 / 210
and step distance 5 (the network's long back step, `a00_11_0031_StepBackRightLong`, StepDistance blend 0-5). It is
not an attack. The combat loop (`G2147483643`, state s3) rolls after every finished attack:

- roll 1-9: the step routine;
- roll 1-24, you within 5 m and 60° or more off her nose: the step routine;
- roll 1-24, you 5 m or further: strafing for 5-7 s (`G2147483639(5, 7, 4, 25)`, ends if you come within 4 m or pass 25 m);
- otherwise she picks the next attack at once.

The step routine does nothing unless you are **within 7 m**. Then 64 % of the time she steps back if you are within
90° of her front, otherwise she sidesteps. So after an attack the hop back comes about 6 % of the time with you in front
of her and about 15 % with you at her side within 5 m. Behind her she only sidesteps. **Corrected 13 Nov:** the step
blend runs from StepDistance 0 (`a00_11_0010_StepFrontRightShort`, no travel) to 5 (the long steps, **20 m** of
travel each, measured in the exported clips), and the hop back asks for 5. So the hop is most likely about 20 m, not
5 m: it leaves you roughly 20-27 m away, where her next roll is a leap about half the time (19-25 m band).

**Inside 9.5 m** a leap is not started. All three leap routines (`G2147483621` / `G2147483627` / `G2147483629`, states
s10 / s11) drop the leap if you are within 9.5 m while she turns or walks in. The close-range picks (modes 73 / 74 / 75,
from the 0-8 m bands) go through wrappers (`G2147483628` / `G2147483622` / `G2147483625`) that first turn her to you,
then switch to the lunge combos 50 / 51 (50 / 50) if you are within 6 m. From 6 m out they try the leap, which the
9.5 m check then drops. The check runs only while she lines up: once the leap has started, closing in does not stop it.
How the check is timed against the sub-routine's return is read from the script, not watched live.

**Beyond 25 m** is not safe. Over 25 m she picks a leap about 25 % of the time (the ranged attack 22 takes 65 %). Any
leap picked from over 20 m makes her walk in to 15 m first and leap from there. Since a leap only ever starts with you
9.5-20 m away, 25 m matters only mid-leap, and gaining 5-10 m in under a second while she closes in is not realistic.

## 6. What decides whether she leaps: dice, distance and cooldowns

Choosing a leap is a dice roll; the leap itself has no randomness. Inputs to the choice:

- **Distance band at the roll.** The roll happens when her previous action ends. If you are 60° or more off her nose
  and 10 m or further away, she first turns to face you (state s18, until within 15°), then rolls.
- **Angle** only matters inside 8 m (separate front / side tables), where leaps are dropped anyway (section 5).
- **Cooldowns.** Each pick is checked with `f5091(attack)`. Taken with [the Amana orb work](../builds/shrine-of-amana-pc.md#2-the-spellcasters-amana-shrine-maidens), that is the attack's
  cooldown being over. An unavailable pick sends her back to roll again, so its share goes to the others. Not confirmed
  in code: no instruction compares against 5091, so the ezstate dispatch is indirect. Her rows
  (EnemyBehavior*Param 679000, `atk_cooldown_min/max`): leaps 14 and 15 **none**, grab 16 **15 s**, attack 21 15 s,
  attack 22 (her main long-range attack) **25 s**, attack 23 20 s.

Leap share of the roll (approximate, rolls 1-99 by band):

| your distance | everything ready | 22 cooling down | 16 cooling down | both cooling down |
|---|---|---|---|---|
| 8-15 m | 61 % | 67 % | 51 % | 58 % |
| 15-19 m | 61 % | 76 % | 51 % | 68 % |
| 19-25 m | 51 % | 85 % | 38 % | 77 % |
| over 25 m | 25 % | 74 % | 17 % | 63 % |

So for 25 s after her long-range attack, staying far away makes a leap likely, not unlikely.

- **After the roll**: closing to within 9.5 m while she lines up drops the leap (section 5); from over 20 m she walks
  to 15 m first.
- **Not inputs**: her health. The only health check (`f5200() < 30`, for attack 20) sits behind
  `f5703() > 1 && f5703() < 1`, which can never be true, so that branch is switched off. Nothing identifiable in the
  script reacts to what the player did (healing, rolling), but most ezstate functions are unnamed.
- **Unnamed gate**: when `f5703()` is between 3 and 98, the logic script sets slot 5 to 37, which turns the grab off.
  The leap routines then also start with a turn routine (`G2147483619`). Possibly the number of players; not identified.
- Attacks 5 / 6 (close range) can't be used three times running: slot 6 / 7 bookkeeping in `G2147483618`.

## 7. The run-off, and a no-roll player (added later the same day)

The author is on a no-roll run and found sideways running useless; that matches the data.

**The run-off.** At the start of every combat cycle (`G2147483643`, state s2) she rolls 1-100. If you are **within 8 m**
and the roll is under 20 (about **19 %**), she runs to a fixed spot (`G2147483620`, command 1:7005 over a range of
EventLocation ids) instead of attacking:

- Aava (logic slot 5 = 0): `EventLocation_m50_37_00_00` rows **51000000 (-79.6, -23.4, -12.8)** and **51000001
  (-3.2, -23.4, -3.1)**, two spots about **77 m apart** in her arena (the arena entry is at (-88.5, -22.3, -30.0)). Which
  one she picks is decided by 7005 and can't be told from the script.
- Lud & Zallen (slot 5 set to 37 by the logic script): rows 51001000-51001003, the four corners of a roughly 40 x 22 m
  rectangle in the Expulsion Chamber (arrival (-379.0, -69.3, 351.6)). The same slot turns their grab off.

On arrival she turns to face you, holds while you are within 15 m in front of her, and stops after 5 s from the start
or when you come within 5 m. Then she rolls for an attack from wherever you are, usually the over-25 m band:
ranged attack 22 65 % if ready, leaps 25 %, or 74 % if 22 is cooling down.

**Rough leap risk per cycle while you stay within 8 m**: 19 % run-off x 25-74 % leap = about **5-14 %**. At 9.5-15 m
it is about 60 % every cycle. Inside 8 m is still far safer, but not leap-free.

**Sideways running can't beat the turn.** Running across her line at speed v swings her bearing at v / d radians per
second. At a sprint of about 6 m/s that is ~34°/s at 10 m, ~57°/s at 6 m and ~86°/s at 4 m. She turns at (most
likely) 90°/s in the air and 180°/s through the landing hit, after a 360°/s snap at take-off. So sprinting sideways
only out-turns her within about 4 m, when the landing turn is already 180°/s. Running straight away is matched by the
distance blend. With no roll, a launched leap can be outrun only when it starts near the top of its range and the long
clip can't stretch further. Sprint speed is taken from [the baby-jump work](baby-jumps-pc.md) (5.9-6.15); the root-motion length of the
`_L` clips is not decoded.

## 8. Run-off distances and the race (13 Nov)

The author asked how far the run-off goes, and whether running the other way gets you past 25 m before she leaps (a friend
reported outrunning the leap after the run-off).

**[Developer names](npc-invaders-pc.md#3-the-edd-files)** (PS3 Scholar `ai679000.edd`, 32-bit layout: base 0x94 as on PC; string table offset / count at
0xC0 / 0xC4 with 8-byte entries {u32 offset, u32 UTF-16 length}; functions at 0xA0 / 0xA4, 8-byte {s32 id, u16 string,
u16}; commands at 0x98 / 0x9C, 16-byte {s32 id, s32 -1, s32 0, u16 string, u16}):
- 1:7005 【索敵】ナビポイント_ターゲット設定 "set a nav point as the target";
- 1:5535 【移動】ダッシュ_ロック_ランダム "dash, locked on, random";
- 1:5221 【攻撃】ランダム回避_角度指定 "random evade with given angles" (the hop back);
- f5091 行動が実行可能か "can the action be performed";
- f5703 グループ人数 "group size" (the Lud & Zallen switch);
- f7018 目標未来位置との距離チェック "distance check against the target's predicted future position";
- f7001 distance to target, f7002 angle to target, f5200 HP, 1:7010 / 7011 save / load target.

**The run-off** (`G2147483620(51000000, 51000001, 15, 5)`): she saves you as her target, sets one of the two nav
points as the target (the choice rule inside 7005 is not readable from the script), and dashes to it. She stops
**within 5 m of the point, or 5 s after starting**, then reloads you as the target and quick-turns to face you
(at least 0.5 s if you are more than 45° off).

**Arena geometry** (map-local): arena entry (-88.5, -22.3, -30.0); spot A (-79.6, -23.4, -12.8), about 19 m from the
entry; spot B (-3.2, -23.4, -3.1); Aava's generator 907 (6.8, -16.7, -13.2), 14 m from B. The floor runs roughly
95 m east-west, and the spots are 77 m apart. From mid-arena each spot is about 38 m away, so the dash is about 33 m.

**Speeds and travel** (Master bone of the DS2AnimToolset glTF exports, `Export\c6790\motion_gltf`):
- her run (`a00_04_0010_Jog`, also the Wep01 version): 9.5 m per 0.633 s cycle = **about 15 m/s**; walk 2 m/s;
- the player's sprint is about 6 m/s (baby-jump work), so she runs about 2.5 times as fast;
- leaps:

| leap | short clip | long clip | travel happens |
|---|---|---|---|
| 14 | 6.5 m | 20.0 m | 0.4-1.5 s, most by 1.0 s |
| 15 | 7.9 m | 18.8 m | 0.4-1.5 s, most by 1.0 s |
| 16 grab | 6.0 m | 21.0 m | 0.6-1.2 s |

- Her head is 3.6-4.5 m ahead of her ground point through the hit window.
- The `Character` node also moves in the leap clips (14.5 m in 14 `_L`, 2 m in 15 `_L`, 1 m in 14 short, none in 16
  or in run / step clips). Ignoring it keeps the body 1.4 m behind Master in every clip, so it is taken as an
  export artefact. The Executioner's Chariot shows the same split (render project notes).

**The race, if you start where she does and run the opposite way** (mid-arena):
- She dashes about 33 m in roughly 2.2-2.5 s, then turns (0.5 s or more).
- You turn and sprint about 15-18 m in that time.
- So the gap at her next roll is about 50 m. Even standing still it would be about 33 m. You are past 25 m easily;
  the gap comes mostly from her own dash. Only when she is already near the spot she picked (fighting near an end)
  is the dash short enough to leave you within 20 m, where she can turn and leap at once.

**But 25 m is a roll band, not a safety line:**
- Over 25 m her ranged attack takes 65 % if ready. Leaps take about 25 %, or about 74 % if attack 22 is on its 25 s
  cooldown.
- A leap picked from over 20 m makes her dash back to 15 m (`G2147483615`): success = within 15 m (predicted
  position, f7018) and facing (s27, return 0). It **gives up after 5 s** (timer started on leaving s1; s43, return 1),
  and the leap routine then drops the leap (s12) and she rolls again.
- Closing at about 9 m/s (15 - 6) while you keep sprinting away, she reaches 15 m in under 5 s unless the gap at the
  start of her approach is about 60 m or more. In a 95 m arena that needs you at one end and her at the other, and
  sprinting the whole time (stamina, walls).

**Outrunning a launched leap: not settled from the files.** A frame-by-frame model was built: the short / long mix
recomputed from the live distance while event 125 runs, then frozen; her front = head / forepaw bones + 0.5-1.5 m;
the player running straight away at 5-6 m/s. It **fails its own control**: leaps 14 and 15 would miss a player
standing 12-20 m away, because the mix slides back to the short clip as she closes in. Leap 16 hits standing players
at every distance (its measuring ends before it leaves the ground) and misses players already sprinting by about
2 m. Missing pieces: the real capsule sizes (EnemyDamageParam radius x0.6-1.3, length x0.8-2.0 on the dummy points'
base sizes), and whether the game's distance is what the model assumes. A live measurement is needed: log her
position, the player's and the Target_Distance parameter through a few leaps. What does hold: the grab (16) fixes
its length at about 0.67 s, just as it leaves the ground, so it is the version a sprint started before take-off is
most likely to beat. 14 and 15 keep re-measuring to about 1.0 / 0.9 s (clip time; real time about 1.37 / 1.27 s, section 9).

## 9. Game speed: the take-off is played slowly (13 Nov)

The game doesn't play attacks at the clip's own speed (found in the render project, `DS2 Boss Move Renders\PLAN.md`):
- while duration event **150** (`AttackSpeedControl` track) runs, the clip plays at the behaviour row's `attack_speed`;
- while **151** runs, at its `recovery_speed`;
- otherwise at 1.0.

Aava's rows (`EnemyBehaviorSecondParam` 679000, groups 5-7):

| leap | start-up speed, clip window | recovery speed, clip window | take-off (real) | hit window, clip -> real |
|---|---|---|---|---|
| 14 | 0.5, 0-0.367 s | 0.5, 1.333-1.967 s | ~0.77 s | 0.733-1.100 -> **1.100-1.467 s** (0.367 s) |
| 15 | 0.5, 0-0.367 s | 0.45, 1.233-2.333 s | ~0.77 s | 0.733-1.133 -> **1.100-1.500 s** (0.400 s) |
| 16 grab | 0.8, 0-1.167 s | 0.75, 1.233-2.8 s | ~0.79 s | 0.733-1.133 -> **0.917-1.417 s** (0.500 s) |

- **Leaps 14 and 15:** the crouch before the jump lasts twice as long as the clip suggests. The landing itself plays at normal speed.
- **The grab (16):** slowed by 0.8 right through its hit window, so the window is 0.5 s long.
- **Measuring (event 125) in real time:** 14 to 1.37 s, 15 to 1.27 s, 16 to 0.83 s. So for 14 and 15 your distance is still being
  read 0.27 / 0.17 s into the hit window.
- **Turning (section 3) in real time:** the 360°/s snap is at about 0.77-0.87 s (16: 0.79-0.92 s), if those values are per real second.
- **Not measured live.** The rule comes from the toolset's event tooltips, and the render project applies it to every boss.
  Sections 7-8 and race.py used clip times. A start-up twice as slow gives a no-roll player about 0.37 s more warning
  before take-off than they assumed. The [live test plan](../tools/aava-leap-live-test-plan.md)'s Run B / C timings should use these real times.

## 10. Rolling the leap with under 92 AGL (13 Nov, model)

The author asked whether the leap can be rolled at all below 92 AGL, given the hitbox duration and the i-frames.

**Answer: yes.** A well-timed roll avoids every version from almost every spot from 88 AGL up, and from most spots even
at 85. 91 → 92 changes almost nothing, so 92 isn't a threshold for this attack. The fact that the hit window outlasts
your i-frames doesn't decide it. The hit only lands if a capsule still touches you after your i-frames end, and her
capsules move with her body while your roll moves you.

**Hit windows vs i-frames (PC, 60 fps).** Hit windows from the TAE, with section 9's speeds:

| leap | capsules (TAE 2202 rows + follow-ups) | later capsules (TAE 2200 = behaviour damage ids, inferred) | real window |
|---|---|---|---|
| 14 | 470, 480+485, 490+495 | 410+411 (1.200-1.467 s) | 1.100-1.467 s, 22 frames |
| 15 | 570, 580+585, 590+595 | 510+511 (1.267-1.500 s) | 1.100-1.500 s, 24 frames |
| 16 grab | 970, 975, 980+985, 990 | 910+920 (1.208-1.492 s) | 0.917-1.492 s, 34 frames |

Your [roll i-frames](agility-iframes.md) (counter frames): `floor(60 x max(0.133, 0.1 + 0.467 r)) - 2`, live ±1. That gives:
- 85 AGL: 11-14 (0.18-0.23 s);
- 88: 16;
- 90-91: 17;
- 92: 18 (0.30 s);
- 99: 22;
- 120: 32.

To stand inside a capsule for its whole window you'd need about 98 AGL (14) or 102 (15), and no AGL is enough for 16.
So nobody outlasts the window; every successful roll gets out of the capsules.

**Capsule sizes are plain metres.** EnemyDamageParam `hitbox_radius_mult` / `hitbox_length_mult` are the live radius
and length. The 6 Nov hit log matched them exactly for the hollow soldier and large knight rows; the capsule runs from
the dummy point along its forward vector. Aava's dummy points (c6790 FLVER, parent bones):

| dummy | bone | capsule |
|---|---|---|
| 249 | Spine1 | r 1.3 m, 2.0 m long (her whole torso) |
| 7 | Head | sphere r 1.2 m (grab: capsule r 0.8, 0.7 m) |
| 10 / 20 | R / L forearm | r 0.7-0.8 m, 0.8-1.0 m |
| 21 | L upper arm | r 0.9 m, 0.8 m |
| 50 / 60 | R / L hind leg | r 0.6 m, 1.2 m |

Her hind legs and paws sweep along the ground through the whole window (4-7 m in the short version, about 10 m in the
long one). The torso comes down to 1.6-1.9 m, so its capsule's underside is at about 0.3-0.6 m.

**Model** (`tools\aava_roll\`):
- **Her capsules.** Placed on the glTF exports of the six leap clips (short and `_L` for each leap), with the section 9
  game speed applied.
- **Your body.** A real logged roll (6 Nov run 6, roll 1752: 1.71 m at 15 frames), fitted per frame as one upright capsule.
- **Hit rules** ([6 Nov live log](phantom-hitboxes-pc.md#live-test-2026-11-06-pc-scholar-1030-main-steam-install-steam-offline)):
  - contact is tested on the previous frame's poses;
  - a hitbox's first frame never hurts;
  - the current frame's i-frame counter decides;
  - a contact on the last protected frame lands on the next frame.
- **What's tested.**
  - Grid of standing spots 0.5 m apart, keeping every spot where standing still gets hit.
  - 8 roll directions.
  - Roll starts from 20 frames before to 2 after the frame you'd be hit.
  - A start counts only if the same roll with no i-frames gets hit, so it is the i-frames that save you, not moving out early.

**Results** (share of the spots she hits where some roll direction works; short version / long version):

| AGL (i-frames) | 14 | 15 | 16 grab | timing leeway, best direction (median frames) |
|---|---|---|---|---|
| 85, min investment (11) | 77 / 96 % | 75 / 85 % | 95 / 99 % | 6-9 |
| 85, 5 points (14) | 93 / 100 % | 88 / 93 % | 98 / 100 % | 9-12 |
| 88 (16) | 99.5 / 100 % | 96 / 98 % | 99 / 100 % | 12-15 |
| 90-91 (17) | 100 / 100 % | 98 / 99 % | 99 / 100 % | 13-16 |
| 92 (18) | 100 / 100 % | 99 / 99.7 % | 99 / 100 % | 14-17 |
| 99 (22) | 100 % | 100 % | 100 % | 19-21 (the cap of the test) |

- **Where no roll works:** right under where her body comes down, between her hips and her head.
  - 14 and 15: about 1-2 m to her right of the line, 6-9 m out (short) or 15-20 m out (long).
  - Grab: 2.5-3.5 m out, under her hind legs and belly as she takes off.
  - There you are inside the torso capsule or the head sphere for longer than low-AGL i-frames last.
- **Direction:** rolling through her or sideways beats rolling away. For example, 15 short at 88 AGL: towards 75 %,
  sideways 73-82 %, away 55 %. Rolling away keeps you under her descending body and paws, which travel the same way.

**Not modelled, so treat it as an upper bound:**
- For 14 and 15 she keeps re-reading your distance for the first 0.27 / 0.17 s of the window (section 9), and she
  keeps turning at 90-180°/s. Both follow a roll partly, most of all a roll away.
- The blend is fixed at all-short or all-long.
- Your hurtbox is one fitted capsule, not the 11-part hit model, from one roll at one equip load.
- The 2200 rows and the capsule axis are inferred.
- The game-speed rule is not measured live.

The planned logger ([Aava leap live test plan](../tools/aava-leap-live-test-plan.md)) records her colliders; a roll run would check this.

## Why it sometimes falls short

- You were already past the long leap's design distance (20 m for 14, 25 m for 15 and 16) when the measuring stopped.
- It was the grab (16), whose measuring stops earlier (0.67 s).
- You moved sideways after the take-off snap, beyond what about 90°/s of turning can follow.

## Method

`nmb.py pcsotfs c6790 nodes`, blend inputs / weights from the BlendN attributes (type 9), clip lengths from the
source-anim attribute (type 21, +0x88), `nmb_events.py c6790 125 126`, TAE via `tae_compare_pc.parse` on
`c679000_pl.tae`, the AI script dumped with `esd.py` (sub-machine ids kept), Ghidra headless `RefsTo` /
`DecompRange` / `FindOffsetUses` on DS2SPC.
