# Grab timing and i-frames (PC Scholar 1.0.3.0, live 18 Nov 2026)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

**Questions.**
1. Do grabs land with the same one-frame lag as normal hits ([phantom hitboxes](phantom-hitboxes-pc.md))?
2. Do grabs ignore i-frames, as an online claim says?

**Answers.**
1. **Same lag.** A grab catches on the first physics frame its hand really overlaps you, like any other attack (see
   [phantom hitboxes](phantom-hitboxes-pc.md)): one frame after the drawn hitbox reaches you, or two when the drawn
   hitbox only grazes you at first.
2. **No.** In 25 grabs where the hands overlapped a rolling player during invincibility, no catch happened while
   the invincibility was up. One catch landed on the very frame the invincibility ended, with the hands still on the
   player. The Ogre's grab is big and slow to clear: two 1 m spheres out for 14 frames. So invincibility that runs
   out while the hands are still around you is caught at once. To a player that looks like being grabbed through
   the roll, which is the likeliest source of the claim.

Eight 60-second runs were logged frame by frame and recorded on video, one video per run. The test plan, per-run
notes, logs and videos are kept with the source notes, not on this site (local: `grab-timing-test-plan.md`,
`findings\hitlog-2026-11-18\`).

## Setup

- **Enemy:** the Ogre (EnemyParam 300000) by The Crestfallen's Retreat. Its grab = EnemyDamageParam 130000310 and
  130000311.
  - Two spheres, radius 1.0 m (read live), on layer 26 like other attacks, both linked to GrabParam 130000310.
  - They spawn together and live 14 frames.
- **Player:** HP 2505, offline. A memory write kept HP from reaching 0 and topped it back up between grabs.
  - A roll keeps +0x608 up for **31 frames** (a 1 x3, 2 x3, 1 x25 pattern).
- **Logging and video:**
  - An outside reader logged every game frame: the player's 11-part hit model (the real hurtbox), the attack hitbox
    bodies, HP, the three invincibility fields (TimeAct flags +0x5F0 / +0x5F4 / +0x608) and the raw
    `ChrGrabActionCtrl`.
  - Each run was recorded at 60 fps. The log and the video share one clock, so a logged frame can be found in the
    video to within one frame (checked against the HP bar).
- **Catch markers:**
  - The HP drop (122 or 115 per catch).
  - Grab controller +0x100 / +0x108 (controller at PlayerCtrl +0x5C8 in this build). These change only on catches,
    always on the same frame as the HP drop.
  - +0x5F0 rising is **not** a reliable marker: it came 1-23 frames after the catch.

## 1. Detection: same as a normal hit (standing still, 4 runs, 10 catches)

P = the first frame the physics sphere overlaps the hit model in our geometry. D = the first frame the drawn
hitbox does.

| first-frame overlap at P | grabs | catch |
|---|---|---|
| 5.3-18.8 cm | 6 | on P |
| 0.8-2.7 cm | 5 | P + 1 (overlap then 18-21 cm) |

- **Drawn touch:** D = P - 1 in 8 of 10 (0 or -2 in the others).
- **Never earlier:** no catch came before a real overlap.
- **The 3-5 cm:** catches need about 3-5 cm of overlap in our model. Resolved later the same day: our geometry was
  right. Havok, with the game's settings, reports a contact on the same frame only if the overlap passes a speed-
  and size-dependent cut-off; otherwise it reports it one frame later. See [contact margin](contact-margin-pc.md).
- **In the roll runs:** the 7 unprotected catches followed the same rule (on the first overlap frame, or one later
  when that overlap was 0.6 cm).

## 2. I-frames: respected, with a sharp edge (late rolls, 4 runs, 33 grabs)

| case | grabs | caught |
|---|---|---|
| Hands overlapped only while +0x608 was up (by up to 104 cm) | 24 | **0** |
| +0x608 ended while the hands still overlapped (boundary) | 1 | 1, on the first free frame |
| Rolled early: +0x608 over before the first overlap | 5 | 5, at the first overlap |
| No roll | 1 | 1 |
| Missed by distance | 2 | 0 |

Also from the standing runs: one grab overlapped 13 cm while all three invincibility fields were up during a
get-up after a knockdown, and didn't catch.

**The boundary grab** (run 2, grab spawned at frame 33261):
- **+8 to +12:** overlap 2 -> 65 cm, +0x608 = 1. Nothing happened.
- **+13** (the hitbox's last frame): +0x608 = 0, overlap 74 cm. Caught (HP drop + grab controller).

This matches the end-of-roll rule for normal hits (6 Nov): a hitbox still touching you on your first unprotected
frame lands then. One clean example. Four more came within 5-10 frames of it on either side and behaved as the rule
predicts.

## How it looks (video)

- **Standing (run 1, grab 1):** the HP bar moves on the catch frame. The character snaps between the hands **2
  frames later**.
- **Boundary grab:** the hands close over the player at +6..+13. The player is lifted into the grip from +15,
  again 2 frames after the catch. Contact sheet kept locally: `hitlog-2026-11-18\boundary-grab-run2.png` (frames
  +6..+17; the player is partly hidden by a bush).
- **Total from drawn touch to visible hold:** about 3 frames (50 ms) on a normal catch.

## Not covered

- **One grab, one enemy:** only the Ogre's. Other grabs use the same hitbox type (all 184 grab rows are rigid
  bodies, 18 Nov params) but differ in size and lifetime. A small, short grab gives the boundary less room.
- **The grabber's side of the hand-over:** animation ids are still unknown.
- **Part B:** player backstabs / ripostes, which start from a position zone, not a hitbox.
- **Offline only:** network play adds its own delay.

## Method note

- **Read hitbox sizes fresh every frame.** The logger first cached each shape's radius by its memory address. The
  game frees hitbox shapes and reuses the addresses for new ones, so some sizes were stale: run 3 logged the grab
  spheres at 0.1 m, the size of another Ogre hitbox. Runs 1-2 were unaffected; run 3 was re-checked at 1.0 m, and the
  logger was fixed for runs 4-8. Logs taken with the old cache (including the 6 Nov hit-timing logs) should have
  their sizes checked against the damage parameters.
- Local working notes (tools changed for this test): `grab-timing-test-plan.md`.
