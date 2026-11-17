# "Phantom" hitboxes (LordRadai's video) checked against PC Scholar (2026-11-02)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

Source: Radai's follow-up video on hitbox timing (no audio; on-screen text read from the author's 56 screenshots,
`<local path> 2026-11-02 2058*-2110*.png`). It corrects his earlier clip,
which looked like hits landing a frame late. His claims:

1. Hurtboxes (character hit model) are one frame behind. The debug drawing matches the real data.
2. Normal hitboxes (`PXBulletRigidBody`) are also one frame behind, but the debug drawing shows where they will be next
   frame. Hitbox and hurtbox lag cancel out, so damage shows up the frame after the drawn overlap.
3. "Phantom" hitboxes (`PXBulletPhantom`) are made when the hitbox is expanding or can hit the same target more than
   once. They are on the current frame (one frame ahead of the hurtbox) and drawn correctly, so damage shows up on the
   frame of the drawn overlap. Proof: setting a hollow's attack to a hit interval > 0, or to expanding with a maximum
   radius of 0, makes it a phantom.
4. About 90 % of hitboxes are normal. Examples of phantoms: twinblade two-hand light attacks, FoFG flame pillars
   (multi-hit); Force, Wrath of the Gods, explosions, Soul Bolt, Focus Souls (size-changing).
5. Wheel skeleton rolls are not multi-hit hitboxes; they loop a short animation (blade dash) with an
   `InvokeWeaponDamage` TAE event.
6. PX is FromSoftware's physics layer over Havok 2014.1.

(A caption calls Wrath of the Gods "Satisfies 2"; it is the expanding case, his condition 1 / B.)

## Verdict

| Claim | Status | How |
|---|---|---|
| Classes `PXBulletPhantom`, `PXBulletRigidBody` (base `PXBulletBase`) exist | **Confirmed** | RTTI in DarkSoulsII.exe 1.0.3.0 |
| Phantom chosen for multi-hit OR expanding | **Confirmed in code** | FUN_1403b09d0, below |
| "Expanding" means the expand-time field, not the size (his max-radius-0 test) | **Confirmed in code** | the test reads +0x78 only |
| ~90 % normal | **Confirmed** (by row count) | 93.9 % of PC Scholar damage rows, 94.2 % of PS3 v1.00 |
| His examples are phantoms | **Confirmed** (data) | developer row names, below |
| Wheel skeleton roll is not multi-hit | **Confirmed** (data) | all its damage rows have cycle 0 |
| Normal hitbox one frame behind its target | **Confirmed live** (2026-11-06) | body position and rotation = last frame's target, every frame |
| Phantom on time | Mechanism in code only | phantom = immediate query; not tested live |
| Hurtboxes one frame behind | **Confirmed in effect, live** | the hurtbox is set to the new pose after the physics step, so the step tests the previous pose |
| Damage one frame after the drawn contact | **Confirmed live** | 8 of 12 clean hits exactly; never earlier |
| End of roll: blocked touch carried over | **Found live** | 12 of 12 hits right after i-frames; 2 were ghost hits |
| What DebugManager draws | Radai's statement only | the logged target is what it is said to draw |

## Code (DarkSoulsII.exe 1.0.3.0, DS2SPC Ghidra project, headless)

RTTI (MSVC x64): `PXBulletPhantom` vtable **0x1411e34d8** (ctor FUN_140bc1a50, dtor FUN_140bc1bd0, alloc 0x130),
`PXBulletRigidBody` **0x1411e4668** + 0x1411e46f8 at +0xf0 (an `hkpContactListener` base; ctor FUN_140bda8f0, dtor
FUN_140bda950, alloc 0x120), `PXBulletBase` 0x1411e78b8, `PXPhantom` 0x1411e7948, `PXRigidBody` 0x1411e3c88. Havok:
hkpSimpleShapePhantom 0x141229738, hkpCachingShapePhantom 0x14122f708, hkpAabbPhantom 0x141127748, hkpRigidBody
0x141126578.

**Selector FUN_1403b09d0** (called by FUN_1403186a0, FUN_1403f1ab0, FUN_140446270) gets a request whose +0x00 is
the damage id; it looks the DAMAGE_PARAM row up in the damage tables ([[0x1416148f0]+0x10]+0xb40, the same container
as the distance-cut work) and branches:

```c
row = lookup(table + 0xb40, request[+0x00]);
if (0.0 < request[+0x34] || (row && 0.0 < row[+0x78]))   new PXBulletPhantom   (FUN_140bc1a50, init FUN_140bc1eb0)
else                                                      new PXBulletRigidBody (FUN_140bda8f0, init FUN_140bdada0)
```

The caller FUN_1403f1ab0 builds the request from the same row: +0x33 shape (1 = capsule -> length from +0x38),
+0x34 radius, +0x0c/+0x10/+0x14 damage with types at +0x19..+0x1b, and **request +0x34 = row +0x3c**. With the
Paramdex layout (paramdef.py, 124-byte DAMAGE_PARAM): +0x3c = `cycle_period` (DebugManager: "Continuous Hit
Interval"), +0x78 = `hitbox_expand_speed` ("Expansion Time"). `hitbox_max_size` (+0x74, "Expansion Size") is not
tested, which is why Radai's expanding-with-max-radius-0 test still made a phantom.

So: **phantom <=> cycle_period > 0 or hitbox_expand_speed > 0.** A second phantom creation site is FUN_14012e900
(a vtable slot, data refs 0x14136c2ec / 0x14136c2fc, also called by FUN_140445020; it needs an object's +0x34 > 0 and a
type byte +0xb1 or +0xb2 == 3). What it builds is not identified.

**Per-frame update** (vtable slots 8 and 9, shared base FUN_140bdd640 / FUN_140bdd280):

- `PXBulletRigidBody` slot 8 = FUN_140bdafe0(this, dt): reads the Havok body's actual position (+0x1a0) and rotation
  (+0x1e0), stores them at +0x40 / +0x50, then sets linear velocity = (target +0x20 - actual) x 1/dt and the matching
  angular velocity (motion vfuncs +0x98 / +0xa0). That is Havok's hard keyframe: the body only gets to this frame's
  target during the next physics step, and hits arrive through the contact listener during that step. Up to two
  bodies (+0x108, +0x110; the second on layers 0x33 / 0x34 when a request flag is set), like the PS3 demo's fat
  damage capsule + thin weapon check.
- `PXBulletPhantom` slot 9 = FUN_140bc22d0 -> FUN_140bc1f70: places the phantom at last frame's target and, right then,
  runs a penetration query (shape vfunc +0x90) and a linear cast to this frame's target (+0x88) with an
  `hkpAllCdPointCollector`, handing the points to FUN_140bc2500. Long moves are split into steps (count from
  FUN_140bc2640). Then +0x40 = +0x20.

So the rigid body's hits are found one physics step after its target is set; the phantom's are found in the same
update, swept along the whole move. That is the mechanism behind claims 2 and 3. Not traced: where slots 8 / 9 sit
relative to the hkpWorld step in the frame, how contact-callback hits become damage, and whether the character hit
model is hard-keyframed too (claim 1). Those decide that the gap is exactly one frame.

## Data

Rule applied to all three damage tables (`cycle_period > 0 or hitbox_expand_speed > 0`):

| Build | Phantom rows | Normal |
|---|---|---|
| PC Scholar 1.0.3 | 665 / 10,842 (Player 150 / 3,082, Enemy 430 / 7,497, System 85 / 263) | 93.9 % |
| PS3 v1.00 | 465 / 8,047 (Player 88 / 2,359, Enemy 312 / 5,498, System 65 / 190) | 94.2 % |

One player row and one enemy row in each build have a max size but no expand time, so they stay normal.

Named examples (PC Scholar ids; names = developer row names from the 31 Mar 2014 PC build):

- Multi-hit: twinblade 2H light 1 / 2 (双刃剣_両手近接_弱1 10015900, 弱２ 10016101-10016201) and also twinblade 1H
  light 2, rolling attacks and 2H heavy 1; spear and lance heavies / dashes; UGS 1H light 2; Firestorm pillars
  (炎の嵐（火柱） 33050030), poison mists, 雷の源 ("source of lightning") 32180010, 渦巻く魔力球 31150000 (probably
  Soul Vortex), butterfly armour scales; 85 map / system
  hazards (SystemDamageParam 200001011-330040000, intervals 0.1-5 s).
- Expanding: Force 32100001 (フォース（弾）, max 3.5 m, 0.2 s), Wrath of the Gods 32110001 (max 4.5 m), Emit Force,
  Lightning Spear / Great / Sunlight Spear impacts, Great / Chaos Fireball impacts, Old Sun Ring explosion, the urns
  (魔力瓶 / 雷瓶 / 炎瓶 / 黒炎瓶 / 闇瓶 小-爆発 60550010-60620010).
- Both: Soul Bolt 31160010 / 31160011 (魔力の閃光, max 70 m), 31310013-31310044 (unnamed; likely Focus Souls),
  dragon head stone breath, dark fog.
- v1.00 has the same Force / WoG / Soul Bolt / twinblade rows, but its fire urn explosion (60570010) is a fixed 1.3 m
  sphere with no expand time, so in v1.00 urn explosions are normal hitboxes.

Wheel skeleton (EnemyBehaviorParam 133000 / 133010 / 133020, 車輪スケルトン 素手 / 曲剣 / 大曲剣): every damage row
it uses (EnemyDamageParam 112100010 / 020 / 030) has cycle 0 and no expansion, so repeated roll hits must come from
re-triggering the attack (Radai's blade-dash loop), not from one multi-hit box.

## PS3 Network Test side

- 2026-10-20 scan (side session "Draw attack hitboxes as 3D wireframe capsules"): melee attack capsules are
  hkpRigidBody (vtable 0x19e19a8), the "normal" kind. They kept a steady offset to the **same-frame** hand capsule of
  the hit model (layer 3). That session read this as "no lag against the skeleton", but by Radai's account the hit
  model is itself a frame behind the pose, so the test only shows that hitbox and hurtbox move in step. That fits
  claim 2's "they cancel out" and cannot see the lag.
- The demo does have Havok phantoms (hkpSimpleShapePhantom 0x19e1e48, hkpCachingShapePhantom 0x19f3518,
  hkpAabbPhantom 0x19e1d30) but no `PXBullet*` RTTI names (only PXUserData* classes), so its wrapper may differ.
- `tools/hitboxes.py` and the overlay match only the hkpRigidBody vtable, so **they cannot show phantom hitboxes**.
- The demo's DAMAGE_PARAM rows do not fit the community layout (99.9 % "flagged", radius 0), so the phantom rule
  cannot be applied to NT data until its layout is worked out.

## Live test (2026-11-06, PC Scholar 1.0.3.0, main Steam install, Steam offline)

The author's character (all stats 99, 15-frame roll i-frames) against hollow soldiers in the Forest of Fallen Giants
(EnemyDamageParam 1102011xx / 1102012xx / 1102021xx-1102024xx) and then a large knight (1330012x0 / x1, 1331012x1:
capsules r 0.6 m x 2.2 m), all normal rigid hitboxes. Logged every game frame (~59 / s) with `tools\hitlog_pc.py log`,
analysed with `tools\hitlog_analyse.py rolls` and short scripts. Logs in `hitlog-2026-11-06\`: `run1-standing` (30 s,
hits), `run2-running` (10 s, straight run), `run3-standing-geom` (30 s, hits), `run4-rolls` (60 s, rolls through
hollow swings), `run5-rolls` (60 s, a bonfire warp, no swings: discarded), `run6-knight-rolls` (60 s, 24 rolls through
the knight's swings). Death guard: `pcmem.py nodeath 3600 topup` (writes logged; its HP top-up to the 797 cap, clamped
back to 598 by the game, shows up in the logs as fake HP changes, not hits).

**Frame order (code, GameManagerImp update FUN_1401bf200):** characters part 1 -> PXWorld [+0x660] step (FUN_140bc0e90:
pre-step listeners, hkpWorld step; FUN_140bc0f20: post-step listeners) -> characters part 2. Each frame's snapshot is
read ~6 ms after GameManagerImp +0x104 (a frame counter, +1 per game frame) moves, checked by a second read (2 of
7,722 frames unstable).

**Found live:**
- Damage colliders: DamageManager [[BaseA]+0x10] vector +0x10 / +0x18, 0x128-byte records, PXBullet* at +0x08, damage
  id at +0x1c. Five map-hazard colliders near the spot (SystemDamageParam 210100110-150, cycle 0.15 s) are
  `PXBulletPhantom`: the multi-hit rule, live.
- **Correction 2026-11-11:** the hurtbox is NOT the ragdoll below. The game's collision filter (`PXGroupFilter`, 64 x 64
  layer table at filter +0x48, read live) has attack hitboxes on layer 26, which collides with layer 3 and not with
  layer 5. Layer 3 is the 11-part hit model, a `CharacterRigidBody` (vtable 0x1410e0798) at [[CharacterCtrl+0xE0]+0x1F8]
  for the player and +0x1B8 for enemies; its bodies are `PXKeyframedRigidBody`. The ragdoll and the hit model overlap
  closely, which is why the hits below fitted. The hurtbox timing results in this file were measured on the ragdoll and
  have not been repeated on the hit model. See `DS2 Practice Tool\DLL-PLAN.md` section 9.
- Hurtbox: the 18 ragdoll capsules of [[PlayerCtrl+0x100]+0x458] (hkaRagdollInstance; layer 5, group 0x21; radii
  0.042-0.135 m; body 0 = pelvis). These fit the hits (below), so they are taken as the hit model.
- Your own roll makes a 3-frame rigid collider, damage id 9000 (probably the pot / barrel breaker); excluded.

**1. Hitbox lag (run 1, 453 frame pairs with the target moving 5 cm+):** the physics body is at **last frame's target**
(median 0.0000 m, max 0.155 m), a median 0.160 m (max 0.392 m) short of this frame's target. Rotation the same (run 3,
237 pairs, largest gap 0.0045). Claim 2 confirmed.

**2. Hurtbox (run 2, 404 running frames at 0.101 m / frame):** at the end of a frame the pelvis capsule is on **this**
frame's root (0.018 m, body sway) and 0.105 m from last frame's. It is set to the new pose in characters part 2, after
the step, so the next frame's physics step tests it at the previous pose, the same pose the hitbox body has then.
Both lag by one pose: Radai's "they cancel out".

**3. When damage lands (runs 3, 4, 6: 12 clean hits with i-frames off):** pairing "hitbox body at N vs hurtbox at the
end of N-1" (what the step at N sees):
- 8 hits: drawn hitbox first touches at F-1, physics first touches at F, HP drops at F.
- 1 hit (29783): both short by 6 mm at F; contact tolerance / noise.
- 3 hits (31617, 31750, 72761): physics touch at F-1, HP at F. Unexplained (not i-frames, not a new collider, not a
  capsule that never counts).
- Never before the physics touch; never on the drawn-touch frame.

**4. End of roll (runs 3, 4, 6: 12 hits right after a roll's i-frames):**
- The damage check uses the **current frame's** i-frame counter. All 12 boundary hits landed on the first frame whose
  counter was 0, while the geometry being tested is one pose old.
- A touch blocked by i-frames is **not used up**, but only a touch on the **last protected frame** carries over. All
  12 boundary hits had the hitbox touching on the last protected frame and still alive on the next.
- No hit followed in these counter-cases:
  - touch on the last protected frame, but the hitbox was deleted that frame: rolls 28965 and 72551;
  - last touch 1, 3 or 4 frames before the end, hitbox alive afterwards: rolls 71730, 22725 and 72164.
- **Ghost hits: 2 of the 12.** The contact from the last protected frame was applied on the next frame although the
  hitbox had already moved clear:
  - 30372 (hollow): physics 0.55 m and drawn 0.70 m clear at the hit frame; drawn already 0.56 m clear on frame 0;
  - 71107 (knight): touch on the last protected frame only; physics 1.43 m and drawn 2.84 m clear at the hit frame.
- In the other 10 the hitbox was still touching on the hit frame.
- In effect, i-frames end one frame early against an ongoing contact. Mechanism not traced: the PXBullet hit list
  (FUN_140bdd280) or Havok keeping the contact for a step.

**5. Start of roll (checked 2026-11-07, same logs, 46 windows, all 15 frames; counter 1,1,1,2,2,2,1 x 9):**
- No HP drop on any frame with the counter on, and no extra protected frame before it: the counter is used as is.
- The first counter frame protects: run 6 roll 72272, knight hitbox h829 appeared 0.46-0.50 m inside the player on the
  frame before the counter came on and stayed in for 10 frames: no damage. Run 4 roll 31442 the same (hollow, 0.37 m,
  appeared on the first protected frame). A hitbox seems to join the physics step only on the frame after it
  appears (body = its spawn target then), so its first frame never hurts.
- Against the drawn models the window still sits one frame early, from the one-frame lag: the first counter frame
  judges the positions drawn on the frame before it, and the last protected pose is judged on the next frame, when
  the counter is 0. With the carry-over (4), a blade touching on the second-last protected pose also lands. No clean
  example of a non-new hitbox drawn-touching exactly one frame before the counter is in the logs; this follows from
  rules 3 and 4.
- Mirror case on the attack side: h827 (knight, same roll) was drawn 0.66 m inside the player on its last frame but
  never hit: its physics body was still 7 mm short and the hitbox was deleted before the step that would test its
  last position. A hitbox's last drawn position is never tested.

Caveats: two enemy types, one character, 12 boundary hits and 12 clean hits; ~59 fps; phantoms and PS3 builds not
tested.

## Still open

- Phantom timing live (Force, a twinblade two-hand light, a map hazard): expect HP on the drawn-contact frame.
- Why 31617 / 31750 / 72761 hit a frame after the physics touch; what keeps a contact listed one frame after
  separation (the PXBullet hit list FUN_140bdd280 or Havok's contact persistence). Possibly one cause: the hit list
  being a frame stale.
- A controlled i-frame hold / release (write +0x5F0) would give boundary hits on demand.
- Leading edge (DS2 i-frames "come out early"): plan in `leading-edge-test-plan.md` (2026-11-07, not run).
- PS3 demo: add the phantom vtables to hitboxes.py / the overlay (TODO "Phantom hitboxes" follow-up (d)).
